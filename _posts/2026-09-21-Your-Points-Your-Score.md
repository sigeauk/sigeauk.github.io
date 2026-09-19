---
layout: post
title: "Your Points, Your Score: Rule Health, Grown Up"
date: 2026-09-21
author: Sigea Team
description: "Discover how the new percentage based, fully customizable rule scoring model in TIDE brings clarity, adaptability, and explainability to SOC detection engineering."
categories: [Cybersecurity, Detection Engineering, Workflow]
tags: [TIDE, Rule Health, Detection Engineering, ElasticSearch, Kibana]
---

A detection rule can be beautifully documented, tagged, and signed off, yet still never fire simply because the index pattern it queries lacks the underlying fields in ElasticSearch. Measuring the operational readiness of a detection library requires a scoring model that adapts to the specific priorities of your security operations center.

<span class="text-accent">**TIDE**</span> (Threat Informed Detection Engine) is an open source, standalone, containerized platform that audits detection rules in a SIEM such as Elastic or Kibana and maps them to MITRE ATT&CK. The initial Rule Health score introduced a solid framework by balancing Quality and Meta criteria on a fixed 100 point scale. As detection engineering practices mature, scoring models must offer greater precision, custom weighting, and transparent diagnostics. The upcoming release of <span class="text-accent">**TIDE**</span> elevates Rule Health into a fully customizable, percentage based engine designed to fit any operational environment.

<div class="gallery-item"><img src="{{ site.baseurl }}/images/rule-scoring/01-rule-health.png" alt="Rule Health cards showing a score ring per rule"></div>

---

## 1. Evolution of Rule Scoring

The first iteration of the scoring engine established an automated baseline across core detection properties: checking field mappings, query structure, author details, and investigation guides. That foundation proved the value of continuous rule auditing. Building on how the first version behaved in practice, the next release introduces key enhancements that make rule scores even more accurate and meaningful:

* **Weighted proportion over rigid additions.** Rather than assuming every SOC values investigation guides or MITRE tags identically, point allocations can now be customized per team.
* **Context aware handling of missing metrics.** When an index pattern cannot be evaluated or a rule has never executed, those metrics drop out of the calculation gracefully rather than acting as automatic failures.
* **Enhanced query and field parsing.** The underlying parser now isolates string literals, URLs, and file paths inside quotes more effectively, preventing arbitrary query parameters from distorting mapping validation.
* **Stable performance tracking.** Search time evaluation now utilizes a median measurement over recent runs, smoothing out single query spikes and providing a reliable view of operational efficiency.

---

## 2. The New Score in One Idea

The refreshed scoring engine calculates health using a dynamic proportion:

> **score % = points earned / points that apply x 100**

Each check returns a fraction from 0 to 1 representing performance, or returns **n/a** when a check cannot be evaluated. 

Every client defines how many points each check is worth. If a check is allocated 0 points, it is turned off and disappears from the evaluation. Crucially, any **n/a** check is excluded from the denominator rather than counted as a zero. Allocating 55 total points across your active checks still yields a clear percentage out of 100.

| Check | Default Points | How It Is Scored |
|---|---|---|
| **Mapping** | 37 | Share of rule fields existing in mapped index patterns. Unreachable patterns are omitted. |
| **Field Type** | 12 | Evaluates field suitability for detection (keyword scores 100%, text or numbers 50%). |
| **Investigation Guide** | 15 | Scaled by character length (600+ characters earns full points). |
| **MITRE Mapping** | 10 | 30% for tactic assignment, 70% for technique coverage. |
| **Search Time** | 8 | Median duration of recent runs. Returns n/a until executions occur. |
| **Timestamp Override** | 8 | Full score when configured to `event.ingested` and present in the mapping. |
| **Author** | 5 | Full score when populated. |
| **Highlighted Fields** | 5 | Full score when set and verified against active mappings. |

Eleven additional checks exist at 0 points by default, including schedule coverage, rule freshness, false positive references, alert suppression, and query language preference.

---

## 3. Your SOC, Your Points

