---
name: express-microservices-architecture
description: Complete guide for building scalable microservices with Express.js including middleware patterns, routing strategies, error handling, production architecture, and deployment best practices
tags: [express, microservices, nodejs, middleware, routing, scalability, architecture, production]
tier: tier-1
---

# Express.js Microservices Architecture

A comprehensive skill for building production-ready microservices with Express.js. Master middleware patterns, routing strategies, error handling, scalability techniques, and deployment architectures for Node.js microservices at scale.

## When to Use This Skill

Use this skill when:

- Building RESTful APIs and microservices with Node.js
- Designing scalable distributed systems with Express.js
- Implementing middleware-based architecture patterns
- Creating API gateways and service mesh architectures
- Developing production-ready Node.js applications
- Migrating monoliths to microservices architecture
- Building event-driven microservices
- Implementing authentication, authorization, and security layers
- Optimizing Express.js applications for high performance
- Setting up monitoring, logging, and observability
- Deploying Express.js apps with Docker and Kubernetes
- Implementing circuit breakers and resilience patterns

## Core Concepts

### Express.js Fundamentals

Express.js is a minimal and flexible Node.js web application framework that provides robust features for web and mobile applications. It's the de facto standard for building Node.js APIs and microservices.

**Key Characteristics:**
- **Minimal**: Unopinionated framework with essential web app features
- **Middleware-based**: Request/response pipeline architecture
- **Routing**: Powerful routing mechanism with parameter support
- **Template Engines**: Support for various view engines
- **Performance**: Built on top of Node.js for high performance
- **Extensible**: Rich ecosystem of middleware and plugins

### Middleware Architecture

Middleware functions are the backbone of Express.js applications. They have access to the request object (`req`), response object (`res`), and the next middleware function (`next`).

**Middleware Flow:**
```
Request → Middleware 1 → Middleware 2 → ... → Route Handler → Response
                ↓              ↓                      ↓
           Error Handler  Error Handler         Error Handler
```

**Middleware Types:**
1. **Application-level middleware**: Bound to `app` instance
2. **Router-level middleware**: Bound to `express.Router()` instance
3. **Error-handling middleware**: Has 4 parameters (err, req, res, next)
4. **Built-in middleware**: Express built-in functions (static, json, urlencoded)
5. **Third-party middleware**: External packages (cors, helmet, morgan)

### Routing Strategies

Express routing enables you to map HTTP methods and URLs to handler functions.

**Routing Components:**
- **Route paths**: String patterns, regex, or path parameters
- **Route parameters**: Named URL segments (:userId)
- **Route handlers**: Single or multiple callback functions
- **Response methods**: res.send(), res.json(), res.status(), etc.
- **Router instances**: Modular, mountable route handlers

### Error Handling

Error handling in Express requires special middleware with 4 parameters: `(err, req, res, next)`.

**Error Handling Flow:**
1. Synchronous errors are caught automatically
2. Asynchronous errors must be passed to `next(err)`
3. Error middleware processes errors centrally
4. Proper status codes and error formats returned

### Microservices Principles

**Characteristics of Microservices:**
- **Single Responsibility**: Each service does one thing well
- **Independence**: Services can be deployed independently
- **Decentralized**: Each service owns its data
- **Resilience**: Failure in one service doesn't crash entire system
- **Scalability**: Scale services independently based on demand
- **Technology Diversity**: Different services can use different tech stacks

## Microservices Patterns

### Pattern 1: API Gateway Pattern

The API Gateway acts as a single entry point for all client requests, routing them to appropriate microservices.

**Benefits:**
- Single entry point for clients
- Request routing and composition
- Authentication and authorization
- Rate limiting and throttling
- Request/response transformation
- Protocol translation

**Implementation Structure:**
```
Client → API Gateway → Microservice 1 (Users)
                    → Microservice 2 (Orders)
                    → Microservice 3 (Products)
                    → Microservice 4 (Notifications)
```

### Pattern 2: Service Discovery

Services register themselves and discover other services dynamically.

**Approaches:**
- **Client-side discovery**: Client queries service registry
- **Server-side discovery**: Load balancer queries registry
- **DNS-based discovery**: Using DNS for service location

**Popular Tools:**
- Consul
- Eureka
- etcd
- Kubernetes built-in discovery

### Pattern 3: Circuit Breaker

Prevents cascading failures by stopping requests to failing services.

**States:**
- **Closed**: Normal operation, requests pass through
- **Open**: Service failing, requests fail immediately
- **Half-Open**: Testing if service recovered

### Pattern 4: Event-Driven Architecture

Services communicate through events instead of direct calls.

**Components:**
- **Event producers**: Services that emit events
- **Event consumers**: Services that listen to events
- **Message broker**: RabbitMQ, Kafka, Redis
- **Event store**: Persist events for replay

### Pattern 5: Database per Service

Each microservice owns its database, ensuring loose coupling.

**Benefits:**
- Service independence
- Technology diversity
- Easier scaling
- Clear boundaries

