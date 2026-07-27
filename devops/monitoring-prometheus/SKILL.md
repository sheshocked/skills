---
name: monitoring-prometheus
description: 
category: devops
tags: [monitoring-prometheus]
---

## When to Use
Set up Prometheus monitoring with alerting, Grafana dashboards, and service discovery for infrastructure and application observability. Covers metrics collection, recording rules, alerting rules, and long-term storage.

## Core Concepts
- **Scrape**: Pull metrics from `/metrics` endpoints at intervals
- **PromQL**: Query language for time-series data
- **Recording rules**: Pre-compute expensive queries for dashboards
- **Alerting rules**: Evaluate conditions and fire to Alertmanager
- **Service discovery**: Auto-detect targets (Kubernetes, Consul, DNS)
- **Remote write**: Send metrics to Thanos/Mimir/Cortex for long-term storage

## Workflow
1. Deploy Prometheus with appropriate scrape configs
2. Add exporters (node, postgres, nginx, redis)
3. Write recording rules for dashboard performance
4. Configure alerting rules and Alertmanager
5. Build Grafana dashboards
6. Set up long-term storage with Thanos or remote_write

## Key Patterns
```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    cluster: production

rule_files:
  - "alerts/*.yml"
  - "recording/*.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']

scrape_configs:
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node1:9100', 'node2:9100']
    relabel_configs:
      - source_labels: [__address__]
        regex: '(.+):\d+'
        target_label: instance

  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        target_label: __address__
        regex: (.+)
        replacement: ${1}
```

```yaml
# alerts/production.yml
groups:
  - name: infrastructure
    rules:
      - alert: HighCPU
        expr: 100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 85
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "High CPU on {{ $labels.instance }}"
          description: "CPU usage above 85% for 10 minutes"

      - alert: DiskSpaceLow
        expr: (node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) * 100 < 15
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Low disk space on {{ $labels.instance }}"
          description: "Less than 15% disk space remaining"

      - alert: HighMemory
        expr: (1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100 > 90
        for: 5m
        labels:
          severity: warning

      - alert: ServiceDown
        expr: up == 0
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Service {{ $labels.job }} is down"
```

```yaml
# recording/dashboard.yml
groups:
  - name: node_recording
    interval: 30s
    rules:
      - record: instance:node_cpu_utilization:rate5m
        expr: 100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
      - record: instance:node_memory_utilization:ratio
        expr: 1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes
```

```bash
# PromQL cheat sheet
# CPU usage percentage
100 - (avg by(instance)(irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory usage percentage
(1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100

# Disk usage percentage
(1 - node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) * 100

# HTTP request rate
rate(http_requests_total[5m])

# 95th percentile latency
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
```

## Pitfalls
- **Scrape interval vs evaluation**: Alert evaluation interval should be ≥ scrape interval
- **Stale targets**: Use `honor_labels: true` for Kubernetes SD
- **High cardinality**: Avoid high-cardinality labels (user IDs, request IDs)
- **Alert fatigue**: Tune thresholds to avoid false positives
- **Dashboard performance**: Use recording rules for expensive queries
- **Storage**: 15 days default; use Thanos/Mimir for longer retention

## Verification
```bash
# Verify targets are up
curl -s http://prometheus:9090/api/v1/targets | jq '.data.activeTargets[].health'

# Test alert rules
curl -s http://prometheus:9090/api/v1/alerts | jq

# Verify recording rules
curl -s http://prometheus:9090/api/v1/rules | jq '.data.groups[].rules[] | select(.type=="recording")'

# Check scrape health
curl -s http://prometheus:9090/api/v1/targets | jq '.data.activeTargets[] | select(.health!="up")'
```