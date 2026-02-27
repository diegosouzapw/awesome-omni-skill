---
name: backend-architect
description: Design RESTful APIs, microservice boundaries, and database schemas. Use when designing API endpoints, planning service architecture, creating database schemas, implementing caching strategies, or setting up authentication patterns.
---

# Backend Architect

Guidelines for scalable API design and microservices architecture.

## Core Principles

1. **Clear service boundaries** - Each service owns its data
2. **Contract-first APIs** - Design before implementation
3. **Data consistency** - Understand CAP tradeoffs
4. **Plan for scale** - Horizontal scaling from day one
5. **Keep it simple** - Avoid premature optimization

## RESTful API Design

### Resource Naming

```
# Good - nouns, plural, hierarchical
GET    /users
GET    /users/{id}
GET    /users/{id}/orders
POST   /users/{id}/orders
GET    /orders/{id}/items

# Bad - verbs, singular, flat
GET    /getUser
POST   /createOrder
GET    /user-order-list
```

### HTTP Methods

| Method | Purpose | Idempotent | Body |
|--------|---------|------------|------|
| GET | Read resource | Yes | No |
| POST | Create resource | No | Yes |
| PUT | Replace resource | Yes | Yes |
| PATCH | Partial update | No | Yes |
| DELETE | Remove resource | Yes | No |

### Response Codes

```
200 OK              - Successful GET/PUT/PATCH
201 Created         - Successful POST (include Location header)
204 No Content      - Successful DELETE
400 Bad Request     - Invalid input
401 Unauthorized    - Missing/invalid auth
403 Forbidden       - Valid auth, insufficient permissions
404 Not Found       - Resource doesn't exist
409 Conflict        - State conflict (duplicate, version mismatch)
422 Unprocessable   - Valid syntax, invalid semantics
429 Too Many Reqs   - Rate limited
500 Internal Error  - Server error (log details, return generic msg)
```

### Request/Response Format

```json
// POST /users
{
  "email": "user@example.com",
  "name": "John Doe",
  "role": "member"
}

// Response 201 Created
{
  "id": "usr_abc123",
  "email": "user@example.com",
  "name": "John Doe",
  "role": "member",
  "createdAt": "2024-01-15T10:30:00Z",
  "updatedAt": "2024-01-15T10:30:00Z"
}

// Error Response
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid request parameters",
    "details": [
      { "field": "email", "message": "Invalid email format" }
    ]
  }
}
```

### Pagination

```
GET /users?page=2&limit=20
GET /users?cursor=eyJpZCI6MTAwfQ&limit=20

Response:
{
  "data": [...],
  "pagination": {
    "total": 150,
    "page": 2,
    "limit": 20,
    "hasMore": true,
    "nextCursor": "eyJpZCI6MTIwfQ"
  }
}
```

### Filtering and Sorting

```
GET /orders?status=pending&createdAfter=2024-01-01
GET /orders?sort=-createdAt,+total
GET /users?fields=id,name,email
```

## API Versioning

```
# URL path (recommended)
GET /v1/users
GET /v2/users

# Header
GET /users
Accept: application/vnd.api+json;version=2

# Query param
GET /users?version=2
```

## Database Schema Design

### Users and Auth

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    email_verified_at TIMESTAMPTZ,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    token_hash VARCHAR(255) UNIQUE NOT NULL,
    expires_at TIMESTAMPTZ NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    INDEX idx_sessions_user (user_id),
    INDEX idx_sessions_expires (expires_at)
);
```

### Soft Deletes

```sql
ALTER TABLE users ADD COLUMN deleted_at TIMESTAMPTZ;

-- Query active users
SELECT * FROM users WHERE deleted_at IS NULL;

-- Create view for convenience
CREATE VIEW active_users AS
SELECT * FROM users WHERE deleted_at IS NULL;
```

### Optimistic Locking

```sql
ALTER TABLE orders ADD COLUMN version INTEGER DEFAULT 1;

-- Update with version check
UPDATE orders
SET status = 'shipped', version = version + 1
WHERE id = $1 AND version = $2;
-- If rows affected = 0, version conflict occurred
```

## Microservice Patterns

### Service Boundaries

```
┌─────────────────┐     ┌─────────────────┐
│  User Service   │     │  Order Service  │
│  - Registration │     │  - Cart         │
│  - Auth         │     │  - Checkout     │
│  - Profile      │     │  - History      │
│  [users DB]     │     │  [orders DB]    │
└────────┬────────┘     └────────┬────────┘
         │                       │
         └───────────┬───────────┘
                     │
              ┌──────┴──────┐
              │ API Gateway │
              │  - Routing  │
              │  - Auth     │
              │  - Rate Lim │
              └─────────────┘
```

### Inter-Service Communication

**Synchronous (HTTP/gRPC)**
- Request/response pattern
- Use for queries, real-time needs
- Include timeouts and circuit breakers

**Asynchronous (Message Queue)**
- Event-driven pattern
- Use for commands, eventual consistency
- Enables loose coupling

### Event-Driven Architecture

```typescript
// Event structure
interface DomainEvent {
  id: string;
  type: string;
  aggregateId: string;
  timestamp: string;
  data: unknown;
}

// OrderCreated event
{
  "id": "evt_123",
  "type": "order.created",
  "aggregateId": "ord_456",
  "timestamp": "2024-01-15T10:30:00Z",
  "data": {
    "userId": "usr_789",
    "total": 99.99,
    "items": [...]
  }
}
```

## Caching Strategies

### Cache-Aside Pattern

```typescript
async function getUser(id: string): Promise<User> {
  // Check cache first
  const cached = await cache.get(`user:${id}`);
  if (cached) return JSON.parse(cached);

  // Cache miss - fetch from DB
  const user = await db.users.findById(id);

  // Populate cache
  await cache.set(`user:${id}`, JSON.stringify(user), 'EX', 3600);

  return user;
}
```

### Cache Invalidation

```typescript
// Invalidate on write
async function updateUser(id: string, data: Partial<User>) {
  await db.users.update(id, data);
  await cache.del(`user:${id}`);
}

// Use cache tags for related data
await cache.set(`user:${id}`, data, { tags: [`user:${id}`, 'users'] });
await cache.invalidateTag('users'); // Clear all user caches
```

## Authentication Patterns

### JWT Structure

```typescript
// Access Token (short-lived, 15min)
{
  "sub": "usr_123",
  "email": "user@example.com",
  "roles": ["user"],
  "exp": 1705312200,
  "iat": 1705311300
}

// Refresh Token (long-lived, stored in DB)
{
  "sub": "usr_123",
  "jti": "tok_abc", // Unique ID for revocation
  "exp": 1707903300
}
```

### Auth Flow

```
1. POST /auth/login → { accessToken, refreshToken }
2. GET /api/users (Authorization: Bearer <accessToken>)
3. POST /auth/refresh → { accessToken } (when expired)
4. POST /auth/logout → Revoke refresh token
```

## Rate Limiting

```typescript
// Token bucket algorithm
interface RateLimiter {
  identifier: string;  // IP or user ID
  maxRequests: number; // e.g., 100
  windowMs: number;    // e.g., 60000 (1 minute)
}

// Response headers
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 42
X-RateLimit-Reset: 1705312200
```

## Health Checks

```typescript
// GET /health
{
  "status": "healthy",
  "version": "1.2.3",
  "timestamp": "2024-01-15T10:30:00Z",
  "checks": {
    "database": { "status": "up", "latency": 5 },
    "cache": { "status": "up", "latency": 1 },
    "queue": { "status": "up" }
  }
}
```
