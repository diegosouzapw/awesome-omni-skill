---
name: api-design
description: "API Design & Integration: best practices for RESTful and GraphQL APIs, versioning, error handling, and performance."
license: "See repository LICENSE"
---

# API Design & Integration

## Overview
Best practices for designing, implementing, and integrating RESTful and GraphQL APIs that are scalable, maintainable, and developer-friendly.

## When to Use This Skill
- Designing new API endpoints
- Integrating third-party APIs
- Implementing API versioning
- Building API documentation
- Handling API errors and edge cases
- Optimizing API performance

## RESTful API Design Principles

### 1. Resource-Based URLs
```
✅ Good - Resource-oriented
GET    /api/users              # List users
GET    /api/users/123          # Get user
POST   /api/users              # Create user
PUT    /api/users/123          # Update user
DELETE /api/users/123          # Delete user
GET    /api/users/123/orders   # Get user's orders

❌ Bad - Action-oriented
GET    /api/getAllUsers
POST   /api/createUser
POST   /api/deleteUser/123
GET    /api/getUserOrders/123
```

### 2. HTTP Methods & Status Codes

**HTTP Methods:**
- `GET` - Retrieve resource(s) (idempotent, cacheable)
- `POST` - Create new resource (non-idempotent)
- `PUT` - Replace entire resource (idempotent)
- `PATCH` - Partial update (idempotent)
- `DELETE` - Remove resource (idempotent)

**Common Status Codes:**
```
2xx Success
200 OK - Request succeeded
201 Created - Resource created
202 Accepted - Accepted for processing
204 No Content - Success but no body

3xx Redirection
301 Moved Permanently
304 Not Modified - Use cached version

4xx Client Errors
400 Bad Request - Invalid request
401 Unauthorized - Authentication required
403 Forbidden - Authenticated but not authorized
404 Not Found - Resource doesn't exist
409 Conflict - Resource conflict
422 Unprocessable Entity - Validation errors
429 Too Many Requests - Rate limited

5xx Server Errors
500 Internal Server Error
502 Bad Gateway - Upstream error
503 Service Unavailable - Temporary downtime
504 Gateway Timeout
```

### 3. Consistent Response Format
```javascript
// ✅ Good - Consistent response structure
{
  "data": {
    "id": 123,
    "name": "John Doe",
    "email": "john@example.com"
  },
  "meta": {
    "timestamp": "2026-02-12T10:00:00Z"
  }
}

// Error response
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid email format",
    "details": [
      { "field": "email", "message": "Must be a valid email" }
    ]
  },
  "meta": {
    "timestamp": "2026-02-12T10:00:00Z",
    "requestId": "abc-123"
  }
}

// ❌ Bad - Inconsistent responses
// Success returns object, error returns string
{ "id": 123, "name": "John" }
"Error: Invalid email"
```

### 4. Pagination
```javascript
// Offset-based pagination
GET /api/users?page=2&limit=20

{
  "data": [...],
  "pagination": {
    "page": 2,
    "limit": 20,
    "total": 100,
    "totalPages": 5,
    "hasNext": true,
    "hasPrevious": true
  }
}

// Cursor-based pagination (better for large datasets)
GET /api/users?cursor=eyJpZCI6MTIzfQ&limit=20

{
  "data": [...],
  "pagination": {
    "limit": 20,
    "nextCursor": "eyJpZCI6MTQzfQ",
    "hasNext": true
  }
}
```

### 5. Filtering, Sorting, and Searching
```javascript
// Filtering
GET /api/users?status=active&role=admin

// Sorting
GET /api/users?sort=createdAt:desc,name:asc

// Searching
GET /api/users?search=john&searchFields=name,email

// Combining
GET /api/users?status=active&sort=createdAt:desc&page=1&limit=20

// Implementation example
app.get('/api/users', async (req, res) => {
  const {
    status,
    role,
    sort = 'createdAt:desc',
    page = 1,
    limit = 20,
    search
  } = req.query;
  
  const query = {};
  if (status) query.status = status;
  if (role) query.role = role;
  if (search) query.$or = [
    { name: { $regex: search, $options: 'i' } },
    { email: { $regex: search, $options: 'i' } }
  ];
  
  const [field, order] = sort.split(':');
  const users = await User.find(query)
    .sort({ [field]: order === 'desc' ? -1 : 1 })
    .skip((page - 1) * limit)
    .limit(limit);
  
  const total = await User.countDocuments(query);
  
  res.json({
    data: users,
    pagination: {
      page: parseInt(page),
      limit: parseInt(limit),
      total,
      totalPages: Math.ceil(total / limit)
    }
  });
});
```

