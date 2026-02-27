---
name: api-design
description: REST, GraphQL, and API design best practices. Use when designing APIs, defining contracts, or reviewing API architecture.
---

# API Design Skill

## Purpose
Design consistent, intuitive, and maintainable APIs following industry best practices.

## REST API Design

### URL Structure

```
https://api.example.com/v1/users/{userId}/orders/{orderId}
        └─────┬─────┘ └┬┘ └──────────────┬────────────────┘
           Base URL  Version          Resource Path
```

**Conventions:**
- Use nouns, not verbs: `/users` not `/getUsers`
- Use plural nouns: `/users` not `/user`
- Use kebab-case: `/user-profiles` not `/userProfiles`
- Nest resources logically: `/users/{id}/orders`
- Limit nesting to 2 levels

### HTTP Methods

| Method | Usage | Idempotent | Safe |
|--------|-------|------------|------|
| GET | Retrieve resource(s) | Yes | Yes |
| POST | Create new resource | No | No |
| PUT | Replace entire resource | Yes | No |
| PATCH | Partial update | No | No |
| DELETE | Remove resource | Yes | No |

### HTTP Status Codes

```
2xx Success
├── 200 OK              - Successful GET/PUT/PATCH
├── 201 Created         - Successful POST (with Location header)
├── 202 Accepted        - Async operation started
└── 204 No Content      - Successful DELETE

4xx Client Error
├── 400 Bad Request     - Invalid request body/params
├── 401 Unauthorized    - Missing/invalid authentication
├── 403 Forbidden       - Authenticated but not authorized
├── 404 Not Found       - Resource doesn't exist
├── 409 Conflict        - Resource conflict (duplicate)
├── 422 Unprocessable   - Validation failed
└── 429 Too Many Requests - Rate limited

5xx Server Error
├── 500 Internal Error  - Unexpected server error
├── 502 Bad Gateway     - Upstream service error
├── 503 Unavailable     - Service temporarily down
└── 504 Gateway Timeout - Upstream timeout
```

### Request/Response Design

```json
// POST /api/v1/users
// Request
{
  "email": "user@example.com",
  "name": "John Doe",
  "role": "user"
}

// Response (201 Created)
{
  "id": "usr_abc123",
  "email": "user@example.com",
  "name": "John Doe",
  "role": "user",
  "createdAt": "2024-01-15T10:30:00Z",
  "updatedAt": "2024-01-15T10:30:00Z"
}
```

### Pagination

```json
// GET /api/v1/users?page=2&limit=20

{
  "data": [...],
  "pagination": {
    "page": 2,
    "limit": 20,
    "total": 150,
    "totalPages": 8,
    "hasNext": true,
    "hasPrev": true
  },
  "links": {
    "self": "/api/v1/users?page=2&limit=20",
    "first": "/api/v1/users?page=1&limit=20",
    "prev": "/api/v1/users?page=1&limit=20",
    "next": "/api/v1/users?page=3&limit=20",
    "last": "/api/v1/users?page=8&limit=20"
  }
}
```

### Filtering and Sorting

```
# Filtering
GET /api/v1/products?status=active&category=electronics
GET /api/v1/products?price[gte]=100&price[lte]=500
GET /api/v1/products?search=iphone

# Sorting
GET /api/v1/products?sort=price         # Ascending
GET /api/v1/products?sort=-price        # Descending
GET /api/v1/products?sort=-createdAt,name  # Multiple

# Field selection
GET /api/v1/users?fields=id,email,name
```

### Error Response Format

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "The request data is invalid",
    "details": [
      {
        "field": "email",
        "message": "Must be a valid email address",
        "code": "INVALID_FORMAT"
      },
      {
        "field": "age",
        "message": "Must be at least 18",
        "code": "MIN_VALUE"
      }
    ],
    "requestId": "req_abc123",
    "timestamp": "2024-01-15T10:30:00Z"
  }
}
```

## GraphQL Design

### Schema Design

```graphql
type User {
  id: ID!
  email: String!
  name: String!
  profile: Profile
  orders(first: Int, after: String): OrderConnection!
  createdAt: DateTime!
}

