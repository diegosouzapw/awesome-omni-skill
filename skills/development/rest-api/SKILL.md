---
name: rest-api
description: Expert guidance for designing and building RESTful APIs following industry standards. Use when creating APIs, reviewing designs, implementing endpoints, or troubleshooting API issues.
license: MIT
metadata:
  author: Tresic Engineering
  version: "1.0.0"
---

# REST API Standards

## Overview

This skill provides expert guidance for designing, building, and maintaining RESTful APIs that follow industry standards and best practices. It covers resource design, HTTP semantics, error handling, pagination, versioning, security, and documentation.

## When to Use This Skill

- Designing new API endpoints or services
- Reviewing existing API designs for standards compliance
- Implementing CRUD operations for resources
- Adding pagination, filtering, or sorting to endpoints
- Designing error responses and status codes
- Creating OpenAPI/Swagger documentation
- Implementing authentication and authorization patterns
- Versioning APIs for backward compatibility

---

## Core Principles

### 1. Resource-Oriented Design

APIs expose **resources** (nouns), not actions (verbs). Resources are entities that can be identified, named, addressed, and handled.

```
✅ GET  /users/123
✅ POST /orders
✅ PUT  /products/456

❌ GET  /getUser?id=123
❌ POST /createOrder
❌ POST /updateProduct
```

### 2. Use HTTP Methods Correctly

| Method | Purpose | Idempotent | Safe | Request Body | Idempotency-Key |
|--------|---------|------------|------|--------------|-----------------|
| GET | Retrieve resource(s) | Yes | Yes | No | Not needed |
| POST | Create new resource | No* | No | Yes | **Required** |
| PUT | Replace entire resource | Yes | No | Yes | Not needed |
| PATCH | Partial update | No* | No | Yes | **Required** |
| DELETE | Remove resource | Yes | No | No | Not needed |

*POST and PATCH become idempotent when using idempotency keys (see [Idempotency](#idempotency) section).

### 3. Use Proper HTTP Status Codes

**Success (2xx)**
| Code | Meaning | When to Use |
|------|---------|-------------|
| 200 | OK | Successful GET, PUT, PATCH, or DELETE |
| 201 | Created | Successful POST that creates a resource |
| 202 | Accepted | Request accepted for async processing |
| 204 | No Content | Successful DELETE with no response body |

**Client Errors (4xx)**
| Code | Meaning | When to Use |
|------|---------|-------------|
| 400 | Bad Request | Malformed syntax, invalid parameters |
| 401 | Unauthorized | Missing or invalid authentication |
| 403 | Forbidden | Authenticated but not authorized |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Resource state conflict (e.g., duplicate) |
| 422 | Unprocessable Entity | Validation errors |
| 429 | Too Many Requests | Rate limit exceeded |

**Server Errors (5xx)**
| Code | Meaning | When to Use |
|------|---------|-------------|
| 500 | Internal Server Error | Unexpected server error |
| 502 | Bad Gateway | Upstream service failure |
| 503 | Service Unavailable | Temporary overload or maintenance |
| 504 | Gateway Timeout | Upstream service timeout |

---

## URL Design

### Resource Naming Conventions

```
# Use plural nouns for collections
GET /users
GET /orders
GET /products

# Use identifiers for specific resources
GET /users/{userId}
GET /orders/{orderId}

# Use nesting for relationships (max 2 levels)
GET /users/{userId}/orders
GET /orders/{orderId}/items

# Avoid deep nesting - use query parameters instead
❌ GET /users/{userId}/orders/{orderId}/items/{itemId}/reviews
✅ GET /reviews?orderId={orderId}&itemId={itemId}
```

### Naming Rules

- Use lowercase letters
- Use hyphens for multi-word resources: `/user-profiles`
- Never use verbs in URLs
- Never use file extensions: `/users.json`
- Use consistent plural/singular (prefer plural for collections)

### Query Parameters

```
# Filtering
GET /users?status=active&role=admin

# Sorting (prefix with - for descending)
GET /users?sort=created_at
GET /users?sort=-updated_at,name

# Pagination
GET /users?page=2&per_page=25
GET /users?cursor=eyJpZCI6MTIzfQ&limit=25

# Field selection (sparse fieldsets)
GET /users?fields=id,name,email

# Search
GET /users?q=john
GET /products?search=laptop
```

---

## Request & Response Design

### Request Bodies

```json
// POST /users - Create
{
  "name": "John Doe",
  "email": "john@example.com",
  "role": "admin"
}

// PUT /users/123 - Full replacement
{
  "id": "123",
  "name": "John Doe",
  "email": "john.doe@example.com",
  "role": "admin",
  "status": "active"
}

// PATCH /users/123 - Partial update
{
  "email": "newemail@example.com"
}
```

### Response Structure

**Single Resource**
```json
{
  "data": {
    "id": "123",
    "type": "user",
    "attributes": {
      "name": "John Doe",
      "email": "john@example.com",
      "created_at": "2025-01-15T10:30:00Z"
    },
    "links": {
      "self": "/users/123",
      "orders": "/users/123/orders"
    }
  }
}
```

**Collection Response**
```json
{
  "data": [
    { "id": "1", "type": "user", "attributes": { ... } },
    { "id": "2", "type": "user", "attributes": { ... } }
  ],
  "meta": {
    "total_count": 150,
    "page": 1,
    "per_page": 25,
    "total_pages": 6
  },
  "links": {
    "self": "/users?page=1",
    "first": "/users?page=1",
    "prev": null,
    "next": "/users?page=2",
    "last": "/users?page=6"
  }
}
```

---

## Error Handling

### Standard Error Response Format

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "The request contains invalid parameters",
    "details": [
      {
        "field": "email",
        "code": "INVALID_FORMAT",
        "message": "Email must be a valid email address"
      },
      {
        "field": "age",
        "code": "OUT_OF_RANGE",
        "message": "Age must be between 18 and 120"
      }
    ],
    "request_id": "req_abc123xyz",
    "documentation_url": "https://api.example.com/docs/errors#VALIDATION_ERROR"
  }
}
```

### Error Code Categories

```
# Authentication/Authorization
UNAUTHORIZED          - Missing or invalid credentials
FORBIDDEN             - Valid credentials but insufficient permissions
TOKEN_EXPIRED         - JWT or session has expired
INVALID_API_KEY       - API key is malformed or revoked

