---
layout: default
title: "Introducing TIDE: Threat Informed Detection Engineering"
date: 2026-02-24
description: How and why TIDE was developed.
---

# Introducing TIDE: Threat Informed Detection Engineering

In modern security operations, the gap between raw threat intelligence and actionable defense is a constant challenge. **TIDE** (Threat Informed Detection Engine) is our open-source solution designed to bridge that gap by treating detection as an engineering discipline.

### Why TIDE?
TIDE provides a "Human-in-the-Loop" interface for managing detection rules and analyzing threat coverage. Our philosophy is built on three pillars:

* **Transparency:** Open-source tools that are auditable and trustworthy.
* **Alignment:** Mapping defensive measures to adversary behaviors via MITRE ATT&CK®.
* **Automation:** Programmatic lifecycles that reduce manual SOC burdens.

### The Technical Stack
TIDE is built for speed and portability, utilizing:
* **FastAPI:** High-performance backend logic.
* **DuckDB:** Embedded analytics for zero-dependency data handling.
* **HTMX:** A responsive frontend without heavy JS overhead.

### Quick Start
You can deploy the full TIDE stack using Docker:

```bash
docker compose up -d
```