**Challenges:**
- Distributed transactions
- Data consistency
- Joins across services

### Pattern 6: Saga Pattern

Manages distributed transactions across multiple services.

**Types:**
- **Choreography**: Services coordinate through events
- **Orchestration**: Central coordinator manages transaction

### Pattern 7: CQRS (Command Query Responsibility Segregation)

Separate read and write operations into different models.

**Benefits:**
- Optimized read/write models
- Scalability
- Performance
- Flexibility

## Middleware Architecture Patterns

### Custom Middleware Development

Middleware functions execute in the order they're defined.

**Basic Middleware Structure:**
```javascript
const express = require('express');
const app = express();

// Basic middleware
const requestLogger = (req, res, next) => {
  console.log(`[${new Date().toISOString()}] ${req.method} ${req.url}`);
  next(); // Pass control to next middleware
};

app.use(requestLogger);
```

**From Context7 - Saving Data in Request Object:**
```javascript
const express = require('express');
const app = express();
const port = 3000;

// Middleware to add user data to the request object
const addUserInfo = (req, res, next) => {
  req.user = {
    id: 123,
    username: 'testuser'
  };
  next();
};

// Middleware to add request timestamp
const addTimestamp = (req, res, next) => {
  req.requestTime = Date.now();
  next();
};

// Apply middleware globally
app.use(addUserInfo);
app.use(addTimestamp);

app.get('/', (req, res) => {
  const userId = req.user.id;
  const username = req.user.username;
  const timestamp = req.requestTime;

  res.send(`User ID: ${userId}, Username: ${username}, Request Time: ${new Date(timestamp).toISOString()}`);
});

app.listen(port, () => {
  console.log(`Request data sharing example listening at http://localhost:${port}`);
});
```

### Error-Handling Middleware

Error middleware has 4 parameters and should be defined after all other middleware.

**From Context7 - Error Handling Middleware:**
```javascript
const express = require('express');
const app = express();
const port = 3000;

// A regular middleware
app.use((req, res, next) => {
  console.log('Request received');
  next(); // Pass control to the next middleware
});

// A route that might throw an error
app.get('/throw-error', (req, res, next) => {
  // Simulate an error
  const error = new Error('This is a simulated error');
  error.status = 400;
  next(error);
});

// Error-handling middleware (must have 4 arguments)
app.use((err, req, res, next) => {
  console.error('Error caught:', err.message);
  res.status(err.status || 500).send(`An error occurred: ${err.message}`);
});

app.listen(port, () => {
  console.log(`Error middleware example listening at http://localhost:${port}`);
});
```

**From Context7 - Global Error Handler:**
```javascript
app.use(express.bodyParser())
app.use(express.cookieParser())
app.use(express.session())
app.use(app.router) // the router itself (app.get(), app.put() etc)
app.use(function(err, req, res, next){
  // if an error occurs Connect will pass it down
  // through these "error-handling" middleware
  // allowing you to respond however you like
  res.send(500, { error: 'Sorry something bad happened!' });
})
```

### Route-Specific Middleware

Apply middleware to specific routes for targeted functionality.

**From Context7 - Route Middleware:**
```javascript
const express = require('express');
const app = express();
const port = 3000;

// Middleware function
const requestLogger = (req, res, next) => {
  console.log(`[${new Date().toISOString()}] ${req.method} ${req.url}`);
  next();
};

// Apply middleware to a specific route
app.get('/protected', requestLogger, (req, res) => {
  res.send('This route is protected by middleware!');
});

// Apply middleware to multiple routes
const adminMiddleware = (req, res, next) => {
  console.log('Admin access check...');
  // In a real app, you'd check user roles here
  next();
};

app.get('/admin/dashboard', adminMiddleware, (req, res) => {
  res.send('Welcome to the admin dashboard!');
});

// Middleware applied globally
app.use(requestLogger);

app.get('/', (req, res) => {
  res.send('Hello, world!');
});

app.listen(port, () => {
  console.log(`Route middleware example listening at http://localhost:${port}`);
});
```

### Authentication Middleware

```javascript
const jwt = require('jsonwebtoken');

// JWT Authentication Middleware
const authenticateToken = (req, res, next) => {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1];

  if (!token) {
    return res.status(401).json({ error: 'Access token required' });
  }

  jwt.verify(token, process.env.JWT_SECRET, (err, user) => {
    if (err) {
      return res.status(403).json({ error: 'Invalid or expired token' });
    }
    req.user = user;
    next();
  });
};

// Role-based Authorization Middleware
const authorize = (...roles) => {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Unauthorized' });
    }

    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Insufficient permissions' });
    }

    next();
  };
};

// Usage
app.get('/admin/users', authenticateToken, authorize('admin'), (req, res) => {
  res.json({ users: [] });
});
```

### Request Validation Middleware

```javascript
const { body, param, query, validationResult } = require('express-validator');

