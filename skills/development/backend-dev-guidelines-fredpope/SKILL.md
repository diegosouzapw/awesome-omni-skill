---
name: backend-dev-guidelines
description: Comprehensive backend development guide for Node.js/Express/TypeScript microservices. Use when creating routes, controllers, services, repositories, middleware, or working with Express APIs, Drizzle ORM database access, PostgreSQL/RDS, AWS Lambda handlers, SQS consumers, S3 operations, Sentry error tracking, Zod validation, unifiedConfig, dependency injection, or async patterns. Covers layered architecture (routes → controllers → services → repositories), BaseController pattern, error handling, performance monitoring, testing strategies with Playwright.
---

# Backend Development Guidelines

## Purpose

Establish consistency and best practices across backend services using modern Node.js/Express/TypeScript patterns with AWS infrastructure (Lambda, SQS, S3, RDS).

## When to Use This Skill

Automatically activates when working on:
- Creating or modifying routes, endpoints, APIs
- Building controllers, services, repositories
- Implementing middleware (auth, validation, error handling)
- Database operations with Drizzle ORM (PostgreSQL/RDS)
- AWS Lambda handlers
- SQS message consumers
- S3 file operations
- Error tracking with Sentry
- Input validation with Zod
- Configuration management
- Backend testing with Playwright

---

## Quick Start

### New Backend Feature Checklist

- [ ] **Route**: Clean definition, delegate to controller
- [ ] **Controller**: Extend BaseController
- [ ] **Service**: Business logic with DI
- [ ] **Repository**: Database access (if complex)
- [ ] **Validation**: Zod schema
- [ ] **Sentry**: Error tracking
- [ ] **Tests**: Unit + integration tests
- [ ] **Config**: Use unifiedConfig

### New Microservice Checklist

- [ ] Directory structure (see [architecture-overview.md](architecture-overview.md))
- [ ] instrument.ts for Sentry
- [ ] unifiedConfig setup
- [ ] BaseController class
- [ ] Middleware stack
- [ ] Error boundary
- [ ] Testing framework

---

## Architecture Overview

### Layered Architecture

```
HTTP Request / Lambda Event / SQS Message
    ↓
Routes/Handlers (routing only)
    ↓
Controllers (request handling)
    ↓
Services (business logic)
    ↓
Repositories (data access)
    ↓
Database (Drizzle ORM → PostgreSQL/RDS)
```

**Key Principle:** Each layer has ONE responsibility.

See [architecture-overview.md](architecture-overview.md) for complete details.

---

## Directory Structure

```
service/src/
├── config/              # UnifiedConfig
├── controllers/         # Request handlers
├── services/            # Business logic
├── repositories/        # Data access
├── routes/              # Route definitions
├── handlers/            # Lambda handlers
├── consumers/           # SQS consumers
├── middleware/          # Express middleware
├── db/                  # Drizzle schema & migrations
│   ├── schema.ts        # Drizzle table definitions
│   ├── index.ts         # DB connection
│   └── migrations/      # SQL migrations
├── types/               # TypeScript types
├── validators/          # Zod schemas
├── utils/               # Utilities
├── tests/               # Playwright tests
├── instrument.ts        # Sentry (FIRST IMPORT)
├── app.ts               # Express setup
└── server.ts            # HTTP server
```

**Naming Conventions:**
- Controllers: `PascalCase` - `UserController.ts`
- Services: `camelCase` - `userService.ts`
- Routes: `camelCase + Routes` - `userRoutes.ts`
- Repositories: `PascalCase + Repository` - `UserRepository.ts`

---

## Core Principles (7 Key Rules)

### 1. Routes Only Route, Controllers Control

```typescript
// ❌ NEVER: Business logic in routes
router.post('/submit', async (req, res) => {
    // 200 lines of logic
});

// ✅ ALWAYS: Delegate to controller
router.post('/submit', (req, res) => controller.submit(req, res));
```

### 2. All Controllers Extend BaseController

```typescript
export class UserController extends BaseController {
    async getUser(req: Request, res: Response): Promise<void> {
        try {
            const user = await this.userService.findById(req.params.id);
            this.handleSuccess(res, user);
        } catch (error) {
            this.handleError(error, res, 'getUser');
        }
    }
}
```

### 3. All Errors to Sentry

```typescript
try {
    await operation();
} catch (error) {
    Sentry.captureException(error);
    throw error;
}
```

### 4. Use unifiedConfig, NEVER process.env

```typescript
// ❌ NEVER
const timeout = process.env.TIMEOUT_MS;

// ✅ ALWAYS
import { config } from './config/unifiedConfig';
const timeout = config.timeouts.default;
```

