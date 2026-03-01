---
name: api-best-practices
description: REST API design patterns, OpenAPI specifications, versioning strategies, authentication, error handling, and security best practices. Use when designing APIs, creating endpoints, documenting APIs, or implementing backend services that expose HTTP APIs.
---

# API Best Practices

This skill provides comprehensive guidance for designing, implementing, and documenting RESTful APIs following industry best practices.

## RESTful Design Principles

### Resource-Oriented Design

APIs should be designed around resources (nouns), not actions (verbs):

**Good**:
```http
GET    /api/v1/users
POST   /api/v1/users
GET    /api/v1/users/{id}
PUT    /api/v1/users/{id}
DELETE /api/v1/users/{id}
```

**Bad**:
```http
GET    /api/v1/getUsers
POST   /api/v1/createUser
POST   /api/v1/updateUser
POST   /api/v1/deleteUser
```

### HTTP Methods and Their Meanings

- **GET**: Retrieve a resource (safe, idempotent, cacheable)
- **POST**: Create a new resource (not idempotent)
- **PUT**: Replace entire resource (idempotent)
- **PATCH**: Partial update (not necessarily idempotent)
- **DELETE**: Remove a resource (idempotent)

### HTTP Status Codes

**Success (2xx)**:
- `200 OK`: Successful GET, PUT, PATCH, DELETE
- `201 Created`: Successful POST with resource creation
- `202 Accepted`: Request accepted for async processing
- `204 No Content`: Successful DELETE or update with no response body

**Client Errors (4xx)**:
- `400 Bad Request`: Malformed request, validation error
- `401 Unauthorized`: Authentication required
- `403 Forbidden`: Authenticated but not authorized
- `404 Not Found`: Resource doesn't exist
- `409 Conflict`: Resource conflict (duplicate, version mismatch)
- `422 Unprocessable Entity`: Valid syntax but semantic errors
- `429 Too Many Requests`: Rate limit exceeded

**Server Errors (5xx)**:
- `500 Internal Server Error`: Unexpected server error
- `502 Bad Gateway`: Upstream service failure
- `503 Service Unavailable`: Temporary overload or maintenance
- `504 Gateway Timeout`: Upstream timeout

## API Versioning

### URL Versioning (Recommended)
```http
GET /api/v1/users
GET /api/v2/users
```

**Pros**: Clear, easy to route, visible in logs
**Cons**: Duplicate code across versions

### Header Versioning
```http
GET /api/users
Accept: application/vnd.myapi.v1+json
```

**Pros**: Clean URLs
**Cons**: Harder to test, less visible

### Version Management Rules
1. Never break backwards compatibility in same version
2. Deprecate old versions with advance notice (6-12 months)
3. Document migration guides between versions
4. Support at least 2 major versions simultaneously

## Request/Response Patterns

### Standard Request Format

**JSON Request Body**:
```json
{
  "email": "user@example.com",
  "name": "John Doe",
  "preferences": {
    "newsletter": true,
    "notifications": false
  }
}
```

**Query Parameters** (for filtering, pagination, sorting):
```http
GET /api/v1/users?role=admin&status=active&page=2&limit=20&sort=-created_at
```

### Standard Response Format

**Success Response**:
```json
{
  "data": {
    "id": "user_123",
    "email": "user@example.com",
    "name": "John Doe",
    "createdAt": "2025-10-16T10:30:00Z"
  }
}
```

**Error Response**:
```json
{
  "error": {
    "code": "INVALID_EMAIL",
    "message": "Email address is invalid",
    "field": "email",
    "details": "Email must contain @ symbol"
  }
}
```

**Collection Response with Pagination**:
```json
{
  "data": [
    { "id": 1, "name": "User 1" },
    { "id": 2, "name": "User 2" }
  ],
  "pagination": {
    "page": 2,
    "limit": 20,
    "total": 156,
    "totalPages": 8,
    "hasNext": true,
    "hasPrev": true
  },
  "links": {
    "self": "/api/v1/users?page=2",
    "next": "/api/v1/users?page=3",
    "prev": "/api/v1/users?page=1",
    "first": "/api/v1/users?page=1",
    "last": "/api/v1/users?page=8"
  }
}
```