// Validation middleware factory
const validate = (validations) => {
  return async (req, res, next) => {
    await Promise.all(validations.map(validation => validation.run(req)));

    const errors = validationResult(req);
    if (errors.isEmpty()) {
      return next();
    }

    res.status(400).json({
      error: 'Validation failed',
      details: errors.array()
    });
  };
};

// Usage
app.post('/users', validate([
  body('email').isEmail().normalizeEmail(),
  body('password').isLength({ min: 8 }),
  body('name').trim().notEmpty()
]), (req, res) => {
  // Request is validated
  res.json({ success: true });
});
```

### Rate Limiting Middleware

```javascript
const rateLimit = require('express-rate-limit');
const RedisStore = require('rate-limit-redis');
const Redis = require('ioredis');

// In-memory rate limiter
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Limit each IP to 100 requests per windowMs
  message: 'Too many requests, please try again later',
  standardHeaders: true,
  legacyHeaders: false,
});

// Redis-based rate limiter for distributed systems
const redisClient = new Redis({
  host: process.env.REDIS_HOST,
  port: process.env.REDIS_PORT,
});

const distributedLimiter = rateLimit({
  store: new RedisStore({
    client: redisClient,
    prefix: 'rl:',
  }),
  windowMs: 15 * 60 * 1000,
  max: 100,
});

// Apply to all routes
app.use('/api/', distributedLimiter);

// Apply to specific routes
app.post('/api/login', rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5, // Only 5 login attempts per 15 minutes
}), loginHandler);
```

### Logging Middleware

```javascript
const morgan = require('morgan');
const winston = require('winston');
const { format } = winston;

// Create Winston logger
const logger = winston.createLogger({
  level: 'info',
  format: format.combine(
    format.timestamp(),
    format.errors({ stack: true }),
    format.json()
  ),
  defaultMeta: { service: 'user-service' },
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' }),
  ],
});

// Console logging in development
if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: format.combine(
      format.colorize(),
      format.simple()
    )
  }));
}

// HTTP request logging with Morgan
app.use(morgan('combined', {
  stream: {
    write: (message) => logger.info(message.trim())
  }
}));

// Custom logging middleware
const requestLogger = (req, res, next) => {
  const start = Date.now();

  res.on('finish', () => {
    const duration = Date.now() - start;
    logger.info({
      method: req.method,
      path: req.path,
      status: res.statusCode,
      duration: `${duration}ms`,
      ip: req.ip,
      userAgent: req.get('user-agent')
    });
  });

  next();
};

app.use(requestLogger);
```

### CORS Middleware

```javascript
const cors = require('cors');

// Basic CORS
app.use(cors());

// Configured CORS
const corsOptions = {
  origin: function (origin, callback) {
    const allowedOrigins = [
      'https://example.com',
      'https://app.example.com',
      process.env.FRONTEND_URL
    ];

    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  credentials: true,
  optionsSuccessStatus: 200,
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  exposedHeaders: ['X-Total-Count', 'X-Page-Number'],
  maxAge: 86400, // 24 hours
};

app.use(cors(corsOptions));

// CORS for specific routes
app.options('/api/admin/*', cors(adminCorsOptions));
app.use('/api/admin/', cors(adminCorsOptions));
```

### Security Middleware

```javascript
const helmet = require('helmet');
const mongoSanitize = require('express-mongo-sanitize');
const xss = require('xss-clean');
const hpp = require('hpp');

// Helmet - Set security headers
app.use(helmet());

// Custom security headers
app.use((req, res, next) => {
  res.setHeader('X-Content-Type-Options', 'nosniff');
  res.setHeader('X-Frame-Options', 'DENY');
  res.setHeader('X-XSS-Protection', '1; mode=block');
  next();
});

// Sanitize data against NoSQL injection
app.use(mongoSanitize());

// Prevent XSS attacks
app.use(xss());

// Prevent HTTP Parameter Pollution
app.use(hpp({
  whitelist: ['sort', 'fields', 'page', 'limit']
}));

// Content Security Policy
app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    styleSrc: ["'self'", "'unsafe-inline'"],
    scriptSrc: ["'self'"],
    imgSrc: ["'self'", 'data:', 'https:'],
  },
}));
```

## Routing Strategies

### Basic Routing

**From Context7 - Express Routing:**
```javascript
app.get('/', home);
app.use('/public', require('st')(process.cwd()));
app.get('/users', users.list);
app.post('/users', users.create);
```

### Route Method Chaining

**From Context7 - Route Chaining:**
```javascript
app.route('/users')
.get(function(req, res, next) {
  // Get all users
  res.json({ users: [] });
})
.post(function(req, res, next) {
  // Create new user
  res.status(201).json({ user: {} });
});
```

### Router Modules

```javascript
// routes/users.js
const express = require('express');
const router = express.Router();

// Middleware specific to this router
router.use((req, res, next) => {
  console.log('Time: ', Date.now());
  next();
});

// Define routes
router.get('/', (req, res) => {
  res.json({ users: [] });
});

router.get('/:id', (req, res) => {
  res.json({ user: { id: req.params.id } });
});

