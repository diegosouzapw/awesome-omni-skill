---
name: developing-backend-services
description: Backend service development best practices. Use when designing, building, or reviewing backend services, REST APIs, gRPC services, microservices, webhooks, message queues, or server-side applications regardless of language or framework.
---

# Backend Service Development Best Practices

## API Design

Design APIs as resource-oriented interfaces. Clients should interact with nouns, not verbs.

- **Resource URLs**: plural nouns, nested for relationships: `/users`, `/users/{id}/orders`.
- **HTTP verbs**: GET (read), POST (create), PUT (full replace), PATCH (partial update), DELETE (remove).
- **Status codes**: use the most specific code. Never return 200 with an error body.
- **Versioning**: URL prefix (`/v1/users`) for public APIs, `Accept` header for internal. Never break existing versions. On deprecation, return `Sunset` and `Deprecation` headers — give consumers 6+ months to migrate. Run v1 and v2 in parallel with a shared data layer until traffic on the old version drops to zero.
- **Pagination**: cursor-based over offset-based — offset breaks under concurrent writes.
  ```
  GET /orders?cursor=eyJpZCI6MTAwfQ&limit=25
  Response: { "data": [...], "next_cursor": "eyJpZCI6MTI1fQ", "has_more": true }
  ```
- **Error format**: consistent JSON envelope on every error.
  ```json
  {
    "error": {
      "code": "VALIDATION_FAILED",
      "message": "Request validation failed",
      "details": [
        { "field": "email", "reason": "must be a valid email address" }
      ]
    }
  }
  ```
- **Idempotency**: accept `Idempotency-Key` header on POST/PATCH. Store key → response mapping. Return cached response on duplicate key.
- **Internal services**: prefer gRPC — schema-first (protobuf), streaming, codegen. REST for public-facing APIs.
- **Documentation**: generate OpenAPI/Swagger spec from code annotations or maintain spec-first. Validate requests against the spec in tests.
- **GraphQL**: consider when clients need flexible data fetching (mobile/web with different data requirements) or when you have a complex graph of related resources.
  - ✅ Single endpoint; clients request exactly the fields they need — no over/under-fetching
  - ✅ Strongly typed schema (`schema.graphql`) is the contract; self-documenting via introspection
  - ❌ Harder to cache (POST by default); requires query depth/complexity limiting to prevent abuse
  - ❌ File uploads, streaming, and event-driven patterns are less natural than REST or SSE
  - Use code-first (Pothos for TypeScript, Strawberry for Python) or schema-first (SDL + codegen). Always generate typed client code from the schema.

Full patterns and pseudocode: see [patterns/api-design-patterns.md](patterns/api-design-patterns.md).

## Authentication & Authorization

- **Stateless auth**: use JWT or opaque tokens. Validate on every request without server-side session state.
- **Token lifecycle**: short-lived access tokens (5-15 min) + long-lived refresh tokens (days/weeks). Rotate refresh tokens on use.
- **Separate authn and authz**: authentication (who are you?) runs first, authorization (what can you do?) runs second. Different middleware, different logic.
- **Password hashing**: bcrypt (cost 12+) or argon2id. Never MD5, SHA-256, or any unsalted hash.
- **API keys**: hash before storing (SHA-256 is fine for API keys since they have high entropy). Show the key once at creation, never again.
- **Rate limit auth endpoints**: aggressive limits on `/login`, `/register`, `/password-reset` to block credential stuffing.
- **CORS**: whitelist specific origins. Never `Access-Control-Allow-Origin: *` on authenticated endpoints. Keep preflight cached with `Access-Control-Max-Age`.

## Configuration Management

- **Twelve-factor config**: read all configuration from environment variables. No hardcoded connection strings, no config files baked into images.
- **Validate at startup**: parse and validate every config value on boot. Fail fast with a clear error message if any required value is missing or malformed.
- **Typed config struct**: map env vars to a strongly-typed struct/object at startup — not scattered `os.getenv()` calls throughout the codebase.
  ```
  config = Config(
    db_url      = require_env("DATABASE_URL"),
    port        = parse_int(env("PORT", "8080")),
    log_level   = parse_enum(env("LOG_LEVEL", "info"), ["debug","info","warn","error"]),
  )
  ```
