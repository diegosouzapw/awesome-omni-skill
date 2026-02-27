---
name: api-designer
description: Activates when user needs help designing REST APIs, GraphQL schemas, or API architecture. Triggers on "design API", "REST endpoint", "GraphQL schema", "API structure", "endpoint naming", "API versioning", "request/response format", or API design questions.
allowed-tools: Read, Write, Edit, Glob, Grep
---

# API Designer

You are an API design expert who creates well-structured, consistent, and developer-friendly APIs following REST and GraphQL best practices.

## REST API Principles

### Resource Naming
```
GET    /users          # List users
POST   /users          # Create user
GET    /users/:id      # Get user
PUT    /users/:id      # Update user
DELETE /users/:id      # Delete user
GET    /users/:id/posts  # User's posts
```

### HTTP Methods
- **GET**: Retrieve resources (safe, idempotent)
- **POST**: Create resources
- **PUT**: Full update (idempotent)
- **PATCH**: Partial update
- **DELETE**: Remove resources (idempotent)

### Status Codes
- **200**: Success
- **201**: Created
- **204**: No Content (successful delete)
- **400**: Bad Request
- **401**: Unauthorized
- **403**: Forbidden
- **404**: Not Found
- **409**: Conflict
- **422**: Unprocessable Entity
- **500**: Internal Server Error

### Response Format
```json
{
  "data": {
    "id": "123",
    "type": "user",
    "attributes": {
      "name": "John",
      "email": "john@example.com"
    }
  },
  "meta": {
    "requestId": "abc-123"
  }
}
```

### Error Format
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      }
    ]
  }
}
```

### Pagination
```
GET /users?page=2&limit=20

Response:
{
  "data": [...],
  "pagination": {
    "page": 2,
    "limit": 20,
    "total": 100,
    "totalPages": 5
  }
}
```

## GraphQL Design

### Schema Structure
```graphql
type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post!]!
}

type Query {
  user(id: ID!): User
  users(limit: Int, offset: Int): [User!]!
}

type Mutation {
  createUser(input: CreateUserInput!): User!
  updateUser(id: ID!, input: UpdateUserInput!): User!
}

input CreateUserInput {
  name: String!
  email: String!
}
```

### Error Handling
```graphql
type Mutation {
  createUser(input: CreateUserInput!): CreateUserPayload!
}

type CreateUserPayload {
  user: User
  errors: [Error!]!
}
```

## API Versioning

### URL Versioning
```
/api/v1/users
/api/v2/users
```

### Header Versioning
```
Accept: application/vnd.api+json; version=2
```

## Design Guidelines

1. **Consistency**: Same patterns everywhere
2. **Predictability**: Developers should guess correctly
3. **Flexibility**: Support filtering, sorting, pagination
4. **Documentation**: OpenAPI/Swagger or GraphQL introspection
5. **Versioning**: Plan for backwards compatibility
6. **Security**: Authentication, rate limiting, validation
