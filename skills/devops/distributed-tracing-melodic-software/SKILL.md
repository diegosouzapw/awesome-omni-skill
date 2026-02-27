---
name: distributed-tracing
description: Use when implementing distributed tracing, understanding trace propagation, or debugging cross-service issues. Covers OpenTelemetry, span context, and trace correlation.
allowed-tools: Read, Glob, Grep
---

# Distributed Tracing

Patterns and practices for implementing distributed tracing across microservices and understanding request flows in distributed systems.

## When to Use This Skill

- Implementing distributed tracing in microservices
- Debugging cross-service request issues
- Understanding trace propagation
- Choosing tracing infrastructure
- Correlating logs, metrics, and traces

## Why Distributed Tracing?

```text
Problem: Request flows through multiple services
How do you debug when something fails?

Without tracing:
User → API → ??? → ??? → Error somewhere

With tracing:
User → API (50ms) → OrderService (20ms) → PaymentService (ERROR: timeout)
         └── Full visibility into request flow
```

## Core Concepts

### Traces, Spans, and Context

```text
Trace: End-to-end request journey
├── Span: Single operation within a service
│   ├── SpanID: Unique identifier
│   ├── ParentSpanID: Link to parent span
│   ├── TraceID: Shared across all spans
│   ├── Operation Name: What is being done
│   ├── Start/End Time: Duration
│   ├── Status: Success/Error
│   ├── Attributes: Key-value metadata
│   └── Events: Point-in-time annotations
│
└── Context: Propagated across service boundaries
    ├── TraceID
    ├── SpanID
    ├── Trace Flags
    └── Trace State
```

### Trace Visualization

```text
TraceID: abc123

Service A (API Gateway)
├──────────────────────────────────────────────────────┤ 200ms
    │
    └─► Service B (Order Service)
        ├───────────────────────────────────┤ 150ms
            │
            ├─► Service C (Inventory)
            │   ├───────────────┤ 50ms
            │
            └─► Service D (Payment)
                ├───────────────────────┤ 80ms
                    │
                    └─► External API
                        ├─────────┤ 60ms
```

## OpenTelemetry

### Overview

```text
OpenTelemetry = Unified observability framework

Components:
┌─────────────────────────────────────────────────────┐
│  Application                                        │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │
│  │    SDK      │  │   Tracer    │  │   Meter     │ │
│  │             │  │   Provider  │  │   Provider  │ │
│  └─────────────┘  └─────────────┘  └─────────────┘ │
└─────────────────────────────────────────────────────┘
           │               │               │
           └───────────────┼───────────────┘
                           ▼
              ┌─────────────────────────┐
              │    OTLP Exporter        │
              └─────────────────────────┘
                           │
                           ▼
              ┌─────────────────────────┐
              │    Collector            │
              │  (Optional)             │
              └─────────────────────────┘
                           │
           ┌───────────────┼───────────────┐
           ▼               ▼               ▼
      ┌─────────┐    ┌─────────┐    ┌─────────┐
      │ Jaeger  │    │  Zipkin │    │  Tempo  │
      └─────────┘    └─────────┘    └─────────┘
```

### Trace Context Propagation

```text
HTTP Headers (W3C Trace Context):
traceparent: 00-{trace-id}-{span-id}-{flags}
tracestate: vendor1=value1,vendor2=value2

Example:
traceparent: 00-0af7651916cd43dd8448eb211c80319c-b7ad6b7169203331-01
              │   │                               │                └─ sampled
              │   │                               └─ parent span id
              │   └─ trace id (128-bit)
              └─ version

Propagation across services:
┌─────────────┐                      ┌─────────────┐
│  Service A  │  ─── HTTP ──────────►│  Service B  │
│             │  traceparent: 00-... │             │
│ Create Span │                      │ Extract     │
│ Inject      │                      │ Create Span │
└─────────────┘                      └─────────────┘
```

### Span Attributes

```text
Semantic conventions (standard attributes):

HTTP:
- http.method: GET, POST, etc.
- http.url: Full URL
- http.status_code: 200, 404, 500
- http.route: /users/{id}

Database:
- db.system: postgresql, mysql
- db.statement: SELECT * FROM...
- db.operation: query, insert

RPC:
- rpc.system: grpc
- rpc.service: OrderService
- rpc.method: CreateOrder

Custom:
- user.id: 12345
- order.total: 99.99
- feature.flag: experiment_v2
```

## Tracing Backends

### Jaeger

```text
Features:
- Open source (CNCF)
- Built-in UI
- Multiple storage backends
- OpenTelemetry native

Architecture:
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│   Agent     │─►│  Collector  │─►│   Storage   │
│ (optional)  │  │             │  │ (Cassandra/ │
└─────────────┘  └─────────────┘  │ Elasticsearch)
                       │          └─────────────┘
                       ▼
                ┌─────────────┐
                │    Query    │
                │   Service   │
                └─────────────┘
                       │
                       ▼
                ┌─────────────┐
                │     UI      │
                └─────────────┘
```

