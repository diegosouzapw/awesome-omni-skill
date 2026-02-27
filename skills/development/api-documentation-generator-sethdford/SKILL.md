---
name: API Documentation Generator
description: Generate comprehensive API documentation for Confluence from code, OpenAPI specs, or endpoints. Use when documenting REST APIs, GraphQL APIs, or any API endpoints. Creates structured documentation with examples, parameters, responses, and error codes.
allowed-tools: Read, Grep, Bash
---

# API Documentation Generator

Expert assistance for creating comprehensive API documentation in Confluence.

## When to Use This Skill

- Documenting new API endpoints
- Creating API reference pages
- Converting OpenAPI/Swagger specs to Confluence
- User mentions: API, endpoints, REST, GraphQL, documentation
- After implementing new API features

## API Documentation Structure

### Complete API Page Template

```markdown
# [API Name] API

## Overview
Brief description of what this API does and its purpose.

## Base URL
```
https://api.example.com/v1
```

## Authentication
How to authenticate with this API.

## Endpoints

### GET /resource
Brief description of what this endpoint does.

#### Parameters
[Parameter table]

#### Request Example
[Code block with example]

#### Response
[Success response example]

#### Error Codes
[Error table]

## Rate Limiting
API rate limit information.

## Changelog
Version history and changes.
```

## Endpoint Documentation

### Standard Sections

#### 1. Endpoint Header
```markdown
### POST /api/users
Create a new user account
```

#### 2. Description
```markdown
Creates a new user account with the provided information.
Sends a verification email to the user's address.

**Permissions**: Requires `admin` role
**Rate Limit**: 10 requests per minute
```

#### 3. Parameters Table

##### Path Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string | Yes | User ID (UUID format) |
| `version` | integer | No | API version (default: 1) |

##### Query Parameters
| Parameter | Type | Required | Description | Default |
|-----------|------|----------|-------------|---------|
| `page` | integer | No | Page number | 1 |
| `limit` | integer | No | Items per page | 20 |
| `sort` | string | No | Sort field | `created_at` |
| `order` | string | No | Sort order (asc/desc) | `desc` |

##### Request Body
| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `email` | string | Yes | User email address | Valid email format |
| `username` | string | Yes | Username | 3-20 alphanumeric chars |
| `password` | string | Yes | Password | Min 8 chars, 1 uppercase, 1 number |
| `full_name` | string | No | Full name | Max 100 chars |
| `role` | string | No | User role | One of: user, admin, moderator |

#### 4. Request Example

**cURL**:
```bash
curl -X POST https://api.example.com/v1/users \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "email": "user@example.com",
    "username": "johndoe",
    "password": "SecurePass123",
    "full_name": "John Doe",
    "role": "user"
  }'
```

**JavaScript (fetch)**:
```javascript
const response = await fetch('https://api.example.com/v1/users', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer YOUR_API_KEY'
  },
  body: JSON.stringify({
    email: 'user@example.com',
    username: 'johndoe',
    password: 'SecurePass123',
    full_name: 'John Doe',
    role: 'user'
  })
});

const data = await response.json();
console.log(data);
```

**Python (requests)**:
```python
import requests

url = "https://api.example.com/v1/users"
headers = {
    "Content-Type": "application/json",
    "Authorization": "Bearer YOUR_API_KEY"
}
data = {
    "email": "user@example.com",
    "username": "johndoe",
    "password": "SecurePass123",
    "full_name": "John Doe",
    "role": "user"
}

response = requests.post(url, headers=headers, json=data)
print(response.json())
```

#### 5. Response Examples

**Success Response (201 Created)**:
```json
{
  "status": "success",
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "email": "user@example.com",
    "username": "johndoe",
    "full_name": "John Doe",
    "role": "user",
    "email_verified": false,
    "created_at": "2024-01-15T10:30:00Z",
    "updated_at": "2024-01-15T10:30:00Z"
  }
}
```

**Response Fields**:
| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique user identifier (UUID) |
| `email` | string | User's email address |
| `username` | string | Username |
| `full_name` | string | User's full name |
| `role` | string | User's role |
| `email_verified` | boolean | Email verification status |
| `created_at` | string | Account creation timestamp (ISO 8601) |
| `updated_at` | string | Last update timestamp (ISO 8601) |

#### 6. Error Responses

| Status Code | Error Code | Description | Resolution |
|-------------|------------|-------------|------------|
| 400 | `INVALID_EMAIL` | Email format is invalid | Provide valid email address |
| 400 | `WEAK_PASSWORD` | Password doesn't meet requirements | Use min 8 chars, 1 uppercase, 1 number |
| 400 | `USERNAME_TAKEN` | Username already exists | Choose different username |
| 401 | `UNAUTHORIZED` | Missing or invalid API key | Include valid Authorization header |
| 403 | `FORBIDDEN` | Insufficient permissions | Requires admin role |
| 429 | `RATE_LIMIT_EXCEEDED` | Too many requests | Wait before retrying |
| 500 | `INTERNAL_ERROR` | Server error | Contact support if persists |