## Authentication Patterns

### JWT (JSON Web Tokens)

**Login Flow**:
```http
POST /api/v1/auth/login
{
  "email": "user@example.com",
  "password": "SecurePassword123"
}

Response (200):
{
  "accessToken": "eyJhbGc...",
  "refreshToken": "eyJhbGc...",
  "expiresIn": 900
}
```

**Using Access Token**:
```http
GET /api/v1/users/me
Authorization: Bearer eyJhbGc...
```

**Token Refresh**:
```http
POST /api/v1/auth/refresh
{
  "refreshToken": "eyJhbGc..."
}

Response (200):
{
  "accessToken": "eyJhbGc...",
  "expiresIn": 900
}
```

### API Keys

**Header-based** (recommended):
```http
GET /api/v1/data
X-API-Key: sk_live_abc123xyz
```

**Query parameter** (less secure, use only for public data):
```http
GET /api/v1/public-data?api_key=sk_live_abc123xyz
```

### OAuth 2.0 Flows

**Authorization Code Flow** (for web apps):
1. Redirect to `/oauth/authorize`
2. User grants permission
3. Receive authorization code
4. Exchange code for access token at `/oauth/token`
5. Use access token for API requests

**Client Credentials Flow** (for server-to-server):
```http
POST /oauth/token
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&client_id=abc&client_secret=xyz
```

## Error Handling

### Validation Errors

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "errors": [
      {
        "field": "email",
        "message": "Email is required"
      },
      {
        "field": "age",
        "message": "Age must be at least 18"
      }
    ]
  }
}
```

### Business Logic Errors

```json
{
  "error": {
    "code": "INSUFFICIENT_FUNDS",
    "message": "Account balance too low for this transaction",
    "details": {
      "balance": 50.00,
      "required": 100.00,
      "currency": "USD"
    }
  }
}
```

### Rate Limiting Errors

```http
HTTP/1.1 429 Too Many Requests
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1634400000
Retry-After: 3600

{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "API rate limit exceeded",
    "retryAfter": 3600
  }
}
```

## Pagination Strategies

### Offset Pagination (Simple)
```http
GET /api/v1/users?offset=40&limit=20
```

**Pros**: Simple, allows jumping to any page
**Cons**: Performance degrades with large offsets, inconsistent if data changes

### Cursor Pagination (Recommended for large datasets)
```http
GET /api/v1/users?cursor=eyJpZCI6MTIzfQ&limit=20

Response:
{
  "data": [...],
  "pagination": {
    "nextCursor": "eyJpZCI6MTQzfQ",
    "hasMore": true
  }
}
```

**Pros**: Consistent results, performant at any scale
**Cons**: Can't jump to specific page

### Page-Number Pagination (User-friendly)
```http
GET /api/v1/users?page=3&limit=20
```

**Pros**: User-friendly, easy to understand
**Cons**: Same issues as offset pagination

## Rate Limiting

### Implementation Pattern

**Headers to include**:
```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1634400000
```

**Tiered Limits**:
- Anonymous: 100 requests/hour
- Basic tier: 1,000 requests/hour
- Pro tier: 10,000 requests/hour
- Enterprise: Custom limits

### Rate Limiting Algorithms

**Token Bucket** (recommended):
- Allows bursts
- Smooth long-term rate
- Most flexible

**Fixed Window**:
- Simple to implement
- Can allow double limit at window boundaries
- Less flexible

**Sliding Window**:
- More accurate than fixed window
- More complex to implement
- Better user experience

## API Security Best Practices

### 1. Always Use HTTPS
Never send sensitive data over HTTP. Enforce HTTPS at the load balancer level.

### 2. Validate All Inputs
```python
from pydantic import BaseModel, EmailStr, constr

class UserCreate(BaseModel):
    email: EmailStr
    password: constr(min_length=8, max_length=100)
    name: constr(min_length=1, max_length=100)
```

### 3. Sanitize Outputs
Prevent injection attacks by escaping output:
```python
import html
safe_output = html.escape(user_input)
```

### 4. Use Parameterized Queries
```python
# ✅ SAFE - Parameterized
cursor.execute("SELECT * FROM users WHERE email = ?", (email,))

