---
name: api-design
description: Design REST and GraphQL APIs. Covers endpoint naming, versioning, OpenAPI 3.1 specs, pagination, error handling, and auth patterns.
---

# API Design

---

## Quick Start

Just describe your domain:

```
design an API for an e-commerce platform with products, orders, and customers
```

You'll get a complete endpoint design and OpenAPI snippet like:

```yaml
paths:
  /v1/products:
    get:
      summary: List products
      parameters:
        - $ref: '#/components/parameters/PageLimit'
        - $ref: '#/components/parameters/PageCursor'
      responses:
        '200':
          description: Paginated product list
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ProductListResponse'
  /v1/products/{id}:
    get:
      summary: Get product by ID
      responses:
        '200':
          description: Product detail
        '404':
          $ref: '#/components/responses/NotFound'
```

**What to include in your request:**
- Domain or system name (e-commerce, billing, identity)
- Key resources (users, products, orders)
- Relationships (orders belong to users, orders have line items)
- Consumer context (mobile app, internal service, public API)
- Existing constraints (auth system, versioning requirements)

---

## Triggers

| Trigger | Example |
|---------|---------|
| `design an API for` | "design an API for a task management system" |
| `create endpoints for` | "create endpoints for user authentication" |
| `OpenAPI spec for` | "OpenAPI spec for the billing service" |
| `GraphQL schema for` | "GraphQL schema for a content management domain" |
| `review api` | "review this API for problems" |
| `add endpoints for` | "add endpoints for order fulfillment" |
| `design REST API` | "design REST API for the notification service" |

---

## Key Terms

| Term | Definition |
|------|------------|
| **REST** | Representational State Transfer — stateless, resource-oriented HTTP API style |
| **GraphQL** | Query language for APIs with a single endpoint and client-driven data fetching |
| **OpenAPI** | Standard specification format for describing REST APIs (current version: 3.1) |
| **Endpoint** | A URL path + HTTP method combination that handles a specific operation |
| **Resource** | A noun representing an entity in your domain (user, order, product) |
| **Schema** | Formal definition of the shape and constraints of request/response data |
| **Idempotency** | An operation that produces the same result no matter how many times it's called |
| **HATEOAS** | Hypermedia As The Engine Of Application State — embedding navigable links in responses |
| **Pagination** | Strategy for returning large datasets in chunks (offset-based or cursor-based) |
| **Rate Limiting** | Restricting how often a client can call an endpoint within a time window |

---

## Quick Reference

| Task | Approach | Key Consideration |
|------|----------|-------------------|
| New API | Start with resource identification | Model the domain, not the UI |
| Versioning | URL path `/v1/` for simplicity | Commit to support policy before launch |
| Pagination | Cursor-based for large/live data | Offset breaks on concurrent writes |
| Auth | Bearer token (JWT) as default | Scope to least privilege |
| Error handling | `{ error: { type, message, subcode } }` envelope | Machine-readable codes, human-readable messages |
| GraphQL vs REST | REST for CRUD, GraphQL for flexible queries | Consider client diversity and caching needs |

---

## Process Overview

```
Your API Requirements
    |
    v
+-----------------------------------------------------+
| Phase 1: REQUIREMENTS                               |
| * Identify resources and relationships              |
| * Determine consumers (mobile, web, service-to-service) |
| * Define auth model and access control needs        |
| * Enumerate constraints (rate limits, SLAs, quotas) |
+-----------------------------------------------------+
    |
    v
+-----------------------------------------------------+
| Phase 2: DESIGN                                     |
| * Define URL structure and resource hierarchy       |
| * Map HTTP methods to CRUD operations               |
| * Design request/response schemas                   |
| * Choose auth mechanism and scoping                 |
+-----------------------------------------------------+
    |
    v
+-----------------------------------------------------+
| Phase 3: SPECIFY                                    |
| * Generate OpenAPI 3.1 spec (or GraphQL SDL)        |
| * Define error codes and Problem Details format     |
| * Document pagination strategy and parameters       |
| * Add examples for every endpoint                   |
+-----------------------------------------------------+
    |
    v
+-----------------------------------------------------+
| Phase 4: VALIDATE                                   |
| * Run verification checklist                        |
| * Assess breaking change risk                       |
| * Review auth coverage                              |
| * Confirm documentation completeness               |
+-----------------------------------------------------+
    |
    v
Production-Ready API Specification
```

