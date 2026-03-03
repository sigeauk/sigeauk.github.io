---
layout: post
title:  "USER CASE: Seamless Onboarding—From CISO Intent to Engineered Defense"
date:   2026-03-09
author: Sigea Team
categories: [Use Cases, Enterprise Security, Workflow]
tags: [Onboarding, CISO, Detection Engineering, Sigma, Remote Access]
---

Security is often siloed. A CISO identifies a risk, a manager assigns a task, and an engineer writes a rule—but often, no one can actually prove the original risk was neutralized. 

In this use case, we look at the **Enterprise Onboarding Process**: a structured journey that takes a high-level security concern and turns it into a verified, high-quality detection in your SIEM. 

Here is how TIDE facilitates that journey.

---

### Step 1: The Strategic Kickoff (The CISO)
**The Objective:** Define what matters.

During an initial onboarding session, the CISO isn't looking at code; they are looking at the business. For this example, let's say the CISO identifies **Remote Access** as the #1 priority due to a shift in the company's hybrid-work policy.

* **Action:** In the TIDE dashboard, a project is created: "Remote Access Hardening."
* **The Goal:** Ensure any unauthorized entry or lateral movement via remote protocols is detected instantly.

> **CISO View:** "I’ve defined the risk. Now I need to see the roadmap to mitigation."

---

### Step 2: The Tactical Blueprint (The Manager)
**The Objective:** Translate risk into TTPs.

The Security Manager takes the "Remote Access" initiative and uses TIDE to map it to the **STRIDE** framework. 
* **Threat:** *Spoofing* and *Elevation of Privilege* via remote services.
* **The TTP List:** TIDE suggests a baseline of MITRE ATT&CK techniques, such as:
    * **T1133** (External Remote Services)
    * **T1021.001** (Remote Desktop Protocol)
    * **T1021.004** (SSH)

**The Gap Discovery:** TIDE compares these techniques against the current rule set. The Manager sees that while RDP is covered, **SSH (T1021.004)** is a "Gray Zone." This becomes the Engineering team's primary objective.

---

### Step 3: The Technical Execution (The Engineer)
**The Objective:** Find, convert, and deploy the logic.

The Engineer logs into TIDE and sees the specific TTP gaps. Instead of starting from scratch, they use TIDE’s integration with the **SigmaHQ** library.

1.  **Sourcing:** The Engineer selects **T1021.004**. TIDE displays all relevant Sigma rules.
2.  **Selection:** They pick a rule for "Anomalous SSH Login."
3.  **Quality Assurance:** This is where TIDE adds massive value. The Engineer checks the **Rule Quality Score**. 
    * Does the rule have correct metadata? 
    * Is the logic compatible with our specific Linux logs? 
    * Are there missing fields?
4.  **Conversion:** Satisfied with the quality, the Engineer uses TIDE to convert the Sigma YAML into the native language of their SIEM (e.g., Splunk SPL or Sentinel KQL).

---

### Step 4: Closing the Loop (Verification)
**The Objective:** Confirming the defense is live.

Once the Engineer hits **Promote**, the rule is moved to the production environment. 

* **Automatic Update:** TIDE’s MITRE heatmap updates instantly. The "Gray Zone" for SSH becomes **Green**.
* **Executive Proof:** The CISO logs back in. They don't need to read the KQL or the Sigma logic. They see the "Remote Access Hardening" project move to **100% Covered**.

> **The CISO Result:** "I asked for remote access protection. TIDE shows me exactly how, where, and when we achieved it."

---

### The Onboarding "Must-Haves"
If you’re using TIDE for the first time, we recommend onboarding these three "System Actor" profiles immediately to show instant value across all levels:

| Profile | Key TTP to Map | Target Log Source |
| :--- | :--- | :--- |
| **Identity Defense** | T1078 (Valid Accounts) | AD / Okta / Azure AD |
| **Remote Perimeter** | T1133 (External Remote Services) | VPN / Firewall Logs |
| **Endpoint Integrity** | T1070.001 (Clear Event Logs) | Sysmon / EDR |

---

### Summary: A Unified Language for Security
By following this onboarding process, TIDE ensures that security isn't just a series of disconnected technical tasks. It becomes a verifiable business process where the Engineer’s hard work is directly visible as CISO-level risk reduction.

**Is your team speaking the same language? Stop guessing and start onboarding with TIDE.**

[Try TIDE Today] [View Documentation]