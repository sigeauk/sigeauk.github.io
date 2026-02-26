---
layout: default
title: "Introducing TIDE: Threat Informed Detection Engineering"
date: 2026-02-24
description: How and why TIDE was developed.
---

# <span class="highlight">Introducing TIDE: Threat Informed Detection Engineering</span>

In modern security operations, the gap between raw threat intelligence and actionable defense is a constant challenge. <span class="text-accent">**TIDE**</span> (Threat Informed Detection Engine) is our open-source solution designed to bridge that gap by treating detection as an engineering discipline.

## <span class="text-accent">Why TIDE?</span>

TIDE provides a "Human-in-the-Loop" interface for managing detection rules and analyzing threat coverage. Our philosophy is built on three pillars:

* **Transparency:** Open-source tools that are auditable and trustworthy.
* **Alignment:** Mapping defensive measures to adversary behaviors via MITRE ATT&CK®.
* **Automation:** Programmatic lifecycles that reduce manual SOC burdens.

## <span class="text-accent">The Technical Stack</span>

TIDE is built for speed and portability, utilizing:
* **FastAPI:** High-performance backend logic.
* **DuckDB:** Embedded analytics for zero-dependency data handling.
* **HTMX:** A responsive frontend without heavy JS overhead.

## <span class="text-accent">Quick Start</span>
You can deploy the full TIDE stack using Docker:

```yaml
name: tide
services:
  tide-app:
    image: 047741/tide-core:${TIDE_VERSION}
    container_name: tide-app
    env_file:
      - .env
    environment:
      - PYTHONPATH=/app
      - DB_PATH=/app/data/tide.duckdb
      - TRIGGER_DIR=/app/data/triggers
      - VALIDATION_FILE=/app/data/checkedRule.json
      - REQUESTS_CA_BUNDLE=/etc/ssl/certs/ca-certificates.crt
      - SSL_CERT_FILE=/etc/ssl/certs/ca-certificates.crt
    volumes:
      - ./data:/app/data
      - ${RULE_LOG_PATH:-./data/log/rules}:/mnt/rule-logs
      - ./certs/ca.crt:/usr/local/share/ca-certificates/tide-ca.crt:ro
    expose:
      - "8000"
    networks:
      - tide-network
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 10s
    restart: unless-stopped

  tide-nginx:
    image: nginx:alpine
    container_name: tide-nginx
    ports:
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf:ro
      - ./certs/server.crt:/etc/nginx/certs/server.crt:ro
      - ./certs/server.key:/etc/nginx/certs/server.key:ro
    networks:
      - tide-network
    depends_on:
      tide-app:
        condition: service_healthy
    restart: unless-stopped

networks:
  tide-network:
    driver: bridge
```

```env
TIDE_VERSION=2.4.0
APP_URL=https://tide
SYNC_INTERVAL_MINUTES=60

AUTH_DISABLED=false
SSL_VERIFY=true

KEYCLOAK_URL=http://keycloak:8080
KEYCLOAK_INTERNAL_URL=http://keycloak:8080
KEYCLOAK_REALM=tide
KEYCLOAK_CLIENT_ID=tide-app
KEYCLOAK_CLIENT_SECRET=

ELASTICSEARCH_URL=http://elasticsearch:9200
ELASTIC_URL=http://kibana:5601
ELASTIC_API_KEY=
KIBANA_SPACES=production, staging

OPENCTI_URL=http://opencti:8080
OPENCTI_TOKEN=

GITLAB_URL=http://gitlab:8929
GITLAB_TOKEN=

MITRE_SOURCE=/opt/repos/mitre/enterprise-attack.json
MITRE_MOBILE_SOURCE=/opt/repos/mitre/mobile-attack.json
MITRE_ICS_SOURCE=/opt/repos/mitre/ics-attack.json
MITRE_PRE_SOURCE=/opt/repos/mitre/pre-attack.json
SIGMA_SOURCE=/opt/repos/sigma/rules

RULE_LOG_PATH=./data/log/rules
```

```bash
docker compose up -d
```