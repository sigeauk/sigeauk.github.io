---
layout: default
title: "GEKO: The 'O' Part 1"
date: 2025-08-25
description: "STIX and Threat intell"
---

# <span class="highlight">GEKO: The 'O': Part 1</span>

### STIX and Cyber Spies

>“You ever feel like your threat intel is just… vibes? Let's fix that with structure and some OpenCTI.”

### What in the Threat Hell is OpenCTI?

Imagine you’re Sherlock Holmes, but instead of chasing criminals across Victorian London, you’re tracking cybercriminals across IP ranges, phishing kits, and MITRE techniques. That’s OpenCTI, your cyber magnifying glass and evidence board on steroids.
OpenCTI (Open Cyber Threat Intelligence) is an open-source platform that helps you collect, organize, correlate, and visualize threat intelligence, all using STIX 2.1, aka the world's most structured way to say "someone is attacking you."

### Wait, STIX? Like the band?

Sadly, no. But just like the band Styx warned us about robots (🎶 Domo arigato, Mr. Roboto 🎶), STIX warns us about real ones. It stands for Structured Threat Information eXpression, and it’s the Lego brick of CTI.
Everything in STIX is an object — and those objects fall into a few main categories:

| Acronym | Full Name| What it does| Examples| 
|---|---|---|---|
| SDO |STIX Domain Object| Groups SCO types| Threat Actors, Malware|
|SRO| STIX Relationship Opject| Connects objects| APT28-> uses -> Cobolt Strike|
| SCO| STIX Cyber-observable Object| Raw data| IPs, URLs, Hashes| 
| --- | Bundle | Groups SDOs, SROs and SCOs together| APT28-> uses -> Cobolt Strike -> communicates with -> [ipv4-add:value = '1.2.3.4'] |


### Bundles: STIX Lunchables

In OpenCTI, all your STIX data is packaged into Bundles, which are like lunchboxes containing multiple objects and their relationships. Want to describe how APT29 uses Cobalt Strike to target finance orgs? Bundle it, baby.

```json
{
"type": "bundle",
"id": "bundle--1234abcd",
"objects": [
{
"type": "threat-actor",
"name": "APT29",
…
},
{
"type": "malware",
"name": "Cobalt Strike",
…
},
{
"type": "relationship",
"relationship_type": "uses",
"source_ref": "threat-actor--apt29",
"target_ref": "malware--cobalt-strike"
}
]
}
```

### Create Your First Threat Actor

Let’s say you want to create a threat actor named Lord Phishington. Here’s how it might look in OpenCTI (via STIX):
```json
{
"type": "threat-actor",
"name": "Lord Phishington",
"description": "A phishing-themed APT group known for HTML lure docs and bad puns.",
"threat_actor_types": ["crime-syndicate"],
"aliases": ["PhishKing", "HookLineSinker"],
"primary_motivation": "financial-gain"
}
```

You'd pair that with relationships like:

```json
"type": "relationship",
"relationship_type": "uses",
"source_ref": "threat-actor--lord-phishington",
"target_ref": "malware--evil-macro"
```

And boom — your CTI database now knows Phishington uses Evil Macro. You’re building an empire of threat knowledge.

### OpenCTI Magic: Relationships, Relationships, Relationships

Remember that one conspiracy theorist in the movies with strings and pins all over their corkboard? OpenCTI is that — but actually helpful. Every SDO and SCO gets connected via relationships. And the more connections you build, the more useful your threat intel becomes.
Examples:
 - APT29 → uses → Cobalt Strike
 - Cobalt Strike → targets → Windows 10
 - Campaign X → attributed-to → APT29
 - File hash → indicates → Cobalt Strike
From just a few objects, you start to see entire attack narratives.

### Bonus: The Visual Graph (a.k.a. Nerd Nirvana)

Once you’ve added a few bundles into OpenCTI, click on the "Knowledge" tab and open up a threat actor. The visual graph shows how everything connects: malware, campaigns, observables, COAs, and more.
It's like playing cyber Cluedo: APT29, with the phishing email, in the executive inbox.

### Next Time on GEKO:

In Part 2, we take the red pill and deep dive into the MITRE Matrix. Want to know why APT29 is basically the Darth Vader of CTI? Stay tuned: “MITRE Mayhem and the TTP Treasure Hunt” drops soon.