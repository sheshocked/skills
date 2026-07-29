---
name: observability-loki-promtail
description: Establish structured JSON logging, collection pipeline with Grafana Loki and Promtail.
category: devops
tags: [loki, promtail, logging, observability, json-logs]
---

# Observability Loki Promtail

## When to Use
Use to consolidate application, Nginx, and system logs into a single queries dashboard.

## Prerequisites
- Docker Compose.

## Workflow
1. Deploy Loki database container.
2. Install Promtail agent and map application directories.
3. Configure Promtail config rules parsing structured logs.

## Key Patterns
```yaml
# promtail-config.yaml
server:
  http_listen_port: 9080

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: system
    static_configs:
      - targets: [localhost]
        labels:
          job: varlogs
          __path__: /var/log/*.log
```

## Pitfalls
- **Unstructured log parsing:** Avoid raw grep scans; use structured JSON parsing formatters.
- **Log storage limits:** Ensure Loki retention guidelines prevent storage leaks.

## Verification
- Query logs from Grafana Log Explorer using LogQL queries.
- Verify ingestion rates in Loki dashboards.
