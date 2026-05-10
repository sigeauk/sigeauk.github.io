---
layout: post
title:  "Detection as Code: Moving beyond the 'Tick-box' with TIDE"
date: 2026-05-11 00:00:00 +0100
author: Sigea Team
categories: [Use Case, Enterprise Security, Workflow]
tags: [CISO, Detection Engineering, Sigma, DaC]
---

**Picture this:** It’s a quiet Tuesday afternoon. Suddenly, the news breaks, a new state sponsored cyber actor out of Iran is making waves. Your CISO immediately reaches out with the inevitable question: **"Are we protected against their TTPs?"**

In the past, this scenario triggered a massive fire drill. It meant days of manual spreadsheet mapping, guessing, and hoping you didn't miss a critical gap. The old way wasn't necessarily wrong; it was just an evolving process. A while back, I tackled this exact friction by stringing together GitLab pipelines, Elasticsearch, Kibana, and OpenCTI to run Python scripts, a stack affectionately named <span class="text-accent">GEKO</span>. It got the job done, but it eventually became clear that managing security rules needed a dedicated, visual workflow.

That evolution led to the creation of <span class="text-accent">TIDE</span> (Threat Informed Detection Engine). <span class="text-accent">TIDE</span> takes the core philosophy of Detection as Code (DaC) and pairs it with a "Human-in-the-Loop" interface to turn theoretical threat models into live defenses.

**What is Detection as Code and Why Does My Security Team Need It?**
At its core, Detection as Code treats your security rules like software. But the real business value of DaC isn't just about version control, it’s about real world coverage vs. a tick-box exercise.

Having a rule mapped to MITRE T1078 (Valid Accounts) might show up as a green checkmark on a traditional audit, but it doesn't mean you are fully covered against an adversary's specific techniques. Furthermore, just because an actor targets a specific technology doesn't mean you need to write a rule for it if that technology doesn't exist in your environment.

With <span class="text-accent">TIDE</span>, you can actively pull threat intelligence (Mitre ATT&CK built in and live ingest from OpenCTI) for any specific actor and create a "baseline" (attack tree). <span class="text-accent">TIDE</span> instantly allows you to map your current rules to this actor's known MITRE techniques, suggesting rules based on those gaps. It shifts the team from asking **"Do we have rules?"** to **"Do we have the right rules for the specific threats we actually face?"**

**How Can Detection as Code Improve Our Security Response Time?**
When we talk about response time, the immediate focus is usually on the SOC analyst. However, the true bottleneck often lies in Engineering Speed.

Writing, testing, and deploying a new rule in a traditional UI can take days. DaC streamlines this workflow entirely. With <span class="text-accent">TIDE</span>, when you identify a gap in your coverage, the process is incredibly fast:

**One-Click Sourcing:** Find the relevant SigmaHQ rule for the exact TTP you are missing.

**Automated Conversion:** Convert that Sigma rule directly into an Elastic-ready format.

**Safe Staging:** Push the rule to a Staging environment first to observe its behavior against real telemetry without triggering false positives in production.

By automating the translation and deployment phases, engineers can push tested, battle-ready rules in minutes, drastically reducing the time it takes to neutralize a newly discovered threat.

### Detection as Code vs Traditional Security Rules: Which Should We Choose?
Traditional UI-based rule creation was a necessary stepping stone, but it suffers from a major blind spot: SIEMs don't show you "bad rules." A traditional SIEM will gladly let a rule run as long as the syntax compiles. It won't tell you if the rule is actually healthy until it floods your SOC with false positives. <span class="text-accent">TIDE</span> changes this by implementing a rigorous Rule Health scoring engine. Before a rule is fully trusted, <span class="text-accent">TIDE</span> checks:

**Schema Validity:** Does the field this rule is querying even exist in your logs anymore?

**Operational Context:** Is there an investigation guide attached for the analyst? Is there an Author, Is the @timestamp correct, Are highlighted fields set. 

**System Impact:** How long does this query actually take to run?

**Accountability:** <span class="text-accent">TIDE</span> tracks exactly who validated and promoted the rule, meaning there is always a subject matter expert to consult if a rule needs tuning.

While <span class="text-accent">TIDE</span> won't strictly hard-block you from promoting a sub-optimal rule in an emergency, it ensures complete visibility and accountability for every piece of logic running in your environment.

**Ready to Build Your Baseline?**
Security engineering shouldn't be managed in isolated silos or static spreadsheets. If you are ready to modernize your rule management and turn your threat models into verifiable defenses, it is time to deploy <span class="text-accent">TIDE</span>.

<span class="text-accent">TIDE</span> is a standalone, containerized platform with zero external dependencies (powered by FastAPI and an embedded DuckDB).

### Get started today by pulling the image:

**Clone the repository**  
`git clone https://github.com/sigeauk/tide.git`
`cd tide`

**Start <span class="text-accent">TIDE</span>**  
`docker compose up --build -d`

**Access the UI**   
Open `http://localhost`