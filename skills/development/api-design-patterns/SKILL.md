---
name: api-design-patterns
description: Provides REST and GraphQL API design patterns for Node.js, Flask, and FastAPI. Use when designing endpoints, request/response structures, API architecture, pagination, authentication, rate limiting, or when working in /api/ or /routes/ directories.
---

# API Design Patterns Skill

Best practices for designing RESTful and GraphQL APIs.

## Resources

For detailed code examples, see:
- `references/express-examples.md` - Node.js + Express patterns
- `references/fastapi-examples.md` - FastAPI (Python) patterns
- `references/flask-examples.md` - Flask (Python) patterns

## Core Principles

1. **Consistency** - Use consistent naming, structure, and behavior
2. **RESTful** - Follow REST conventions for resource-based APIs
3. **Versioning** - Plan for API evolution
4. **Documentation** - APIs should be self-documenting
5. **Error Handling** - Consistent, informative error responses
6. **Security** - Authentication, authorization, input validation
7. **Performance** - Pagination, caching, rate limiting

## URL Structure

```
GET    /api/v1/users              # List users
GET    /api/v1/users/:id          # Get specific user
POST   /api/v1/users              # Create user
PUT    /api/v1/users/:id          # Update user (full)
PATCH  /api/v1/users/:id          # Update user (partial)
DELETE /api/v1/users/:id          # Delete user
GET    /api/v1/users/:id/posts    # Nested resource
```

**Avoid:**
- Verbs in URLs (`/getUsers`)
- Redundant paths (`/user/create`)
- Deep nesting (>2 levels)

## HTTP Status Codes

| Code | Meaning | When to Use |
|------|---------|-------------|
| 200 | OK | Success (GET, PUT, PATCH) |
| 201 | Created | Resource created (POST) |
| 204 | No Content | Success with no body (DELETE) |
| 400 | Bad Request | Invalid input |
| 401 | Unauthorized | Not authenticated |
| 403 | Forbidden | Authenticated but not authorized |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Duplicate or state conflict |
| 422 | Unprocessable | Validation failed |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Error | Server error |

## Response Format

### Success Response
```json
{
  "data": {
    "id": "123",
    "name": "John Doe"
  },
  "meta": {
    "timestamp": "2025-10-26T10:00:00Z"
  }
}
```

### List Response (Paginated)
```json
{
  "data": [
    { "id": "1", "name": "User 1" }
  ],
  "meta": {
    "total": 100,
    "page": 1,
    "perPage": 20,
    "totalPages": 5
  },
  "links": {
    "first": "/api/v1/users?page=1",
    "prev": null,
    "next": "/api/v1/users?page=2",
    "last": "/api/v1/users?page=5"
  }
}
```

### Error Response
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      }
    ]
  }
}
```

## API Versioning

```typescript
// URL versioning (recommended)
app.use('/api/v1', routesV1);
app.use('/api/v2', routesV2);

// Deprecation headers
res.setHeader('Deprecation', 'true');
res.setHeader('Sunset', 'Wed, 11 Nov 2025 11:11:11 GMT');
```

## GraphQL Patterns

### Schema Design
```graphql
type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post!]!  # Resolver handles N+1 with DataLoader
  createdAt: DateTime!
}

type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
}

type Query {
  user(id: ID!): User
  users(first: Int, after: String): UserConnection!
}

type Mutation {
  createUser(input: CreateUserInput!): User!
  updateUser(id: ID!, input: UpdateUserInput!): User!
  deleteUser(id: ID!): Boolean!
}

input CreateUserInput {
  name: String!
  email: String!
}
```

### Resolver Patterns (Node.js)
```typescript
const resolvers = {
  Query: {
    user: async (_, { id }, { dataSources }) => {
      return dataSources.usersAPI.getUser(id);
    },
    users: async (_, { first, after }, { dataSources }) => {
      return dataSources.usersAPI.getUsers({ first, after });
    },
  },
  Mutation: {
    createUser: async (_, { input }, { dataSources }) => {
      return dataSources.usersAPI.createUser(input);
    },
  },
  User: {
    // Field resolver with DataLoader to prevent N+1
    posts: async (user, _, { loaders }) => {
      return loaders.postsByUserId.load(user.id);
    },
  },
};
```

### Error Handling
```typescript
// Throw typed errors
import { GraphQLError } from 'graphql';

throw new GraphQLError('User not found', {
  extensions: {
    code: 'NOT_FOUND',
    http: { status: 404 },
  },
});

// Common error codes
// UNAUTHENTICATED, FORBIDDEN, NOT_FOUND, VALIDATION_ERROR, INTERNAL_ERROR
```

### Pagination (Relay Cursor-Based)
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

### GraphQL vs REST - When to Use
| Use GraphQL | Use REST |
|-------------|----------|
| Multiple clients with different data needs | Simple CRUD operations |
| Deeply nested data in single request | Caching critical (HTTP caching) |
| Rapid iteration, evolving schema | Public API with stability guarantees |
| Mobile apps (minimize requests) | File uploads, streaming |

## Quick Reference

### Do
- Use plural nouns for collections (`/users`)
- Return appropriate status codes
- Validate all inputs
- Implement pagination for lists
- Use consistent error format
- Version your APIs
- Document with OpenAPI/Swagger
- Implement rate limiting
- Use HTTPS in production

### Don't
- Use verbs in URLs
- Nest resources more than 2 levels
- Return sensitive data
- Use HTTP 200 for errors
- Expose internal error details
- Skip input validation
- Return huge lists without pagination

## Input Validation

Always validate using schemas:

**TypeScript (Zod):**
```typescript
const createUserSchema = z.object({
  name: z.string().min(1).max(100),
  email: z.string().email(),
  age: z.number().int().min(0).max(150).optional(),
});
```

**Python (Pydantic):**
```python
class UserCreate(BaseModel):
    name: str = Field(..., min_length=1, max_length=100)
    email: EmailStr
    age: Optional[int] = Field(None, ge=0, le=150)
```

## Authentication Pattern

1. Extract token from `Authorization: Bearer <token>` header
2. Verify token signature and expiration
3. Load user from token payload
4. Attach user to request context
5. Return 401 for invalid/missing tokens

## Authorization Pattern

1. Check user role/permissions after authentication
2. Use middleware/decorators for role checks
3. Return 403 for insufficient permissions

## Rate Limiting

```typescript
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // requests per window
  message: {
    error: {
      code: 'RATE_LIMIT_EXCEEDED',
      message: 'Too many requests',
    },
  },
});
```

## Testing APIs

```typescript
describe('GET /api/v1/users', () => {
  it('returns paginated users', async () => {
    const response = await request(app)
      .get('/api/v1/users?page=1')
      .expect(200);

    expect(response.body).toHaveProperty('data');
    expect(response.body).toHaveProperty('meta');
  });

  it('returns 401 without auth', async () => {
    await request(app).get('/api/v1/users').expect(401);
  });
});
```

---

Good API design makes your API intuitive, consistent, and easy to use.

---

## Version
- v1.1.0 (2025-12-05): Enriched trigger keywords in description
- v1.0.0 (2025-11-15): Initial version