router.post('/', (req, res) => {
  res.status(201).json({ user: req.body });
});

router.put('/:id', (req, res) => {
  res.json({ user: { id: req.params.id, ...req.body } });
});

router.delete('/:id', (req, res) => {
  res.status(204).send();
});

module.exports = router;

// app.js
const usersRouter = require('./routes/users');
app.use('/api/users', usersRouter);
```

### Route Parameters

```javascript
// Named parameters
app.get('/users/:userId/posts/:postId', (req, res) => {
  const { userId, postId } = req.params;
  res.json({ userId, postId });
});

// Parameter middleware
app.param('userId', (req, res, next, userId) => {
  // Fetch user from database
  User.findById(userId)
    .then(user => {
      if (!user) {
        return res.status(404).json({ error: 'User not found' });
      }
      req.user = user;
      next();
    })
    .catch(next);
});

// Multiple callbacks
app.param('postId', [
  validatePostId,
  fetchPost,
  checkPermissions
]);
```

### Query Parameters

```javascript
// GET /api/users?role=admin&active=true&page=2&limit=10
app.get('/api/users', (req, res) => {
  const {
    role,
    active,
    page = 1,
    limit = 10,
    sort = '-createdAt'
  } = req.query;

  const query = {};
  if (role) query.role = role;
  if (active !== undefined) query.active = active === 'true';

  const skip = (page - 1) * limit;

  User.find(query)
    .sort(sort)
    .limit(parseInt(limit))
    .skip(skip)
    .then(users => res.json({ users, page, limit }))
    .catch(next);
});
```

### API Versioning

```javascript
// Version 1 routes
const v1Router = express.Router();
v1Router.get('/users', (req, res) => {
  res.json({ version: 'v1', users: [] });
});

// Version 2 routes
const v2Router = express.Router();
v2Router.get('/users', (req, res) => {
  res.json({ version: 'v2', users: [], meta: {} });
});

// Mount versioned routes
app.use('/api/v1', v1Router);
app.use('/api/v2', v2Router);

// Header-based versioning
app.use('/api/users', (req, res, next) => {
  const version = req.headers['api-version'] || 'v1';

  if (version === 'v2') {
    return v2UsersHandler(req, res, next);
  }
  return v1UsersHandler(req, res, next);
});
```

### RESTful Route Organization

```javascript
// controllers/users.controller.js
class UsersController {
  async list(req, res, next) {
    try {
      const users = await User.find();
      res.json({ users });
    } catch (error) {
      next(error);
    }
  }

  async get(req, res, next) {
    try {
      res.json({ user: req.user });
    } catch (error) {
      next(error);
    }
  }

  async create(req, res, next) {
    try {
      const user = await User.create(req.body);
      res.status(201).json({ user });
    } catch (error) {
      next(error);
    }
  }

  async update(req, res, next) {
    try {
      const user = await User.findByIdAndUpdate(
        req.params.id,
        req.body,
        { new: true, runValidators: true }
      );
      res.json({ user });
    } catch (error) {
      next(error);
    }
  }

  async delete(req, res, next) {
    try {
      await User.findByIdAndDelete(req.params.id);
      res.status(204).send();
    } catch (error) {
      next(error);
    }
  }
}

// routes/users.routes.js
const router = express.Router();
const controller = new UsersController();

router.get('/', controller.list);
router.get('/:id', controller.get);
router.post('/', controller.create);
router.put('/:id', controller.update);
router.delete('/:id', controller.delete);

module.exports = router;
```

## Scalability Patterns

### Horizontal Scaling

Deploy multiple instances of your service behind a load balancer.

```javascript
// Enable cluster mode
const cluster = require('cluster');
const os = require('os');

