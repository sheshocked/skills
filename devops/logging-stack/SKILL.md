---
name: logging-stack
description: 
category: devops
tags: [logging-stack]
---

## When to Use
Set up centralized logging with the ELK/EFK stack or Loki for collecting, parsing, storing, and querying logs from multiple services and hosts. Covers structured logging, log shipping, retention, and alerting on log patterns.

## Core Concepts
- **Structured logging**: JSON format with consistent fields (timestamp, level, service, trace_id)
- **Log shipping**: Filebeat/Fluentd/Fluent Bit as lightweight collectors
- **Centralized storage**: Elasticsearch or Loki for indexing and search
- **Retention policies**: ILM (Index Lifecycle Management) for Elasticsearch, retention in Loki
- **Log-based alerts**: Alert on error rate spikes or specific patterns
- **Correlation**: Link logs to traces via trace_id

## Workflow
1. Configure applications to emit structured JSON logs
2. Deploy log shipper (Filebeat/Fluent Bit) on each host
3. Set up Elasticsearch/Loki as storage backend
4. Configure Kibana/Grafana for visualization
5. Set up ILM/retention policies
6. Create log-based alerts

## Key Patterns
```yaml
# filebeat.yml — Ship logs to Elasticsearch
filebeat.inputs:
- type: container
  paths:
    - '/var/lib/docker/containers/*/*.log'
  processors:
    - add_docker_metadata:
        host: "unix:///var/run/docker.sock"
    - decode_json_fields:
        fields: ["message"]
        target: ""
        overwrite_keys: true

- type: log
  paths:
    - '/var/log/nginx/access.log'
  fields:
    service: nginx
    log_type: access
  fields_under_root: true

output.elasticsearch:
  hosts: ["elasticsearch:9200"]
  indices:
    - index: "filebeat-%{[service]}-%{+yyyy.MM.dd}"
      when.has_fields: ["service"]

setup.ilm.enabled: true
setup.ilm.rollover_alias: "filebeat"
setup.ilm.pattern: "{now/d}-000001"
```

```yaml
# fluent-bit.conf — Lightweight alternative
[SERVICE]
    Flush        5
    Log_Level    info
    Parsers_File parsers.conf

[INPUT]
    Name         tail
    Path         /var/log/containers/*.log
    Parser       docker
    Tag          containers.*

[FILTER]
    Name         docker
    Match        containers.*
    Merge_Log    On

[OUTPUT]
    Name         es
    Match        *
    Host         elasticsearch
    Port         9200
    Index        k8s-logs
    Type         _doc
```

```yaml
# Loki docker-compose stack
version: "3"
services:
  loki:
    image: grafana/loki:2.9.0
    ports: ["3100:3100"]
    volumes:
      - ./loki-config.yml:/etc/loki/config.yml
    command: -config.file=/etc/loki/config.yml

  promtail:
    image: grafana/promtail:2.9.0
    volumes:
      - /var/log:/var/log:ro
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - ./promtail-config.yml:/etc/promtail/config.yml
    command: -config.file=/etc/promtail/config.yml

# promtail-config.yml
server:
  http_listen_port: 9080

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: docker
    static_configs:
      - targets: [localhost]
        labels:
          job: docker
          __path__: /var/lib/docker/containers/*/*-json.log
    pipeline_stages:
      - docker: {}
      - json:
          expressions:
            level: level
            msg: message
      - labels:
          level:
```

```python
# Application structured logging (Python)
import json
import logging
import uuid

class JSONFormatter(logging.Formatter):
    def format(self, record):
        log_entry = {
            "timestamp": self.formatTime(record),
            "level": record.levelname,
            "service": "api-server",
            "message": record.getMessage(),
            "module": record.module,
            "trace_id": getattr(record, "trace_id", str(uuid.uuid4())),
        }
        if record.exc_info:
            log_entry["exception"] = self.formatException(record.exc_info)
        return json.dumps(log_entry)
```

## Pitfalls
- **Log volume**: Monitor ingestion rate — uncontrolled logging costs storage
- **Field mapping**: Define field types explicitly in Elasticsearch mappings
- **Retention**: Set ILM policies early — default unlimited retention fills disks
- **Sensitive data**: Filter PII before shipping (use pipeline processors)
- **JSON parsing failures**: Handle malformed JSON gracefully with `on_error` configs
- **Time synchronization**: All hosts must use NTP for consistent timestamps

## Verification
```bash
# Check Elasticsearch indices
curl -s "http://localhost:9200/_cat/indices?v&s=index"

# Check Loki targets
curl -s http://localhost:3100/loki/api/v1/targets

# Query Loki logs
curl -G "http://localhost:3100/loki/api/v1/query_range" \
  --data-urlencode 'query={job="docker"} |= "error"'

# Verify Filebeat is shipping
curl -s http://localhost:9090/stats | jq
```