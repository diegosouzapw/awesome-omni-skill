---
name: backend-skills
description: Master Node.js, Express, PHP, Laravel, Java, Spring Boot, API design, and database integration. Build scalable APIs and server applications.
sasmp_version: "1.3.0"
skill_type: atomic
version: "2.0.0"

parameters:
  framework:
    type: string
    enum: [express, fastify, nestjs, laravel, spring-boot, django, gin]
    default: express
  database:
    type: string
    enum: [postgresql, mysql, mongodb, sqlite]
    default: postgresql

validation_rules:
  - pattern: "^(GET|POST|PUT|PATCH|DELETE)$"
    target: http_methods
    message: Use standard HTTP methods
  - pattern: "^/api/v[0-9]+/"
    target: api_routes
    message: API routes should be versioned

retry_config:
  max_attempts: 3
  backoff: exponential
  initial_delay_ms: 1000
  max_delay_ms: 30000

logging:
  on_entry: "[API] {method} {path} - Request started"
  on_success: "[API] {method} {path} - {status} ({duration}ms)"
  on_error: "[API] {method} {path} - {status} {error}"

dependencies:
  agents:
    - backend-server-developer
---

# Backend Development Skills

## Express.js Fundamentals

```javascript
const express = require('express');
const app = express();

app.use(express.json());

// Routes with error handling
app.get('/api/users/:id', async (req, res, next) => {
  try {
    const user = await User.findById(req.params.id);
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    res.json(user);
  } catch (error) {
    next(error);
  }
});

// Global error handler
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({
    error: 'Internal Server Error',
    requestId: req.id
  });
});

app.listen(3000);
```

## RESTful API Design

```
HTTP Methods:
GET    /api/v1/users       - List all
GET    /api/v1/users/:id   - Get one
POST   /api/v1/users       - Create
PUT    /api/v1/users/:id   - Full update
PATCH  /api/v1/users/:id   - Partial update
DELETE /api/v1/users/:id   - Delete

Status Codes:
200 OK          - Success
201 Created     - Resource created
204 No Content  - Success, no body
400 Bad Request - Validation error
401 Unauthorized- Auth required
403 Forbidden   - No permission
404 Not Found   - Resource missing
409 Conflict    - State conflict
429 Too Many    - Rate limited
500 Server Error- Unexpected error
```

## Database Operations with Retry

```javascript
// Connection pool with retry logic
const { Pool } = require('pg');

const pool = new Pool({
  max: 20,
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

async function queryWithRetry(sql, params, maxRetries = 3) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const result = await pool.query(sql, params);
      return result;
    } catch (error) {
      if (attempt === maxRetries) throw error;
      const delay = Math.pow(2, attempt) * 1000;
      await new Promise(r => setTimeout(r, delay));
    }
  }
}
```

## Authentication & Security

```javascript
// JWT Authentication with validation
const jwt = require('jsonwebtoken');

function generateToken(payload) {
  return jwt.sign(payload, process.env.JWT_SECRET, {
    expiresIn: '24h',
    issuer: 'api.example.com',
    audience: 'example.com'
  });
}

// Middleware with proper error handling
const authenticateToken = async (req, res, next) => {
  try {
    const authHeader = req.headers.authorization;
    if (!authHeader?.startsWith('Bearer ')) {
      return res.status(401).json({ error: 'Missing token' });
    }

    const token = authHeader.split(' ')[1];
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (error) {
    if (error.name === 'TokenExpiredError') {
      return res.status(401).json({ error: 'Token expired' });
    }
    return res.status(403).json({ error: 'Invalid token' });
  }
};
```

## Input Validation Pattern

```javascript
const Joi = require('joi');

const userSchema = Joi.object({
  email: Joi.string().email().required(),
  password: Joi.string().min(8).required(),
  name: Joi.string().max(100).required()
});

const validate = (schema) => (req, res, next) => {
  const { error, value } = schema.validate(req.body);
  if (error) {
    return res.status(400).json({
      error: 'Validation failed',
      details: error.details.map(d => ({
        field: d.path.join('.'),
        message: d.message
      }))
    });
  }
  req.validated = value;
  next();
};

app.post('/api/users', validate(userSchema), createUser);
```

## Database Query Optimization

| Technique | Impact | When to Use |
|-----------|--------|-------------|
| Indexing | High | Frequently queried columns |
| Connection Pooling | High | Always in production |
| Query Optimization | Medium | SELECT only needed columns |
| N+1 Prevention | High | Use JOINs/eager loading |
| Caching | High | Frequently accessed data |
| Pagination | Medium | Large result sets |

## Error Handling Best Practices

```javascript
// Structured error response
class ApiError extends Error {
  constructor(statusCode, code, message, details = null) {
    super(message);
    this.statusCode = statusCode;
    this.code = code;
    this.details = details;
  }

  toJSON() {
    return {
      error: {
        code: this.code,
        message: this.message,
        details: this.details
      }
    };
  }
}

// Usage
throw new ApiError(400, 'VALIDATION_ERROR', 'Email is required', [
  { field: 'email', message: 'Must be valid email' }
]);
```

## Unit Test Template

```javascript
const request = require('supertest');
const app = require('../app');

describe('Users API', () => {
  beforeEach(async () => {
    await db.truncate('users');
  });

  describe('GET /api/users/:id', () => {
    test('returns user when found', async () => {
      const user = await createUser({ email: 'test@example.com' });

      const res = await request(app)
        .get(`/api/users/${user.id}`)
        .set('Authorization', `Bearer ${token}`)
        .expect(200);

      expect(res.body.email).toBe('test@example.com');
    });

    test('returns 404 when not found', async () => {
      await request(app)
        .get('/api/users/999')
        .set('Authorization', `Bearer ${token}`)
        .expect(404);
    });
  });
});
```

## Troubleshooting Guide

| Symptom | Cause | Solution |
|---------|-------|----------|
| ECONNREFUSED | Service not running | Check service status |
| ER_DUP_ENTRY | Duplicate key | Handle upsert properly |
| ETIMEDOUT | Slow query | Add timeout, optimize |
| ECONNRESET | Connection dropped | Implement retry logic |

## Key Concepts Checklist

- [ ] Express routing and middleware
- [ ] Request/response handling
- [ ] JSON parsing and validation
- [ ] Database CRUD operations
- [ ] SQL query optimization
- [ ] Authentication (JWT, OAuth)
- [ ] Error handling
- [ ] API versioning
- [ ] Rate limiting
- [ ] CORS configuration
- [ ] Environment variables
- [ ] Testing with Jest/Supertest
- [ ] Logging and monitoring
- [ ] Security best practices

---

**Source**: https://roadmap.sh
**Version**: 2.0.0
**Last Updated**: 2025-01-01
