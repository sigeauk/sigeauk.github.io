---
layout: post
title: "Succeeded, Caught Nothing: Rule Monitoring in TIDE 6.4.0"
date: 2026-11-13
author: Sigea Team
description: "Kibana tells you when a detection rule fails. TIDE 6.4.0 tells you when it succeeds and still can't catch anything: quiet data sources, empty fields, late events, noisy and silent rules, exception sprawl, and how much of your coverage is actually effective."
categories: [Cybersecurity, Detection Engineering, Workflow]
tags: [TIDE, Rule Monitoring, Detection Engineering, ElasticSearch, Kibana]
---

A detection rule that never fires looks exactly like a rule with nothing to catch. Kibana will tell you, loudly, when a rule fails to run. It will not tell you when a rule runs perfectly every five minutes against a data source that stopped sending on Tuesday. The run succeeded. The rule caught nothing. Both statements are true, and only one of them is reassuring.

<span class="text-accent">**TIDE**</span> (Threat Informed Detection Engine) already watched for the loud failures: failed runs, missing indexes, fields the mappings don't have, blind windows between runs. Release 6.4.0 teaches its **Rule Monitoring** to notice the quiet ones, and then uses what it finds to show how much of your coverage is *effective* rather than merely green.

---

## Meet Olympus Freight

To show it working we built a demo client, Olympus Freight, and let the Olympians write its detection rules. Athena wrote careful, well-documented rules. Artemis hunts credential theft. Ares counts failed logons, obviously. Dionysus wrote a rule that fires on anything run from a user's profile folder, because why not. Poseidon looks after the firewall feed. Hermes, being the god of thieves, copied Athena's PowerShell rule under a new name.

<div class="gallery-item"><img src="{{ site.baseurl }}/images/tide/rule-health.jpg" alt="Rule Health: thirteen rules by the Olympians, scored from 100 down to 37"></div>