# ❌ UNSAFE - String concatenation
cursor.execute(f"SELECT * FROM users WHERE email = '{email}'")
```

### 5. Implement CORS Properly
```python
# Be specific with origins
CORS(app, origins=["https://myapp.com", "https://app.myapp.com"])

# ❌ NEVER use wildcard in production
# CORS(app, origins=["*"])  # DANGEROUS
```

### 6. Authenticate Before Authorization
```python
# 1. Verify JWT token (authentication)
# 2. Check user permissions (authorization)
# 3. Process request
```

### 7. Log Security Events
```python
logger.warning(f"Failed login attempt for {email} from {ip_address}")
logger.critical(f"Privilege escalation attempt by user {user_id}")
```

### 8. Rate Limit Authentication Endpoints
Prevent brute force attacks:
- `/auth/login`: 5 attempts per 15 minutes per IP
- `/auth/register`: 3 attempts per hour per IP
- `/auth/reset-password`: 3 attempts per hour per email

## OpenAPI/Swagger Documentation

### OpenAPI 3.0 Example

```yaml
openapi: 3.0.0
info:
  title: My API
  version: 1.0.0
  description: API for managing users and posts

servers:
  - url: https://api.example.com/v1
    description: Production server
  - url: https://staging-api.example.com/v1
    description: Staging server

paths:
  /users:
    get:
      summary: List users
      operationId: listUsers
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
          description: Successful response
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items:
                      $ref: '#/components/schemas/User'
                  pagination:
                    $ref: '#/components/schemas/Pagination'
    post:
      summary: Create user
      operationId: createUser
      tags:
        - Users
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/UserCreate'
      responses:
        '201':
          description: User created
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '400':
          description: Validation error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'

components:
  schemas:
    User:
      type: object
      required:
        - id
        - email
        - name
      properties:
        id:
          type: string
          example: user_123
        email:
          type: string
          format: email
          example: user@example.com
        name:
          type: string
          example: John Doe
        createdAt:
          type: string
          format: date-time
          example: 2025-10-16T10:30:00Z

    UserCreate:
      type: object
      required:
        - email
        - password
        - name
      properties:
        email:
          type: string
          format: email
        password:
          type: string
          minLength: 8
          maxLength: 100
        name:
          type: string
          minLength: 1
          maxLength: 100

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
        hasNext:
          type: boolean
        hasPrev:
          type: boolean

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
              type: object

  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

security:
  - BearerAuth: []
```

## API Endpoint Design Patterns

### Collection and Resource Endpoints

```http
# Collection operations
GET    /api/v1/posts          # List posts
POST   /api/v1/posts          # Create post
GET    /api/v1/posts/{id}     # Get specific post
PUT    /api/v1/posts/{id}     # Replace post
PATCH  /api/v1/posts/{id}     # Update post
DELETE /api/v1/posts/{id}     # Delete post

# Nested resources
GET    /api/v1/posts/{id}/comments     # List comments for post
POST   /api/v1/posts/{id}/comments     # Create comment on post
GET    /api/v1/comments/{id}           # Get specific comment
DELETE /api/v1/comments/{id}           # Delete comment
```

### Action Endpoints (When REST Isn't Enough)

Sometimes you need RPC-style endpoints for actions:

```http
POST /api/v1/users/{id}/verify-email
POST /api/v1/orders/{id}/cancel
POST /api/v1/posts/{id}/publish
POST /api/v1/invoices/{id}/send
```

**Pattern**: `POST /{resource}/{id}/{action}`

Use when:
- Action doesn't fit CRUD model
- State transitions need to be explicit
- Business logic requires specific endpoint

## Request Validation

### Input Validation Pattern

```python
from pydantic import BaseModel, EmailStr, Field, validator

class UserCreate(BaseModel):
    email: EmailStr
    password: str = Field(min_length=8, max_length=100)
    name: str = Field(min_length=1, max_length=100)
    age: int = Field(ge=18, le=120)

    @validator('password')
    def password_strength(cls, v):
        if not any(c.isupper() for c in v):
            raise ValueError('Password must contain uppercase letter')
        if not any(c.isdigit() for c in v):
            raise ValueError('Password must contain digit')
        return v
