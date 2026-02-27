---
name: http-api-design
description: Design and implement lightweight, ergonomic JSON HTTP APIs for machine-to-machine communication. Use this skill whenever the user is designing API endpoints, writing OpenAPI specs, building REST or HTTP API routes, defining request/response schemas, implementing error handling for APIs, or discussing API contracts.
---

# Lightweight HTTP JSON API Design

This skill guides the design and implementation of pragmatic, ergonomic JSON HTTP APIs optimized for machine-to-machine communication. The philosophy is "pragmatic REST" — adopt the parts of REST that genuinely help (resources, statelessness, HTTP semantics) and skip what doesn't earn its keep (HATEOAS, rigid hypermedia specs, heavyweight envelope standards).

The goal: an API that feels obvious to consume, is safe to retry, and can be fully understood from its OpenAPI spec without reading prose documentation.

## Design Principles

These principles are ordered by impact. When two principles conflict, the one listed first wins.

1. **Consistency above all.** An API should look like one person designed it. Every endpoint, field name, error shape, and query parameter should follow the same conventions. Inconsistency is the #1 source of integration bugs.

2. **Optimize for the machine consumer.** This is M2M-first. Every response should be trivially parseable. Stable, typed error codes matter more than friendly messages. Prefer explicit over clever.

3. **Make retries safe by default.** Network failures between services are routine. Idempotency isn't a nice-to-have — it's infrastructure. Design every mutation so clients can safely retry without fear of side effects.

4. **Keep the surface area small.** Start with the minimum viable API. It's easy to add fields and endpoints later; removing them is a breaking change. When unsure, leave it out.

5. **Let HTTP do its job.** Use status codes, methods, headers, and content types as intended. Don't reinvent what the protocol already provides (caching, conditional requests, content negotiation).

## Resource Design

Model your API around domain nouns, not operations. HTTP methods carry the action semantics.

```
GET    /offers                  → List offers
POST   /offers                  → Create an offer
GET    /offers/{offer_id}       → Get a specific offer
PATCH  /offers/{offer_id}       → Partial update
DELETE /offers/{offer_id}       → Remove an offer
```

### URL conventions

- Plural nouns for collections: `/offers`, `/merchants`, `/transactions`
- Identifiers for single resources: `/offers/{offer_id}`
- Lowercase with hyphens for multi-word resources: `/card-linked-offers`
- Shallow nesting — one level maximum: `/merchants/{merchant_id}/offers`
- If a resource has a globally unique ID, also expose it at the top level: `/offers/{offer_id}`
- No verbs in paths. If an action doesn't map cleanly to CRUD, use a noun that represents the process: `POST /offers/{offer_id}/activations` rather than `POST /offers/{offer_id}/activate`

### When nesting vs. top-level

Nest when the child resource has no meaning without the parent and is always accessed in that context. Expose top-level when the resource has a globally unique identifier and consumers may access it independently. It's fine to support both routes to the same resource.

## JSON Conventions

Pick these once. Enforce them everywhere. Lint them in CI.

### Field naming: `snake_case`

```json
{
  "offer_id": "off_8xk2Qp",
  "merchant_name": "Acme Coffee",
  "created_at": "2026-02-11T14:30:00Z",
  "is_active": true
}
```

Why snake_case: it's the convention used by Stripe, GitHub, Slack, and most M2M-oriented APIs. It reads well, avoids ambiguity with acronyms (`api_key` vs `apiKey` vs `APIKey`), and serializes cleanly across languages.

### Core rules

- **Dates and times**: ISO 8601 always, UTC always: `2026-02-11T14:30:00Z`
- **Durations**: ISO 8601 durations (`PT30M`, `P7D`) or integer seconds — pick one
- **Booleans**: Use `is_` or `has_` prefixes: `is_active`, `has_rewards`
- **Enums**: Lowercase `snake_case` strings: `"status": "pending_review"` — never magic numbers
- **Null vs. absent**: Be deliberate. Null means "this field exists but has no value." Absent means "this field doesn't apply." Document which you use and be consistent.
- **Money**: Always a structured object with amount and currency, never a bare number:
  ```json
  { "amount": "19.99", "currency": "USD" }
  ```
  Use string for amount to avoid floating-point issues. Favor composition over inheritance when you need multiple money fields (e.g. `price` and `discounted_price` as separate objects).