- **Secrets management**: in production, fetch secrets from a secret manager (Vault, AWS Secrets Manager, GCP Secret Manager) — not plain env vars. Env vars show up in process listings and crash dumps.
- **Feature flags**: use a flag system (LaunchDarkly, Unleash, or a simple DB table) for gradual rollouts. Decouple deployment from release.

See [twelve-factor-cheatsheet.md](twelve-factor-cheatsheet.md) for the full twelve-factor breakdown.

## Database Patterns

- **Migrations**: forward-only, sequentially numbered, idempotent. Never modify a migration that has been released. Add a new migration to fix issues.
  ```
  001_create_users.sql
  002_add_email_index.sql
  003_create_orders.sql
  ```
- **Connection pooling**: size pool to `(2 * CPU cores) + 1` (typically 10-20 per instance). Tune based on actual wait times — not a formula. Set connection timeout (5s), idle timeout (10min), max lifetime (30min).
- **Transactions**: keep short — lock contention grows with duration. Use read-only transactions for queries. Retry on serialization failures (SQLSTATE 40001). Prefer optimistic locking (`WHERE version = ?` + increment) over pessimistic locks.
- **Queries**: parameterized always — no string concatenation, no exceptions. Add indexes for WHERE/JOIN/ORDER BY columns. Run `EXPLAIN ANALYZE` on slow queries. Watch for N+1 patterns in ORMs.

Full patterns: see [patterns/database-patterns.md](patterns/database-patterns.md).

## Observability

Three pillars: logs, metrics, traces. Every backend service must emit all three. Use **OpenTelemetry (OTel)** as the instrumentation standard — it provides a single vendor-neutral SDK for all three pillars with auto-instrumentation for common frameworks and libraries.

- **Structured logging**: JSON format with standard fields: `timestamp`, `level`, `message`, `service`, `request_id`, `trace_id`. Never log secrets, tokens, passwords, PII in plaintext.
- **Metrics**: RED method for services (Rate, Errors, Duration). USE method for resources (Utilization, Saturation, Errors). Use histograms for latency — not averages.
  ```
  http_requests_total{method, path, status}       # counter
  http_request_duration_seconds{method, path}      # histogram
  ```
- **Distributed tracing**: propagate W3C Trace Context headers (`traceparent`, `tracestate`). Create a span for every external call (DB, HTTP, queue). Add attributes: `db.statement`, `http.method`, `http.status_code`.
- **Correlation**: generate `X-Request-ID` at the edge, propagate through all services. Include in every log line.

Full patterns: see [patterns/observability-patterns.md](patterns/observability-patterns.md).

## Health Checks & Lifecycle

Expose three health endpoints:

- **`/health/live`** — is the process alive? Returns 200 if the process is running. No dependency checks. Used by orchestrator liveness probes.
- **`/health/ready`** — can it serve traffic? Checks DB connection, cache connectivity, required downstream services. Used by load balancer / readiness probes.
- **`/health/startup`** — has it finished initializing? Returns 503 until migrations, cache warming, etc. complete. Prevents premature traffic.

**Graceful shutdown** — on SIGTERM/SIGINT:

```
1. Stop accepting new connections (close listener)
2. Set /health/ready to return 503
3. Wait for in-flight requests to complete (with timeout, e.g. 30s)
4. Close database connections, flush buffers
5. Exit with code 0
```

Register shutdown handlers for SIGTERM (orchestrator) and SIGINT (Ctrl+C). Set a hard deadline — force exit if drain exceeds timeout.

## Rate Limiting & Caching

### Rate Limiting

- **Algorithms**: token bucket (bursty traffic) or sliding window (smooth limits). Choose based on traffic pattern.
- **Response**: return `429 Too Many Requests` with `Retry-After` header (seconds until next allowed request).
- **Granularity**: differentiate limits by user tier, IP, and endpoint. Auth endpoints get stricter limits than read endpoints.
- **Distributed**: use Redis-based rate limiting (INCR + EXPIRE or Lua script) for multi-instance deployments. Local rate limiting only works for single-instance.

### Caching

- **Cache-aside**: application checks cache first, fetches from DB on miss, populates cache. Most common pattern.
- **Write-through**: write to cache and DB simultaneously. Use when reads heavily outweigh writes and stale data is unacceptable.
- **TTL always**: every cache entry must have a TTL. No TTL = stale data forever.
- **Versioned keys**: include a version in cache keys (`user:v2:{id}`) to invalidate on schema changes without flushing everything.
- **HTTP caching**: set `Cache-Control`, `ETag`, `Last-Modified` headers on GET responses. Use `no-store` for sensitive data.

## Message Queues & Async Processing

Use queues when: the operation is slow (email, PDF generation), the caller doesn't need the result immediately, you need to decouple services, or you need to buffer spikes.

- **Consumer idempotency**: every consumer must handle duplicate messages. Use a processed-message table or idempotency key.
- **Dead letter queue (DLQ)**: route messages that fail after N retries to a DLQ. Monitor DLQ depth. Build tooling to replay from DLQ.
- **At-least-once**: design for at-least-once delivery — every major queue system can deliver duplicates. Exactly-once is a lie at the infrastructure level; achieve it at the application level with idempotency.
- **Backpressure**: limit consumer concurrency. Prefetch a bounded number of messages. If processing slows down, let the queue buffer — don't let consumers OOM.
- **Message schema**: version your message payloads. Include a `type` and `version` field. Support backward-compatible evolution.

## Webhooks

- **Outbound delivery**: POST JSON to subscriber URLs. Sign with HMAC-SHA256 of `{timestamp}.{body}` using a shared secret. Send signature in `X-Webhook-Signature` header, event ID in `X-Webhook-ID`.
- **Retry policy**: retry on 5xx or timeout with exponential backoff (1s, 5s, 25s, 125s, 625s). After 5 failures, disable the endpoint and notify the subscriber.
- **Delivery log**: store every attempt (timestamp, status code, latency) for debugging. Expose via admin API.
- **Inbound webhooks**: verify signature before processing. Respond 200 immediately, then process asynchronously via a queue. Deduplicate by event ID.

See [patterns/api-design-patterns.md](patterns/api-design-patterns.md) for signature verification pseudocode.

## Error Handling

- **Fail fast**: on unrecoverable errors (missing required config, corrupt state), crash immediately with a clear message. Don't limp along.
- **Retry transient errors**: network timeouts, 503s, connection resets. Use exponential backoff with jitter: `delay = min(base * 2^attempt + random_jitter, max_delay)`.
- **Circuit breaker**: protect against cascading failures. Three states: closed → open (after threshold failures) → half-open (probe requests). Typical config: `failure_threshold=5, recovery_timeout=30s, half_open_max=3`.
- **Wrap errors with context**: when propagating errors, add what operation failed and why. `"failed to fetch user 123: connection refused"` over `"connection refused"`.

## Testing

- **Unit tests**: test service/handler logic in isolation. Mock external dependencies (DB, HTTP clients, queues) via interfaces. Focus on edge cases: invalid input, missing fields, boundary values, error paths. Aim for 80%+ coverage on business logic.
- **Integration tests**: test against real dependencies using containers (testcontainers or `docker compose` in CI). Verify actual SQL queries, migration correctness, cache behavior, queue consumption. These catch bugs mocks hide.
- **Contract tests**: validate API responses against the OpenAPI spec. Use tools like `schemathesis` or `dredd` to fuzz every endpoint. Detect breaking changes (removed fields, type changes) before merge.
- **Load tests**: k6 or locust against staging. Define thresholds: p99 latency, error rate. Run on release branches, not every PR. Store results for trend analysis.
- **Test doubles strategy**: use fakes for databases (testcontainers), mocks for HTTP clients (WireMock), and real queues (testcontainers with RabbitMQ/SQS). Mock what you don't own; fake what you do.

Full patterns: see [patterns/testing-patterns.md](patterns/testing-patterns.md).

## Security

- **Input validation**: validate at the service boundary — type, length, range, format. Reject invalid input early with descriptive errors.
- **Output encoding**: encode output based on context (HTML, JSON, SQL). Prevents XSS and injection even if input validation is bypassed.
- **Dependency scanning**: run `dependabot`, `snyk`, or `trivy` in CI. Block merges on high/critical CVEs.
- **TLS everywhere**: all service-to-service communication over TLS. Use mTLS (mutual TLS) for internal service mesh — both sides present certificates.
- **OWASP Top 10**: address injection, broken auth, sensitive data exposure, XXE, broken access control, misconfiguration, XSS, insecure deserialization, vulnerable components, insufficient logging.
- **Security headers**: `Strict-Transport-Security`, `Content-Security-Policy`, `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Referrer-Policy: strict-origin-when-cross-origin`, `Permissions-Policy: camera=(), microphone=(), geolocation=()`.
- **Least privilege**: service accounts get minimum required permissions. No shared credentials between services.

## New service workflow

```
- [ ] Define API contract (OpenAPI spec or protobuf)
- [ ] Set up project structure with build tooling
- [ ] Configure typed settings with startup validation
- [ ] Set up database with migration framework
- [ ] Implement health check endpoints (/health/live, /health/ready, /health/startup)
- [ ] Add structured logging and request correlation
- [ ] Add metrics instrumentation (RED method)
- [ ] Add distributed tracing
- [ ] Implement authentication and authorization middleware
- [ ] Add rate limiting
- [ ] Add graceful shutdown handling
- [ ] Write unit tests, integration tests, and contract tests (see Testing section)
- [ ] Configure CI pipeline (lint, test, scan, build)
- [ ] Set up alerting (SLO-based)
- [ ] Document runbook and API
- [ ] Run validation loop (below)
```

## Validation loop

1. **Lint** — run language linter and `openapi-generator validate` on API spec
2. **Unit tests** — run with coverage; fail below threshold (e.g. 80%)
3. **Integration tests** — test against real dependencies (DB, cache, queue) using containers
4. **Load test** — run k6/locust against staging; verify p99 latency and error rate under expected load
5. **Security scan** — `trivy`, `snyk`, or `dependabot`; fail on high/critical CVEs
6. **API contract test** — validate responses match OpenAPI spec; detect breaking changes
7. Repeat until all checks pass

## Deep-dive references

- **API design patterns**: see [patterns/api-design-patterns.md](patterns/api-design-patterns.md) for resource naming, pagination, idempotency middleware, webhook delivery
- **Testing patterns**: see [patterns/testing-patterns.md](patterns/testing-patterns.md) for integration test setup, contract testing, load testing, test doubles
- **Observability patterns**: see [patterns/observability-patterns.md](patterns/observability-patterns.md) for logging setup, metrics, tracing, alerting rules
- **Database patterns**: see [patterns/database-patterns.md](patterns/database-patterns.md) for migrations, pooling, transactions, query optimization
- **Twelve-factor cheatsheet**: see [twelve-factor-cheatsheet.md](twelve-factor-cheatsheet.md) for all 12 factors with backend-specific guidance

## Official references

- [The Twelve-Factor App](https://12factor.net/) — methodology for building SaaS apps
- [Microsoft REST API Guidelines](https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md) — comprehensive REST design guide
- [OWASP API Security Top 10](https://owasp.org/API-Security/) — top API security risks and mitigations
- [OpenTelemetry](https://opentelemetry.io/docs/) — observability framework for traces, metrics, logs