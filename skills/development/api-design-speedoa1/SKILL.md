---
name: api-design
description: RESTful and GraphQL API design patterns and best practices
license: MIT
---

# API Design Skill

Use this skill when designing, implementing, or reviewing APIs.

## When to Use

- Creating new API endpoints
- Refactoring existing APIs
- Designing API architecture
- Reviewing API contracts
- Writing API documentation

## REST API Design

### URL Structure

```
# Resources (plural nouns)
GET    /api/users              # List
POST   /api/users              # Create
GET    /api/users/:id          # Read
PUT    /api/users/:id          # Replace
PATCH  /api/users/:id          # Update
DELETE /api/users/:id          # Delete

# Nested resources (relationships)
GET    /api/users/:id/orders   # User's orders
POST   /api/users/:id/orders   # Create order for user

# Actions (when CRUD doesn't fit)
POST   /api/users/:id/activate
POST   /api/orders/:id/cancel
POST   /api/payments/:id/refund

# Filtering, sorting, pagination
GET    /api/users?status=active
GET    /api/users?sort=-createdAt
GET    /api/users?page=2&limit=20
GET    /api/users?fields=id,name,email
```

### HTTP Methods & Status Codes

| Method | Use | Success | Common Errors |
|--------|-----|---------|---------------|
| GET | Read | 200 | 404 |
| POST | Create | 201 | 400, 409 |
| PUT | Replace | 200 | 400, 404 |
| PATCH | Update | 200 | 400, 404 |
| DELETE | Remove | 204 | 404 |

### Response Format

```typescript
// Success response
{
  "data": {
    "id": "user_123",
    "type": "user",
    "attributes": {
      "name": "John Doe",
      "email": "john@example.com",
      "createdAt": "2024-01-15T10:30:00Z"
    },
    "relationships": {
      "organization": {
        "id": "org_456",
        "type": "organization"
      }
    }
  },
  "meta": {
    "requestId": "req_abc123"
  }
}

// List response
{
  "data": [...],
  "meta": {
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 150,
      "totalPages": 8
    }
  },
  "links": {
    "self": "/api/users?page=1",
    "next": "/api/users?page=2",
    "prev": null,
    "first": "/api/users?page=1",
    "last": "/api/users?page=8"
  }
}

// Error response
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed",
    "details": [
      {
        "field": "email",
        "code": "invalid_format",
        "message": "Invalid email format"
      }
    ]
  },
  "meta": {
    "requestId": "req_abc123",
    "timestamp": "2024-01-15T10:30:00Z"
  }
}
```

### Request Validation

```typescript
import { z } from 'zod';

// Define schema
const CreateUserSchema = z.object({
  name: z.string().min(1).max(100),
  email: z.string().email(),
  password: z.string().min(8),
  role: z.enum(['user', 'admin']).default('user'),
  preferences: z.object({
    notifications: z.boolean().default(true),
    theme: z.enum(['light', 'dark']).optional(),
  }).optional(),
});

// Validation middleware
function validate<T>(schema: z.Schema<T>) {
  return (req: Request, res: Response, next: NextFunction) => {
    const result = schema.safeParse(req.body);
    
    if (!result.success) {
      return res.status(422).json({
        error: {
          code: 'VALIDATION_ERROR',
          message: 'Validation failed',
          details: result.error.issues.map(issue => ({
            field: issue.path.join('.'),
            code: issue.code,
            message: issue.message,
          })),
        },
      });
    }
    
    req.body = result.data;
    next();
  };
}

// Usage
app.post('/api/users', validate(CreateUserSchema), createUser);
```

### Authentication

```typescript
// JWT Bearer token
// Header: Authorization: Bearer <token>

async function authenticate(req: Request, res: Response, next: NextFunction) {
  const token = req.headers.authorization?.replace('Bearer ', '');
  
  if (!token) {
    return res.status(401).json({
      error: {
        code: 'UNAUTHORIZED',
        message: 'Authentication required',
      },
    });
  }

  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET);
    req.user = payload;
    next();
  } catch (error) {
    return res.status(401).json({
      error: {
        code: 'UNAUTHORIZED',
        message: 'Invalid or expired token',
      },
    });
  }
}
```