---

## Commands

| Command | When to Use | Action |
|---------|-------------|--------|
| `design api for {domain}` | Starting fresh | Full endpoint design + OpenAPI spec |
| `add endpoints for {resource}` | Extending existing API | New resource endpoints |
| `openapi spec for {api}` | Formalizing existing design | Complete OpenAPI 3.1 document |
| `graphql schema for {domain}` | GraphQL API | Type definitions + queries + mutations |
| `review api` | Auditing existing API | Checklist audit + recommendations |

**Workflow:** Start with `design api` → formalize with `openapi spec` → extend with `add endpoints` → audit with `review api`

---

## Core Principles

| Principle | WHY | Implementation |
|-----------|-----|----------------|
| Resource-oriented design | Resources are stable; operations on them are predictable | Nouns in URLs, verbs in HTTP methods |
| Consistent naming | Reduces cognitive load for consumers | Plural nouns, kebab-case paths, camelCase fields |
| Explicit versioning | Allows evolution without breaking clients | Version in URL path or header, committed from day one |
| Defensive schemas | Fail fast on bad input; never trust caller data | Required fields, format constraints, max lengths |

---

## Anti-Patterns

| Avoid | Why | Instead |
|-------|-----|---------|
| Verbs in URLs (`/getUser`, `/createOrder`) | Conflates HTTP method semantics | `GET /users/{id}`, `POST /orders` |
| Inconsistent pluralization | Forces clients to memorize per-resource conventions | Always plural: `/users`, `/orders`, `/products` |
| No versioning on day one | First breaking change forces emergency migration | Add `/v1/` from the start |
| Generic error responses (`{"error": "failed"}`) | Client can't distinguish or act on error | `{ error: { type, message, subcode } }` with typed codes |
| Breaking changes without deprecation | Breaks deployed clients silently | Additive changes only; `Sunset` header for removals |
| Nesting resources more than 2 levels deep | URLs become unmaintainable and coupling increases | `/orders/{id}/items` yes; `/users/{id}/orders/{id}/items/{id}` no |
| FLOAT for monetary values in responses | Floating-point rounding corrupts financial data | Return money as integer cents or string decimal |
| Exposing internal IDs or database row numbers | Leaks implementation, complicates migration | Use opaque IDs (UUIDs or slugs) |

---

## Verification Checklist

After designing an API:

- [ ] All resource URLs use plural nouns
- [ ] No verbs in URL paths
- [ ] Nesting depth is 2 levels maximum
- [ ] URL paths use kebab-case for multi-word segments
- [ ] HTTP methods match semantics (GET=read, POST=create, PUT=replace, PATCH=partial, DELETE=remove)
- [ ] PUT and DELETE are idempotent
- [ ] Status codes are appropriate (201 for creation, 204 for no-content DELETE, 409 for conflict)
- [ ] Version is in URL path or header, documented from day one
- [ ] All endpoints with state mutation require auth
- [ ] Error responses use `{ error: { type, message, subcode } }` envelope
- [ ] Validation errors include field-level detail
- [ ] Pagination included for all collection endpoints
- [ ] Pagination response includes `meta.total`, `meta.cursor`, or `links.next`
- [ ] Money/currency fields use integer cents or string decimal (never float)
- [ ] Dates use ISO 8601 in UTC (`2025-10-31T14:30:00Z`)
- [ ] Content-Type is `application/json` on all JSON endpoints
- [ ] Rate limiting headers documented (`X-RateLimit-Limit`, `X-RateLimit-Remaining`)
- [ ] OpenAPI spec validates without errors
- [ ] Every endpoint has at least one request/response example
- [ ] Breaking changes assessed against existing consumers