- **IDs**: Use prefixed opaque strings (`off_8xk2Qp`, `mer_3kLm9x`). The prefix makes IDs self-documenting in logs and debugging. Avoid exposing auto-increment integers (they leak cardinality).
- **Pluralize arrays, singularize objects**: `"items": [...]` not `"item": [...]`
- **Top-level must be an object**: Never return a bare JSON array. Always wrap in an object to allow future extension.

## Response Envelope

Use a minimal, consistent envelope for all responses.

### Success — single resource

```json
{
  "data": {
    "offer_id": "off_8xk2Qp",
    "merchant_name": "Acme Coffee",
    "reward_amount": { "amount": "5.00", "currency": "USD" },
    "status": "active"
  }
}
```

### Success — collection

```json
{
  "data": [
    { "offer_id": "off_8xk2Qp", "merchant_name": "Acme Coffee" },
    { "offer_id": "off_9yL3Rq", "merchant_name": "Bean There" }
  ],
  "has_more": true,
  "next_cursor": "eyJpZCI6Im9mZl85eUwzUnEifQ"
}
```

### Why an envelope

Without it, you cannot add pagination metadata, deprecation warnings, or request diagnostics without a breaking change. The `data` key is a small price for forward compatibility. Don't over-engineer it — `data` plus pagination fields is sufficient. Avoid deeply nested envelope structures like `{ "response": { "data": { ... }, "meta": { ... } } }`.

## Error Handling — RFC 9457 Problem Details

Use RFC 9457 (`application/problem+json`) for all error responses. This is the one lightweight standard worth adopting — it eliminates the need to invent your own error format and is natively supported by many server frameworks.

```json
{
  "type": "https://api.example.com/errors/validation-failed",
  "title": "Validation Failed",
  "status": 400,
  "detail": "One or more fields failed validation.",
  "errors": [
    {
      "field": "email",
      "detail": "Must be a valid email address.",
      "pointer": "/data/email"
    },
    {
      "field": "reward_amount",
      "detail": "Must be greater than zero.",
      "pointer": "/data/reward_amount/amount"
    }
  ]
}
```

### Key decisions

- **`type`**: A stable URI that uniquely identifies the error category. Machine clients branch on this, not on `title` or `detail`. Make it a URL that resolves to documentation when possible.
- **`title`**: Short, human-readable summary of the problem type (not the specific occurrence).
- **`status`**: Advisory mirror of the HTTP status code. Always match the actual response status.
- **`detail`**: Human-readable explanation of this specific occurrence. Clients must not parse this programmatically — use extension fields instead.
- **Extension fields**: Add domain-specific fields freely. `errors` array for validation details. `retry_after` for rate limits. `balance` and `cost` for insufficient-funds errors. These are what machines should actually act on.

### Status code usage

Use the right HTTP status code. The most important ones for M2M:

| Code | Meaning | When |
|------|---------|------|
| 200 | OK | Successful GET, PATCH, DELETE |
| 201 | Created | Successful POST that creates a resource |
| 202 | Accepted | Async operation acknowledged, not yet complete |
| 204 | No Content | Successful DELETE with no body |
| 400 | Bad Request | Validation failure, malformed input |
| 401 | Unauthorized | Missing or invalid authentication |
| 403 | Forbidden | Authenticated but insufficient permissions |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | State conflict (e.g., duplicate, version mismatch) |
| 422 | Unprocessable Entity | Syntactically valid but semantically wrong |
| 429 | Too Many Requests | Rate limited — always include `Retry-After` |
| 500 | Internal Server Error | Server bug — include `request_id` for debugging |
| 503 | Service Unavailable | Temporary overload — include `Retry-After` |

Return 400 for structural input problems (missing fields, wrong types). Return 422 for business rule violations on well-formed input (insufficient balance, conflicting state). This distinction helps M2M clients separate "fix your request" from "fix the preconditions."