```

### Validation Error Response

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "errors": [
      {
        "field": "email",
        "message": "Email is required",
        "code": "REQUIRED_FIELD"
      },
      {
        "field": "password",
        "message": "Password must contain uppercase letter",
        "code": "INVALID_FORMAT"
      }
    ]
  }
}
```

## Filtering, Sorting, Searching

### Filtering
```http
# Single filter
GET /api/v1/posts?status=published

# Multiple filters (AND)
GET /api/v1/posts?status=published&author=john

# Multiple values (OR)
GET /api/v1/posts?tags=tech,ai,ml

# Range filters
GET /api/v1/posts?created_after=2025-01-01&created_before=2025-12-31
```

### Sorting
```http
# Single field ascending
GET /api/v1/posts?sort=created_at

# Single field descending
GET /api/v1/posts?sort=-created_at

# Multiple fields
GET /api/v1/posts?sort=-priority,created_at
```

### Searching
```http
# Full-text search
GET /api/v1/posts?q=machine+learning

# Field-specific search
GET /api/v1/posts?title=contains:machine&author=starts_with:john
```

## Idempotency

### Idempotent Operations (Safe to Retry)
- GET, PUT, DELETE: Always idempotent
- POST: Not idempotent by default

### Idempotency Keys for POST

```http
POST /api/v1/payments
Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000

{
  "amount": 100.00,
  "currency": "USD",
  "description": "Payment for order #123"
}
```

**Server stores idempotency key**:
- First request: Process and return 201
- Duplicate requests with same key: Return cached 201 response
- Different request with same key: Return 409 Conflict

## Async Operations

### Long-Running Tasks

```http
POST /api/v1/reports/generate
{
  "type": "annual_summary",
  "year": 2025
}

Response (202 Accepted):
{
  "id": "job_abc123",
  "status": "processing",
  "statusUrl": "/api/v1/jobs/job_abc123"
}
```

### Check Status

```http
GET /api/v1/jobs/job_abc123

Response:
{
  "id": "job_abc123",
  "status": "completed",
  "result": {
    "reportUrl": "/api/v1/reports/annual_summary_2025.pdf"
  },
  "createdAt": "2025-10-16T10:00:00Z",
  "completedAt": "2025-10-16T10:05:00Z"
}
```

**Status values**: `queued`, `processing`, `completed`, `failed`

## Webhooks

### Webhook Payload

```json
{
  "event": "user.created",
  "timestamp": "2025-10-16T10:30:00Z",
  "id": "evt_abc123",
  "data": {
    "id": "user_123",
    "email": "user@example.com",
    "name": "John Doe"
  }
}
```

### Webhook Security

**HMAC Signature**:
```http
POST https://customer.com/webhooks
X-Webhook-Signature: sha256=abc123...

# Verify signature
import hmac
import hashlib

def verify_webhook(payload, signature, secret):
    expected = hmac.new(
        secret.encode(),
        payload.encode(),
        hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(f"sha256={expected}", signature)
```

## API Performance Best Practices

### 1. Use ETags for Caching

```http
GET /api/v1/users/123
ETag: "33a64df551425fcc55e4d42a148795d9f25f89d4"

# Client sends If-None-Match on subsequent requests
GET /api/v1/users/123
If-None-Match: "33a64df551425fcc55e4d42a148795d9f25f89d4"

Response: 304 Not Modified (if unchanged)
```

### 2. Implement Field Selection

```http
# Get only specific fields
GET /api/v1/users/123?fields=id,email,name

Response:
{
  "id": "user_123",
  "email": "user@example.com",
  "name": "John Doe"
}
```

### 3. Use Compression

```http
Accept-Encoding: gzip, deflate
```

Server should compress responses >1KB.

### 4. Batch Operations

```http
# Instead of N individual requests
GET /api/v1/users/1
GET /api/v1/users/2
GET /api/v1/users/3

# Use batch endpoint
GET /api/v1/users?ids=1,2,3
```