Twelve of the thirteen are enabled (Hera's is switched off), and their runs report success, give or take Zeus's, which searches an AWS index nobody ever created, and Dionysus's, which ran out of room for alerts. On Kibana's view of the world, Olympus Freight is in decent shape. Here is what Rule Monitoring makes of it.

<div class="gallery-item"><img src="{{ site.baseurl }}/images/rule-monitoring/01-figures.png" alt="Rule Monitoring's figures: 12 rules needing action, 2 errors, 7 warnings, 3 info"></div>

Twelve of the twelve enabled rules need something. Findings are grouped by their cause, so a single broken pipeline shows up as one thing to fix rather than forty identical rows, and each says who fixes it: the **Engineer** who owns the data, the rule's **Author**, or TIDE itself.

---

## 1. The source that went quiet

An index pattern can match plenty of indexes while no new event has reached any of them for days. Every rule searching it then finds nothing, without an error in sight.

At each sync, TIDE now asks the SIEM for the newest event in every index pattern your rules search, using the time field each rule actually searches by. Poseidon's firewall feed last delivered three days ago:

<div class="gallery-item"><img src="{{ site.baseurl }}/images/rule-monitoring/02-quiet-source.png" alt="No recent events in the index pattern: logs-network.firewall-*, newest event three days before TIDE looked"></div>

It is an **error** when none of a rule's index patterns has recent data, since the rule cannot raise an alert at all, and a **warning** when its other patterns still do. New data is expected every 24 hours by default. Not every source is chatty, so **Expect new data every…** gives one pattern its own window, and **Don't expect new data** retires it politely.

---

## 2. The field nobody fills

Index templates map most of the fields a source *might* send. That makes a field look present in the mappings, which is all a mapping check can see, while no event ever carries a value in it. A rule matching on it can never fire.

Hephaestus built a sound rule for `svchost.exe` started by anything other than `services.exe`. It relies on `process.parent.command_line`, which the endpoint index maps beautifully and never fills:

<div class="gallery-item"><img src="{{ site.baseurl }}/images/rule-monitoring/03-empty-field.png" alt="Field mapped but has no data: process.parent.command_line has no value in the last 7 days of events"></div>

Once a day, and whenever you choose **Sync this rule**, TIDE checks whether any of the last seven days' events has a value in each field your enabled rules match on. A field that another of the rule's index patterns does fill isn't listed, and a pattern that has gone quiet is reported as quiet instead, so you only fix each problem once.

---

## 3. The events that arrive late

A rule searches back over its run interval plus a little extra, by when each event *happened*. An event that reaches Elasticsearch later than that extra margin falls between two runs and is never searched by either. Nothing anywhere says so.

Each sync now measures how late events arrive in every index pattern your enabled rules search: the 95th percentile of `event.ingested` minus `@timestamp` over the last hour's arrivals. Hestia's rule for changes to `authorized_keys` looks back one minute beyond its five-minute interval. Her auditd collector delivers events about forty minutes late:

<div class="gallery-item"><img src="{{ site.baseurl }}/images/rule-monitoring/04-late-events.png" alt="Events arrive later than the rule looks back: 95% of events arrive within 48 minutes, the rule looks back 60 seconds"></div>

The fix is usually to search by `event.ingested` (rules that already do aren't affected, and aren't listed), or to give the rule more look-back than the delay, and to have a word with whatever is queueing the logs.

---

## 4. Too loud, too quiet, too sudden

Each sync now counts the alerts every rule raised each day for the last 90 days, through Kibana. The rule window gets an **Alerts** section, and Rule Health's table an **Alerts (7 days)** column to sort by.

Dionysus's rule did exactly what it says on the tin:

<div class="gallery-item fit"><img src="{{ site.baseurl }}/images/rule-monitoring/09-alerts.png" alt="The rule window's Alerts section: 1,079 alerts, all open"></div>

Three findings come out of those counts:

* **Too many alerts:** a rule averaging more than 100 alerts a day over the last seven days, or making more than half its space's alerts. Dionysus manages 154 a day and 99% of the space's alerts, which is an achievement of sorts.
* **No alerts for a long time:** an enabled rule, enabled for at least 30 days, that has raised nothing for 90. Rules whose data or fields are already listed as a problem aren't listed again here.
* **Alerts spiked:** more than three times the rule's usual day (and at least 50), against its median day over the previous two weeks. It names the last change TIDE recorded to the rule in that time, such as who edited which fields, or says none was recorded, which points at the data instead.

<div class="gallery-item fit"><img src="{{ site.baseurl }}/images/rule-monitoring/10-rule-window-monitoring.png" alt="The rule window's Rule Monitoring section: alerts spiked, too many alerts, never validated"></div>

The noisy and silent thresholds can be changed per client. The defaults are a starting point, not a moral position.

A quieter companion arrives too: **Searches getting slow** lists a rule whose searches take half the time between its runs or more, or have more than doubled in two weeks, before Kibana starts skipping its runs. The rule window's **Last run** now charts the search time of each run.

---

## 5. Exceptions: where coverage goes to die

Each exception is reasonable on its own. Together, they can mute a rule entirely, and nobody reviews them because nobody can see them. Each sync now reads the exception lists your rules use, and the rule window has an **Exceptions** section listing every item in words, with who added it and when.

<div class="gallery-item fit"><img src="{{ site.baseurl }}/images/rule-monitoring/11-exceptions.png" alt="The rule window's Exceptions section: one item matches almost anything"></div>

Hermes's scheduled task rule has a "temporary" exception. You have met this exception before. It matches `process.executable` against `*`, which is to say everything, and the rule has not raised an alert since:

<div class="gallery-item"><img src="{{ site.baseurl }}/images/rule-monitoring/06-wildcard-exception.png" alt="Exception matches almost anything: the item excepts every event where process.executable matches *"></div>

Rule Monitoring now lists **Exception matches almost anything** for an item made only of wildcards, **Many exceptions** for a rule with more than ten items (Apollo's service account rule has twelve approved jump hosts, each perfectly sensible), and, as information, **Exception added recently**. Any change to an item in Kibana goes into the history of every rule using that list, and if a rule's alerts spike after an exception is removed, the removal is named as the likely cause.

---

## 6. The same rule, twice

A re-import, or one copy from Sigma and another from Elastic, and every event alerts twice. Worse, tuning one copy leaves the other as it was. Hermes copied Athena's rule and gave it a new name, which fooled nobody:

<div class="gallery-item"><img src="{{ site.baseurl }}/images/rule-monitoring/07-duplicate.png" alt="Same detection deployed twice: two rules with the same query on the same index patterns"></div>

**Same detection deployed twice** groups enabled rules in one space with the same type, query (spacing aside), index patterns and the settings that decide what they match. Copies you keep on purpose, such as a rule and the copy you are tuning, can be linked to each other, and then aren't listed.

---

## 7. Drift

Rules age, and their originals move on. **Upstream update not reviewed** lists a rule whose Elastic or Sigma original changed more than 30 days ago without anyone reviewing the update. **Retired upstream** lists one whose original has been deprecated. Elastic's prebuilt rules, which Kibana updates for you, become one finding per destination rather than hundreds. And if the client turns it on, **Validation expired or out of date** lists every enabled rule that has never been validated, whose validation has lapsed, or that was edited since someone last signed it off.

Not all of that is a fault, so Rule Monitoring now has an **Info** level beside Error and Warning, in its own blue, with its own card on the page and the Dashboard.

---

## 8. Effective coverage

All of this would be a tidy to-do list if it stopped there. It doesn't. A rule mapped to a technique counts towards a system's coverage, and until now it counted whether or not it could actually work.

A system's page now shows **Effective** beside **Covered**. Effective leaves out techniques covered only by rules Rule Monitoring lists an error for: an index pattern matching nothing, data that has stopped arriving, a field missing or never filled, a query Kibana can't run.

<div class="gallery-item"><img src="{{ site.baseurl }}/images/rule-monitoring/12-covered-vs-effective.png" alt="Freight Operations: 27% covered, 20% effective, 2 not effective"></div>

Freight Operations, baselined against APT33, is 27% covered and 20% effectively covered. Zeus's AWS rule is mapped to *Cloud Accounts* and is perfectly well written. It searches an index that doesn't exist:

<div class="gallery-item fit"><img src="{{ site.baseurl }}/images/rule-monitoring/13-not-effective-card.png" alt="T1078.004 Cloud Accounts: covered, but not effective"></div>

The technique says **Not effective**, and hovering over its rule says why. Nothing about the mapping changes: the moment Rule Monitoring no longer lists the problem, the rule is effective again. System and threat reports carry the same distinction, along with a **Rule Health** section listing the open findings on the rules behind their figures, so the number in the board pack is the one you can defend.

> A green tick should mean a rule that can still fire.

---

## 9. Seeing it where you work

You don't have to visit Rule Monitoring to find out. A rule's card in Rule Health now says how many problems were found with it, with each one listed when you hover, and the rule window has a **Rule Monitoring** section with each finding in full, who fixes it, and for how long.

<div class="gallery-item fit"><img src="{{ site.baseurl }}/images/rule-monitoring/14-rule-card.png" alt="Zeus's AWS rule card: scored 37%, with an error, a warning and info"></div>

---

## 10. Keeping it current

Rule Monitoring is only as current as the last sync, and until now a sync happened when someone clicked **Sync**. Super admins can now turn on a **Maintenance window**: on the days and at the time you choose, TIDE syncs each client's destinations in turn, makes the backup, and compacts any database file that has grown bloated, with fifteen minutes' notice to anyone signed in and a **Delay** button for the person halfway through a change. It is off until you turn it on.

---

## Keeping expectations honest

* Every finding is as of the last sync. Data freshness and late events are measured at each sync; the empty-field check runs daily, or on **Sync this rule**.
* Alert counts are read through Kibana, so the SIEM's API key must be able to read each space's alerts. When it can't, Rule Monitoring says so on the destination rather than the sync failing.
* Thresholds are per client. Olympus Freight might tolerate Dionysus; you might not.
* Findings are prompts, not verdicts. Anything can be muted, a single rule or a whole cause, with a reason, and **Raise issue** hands a finding to the person who fixes it, as a GitLab issue if you like.

---

## Availability

All of this is in <span class="text-accent">**TIDE**</span> 6.4.0, out now. Next up, 7.0 brings Wazuh's rules, agents and hardening into the same picture, and shows MITRE ATT&CK's own guidance on how to detect each technique, down to the log sources each system needs. [Take a look]({{ '/tide' | absolute_url }}).

<div class="grid btn-grid"><div class="cta-container"><a href="https://github.com/sigeauk/TIDE" class="btn-primary">Try TIDE today</a></div></div>
