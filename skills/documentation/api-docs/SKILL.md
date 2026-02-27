---
name: api-docs
description: Expert API documentation including OpenAPI specs, endpoint documentation, and SDK guides
version: 1.0.0
author: USER
tags: [api-documentation, openapi, swagger, endpoints, sdk]
---

# API Docs

## Purpose
Create comprehensive API documentation including OpenAPI/Swagger specs, endpoint references, authentication guides, and SDK documentation.

## Activation Keywords
- api documentation, api docs
- openapi, swagger
- endpoint documentation
- rest api docs, graphql docs
- sdk documentation

## Core Capabilities

### 1. OpenAPI Specification
- Full spec generation
- Schema definitions
- Example requests/responses
- Security schemes
- Versioning

### 2. Endpoint Documentation
- Method descriptions
- Parameter details
- Response formats
- Error codes
- Rate limits

### 3. Authentication Guides
- Auth flow documentation
- Token management
- Scope explanations
- Security best practices
- Example implementations

### 4. SDK Documentation
- Installation guides
- Quick start examples
- Method references
- Error handling
- Best practices

### 5. Interactive Examples
- cURL commands
- Language-specific examples
- Postman collections
- Live API playground
- Response samples

## OpenAPI Template

```yaml
openapi: 3.0.3
info:
  title: [API Name]
  description: |
    [API Description]

    ## Authentication
    [Auth description]

    ## Rate Limits
    [Rate limit info]
  version: 1.0.0
  contact:
    email: api@example.com

servers:
  - url: https://api.example.com/v1
    description: Production
  - url: https://staging-api.example.com/v1
    description: Staging

security:
  - bearerAuth: []

paths:
  /resource:
    get:
      summary: List resources
      description: Returns a paginated list of resources
      tags:
        - Resources
      parameters:
        - name: limit
          in: query
          schema:
            type: integer
            default: 20
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ResourceList'

components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
  schemas:
    Resource:
      type: object
      properties:
        id:
          type: string
        name:
          type: string
```

## Endpoint Documentation Format

```markdown
## Create User

Creates a new user account.

### Endpoint
```
POST /api/v1/users
```

### Authentication
Requires `admin` scope.

### Request Headers
| Header | Required | Description |
|--------|----------|-------------|
| Authorization | Yes | Bearer token |
| Content-Type | Yes | application/json |

### Request Body
```json
{
  "email": "user@example.com",
  "name": "John Doe",
  "role": "member"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| email | string | Yes | Valid email address |
| name | string | Yes | User's full name |
| role | string | No | One of: admin, member |

### Response

#### Success (201 Created)
```json
{
  "id": "usr_abc123",
  "email": "user@example.com",
  "name": "John Doe",
  "role": "member",
  "created_at": "2024-01-15T10:30:00Z"
}
```

#### Errors
| Status | Code | Description |
|--------|------|-------------|
| 400 | INVALID_EMAIL | Email format invalid |
| 409 | EMAIL_EXISTS | Email already registered |
| 401 | UNAUTHORIZED | Invalid or missing token |

### Example

```bash
curl -X POST https://api.example.com/v1/users \
  -H "Authorization: Bearer sk_live_xxx" \
  -H "Content-Type: application/json" \
  -d '{"email": "user@example.com", "name": "John"}'
```
```

## Example Usage

```
User: "Document our /users endpoint"

API Docs Response:

## Users API

### List Users
```
GET /api/v1/users
```

Returns a paginated list of users.

#### Query Parameters
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| page | integer | 1 | Page number |
| limit | integer | 20 | Items per page (max 100) |
| status | string | all | Filter: active, inactive, all |

#### Response (200 OK)
```json
{
  "data": [
    {
      "id": "usr_abc123",
      "email": "user@example.com",
      "name": "John Doe",
      "status": "active",
      "created_at": "2024-01-15T10:30:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 150,
    "pages": 8
  }
}
```

#### cURL Example
```bash
curl https://api.example.com/v1/users?limit=10 \
  -H "Authorization: Bearer $API_KEY"
```
```
