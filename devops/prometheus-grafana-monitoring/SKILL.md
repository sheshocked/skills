---
name: prometheus-grafana-monitoring
description: Establish metrics collection, node exporter, alerts configuration, and Grafana monitoring panels.
category: devops
tags: [prometheus, grafana, monitoring, node-exporter, alerts]
---

# Prometheus Grafana Monitoring

## When to Use
Use to monitor CPU usage, memory consumption, bandwidth, and connection metrics in real-time.

## Prerequisites
- Docker Compose.

## Workflow
1. Deploy Node Exporter to endpoints.
2. Build Prometheus configs listing target endpoints.
3. Connect Grafana dashboards to analyze output metrics.

## Key Patterns
```yaml
# prometheus.yml config snippet
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "node"
    static_configs:
      - targets: ["node-exporter:9100"]
```

## Pitfalls
- **Disk space depletion:** Prometheus metrics deplete storage fast. Set retention limits: `--storage.tsdb.retention.time=15d`.
- **Ineffective alerting limits:** Avoid noisy alerts; configure rate-change thresholds.

## Verification
- Visit `localhost:9090/targets` to confirm endpoints are active.
- Verify dashboard charts display live server metrics.
