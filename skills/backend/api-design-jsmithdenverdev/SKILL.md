---
name: api-design
description: RESTful and GraphQL API design principles, patterns, and best practices
license: MIT
compatibility: opencode
metadata:
  category: architecture
  domain: backend
  protocols: rest,graphql
---

## What I do

I provide expertise in designing robust, scalable, and maintainable APIs:

- **RESTful API Design**: Resource modeling, HTTP methods, status codes
- **GraphQL Schema Design**: Types, queries, mutations, subscriptions
- **API Versioning**: Strategies for evolving APIs without breaking clients
- **Error Handling**: Consistent error responses and problem details
- **Authentication & Authorization**: OAuth2, JWT, API keys, RBAC
- **Rate Limiting & Pagination**: Scalability and performance patterns
- **Documentation**: OpenAPI/Swagger, GraphQL introspection

## When to use me

Load this skill when you need to:
- Design a new API from scratch
- Refactor or version an existing API
- Implement consistent error handling
- Add authentication/authorization
- Optimize API performance
- Document API contracts

## REST API Principles

### 1. Resource-Oriented Design
```
GET    /users           # List users
POST   /users           # Create user
GET    /users/:id       # Get user
PUT    /users/:id       # Update user (full)
PATCH  /users/:id       # Update user (partial)
DELETE /users/:id       # Delete user

# Nested resources
GET    /users/:id/posts       # List user's posts
POST   /users/:id/posts       # Create post for user
```

### 2. HTTP Status Codes
```
2xx Success
  200 OK              - Successful GET, PUT, PATCH, DELETE
  201 Created         - Successful POST
  204 No Content      - Successful DELETE with no response body

4xx Client Errors
  400 Bad Request     - Invalid input
  401 Unauthorized    - Missing or invalid authentication
  403 Forbidden       - Authenticated but not authorized
  404 Not Found       - Resource doesn't exist
  409 Conflict        - Conflict with current state
  422 Unprocessable   - Validation errors

5xx Server Errors
  500 Internal Server Error
  503 Service Unavailable
```

### 3. Consistent Error Response Format
```typescript
interface ProblemDetails {
  type: string;        // URI reference to problem type
  title: string;       // Short, human-readable summary
  status: number;      // HTTP status code
  detail?: string;     // Human-readable explanation
  instance?: string;   // URI reference to specific occurrence
  errors?: Record<string, string[]>; // Validation errors
}

// Example
{
  "type": "https://api.example.com/problems/validation-error",
  "title": "Validation Error",
  "status": 422,
  "detail": "The request contains invalid fields",
  "instance": "/users",
  "errors": {
    "email": ["Email is required", "Email must be valid"],
    "age": ["Age must be at least 18"]
  }
}
```

### 4. Pagination
```typescript
// Cursor-based (recommended for large datasets)
GET /users?cursor=eyJpZCI6MTIzfQ&limit=20

{
  "data": [...],
  "pagination": {
    "nextCursor": "eyJpZCI6MTQzfQ",
    "hasMore": true
  }
}

// Offset-based (simpler, but less performant)
GET /users?page=2&limit=20

{
  "data": [...],
  "pagination": {
    "page": 2,
    "limit": 20,
    "total": 150,
    "totalPages": 8
  }
}
```

### 5. Filtering, Sorting, Field Selection
```
# Filtering
GET /users?status=active&role=admin

# Sorting
GET /users?sort=createdAt:desc,name:asc

# Field selection (sparse fieldsets)
GET /users?fields=id,name,email

# Combined
GET /users?status=active&sort=name:asc&fields=id,name&limit=50
```

## GraphQL Schema Design

### 1. Type System
```graphql
type User {
  id: ID!
  email: String!
  name: String!
  posts: [Post!]!
  createdAt: DateTime!
}

type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
  published: Boolean!
  createdAt: DateTime!
  updatedAt: DateTime!
}

type Query {
  user(id: ID!): User
  users(filter: UserFilter, page: PageInput): UserConnection!
  post(id: ID!): Post
  posts(filter: PostFilter, page: PageInput): PostConnection!
}

type Mutation {
  createUser(input: CreateUserInput!): User!
  updateUser(id: ID!, input: UpdateUserInput!): User!
  deleteUser(id: ID!): Boolean!
}

input CreateUserInput {
  email: String!
  name: String!
  password: String!
}

input UserFilter {
  status: UserStatus
  role: UserRole
  search: String
}
```

