---
name: api-design-patterns
description: Design robust APIs with RESTful patterns, GraphQL schemas, versioning strategies, and error handling conventions. Supports OpenAPI/Swagger documentation and SDK generation patterns. Triggers on API design, schema definition, endpoint architecture, or developer experience requests.
license: MIT
complexity: intermediate
time_to_learn: 30min
tags:
  - api
  - rest
  - graphql
  - openapi
  - design-patterns
inputs:
  - service-requirements
  - data-models
outputs:
  - api-specification
  - endpoint-design
  - openapi-document
side_effects:
  - creates-files
triggers:
  - user-asks-about-api
  - user-asks-about-rest
  - user-asks-about-graphql
  - file-type:*.openapi.yaml
  - project-has-openapi-spec
complements:
  - backend-implementation-patterns
  - mcp-builder
tier: core
---

# API Design Patterns

Build APIs that developers love to use.

## Design Principles

### The Three Laws of API Design

1. **Predictable**: Consistent patterns throughout
2. **Discoverable**: Self-documenting, intuitive naming
3. **Evolvable**: Can change without breaking clients

### API-First Mindset

```
Design → Document → Mock → Build → Test → Deploy

NOT: Build → Document (maybe) → Hope it works
```

---

## REST Patterns

### Resource Naming

```
# Collection
GET    /users              # List users
POST   /users              # Create user

# Item
GET    /users/{id}         # Get user
PUT    /users/{id}         # Replace user
PATCH  /users/{id}         # Update user
DELETE /users/{id}         # Delete user

# Nested resources
GET    /users/{id}/posts   # User's posts
POST   /users/{id}/posts   # Create post for user

# Actions (when CRUD doesn't fit)
POST   /users/{id}/activate
POST   /orders/{id}/cancel
```

### Naming Conventions

| Do | Don't |
|----|-------|
| `/users` | `/getUsers`, `/user-list` |
| `/users/{id}` | `/users/get/{id}` |
| `/users/{id}/posts` | `/getUserPosts` |
| Plural nouns | Verbs in URLs |
| Lowercase, hyphens | camelCase, underscores |

### HTTP Methods

| Method | Purpose | Idempotent | Safe |
|--------|---------|------------|------|
| GET | Read | Yes | Yes |
| POST | Create | No | No |
| PUT | Replace | Yes | No |
| PATCH | Update | No* | No |
| DELETE | Remove | Yes | No |

*PATCH can be idempotent if designed carefully

### Status Codes

| Code | Meaning | Use When |
|------|---------|----------|
| 200 | OK | Successful GET, PUT, PATCH |
| 201 | Created | Successful POST with new resource |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Invalid input |
| 401 | Unauthorized | Missing/invalid auth |
| 403 | Forbidden | Valid auth, no permission |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Duplicate, state conflict |
| 422 | Unprocessable | Valid syntax, invalid semantics |
| 429 | Too Many Requests | Rate limited |
| 500 | Server Error | Unhandled server error |

---

## Request/Response Patterns

### Request Body

```json
{
  "email": "user@example.com",
  "name": "John Doe",
  "preferences": {
    "newsletter": true
  }
}
```

### Response: Single Resource

```json
{
  "data": {
    "id": "usr_123",
    "type": "user",
    "attributes": {
      "email": "user@example.com",
      "name": "John Doe",
      "createdAt": "2024-01-15T10:30:00Z"
    },
    "relationships": {
      "organization": {
        "id": "org_456",
        "type": "organization"
      }
    }
  }
}
```

### Response: Collection

```json
{
  "data": [
    { "id": "usr_123", "type": "user", ... },
    { "id": "usr_124", "type": "user", ... }
  ],
  "meta": {
    "total": 150,
    "page": 1,
    "perPage": 20
  },
  "links": {
    "self": "/users?page=1",
    "next": "/users?page=2",
    "last": "/users?page=8"
  }
}
```

### Error Response

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid request parameters",
    "details": [
      {
        "field": "email",
        "code": "INVALID_FORMAT",
        "message": "Must be a valid email address"
      }
    ],
    "requestId": "req_abc123"
  }
}
```

---

## Pagination Patterns

### Offset-Based

```
GET /users?page=2&perPage=20
GET /users?offset=20&limit=20
```

Pros: Simple, familiar
Cons: Inconsistent with real-time data, slow on large datasets

### Cursor-Based

```
GET /users?cursor=eyJpZCI6MTIzfQ&limit=20
```

Response:
```json
{
  "data": [...],
  "cursors": {
    "before": "eyJpZCI6MTAzfQ",
    "after": "eyJpZCI6MTIzfQ"
  },
  "hasMore": true
}
```

Pros: Consistent, performant
Cons: Can't jump to page N

### Keyset-Based

```
GET /users?after_id=123&limit=20
```

Pros: Very performant
Cons: Requires sortable unique field

---

## Filtering, Sorting, Search

### Filtering

```
# Simple
GET /users?status=active