## API Versioning Strategies

### 1. URL Versioning (Most Common)
```
GET /api/v1/users
GET /api/v2/users

// Implementation
app.use('/api/v1', routesV1);
app.use('/api/v2', routesV2);
```

### 2. Header Versioning
```
GET /api/users
Accept: application/vnd.myapi.v1+json

// Implementation
app.use((req, res, next) => {
  const version = req.headers['accept']?.match(/v(\d+)/)?.[1] || '1';
  req.apiVersion = version;
  next();
});
```

### 3. Deprecation Strategy
```javascript
// Mark endpoints as deprecated
app.get('/api/v1/legacy-endpoint', (req, res) => {
  res.set('Warning', '299 - "Deprecated API. Use /api/v2/new-endpoint"');
  res.set('Sunset', 'Sat, 31 Dec 2026 23:59:59 GMT');
  // Handle request
});
```

## Error Handling

### Structured Error Responses
```javascript
class APIError extends Error {
  constructor(statusCode, code, message, details = []) {
    super(message);
    this.statusCode = statusCode;
    this.code = code;
    this.details = details;
  }
}

// Error handler middleware
app.use((err, req, res, next) => {
  const statusCode = err.statusCode || 500;
  const response = {
    error: {
      code: err.code || 'INTERNAL_ERROR',
      message: err.message || 'An unexpected error occurred',
      ...(err.details && { details: err.details })
    },
    meta: {
      timestamp: new Date().toISOString(),
      requestId: req.id
    }
  };
  
  // Don't expose internal errors in production
  if (statusCode === 500 && process.env.NODE_ENV === 'production') {
    response.error.message = 'Internal server error';
    delete response.error.details;
  }
  
  // Log error
  logger.error('API Error', {
    statusCode,
    code: err.code,
    message: err.message,
    path: req.path,
    method: req.method,
    stack: err.stack
  });
  
  res.status(statusCode).json(response);
});

// Usage
throw new APIError(400, 'INVALID_INPUT', 'Email is required', [
  { field: 'email', message: 'Email must be provided' }
]);
```

## Authentication & Authorization

### JWT Authentication
```javascript
const jwt = require('jsonwebtoken');

// Generate token
function generateToken(user) {
  return jwt.sign(
    { 
      userId: user.id,
      email: user.email,
      role: user.role
    },
    process.env.JWT_SECRET,
    { expiresIn: '1h' }
  );
}

// Auth middleware
async function authenticate(req, res, next) {
  try {
    const token = req.header('Authorization')?.replace('Bearer ', '');
    
    if (!token) {
      throw new APIError(401, 'NO_TOKEN', 'Authentication required');
    }
    
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    const user = await User.findById(decoded.userId);
    
    if (!user) {
      throw new APIError(401, 'INVALID_TOKEN', 'Invalid authentication token');
    }
    
    req.user = user;
    next();
  } catch (error) {
    if (error.name === 'TokenExpiredError') {
      throw new APIError(401, 'TOKEN_EXPIRED', 'Token has expired');
    }
    throw new APIError(401, 'INVALID_TOKEN', 'Invalid authentication token');
  }
}

// Authorization middleware
function authorize(...roles) {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      throw new APIError(403, 'FORBIDDEN', 'Insufficient permissions');
    }
    next();
  };
}

// Usage
app.get('/api/users', authenticate, authorize('admin'), async (req, res) => {
  const users = await User.find();
  res.json({ data: users });
});
```

### API Key Authentication
```javascript
async function validateAPIKey(req, res, next) {
  const apiKey = req.header('X-API-Key');
  
  if (!apiKey) {
    throw new APIError(401, 'NO_API_KEY', 'API key required');
  }
  
  const client = await APIKey.findOne({ key: hash(apiKey), isActive: true });
  
  if (!client) {
    throw new APIError(401, 'INVALID_API_KEY', 'Invalid API key');
  }
  
  // Update usage stats
  await client.incrementUsage();
  
  req.client = client;
  next();
}
```

## Rate Limiting