### 2. Connection Pattern (Relay-style pagination)
```graphql
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
```

### 3. Error Handling
```typescript
// Use union types for operation results
type CreateUserResult = User | ValidationError | EmailTakenError

type ValidationError {
  message: String!
  fields: [FieldError!]!
}

type FieldError {
  field: String!
  message: String!
}
```

## API Versioning Strategies

### 1. URL Versioning (Explicit)
```
/v1/users
/v2/users
```

### 2. Header Versioning (Preferred)
```
GET /users
Accept: application/vnd.myapi.v2+json
```

### 3. Query Parameter
```
GET /users?version=2
```

### 4. GraphQL Schema Stitching
```graphql
# Deprecate fields gradually
type User {
  id: ID!
  name: String! @deprecated(reason: "Use firstName and lastName instead")
  firstName: String!
  lastName: String!
}
```

## Authentication & Authorization

### 1. JWT Bearer Tokens
```typescript
// Request
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

// Middleware
interface JWTPayload {
  sub: string;      // User ID
  email: string;
  role: string;
  iat: number;      // Issued at
  exp: number;      // Expiration
}

const authenticateJWT = (token: string): JWTPayload => {
  return jwt.verify(token, process.env.JWT_SECRET);
};
```

### 2. API Keys
```typescript
// Request
X-API-Key: sk_live_1234567890abcdef

// Rate limiting per key
interface APIKey {
  key: string;
  userId: string;
  rateLimit: number;
  expiresAt?: Date;
}
```

### 3. Role-Based Access Control
```typescript
enum Permission {
  READ_USERS = 'users:read',
  WRITE_USERS = 'users:write',
  DELETE_USERS = 'users:delete',
}

const authorize = (required: Permission[]) => {
  return (req, res, next) => {
    const userPermissions = req.user.permissions;
    const hasPermission = required.every(p => 
      userPermissions.includes(p)
    );
    
    if (!hasPermission) {
      return res.status(403).json({
        type: 'forbidden',
        title: 'Forbidden',
        status: 403,
      });
    }
    
    next();
  };
};

// Usage
app.delete('/users/:id', 
  authenticateJWT,
  authorize([Permission.DELETE_USERS]),
  deleteUser
);
```

## Rate Limiting

### 1. Token Bucket Algorithm
```typescript
interface RateLimiter {
  limit: number;        // Max requests
  window: number;       // Time window in seconds
  remaining: number;
  resetAt: Date;
}

// Headers
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 87
X-RateLimit-Reset: 1640995200
```

### 2. Response when rate limited
```typescript
// HTTP 429 Too Many Requests
{
  "type": "rate-limit-exceeded",
  "title": "Rate Limit Exceeded",
  "status": 429,
  "detail": "You have exceeded the rate limit. Try again later.",
  "retryAfter": 3600
}

// Header
Retry-After: 3600
```

## API Documentation

### OpenAPI 3.0 Example
```yaml
openapi: 3.0.0
info:
  title: User API
  version: 1.0.0
paths:
  /users:
    get:
      summary: List users
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items:
                      $ref: '#/components/schemas/User'
components:
  schemas:
    User:
      type: object
      required:
        - id
        - email
      properties:
        id:
          type: string
        email:
          type: string
          format: email
```

## Best Practices

1. **Use nouns for resources, not verbs**: `/users` not `/getUsers`
2. **Use plural nouns**: `/users` not `/user`
3. **Return proper status codes**: Don't return 200 for errors
4. **Include request IDs**: For tracing and debugging
5. **Version your API**: Plan for evolution
6. **Validate input**: Use schemas (JSON Schema, Zod, etc.)
7. **Log everything**: Requests, responses, errors
8. **Implement CORS properly**: For web clients
9. **Use HTTPS**: Always, no exceptions
10. **Document everything**: OpenAPI, examples, guides

## References

- RESTful Web APIs by Leonard Richardson
- GraphQL Best Practices: graphql.org/learn/best-practices
- RFC 7807: Problem Details for HTTP APIs
- OpenAPI Specification: swagger.io/specification
