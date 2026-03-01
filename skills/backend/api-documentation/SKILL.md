---
name: api-documentation
description: API documentation standards and patterns
---

# API Documentation Skill

Standards for documenting REST and GraphQL APIs.

## REST API Documentation

### Endpoint Format

```markdown
## Endpoint Name

Brief description of what this endpoint does.

**Method:** `GET` | `POST` | `PUT` | `PATCH` | `DELETE`
**Path:** `/api/v1/resource/:id`
**Auth:** Bearer token | API key | None

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string | Resource ID |

### Query Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| page | integer | No | 1 | Page number |
| limit | integer | No | 20 | Items per page |
| sort | string | No | -createdAt | Sort field |

### Request Body

\`\`\`json
{
  "field1": "value",
  "field2": 123
}
\`\`\`

### Response

**Success (200 OK)**

\`\`\`json
{
  "data": { ... },
  "meta": { ... }
}
\`\`\`

**Errors**

| Status | Code | Description |
|--------|------|-------------|
| 400 | VALIDATION_ERROR | Invalid input |
| 404 | NOT_FOUND | Resource not found |
```

### Authentication Section

```markdown
# Authentication

All API requests require authentication via one of:

## Bearer Token (Recommended)

\`\`\`bash
curl -H "Authorization: Bearer <token>" https://api.example.com/v1/users
\`\`\`

Tokens expire after 1 hour. Use the refresh token to obtain a new access token.

## API Key

\`\`\`bash
curl -H "X-API-Key: <api-key>" https://api.example.com/v1/users
\`\`\`

API keys don't expire but can be revoked in the dashboard.

## OAuth 2.0

For third-party integrations:

1. Redirect to `/oauth/authorize`
2. User grants permission
3. Receive authorization code
4. Exchange code for tokens
```

### Pagination Documentation

```markdown
# Pagination

List endpoints return paginated results.

## Request Parameters

| Parameter | Type | Default | Max | Description |
|-----------|------|---------|-----|-------------|
| page | integer | 1 | - | Page number (1-indexed) |
| limit | integer | 20 | 100 | Items per page |

## Response Format

\`\`\`json
{
  "data": [...],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 150,
    "totalPages": 8,
    "hasNext": true,
    "hasPrev": false
  }
}
\`\`\`

## Cursor-Based Pagination

For large datasets, use cursor pagination:

\`\`\`bash
GET /api/v1/events?cursor=abc123&limit=50
\`\`\`

\`\`\`json
{
  "data": [...],
  "cursors": {
    "next": "def456",
    "prev": null
  }
}
\`\`\`
```

### Error Documentation

```markdown
# Error Handling

## Error Response Format

All errors return a consistent JSON structure:

\`\`\`json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable message",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      }
    ],
    "requestId": "req_abc123"
  }
}
\`\`\`

## Error Codes

### Client Errors (4xx)

| Code | Status | Description | Resolution |
|------|--------|-------------|------------|
| VALIDATION_ERROR | 400 | Invalid input | Check request body |
| UNAUTHORIZED | 401 | No valid credentials | Include auth header |
| FORBIDDEN | 403 | Insufficient permissions | Request access |
| NOT_FOUND | 404 | Resource doesn't exist | Check resource ID |
| CONFLICT | 409 | Resource already exists | Use different values |
| RATE_LIMITED | 429 | Too many requests | Wait and retry |

### Server Errors (5xx)

| Code | Status | Description | Resolution |
|------|--------|-------------|------------|
| INTERNAL_ERROR | 500 | Server error | Contact support |
| SERVICE_UNAVAILABLE | 503 | Maintenance | Retry later |
```

## GraphQL Documentation

### Schema Documentation

```markdown
# GraphQL Schema

## Types

### User

\`\`\`graphql
type User {
  """Unique identifier"""
  id: ID!

  """User's email address"""
  email: String!

  """Display name"""
  name: String

  """Account creation timestamp"""
  createdAt: DateTime!

  """User's orders"""
  orders(first: Int, after: String): OrderConnection!
}
\`\`\`

### Input Types

\`\`\`graphql
input CreateUserInput {
  email: String!
  name: String
  role: UserRole = USER
}
\`\`\`
```

### Query Documentation

```markdown
## Queries

### user

Fetch a single user by ID.

\`\`\`graphql
query GetUser($id: ID!) {
  user(id: $id) {
    id
    email
    name
    createdAt
  }
}
\`\`\`

**Arguments:**

| Argument | Type | Required | Description |
|----------|------|----------|-------------|
| id | ID | Yes | User ID |

**Example:**

\`\`\`json
{
  "id": "usr_123"
}
\`\`\`

**Response:**

\`\`\`json
{
  "data": {
    "user": {
      "id": "usr_123",
      "email": "john@example.com",
      "name": "John Doe",
      "createdAt": "2024-01-15T10:30:00Z"
    }
  }
}
\`\`\`
```

### Mutation Documentation

```markdown
## Mutations

### createUser

Create a new user account.

\`\`\`graphql
mutation CreateUser($input: CreateUserInput!) {
  createUser(input: $input) {
    user {
      id
      email
    }
    errors {
      field
      message
    }
  }
}
\`\`\`

**Input:**

\`\`\`json
{
  "input": {
    "email": "jane@example.com",
    "name": "Jane Smith"
  }
}
\`\`\`

**Success Response:**

\`\`\`json
{
  "data": {
    "createUser": {
      "user": {
        "id": "usr_456",
        "email": "jane@example.com"
      },
      "errors": null
    }
  }
}
\`\`\`

**Error Response:**

\`\`\`json
{
  "data": {
    "createUser": {
      "user": null,
      "errors": [
        {
          "field": "email",
          "message": "Email already exists"
        }
      ]
    }
  }
}
\`\`\`
```

## SDK Examples

### Language-Specific Examples

```markdown
## SDK Examples

### JavaScript/TypeScript

\`\`\`typescript
import { Client } from '@api/sdk'

const client = new Client({ apiKey: 'your-key' })

// List users
const users = await client.users.list({ limit: 10 })

// Create user
const user = await client.users.create({
  email: 'john@example.com',
  name: 'John Doe'
})
\`\`\`

### Python

\`\`\`python
from api_sdk import Client

client = Client(api_key='your-key')

# List users
users = client.users.list(limit=10)

# Create user
user = client.users.create(
    email='john@example.com',
    name='John Doe'
)
\`\`\`

### cURL

\`\`\`bash
# List users
curl -X GET "https://api.example.com/v1/users?limit=10" \
  -H "Authorization: Bearer $API_KEY"

# Create user
curl -X POST "https://api.example.com/v1/users" \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"email":"john@example.com","name":"John Doe"}'
\`\`\`
```

## Integration

Used by:
- `api-doc-writer` agent