See [api-design-checklist.md](references/api-design-checklist.md) for the full pre-launch review checklist.
See [openapi-templates.md](references/openapi-templates.md) for concrete OpenAPI 3.1 spec templates.
See [scalar-api-docs.md](references/scalar-api-docs.md) for interactive API documentation setup (Go, Python, TypeScript, Rust).

---

<details>
<summary><strong>Deep Dive: REST Endpoint Design</strong></summary>

### Resource Naming

| Rule | Good | Bad |
|------|------|-----|
| Plural nouns | `/users`, `/orders` | `/user`, `/order` |
| kebab-case | `/line-items`, `/tax-rates` | `/lineItems`, `/taxRates` |
| No verbs | `POST /orders` | `POST /createOrder` |
| Opaque IDs | `/users/usr_abc123` | `/users/42` |

### URL Structure

```
/v1/{resource}               # collection
/v1/{resource}/{id}          # single resource
/v1/{resource}/{id}/{child}  # sub-resource (max 2 levels)
```

**Examples:**
```
GET    /v1/orders              # list orders
POST   /v1/orders              # create order
GET    /v1/orders/{id}         # get order
PUT    /v1/orders/{id}         # replace order
PATCH  /v1/orders/{id}         # partial update
DELETE /v1/orders/{id}         # delete order
GET    /v1/orders/{id}/items   # list items for order
POST   /v1/orders/{id}/items   # add item to order
```

### HTTP Method Mapping

| Method | Semantics | Idempotent | Body |
|--------|-----------|------------|------|
| GET | Read, no side effects | Yes | No |
| POST | Create, trigger action | No | Yes |
| PUT | Replace entire resource | Yes | Yes |
| PATCH | Partial update | No* | Yes |
| DELETE | Remove resource | Yes | No |

*PATCH can be made idempotent with conditional requests (`If-Match`)

### Status Code Reference

| Code | Meaning | When to Use |
|------|---------|-------------|
| 200 | OK | Successful GET, PUT, PATCH |
| 201 | Created | Successful POST creating a resource |
| 204 | No Content | Successful DELETE, or PUT/PATCH with no body |
| 400 | Bad Request | Malformed request, validation failure |
| 401 | Unauthorized | Missing or invalid authentication |
| 403 | Forbidden | Authenticated but not authorized |
| 404 | Not Found | Resource does not exist |
| 409 | Conflict | Duplicate create, optimistic lock failure |
| 422 | Unprocessable Entity | Semantically invalid (business rule violation) |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Unexpected server failure |
| 503 | Service Unavailable | Temporary outage, maintenance |

### Sub-Resource Patterns

Keep sub-resources to actions that only make sense in context of the parent:

```
# Good: items only exist within an order
GET  /v1/orders/{id}/items
POST /v1/orders/{id}/items

# Prefer flat resources when entities stand alone
GET  /v1/items/{id}       # better than /v1/orders/{oid}/items/{iid}
```

For actions that don't map cleanly to CRUD, use a verb suffix as a last resort:
```
POST /v1/orders/{id}/cancel
POST /v1/orders/{id}/fulfill
POST /v1/users/{id}/verify-email
```

</details>

<details>
<summary><strong>Deep Dive: Response Format</strong></summary>

### Envelope Structure

Use a consistent envelope for all responses:

```json
{
  "data": { ... },
  "meta": {
    "requestId": "req_abc123",
    "timestamp": "2025-10-31T14:30:00Z"
  },
  "links": {
    "self": "/v1/orders/ord_xyz",
    "next": "/v1/orders?cursor=eyJpZCI6MTAwfQ"
  },
  "errors": [ ... ]
}
```

- `data` — the primary payload (object for single, array for collection)
- `meta` — pagination cursors, request IDs, timestamps
- `links` — HATEOAS navigation (self, next, prev, related actions)
- `errors` — only present on error responses (4xx/5xx)

### HATEOAS: Links and Actions