### Rate Limiting

```typescript
import rateLimit from 'express-rate-limit';

const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100,
  standardHeaders: true,
  legacyHeaders: false,
  message: {
    error: {
      code: 'RATE_LIMITED',
      message: 'Too many requests, please try again later',
    },
  },
});

app.use('/api/', apiLimiter);
```

### Versioning

```typescript
// URL versioning (recommended)
app.use('/api/v1', v1Routes);
app.use('/api/v2', v2Routes);

// Header versioning
// Accept: application/vnd.myapi.v2+json

function versionMiddleware(req, res, next) {
  const accept = req.headers.accept || '';
  const match = accept.match(/application\/vnd\.myapi\.v(\d+)\+json/);
  req.apiVersion = match ? parseInt(match[1]) : 1;
  next();
}
```

## GraphQL API Design

### Schema Design

```graphql
# Types
type User {
  id: ID!
  name: String!
  email: String!
  createdAt: DateTime!
  orders(first: Int, after: String): OrderConnection!
}

type Order {
  id: ID!
  total: Float!
  status: OrderStatus!
  items: [OrderItem!]!
  user: User!
}

enum OrderStatus {
  PENDING
  PROCESSING
  SHIPPED
  DELIVERED
  CANCELLED
}

# Connections for pagination
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

# Queries
type Query {
  user(id: ID!): User
  users(filter: UserFilter, first: Int, after: String): UserConnection!
  order(id: ID!): Order
}

# Mutations
type Mutation {
  createUser(input: CreateUserInput!): CreateUserPayload!
  updateUser(id: ID!, input: UpdateUserInput!): UpdateUserPayload!
  deleteUser(id: ID!): DeleteUserPayload!
}

# Inputs
input CreateUserInput {
  name: String!
  email: String!
  password: String!
}

# Payloads (for error handling)
type CreateUserPayload {
  user: User
  errors: [UserError!]!
}

type UserError {
  field: String
  message: String!
  code: String!
}
```

### Resolvers

```typescript
const resolvers = {
  Query: {
    user: async (_, { id }, context) => {
      return context.dataSources.users.findById(id);
    },
    users: async (_, args, context) => {
      return context.dataSources.users.findMany(args);
    },
  },
  
  Mutation: {
    createUser: async (_, { input }, context) => {
      try {
        const user = await context.dataSources.users.create(input);
        return { user, errors: [] };
      } catch (error) {
        return {
          user: null,
          errors: [{ message: error.message, code: 'CREATE_FAILED' }],
        };
      }
    },
  },
  
  User: {
    orders: async (user, args, context) => {
      return context.dataSources.orders.findByUser(user.id, args);
    },
  },
};
```

## OpenAPI Documentation

```yaml
openapi: 3.0.3
info:
  title: My API
  version: 1.0.0
  description: API for managing users and orders

servers:
  - url: https://api.example.com/v1
    description: Production

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
    
    post:
      summary: Create user
      tags: [Users]
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
                $ref: '#/components/schemas/User'
        '422':
          description: Validation error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'

components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: string
        name:
          type: string
        email:
          type: string
          format: email
      required: [id, name, email]
    
    CreateUser:
      type: object
      properties:
        name:
          type: string
          minLength: 1
          maxLength: 100
        email:
          type: string
          format: email
        password:
          type: string
          minLength: 8
      required: [name, email, password]
    
    Error:
      type: object
      properties:
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
```

## API Design Checklist

### Naming & Structure
- [ ] Resources are plural nouns
- [ ] URLs are lowercase with hyphens
- [ ] Consistent naming across endpoints
- [ ] Logical resource hierarchy

### Request/Response
- [ ] Appropriate HTTP methods used
- [ ] Correct status codes returned
- [ ] Consistent response format
- [ ] Proper error responses

### Security
- [ ] Authentication implemented
- [ ] Authorization checked
- [ ] Input validation present
- [ ] Rate limiting configured

### Documentation
- [ ] OpenAPI/Swagger spec complete
- [ ] Examples provided
- [ ] Error codes documented
- [ ] Authentication explained