# Validation
VALIDATION_ERROR      - One or more fields failed validation
MISSING_REQUIRED_FIELD - Required field not provided
INVALID_FORMAT        - Field format is incorrect
OUT_OF_RANGE          - Value outside acceptable range

# Resource Errors
NOT_FOUND             - Resource doesn't exist
ALREADY_EXISTS        - Unique constraint violation
CONFLICT              - State conflict (optimistic locking)
GONE                  - Resource permanently deleted

# Pagination Errors
INVALID_CURSOR        - Cursor is malformed or expired
CURSOR_EXPIRED        - Cursor has expired, restart from beginning

# Idempotency Errors
IDEMPOTENCY_KEY_REQUIRED - POST/PATCH request missing Idempotency-Key header
IDEMPOTENCY_CONFLICT     - Same key used with different request body
IDEMPOTENCY_IN_PROGRESS  - Request with this key is still processing

# Rate Limiting
RATE_LIMIT_EXCEEDED   - Too many requests

# Server Errors
INTERNAL_ERROR        - Unexpected server error
SERVICE_UNAVAILABLE   - Dependent service is down
```

---

## Pagination

### Cursor-Based Pagination (Required)

All list endpoints MUST use cursor-based pagination. This is the only approved pagination pattern.

**Why cursor-based pagination:**
- **Performance**: O(1) vs O(n) for offset-based on large datasets
- **Consistency**: No skipped/duplicate records when data changes
- **Scalability**: Works efficiently with millions of records
- **Real-time friendly**: Handles insertions/deletions gracefully

```
GET /users?cursor=eyJpZCI6MTAwfQ&limit=25