### 5. Database Query Optimization

- Use database indexes on filter fields
- Limit result set size (max 100 items per page)
- Use connection pooling
- Implement query caching for expensive queries

## HATEOAS (Hypermedia)

### Including Links in Responses

```json
{
  "data": {
    "id": "user_123",
    "email": "user@example.com",
    "name": "John Doe"
  },
  "links": {
    "self": "/api/v1/users/123",
    "posts": "/api/v1/users/123/posts",
    "comments": "/api/v1/users/123/comments",
    "avatar": "/api/v1/users/123/avatar"
  }
}
```

**Benefits**:
- Self-documenting API
- Clients discover available actions
- API evolution easier

## Content Negotiation

### Request Format

```http
Content-Type: application/json
Accept: application/json
```

### Support Multiple Formats (Optional)

```http
# Request JSON
Accept: application/json

# Request XML
Accept: application/xml

# Request CSV
Accept: text/csv
```

## Deprecation Strategy

### Announce Deprecation

```http
GET /api/v1/old-endpoint
Sunset: Sat, 31 Dec 2025 23:59:59 GMT
Deprecation: Tue, 1 Oct 2025 00:00:00 GMT
Link: </api/v2/new-endpoint>; rel="alternate"
```

### Migration Guide

Provide clear migration path:
1. Announce deprecation 6-12 months in advance
2. Provide migration guide with code examples
3. Support old and new versions simultaneously
4. Monitor usage of deprecated endpoints
5. Send email notifications to API consumers
6. Finally remove deprecated endpoint

## API Health and Status

### Health Check Endpoint

```http
GET /health

Response (200):
{
  "status": "healthy",
  "version": "1.2.3",
  "timestamp": "2025-10-16T10:30:00Z"
}
```

### Readiness Check (Dependencies)

```http
GET /health/ready

Response (200):
{
  "status": "ready",
  "checks": {
    "database": "ok",
    "cache": "ok",
    "messageQueue": "ok",
    "externalAPI": "ok"
  }
}

Response (503) if any dependency fails:
{
  "status": "not_ready",
  "checks": {
    "database": "ok",
    "cache": "degraded",
    "messageQueue": "failed"
  }
}
```

## Testing APIs

### Unit Testing Controllers

```python
def test_create_user():
    response = client.post("/api/v1/users", json={
        "email": "test@example.com",
        "password": "SecurePass123",
        "name": "Test User"
    })

    assert response.status_code == 201
    assert response.json()["email"] == "test@example.com"
    assert "password" not in response.json()  # Never return passwords
```

### Integration Testing

```python
def test_user_flow():
    # Create user
    response = client.post("/api/v1/users", json=user_data)
    user_id = response.json()["id"]

    # Login
    response = client.post("/api/v1/auth/login", json={
        "email": user_data["email"],
        "password": user_data["password"]
    })
    token = response.json()["accessToken"]

    # Access protected resource
    response = client.get(
        f"/api/v1/users/{user_id}",
        headers={"Authorization": f"Bearer {token}"}
    )
    assert response.status_code == 200
```

## Common API Mistakes to Avoid

1. **Using GET for state changes**: GET should be safe and idempotent
2. **Returning sensitive data**: Never return passwords, tokens, secrets
3. **Inconsistent naming**: Stick to camelCase or snake_case, not both
4. **Missing error details**: Provide helpful error messages
5. **No rate limiting**: Always implement rate limits
6. **Exposing internal IDs**: Use UUIDs or slugs for public APIs
7. **No versioning**: Always version from day one
8. **Ignoring CORS**: Configure properly for web clients
9. **Poor pagination**: Implement cursor-based for large datasets
10. **No documentation**: Always provide OpenAPI docs

## When to Use This Skill

Use this skill when:
- Designing new API endpoints
- Implementing REST APIs
- Reviewing API code
- Creating API documentation
- Troubleshooting API issues
- Discussing authentication/authorization
- Planning API versioning strategy
- Implementing rate limiting
- Handling errors in APIs

---

**Remember**: A well-designed API is intuitive, secure, performant, and well-documented. Follow these patterns to create APIs that developers love to use.
