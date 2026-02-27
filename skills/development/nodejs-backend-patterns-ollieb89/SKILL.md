---
name: nodejs-backend-patterns
description: Build production-ready Node.js backend services with Express/Fastify, implementing middleware patterns, error handling, authentication, database integration, and API design best practices. Use when creating Node.js servers, REST APIs, GraphQL backends, or microservices architectures.
triggers:
  - Creating Node.js REST APIs
  - Setting up Express or Fastify
  - Implementing Node.js authentication
  - Building GraphQL backends
  - Creating Node.js microservices
  - Database integration with Node.js
---

# Node.js Backend Patterns

Comprehensive guidance for building scalable, maintainable, and production-ready Node.js backend applications with modern frameworks, architectural patterns, and best practices.

## When to Use This Skill

- Building REST APIs or GraphQL servers
- Creating microservices with Node.js
- Implementing authentication and authorization
- Designing scalable backend architectures
- Setting up middleware and error handling
- Integrating databases (SQL and NoSQL)
- Building real-time applications with WebSockets
- Implementing background job processing

## Core Frameworks

| Framework      | Best For                                                | Performance |
| -------------- | ------------------------------------------------------- | ----------- |
| **Express.js** | Flexibility, large ecosystem, quick prototyping         | Good        |
| **Fastify**    | High performance, TypeScript support, schema validation | Excellent   |

**Quick Recommendation:** Use **Express.js** for familiarity and ecosystem breadth. Use **Fastify** for performance-critical applications requiring built-in validation.

For detailed setup and patterns, see:

- [Express.js Patterns](./references/guides/express-patterns.md)
- [Fastify Patterns](./references/guides/fastify-patterns.md)

## Architectural Patterns

### Pattern 1: Layered Architecture

**Structure:**

```
src/
├── controllers/     # Handle HTTP requests/responses
├── services/        # Business logic
├── repositories/    # Data access layer
├── models/          # Data models
├── middleware/      # Express/Fastify middleware
├── routes/          # Route definitions
├── utils/           # Helper functions
├── config/          # Configuration
└── types/           # TypeScript types
```

**Service Layer Pattern:**

```typescript
// services/user.service.ts
import { UserRepository } from "../repositories/user.repository";
import { CreateUserDTO, UpdateUserDTO, User } from "../types/user.types";
import { NotFoundError, ValidationError } from "../utils/errors";
import bcrypt from "bcrypt";

export class UserService {
  constructor(private userRepository: UserRepository) {}

  async createUser(userData: CreateUserDTO): Promise<User> {
    const existingUser = await this.userRepository.findByEmail(userData.email);
    if (existingUser) {
      throw new ValidationError("Email already exists");
    }

    const hashedPassword = await bcrypt.hash(userData.password, 10);
    const user = await this.userRepository.create({
      ...userData,
      password: hashedPassword,
    });

    const { password, ...userWithoutPassword } = user;
    return userWithoutPassword as User;
  }

  async getUserById(id: string): Promise<User> {
    const user = await this.userRepository.findById(id);
    if (!user) {
      throw new NotFoundError("User not found");
    }
    const { password, ...userWithoutPassword } = user;
    return userWithoutPassword as User;
  }
}
```

### Pattern 2: Dependency Injection

**DI Container:**

```typescript
// di-container.ts
import { Pool } from "pg";

class Container {
  private instances = new Map<string, any>();

  register<T>(key: string, factory: () => T): void {
    this.instances.set(key, factory);
  }

  resolve<T>(key: string): T {
    const factory = this.instances.get(key);
    if (!factory) {
      throw new Error(`No factory registered for ${key}`);
    }
    return factory();
  }

  singleton<T>(key: string, factory: () => T): void {
    let instance: T;
    this.instances.set(key, () => {
      if (!instance) {
        instance = factory();
      }
      return instance;
    });
  }
}

export const container = new Container();

// Register dependencies
container.singleton(
  "db",
  () =>
    new Pool({
      host: process.env.DB_HOST,
      port: parseInt(process.env.DB_PORT || "5432"),
      database: process.env.DB_NAME,
      user: process.env.DB_USER,
      password: process.env.DB_PASSWORD,
      max: 20,
      idleTimeoutMillis: 30000,
      connectionTimeoutMillis: 2000,
    }),
);

container.singleton(
  "userRepository",
  () => new UserRepository(container.resolve("db")),
);

container.singleton(
  "userService",
  () => new UserService(container.resolve("userRepository")),
);
```

## Middleware & Validation

Middleware patterns are framework-specific. See detailed guides:

- [Express Middleware Patterns](./references/guides/express-patterns.md)
- [Fastify Plugin Patterns](./references/guides/fastify-patterns.md)

### Validation Schema Examples

Common validation patterns using Zod and Joi:

- [Validation Schema Examples](./references/examples/validation-schemas.md)

## Database Patterns

### PostgreSQL Connection Pool

```typescript
// config/database.ts
import { Pool, PoolConfig } from "pg";

const poolConfig: PoolConfig = {
  host: process.env.DB_HOST,
  port: parseInt(process.env.DB_PORT || "5432"),
  database: process.env.DB_NAME,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  max: 20,
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
};

export const pool = new Pool(poolConfig);

pool.on("connect", () => console.log("Database connected"));
pool.on("error", (err) => {
  console.error("Unexpected database error", err);
  process.exit(-1);
});

export const closeDatabase = async () => {
  await pool.end();
};
```

### MongoDB with Mongoose