Use the `links` object for both navigation and available actions on the resource. Each key is a [link relation type](https://www.iana.org/assignments/link-relations/link-relations.xhtml):

```json
{
  "data": {
    "id": "ord_xyz",
    "status": "pending",
    "total": 1099
  },
  "links": {
    "self": { "href": "/v1/orders/ord_xyz" },
    "cancel": { "href": "/v1/orders/ord_xyz/cancel", "method": "POST" },
    "items": { "href": "/v1/orders/ord_xyz/items" },
    "payment": { "href": "/v1/orders/ord_xyz/pay", "method": "POST" }
  }
}
```

**Key principles:**
- Actions are state-dependent — `cancel` only appears when the order is cancellable
- Include `method` when the action is not a GET (avoids client guessing)
- Use IANA-registered relation types where they exist (`self`, `next`, `prev`, `collection`)
- Use descriptive custom relation types for domain actions (`cancel`, `approve`, `payment`)
- Do NOT use a separate `actions` array — this is Siren format, not widely adopted. Standard `links` with method hints covers the same use case with less complexity

### Field Naming Convention

| Convention | When | Example |
|------------|------|---------|
| camelCase | JSON APIs (default) | `createdAt`, `lineItems` |
| snake_case | Python/Go backends matching language convention | `created_at`, `line_items` |

Pick one and apply it everywhere. Document it in your OpenAPI spec's `x-naming-convention` extension.

### Null Handling

- Omit optional fields that have no value rather than returning `null` where possible
- When `null` is semantically meaningful (explicitly unset vs absent), include it with documentation
- Never return `null` for arrays — return `[]`

### Date Format

Always ISO 8601 in UTC:
```
"createdAt": "2025-10-31T14:30:00Z"     # correct
"createdAt": "10/31/2025 2:30 PM"       # wrong — ambiguous and locale-dependent
"createdAt": 1730382600                 # avoid — requires client-side conversion
```

### Monetary Values

```json
{
  "amount": 1099,
  "currency": "USD",
  "amountFormatted": "$10.99"
}
```

- Store and transmit as integer cents (or smallest currency unit)
- Include currency code with every monetary field
- Optionally include a pre-formatted display string for convenience

</details>

<details>
<summary><strong>Deep Dive: Versioning Strategies</strong></summary>

### Strategy Comparison

| Strategy | Example | Pros | Cons |
|----------|---------|------|------|
| **URL path** | `/v1/orders` | Visible, cacheable, easy to test | Pollutes URL, not REST-pure |
| **Request header** | `Accept-Version: 2` | Clean URLs, REST-pure | Harder to test, requires header tooling |
| **Content negotiation** | `Accept: application/vnd.api.v2+json` | Standards-compliant | Verbose, limited browser/tool support |
| **Query parameter** | `/orders?version=2` | Easy to test | Cache-unfriendly, easy to forget |

### Recommendation

**Default: URL path versioning.**
- Easiest to understand, document, and test
- Works natively with all HTTP clients and proxies
- Every environment can hit a specific version directly

Use header versioning only when you have multiple clients with very different needs and want a single URL namespace.

### Version Lifecycle

```
v1 (stable)   ──────────────────────────────────► active
v2 (preview)  ────────► stable ─────────────────► active
v1 (sunset)             Sunset header added ──────► deprecated ──► removed
```

Timeline example:
- **v1 launch**: `/v1/` is stable, no deprecation
- **v2 launch**: Announce v2, keep v1 running
- **v1 sunset**: Add `Sunset: Sat, 31 Dec 2026 23:59:59 GMT` header to v1 responses
- **v1 removal**: 6–12 months after Sunset header, remove v1

### Backward-Compatible Changes (No Version Bump Needed)

- Adding optional request fields
- Adding new response fields
- Adding new endpoints
- Adding new enum values (if clients handle unknown values)
- Relaxing validation (e.g., increasing max length)

### Breaking Changes (Require Version Bump)

- Removing or renaming fields
- Changing field types
- Removing endpoints
- Changing HTTP methods
- Tightening validation
- Changing authentication requirements

</details>

<details>
<summary><strong>Deep Dive: Pagination</strong></summary>

### Offset-Based vs Cursor-Based

| Factor | Offset | Cursor |
|--------|--------|--------|
| Implementation | Simple (LIMIT/OFFSET) | Moderate (encode last-seen ID) |
| Performance on large datasets | Degrades (full table scan) | Consistent (index seek) |
| Handles concurrent writes | Skips/duplicates items | Stable position |
| Supports jump to page N | Yes | No |
| Best for | Small datasets, admin UIs | Large/live datasets, feeds |

### Offset Pagination Response

```json
{
  "data": [ ... ],
  "meta": {
    "total": 1842,
    "page": 3,
    "perPage": 20,
    "totalPages": 93
  },
  "links": {
    "self": "/v1/orders?page=3&perPage=20",
    "prev": "/v1/orders?page=2&perPage=20",
    "next": "/v1/orders?page=4&perPage=20",
    "first": "/v1/orders?page=1&perPage=20",
    "last": "/v1/orders?page=93&perPage=20"
  }
}
```

### Cursor Pagination Response

```json
{
  "data": [ ... ],
  "meta": {
    "hasNext": true,
    "hasPrev": true,
    "cursor": {
      "next": "eyJpZCI6MTAwLCJjcmVhdGVkQXQiOiIyMDI1LTEwLTMxIn0=",
      "prev": "eyJpZCI6ODEsImNyZWF0ZWRBdCI6IjIwMjUtMTAtMzAifQ=="
    }
  }
}
```

**Client usage:**
```
GET /v1/orders?limit=20&after=eyJpZCI6MTAwfQ==
```

### Cursor Implementation Pattern

Encode the sort key(s) and last-seen ID in the cursor:
```json
{ "id": 100, "createdAt": "2025-10-31" }
```
Base64-encode this JSON — opaque to the client, but deterministic on the server.

### Pagination Parameters

| Parameter | Offset | Cursor |
|-----------|--------|--------|
| `page` | Page number (1-indexed) | — |
| `perPage` / `limit` | Items per page | Items per page |
| `after` | — | Cursor for next page |
| `before` | — | Cursor for prev page |

Always enforce a maximum `limit` (e.g., 100) server-side regardless of what the client requests.

</details>

<details>
<summary><strong>Deep Dive: Error Handling</strong></summary>

### Error Response Format

Standard error envelope:

```json
{
  "error": {
    "type": "BAD_REQUEST",
    "message": "Invalid email address format",
    "subcode": 12001
  }
}
```

| Field | Description |
|-------|-------------|
| `error.type` | Machine-readable error category (SCREAMING_SNAKE_CASE) |
| `error.message` | Short, human-readable description of the error |
| `error.subcode` | Numeric code for programmatic handling and documentation lookup |

**Validation errors** add a `fields` array:

```json
{
  "error": {
    "type": "VALIDATION_FAILED",
    "message": "One or more fields failed validation",
    "subcode": 10001,
    "fields": [
      { "field": "quantity", "code": "INVALID_RANGE", "message": "Must be between 1 and 999" },
      { "field": "productId", "code": "NOT_FOUND", "message": "Product prod_zzz does not exist" }
    ]
  }
}
```

### Request and Trace IDs

Every response (success and error) must include correlation headers:

```
X-Request-Id: req_abc123              # Server-generated if client omits
X-Trace-Id: trace_def456             # Distributed tracing correlation
Retry-After: 60                       # Seconds until retry (429, 503)
```

- **X-Request-Id:** If the client sends one, echo it back. Otherwise, generate a UUID server-side. Log it with every request.
- **X-Trace-Id:** Propagated through all downstream service calls for distributed tracing.
- **Do NOT put trace IDs in the error body.** Headers are the canonical location — they're present on all responses (not just errors), are accessible to proxies/load balancers, and avoid coupling error schemas to tracing infrastructure. Clients needing to report an error to support should reference the `X-Request-Id` header value.

### Error Code Registry

Define typed error codes as constants in your API:

```
VALIDATION_FAILED       422   One or more fields are invalid           10001
RESOURCE_NOT_FOUND      404   The requested resource does not exist     10002
DUPLICATE_RESOURCE      409   A resource with this identifier exists    10003
INSUFFICIENT_FUNDS      422   Account balance is insufficient           10004
RATE_LIMIT_EXCEEDED     429   Too many requests; retry after X seconds  10005
UNAUTHORIZED            401   Authentication credentials invalid        10006
FORBIDDEN               403   Caller lacks permission for this action   10007
```

Document the full registry in your OpenAPI spec under `x-error-codes`.

### HTTP Status Decision Tree

```
Is the request malformed JSON?          → 400 Bad Request
Do the fields fail validation?          → 422 Unprocessable Entity
Is authentication missing or invalid?   → 401 Unauthorized
Is the user authenticated but denied?   → 403 Forbidden
Does the resource not exist?            → 404 Not Found
Does a conflict exist (duplicate)?      → 409 Conflict
Did a business rule fail?               → 422 Unprocessable Entity
Is the server broken?                   → 500 Internal Server Error
Is the service temporarily unavailable? → 503 Service Unavailable
```

### RFC 9457 Compatibility

The error format above is intentionally simpler than RFC 9457 (Problem Details). If your API must comply with RFC 9457 (e.g., for interoperability with standards-mandating clients), use:

```json
{
  "type": "https://api.example.com/errors/validation-failed",
  "title": "Validation Failed",
  "status": 422,
  "detail": "One or more fields failed validation.",
  "instance": "/v1/orders/ord_xyz"
}
```

Set `Content-Type: application/problem+json` for RFC 9457 responses.

</details>

<details>
<summary><strong>Deep Dive: Authentication and Authorization</strong></summary>

### Auth Mechanism Comparison

| Mechanism | Best For | Complexity | Stateless |
|-----------|----------|------------|-----------|
| API Key | Server-to-server, simple integrations | Low | Yes |
| Bearer JWT | User-facing APIs, short-lived tokens | Medium | Yes |
| OAuth 2.0 + PKCE | User-delegated access, third-party integrations | High | Yes |
| Session cookies | Traditional web apps, SSR | Low | No |

### Bearer Token (JWT) Pattern

```
Authorization: Bearer eyJhbGciOiJSUzI1NiJ9...
```

JWT payload convention:
```json
{
  "sub": "usr_abc123",
  "iss": "https://auth.example.com",
  "aud": "https://api.example.com",
  "exp": 1730386200,
  "iat": 1730382600,
  "scope": "orders:read orders:write"
}
```

### API Key Pattern

```
X-API-Key: sk_live_abc123xyz
```

- Prefix keys with environment (`sk_live_`, `sk_test_`) for instant recognition
- Hash keys at rest; never store plaintext
- Issue per-integration, not per-user
- Support key rotation without downtime (accept both old and new during transition)

### OAuth 2.0 Flows

| Flow | When |
|------|------|
| Authorization Code + PKCE | Web/mobile apps on behalf of a user |
| Client Credentials | Service-to-service (no user context) |
| Device Code | Devices without browsers (CLI tools, smart TVs) |

### Scoping

Define scopes as `{resource}:{action}` pairs:
```
orders:read          Read orders
orders:write         Create and update orders
orders:delete        Delete orders
users:read           Read user profiles
admin:full           All access (restrict heavily)
```

Request minimum scopes. Validate scopes server-side on every request.

### Rate Limiting Headers

Always return rate limit state in responses:
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 847
X-RateLimit-Reset: 1730383200
```

On 429:
```
Retry-After: 47
```

</details>

<details>
<summary><strong>Deep Dive: OpenAPI 3.1 Specification</strong></summary>

### Skeleton Template

```yaml
openapi: 3.1.0
info:
  title: My Service API
  version: 1.0.0
  description: |
    Brief description of the API's purpose and audience.
  contact:
    name: Platform Team
    email: platform@example.com
  license:
    name: MIT

servers:
  - url: https://api.example.com/v1
    description: Production
  - url: https://api-staging.example.com/v1
    description: Staging

security:
  - bearerAuth: []

paths:
  /resources:
    get:
      summary: List resources
      operationId: listResources
      tags: [Resources]
      parameters:
        - $ref: '#/components/parameters/Limit'
        - $ref: '#/components/parameters/After'
      responses:
        '200':
          description: Paginated resource list
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ResourceListResponse'
        '401':
          $ref: '#/components/responses/Unauthorized'

components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
    apiKey:
      type: apiKey
      in: header
      name: X-API-Key

  parameters:
    Limit:
      name: limit
      in: query
      schema:
        type: integer
        minimum: 1
        maximum: 100
        default: 20
    After:
      name: after
      in: query
      schema:
        type: string
        description: Cursor from previous response for next page

  responses:
    Unauthorized:
      description: Authentication credentials missing or invalid
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ProblemDetails'
    NotFound:
      description: Resource not found
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ProblemDetails'

  schemas:
    ProblemDetails:
      type: object
      required: [type, title, status]
      properties:
        type:
          type: string
          format: uri
        title:
          type: string
        status:
          type: integer
        detail:
          type: string
        instance:
          type: string
        requestId:
          type: string
```

### Reusable Schema Patterns

**Envelope response:**
```yaml
ResourceListResponse:
  type: object
  properties:
    data:
      type: array
      items:
        $ref: '#/components/schemas/Resource'
    meta:
      $ref: '#/components/schemas/PaginationMeta'
    links:
      $ref: '#/components/schemas/PaginationLinks'

PaginationMeta:
  type: object
  properties:
    hasNext:
      type: boolean
    hasPrev:
      type: boolean

PaginationLinks:
  type: object
  properties:
    self:
      type: string
      format: uri
    next:
      type: string
      format: uri
    prev:
      type: string
      format: uri
```

### `$ref` Strategy

- All reusable schemas in `components/schemas`
- All reusable parameters in `components/parameters`
- All reusable responses in `components/responses`
- Never inline schema objects in `paths` — always `$ref`

### Validation Tooling

- `spectral` — OpenAPI linting with configurable rulesets
- `redocly` — Spec validation + doc generation
- `swagger-cli validate` — Basic structural validation

</details>

<details>
<summary><strong>Deep Dive: GraphQL Schema Design</strong></summary>

### Type Definitions

```graphql
scalar DateTime
scalar Decimal

type User {
  id: ID!
  email: String!
  createdAt: DateTime!
  orders(first: Int, after: String): OrderConnection!
}

type Order {
  id: ID!
  status: OrderStatus!
  total: Decimal!
  currency: String!
  lineItems: [LineItem!]!
  createdAt: DateTime!
}

type LineItem {
  id: ID!
  product: Product!
  quantity: Int!
  unitPrice: Decimal!
}

enum OrderStatus {
  PENDING
  CONFIRMED
  SHIPPED
  DELIVERED
  CANCELLED
}
```

### Query, Mutation, Subscription Patterns

```graphql
type Query {
  # Single resource by ID
  order(id: ID!): Order

  # Paginated collection (Relay-style)
  orders(first: Int, after: String, filter: OrderFilter): OrderConnection!

  # Current user context
  me: User
}

type Mutation {
  # Returns the created/mutated resource
  createOrder(input: CreateOrderInput!): CreateOrderPayload!
  cancelOrder(id: ID!): CancelOrderPayload!
}

type Subscription {
  # Subscribe to events for a specific resource
  orderStatusChanged(orderId: ID!): Order!
}
```

### Input Types

```graphql
input CreateOrderInput {
  lineItems: [LineItemInput!]!
  shippingAddressId: ID!
  couponCode: String
}

input LineItemInput {
  productId: ID!
  quantity: Int!
}
```

### Relay-Style Pagination (Connection Pattern)

```graphql
type OrderConnection {
  edges: [OrderEdge!]!
  pageInfo: PageInfo!
}

type OrderEdge {
  cursor: String!
  node: Order!
}

type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}
```

### Error Handling: Errors Array vs Union Types

**Errors array (GraphQL default):**
```json
{
  "data": { "createOrder": null },
  "errors": [{ "message": "Product not found", "path": ["createOrder"] }]
}
```

**Union result types (recommended for mutations):**
```graphql
union CreateOrderResult = Order | ValidationError | NotFoundError