if (cluster.isMaster) {
  const numCPUs = os.cpus().length;

  console.log(`Master process ${process.pid} is running`);
  console.log(`Forking ${numCPUs} workers...`);

  // Fork workers
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died. Restarting...`);
    cluster.fork();
  });
} else {
  // Workers share the TCP connection
  const app = require('./app');
  const port = process.env.PORT || 3000;

  app.listen(port, () => {
    console.log(`Worker ${process.pid} started on port ${port}`);
  });
}
```

### Load Balancing

```nginx
# nginx.conf
upstream backend {
    least_conn;
    server localhost:3001;
    server localhost:3002;
    server localhost:3003;
    server localhost:3004;
}

server {
    listen 80;

    location / {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

### Caching Strategies

```javascript
const Redis = require('ioredis');
const redis = new Redis({
  host: process.env.REDIS_HOST,
  port: process.env.REDIS_PORT,
});

// Cache middleware
const cacheMiddleware = (duration = 300) => {
  return async (req, res, next) => {
    if (req.method !== 'GET') {
      return next();
    }

    const key = `cache:${req.originalUrl}`;

    try {
      const cached = await redis.get(key);

      if (cached) {
        return res.json(JSON.parse(cached));
      }

      // Store original res.json
      const originalJson = res.json.bind(res);

      // Override res.json
      res.json = (body) => {
        redis.setex(key, duration, JSON.stringify(body));
        return originalJson(body);
      };

      next();
    } catch (error) {
      console.error('Cache error:', error);
      next();
    }
  };
};

// Usage
app.get('/api/products', cacheMiddleware(600), async (req, res) => {
  const products = await Product.find();
  res.json({ products });
});

// Cache invalidation
const invalidateCache = async (pattern) => {
  const keys = await redis.keys(pattern);
  if (keys.length > 0) {
    await redis.del(...keys);
  }
};

// Invalidate on updates
app.post('/api/products', async (req, res) => {
  const product = await Product.create(req.body);
  await invalidateCache('cache:/api/products*');
  res.status(201).json({ product });
});
```

### Database Connection Pooling

```javascript
const mongoose = require('mongoose');

// MongoDB connection with pooling
mongoose.connect(process.env.MONGODB_URI, {
  useNewUrlParser: true,
  useUnifiedTopology: true,
  poolSize: 10, // Maintain up to 10 socket connections
  socketTimeoutMS: 45000,
  family: 4,
});

// PostgreSQL with connection pooling
const { Pool } = require('pg');
const pool = new Pool({
  host: process.env.DB_HOST,
  port: process.env.DB_PORT,
  database: process.env.DB_NAME,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  max: 20, // Maximum number of clients
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

// Query helper
const query = async (text, params) => {
  const start = Date.now();
  const res = await pool.query(text, params);
  const duration = Date.now() - start;
  console.log('Executed query', { text, duration, rows: res.rowCount });
  return res;
};

module.exports = { query, pool };
```

### Response Compression

```javascript
const compression = require('compression');

// Basic compression
app.use(compression());

// Custom compression settings
app.use(compression({
  filter: (req, res) => {
    if (req.headers['x-no-compression']) {
      return false;
    }
    return compression.filter(req, res);
  },
  level: 6, // Compression level (0-9)
  threshold: 1024, // Minimum size to compress (bytes)
}));
```

### Request Throttling

```javascript
const { Throttle } = require('stream-throttle');

// Throttle large responses
app.get('/api/large-dataset', (req, res) => {
  const dataStream = getLargeDataStream();

  // Throttle to 1MB/s
  const throttle = new Throttle({ rate: 1024 * 1024 });

  res.setHeader('Content-Type', 'application/json');
  dataStream.pipe(throttle).pipe(res);
});
```

## Production Architecture

### Docker Containerization

```dockerfile
# Dockerfile
FROM node:18-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy source code
COPY . .

# Production image
FROM node:18-alpine

WORKDIR /app

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Copy from builder
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --chown=nodejs:nodejs . .

# Switch to non-root user
USER nodejs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s \
  CMD node healthcheck.js

# Start application
CMD ["node", "server.js"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  api:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - REDIS_HOST=redis
      - MONGODB_URI=mongodb://mongo:27017/app
    depends_on:
      - redis
      - mongo
    restart: unless-stopped
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '0.5'
          memory: 512M

  redis:
    image: redis:7-alpine
    volumes:
      - redis-data:/data
    restart: unless-stopped

  mongo:
    image: mongo:6
    volumes:
      - mongo-data:/data/db
    restart: unless-stopped

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - api
    restart: unless-stopped

volumes:
  redis-data:
  mongo-data:
```

### Process Management with PM2

```javascript
// ecosystem.config.js
module.exports = {
  apps: [{
    name: 'api',
    script: './server.js',
    instances: 'max',
    exec_mode: 'cluster',
    autorestart: true,
    watch: false,
    max_memory_restart: '1G',
    env: {
      NODE_ENV: 'development',
      PORT: 3000
    },
    env_production: {
      NODE_ENV: 'production',
      PORT: 3000
    },
    error_file: './logs/err.log',
    out_file: './logs/out.log',
    log_file: './logs/combined.log',
    time: true,
    merge_logs: true,
  }]
};
```

### Health Checks

```javascript
// healthcheck.js
const http = require('http');

const options = {
  host: 'localhost',
  port: process.env.PORT || 3000,
  path: '/health',
  timeout: 2000
};

const healthCheck = http.request(options, (res) => {
  console.log(`HEALTHCHECK STATUS: ${res.statusCode}`);
  if (res.statusCode === 200) {
    process.exit(0);
  } else {
    process.exit(1);
  }
});

healthCheck.on('error', (err) => {
  console.error('ERROR:', err);
  process.exit(1);
});

healthCheck.end();

// Health endpoint
app.get('/health', async (req, res) => {
  const health = {
    uptime: process.uptime(),
    message: 'OK',
    timestamp: Date.now()
  };

  try {
    // Check database connection
    await mongoose.connection.db.admin().ping();
    health.database = 'connected';

    // Check Redis connection
    await redis.ping();
    health.cache = 'connected';

    res.status(200).json(health);
  } catch (error) {
    health.message = error.message;
    res.status(503).json(health);
  }
});

// Readiness check
app.get('/ready', (req, res) => {
  res.status(200).json({ ready: true });
});

// Liveness check
app.get('/live', (req, res) => {
  res.status(200).json({ alive: true });
});
```

### Graceful Shutdown

```javascript
// server.js
const gracefulShutdown = () => {
  console.log('Received shutdown signal, closing server gracefully...');

  server.close(async () => {
    console.log('HTTP server closed');

    try {
      // Close database connections
      await mongoose.connection.close();
      console.log('MongoDB connection closed');

      // Close Redis connection
      await redis.quit();
      console.log('Redis connection closed');

      // Close other resources
      // ...

      console.log('Graceful shutdown completed');
      process.exit(0);
    } catch (err) {
      console.error('Error during shutdown:', err);
      process.exit(1);
    }
  });

  // Force shutdown after 10 seconds
  setTimeout(() => {
    console.error('Forcing shutdown after timeout');
    process.exit(1);
  }, 10000);
};

process.on('SIGTERM', gracefulShutdown);
process.on('SIGINT', gracefulShutdown);
```

### Monitoring and Observability

```javascript
const promClient = require('prom-client');

// Create a Registry
const register = new promClient.Registry();

// Add default metrics
promClient.collectDefaultMetrics({ register });

// Custom metrics
const httpRequestDuration = new promClient.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.1, 0.5, 1, 2, 5]
});

const httpRequestTotal = new promClient.Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status_code']
});