```typescript
// config/mongoose.ts
import mongoose from "mongoose";

const connectDB = async () => {
  await mongoose.connect(process.env.MONGODB_URI!, {
    maxPoolSize: 10,
    serverSelectionTimeoutMS: 5000,
    socketTimeoutMS: 45000,
  });
  console.log("MongoDB connected");
};

// Model example
interface IUser extends Document {
  name: string;
  email: string;
  password: string;
}

const userSchema = new Schema<IUser>(
  {
    name: { type: String, required: true },
    email: { type: String, required: true, unique: true },
    password: { type: String, required: true },
  },
  { timestamps: true },
);

userSchema.index({ email: 1 });
export const User = model<IUser>("User", userSchema);
```

### Transaction Pattern

```typescript
// services/order.service.ts
import { Pool } from "pg";

export class OrderService {
  constructor(private db: Pool) {}

  async createOrder(userId: string, items: any[]) {
    const client = await this.db.connect();
    try {
      await client.query("BEGIN");

      const orderResult = await client.query(
        "INSERT INTO orders (user_id, total) VALUES ($1, $2) RETURNING id",
        [userId, calculateTotal(items)],
      );
      const orderId = orderResult.rows[0].id;

      for (const item of items) {
        await client.query(
          "INSERT INTO order_items (order_id, product_id, quantity, price) VALUES ($1, $2, $3, $4)",
          [orderId, item.productId, item.quantity, item.price],
        );
      }

      await client.query("COMMIT");
      return orderId;
    } catch (error) {
      await client.query("ROLLBACK");
      throw error;
    } finally {
      client.release();
    }
  }
}
```

## Authentication & Authorization

### JWT Authentication Service

```typescript
// services/auth.service.ts
import jwt from "jsonwebtoken";
import bcrypt from "bcrypt";
import { UnauthorizedError } from "../utils/errors";

export class AuthService {
  constructor(private userRepository: UserRepository) {}

  async login(email: string, password: string) {
    const user = await this.userRepository.findByEmail(email);
    if (!user) {
      throw new UnauthorizedError("Invalid credentials");
    }

    const isValid = await bcrypt.compare(password, user.password);
    if (!isValid) {
      throw new UnauthorizedError("Invalid credentials");
    }

    const token = jwt.sign(
      { userId: user.id, email: user.email },
      process.env.JWT_SECRET!,
      { expiresIn: "15m" },
    );

    const refreshToken = jwt.sign(
      { userId: user.id },
      process.env.REFRESH_TOKEN_SECRET!,
      { expiresIn: "7d" },
    );

    return {
      token,
      refreshToken,
      user: { id: user.id, name: user.name, email: user.email },
    };
  }
}
```

## Caching Strategies

```typescript
// utils/cache.ts
import Redis from "ioredis";

const redis = new Redis({
  host: process.env.REDIS_HOST,
  port: parseInt(process.env.REDIS_PORT || "6379"),
});

export class CacheService {
  async get<T>(key: string): Promise<T | null> {
    const data = await redis.get(key);
    return data ? JSON.parse(data) : null;
  }

  async set(key: string, value: any, ttl?: number): Promise<void> {
    const serialized = JSON.stringify(value);
    if (ttl) {
      await redis.setex(key, ttl, serialized);
    } else {
      await redis.set(key, serialized);
    }
  }

  async invalidatePattern(pattern: string): Promise<void> {
    const keys = await redis.keys(pattern);
    if (keys.length > 0) {
      await redis.del(...keys);
    }
  }
}

// Cache decorator
export function Cacheable(ttl: number = 300) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor,
  ) {
    const originalMethod = descriptor.value;
    descriptor.value = async function (...args: any[]) {
      const cache = new CacheService();
      const cacheKey = `${propertyKey}:${JSON.stringify(args)}`;
      const cached = await cache.get(cacheKey);
      if (cached) return cached;
      const result = await originalMethod.apply(this, args);
      await cache.set(cacheKey, result, ttl);
      return result;
    };
    return descriptor;
  };
}
```

## Security Best Practices

For comprehensive security guidelines, see:

- [API Security Checklist](./references/checklists/api-security.md)

### Quick Security Summary

1. **Always use HTTPS** in production
2. **Validate all inputs** using Zod or Joi
3. **Hash passwords** with bcrypt (cost 10-12)
4. **Use JWT** with short expiration for access tokens
5. **Implement rate limiting** on all endpoints
6. **Use security headers** (Helmet)
7. **Configure CORS** properly (no wildcards in production)
8. **Never expose** stack traces or sensitive data in errors
9. **Use parameterized queries** to prevent SQL injection
10. **Implement proper logging** without sensitive data

## Best Practices Summary

1. **Use TypeScript** - Type safety prevents runtime errors
2. **Implement proper error handling** - Use custom error classes
3. **Validate input** - Use libraries like Zod or Joi
4. **Use environment variables** - Never hardcode secrets
5. **Implement logging** - Use structured logging (Pino, Winston)
6. **Add rate limiting** - Prevent abuse
7. **Use HTTPS** - Always in production
8. **Implement CORS properly** - Don't use `*` in production
9. **Use dependency injection** - Easier testing and maintenance
10. **Write tests** - Unit, integration, and E2E tests
11. **Handle graceful shutdown** - Clean up resources
12. **Use connection pooling** - For databases
13. **Implement health checks** - For monitoring
14. **Use compression** - Reduce response size
15. **Monitor performance** - Use APM tools

## Testing Patterns

See `javascript-testing-patterns` skill for comprehensive testing guidance.

## Resources

- **Node.js Best Practices**: https://github.com/goldbergyoni/nodebestpractices
- **Express.js Guide**: https://expressjs.com/en/guide/
- **Fastify Documentation**: https://www.fastify.io/docs/
- **TypeScript Node Starter**: https://github.com/microsoft/TypeScript-NodeStarter
