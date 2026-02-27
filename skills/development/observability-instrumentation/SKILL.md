---
name: observability-instrumentation
description: Use when adding endpoints/background jobs/integrations where telemetry matters. Do NOT refactor unrelated code. Prefer OpenTelemetry-friendly patterns.
---

Principles:
- Low-cardinality metrics labels.
- Structured logs with correlation IDs; never log secrets.
- Trace the "edges": inbound request, outbound calls, key domain operations.

Checklist:
- Tracing
  - Create spans around external calls (HTTP, DB, message bus, 3rd party SDK)
  - Add useful span attributes (route, status code, dependency name)
  - Propagate context across async boundaries and between services
- Metrics
  - Add counters for throughput and errors
  - Add histograms for latency where needed
  - Avoid user IDs, request IDs, or other high-cardinality labels
- Logging
  - Structured logging (message templates)
  - Correlation/trace IDs included (or derived from OTel)
  - Redaction of PII/secrets
- Health & readiness
  - Ensure health endpoints or probes reflect real dependencies (as per repo convention)
- Verification
  - Verify local export path (console/OTLP) and sample traces appear
  - Add minimal docs: how to run locally + where to look in dashboards

Finish with:
- What telemetry was added (spans/metrics/logs)
- How to validate locally
- Risks (cardinality, PII, perf)
