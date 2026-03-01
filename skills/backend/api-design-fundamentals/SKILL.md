---
name: api-design-fundamentals
description: Use when designing APIs, choosing between REST/GraphQL/gRPC, or understanding API design best practices. Covers protocol selection, resource modeling, and API patterns.
allowed-tools: Read, Glob, Grep
---

# API Design Fundamentals

Guidance for designing effective APIs including protocol selection, resource modeling, and best practices.

## When to Use This Skill

- Choosing between REST, GraphQL, and gRPC
- Designing resource models and endpoints
- Understanding API design best practices
- Creating consistent API conventions
- Designing for developer experience

## Protocol Comparison

### REST (Representational State Transfer)

**Best for:** CRUD operations, public APIs, broad client compatibility

```text
Characteristics:
- Resource-oriented (nouns, not verbs)
- HTTP methods map to operations (GET, POST, PUT, DELETE)
- Stateless
- Cacheable responses
- Self-descriptive messages

Example:
GET    /users          - List users
GET    /users/{id}     - Get user
POST   /users          - Create user
PUT    /users/{id}     - Update user
DELETE /users/{id}     - Delete user
```

**Strengths:**

- Simple, widely understood
- Excellent caching support
- Works with any HTTP client
- Good for public APIs

**Weaknesses:**

- Over-fetching (get more data than needed)
- Under-fetching (multiple requests needed)
- No built-in schema/types

### GraphQL

**Best for:** Complex data requirements, mobile apps, aggregating multiple services

```text
Characteristics:
- Single endpoint
- Client specifies exact data needed
- Strongly typed schema
- Introspection support
- Real-time with subscriptions

Example:
query {
  user(id: "123") {
    name
    email
    posts(limit: 5) {
      title
      comments { count }
    }
  }
}
```

**Strengths:**

- No over/under-fetching
- Strong typing and schema
- Excellent developer tooling
- Version-free evolution

**Weaknesses:**

- Caching complexity
- N+1 query problems
- Learning curve
- Not ideal for simple APIs

### gRPC

**Best for:** Internal microservices, high-performance, polyglot systems

```text
Characteristics:
- Protocol Buffers (binary format)
- HTTP/2 transport
- Bi-directional streaming
- Code generation
- Strong typing

Example (proto):
service UserService {
  rpc GetUser(GetUserRequest) returns (User);
  rpc ListUsers(ListUsersRequest) returns (stream User);
  rpc CreateUser(CreateUserRequest) returns (User);
}
```

**Strengths:**

- High performance (binary, HTTP/2)
- Strong contracts (protobuf)
- Bi-directional streaming
- Excellent for microservices

**Weaknesses:**

- Browser support limited (needs grpc-web)
- Not human-readable
- Steeper learning curve
- Debugging more complex

## Protocol Selection Guide

```text
Decision Tree:

Is this a public API for external developers?
├── Yes → REST (broadest compatibility)
└── No
    └── Do clients need flexible queries?
        ├── Yes → GraphQL
        └── No
            └── Is performance critical?
                ├── Yes → gRPC
                └── No → REST or GraphQL
```

| Factor | REST | GraphQL | gRPC |
| ------ | ---- | ------- | ---- |
| Public APIs | ✅ Best | ⚠️ Possible | ❌ Poor |
| Mobile apps | ⚠️ OK | ✅ Best | ⚠️ Limited |
| Microservices | ⚠️ OK | ⚠️ OK | ✅ Best |
| Real-time | ⚠️ WebSocket | ✅ Subscriptions | ✅ Streaming |
| Browser support | ✅ Native | ✅ Native | ⚠️ grpc-web |
| Caching | ✅ Easy | ⚠️ Complex | ❌ Manual |
| Learning curve | ✅ Low | ⚠️ Medium | ⚠️ Medium |

## REST API Design Best Practices

### Resource Naming

