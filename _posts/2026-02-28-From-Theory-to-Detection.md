---
layout: post
title: "Automating STRIDE and DREAD with TIDE"
date: 2026-02-28
description: From STRIDE and DREAD to actionable detections.
categories: [Cybersecurity, Threat Modeling]
tags: [STRIDE, DREAD, Sigma, MITRE, TIDE]
---

In the world of Security, we often talk about "shifting left." We want to catch vulnerabilities before they hit production. To do that, we use two industry-standard frameworks: **STRIDE** and **DREAD**. 

**But there’s a problem:** manual threat modeling is slow, static, and often disconnected from the actual alerts firing in your SIEM. 

Today, we’re looking at how these frameworks work and how our tool, <span class="text-accent">**TIDE (Threat Informed Detection Engine)**</span>, turns these theoretical models into automated, battle-ready detections.

---

## 1. STRIDE: Identifying the "How"
Developed by Microsoft, STRIDE is a mnemonic used to identify what kind of threats a system faces. It’s the "What could go wrong?" phase of modeling.

| Threat | Security Property | Description | Examples |
| :--- | :--- | :--- | :--- |
| **S**poofing | **Authenticity** | Pretending to be something or someone else. | Stolen session cookies, IP spoofing. |
| **T**ampering | **Integrity** | Modifying data or code without authorization. | Modifying a SQL database, changing binary files. |
| **R**epudiation | **Non-repudiability** | Performing an action and claiming you didn't. | Deleting logs to hide a trail, lack of audit trails. |
| **I**nformation Disclosure | **Confidentiality** | Exposing data to unauthorized users. | Leaking PII, directory traversal. |
| **D**enial of Service | **Availability** | Making a service or system unavailable. | DDoS attacks, resource exhaustion. |
| **E**levation of Privilege | **Authorization** | Gaining more access than intended. | Horizontal/Vertical privilege escalation. |

---

## 2. DREAD: Quantifying the "So What?"
Once you have a list of STRIDE threats, you need to prioritize them. **DREAD** provides a risk-rating system (typically scored 1-10) to determine which fire to put out first.

1. **Damage Potential:** If the attack succeeds, how bad is it?
2. **Reproducibility:** How easy is it for an attacker to repeat this?
3. **Exploitability:** What is the "barrier to entry" (skill/tools) for the attacker?
4. **Affected Users:** How many people lose access or data?
5. **Discoverability:** How easy is it for a hacker to find the vulnerability?

---

## 3. The TIDE Workflow: From Threat Actor to SIEM Coverage
While STRIDE and DREAD tell you *what* to worry about, <span class="text-accent">**TIDE**</span> tells you *how to stop it*. Here is how TIDE automates the journey from an abstract threat to a live detection rule:

### Step 1: Intelligence Led Discovery
Start by picking a **Threat Actor** or **Intrusion Set** (via OpenCTI). TIDE immediately shows you their known techniques and your **Current Coverage** heatmap.

<div class="gallery-item">
    <img src="{{ site.baseurl }}/images/Heatmap.png" alt="Heatmap">
</div>

### Step 2: Identify the Gap
You notice a gap in **Valid Accounts (T1078)**—a classic **Spoofing** and **Elevation of Privilege** threat. TIDE display this as a gap based on the actor's profile.

<div class="gallery-item">
    <img src="{{ site.baseurl }}/images/Sigma.jpeg" alt="Sigma">
</div>

### Step 3: Source the Logic
Clicking T1078 takes you to a dedicated **Sigma page**. TIDE filters all available SigmaHQ rules for that specific technique. You pick the most relevant rule for your environment.

<div class="gallery-item">
    <img src="{{ site.baseurl }}/images/Suggestions.png" alt="Suggestions">
</div>

### Step 4: Convert and Stage
TIDE converts the Sigma rule into your specific **SIEM language** (Elastic, Splunk, etc.) and deploys it to your **Staging environment**.

### Step 5: Audit and Quality Control
In the Staging view, you can review the **Rule Score** and logic. TIDE highlights metadata gaps so you can refine the rule before it goes live.

<div class="gallery-item">
    <img src="{{ site.baseurl }}/images/Health.jpeg" alt="Rule Health">
</div>

### Step 6: Promote and Visualize
Once you're happy, hit **Promote** in TIDE. The rule moves to production, and your MITRE heatmap updates instantly. You can now see—in green—that your "Elevation of Privilege" risk is actively mitigated.

---

## Conclusion: Stop Modeling in a Vacuum
Threat modeling shouldn't be a one-time exercise that lives in a PDF. With **TIDE**, your STRIDE and DREAD models become living, breathing components of your SOC. 

<div class="grid btn-grid">
    <div class="cta-container">
      <a href="https://github.com/sigeauk/TIDE" class="btn-primary">Try TIDE today</a>
    </div>  
</div>