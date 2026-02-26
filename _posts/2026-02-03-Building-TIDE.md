---
layout: default
title: "Building TIDE: Threat Informed Detection Engineering"
date: 2026-02-03
description: "A technical deep-dive into the architecture of TIDE—bridging the gap between raw threat intelligence and validated detection logic."
---

# <span class="highlight">Building TIDE: Threat Informed Detection Engineering</span>

As Detection Engineers, we often find ourselves straddling two disconnected worlds. In one browser tab, we monitor our SIEM (Elastic, Splunk, etc.), managing a vast library of detection rules. In another, we scroll through threat intelligence feeds (OpenCTI, MITRE ATT&CK®), trying to keep up with the latest adversary behaviors.

But a critical question often remains unanswered: **"If APT29 attacked us today, would our currently enabled rules actually catch them?"**

I built <span class="text-accent">TIDE</span> (Threat Informed Detection Engine) to answer that question programmatically. TIDE is a platform that audits the health of your detection rules, ingests live threat intelligence, and maps the two together to visualize actual defensive coverage gaps.

## <span class="text-accent">1. The Problem: "Broken" Rules and Blind Spots</span>

We faced two primary challenges that TIDE was designed to solve:

1.  **Rule Rot:** Detection rules can silently fail. For example, a rule might query `process.command_line`, but if an ingestion update changes that field to `process.cmdline` in your actual index, the rule stops finding threats without ever throwing an error.
2.  **Theoretical vs. Actual Coverage:** We might think we have coverage for MITRE T1059 (Command-Line Interface), but do our rules cover the specific procedures used by the threat actors actually targeting our sector?

## <span class="text-accent">2. Rule Health: Auditing at Scale</span>

TIDE doesn't just list your rules; it grades them. The engine calculates a **Health Score (0-100)** for each rule based on two primary pillars:
* **Quality Score (50 pts):** TIDE parses the rule query, supporting KQL, EQL, and ESQL and uses Regex to extract every referenced field name. It then queries the specific Elasticsearch index mappings to verify those fields actually exist and checks their data types.
* **Meta Score (50 pts):** It checks for essential context, including investigation guides, author data, and correct MITRE ATT&CK® mapping completeness.

## <span class="text-accent">3. Threat Intelligence: Knowing the Enemy</span>

To understand our defensive coverage, we first need to define what we are defending against. TIDE integrates directly with **MITRE ATT&CK®** and **OpenCTI**.

Using **STIX 2.1** parsing logic, TIDE ingests "Intrusion Sets" (Threat Actors) and extracts their known "Attack Patterns" (TTPs). This data is stored locally in an analytical database, ensuring the dashboard remains responsive without needing constant external API calls.

## <span class="text-accent">4. The Killer Feature: Dynamic Coverage Mapping</span>

This is where TIDE truly shines. It creates a real-time intersection between your **Defensive Posture** and the **Offensive Landscape**. In the Coverage Matrix, TIDE visualizes this intersection using a heatmap:

* <span class="coverage-green">Green (Covered):</span> Techniques used by the actor for which you have active, healthy rules.
* <span class="coverage-red">Red (Critical Gap):</span> Techniques the actor uses for which you have zero current visibility.
* <span class="coverage-blue">Blue (Defense in Depth):</span> Active rules you have that are not currently utilized by the selected actor.

## <span class="text-accent">5. Under the Hood: The Tech Stack</span>

TIDE has moved away from its Streamlit origins toward a more robust, production-ready architecture:

* **Backend:** **FastAPI** drives the logic, providing high-performance asynchronous API endpoints.
* **Database:** **DuckDB** serves as our analytical engine. It is an in-process SQL OLAP database that is incredibly fast at joining thousands of rules against TTPs instantly.
* **Frontend:** **HTMX** and **Tailwind CSS 4** provide a reactive, modern UI experience without the overhead of heavy JavaScript frameworks.
* **Orchestration:** A dedicated **worker.py** runs on a schedule in a separate Docker container, syncing data from Elastic, OpenCTI, and MITRE in the background.

## <span class="text-accent">6. The Future</span>

TIDE moves us away from "collecting rules" to **"engineering coverage."** The roadmap involves making it an active engineering tool, including:

1.  **Automated Suggestions:** Identifying "Red" gaps and suggesting open-source Sigma rules to fill them.
2.  **Cross-Platform Conversion:** Leveraging the Sigma backend to automatically convert rules between Elastic, Splunk, and Microsoft Sentinel.