### Zipkin

```text
Features:
- Mature, battle-tested
- Simple architecture
- Low resource overhead
- Good ecosystem support

Best for:
- Simpler setups
- Lower resource environments
- Teams familiar with Zipkin
```

### Grafana Tempo

```text
Features:
- Object storage backend (cheap)
- Deep Grafana integration
- Log-based trace discovery
- Exemplars support

Best for:
- Grafana-heavy environments
- Cost-sensitive deployments
- Large-scale traces
```

### Cloud Native Options

| Provider | Service | Integration |
| -------- | ------- | ----------- |
| AWS | X-Ray | Native AWS services |
| GCP | Cloud Trace | Native GCP services |
| Azure | Application Insights | Native Azure services |
| Datadog | APM | Full-stack observability |

## Sampling Strategies

### Why Sample?

```text
High-traffic systems generate millions of spans.
Storing all spans is expensive and often unnecessary.

Sampling: Collect a subset of traces

Goal: Keep enough data to debug issues
      while managing costs
```

### Sampling Types

```text
1. Head-based sampling (at trace start):
   - Decision made when trace begins
   - Consistent across services
   - Simple but may miss rare events

2. Tail-based sampling (after trace complete):
   - Decision made after seeing full trace
   - Can keep interesting traces (errors, slow)
   - Requires buffering spans
   - More complex infrastructure

3. Priority sampling:
   - Assign priority based on attributes
   - Keep all errors, sample normal traffic
```

### Sampling Strategies

```text
Rate-based:
- Sample 10% of all traces
- Simple, predictable cost

Priority-based:
- 100% of errors
- 100% of slow requests (>1s)
- 5% of normal requests

Adaptive:
- Adjust rate based on traffic
- Target specific traces/second
- Handle traffic spikes
```

## Correlation Patterns

### Logs-Traces-Metrics

```text
Three Pillars of Observability:

Logs ◄──────────► Traces ◄──────────► Metrics
  │                  │                   │
  │ trace_id         │ exemplars         │
  │ span_id          │                   │
  └──────────────────┴───────────────────┘

Correlation:
1. Add trace_id/span_id to log entries
2. Add exemplars (trace links) to metrics
3. Click from metric → trace → logs
```

### Log Correlation

```text
Structured log with trace context:

{
  "timestamp": "2024-01-15T10:30:00Z",
  "level": "ERROR",
  "message": "Payment failed",
  "trace_id": "abc123def456",
  "span_id": "789xyz",
  "service": "payment-service",
  "user_id": "12345",
  "error": "Card declined"
}

Query in log aggregator:
trace_id:"abc123def456"
→ See all logs for this request
```

### Exemplars (Metrics to Traces)

```text
Metric with exemplar:
http_request_duration{service="api"} = 2.5s
  └── exemplar: trace_id=abc123

When latency spikes:
1. See metric spike in dashboard
2. Click on data point
3. Jump directly to slow trace
4. See exactly what caused latency
```

## Instrumentation Patterns

### Automatic Instrumentation

```text
Zero-code instrumentation:
- HTTP clients/servers
- Database clients
- Message queues
- gRPC

Pros: Easy, comprehensive
Cons: Less control, more noise
```

### Manual Instrumentation

```text
Add spans for business logic:

with tracer.start_span("process_order") as span:
    span.set_attribute("order.id", order_id)
    span.set_attribute("order.items", len(items))

    result = process(order)

    if result.error:
        span.set_status(Status(StatusCode.ERROR))
        span.record_exception(result.error)

Pros: Precise, business-relevant
Cons: More code, maintenance
```

### Hybrid Approach (Recommended)

```text
1. Auto-instrument infrastructure:
   - HTTP, database, queue calls

2. Manual instrument business logic:
   - Key operations
   - Business metrics
   - Error context
```

## Best Practices

### Span Design

```text
Good span names:
- HTTP GET /api/orders/{id}
- ProcessPayment
- db.query users

Bad span names:
- Handler (too generic)
- /api/orders/12345 (cardinality explosion)
- doStuff (meaningless)
```

### Attribute Guidelines

```text
Do:
- Use semantic conventions
- Add business context (user_id, order_id)
- Keep cardinality low
- Include error details

Don't:
- Add PII (personally identifiable info)
- Use high-cardinality values as attributes
- Add large payloads
- Include sensitive data
```

### Performance Considerations

```text
1. Use async span export
2. Sample appropriately
3. Limit attribute count
4. Use span processor batching
5. Consider span limits
```

## Troubleshooting with Traces

### Common Patterns

```text
Finding slow requests:
1. Query traces by duration > threshold
2. Identify slow spans
3. Check span attributes for context

Finding errors:
1. Query traces by status = ERROR
2. See error span and context
3. Check exception details

Finding dependencies:
1. View service map from traces
2. Identify critical paths
3. Find hidden dependencies
```

## Related Skills

- `observability-patterns` - Three pillars overview
- `slo-sli-error-budget` - Using traces for SLIs
- `incident-response` - Using traces in incidents