register.registerMetric(httpRequestDuration);
register.registerMetric(httpRequestTotal);

// Metrics middleware
app.use((req, res, next) => {
  const start = Date.now();

  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    const route = req.route ? req.route.path : req.path;

    httpRequestDuration.labels(req.method, route, res.statusCode).observe(duration);
    httpRequestTotal.labels(req.method, route, res.statusCode).inc();
  });

  next();
});

// Metrics endpoint
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType);
  res.end(await register.metrics());
});
```

### Distributed Tracing

```javascript
const { trace, context } = require('@opentelemetry/api');
const { NodeTracerProvider } = require('@opentelemetry/sdk-trace-node');
const { registerInstrumentations } = require('@opentelemetry/instrumentation');
const { HttpInstrumentation } = require('@opentelemetry/instrumentation-http');
const { ExpressInstrumentation } = require('@opentelemetry/instrumentation-express');

// Create tracer provider
const provider = new NodeTracerProvider();
provider.register();

// Register instrumentations
registerInstrumentations({
  instrumentations: [
    new HttpInstrumentation(),
    new ExpressInstrumentation(),
  ],
});

const tracer = trace.getTracer('user-service');

// Custom tracing middleware
const tracingMiddleware = (req, res, next) => {
  const span = tracer.startSpan(`HTTP ${req.method} ${req.path}`);

  span.setAttributes({
    'http.method': req.method,
    'http.url': req.url,
    'http.user_agent': req.get('user-agent'),
  });

  res.on('finish', () => {
    span.setAttributes({
      'http.status_code': res.statusCode,
    });
    span.end();
  });

  next();
};

app.use(tracingMiddleware);
```

## Best Practices

### Project Structure

```
express-microservice/
├── src/
│   ├── config/
│   │   ├── database.js
│   │   ├── redis.js
│   │   └── logger.js
│   ├── controllers/
│   │   ├── users.controller.js
│   │   └── auth.controller.js
│   ├── middleware/
│   │   ├── auth.js
│   │   ├── validation.js
│   │   ├── errorHandler.js
│   │   └── requestLogger.js
│   ├── models/
│   │   └── user.model.js
│   ├── routes/
│   │   ├── index.js
│   │   ├── users.routes.js
│   │   └── auth.routes.js
│   ├── services/
│   │   ├── users.service.js
│   │   ├── auth.service.js
│   │   └── email.service.js
│   ├── utils/
│   │   ├── apiError.js
│   │   ├── catchAsync.js
│   │   └── validators.js
│   ├── app.js
│   └── server.js
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
├── .env
├── .env.example
├── .dockerignore
├── Dockerfile
├── docker-compose.yml
├── ecosystem.config.js
├── package.json
└── README.md
```

### Environment Configuration

```javascript
// config/env.js
const dotenv = require('dotenv');
const Joi = require('joi');

dotenv.config();

const envSchema = Joi.object({
  NODE_ENV: Joi.string()
    .valid('development', 'production', 'test')
    .default('development'),
  PORT: Joi.number().default(3000),
  MONGODB_URI: Joi.string().required(),
  REDIS_HOST: Joi.string().required(),
  REDIS_PORT: Joi.number().default(6379),
  JWT_SECRET: Joi.string().required(),
  JWT_EXPIRES_IN: Joi.string().default('7d'),
  LOG_LEVEL: Joi.string()
    .valid('error', 'warn', 'info', 'debug')
    .default('info'),
}).unknown();

const { value: env, error } = envSchema.validate(process.env);