### Basic Rate Limiting
```javascript
const rateLimit = require('express-rate-limit');
const RedisStore = require('rate-limit-redis');

const limiter = rateLimit({
  store: new RedisStore({
    client: redisClient,
    prefix: 'rl:'
  }),
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
  message: {
    error: {
      code: 'RATE_LIMIT_EXCEEDED',
      message: 'Too many requests, please try again later'
    }
  },
  standardHeaders: true, // Return rate limit info in headers
  legacyHeaders: false
});

app.use('/api/', limiter);
```

### Per-User Rate Limiting
```javascript
const userLimiter = rateLimit({
  keyGenerator: (req) => req.user?.id || req.ip,
  windowMs: 60 * 1000, // 1 minute
  max: 60, // 60 requests per minute per user
  skipSuccessfulRequests: false
});

app.use('/api/', authenticate, userLimiter);
```

## API Documentation

### OpenAPI/Swagger
```javascript
const swaggerJsdoc = require('swagger-jsdoc');
const swaggerUi = require('swagger-ui-express');

const swaggerOptions = {
  definition: {
    openapi: '3.0.0',
    info: {
      title: 'My API',
      version: '1.0.0',
      description: 'API documentation'
    },
    servers: [
      { url: 'http://localhost:3000', description: 'Development' },
      { url: 'https://api.example.com', description: 'Production' }
    ],
    components: {
      securitySchemes: {
        bearerAuth: {
          type: 'http',
          scheme: 'bearer',
          bearerFormat: 'JWT'
        }
      }
    }
  },
  apis: ['./routes/*.js']
};

const swaggerSpec = swaggerJsdoc(swaggerOptions);
app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerSpec));

/**
 * @swagger
 * /api/users:
 *   get:
 *     summary: Get all users
 *     tags: [Users]
 *     security:
 *       - bearerAuth: []
 *     parameters:
 *       - in: query
 *         name: page
 *         schema:
 *           type: integer
 *         description: Page number
 *     responses:
 *       200:
 *         description: List of users
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 data:
 *                   type: array
 *                   items:
 *                     $ref: '#/components/schemas/User'
 */
app.get('/api/users', async (req, res) => {
  // Implementation
});
```

## Third-Party API Integration

### Retry Logic with Exponential Backoff
```javascript
async function fetchWithRetry(url, options = {}, maxRetries = 3) {
  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      const response = await fetch(url, {
        ...options,
        timeout: 10000 // 10 second timeout
      });
      
      if (!response.ok) {
        // Retry on 5xx errors or 429 (rate limit)
        if (response.status >= 500 || response.status === 429) {
          if (attempt < maxRetries) {
            const delay = Math.min(1000 * Math.pow(2, attempt), 10000);
            await new Promise(resolve => setTimeout(resolve, delay));
            continue;
          }
        }
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      
      return await response.json();
    } catch (error) {
      if (attempt === maxRetries) {
        throw new APIError(502, 'UPSTREAM_ERROR', `Failed to fetch from ${url}`, [
          { message: error.message }
        ]);
      }
    }
  }
}
```

### Circuit Breaker Pattern
```javascript
class CircuitBreaker {
  constructor(threshold = 5, timeout = 60000) {
    this.failureCount = 0;
    this.threshold = threshold;
    this.timeout = timeout;
    this.state = 'CLOSED'; // CLOSED, OPEN, HALF_OPEN
    this.nextAttempt = Date.now();
  }
  
  async execute(fn) {
    if (this.state === 'OPEN') {
      if (Date.now() < this.nextAttempt) {
        throw new Error('Circuit breaker is OPEN');
      }
      this.state = 'HALF_OPEN';
    }
    
    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }
  
  onSuccess() {
    this.failureCount = 0;
    this.state = 'CLOSED';
  }
  
  onFailure() {
    this.failureCount++;
    if (this.failureCount >= this.threshold) {
      this.state = 'OPEN';
      this.nextAttempt = Date.now() + this.timeout;
    }
  }
}

// Usage
const breaker = new CircuitBreaker(5, 60000);
const data = await breaker.execute(() => fetchExternalAPI());
```

## API Performance Optimization

### Caching
```javascript
const Redis = require('ioredis');
const redis = new Redis();

// Cache middleware
function cacheMiddleware(duration = 300) {
  return async (req, res, next) => {
    const key = `cache:${req.originalUrl}`;
    
    try {
      const cached = await redis.get(key);
      if (cached) {
        return res.json(JSON.parse(cached));
      }
    } catch (error) {
      console.error('Cache error:', error);
    }
    
    // Override res.json to cache response
    const originalJson = res.json.bind(res);
    res.json = (data) => {
      redis.setex(key, duration, JSON.stringify(data)).catch(console.error);
      return originalJson(data);
    };
    
    next();
  };
}

// Usage
app.get('/api/popular-products', cacheMiddleware(600), async (req, res) => {
  const products = await getPopularProducts();
  res.json({ data: products });
});
```