### 5. Validate All Input with Zod

```typescript
const schema = z.object({ email: z.string().email() });
const validated = schema.parse(req.body);
```

### 6. Use Repository Pattern with Drizzle ORM

```typescript
// Service → Repository → Database (Drizzle)
import { db } from '@/db';
import { users } from '@/db/schema';
import { eq } from 'drizzle-orm';

const activeUsers = await db.select().from(users).where(eq(users.active, true));
```

### 7. Comprehensive Testing Required

```typescript
describe('UserService', () => {
    it('should create user', async () => {
        expect(user).toBeDefined();
    });
});
```

---

## Common Imports

```typescript
// Express
import express, { Request, Response, NextFunction, Router } from 'express';

// Validation
import { z } from 'zod';

// Database (Drizzle ORM)
import { drizzle } from 'drizzle-orm/node-postgres';
import { eq, and, or, desc, asc } from 'drizzle-orm';
import * as schema from './db/schema';

// AWS SDK v3
import { S3Client, GetObjectCommand, PutObjectCommand } from '@aws-sdk/client-s3';
import { SQSClient, SendMessageCommand } from '@aws-sdk/client-sqs';
import type { APIGatewayProxyHandler, SQSHandler } from 'aws-lambda';

// Sentry
import * as Sentry from '@sentry/node';

// Config
import { config } from './config/unifiedConfig';

// Middleware
import { SSOMiddlewareClient } from './middleware/SSOMiddleware';
import { asyncErrorWrapper } from './middleware/errorBoundary';
```

---

## Quick Reference

### HTTP Status Codes

| Code | Use Case |
|------|----------|
| 200 | Success |
| 201 | Created |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 500 | Server Error |

### Service Templates

**Blog API** (✅ Mature) - Use as template for REST APIs
**Auth Service** (✅ Mature) - Use as template for authentication patterns

---

## Anti-Patterns to Avoid

❌ Business logic in routes/handlers
❌ Direct process.env usage
❌ Missing error handling
❌ No input validation
❌ Raw SQL queries (use Drizzle)
❌ console.log instead of Sentry
❌ Synchronous S3/SQS operations without error handling

---

## Navigation Guide

| Need to... | Read this |
|------------|-----------|
| Understand architecture | [architecture-overview.md](architecture-overview.md) |
| Create routes/controllers | [routing-and-controllers.md](routing-and-controllers.md) |
| Organize business logic | [services-and-repositories.md](services-and-repositories.md) |
| Validate input | [validation-patterns.md](validation-patterns.md) |
| Add error tracking | [sentry-and-monitoring.md](sentry-and-monitoring.md) |
| Create middleware | [middleware-guide.md](middleware-guide.md) |
| Database access | [database-patterns.md](database-patterns.md) |
| Manage config | [configuration.md](configuration.md) |
| Handle async/errors | [async-and-errors.md](async-and-errors.md) |
| Write tests | [testing-guide.md](testing-guide.md) |
| See examples | [complete-examples.md](complete-examples.md) |

---

## Resource Files

### [architecture-overview.md](architecture-overview.md)
Layered architecture, request lifecycle, separation of concerns

### [routing-and-controllers.md](routing-and-controllers.md)
Route definitions, BaseController, error handling, examples

### [services-and-repositories.md](services-and-repositories.md)
Service patterns, DI, repository pattern, caching

### [validation-patterns.md](validation-patterns.md)
Zod schemas, validation, DTO pattern

### [sentry-and-monitoring.md](sentry-and-monitoring.md)
Sentry init, error capture, performance monitoring

### [middleware-guide.md](middleware-guide.md)
Auth, audit, error boundaries, AsyncLocalStorage

### [database-patterns.md](database-patterns.md)
Drizzle ORM, repositories, transactions, PostgreSQL/RDS optimization

### [configuration.md](configuration.md)
UnifiedConfig, environment configs, secrets

### [async-and-errors.md](async-and-errors.md)
Async patterns, custom errors, asyncErrorWrapper

### [testing-guide.md](testing-guide.md)
Unit/integration tests, mocking, coverage

### [complete-examples.md](complete-examples.md)
Full examples, refactoring guide

---

## Related Skills

- **database-verification** - Verify column names and schema consistency
- **error-tracking** - Sentry integration patterns
- **skill-developer** - Meta-skill for creating and managing skills

---

**Skill Status**: COMPLETE ✅
**Line Count**: < 500 ✅
**Progressive Disclosure**: 11 resource files ✅
