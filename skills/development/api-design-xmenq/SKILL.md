---
name: API Design
description: Guidelines for designing REST and GraphQL APIs — endpoints, contracts, versioning, error handling
---

# API Design Skill

## When to use this skill

Use when designing new API endpoints, modifying existing APIs, defining request/response contracts, or implementing API error handling.

---

## API Design Principles

### 1. Contract first
- Define the API contract (types/schemas) before implementing
- The contract is the source of truth — code implements the contract
- Breaking contract changes require versioning

### 2. Predictability over cleverness
- Follow established conventions consistently
- A developer should guess the endpoint URL correctly on the first try
- Consistent naming, consistent response shapes, consistent error formats

### 3. Design for the consumer
- Think about who will call this API (frontend, mobile, third-party)
- Return what consumers need, not what the database stores
- Minimize the number of calls needed for common operations

---

## REST API Conventions

### URL structure
```
[verb] /api/v{version}/{resource}/{id?}/{sub-resource?}
```

| Method | URL | Purpose |
|--------|-----|---------|
| `GET` | `/api/v1/users` | List users (paginated) |
| `GET` | `/api/v1/users/:id` | Get single user |
| `POST` | `/api/v1/users` | Create user |
| `PUT` | `/api/v1/users/:id` | Full update user |
| `PATCH` | `/api/v1/users/:id` | Partial update user |
| `DELETE` | `/api/v1/users/:id` | Delete user |
| `GET` | `/api/v1/users/:id/orders` | List user's orders |

### Naming rules
- **Resources are nouns** — `/users`, not `/getUsers`
- **Plural names** — `/users`, not `/user`
- **Kebab-case** for multi-word resources — `/order-items`, not `/orderItems`
- **No verbs in URLs** (usually) — use HTTP methods instead
- **Exception:** Actions that don't map to CRUD — `POST /api/v1/orders/:id/cancel`

---

## Request & Response Format

### Standard response envelope
```json
{
  "data": { ... },           // The actual response data
  "meta": {                  // Metadata (pagination, etc.)
    "page": 1,
    "perPage": 20,
    "total": 142
  }
}
```

### Standard error response
```json
{
  "error": {
    "code": "VALIDATION_ERROR",    // Machine-readable error code
    "message": "Email is required", // Human-readable message
    "details": [                    // Field-level errors (optional)
      {
        "field": "email",
        "message": "Email is required"
      }
    ]
  }
}
```

### HTTP Status Codes

| Code | When to use |
|------|------------|
| `200` | Successful GET, PUT, PATCH |
| `201` | Successful POST (resource created) |
| `204` | Successful DELETE (no content to return) |
| `400` | Bad request — validation failed, malformed input |
| `401` | Unauthorized — not authenticated |
| `403` | Forbidden — authenticated but not allowed |
| `404` | Not found — resource doesn't exist |
| `409` | Conflict — resource state conflict (e.g., duplicate) |
| `422` | Unprocessable — valid format but semantic error |
| `429` | Too many requests — rate limited |
| `500` | Internal server error — something unexpected broke |

---

## Pagination

### Cursor-based (preferred for large/changing datasets)
```
GET /api/v1/orders?cursor=abc123&limit=20
→ { "data": [...], "meta": { "nextCursor": "def456", "hasMore": true } }
```

### Offset-based (simpler, fine for small datasets)
```
GET /api/v1/orders?page=2&perPage=20
→ { "data": [...], "meta": { "page": 2, "perPage": 20, "total": 142 } }
```

---

## Filtering, Sorting & Search

### Filtering
```
GET /api/v1/orders?status=pending&userId=123
GET /api/v1/orders?createdAfter=2025-01-01
```

### Sorting
```
GET /api/v1/orders?sort=createdAt&order=desc
GET /api/v1/orders?sort=-createdAt  (prefix with - for descending)
```

### Search
```
GET /api/v1/users?search=alice
```

---

## Versioning

- **Use URL versioning** — `/api/v1/`, `/api/v2/`
- **Version when you make breaking changes** — field removal, type changes, behavior changes
- **Non-breaking changes don't need versioning** — adding optional fields, new endpoints
- **Support old versions** for a defined deprecation period
- **Document deprecations clearly** with sunset dates

---

## Authentication & Authorization

- **Use standard auth headers** — `Authorization: Bearer <token>`
- **Validate auth on every request** — middleware/filter, not per-endpoint
- **Return 401 for missing/invalid auth** — don't leak whether resource exists
- **Return 403 for insufficient permissions** — authenticated but not allowed
- **Rate limit by API key/user** — prevent abuse

---

## API Documentation

Every endpoint should be documented with:
- **URL and method**
- **Request parameters** (path, query, body) with types and required/optional
- **Response shape** with example
- **Error cases** and their response codes
- **Authentication requirements**

Use OpenAPI/Swagger for auto-generated docs when possible.

---

## PR Checklist for API Changes

- [ ] Follows REST conventions (nouns, plural, correct HTTP methods)
- [ ] Request/response types defined in `types/`
- [ ] Input validation at the boundary
- [ ] Consistent error response format
- [ ] Pagination for list endpoints
- [ ] Auth/authz checked
- [ ] Status codes match semantics
- [ ] No breaking changes (or version bumped)
- [ ] API documentation updated
- [ ] Integration tests for new endpoints