type Profile {
  bio: String
  avatar: URL
  location: String
}

type OrderConnection {
  edges: [OrderEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type OrderEdge {
  node: Order!
  cursor: String!
}

type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}
```

### Query Design

```graphql
type Query {
  # Single resource
  user(id: ID!): User

  # List with filtering
  users(
    filter: UserFilter
    orderBy: UserOrderBy
    first: Int
    after: String
  ): UserConnection!

  # Search
  searchUsers(query: String!, limit: Int = 10): [User!]!
}

input UserFilter {
  status: UserStatus
  role: UserRole
  createdAfter: DateTime
}

enum UserOrderBy {
  CREATED_AT_ASC
  CREATED_AT_DESC
  NAME_ASC
  NAME_DESC
}
```

### Mutation Design

```graphql
type Mutation {
  # Create
  createUser(input: CreateUserInput!): CreateUserPayload!

  # Update
  updateUser(id: ID!, input: UpdateUserInput!): UpdateUserPayload!

  # Delete
  deleteUser(id: ID!): DeleteUserPayload!
}

input CreateUserInput {
  email: String!
  name: String!
  role: UserRole = USER
}

type CreateUserPayload {
  user: User
  errors: [UserError!]
}

type UserError {
  field: String
  message: String!
  code: ErrorCode!
}
```

## API Versioning

### Strategies

| Strategy | Example | Pros | Cons |
|----------|---------|------|------|
| URL Path | `/v1/users` | Clear, cacheable | Multiple codebases |
| Header | `Accept: application/vnd.api.v1+json` | Clean URLs | Harder to test |
| Query Param | `/users?version=1` | Easy to test | Caching issues |

**Recommendation:** URL path versioning for simplicity.

### Deprecation Process

```http
# Response headers for deprecated endpoints
Deprecation: true
Sunset: Sat, 1 Jan 2025 00:00:00 GMT
Link: </v2/users>; rel="successor-version"
```

## Authentication & Authorization

### Token-Based Auth

```http
# Request
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...

# Response for expired token (401)
WWW-Authenticate: Bearer realm="api",
                  error="invalid_token",
                  error_description="Token has expired"
```

### API Key Auth

```http
# Header approach (recommended)
X-API-Key: sk_live_abc123

# Query param (avoid - logs exposure)
GET /api/users?api_key=sk_live_abc123
```

### OAuth 2.0 Scopes

```http
Authorization: Bearer token

# Token includes scopes
{
  "scope": "read:users write:orders",
  "exp": 1705312200
}
```

## Rate Limiting

### Response Headers

```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1705312200
Retry-After: 60
```

### Rate Limit Response

```json
// 429 Too Many Requests
{
  "error": {
    "code": "RATE_LIMITED",
    "message": "Rate limit exceeded. Try again in 60 seconds.",
    "retryAfter": 60
  }
}
```

## API Documentation (OpenAPI)

```yaml
openapi: 3.0.3
info:
  title: User API
  version: 1.0.0
  description: API for managing users

paths:
  /users:
    get:
      summary: List users
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
        '401':
          $ref: '#/components/responses/Unauthorized'

components:
  schemas:
    User:
      type: object
      required: [id, email, name]
      properties:
        id:
          type: string
          example: usr_abc123
        email:
          type: string
          format: email
        name:
          type: string
```

## API Design Checklist

### Consistency
- [ ] Consistent naming conventions
- [ ] Consistent error format
- [ ] Consistent pagination
- [ ] Consistent date formats (ISO 8601)

### Usability
- [ ] Intuitive URL structure
- [ ] Clear error messages
- [ ] Comprehensive documentation
- [ ] SDK/client libraries

### Security
- [ ] Authentication required
- [ ] Rate limiting
- [ ] Input validation
- [ ] CORS configured
- [ ] HTTPS only

### Performance
- [ ] Pagination for lists
- [ ] Field selection
- [ ] Caching headers
- [ ] Compression

### Maintainability
- [ ] Versioning strategy
- [ ] Deprecation policy
- [ ] Changelog maintained
- [ ] OpenAPI spec up-to-date