type Mutation {
  createOrder(input: CreateOrderInput!): CreateOrderResult!
}
```
This approach makes errors first-class, typed, and queryable by clients.

### Complexity and Depth Limiting

Prevent abuse by enforcing query complexity limits:
- **Max depth**: 7–10 levels (reject deeply nested queries)
- **Max complexity**: Assign cost to each field; reject queries over budget
- **Introspection**: Disable in production for public APIs

Libraries: `graphql-depth-limit`, `graphql-query-complexity`

</details>

<details>
<summary><strong>Deep Dive: Breaking Change Management</strong></summary>

### What Counts as Breaking

| Change | Breaking? | Notes |
|--------|-----------|-------|
| Add optional request field | No | Clients ignore unknown fields |
| Add response field | No | Clients ignore unknown fields |
| Remove request field | Yes | Existing clients may send it |
| Remove response field | Yes | Existing clients may read it |
| Rename field | Yes | Equivalent to remove + add |
| Change field type | Yes | Parsing will fail |
| Tighten validation | Yes | Previously valid requests rejected |
| Loosen validation | No | More permissive |
| Remove endpoint | Yes | 404 where clients expect 200 |
| Change auth requirement | Yes | Auth failures for existing clients |
| Add required request field | Yes | Existing requests missing the field |
| Add new enum value | Usually no* | Depends on client handling unknown values |

*Clients should be designed to handle unknown enum values gracefully.

### Additive-Only Evolution

Design for additive-only changes within a major version:
- Add new optional fields to requests
- Add new fields to responses
- Add new endpoints
- Add new optional query parameters
- Add new enum values (with consumer guidance on handling unknowns)

### Deprecation Process

**Step 1: Mark deprecated in OpenAPI spec**
```yaml
/v1/orders/{id}/cancel:
  post:
    deprecated: true
    description: "Deprecated: Use PATCH /v1/orders/{id} with status=cancelled instead."