**Error Response Format**:
```json
{
  "status": "error",
  "error": {
    "code": "USERNAME_TAKEN",
    "message": "The username 'johndoe' is already in use",
    "details": {
      "field": "username",
      "value": "johndoe"
    }
  }
}
```

## Authentication Documentation

### API Key Authentication

```markdown
## Authentication

All API requests require authentication using an API key.

### Obtaining an API Key
1. Log in to your account
2. Navigate to Settings > API Keys
3. Click "Generate New Key"
4. Store the key securely (shown only once)

### Using the API Key

Include the API key in the `Authorization` header:

```
Authorization: Bearer YOUR_API_KEY
```

**Example**:
```bash
curl -H "Authorization: Bearer sk_test_abc123..." \
  https://api.example.com/v1/users
```

### Security Best Practices
- Never commit API keys to version control
- Rotate keys regularly (every 90 days)
- Use environment variables for key storage
- Different keys for development/production
```

### OAuth 2.0 Documentation

```markdown
## Authentication

This API uses OAuth 2.0 for authentication.

### Authorization Flow

1. **Redirect user to authorization URL**:
```
https://api.example.com/oauth/authorize?
  client_id=YOUR_CLIENT_ID&
  redirect_uri=YOUR_REDIRECT_URI&
  response_type=code&
  scope=read write
```

2. **User authorizes your app**

3. **Receive authorization code**:
```
https://your-redirect-uri?code=AUTH_CODE
```

4. **Exchange code for access token**:
```bash
curl -X POST https://api.example.com/oauth/token \
  -d "client_id=YOUR_CLIENT_ID" \
  -d "client_secret=YOUR_CLIENT_SECRET" \
  -d "code=AUTH_CODE" \
  -d "grant_type=authorization_code"
```

5. **Use access token in requests**:
```
Authorization: Bearer ACCESS_TOKEN
```

### Scopes

| Scope | Description |
|-------|-------------|
| `read` | Read access to resources |
| `write` | Create and update resources |
| `delete` | Delete resources |
| `admin` | Full administrative access |
```

## Rate Limiting Documentation

```markdown
## Rate Limiting

API requests are rate limited to ensure fair usage.

### Limits

| Tier | Requests per minute | Requests per day |
|------|-------------------|------------------|
| Free | 60 | 10,000 |
| Pro | 600 | 100,000 |
| Enterprise | Unlimited | Unlimited |

### Rate Limit Headers

Each response includes rate limit information:

```
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1642247400
```

| Header | Description |
|--------|-------------|
| `X-RateLimit-Limit` | Total requests allowed in window |
| `X-RateLimit-Remaining` | Requests remaining in window |
| `X-RateLimit-Reset` | Unix timestamp when limit resets |

### Handling Rate Limits

When rate limited, you'll receive a `429 Too Many Requests` response:

```json
{
  "status": "error",
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "API rate limit exceeded",
    "retry_after": 42
  }
}
```

**Best practices**:
- Monitor `X-RateLimit-Remaining` header
- Implement exponential backoff
- Cache responses when possible
- Use webhooks instead of polling
```

## Pagination Documentation

```markdown
## Pagination

List endpoints return paginated results.

### Request Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | integer | 1 | Page number (1-indexed) |
| `limit` | integer | 20 | Items per page (max 100) |

### Example Request

```bash
GET /api/users?page=2&limit=50
```

### Response Format

```json
{
  "data": [
    {...},
    {...}
  ],
  "pagination": {
    "page": 2,
    "limit": 50,
    "total_pages": 10,
    "total_items": 487,
    "has_next": true,
    "has_prev": true
  },
  "links": {
    "first": "/api/users?page=1&limit=50",
    "prev": "/api/users?page=1&limit=50",
    "next": "/api/users?page=3&limit=50",
    "last": "/api/users?page=10&limit=50"
  }
}
```
```

## From OpenAPI/Swagger Spec

### Converting OpenAPI to Confluence

When given an OpenAPI specification:

1. **Extract metadata**:
   - API title and version
   - Base URL
   - Contact information

2. **Parse endpoints**:
   - HTTP method and path
   - Summary and description
   - Parameters (path, query, body)
   - Response schemas
   - Status codes

3. **Generate examples**:
   - Request examples in multiple languages
   - Response examples with real data
   - Error examples

4. **Add documentation**:
   - Authentication requirements
   - Rate limiting
   - Versioning strategy

### Example: OpenAPI → Confluence

**OpenAPI Spec**:
```yaml
paths:
  /users/{id}:
    get:
      summary: Get user by ID
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: string
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '404':
          description: User not found
```