```text
DO:
- Use nouns, not verbs: /users, /orders, /products
- Use plural form: /users (not /user)
- Use kebab-case: /user-profiles (not /userProfiles)
- Nest for relationships: /users/{id}/orders

DON'T:
- /getUsers, /createOrder (verbs)
- /user (singular)
- /user_profiles (snake_case in URLs)
```

### HTTP Methods

| Method | Purpose | Idempotent | Safe |
| ------ | ------- | ---------- | ---- |
| GET | Read resource | Yes | Yes |
| POST | Create resource | No | No |
| PUT | Replace resource | Yes | No |
| PATCH | Partial update | No* | No |
| DELETE | Remove resource | Yes | No |

*PATCH can be idempotent depending on implementation

### Status Codes

| Code | Meaning | When to Use |
| ---- | ------- | ----------- |
| 200 | OK | Successful GET, PUT, PATCH |
| 201 | Created | Successful POST |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Invalid request body |
| 401 | Unauthorized | Missing/invalid auth |
| 403 | Forbidden | Insufficient permissions |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Resource conflict |
| 422 | Unprocessable | Validation failed |
| 429 | Too Many Requests | Rate limited |
| 500 | Server Error | Unexpected error |

### Pagination

```text
Offset-based (simple, but problematic at scale):
GET /users?offset=20&limit=10

Cursor-based (recommended for large datasets):
GET /users?cursor=eyJpZCI6MTAwfQ&limit=10

Response:
{
  "data": [...],
  "pagination": {
    "next_cursor": "eyJpZCI6MTEwfQ",
    "has_more": true
  }
}
```

### Filtering and Sorting

```text
Filtering:
GET /products?category=electronics&price_min=100&price_max=500

Sorting:
GET /products?sort=price:asc,name:desc

Field selection (partial responses):
GET /users?fields=id,name,email
```

### Error Responses

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid request parameters",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      }
    ],
    "request_id": "req_abc123"
  }
}
```

## GraphQL Best Practices

### Schema Design

```graphql
# Use clear, descriptive types
type User {
  id: ID!
  email: String!
  profile: UserProfile
  posts(first: Int, after: String): PostConnection!
}

# Use connections for pagination
type PostConnection {
  edges: [PostEdge!]!
  pageInfo: PageInfo!
}

# Use input types for mutations
input CreateUserInput {
  email: String!
  name: String!
}
```

### Query Complexity Limits

```text
Protect against expensive queries:
- Depth limiting (max nesting level)
- Complexity scoring (assign costs to fields)
- Query timeout
- Rate limiting per client
```

### N+1 Prevention

```text
Use DataLoader pattern:
- Batch requests for same type
- Cache within single request
- Prevents N+1 database queries
```

## gRPC Best Practices

### Service Design

```protobuf
// Keep messages focused
message User {
  string id = 1;
  string email = 2;
  string name = 3;
}

// Use request/response wrappers
message GetUserRequest {
  string id = 1;
}

message GetUserResponse {
  User user = 1;
}

// Support streaming for large datasets
service UserService {
  rpc GetUser(GetUserRequest) returns (GetUserResponse);
  rpc ListUsers(ListUsersRequest) returns (stream User);
}
```

### Error Handling

```text
Use standard gRPC status codes:
- OK (0): Success
- INVALID_ARGUMENT (3): Bad request
- NOT_FOUND (5): Resource missing
- PERMISSION_DENIED (7): Forbidden
- INTERNAL (13): Server error
- UNAVAILABLE (14): Service down
```

## API Evolution

### Backward Compatibility Rules

```text
Safe changes (backward compatible):
- Adding new endpoints
- Adding optional fields
- Adding new enum values (at end)
- Relaxing validation rules

Breaking changes (avoid):
- Removing endpoints
- Removing fields
- Changing field types
- Renaming fields
- Adding required fields
```

### Deprecation Strategy

```text
1. Mark as deprecated (add header/annotation)
2. Document migration path
3. Set sunset date
4. Monitor usage
5. Remove after sunset
```

## Related Skills

- `rate-limiting-patterns` - API protection
- `idempotency-patterns` - Reliable APIs
- `api-versioning` - API evolution