### Compression
```javascript
const compression = require('compression');

app.use(compression({
  level: 6,
  threshold: 1024, // Only compress responses > 1KB
  filter: (req, res) => {
    if (req.headers['x-no-compression']) {
      return false;
    }
    return compression.filter(req, res);
  }
}));
```

### Response Time Monitoring
```javascript
app.use((req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = Date.now() - start;
    logger.info('API Request', {
      method: req.method,
      path: req.path,
      status: res.statusCode,
      duration,
      userAgent: req.get('user-agent')
    });
    
    // Alert on slow requests
    if (duration > 1000) {
      logger.warn('Slow API Request', {
        method: req.method,
        path: req.path,
        duration
      });
    }
  });
  
  next();
});
```

## GraphQL API Design

### Schema Definition
```graphql
type User {
  id: ID!
  email: String!
  name: String!
  orders: [Order!]!
  createdAt: DateTime!
}

type Order {
  id: ID!
  userId: ID!
  user: User!
  total: Float!
  status: OrderStatus!
  items: [OrderItem!]!
}

enum OrderStatus {
  PENDING
  COMPLETED
  CANCELLED
}

type Query {
  user(id: ID!): User
  users(page: Int, limit: Int): UserConnection!
  order(id: ID!): Order
}

type Mutation {
  createUser(email: String!, name: String!): User!
  updateUser(id: ID!, name: String): User!
  deleteUser(id: ID!): Boolean!
}

type UserConnection {
  edges: [UserEdge!]!
  pageInfo: PageInfo!
}

type PageInfo {
  hasNextPage: Boolean!
  endCursor: String
}
```

### Resolver Implementation
```javascript
const resolvers = {
  Query: {
    user: async (_, { id }, context) => {
      if (!context.user) {
        throw new Error('Authentication required');
      }
      return await User.findById(id);
    },
    
    users: async (_, { page = 1, limit = 20 }, context) => {
      const users = await User.find()
        .skip((page - 1) * limit)
        .limit(limit);
      
      return {
        edges: users.map(user => ({ node: user })),
        pageInfo: {
          hasNextPage: users.length === limit,
          endCursor: users[users.length - 1]?.id
        }
      };
    }
  },
  
  Mutation: {
    createUser: async (_, { email, name }, context) => {
      if (!context.user || context.user.role !== 'admin') {
        throw new Error('Forbidden');
      }
      return await User.create({ email, name });
    }
  },
  
  User: {
    orders: async (user) => {
      return await Order.find({ userId: user.id });
    }
  }
};
```

## API Testing

### Unit Testing API Endpoints
```javascript
const request = require('supertest');
const app = require('../app');

describe('User API', () => {
  it('GET /api/users should return users list', async () => {
    const response = await request(app)
      .get('/api/users')
      .set('Authorization', `Bearer ${validToken}`)
      .expect(200);
    
    expect(response.body.data).toBeInstanceOf(Array);
    expect(response.body.pagination).toBeDefined();
  });
  
  it('POST /api/users should create user', async () => {
    const userData = {
      email: 'test@example.com',
      name: 'Test User'
    };
    
    const response = await request(app)
      .post('/api/users')
      .send(userData)
      .expect(201);
    
    expect(response.body.data.email).toBe(userData.email);
  });
});
```

## API Best Practices Checklist

- [ ] RESTful URL structure (resource-based)
- [ ] Proper HTTP methods and status codes
- [ ] Consistent response format
- [ ] Comprehensive error handling
- [ ] Authentication & authorization
- [ ] Rate limiting implemented
- [ ] API versioning strategy
- [ ] Input validation on all endpoints
- [ ] Pagination for list endpoints
- [ ] Filtering and sorting support
- [ ] API documentation (Swagger/OpenAPI)
- [ ] Caching for expensive operations
- [ ] Request/response logging
- [ ] CORS properly configured
- [ ] Compression enabled
- [ ] Security headers (Helmet.js)
- [ ] Idempotency for PUT/DELETE
- [ ] Circuit breaker for external APIs
- [ ] Monitoring and alerting
- [ ] Integration tests coverage