**Generated Confluence Page**:
```markdown
### GET /users/{id}
Retrieve user information by user ID.

#### Parameters

| Parameter | Type | Location | Required | Description |
|-----------|------|----------|----------|-------------|
| `id` | string | path | Yes | User ID |

#### Response (200 OK)
```json
{
  "id": "123",
  "username": "johndoe",
  "email": "john@example.com"
}
```

#### Errors
- **404 Not Found**: User with specified ID does not exist
```

## Code from Implementation

### Extracting API Docs from Code

When documenting from actual code:

1. **Identify endpoints**:
   - Search for route definitions
   - Extract HTTP methods and paths

2. **Parse parameters**:
   - Look for request validation
   - Find query/body parameter definitions

3. **Extract responses**:
   - Identify return statements
   - Find response status codes

4. **Add context**:
   - Code comments
   - Function documentation
   - Type definitions

### Example: Express.js → Confluence

**Code**:
```javascript
/**
 * Create a new user
 * @route POST /api/users
 * @param {string} email - User email
 * @param {string} username - Username
 * @returns {object} Created user object
 */
app.post('/api/users', async (req, res) => {
  const { email, username } = req.body;

  if (!email || !username) {
    return res.status(400).json({
      error: 'Email and username are required'
    });
  }

  const user = await db.users.create({ email, username });

  res.status(201).json({ data: user });
});
```

**Generated Documentation**:
```markdown
### POST /api/users
Create a new user account.

#### Request Body
```json
{
  "email": "user@example.com",
  "username": "johndoe"
}
```

#### Response (201 Created)
```json
{
  "data": {
    "id": "123",
    "email": "user@example.com",
    "username": "johndoe"
  }
}
```

#### Errors
- **400 Bad Request**: Email and username are required
```

## Best Practices

### 1. Clear Naming
- Use descriptive endpoint names
- Follow REST conventions (GET, POST, PUT, DELETE)
- Consistent resource naming

### 2. Complete Examples
- Show real, working examples
- Include authentication
- Cover common use cases
- Multiple languages (cURL, JavaScript, Python)

### 3. Error Documentation
- Document all possible errors
- Include error codes
- Explain how to resolve errors
- Show error response format

### 4. Versioning
- Document version in URL (`/v1/users`)
- Note deprecated endpoints
- Maintain changelog
- Migration guides between versions

### 5. Security
- Authentication requirements clear
- Note permissions needed
- Security best practices
- Data sensitivity warnings

## Confluence Organization

### API Documentation Structure

```
API Documentation Space
├── Overview
│   ├── Getting Started
│   ├── Authentication
│   ├── Rate Limiting
│   └── Versioning
├── Endpoints
│   ├── Users API
│   │   ├── GET /users
│   │   ├── POST /users
│   │   ├── GET /users/{id}
│   │   ├── PUT /users/{id}
│   │   └── DELETE /users/{id}
│   ├── Posts API
│   └── Comments API
├── Reference
│   ├── Error Codes
│   ├── Status Codes
│   └── Data Models
└── Guides
    ├── Quick Start
    ├── Common Patterns
    └── Best Practices
```

## How I'll Help

When creating API documentation, I will:

1. **Analyze the API**:
   - Read code or OpenAPI spec
   - Identify endpoints and methods
   - Extract parameters and responses

2. **Structure documentation**:
   - Create logical sections
   - Organize by resource or functionality
   - Add navigation and links

3. **Generate examples**:
   - Multiple programming languages
   - Real, working code
   - Common use cases

4. **Add details**:
   - Authentication requirements
   - Error handling
   - Rate limiting
   - Best practices

5. **Create in Confluence**:
   - Convert to ADF format
   - Use proper formatting
   - Add code highlighting
   - Create tables

6. **Maintain**:
   - Update when API changes
   - Add deprecation notices
   - Version documentation

## Example Interaction

**You**: "Document the new payment API I just implemented"

**Me**: "I'll create comprehensive API documentation for the payment API.

[Analyzes code and finds endpoints]

Found endpoints:
- POST /api/payments - Create payment
- GET /api/payments/{id} - Get payment status
- POST /api/payments/{id}/refund - Refund payment

Creating Confluence documentation with:
- Overview and authentication
- Detailed endpoint documentation
- Request/response examples in cURL, JavaScript, Python
- Error codes and handling
- Payment flow diagrams

Which Confluence space should I use?"

**You**: "Space 789012"

**Me**: [Creates comprehensive API documentation]
"Created 'Payment API Documentation' in space 789012!

Includes:
- 3 endpoint pages with full details
- Authentication guide
- Error reference
- Code examples in 3 languages

Link: https://your-domain.atlassian.net/wiki/spaces/789012/pages/..."
