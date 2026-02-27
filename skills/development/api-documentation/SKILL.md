---
name: api-documentation
description: Generate OpenAPI specs and API documentation from code. Triggers: document API, generate OpenAPI, create API docs, swagger spec, GraphQL schema, API specification, document endpoints
---

# API Documentation Skill

Generate comprehensive API documentation including OpenAPI/Swagger specs from source code.

## When to Use

- User asks to "document API" or "generate OpenAPI spec"
- User wants Swagger documentation
- User needs GraphQL schema documentation
- User asks to document endpoints

## Supported API Types

### REST APIs
- Express.js, Fastify, Koa (Node.js)
- Flask, FastAPI, Django (Python)
- Spring Boot (Java)
- Gin, Echo, Fiber (Go)
- Rails, Sinatra (Ruby)

### GraphQL APIs
- Apollo Server
- GraphQL Yoga
- Strawberry, Ariadne (Python)
- gqlgen (Go)

## Documentation Process

### Step 1: Discover Endpoints

Scan the codebase for API definitions:

```bash
# Find route definitions (Node.js/Express)
grep -r "app\.\(get\|post\|put\|patch\|delete\)" --include="*.js" --include="*.ts"

# Find FastAPI routes (Python)
grep -r "@app\.\(get\|post\|put\|patch\|delete\)" --include="*.py"

# Find Spring mappings (Java)
grep -r "@\(Get\|Post\|Put\|Delete\)Mapping" --include="*.java"

# Find GraphQL resolvers
grep -r "Query\|Mutation\|Subscription" --include="*.graphql"
```

### Step 2: Extract Route Information

For each endpoint, gather:
- HTTP method (GET, POST, PUT, PATCH, DELETE)
- Path with parameters (`/users/:id` or `/users/{id}`)
- Request body schema
- Query parameters
- Path parameters
- Headers (especially auth)
- Response schemas (success and error)

### Step 3: Generate Schema Definitions

Analyze types, interfaces, and models:
- TypeScript interfaces
- Python dataclasses/Pydantic models
- Java DTOs
- Database models
- Validation schemas (Zod, Joi, etc.)

## OpenAPI 3.0 Output Format

```yaml
openapi: 3.0.3
info:
  title: API Name
  description: |
    Brief description of the API.

    ## Authentication
    Describe auth method here.
  version: 1.0.0
  contact:
    name: API Support
    email: support@example.com

servers:
  - url: https://api.example.com/v1
    description: Production
  - url: https://staging-api.example.com/v1
    description: Staging

tags:
  - name: users
    description: User management operations
  - name: auth
    description: Authentication endpoints

paths:
  /users:
    get:
      tags:
        - users
      summary: List all users
      description: Retrieve a paginated list of users
      operationId: listUsers
      parameters:
        - name: page
          in: query
          description: Page number
          schema:
            type: integer
            default: 1
        - name: limit
          in: query
          description: Items per page
          schema:
            type: integer
            default: 20
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
      tags:
        - users
      summary: Create a new user
      operationId: createUser
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
        '400':
          $ref: '#/components/responses/BadRequest'
        '409':
          description: User already exists

  /users/{id}:
    get:
      tags:
        - users
      summary: Get user by ID
      operationId: getUserById
      parameters:
        - name: id
          in: path
          required: true
          description: User ID
          schema:
            type: string
            format: uuid
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '404':
          $ref: '#/components/responses/NotFound'

components:
  schemas:
    User:
      type: object
      required:
        - id
        - email
        - createdAt
      properties:
        id:
          type: string
          format: uuid
          example: "123e4567-e89b-12d3-a456-426614174000"
        email:
          type: string
          format: email
          example: "user@example.com"
        name:
          type: string
          example: "John Doe"
        createdAt:
          type: string
          format: date-time
        updatedAt:
          type: string
          format: date-time

    CreateUserRequest:
      type: object
      required:
        - email
        - password
      properties:
        email:
          type: string
          format: email
        password:
          type: string
          format: password
          minLength: 8
        name:
          type: string

    UserList:
      type: object
      properties:
        data:
          type: array
          items:
            $ref: '#/components/schemas/User'
        pagination:
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
        code:
          type: string
        message:
          type: string
        details:
          type: object

  responses:
    BadRequest:
      description: Bad request
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
    Unauthorized:
      description: Authentication required
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
    NotFound:
      description: Resource not found
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'

  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
    apiKey:
      type: apiKey
      in: header
      name: X-API-Key

security:
  - bearerAuth: []
```

## GraphQL Schema Documentation

For GraphQL APIs, generate SDL with descriptions:

```graphql
"""
User account information
"""
type User {
  """Unique identifier"""
  id: ID!

  """User's email address"""
  email: String!

  """Display name"""
  name: String

  """Account creation timestamp"""
  createdAt: DateTime!

  """User's posts"""
  posts(first: Int = 10, after: String): PostConnection!
}

"""
Paginated post results
"""
type PostConnection {
  edges: [PostEdge!]!
  pageInfo: PageInfo!
}

type Query {
  """Get current authenticated user"""
  me: User

  """Get user by ID"""
  user(id: ID!): User

  """List users with pagination"""
  users(first: Int = 20, after: String): UserConnection!
}

type Mutation {
  """Create a new user account"""
  createUser(input: CreateUserInput!): User!

  """Update user profile"""
  updateUser(id: ID!, input: UpdateUserInput!): User!

  """Delete user account"""
  deleteUser(id: ID!): Boolean!
}

input CreateUserInput {
  email: String!
  password: String!
  name: String
}
```

## Output Deliverables

When generating API documentation, provide:

1. **OpenAPI Spec** (YAML format) - Complete specification file
2. **Endpoint Summary** - Quick reference table
3. **Schema Definitions** - All data models
4. **Authentication Guide** - How to authenticate
5. **Error Codes** - Common error responses

### Endpoint Summary Table

| Method | Path | Description | Auth |
|--------|------|-------------|------|
| GET | /users | List all users | Bearer |
| POST | /users | Create user | Bearer |
| GET | /users/{id} | Get user by ID | Bearer |
| PUT | /users/{id} | Update user | Bearer |
| DELETE | /users/{id} | Delete user | Bearer |

## Tips

1. **Use consistent naming**: camelCase for JSON, snake_case for Python
2. **Document all status codes**: Include error responses
3. **Add examples**: Real example values help consumers
4. **Version your API**: Include version in path or header
5. **Keep schemas DRY**: Use $ref for reusable components
