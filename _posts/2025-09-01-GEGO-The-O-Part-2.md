---
layout: default
title: "GEKO: The 'O' Part 2"
date: 2025-09-01
description: "MITRE ATT&CK and TTPs"
---

# <span class="highlight">GEKO: The 'O': Part 2</span>

### MITRE Mayhem and the TTP Treasure Hunt

> “Some people play Sudoku for fun. CTI analysts play ‘connect the dots’ with nation-states. MITRE ATT&CK just gives us a bigger crayon box.”

### What Is MITRE ATT&CK, and Why Should You Care?

Think of MITRE ATT&CK as the Pokédex of threat behavior. It catalogs every known trick in the adversary playbook: from phishing emails to credential dumping to full-on "I live in your network now" persistence.

In OpenCTI, MITRE isn’t just a reference. It’s deeply integrated. Each tactic, technique, and sub-technique can be linked to threat actors, malware, intrusion sets, and more. Basically, it’s your framework for understanding how the bad guys actually do their badness.

### TTPs: Techniques, Tactics, and Pathways to Panic

**Tactics** = Why - Credential Access
**Techniques** = How - Brute force
**Sub-technique** = How - Password guessing  
In STIX/OpenCTI lingo, these are modeled as attack-pattern SDOs. When you start mapping these to real-world threat actors, it’s like painting a digital mugshot:
`"APT29"` → `"uses"` → `"Spearphishing Attachment (T1566.001)"`  
Just like that, you’re one step closer to figuring out what they’ll do next.

### Meet the Cast: APTs, Malware, and Other Shady Characters

Let’s say you’re investigating APT29 (a.k.a. “Cozy Bear”, “The Nation-State Who Ghosted Me”). You want to map out how they operate.
In OpenCTI, you can build a full profile:
```json
{
"type": "intrusion-set",
"name": "APT29",
"aliases": ["Cozy Bear", "The Dukes"],
"primary_motivation": "espionage",
…
}
```
Then connect it to:
-> **Malware:** WellMess, GoldMax, Cobalt Strike
-> **Techniques:** Credential Dumping (T1003), Spearphishing Attachment (T1566.001)
-> **Infrastructure:** Command and Control servers

### Mapping TTPs in OpenCTI (a.k.a. “Treasure Maps for Nerds”)

OpenCTI lets you build out an attack graph — a living, breathing map of who did what, how, and why. Here’s what a typical path might look like:

```
APT29
|--- uses ---> Spearphishing Attachments [T1566.001]
       |--- delivers ---> WellMess [Malware]
               |--- communicates-with ---> C2 Infrastructure
```

And yes, all of this is clickable in the OpenCTI UI. It’s like digital conspiracy theory — except it’s real.

### Sample STIX Object: MITRE Technique

```json
{
  "type": "attack-pattern",
  "name": "Spearphishing Attachment",
  "external_references": [
    {
      "source_name": "mitre-attack",
      "external_id": "T1566.001",
      "url": "https://attack.mitre.org/techniques/T1566/001/"
    }
  ],
  "description": "Adversaries may send phishing emails with malicious attachments."
}
```
Now link this object to your intrusion set (APT29) and malware (WellMess) using relationship objects. Boom — now you’ve got TTP coverage.

### Heat Mapping: The Matrix View

One of the coolest features of OpenCTI is the MITRE matrix heat map. Once your entities are linked to attack patterns, you can see:
-> Which techniques are covered
-> Which ones are still blank (👀 “detection gaps!”)
-> What each APT's TTP fingerprint looks like.
It’s like Sudoku, but with more nation-state sabotage.

### Use the MITRE Connector

OpenCTI has a built-in MITRE ATT&CK connector that imports all known TTPs and updates them over time. No more manually creating techniques. Just run the connector, grab coffee, and come back to a fully MITRE-mapped world.

```yaml
version: '3'
services:
  connector-mitre:
    image: opencti/connector-mitre:6.7.1
    environment:
      - OPENCTI_URL=http://localhost
      - OPENCTI_TOKEN=ChangeMe
      - CONNECTOR_ID=ChangeMe
      - "CONNECTOR_NAME=MITRE Datasets"
      - CONNECTOR_SCOPE=tool,report,malware,identity,campaign,intrusion-set,attack-pattern,course-of-action,x-mitre-data-source,x-mitre-data-component,x-mitre-matrix,x-mitre-tactic,x-mitre-collection
      - CONNECTOR_RUN_AND_TERMINATE=false
      - CONNECTOR_LOG_LEVEL=error
      - MITRE_REMOVE_STATEMENT_MARKING=true
      - MITRE_INTERVAL=7 # In days
    restart: always
```

### Coming Soon in Part 3:

You’ve mapped your threats. Now let’s fight back. In Part 3, we’ll turn Elastic detection rules into CTI power moves, importing them into OpenCTI as Courses of Action and mapping them to MITRE techniques. Title drop: “Elastic Rules and the COA Conundrum.” You won’t want to miss it.
