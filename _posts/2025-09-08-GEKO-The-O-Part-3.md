---
layout: default
title: "GEKO: The 'O' Part 3"
date: 2025-09-08
description: "Detection Rules and Courses of Action"
---

# <span class="highlight">GEKO: The 'O': Part 3</span>

### Elastic Rules and the COA Conundrum

>“You’ve tracked the threat actors, mapped the TTPs, and now it’s time to fight back, with JSON, STIX, and Courses of Action that would make Sun Tzu proud.”

### What’s a Course of Action?

In STIX 2.1 and OpenCTI land, a Course of Action (COA) is the “What are you gonna do about it?” of threat intelligence. It’s the countermeasure — the detection, prevention, or response technique used to defend against known behaviors.

COAs can be:
- Elastic detection rules 💥
- Sigma/YARA rules
- Response playbooks
- Policy updates
- Firewall blocks

### COAs in OpenCTI – How They Work

Courses of Action are STIX Domain Objects (SDOs). You can link them to attack patterns, malware, threat actors, and more.
Example:
```json
{
  "type": "course-of-action",
  "name": "Detect Cobalt Strike Beacon via Elastic Rule",
  "description": "Elastic rule to detect default Cobalt Strike beacon traffic patterns using JA3/SNI.",
  "action_type": "detection",
  "object_marking_refs": ["marking-definition--TLP:GREEN"]
}
```
Then link it like this:
```json
{
"type": "relationship",
"relationship_type": "mitigates",
"source_ref": "course-of-action--elastic-detect-cobalt",
"target_ref": "attack-pattern--cobalt-strike-beacon"
}
```
Voilà: You’ve tied a detection rule directly to the technique it helps identify.

### Importing Elastic Rules into OpenCTI

Elastic’s detection rules live in .toml, .yml or .json formats, usually with metadata that maps nicely to STIX. You don’t need to hand-type each one like a 2003 threat analyst with carpal tunnel.

**Option 1:** Manual Import via OpenCTI UI
Convert your Elastic rule into a STIX COA object.
Use the OpenCTI UI to create a new Course of Action.
Link it to a MITRE technique (attack-pattern) using a mitigates relationship.
*Optional:* Attach Observables, external references, or detection notes.

**Option 2:** Script or Connector
OpenCTI supports custom connectors. You can:
Parse .toml/.yml rules
Convert to STIX Course of Action
Auto-link to attack-patterns using MITRE IDs

### Closing the Loop: CTI → Detection → Feedback

Once your COAs are imported, you’ve created a bidirectional intelligence feedback loop:
- Threat Intel informs you what to look for (TTPs).
- Detection Rules become COAs that mitigate those TTPs.
- Detection Alerts feed into threat enrichment (e.g., updating observables or attribution).
- OpenCTI reflects all of it — in beautiful, traceable, contextual knowledge graphs.
It’s like playing 4D chess with threat actors, and you’re finally seeing the whole board.

### MITRE Coverage View in OpenCTI

OpenCTI’s dashboard view now shows:
-> Which techniques are observed vs mitigated
-> Heatmaps based on Elastic rule coverage
-> Detection gaps for priority threat actors
-> Ability to export custom reports for CISO pats-on-the-back
Now your detection rules don’t just live in Elastic — they’re part of a strategic intel plan.

### Final Thoughts from GEKO

GitLab gave you pipelines. Elastic gave you detection. Kibana gave you pretty dashboards. But OpenCTI? OpenCTI gives you narrative-driven threat intelligence. It turns TTPs into stories, observables into evidence, and detection into proactive defense.