Response:
{
  "data": [
    { "id": "101", "type": "user", "attributes": { ... } },
    { "id": "102", "type": "user", "attributes": { ... } },
    ...
  ],
  "meta": {
    "has_more": true,
    "next_cursor": "eyJpZCI6MTI1fQ",
    "limit": 25
  },
  "links": {
    "self": "/users?cursor=eyJpZCI6MTAwfQ&limit=25",
    "next": "/users?cursor=eyJpZCI6MTI1fQ&limit=25"
  }
}
```

### Cursor Implementation

**Cursor structure** (base64-encoded JSON):
```json
{
  "id": "last-record-id",
  "sort_field": "sort-value",
  "direction": "next"
}
```

**Query parameters:**
| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `cursor` | No | null | Opaque cursor from previous response |
| `limit` | No | 25 | Number of records (max: 100) |

**First page request** (no cursor):
```
GET /users?limit=25
```

**Subsequent requests** (with cursor):
```
GET /users?cursor=eyJpZCI6IjEyNSIsImRpcmVjdGlvbiI6Im5leHQifQ&limit=25
```

### Response Structure

```json
{
  "data": [...],
  "meta": {
    "has_more": true,
    "next_cursor": "eyJpZCI6MTI1fQ",
    "prev_cursor": "eyJpZCI6MTAxfQ",
    "limit": 25
  },
  "links": {
    "self": "/users?cursor=eyJpZCI6MTAwfQ&limit=25",
    "next": "/users?cursor=eyJpZCI6MTI1fQ&limit=25",
    "prev": "/users?cursor=eyJpZCI6MTAxLCJkaXJlY3Rpb24iOiJwcmV2In0&limit=25"
  }
}
```

### Last Page Response

When no more records exist:
```json
{
  "data": [...],
  "meta": {
    "has_more": false,
    "next_cursor": null,
    "limit": 25
  },
  "links": {
    "self": "/users?cursor=eyJpZCI6MjQwfQ&limit=25",
    "next": null
  }
}
```

### Cursor Rules

1. **Opaque cursors**: Clients must treat cursors as opaque strings—never parse or construct them
2. **Stateless**: Cursors encode position, not server state
3. **Short-lived**: Cursors may expire; clients should handle `400 INVALID_CURSOR` gracefully
4. **Stable sorting**: Always include a unique field (e.g., `id`) as tie-breaker in sort

### Sorting with Cursors

```
GET /users?sort=-created_at,id&limit=25
GET /users?cursor=eyJjcmVhdGVkX2F0IjoiMjAyNS0wMS0xNSIsImlkIjoiMTI1In0&sort=-created_at,id&limit=25
```

The cursor must encode all sort field values to maintain position.

### ❌ Offset-Based Pagination is NOT Allowed

Do not use offset/page-based pagination:
```
❌ GET /users?page=3&per_page=25
❌ GET /users?offset=50&limit=25
```

**Problems with offset pagination:**
- Performance degrades with large offsets (database must scan all skipped rows)
- Inconsistent results when data is inserted/deleted between requests
- "Page drift" causes duplicate or missing records
- Does not scale for large datasets

---

## Versioning

### URL Path Versioning (Recommended)

```
GET /v1/users
GET /v2/users
```

**Pros:** Explicit, easy to route, cacheable
**Cons:** Not truly RESTful (URL should identify resource)

### Header Versioning

```
GET /users
Accept: application/vnd.api+json; version=2
```

**Pros:** Clean URLs, truly RESTful
**Cons:** Harder to test, less visible

### Query Parameter Versioning

```
GET /users?version=2
```

**Pros:** Easy to implement
**Cons:** Optional parameter, caching issues

### Versioning Strategy

1. Start with `/v1/` from day one
2. Maintain backward compatibility within major versions
3. Deprecate old versions with `Sunset` header
4. Provide migration guides for breaking changes

```
Sunset: Sat, 01 Jun 2026 00:00:00 GMT
Deprecation: true
Link: <https://api.example.com/docs/migration/v1-to-v2>; rel="deprecation"
```

---

## HATEOAS (Hypermedia)

Include links to related resources and available actions:

```json
{
  "data": {
    "id": "order-123",
    "type": "order",
    "attributes": {
      "status": "pending",
      "total": 99.99
    },
    "links": {
      "self": "/orders/order-123",
      "customer": "/customers/cust-456",
      "items": "/orders/order-123/items",
      "payment": "/orders/order-123/payment"
    },
    "actions": {
      "cancel": {
        "href": "/orders/order-123/cancel",
        "method": "POST",
        "title": "Cancel this order"
      },
      "pay": {
        "href": "/orders/order-123/payment",
        "method": "POST",
        "title": "Submit payment"
      }
    }
  }
}
```

---

## Authentication & Security

### Idempotency

Idempotency ensures that multiple identical requests produce the same result as a single request. This is critical for handling network failures, retries, and duplicate submissions.

#### HTTP Method Idempotency

| Method | Idempotent | Safe | Notes |
|--------|------------|------|-------|
| GET | ✅ Yes | ✅ Yes | Always idempotent by definition |
| HEAD | ✅ Yes | ✅ Yes | Always idempotent by definition |
| OPTIONS | ✅ Yes | ✅ Yes | Always idempotent by definition |
| PUT | ✅ Yes | ❌ No | Replaces entire resource; same input = same state |
| DELETE | ✅ Yes | ❌ No | Deleting twice = same result (resource gone) |
| POST | ❌ No | ❌ No | **Requires idempotency key** |
| PATCH | ❌ No | ❌ No | **Requires idempotency key** |

#### Idempotency Keys (Required for POST/PATCH)

All `POST` and `PATCH` requests MUST support idempotency keys to enable safe retries.

**Request:**
```
POST /orders
Idempotency-Key: ord_req_abc123xyz789
Content-Type: application/json