# Multiple values
GET /users?status=active,pending

# Operators
GET /users?created_at[gte]=2024-01-01
GET /users?name[contains]=john

# Nested
GET /users?organization.name=Acme
```

### Sorting

```
# Single field
GET /users?sort=createdAt

# Descending
GET /users?sort=-createdAt

# Multiple fields
GET /users?sort=-createdAt,name
```

### Field Selection

```
GET /users?fields=id,name,email
GET /users?fields[user]=id,name&fields[posts]=title
```

### Search

```
GET /users?q=john
GET /users?search=john+doe
```

---

## Versioning Strategies

### URL Path (Recommended for major versions)

```
GET /v1/users
GET /v2/users
```

Pros: Explicit, cacheable
Cons: URL pollution

### Header

```
GET /users
Accept: application/vnd.api+json; version=2
```

Pros: Clean URLs
Cons: Harder to test, less visible

### Query Parameter

```
GET /users?version=2
```

Pros: Easy to test
Cons: Breaks caching, URL pollution

### Deprecation Pattern

```
Deprecation: true
Sunset: Sat, 31 Dec 2024 23:59:59 GMT
Link: </v2/users>; rel="successor-version"
```

---

## Authentication Patterns

### API Keys

```
# Header
Authorization: Api-Key sk_live_abc123

# Query (avoid - logged in URLs)
GET /users?api_key=sk_live_abc123 <!-- allow-secret -->
```

### Bearer Tokens (OAuth2/JWT)

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

### Signing Requests (AWS-style)

```
Authorization: HMAC-SHA256 Credential=key/date/region/service,
               SignedHeaders=host;x-date,
               Signature=abc123
```

---

## Rate Limiting

### Headers

```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1640000000
Retry-After: 60
```

### Response (429)

```json
{
  "error": {
    "code": "RATE_LIMITED",
    "message": "Rate limit exceeded",
    "retryAfter": 60
  }
}
```

### Strategies

| Strategy | Description | Use Case |
|----------|-------------|----------|
| Fixed Window | X requests per minute | Simple limiting |
| Sliding Window | Rolling time window | Smoother limiting |
| Token Bucket | Burst allowance | Traffic spikes |
| Leaky Bucket | Constant rate | Steady throughput |

---

## GraphQL Patterns

### Schema Design

```graphql
type User {
  id: ID!
  email: String!
  name: String
  posts(first: Int, after: String): PostConnection!
  createdAt: DateTime!
}

type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
}

type PostConnection {
  edges: [PostEdge!]!
  pageInfo: PageInfo!
}

type Query {
  user(id: ID!): User
  users(filter: UserFilter, first: Int, after: String): UserConnection!
}

type Mutation {
  createUser(input: CreateUserInput!): CreateUserPayload!
  updateUser(id: ID!, input: UpdateUserInput!): UpdateUserPayload!
}
```

### Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Types | PascalCase | `User`, `BlogPost` |
| Fields | camelCase | `firstName`, `createdAt` |
| Arguments | camelCase | `userId`, `first` |
| Enums | SCREAMING_SNAKE | `USER_STATUS`, `ACTIVE` |
| Mutations | verbNoun | `createUser`, `updatePost` |

### Error Handling

```json
{
  "data": null,
  "errors": [
    {
      "message": "User not found",
      "locations": [{ "line": 2, "column": 3 }],
      "path": ["user"],
      "extensions": {
        "code": "NOT_FOUND",
        "timestamp": "2024-01-15T10:30:00Z"
      }
    }
  ]
}
```

---

## OpenAPI Documentation

### Basic Structure

```yaml
openapi: 3.0.3
info:
  title: My API
  version: 1.0.0
  description: API description

servers:
  - url: https://api.example.com/v1
    description: Production

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
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserList'

components:
  schemas:
    User:
      type: object
      required: [id, email]
      properties:
        id:
          type: string
          format: uuid
        email:
          type: string
          format: email
```

---

---

## Related Skills

### Complementary Skills (Use Together)
- **[backend-implementation-patterns](../backend-implementation-patterns/)** - Implement the APIs you design
- **[testing-patterns](../testing-patterns/)** - Test your API endpoints
- **[deployment-cicd](../deployment-cicd/)** - Deploy and version your APIs

### Alternative Skills (Similar Purpose)
- **[mcp-builder](../mcp-builder/)** - If building MCP servers instead of REST/GraphQL APIs

### Prerequisite Skills (Learn First)
- None required - this is a foundational design skill

---

## References

- `references/openapi-template.md` - Full OpenAPI template
- `references/error-codes.md` - Standard error code catalog
- `references/sdk-patterns.md` - Client SDK design patterns
