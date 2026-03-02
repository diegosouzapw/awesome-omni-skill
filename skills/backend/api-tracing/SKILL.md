---
name: api-tracing
description: Instrument API requests with spans and distributed tracing. Use when tracking request latency or debugging API issues.
triggers:
  - "API tracing"
  - "track API requests"
  - "distributed tracing"
priority: 2
---

# API Tracing

Measure API requests and correlate with backend traces.

## What to Capture (OTel-Compatible)

| Attribute | OTel Name | Purpose |
|-----------|-----------|---------|
| Method | `http.request.method` | GET, POST, etc. |
| Status | `http.response.status_code` | Success/failure |
| URL | `url.path` | Endpoint (sanitized) |
| Duration | `http.request.duration` | Request time (ms) |

Using OTel naming now = easier migration later.

## Key Thresholds

| Metric | Good | Acceptable | Poor |
|--------|------|------------|------|
| p50 | <200ms | <500ms | >500ms |
| p95 | <1s | <2s | >2s |
| Error rate | <0.1% | <1% | >1% |

## What NOT to Log

| Don't | Why |
|-------|-----|
| Request bodies | PII risk |
| Auth headers | Security |
| Full response data | Size limits |
| User tokens | Security |

**Do log:** Request ID, sanitized path, status code, duration, error type.

## Implementation

See `templates/api-interceptor.ts` for fetch/axios interceptors.

Use Read tool to load template when generating implementation.

## Related

- `skills/error-tracking` - API error handling
- `skills/route-transition-tracking` - Data fetch during navigation
- `references/otel-web.md` - OpenTelemetry naming