{
  "product_id": "prod_123",
  "quantity": 2
}
```

**Response (first request):**
```
HTTP/1.1 201 Created
Idempotency-Key: ord_req_abc123xyz789
X-Idempotent-Replayed: false

{
  "data": {
    "id": "order_456",
    "status": "created"
  }
}
```

**Response (duplicate request with same key):**
```
HTTP/1.1 201 Created
Idempotency-Key: ord_req_abc123xyz789
X-Idempotent-Replayed: true

{
  "data": {
    "id": "order_456",
    "status": "created"
  }
}
```

#### Idempotency Key Rules

1. **Client-generated**: Clients generate unique keys (UUID v4 recommended)
2. **Scoped**: Keys are scoped to endpoint + API key/user
3. **Immutable request**: Same key with different body = `422 IDEMPOTENCY_CONFLICT`
4. **Expiration**: Keys expire after 24 hours (configurable)
5. **Storage**: Store key → response mapping in Redis or database

#### Idempotency Key Format

```
# Recommended formats
Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000  # UUID v4
Idempotency-Key: req_abc123xyz789                       # Prefixed random
Idempotency-Key: order_user123_1705312200              # Semantic (user + timestamp)
```

#### Implementation Pattern

```
1. Receive request with Idempotency-Key header
2. Check if key exists in idempotency store
   - If exists AND request body matches → return stored response
   - If exists AND request body differs → return 422 IDEMPOTENCY_CONFLICT
   - If not exists → continue processing
3. Process request
4. Store key → response mapping with TTL (24h)
5. Return response with X-Idempotent-Replayed: false
```

#### Idempotency Error Responses

**Missing idempotency key (when required):**
```json
{
  "error": {
    "code": "IDEMPOTENCY_KEY_REQUIRED",
    "message": "POST requests require an Idempotency-Key header",
    "documentation_url": "https://api.example.com/docs/idempotency"
  }
}
```

**Conflicting request body:**
```json
{
  "error": {
    "code": "IDEMPOTENCY_CONFLICT",
    "message": "Idempotency key has already been used with a different request body",
    "idempotency_key": "ord_req_abc123xyz789"
  }
}
```

**Concurrent request in progress:**
```json
{
  "error": {
    "code": "IDEMPOTENCY_IN_PROGRESS",
    "message": "A request with this idempotency key is currently being processed",
    "retry_after": 2
  }
}
```

#### Idempotency for Webhooks

When sending webhooks, include idempotency information so receivers can deduplicate:

```json
{
  "id": "evt_abc123",
  "idempotency_key": "evt_abc123",
  "type": "order.created",
  "created_at": "2025-01-15T10:30:00Z",
  "data": { ... }
}
```

### Authentication Methods

| Method | Use Case | Security Level |
|--------|----------|----------------|
| API Keys | Server-to-server, simple integrations | Medium |
| OAuth 2.0 | User authorization, third-party access | High |
| JWT | Stateless authentication | High |
| Basic Auth | Development/testing only | Low |

### Security Headers

```
# Always include
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Content-Security-Policy: default-src 'none'

# Idempotency (for POST/PATCH responses)
Idempotency-Key: ord_req_abc123xyz789
X-Idempotent-Replayed: true|false

# Rate limiting info
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 998
X-RateLimit-Reset: 1640000000
Retry-After: 60
```

### Request ID Tracing

Always generate and return a request ID:

```
Request:
X-Request-ID: client-generated-id (optional)