Customizing scoring criteria takes place directly inside **Management** under the **Linked SIEMs** section.

<div class="gallery-item"><img src="{{ site.baseurl }}/images/rule-scoring/04-scoring-editor.png" alt="Rule scoring editor with a points box per check"></div>

The **Rule scoring** panel organizes checks into three logical groups: *Quality*, *Operations*, and *Meta*. Adjusting point values recalculates rule health immediately across the entire library without querying the SIEM again, because <span class="text-accent">**TIDE**</span> retains the raw per-check fractions locally.

<div class="gallery-item"><img src="{{ site.baseurl }}/images/rule-scoring/05-editor-55-points.png" alt="Editor with 55 points allocated to Mapping and a re-score confirmation"></div>

When focusing exclusively on data availability by allocating 55 points to Mapping and setting all other checks to 0, the rule window adjusts to display only active checks while maintaining a complete percentage scale.

<div class="gallery-item"><img src="{{ site.baseurl }}/images/rule-scoring/06-only-mapping.png" alt="Rule window listing only the Mapping check at 100%"></div>

---

## 4. A Score You Can Explain

Transparency is central to the redesigned rule experience. Opening any rule window reveals a detailed **Score breakdown** along with the active **model v2** tag.

<div class="gallery-item"><img src="{{ site.baseurl }}/images/rule-scoring/02-score-breakdown.png" alt="Score breakdown listing each check with its points"></div>

Hovering over any check reveals both the underlying scoring logic and the rule's specific values, such as exact character counts or missing field names. 

Historical tracking is equally clear. The score chart records changes over time and displays clear markers whenever the underlying scoring model or client point allocations are updated.

<div class="gallery-item"><img src="{{ site.baseurl }}/images/rule-scoring/03-score-chart.png" alt="Score-over-time chart with a marker where the client's points changed"></div>

> A score is only useful if you can explain why it moved.

---

## 5. Watch It Move: The Walkthrough

To see the system in action, consider a test rule named `DEMO - Encoded PowerShell Command Execution` targeting three fields (`process.name`, `process.args`, `process.parent.executable`) against an index pattern in ElasticSearch.

The **Sync this rule** action triggers a targeted sync that refreshes index mappings and re-scores just that one rule, without waiting for a full sync of the library.

When the target index is missing entirely, every field is reported as having no index and the Mapping check earns nothing. Because the score is worked out from the checks that can be evaluated, the rule lands at **38%** despite complete documentation, and Field type and Search time show **n/a** rather than dragging it down further.

<div class="gallery-item"><img src="{{ site.baseurl }}/images/rule-scoring/07-demo-38.png" alt="Demo rule at 38% because its index does not exist"></div>

Once the underlying index mapping is fully populated, running **Sync this rule** again moves the score to **100%**. The rule now finds all three fields on the `winlogbeat-*` pattern, and a field counts as long as any of the rule's index patterns has it.

<div class="gallery-item"><img src="{{ site.baseurl }}/images/rule-scoring/08-demo-100.png" alt="Demo rule at 100% after the mapping is fixed"></div>

To keep syncs fast, <span class="text-accent">**TIDE**</span> remembers each SIEM's index mappings for 24 hours, reading field definitions across all matching indices. Using **Sync this rule** or a full mapping sync refreshes those remembered details straight away whenever the underlying schema changes.

---

## 6. Keeping Expectations Honest

While the updated model brings significantly greater flexibility, a few operational details are worth noting:

* Point allocations are set per client rather than per individual SIEM connection or specific rule.
* Historical scores generated under different model versions or point weights reflect their respective configurations and are marked on trend charts accordingly.
* Index mapping changes appear at the next refresh of the remembered mappings, or immediately when using **Sync this rule**.

---

## Availability

These scoring updates are coming in the next release of <span class="text-accent">**TIDE**</span>.

<div class="grid btn-grid"><div class="cta-container"><a href="https://github.com/sigeauk/TIDE" class="btn-primary">Try TIDE today</a></div></div>