if (error) {
  throw new Error(`Config validation error: ${error.message}`);
}

module.exports = {
  env: env.NODE_ENV,
  port: env.PORT,
  mongodb: {
    uri: env.MONGODB_URI,
  },
  redis: {
    host: env.REDIS_HOST,
    port: env.REDIS_PORT,
  },
  jwt: {
    secret: env.JWT_SECRET,
    expiresIn: env.JWT_EXPIRES_IN,
  },
  logging: {
    level: env.LOG_LEVEL,
  },
};
```

### Error Handling Best Practices

```javascript
// utils/apiError.js
class ApiError extends Error {
  constructor(statusCode, message, isOperational = true, stack = '') {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = isOperational;
    if (stack) {
      this.stack = stack;
    } else {
      Error.captureStackTrace(this, this.constructor);
    }
  }
}

module.exports = ApiError;

// utils/catchAsync.js
const catchAsync = (fn) => {
  return (req, res, next) => {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
};

module.exports = catchAsync;

// middleware/errorHandler.js
const config = require('../config/env');
const logger = require('../config/logger');
const ApiError = require('../utils/apiError');

const errorConverter = (err, req, res, next) => {
  let error = err;
  if (!(error instanceof ApiError)) {
    const statusCode = error.statusCode || 500;
    const message = error.message || 'Internal Server Error';
    error = new ApiError(statusCode, message, false, err.stack);
  }
  next(error);
};

const errorHandler = (err, req, res, next) => {
  let { statusCode, message } = err;

  if (config.env === 'production' && !err.isOperational) {
    statusCode = 500;
    message = 'Internal Server Error';
  }

  res.locals.errorMessage = err.message;

  const response = {
    code: statusCode,
    message,
    ...(config.env === 'development' && { stack: err.stack }),
  };

  if (config.env === 'development') {
    logger.error(err);
  }

  res.status(statusCode).json(response);
};

module.exports = {
  errorConverter,
  errorHandler,
};
```

### Testing Strategies

```javascript
// tests/integration/users.test.js
const request = require('supertest');
const app = require('../../src/app');
const { User } = require('../../src/models');

describe('User API', () => {
  beforeEach(async () => {
    await User.deleteMany({});
  });

  describe('POST /api/users', () => {
    it('should create a new user', async () => {
      const userData = {
        name: 'John Doe',
        email: 'john@example.com',
        password: 'password123',
      };

      const res = await request(app)
        .post('/api/users')
        .send(userData)
        .expect(201);

      expect(res.body).toHaveProperty('user');
      expect(res.body.user).toHaveProperty('id');
      expect(res.body.user.email).toBe(userData.email);
      expect(res.body.user).not.toHaveProperty('password');
    });

    it('should return 400 for invalid email', async () => {
      const userData = {
        name: 'John Doe',
        email: 'invalid-email',
        password: 'password123',
      };

      const res = await request(app)
        .post('/api/users')
        .send(userData)
        .expect(400);

      expect(res.body).toHaveProperty('error');
    });
  });

  describe('GET /api/users/:id', () => {
    it('should return user by id', async () => {
      const user = await User.create({
        name: 'John Doe',
        email: 'john@example.com',
        password: 'hashedpassword',
      });

      const res = await request(app)
        .get(`/api/users/${user.id}`)
        .expect(200);

      expect(res.body.user.id).toBe(user.id);
    });

    it('should return 404 for non-existent user', async () => {
      await request(app)
        .get('/api/users/507f1f77bcf86cd799439011')
        .expect(404);
    });
  });
});
```

### Performance Optimization

```javascript
// Enable gzip compression
const compression = require('compression');
app.use(compression());

// Use efficient JSON parsing
app.use(express.json({ limit: '10mb' }));

// Database query optimization
const getUsers = async (filters) => {
  return User.find(filters)
    .select('name email role') // Select only needed fields
    .lean() // Return plain objects instead of Mongoose documents
    .limit(100);
};

// Implement pagination
const paginateResults = async (model, page = 1, limit = 10) => {
  const skip = (page - 1) * limit;

  const [results, total] = await Promise.all([
    model.find().skip(skip).limit(limit).lean(),
    model.countDocuments(),
  ]);

  return {
    results,
    pagination: {
      page,
      limit,
      total,
      pages: Math.ceil(total / limit),
    },
  };
};

// Use indexes
userSchema.index({ email: 1 });
userSchema.index({ role: 1, createdAt: -1 });

// Connection pooling and keep-alive
const agent = new http.Agent({
  keepAlive: true,
  maxSockets: 50,
});
```

### Security Best Practices

```javascript
// Validate and sanitize inputs
const { body } = require('express-validator');

const userValidation = [
  body('email').isEmail().normalizeEmail(),
  body('password').isLength({ min: 8 }).trim(),
  body('name').trim().escape(),
];

// Implement rate limiting
const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
  skipSuccessfulRequests: true,
});

app.post('/api/auth/login', loginLimiter, loginHandler);