Response:
X-Request-ID: req_abc123xyz789
```

---

## OpenAPI Documentation

Every API must have OpenAPI 3.1 documentation:

```yaml
openapi: 3.1.0
info:
  title: User Service API
  version: 1.0.0
  description: API for managing users
  
servers:
  - url: https://api.example.com/v1
    description: Production
  - url: https://api.staging.example.com/v1
    description: Staging

paths:
  /users:
    get:
      summary: List users
      operationId: listUsers
      tags: [Users]
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
        - name: per_page
          in: query
          schema:
            type: integer
            default: 25
            maximum: 100
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserList'
        '401':
          $ref: '#/components/responses/Unauthorized'
          
    post:
      summary: Create user
      operationId: createUser
      tags: [Users]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateUserRequest'
      responses:
        '201':
          description: User created
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '422':
          $ref: '#/components/responses/ValidationError'

components:
  schemas:
    User:
      type: object
      required: [id, email, name]
      properties:
        id:
          type: string
          format: uuid
        email:
          type: string
          format: email
        name:
          type: string
        created_at:
          type: string
          format: date-time
          
  responses:
    Unauthorized:
      description: Authentication required
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
            
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

security:
  - bearerAuth: []
```

---

## Implementation Checklist

### Before Development
- [ ] Define resource models and relationships
- [ ] Design URL structure following naming conventions
- [ ] Document endpoints in OpenAPI format
- [ ] Define error codes and response formats
- [ ] Plan versioning strategy
- [ ] Plan idempotency key storage (Redis/database)
- [ ] Determine authentication method

### During Development
- [ ] Use correct HTTP methods and status codes
- [ ] Implement consistent error handling
- [ ] Add cursor-based pagination for all list endpoints
- [ ] Implement idempotency keys for POST/PATCH endpoints
- [ ] Include HATEOAS links where appropriate
- [ ] Add request ID tracing
- [ ] Implement rate limiting
- [ ] Add security headers

### Before Release
- [ ] OpenAPI spec is complete and accurate
- [ ] All endpoints have integration tests
- [ ] Error responses are consistent
- [ ] Idempotency keys tested for POST/PATCH endpoints
- [ ] Rate limits are documented
- [ ] Authentication is working
- [ ] CORS is configured correctly
- [ ] Deprecation strategy is defined

---

## Anti-Patterns to Avoid

```
❌ Using verbs in URLs
   POST /createUser → POST /users

❌ Using wrong HTTP methods
   POST /users/123/delete → DELETE /users/123

❌ Inconsistent naming
   /users, /order, /ProductCategories → /users, /orders, /product-categories

❌ Returning 200 for errors
   { "status": 200, "error": "Not found" } → HTTP 404 with error body

❌ Exposing internal IDs
   /users/1 → /users/usr_abc123

❌ Deeply nested URLs
   /users/1/orders/2/items/3/reviews/4 → /reviews/4?item_id=3

❌ Breaking changes without versioning
   Removing fields → Deprecate, then remove in next major version

❌ Inconsistent date formats
   "2025-01-15", "01/15/2025", 1705312200 → Always use ISO 8601

❌ Using offset-based pagination
   GET /users?page=3&per_page=25 → GET /users?cursor=xyz&limit=25
   Offset pagination causes performance issues and data inconsistency

❌ Exposing total_count with cursor pagination
   total_count requires full table scan → Use has_more boolean instead

❌ POST/PATCH without idempotency support
   Creates duplicate resources on retry → Require Idempotency-Key header

❌ Server-generated idempotency keys
   Client can't safely retry → Client must generate and send the key
```

---

## Related Resources

### Tresic Documentation
- [Intelligence Cloud API Specification](https://tresic.atlassian.net/wiki/spaces/PM/pages/25821465)
- [OpenAPI 3.1 Specification](https://tresic.atlassian.net/wiki/spaces/PM/pages/26083364)
- [Coding Standards](https://tresic.atlassian.net/wiki/spaces/PM/pages/753885)
- [Definition of Done](https://tresic.atlassian.net/wiki/spaces/PM/pages/1048577)

### External References
- [Microsoft REST API Guidelines](https://github.com/microsoft/api-guidelines)
- [Google API Design Guide](https://cloud.google.com/apis/design)
- [JSON:API Specification](https://jsonapi.org/)
- [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
