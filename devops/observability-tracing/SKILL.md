---
name: observability-tracing
description: 
category: devops
tags: [observability-tracing]
---

## When to Use
Implement distributed tracing with OpenTelemetry, Jaeger, or Zipkin to track requests across microservices. Covers trace propagation, context injection, sampling strategies, and correlation with logs/metrics.

## Core Concepts
- **Trace**: End-to-end request journey across services
- **Span**: Single unit of work within a trace (has start time, duration, status)
- **Context propagation**: Pass trace context (traceparent header) between services
- **Sampling**: Decide which traces to collect (head-based, tail-based)
- **Baggage**: Key-value pairs that propagate across service boundaries
- **Trace-ID correlation**: Link logs and metrics to traces

## Workflow
1. Instrument services with OpenTelemetry SDK
2. Configure OTLP exporter to collector
3. Deploy collector as central aggregation point
4. Forward traces to Jaeger/Tempo for storage
5. Configure sampling strategy
6. Correlate traces with logs via trace_id

## Key Patterns
```python
# OpenTelemetry Python instrumentation
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.resources import Resource
from opentelemetry.instrumentation.fastapi import FastAPIInstrumentor
from opentelemetry.instrumentation.requests import RequestsInstrumentor

# Setup
resource = Resource.create({
    "service.name": "api-server",
    "service.version": "1.2.3",
    "deployment.environment": "production",
})

provider = TracerProvider(resource=resource)
provider.add_span_processor(
    BatchSpanProcessor(OTLPSpanExporter(endpoint="otel-collector:4317"))
)
trace.set_tracer_provider(provider)

# Auto-instrument FastAPI
app = FastAPI()
FastAPIInstrumentor.instrument_app(app)
RequestsInstrumentor().instrument()

# Manual spans
tracer = trace.get_tracer(__name__)

@app.get("/api/users/{user_id}")
async def get_user(user_id: str):
    with tracer.start_as_current_span("fetch_user") as span:
        span.set_attribute("user.id", user_id)

        with tracer.start_as_current_span("db_query"):
            user = await db.fetch_user(user_id)

        with tracer.start_as_current_span("enrich_profile"):
            profile = await enrich(user)

        return profile
```

```yaml
# OpenTelemetry Collector config
# otel-collector-config.yml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318

processors:
  batch:
    timeout: 5s
    send_batch_size: 1024
  memory_limiter:
    check_interval: 1s
    limit_mib: 512
    spike_limit_mib: 128
  tail_sampling:
    decision_wait: 10s
    policies:
      - name: errors
        type: status_code
        status_code: {status_codes: [ERROR]}
      - name: slow-traces
        type: latency
        latency: {threshold_ms: 1000}
      - name: probabilistic
        type: probabilistic
        probabilistic: {sampling_percentage: 10}

exporters:
  otlp/jaeger:
    endpoint: jaeger-collector:4317
    tls:
      insecure: false
  prometheus:
    endpoint: 0.0.0.0:8889

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [memory_limiter, tail_sampling, batch]
      exporters: [otlp/jaeger]
    metrics:
      receivers: [otlp]
      processors: [batch]
      exporters: [prometheus]
```

```yaml
# Context propagation in Go (middleware)
# Middleware to inject trace context into outgoing requests
func TracingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        ctx := r.Context()
        span := trace.SpanFromContext(ctx)

        // Add trace ID to response headers for debugging
        w.Header().Set("X-Trace-ID", span.SpanContext().TraceID().String())

        next.ServeHTTP(w, r)
    })
}

// Inject context into outgoing HTTP requests
func DoWithTracing(ctx context.Context, req *http.Request) (*http.Response, error) {
    req = req.WithContext(ctx)  // propagates trace context
    return http.DefaultClient.Do(req)
}
```

```bash
# Trace analysis in Jaeger UI
# 1. Search by service name
# 2. Filter by duration (>1s = slow)
# 3. Filter by errors
# 4. Compare traces to find bottlenecks
# 5. View span timeline to identify slow services

# CLI trace query
curl -s "http://jaeger:16682/api/traces?service=api-server&limit=5" | jq '.data[].spans[].operationName'
```

## Pitfalls
- **Context propagation**: Every service must propagate trace headers
- **Sampling rate**: Too high = expensive storage; too low = miss issues
- **Span naming**: Use low-cardinality names (`/users/{id}` not `/users/123`)
- **Attribute bloat**: Don't put large payloads as span attributes
- **Clock skew**: Use NTP across all services; Jaeger compensates for skew
- **Auto-instrumentation gaps**: Manual spans needed for async/background work

## Verification
```bash
# Verify collector is receiving traces
curl -s http://otel-collector:8888/metrics | grep traces_received

# Verify Jaeger has traces
curl -s "http://jaeger:16682/api/services" | jq '.data'

# Check trace propagation in service
curl -H "traceparent: 00-{trace_id}-{span_id}-01" http://api:8080/health
```