// Use parameterized queries
const getUserByEmail = async (email) => {
  return User.findOne({ email }); // Protected against NoSQL injection
};

// Implement CSRF protection
const csrf = require('csurf');
app.use(csrf({ cookie: true }));

// Set secure cookies
res.cookie('token', token, {
  httpOnly: true,
  secure: process.env.NODE_ENV === 'production',
  sameSite: 'strict',
  maxAge: 7 * 24 * 60 * 60 * 1000, // 7 days
});

// Hash passwords
const bcrypt = require('bcrypt');
const hashPassword = async (password) => {
  return bcrypt.hash(password, 12);
};
```

## Examples

### Example 1: Basic Express Microservice

```javascript
const express = require('express');
const helmet = require('helmet');
const cors = require('cors');
const morgan = require('morgan');

const app = express();

// Middleware
app.use(helmet());
app.use(cors());
app.use(morgan('combined'));
app.use(express.json());

// Routes
app.get('/health', (req, res) => {
  res.json({ status: 'healthy' });
});

app.get('/api/users', (req, res) => {
  res.json({ users: [] });
});

// Error handling
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: 'Something went wrong!' });
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

### Example 2: Authentication Service

```javascript
const express = require('express');
const jwt = require('jsonwebtoken');
const bcrypt = require('bcrypt');
const { body, validationResult } = require('express-validator');

const router = express.Router();

// Register
router.post('/register',
  body('email').isEmail(),
  body('password').isLength({ min: 8 }),
  async (req, res, next) => {
    try {
      const errors = validationResult(req);
      if (!errors.isEmpty()) {
        return res.status(400).json({ errors: errors.array() });
      }

      const { email, password } = req.body;

      // Check if user exists
      const existingUser = await User.findOne({ email });
      if (existingUser) {
        return res.status(409).json({ error: 'User already exists' });
      }

      // Hash password
      const hashedPassword = await bcrypt.hash(password, 12);

      // Create user
      const user = await User.create({
        email,
        password: hashedPassword,
      });

      // Generate token
      const token = jwt.sign(
        { userId: user.id },
        process.env.JWT_SECRET,
        { expiresIn: '7d' }
      );

      res.status(201).json({ token, user: { id: user.id, email: user.email } });
    } catch (error) {
      next(error);
    }
  }
);

// Login
router.post('/login',
  body('email').isEmail(),
  body('password').exists(),
  async (req, res, next) => {
    try {
      const errors = validationResult(req);
      if (!errors.isEmpty()) {
        return res.status(400).json({ errors: errors.array() });
      }

      const { email, password } = req.body;

      // Find user
      const user = await User.findOne({ email }).select('+password');
      if (!user) {
        return res.status(401).json({ error: 'Invalid credentials' });
      }

      // Check password
      const isValidPassword = await bcrypt.compare(password, user.password);
      if (!isValidPassword) {
        return res.status(401).json({ error: 'Invalid credentials' });
      }

      // Generate token
      const token = jwt.sign(
        { userId: user.id },
        process.env.JWT_SECRET,
        { expiresIn: '7d' }
      );

      res.json({ token, user: { id: user.id, email: user.email } });
    } catch (error) {
      next(error);
    }
  }
);

module.exports = router;
```

### Example 3: API Gateway Pattern

```javascript
const express = require('express');
const { createProxyMiddleware } = require('http-proxy-middleware');

const app = express();

// Service discovery (simplified)
const services = {
  users: 'http://users-service:3001',
  products: 'http://products-service:3002',
  orders: 'http://orders-service:3003',
};

// Authentication middleware
const authenticate = (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];

  if (!token) {
    return res.status(401).json({ error: 'No token provided' });
  }

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (error) {
    res.status(401).json({ error: 'Invalid token' });
  }
};

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100,
});

app.use(limiter);

// Proxy routes
app.use('/api/users', authenticate, createProxyMiddleware({
  target: services.users,
  changeOrigin: true,
  pathRewrite: { '^/api/users': '' },
}));

app.use('/api/products', createProxyMiddleware({
  target: services.products,
  changeOrigin: true,
  pathRewrite: { '^/api/products': '' },
}));

app.use('/api/orders', authenticate, createProxyMiddleware({
  target: services.orders,
  changeOrigin: true,
  pathRewrite: { '^/api/orders': '' },
}));

// Error handling
app.use((err, req, res, next) => {
  console.error(err);
  res.status(500).json({ error: 'Gateway error' });
});

const PORT = process.env.PORT || 8000;
app.listen(PORT, () => {
  console.log(`API Gateway running on port ${PORT}`);
});
```

See EXAMPLES.md for 15+ additional comprehensive examples including circuit breakers, event-driven patterns, service mesh integration, and more.

---

**Skill Version**: 1.0.0
**Last Updated**: October 2025
**Skill Category**: Microservices, Backend Development, Node.js, Production Architecture
**Compatible With**: Express.js 4.x/5.x, Node.js 16+, Docker, Kubernetes
