---
name: api-design
description: REST API, GraphQL, and API design patterns and best practices
license: MIT
compatibility: opencode
---

# API Design Skill

Comprehensive patterns and best practices for designing REST APIs, GraphQL APIs, and API-first development.

## What I Know

### REST API Design Principles

**Resource Naming**
```
# Good - nouns, plural, lowercase, hyphenated
GET /users
GET /users/{id}
GET /users/{id}/posts
GET /blog-posts
GET /order-items

# Bad - verbs, singular, camelCase
GET /getUsers
GET /user
GET /getUserPosts
GET /blogPosts
```

**HTTP Methods**
| Method | Usage | Idempotent | Request Body | Response |
|--------|-------|------------|--------------|----------|
| GET | Read resource(s) | Yes | No | Resource(s) |
| POST | Create resource | No | Yes | Created resource |
| PUT | Replace resource | Yes | Yes | Updated resource |
| PATCH | Partial update | No* | Yes | Updated resource |
| DELETE | Remove resource | Yes | No | Empty or confirmation |

**Status Codes**
```
# Success
200 OK              - Successful GET, PUT, PATCH
201 Created         - Successful POST
204 No Content      - Successful DELETE

# Client Errors
400 Bad Request     - Invalid request body/params
401 Unauthorized    - Missing/invalid authentication
403 Forbidden       - Valid auth, insufficient permissions
404 Not Found       - Resource doesn't exist
409 Conflict        - Resource state conflict (duplicate)
422 Unprocessable   - Validation errors

# Server Errors
500 Internal Error  - Unexpected server error
502 Bad Gateway     - Upstream service error
503 Service Unavailable - Temporary overload
```

### Standardized Response Format

**Success Response**
```json
{
  "success": true,
  "data": {
    "id": 123,
    "email": "user@example.com",
    "name": "John Doe"
  }
}
```

**List Response with Pagination**
```json
{
  "success": true,
  "data": [
    { "id": 1, "name": "Item 1" },
    { "id": 2, "name": "Item 2" }
  ],
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 100,
    "totalPages": 5,
    "hasMore": true
  }
}
```

**Error Response**
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      },
      {
        "field": "password",
        "message": "Password must be at least 8 characters"
      }
    ]
  }
}
```

### Pagination Patterns

**Offset-Based (Simple)**
```
GET /users?page=2&limit=20

Response:
{
  "data": [...],
  "meta": {
    "page": 2,
    "limit": 20,
    "total": 150,
    "totalPages": 8
  }
}
```

**Cursor-Based (Scalable)**
```
GET /users?cursor=abc123&limit=20

Response:
{
  "data": [...],
  "meta": {
    "limit": 20,
    "nextCursor": "def456",
    "hasMore": true
  }
}
```

### Filtering & Sorting

**Query Parameters**
```
# Filtering
GET /users?status=active
GET /users?role=admin&status=active
GET /products?price_min=10&price_max=100
GET /posts?created_after=2024-01-01

# Sorting
GET /users?sort=name        # Ascending
GET /users?sort=-createdAt  # Descending
GET /users?sort=name,-createdAt  # Multiple

# Field Selection
GET /users?fields=id,name,email

# Search
GET /users?search=john
GET /products?q=laptop
```

### API Versioning

**URI Versioning (Recommended)**
```
GET /api/v1/users
GET /api/v2/users
```

**Header Versioning**
```
GET /api/users
Accept: application/vnd.api+json;version=1
```

**Query Parameter**
```
GET /api/users?version=1
```

### Authentication Patterns

**JWT Bearer Token**
```
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**API Key**
```
# Header
X-API-Key: your-api-key

# Query (less secure)
GET /api/users?api_key=your-api-key
```

**OAuth 2.0 Flows**
```
# Authorization Code (Web apps)
# Client Credentials (Server-to-server)
# PKCE (Mobile/SPA)
```

### Rate Limiting

**Headers**
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1704067200
Retry-After: 60
```

**429 Response**
```json
{
  "success": false,
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests",
    "retryAfter": 60
  }
}
```

### OpenAPI/Swagger Specification

```yaml
openapi: 3.0.3
info:
  title: My API
  version: 1.0.0
  description: API for managing users

servers:
  - url: https://api.example.com/v1
    description: Production

paths:
  /users:
    get:
      summary: List users
      tags:
        - Users
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
        - name: limit
          in: query
          schema:
            type: integer
            default: 20
            maximum: 100
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserList'
        '401':
          $ref: '#/components/responses/Unauthorized'

    post:
      summary: Create user
      tags:
        - Users
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateUser'
      responses:
        '201':
          description: Created
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserResponse'
        '400':
          $ref: '#/components/responses/BadRequest'
        '409':
          $ref: '#/components/responses/Conflict'

  /users/{id}:
    get:
      summary: Get user by ID
      tags:
        - Users
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: integer
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserResponse'
        '404':
          $ref: '#/components/responses/NotFound'

components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: integer
        email:
          type: string
          format: email
        name:
          type: string
        createdAt:
          type: string
          format: date-time
      required:
        - id
        - email
        - name

    CreateUser:
      type: object
      properties:
        email:
          type: string
          format: email
        name:
          type: string
          minLength: 2
          maxLength: 100
        password:
          type: string
          minLength: 8
      required:
        - email
        - name
        - password

    UserResponse:
      type: object
      properties:
        success:
          type: boolean
        data:
          $ref: '#/components/schemas/User'

    UserList:
      type: object
      properties:
        success:
          type: boolean
        data:
          type: array
          items:
            $ref: '#/components/schemas/User'
        meta:
          $ref: '#/components/schemas/Pagination'

    Pagination:
      type: object
      properties:
        page:
          type: integer
        limit:
          type: integer
        total:
          type: integer
        totalPages:
          type: integer

    Error:
      type: object
      properties:
        success:
          type: boolean
          example: false
        error:
          type: object
          properties:
            code:
              type: string
            message:
              type: string
            details:
              type: array
              items:
                type: object

  responses:
    BadRequest:
      description: Bad Request
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
    Unauthorized:
      description: Unauthorized
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
    NotFound:
      description: Not Found
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
    Conflict:
      description: Conflict
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

