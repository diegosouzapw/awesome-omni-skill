---
name: Implementing Observability
description: Instrument the application with Logging, Metrics, and Tracing (OpenTelemetry) to understand system behavior and debug production issues.
---

# Implementing Observability

## Goal
Make the system's internal state inferable from its external outputs. Answer "Why is it slow?" and "Why did it fail?" without SSH-ing into a server.

## When to Use
- Before launching to production.
- When debugging a performance bottleneck.
- When integrating a new microservice or external API.

## Instructions

### 1. Structured Logging
Text logs are hard to query. Use JSON.
- **Context**: Every log must have `trace_id`, `request_id`, `user_id`.
- **Levels**: `INFO` for normal ops, `WARN` for handled issues, `ERROR` for unhandled crashes.

```json
{"level": "info", "msg": "User logged in", "user_id": 123, "trace_id": "abc-123"}
```

### 2. Distributed Tracing (OpenTelemetry)
Trace a request across boundaries (Frontend -> API -> DB).
- Instrument HTTP clients and server frameworks.
- Visualize the "waterfall" to find the slow span.

### 3. Golden Signals (Metrics)
Track the four key metrics for every service:
- **Latency**: Time to serve a request.
- **Traffic**: Request rate (RPS).
- **Errors**: Rate of 5xx responses.
- **Saturation**: CPU/Memory/Disk usage.

### 4. Alerting
Alert on symptoms (High Error Rate), not causes (High CPU).
- **Page**: If `Error Rate > 1%` for 5 minutes.
- **Ticket**: If `Disk Usage > 80%`.

## Constraints

### ✅ Do
- **DO**: Use OpenTelemetry standards for portability.
- **DO**: Correlate logs and traces (inject trace ID into logs).
- **DO**: Sample high-volume traces (10%) to save costs, but keep 100% of errors.

### ❌ Don't
- **DON'T**: Log PII (Emails, Passwords, Credit Cards).
- **DON'T**: Create alerts that auto-resolve in seconds (flapping).
- **DON'T**: Rely solely on "system up" checks; check "business logic working".

## Output Format
- `docker-compose.yml` with Prometheus/Grafana/Jaeger (for dev).
- Code instrumentation (e.g., `tracing.py`).

## Dependencies
- `backend/managing-flask-middleware/SKILL.md` (where instrumentation lives)
- `shared/debugging/SKILL.md`