```

**Step 2: Add Sunset header to responses**
```
Sunset: Sat, 31 Dec 2026 23:59:59 GMT
Deprecation: true
Link: <https://api.example.com/docs/migrations/v1-to-v2>; rel="deprecation"
```

**Step 3: Notify consumers**
- Email registered API consumers
- Post in developer changelog
- Add in-response warning in `meta.warnings` array

**Step 4: Enforce removal timeline**
- Minimum 6 months from Sunset header to removal for external APIs
- Minimum 2 sprints for internal service-to-service APIs

### Migration Guide Template

```markdown
## Migrating from v1 to v2: Orders

### What changed
- `GET /v1/orders` response: `items` renamed to `lineItems`
- `POST /v1/orders` now requires `shippingAddressId`
- Error format changed from `{"error": "..."}` to `{ error: { type, message, subcode } }`

### Migration steps
1. Update `items` → `lineItems` in your order parsing code
2. Add `shippingAddressId` to order creation requests
3. Update error handling to read `type` and `detail` from responses

### Timeline
- v1 sunset: 2026-12-31
- v1 removal: 2027-03-31
```

</details>

---

## Extension Points

1. **Framework-Specific Patterns:** Express middleware, FastAPI dependency injection, Go `net/http` routing conventions
2. **API Gateway Integration:** Kong, AWS API Gateway, Apigee — rate limiting, auth delegation, routing
3. **Contract Testing:** Pact for consumer-driven contract tests; validate against OpenAPI spec with `dredd`
4. **SDK Generation:** `openapi-generator` for client SDKs from OpenAPI spec; `graphql-code-generator` for TypeScript types
5. **Spec Validation Automation:** `spectral` rulesets in CI, `redocly lint` for doc quality gates
6. **API Mocking:** `prism` for mock servers from OpenAPI spec; enables parallel frontend/backend development
7. **Interactive Documentation:** [Scalar](https://github.com/scalar/scalar) for modern, themeable API reference UI — see [references/scalar-api-docs.md](references/scalar-api-docs.md)