### GraphQL Patterns

**Schema Design**
```graphql
type Query {
  user(id: ID!): User
  users(
    page: Int = 1
    limit: Int = 20
    filter: UserFilter
  ): UserConnection!
}

type Mutation {
  createUser(input: CreateUserInput!): UserPayload!
  updateUser(id: ID!, input: UpdateUserInput!): UserPayload!
  deleteUser(id: ID!): DeletePayload!
}

type User {
  id: ID!
  email: String!
  name: String!
  posts(first: Int): [Post!]!
  createdAt: DateTime!
}

input CreateUserInput {
  email: String!
  name: String!
  password: String!
}

input UpdateUserInput {
  email: String
  name: String
}

input UserFilter {
  status: UserStatus
  role: UserRole
  search: String
}

type UserConnection {
  edges: [UserEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type UserEdge {
  node: User!
  cursor: String!
}

type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}

type UserPayload {
  user: User
  errors: [Error!]
}

type DeletePayload {
  success: Boolean!
  errors: [Error!]
}

type Error {
  field: String
  message: String!
}
```

**Resolver Patterns**
```typescript
const resolvers = {
  Query: {
    user: async (_, { id }, context) => {
      return context.dataSources.users.findById(id)
    },
    users: async (_, { page, limit, filter }, context) => {
      return context.dataSources.users.findAll({ page, limit, filter })
    },
  },
  Mutation: {
    createUser: async (_, { input }, context) => {
      try {
        const user = await context.dataSources.users.create(input)
        return { user, errors: [] }
      } catch (error) {
        return { user: null, errors: [{ message: error.message }] }
      }
    },
  },
  User: {
    posts: async (user, { first }, context) => {
      return context.dataSources.posts.findByUserId(user.id, { first })
    },
  },
}
```

### Error Handling Best Practices

**Error Codes**
```typescript
enum ErrorCode {
  // Authentication
  INVALID_TOKEN = 'INVALID_TOKEN',
  TOKEN_EXPIRED = 'TOKEN_EXPIRED',
  UNAUTHORIZED = 'UNAUTHORIZED',

  // Authorization
  FORBIDDEN = 'FORBIDDEN',
  INSUFFICIENT_PERMISSIONS = 'INSUFFICIENT_PERMISSIONS',

  // Validation
  VALIDATION_ERROR = 'VALIDATION_ERROR',
  INVALID_INPUT = 'INVALID_INPUT',

  // Resource
  NOT_FOUND = 'NOT_FOUND',
  ALREADY_EXISTS = 'ALREADY_EXISTS',
  CONFLICT = 'CONFLICT',

  // Rate Limiting
  RATE_LIMIT_EXCEEDED = 'RATE_LIMIT_EXCEEDED',

  // Server
  INTERNAL_ERROR = 'INTERNAL_ERROR',
  SERVICE_UNAVAILABLE = 'SERVICE_UNAVAILABLE',
}
```

### HATEOAS (Hypermedia)

```json
{
  "success": true,
  "data": {
    "id": 123,
    "name": "John Doe",
    "email": "john@example.com"
  },
  "links": {
    "self": "/api/v1/users/123",
    "posts": "/api/v1/users/123/posts",
    "update": "/api/v1/users/123",
    "delete": "/api/v1/users/123"
  }
}
```

### Caching Headers

```
# Cache for 1 hour
Cache-Control: max-age=3600, public

# No cache
Cache-Control: no-store, no-cache, must-revalidate

# ETag for conditional requests
ETag: "abc123"
If-None-Match: "abc123"

# Last-Modified
Last-Modified: Wed, 21 Oct 2024 07:28:00 GMT
If-Modified-Since: Wed, 21 Oct 2024 07:28:00 GMT
```

### Security Best Practices

1. **Always use HTTPS**
2. **Validate all input** - Never trust client data
3. **Rate limit endpoints** - Prevent abuse
4. **Use proper authentication** - JWT, OAuth 2.0
5. **Implement authorization** - Check permissions
6. **Sanitize output** - Prevent XSS
7. **Use parameterized queries** - Prevent SQL injection
8. **Log security events** - Audit trail
9. **Don't expose internals** - Generic error messages
10. **Set security headers** - CORS, CSP, etc.

### API Documentation

**Good Documentation Includes:**
- Authentication guide
- Request/response examples
- Error code reference
- Rate limiting info
- Changelog/versioning
- SDKs and code samples
- Interactive playground (Swagger UI)

### Common Pitfalls

1. **Inconsistent naming** - Stick to one convention
2. **Missing validation** - Always validate input
3. **No versioning** - Version from the start
4. **Poor error messages** - Be helpful, not cryptic
5. **No rate limiting** - Protect from abuse
6. **Exposing internals** - Don't leak stack traces
7. **Ignoring caching** - Set proper cache headers
8. **No pagination** - Always paginate lists
9. **Breaking changes** - Use versioning properly
10. **Missing documentation** - Document everything

---

*Part of SuperAI GitHub - Centralized OpenCode Configuration*