## Pagination

Use cursor-based pagination for all list endpoints. It's stable under concurrent writes and performs well at scale. Offset-based pagination breaks when rows are inserted or deleted between pages.

```
GET /offers?limit=25&cursor=eyJpZCI6Im9mZl85eUwzUnEifQ
```

Response:
```json
{
  "data": [...],
  "has_more": true,
  "next_cursor": "eyJpZCI6Im9mZl8xMGFCNHMifQ"
}
```

- `limit`: Maximum items to return (set a sensible default, e.g. 25, and a max, e.g. 100)
- `cursor`: Opaque string. Base64-encode the pagination state. Never expose raw IDs or offsets in cursors.
- `has_more`: Boolean — tells the client whether to keep paginating without decoding the cursor
- `next_cursor`: Only present when `has_more` is true

## Filtering and Sorting

Keep it simple. Use query parameters for filtering and sorting. Don't invent a query language unless you genuinely need one.

```
GET /offers?status=active&merchant_id=mer_3kLm9x&sort=created_at:desc
```

- Filter fields should match the resource's field names exactly
- Sort format: `field:direction` where direction is `asc` or `desc`
- For multiple filters on the same field, use comma separation: `?status=active,pending`
- For date ranges: `?created_after=2026-01-01T00:00:00Z&created_before=2026-02-01T00:00:00Z`

Avoid deeply expressive filter DSLs. If consumers need ad-hoc queries over your data, that's a different product (a query API or GraphQL layer), not a REST endpoint.

## Idempotency

Every mutation endpoint must be safe to retry.

- **GET, HEAD, OPTIONS**: Inherently safe and idempotent — no special handling needed.
- **PUT, DELETE**: Idempotent by definition when designed correctly (same input → same result).
- **POST**: Requires explicit idempotency support via `Idempotency-Key` header.

### Implementation

Clients send a unique key (recommend V4 UUID) with each POST request:

```
POST /offers
Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000
Content-Type: application/json

{ "merchant_id": "mer_3kLm9x", "reward_type": "cashback" }
```

Server behavior:

1. **First request with this key**: Process normally, cache the response (status code + body) keyed by the idempotency key.
2. **Subsequent requests with same key and same parameters**: Return the cached response without re-executing.
3. **Same key but different parameters**: Return 422 error — prevents accidental misuse.
4. **Key expiry**: Automatically prune keys after 24 hours. A reused key after expiry starts a new request.

Cache the response regardless of whether it succeeded or failed (including 500s). This prevents double-execution when clients retry after ambiguous failures.

For requests that fail validation before execution begins (parameter errors, auth failures), don't store the idempotency result — let the client fix and retry with the same key.

## Rate Limiting

Return rate limit state on every response so clients can self-throttle:

```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 847
X-RateLimit-Reset: 1739290200
```

On 429 responses, always include the `Retry-After` header (seconds until the client should retry). Also include it on 503 responses during load shedding.

Machines respect `Retry-After` if you provide it. If you don't, they'll hammer you with exponential backoff guesses that are usually more aggressive than necessary.

## Operational Headers

Include these on every response:

- **`X-Request-Id`**: Unique identifier for the request. Generate server-side if the client doesn't provide one. Invaluable for distributed tracing and support debugging.
- **`Content-Type: application/json`** (or `application/problem+json` for errors)

Accept these from clients:

- **`Idempotency-Key`**: For POST requests (see above)
- **`X-Request-Id`**: If the client sends one, use it (or derive from it) so logs correlate across services

## Authentication

For M2M server-to-server communication, API keys are the pragmatic default:

- Send via `Authorization: Bearer <api_key>` header (not query params, not custom headers)
- Prefix keys for identification: `sk_live_...`, `sk_test_...`
- Support key rotation: allow multiple active keys per client with independent expiry

Use OAuth 2.0 client credentials flow when you need scoped access or third-party delegation. Use JWTs when you need claims to propagate across service boundaries without a lookup.

Always require HTTPS. No exceptions.

## Async Operations

For operations that take longer than a few seconds, return 202 Accepted with a status resource:

