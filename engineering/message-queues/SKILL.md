---
name: message-queues
description: - Decoupling services that need to communicate asynchronously
category: engineering
tags: [message-queues]
---

## When to Use

- Decoupling services that need to communicate asynchronously
- Workloads that can tolerate eventual consistency (email, analytics, notifications)
- Buffering bursts — producer sends faster than consumer can process
- Fan-out: one event triggers work in multiple downstream services

## Core Concepts

- **At-Most-Once**: Fire and forget. Fast, can lose messages. Good for non-critical telemetry.
- **At-Least-Once**: Retry until acknowledged. No data loss, but duplicates possible. Consumer must be idempotent.
- **Exactly-Once**: Technically impossible across network boundaries. Achieved via at-least-once + idempotent consumer.
- **Dead Letter Queue (DLQ)**: Failed messages land here for inspection. Never poison the main queue.
- **Backpressure**: When consumer falls behind, queue grows. Set max queue length, implement shedding.

## Workflow

1. **Identify async boundaries** — what can be deferred vs must be synchronous
2. **Choose broker** — SQS (managed, simple), Kafka (streaming, high throughput), RabbitMQ (routing, priorities)
3. **Design message schema** — envelope with metadata, versioned payload
4. **Implement idempotency** — dedup key in consumer, check before processing
5. **Set up DLQ + alerts** — messages in DLQ mean something is broken
6. **Monitor lag** — consumer lag = how far behind you are. Alert on threshold.

## Key Patterns

```python
# SQS consumer with idempotency and DLQ
import boto3
import json
import hashlib
from datetime import datetime

sqs = boto3.client("sqs")
QUEUE_URL = "https://sqs.us-east-1.amazonaws.com/123456789/orders"

def dedup_key(message: dict) -> str: