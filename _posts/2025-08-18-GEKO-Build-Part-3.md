---
layout: default
title: "GEKO: Build Part 3"
date: 2025-08-18
description: "Building GEKO - the GEK"
---

# GEKO: Build Part 3

Welcome back, GEKO enthusiasts! If you’ve made it this far, you’ve wrangled GitLab, spun up your runner, and peeked under the Docker hood. Now, it’s time to unleash the real power of the GEKO stack: integrating Elasticsearch and Kibana, and putting Detection as Code (DaC) into action.  
Where we stand;
* GitLab is up and running, ready to manage your detection rules as code.
* The GitLab Runner is registered, standing by to automate your workflows.
* Docker Compose is orchestrating your stack. No more “it works on my machine” headaches.
But we’re not just here to admire our handiwork. Let’s get Elasticsearch and Kibana online, wire up the pieces, and see how you can automate, validate, and visualize your detections; GEKO 🦎 style.

#### Step 1: Start Elasticsearch and Kibana

If you haven’t already, it’s time to bring up the rest of the stack. In your /opt/docker/ directory, run:

`docker-compose up -d elasticsearch kibana`

Give Elasticsearch a minute to initialise. Logs are your friend here (docker logs -f es). Once it’s ready, generate the Kibana service token:

`docker exec -it es ./bin/elasticsearch-service-tokens create elastic/kibana kibana`

Kibana should now be accessible at http://localhost:5601 (or your chosen host IP). Log in with the elastic user and the password you set (or retrieved from the logs).

#### Step 2: Detection as Code: Automating Your Rules

Now for the fun part: DaC. With GitLab as your source of truth, you can manage detection rules (think Sigma or Elastic rules) just like any other codebase. Here’s a typical workflow:

Store Rules in GitLab: Create a repository for your detection rules; YAML, JSON, SIGMA or whatever format your tooling supports.

Pipeline Automation: Use GitLab CI/CD to validate, and test your rules on every commit. For example, add a .gitlab-ci.yml file:

```yaml
stages:
  - Convert
  - Validate
  - Import
  - Export
  - DaC

before_script:
  - git config --global user.email "user@gitlab.local"
  - git config --global user.name "DaC"
  - git remote set-url origin https://oauth:$DAC_PAT@localhost/project/repo.git

Sigma-convert:
  stage: Convert
  script:
    - python3 -m sigma convert -f siem_rule -t lucene -p ecs_windows ./sigma/$filepath > ./rule/$filepath
    - git add ./rule
    - git commit -m "Converted $filepath from sigma ready for elastic"
    - git push origin
  variables:
    - filepath: rules/windows/...

Sigma-validate:
  stage: Validate
    - python3 validate.py $filepath
  variables:
    - filepath: rules/windows/...

Elastic-import:
  stage: Import
    - python3 import.py $filepath
  variables:
    - filepath: rules/windows/...

Elastic-export:
  stage: Export
    - python3 export.py $filepath
  variables:
    - filepath: rules/windows/...

DaC:
  stage: DaC
  script:
    - python3 -m sigma convert -f siem_rule -t lucene -p ecs_windows ./sigma/$filepath > ./rule/$filepath
    - git add ./rule
    - git commit -m "Converted $filepath from sigma ready for elastic"
    - git push origin
    - python3 validate.py $filepath
    - python3 import.py $filepath 
  variables:
    - filepath: rules/windows/... 
```

This is an example of a .gitlab-ci.yml pipeline, and will require some addition steps for importing to elastic, I've done it with some python scripts but you could write the code direct into the file. 

#### Step 3: Visualize and Hunt with Kibana

With your rules in Elasticsearch, Kibana becomes your security command center:
**Dashboards:** Build custom dashboards to visualize alerts, rule matches, and trends.
**Saved Searches:** Query your logs for specific indicators or behaviors.
**Alerting:** Set up Kibana alerts to notify you (via email, Slack, etc.) when detections trigger.
**Scaling:** For production, consider increasing resources and using external storage for Elasticsearch data.
**Security:** Change default passwords, restrict network access, and use SSL/TLS for sensitive environments.

#### Conclusion: GEKO Takes Flight

You’ve now built a Detection as Code platform that’s portable, automated, and visible from end to end. GitLab manages your rules, runners automate your pipelines, Elasticsearch indexes your detections, and Kibana brings your data to life. The days of manual rule management and opaque security workflows are over.

GEKO isn’t just a stack—it’s a new way to think about security engineering. With version control, automation, and search-driven visibility, you’re equipped to adapt, scale, and respond faster than ever.

Oh my, that's a wrap on the GEKO: Gitlab + ElasticSearch + Kibana trilogy.

The GEKO: The 'O' is for OpenCTI trilogy has now began:
Part 1 – STIX & Cyber Spies
Part 2 – MITRE Mayhem & the TTP Treasure Hunt
Part 3 – Elastic Rules & the COA Conundrum