```json
{
  "data": {
    "operation_id": "op_7mN4kP",
    "status": "processing",
    "status_url": "/operations/op_7mN4kP",
    "created_at": "2026-02-11T14:30:00Z"
  }
}
```

Clients poll `status_url` until `status` transitions to `completed` or `failed`. The completed response should include the result resource or a link to it.

## Versioning

Do not version in the URL. Instead, design for evolution:

- **Additive changes are not breaking**: New fields in responses, new optional query parameters, new endpoints, new enum values in documentation — none of these break existing clients.
- **Require tolerant readers**: Clients must ignore unknown fields. Document this as a contract requirement in your API spec. This is the single most important compatibility rule.
- **When breaking changes are truly unavoidable**: Use a versioned media type in the `Accept` header:
  ```
  Accept: application/json; version=2
  ```
  Or a custom header:
  ```
  API-Version: 2024-01-15
  ```
  Date-based versions (Stripe's approach) communicate when the contract was established and make deprecation timelines intuitive.

- **Deprecation process**: Add `Deprecation` and `Sunset` headers to responses for endpoints or fields being retired. Provide migration guides. Give consumers at least 6 months notice.

The reason to avoid URL versioning: it fragments your API surface, breaks caching semantics, complicates routing, and encourages lazy breaking changes instead of thoughtful evolution. An API that never breaks is better than one that versions easily.

## OpenAPI Specification

Treat your OpenAPI spec as a first-class artifact — not generated documentation, but the source of truth.

- Use OpenAPI 3.1 (full JSON Schema compatibility)
- Write the spec first, before implementation (API-First design)
- Version control it alongside your code
- Lint it in CI (use Spectral, Redocly, or similar)
- Include complete `examples` for every request and response
- Mark fields as `required` vs optional explicitly
- Use `$ref` for shared schemas (error responses, pagination, money objects)
- Publish it at a well-known path: `GET /openapi.json`

Consumers will generate clients from this spec. A good spec with accurate types, clear constraints, and realistic examples is worth more than pages of prose docs.

## Implementation Checklist

When implementing an API following this skill, verify:

- [ ] All URLs use plural nouns, lowercase-hyphenated, max one level of nesting
- [ ] All JSON fields use `snake_case`
- [ ] All dates are ISO 8601 UTC
- [ ] All responses are wrapped in a `{ "data": ... }` envelope
- [ ] All errors use RFC 9457 Problem Details format
- [ ] All errors include a `type` URI that machines can branch on
- [ ] All list endpoints use cursor-based pagination with `has_more` and `next_cursor`
- [ ] All POST endpoints accept `Idempotency-Key` header
- [ ] All responses include `X-Request-Id` header
- [ ] All responses include rate limit headers
- [ ] 429 and 503 responses include `Retry-After`
- [ ] No bare arrays in responses — always an object at top level
- [ ] IDs use prefixed opaque strings
- [ ] Money is always `{ amount, currency }` with string amount
- [ ] HTTPS required, no exceptions
- [ ] OpenAPI 3.1 spec exists, is linted, and is published at `/openapi.json`
- [ ] Clients are expected to ignore unknown fields (documented)

## What Not to Adopt

These are intentionally excluded because they add weight without proportional value for M2M JSON APIs:

- **HATEOAS / HAL / Siren / JSON-LD**: Adds envelope complexity and response bloat. Machine clients don't discover APIs dynamically — integration code is written against a spec, not navigated at runtime.
- **JSON:API**: Opinionated spec with rigid envelope structures, relationship graphs, and compound documents. Overkill for most M2M APIs.
- **OData**: Enterprise-oriented query language, more suited to generic CRUD over large ERP-style entity models.
- **GraphQL**: Different paradigm. Strong for client-driven queries but adds operational complexity (per-field authorization, caching, rate limiting) that's unnecessary when you control the API surface.
- **URL versioning**: Fragments the API surface and discourages evolutionary design. See Versioning section.

For further reading, see the references directory:
- `references/error-examples.md` — Expanded RFC 9457 error examples for common scenarios
- `references/openapi-template.md` — Starter OpenAPI 3.1 template following these conventions
