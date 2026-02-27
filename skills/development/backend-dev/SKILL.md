---
name: backend-dev
description: "Backend development workflow with API design and data modeling. Trigger: When building, refactoring, or scaling backend apps."
license: "Apache 2.0"
metadata:
  version: "1.0"
  type: universal
---

# Backend Development Skill

Universal backend workflow guiding API design, data modeling, error handling, and deployment. Technology-agnostic, orchestrates technical skills (nodejs, express, nest) without duplicating patterns.

## When to Use

- Designing, building, or refactoring APIs
- Modeling data and business logic
- Preparing for deployment or CI/CD
- Reviewing code quality

Don't use for:

- Technology-specific code (use nodejs, express, nest skills)
- Frontend development (use frontend-dev skill)

---

## Critical Patterns

### ✅ REQUIRED: API Design with Versioning

Define clear, versioned contracts to prevent breaking changes.

**Decision Tree**:

```
Creating new endpoint?
  → Is this a new feature? → /api/v1/feature
  → Breaking change to existing endpoint? → Create /api/v2/endpoint, deprecate v1
  → Non-breaking addition (new field)? → Add to current version

Versioning strategy?
  → URL versioning (/api/v1, /api/v2) - RECOMMENDED (clear, cacheable)
  → Header versioning (Accept: application/vnd.api.v2+json) - Advanced cases
  → Query param (?version=2) - Avoid (harder to route, cache)
```

**Implementation**: Delegate to framework skills ([express](../express/SKILL.md), [nest](../nest/SKILL.md), [hono](../hono/SKILL.md))

**Deprecation Timeline**:

1. Announce deprecation (release notes, docs, headers)
2. Maintain both versions (3-6 months)
3. Remove deprecated version (after migration period)

**Example Versioning Strategy**:

```
v1 (current production)
  → Add v2 with breaking changes
  → v1 returns Deprecation header: "Deprecated. Use /api/v2. Sunset: 2026-09-01"
  → Monitor v1 usage (analytics)
  → Remove v1 after sunset date
```

### ✅ REQUIRED: Data Modeling with Validation

Validate at boundaries and separate domain logic from persistence.

**Decision Tree**:

```
Validating input?
  → At API boundary (controller/route)? → YES (always validate)
  → In domain logic? → NO (assume valid after boundary check)
  → In database layer? → NO (database constraints are last resort)

Validation library?
  → TypeScript-first? → zod (RECOMMENDED)
  → Existing project with yup? → yup (maintain consistency)
  → Legacy JavaScript? → joi
```

**Implementation**: Delegate to [form-validation](../form-validation/SKILL.md) for validation patterns

**Validation Layers**:

1. **API Boundary** (controller): Validate shape, types, format (zod/yup)
2. **Domain Logic** (service): Business rules (e.g., "cannot delete user with active orders")
3. **Database** (schema): Constraints as safety net (NOT NULL, UNIQUE, CHECK)

**Example Data Flow**:

```
Client request
  ↓
Controller validates input (zod schema)
  ↓
Service applies business logic
  ↓
Repository persists to database
  ↓
Database enforces constraints (last resort)
```

### ✅ REQUIRED: Centralized Error Handling

Consistent error responses and logging across all endpoints.

**Decision Tree**:

```
Handling errors?
  → Operational error (expected, like 404)? → AppError class, send to client
  → Programming error (unexpected, like null reference)? → Log, send generic 500

Error response format?
  → REST API? → { status: "error", message: "...", code: "USER_NOT_FOUND" }
  → GraphQL? → errors array with extensions { code, message }
  → RFC 7807? → Problem Details JSON (type, title, status, detail)
```

**Implementation**: Delegate to framework skills ([express](../express/SKILL.md), [nest](../nest/SKILL.md))

**Error Categories**:

- **Validation errors** (400): Input doesn't match schema
- **Authentication errors** (401): Missing or invalid credentials
- **Authorization errors** (403): Valid credentials, insufficient permissions
- **Not found errors** (404): Resource doesn't exist
- **Conflict errors** (409): State conflict (e.g., duplicate email)
- **Server errors** (500): Unhandled exceptions

**Error Context to Log**:

- Request ID (for tracing)
- User ID (if authenticated)
- Endpoint + method (GET /api/v1/users)
- Timestamp
- Stack trace (for 500 errors only)
- NEVER log passwords or tokens

### ✅ REQUIRED: Environment-Based Configuration

Never hardcode config values. Use environment variables.

**Decision Tree**:

```
Need configuration?
  → Database URL? → DATABASE_URL
  → API keys/secrets? → JWT_SECRET, STRIPE_KEY (NEVER commit)
  → Feature flags? → ENABLE_FEATURE=true/false
  → Port? → PORT (with fallback to 3000)

Validating env vars?
  → Yes, ALWAYS at startup (fail fast if missing)
  → Use zod to validate env schema
```

**Implementation**: Delegate to [nodejs](../nodejs/SKILL.md) for process.env patterns

**Configuration Pattern**:

```
env.ts
  → Validates all required env vars at startup
  → Exports typed config object

.env.example
  → Documents all required env vars (committed)

.env
  → Actual values (NOT committed, in .gitignore)
```

**Fail Fast on Missing Env Vars**:

```
App startup
  → Load .env file
  → Validate env vars with zod schema
  → If validation fails → Throw error, exit process (don't start server)
  → If validation passes → Continue startup
```

### ✅ REQUIRED: API Contract Design

**When designing API contracts**:

1. **Identify resources**: What entities exist? (users, products, orders)
2. **Define operations**: CRUD (POST create, GET read, PATCH update, DELETE delete)
3. **Model relationships**: users.orders (1:N), orders.products (N:M)
4. **Version from start**: Start with /api/v1 (easier to add v2 later)

**RESTful Patterns**:

```
GET /api/v1/users → List users (pagination, filters)
GET /api/v1/users/:id → Get single user
POST /api/v1/users → Create user
PATCH /api/v1/users/:id → Update user (partial)
DELETE /api/v1/users/:id → Delete user

Relationships:
GET /api/v1/users/:id/orders → User's orders (nested resource)
```

**Red Flags**:

- Non-RESTful URLs: /api/getUserById?id=123 (use GET /api/v1/users/123)
- Verbs in URLs: /api/deleteUser (use DELETE /api/v1/users/:id)
- No versioning: /api/users (add /v1 from start)

### ✅ REQUIRED: Data Persistence Strategy

**Decision Tree**:

```
Choosing database?
  → Structured data + relations? → PostgreSQL (RECOMMENDED)
  → Document-oriented + flexible schema? → MongoDB
  → Key-value cache? → Redis
  → Time-series data? → TimescaleDB, InfluxDB

ORM vs Query Builder?
  → TypeScript + type safety? → Prisma (RECOMMENDED)
  → Need raw SQL flexibility? → Knex, Drizzle
  → Existing project with TypeORM? → TypeORM (maintain consistency)
```

**Implementation**: Delegate to [nodejs](../nodejs/SKILL.md) for database patterns

**Repository Pattern**:

```
Controller (HTTP layer)
  ↓
Service (Business logic)
  ↓
Repository (Data access - abstraction over database)
  ↓
Database (PostgreSQL, MongoDB, etc.)
```

**Benefits of Repository Pattern**:

- Easy to test (mock repository)
- Database-agnostic (swap Postgres → MongoDB)
- Centralized query logic

### ✅ REQUIRED: Performance Optimization

**When to optimize**:

1. Measure FIRST (APM tools, profiling, load testing)
2. Identify bottlenecks (slow queries, N+1 problems, large payloads)
3. Apply targeted fixes (NOT premature optimization)

**Common Optimizations**:

- **Database queries**: Add indexes, use EXPLAIN ANALYZE, fix N+1 queries
- **Caching**: Redis for frequently accessed data (user sessions, product catalog)
- **Pagination**: Limit list endpoints to 50-100 items per page
- **Compression**: gzip middleware for response bodies
- **Rate limiting**: Prevent abuse, protect from DDoS

**Red Flags (Premature Optimization)**:

- Caching everything "just in case"
- Denormalizing database before profiling queries
- Optimizing endpoints with <100 requests/day

---

## Decision Tree

```
New endpoint?
  → Define contract/schema first, then document

Data model change?
  → Migrate safely, validate with staging data

Deployment?
  → Automate with CI/CD pipeline

Bug found?
  → Add/expand test coverage before fixing
```

---

## Example

Building a `POST /api/v1/orders` endpoint end-to-end: validation → service → repository → response.

```typescript
// 1. Controller: validate input, delegate to service
app.post("/api/v1/orders", async (req, res) => {
  const parsed = createOrderSchema.safeParse(req.body);  // zod boundary check
  if (!parsed.success) return res.status(400).json({ error: parsed.error.flatten() });

  const result = await orderService.createOrder(parsed.data, req.user.id);
  if (!result.isSuccess) return res.status(409).json({ error: result.error });
  res.status(201).json(result.value);
});

// 2. Service: business rules
class OrderService {
  async createOrder(dto: CreateOrderDTO, userId: string): Promise<Result<Order>> {
    const user = await this.userRepo.findById(userId);
    if (!user) return Result.fail("User not found");
    if (dto.items.length === 0) return Result.fail("Order must have at least one item");
    const order = Order.create({ ...dto, userId });
    await this.orderRepo.save(order);
    return Result.ok(order);
  }
}

// 3. Repository: data access abstraction
class PostgresOrderRepository implements IOrderRepository {
  async save(order: Order): Promise<void> {
    await prisma.order.create({ data: order.toPersistence() });
  }
}
```

Patterns applied: `/api/v1` versioning, zod boundary validation, service owns business rules, repository abstracts DB, centralized error format.

## Edge Cases

- **Data migration failures**: Always implement rollback strategy. Use transactions for multi-step migrations. Test migrations on staging data first.

- **API versioning and backward compatibility**: Maintain old versions until all clients migrate. Document deprecation timeline (e.g., "v1 deprecated 2026-06-01, removed 2026-09-01").

- **Security edge cases**: Validate all inputs to prevent injection attacks. Implement rate limiting per endpoint. Use parameterized queries for SQL. Sanitize user input before logging.

- **Race conditions**: Use database transactions for operations that must be atomic. Implement optimistic locking for concurrent updates. Consider distributed locks for multi-instance deployments.

- **Large response payloads**: Implement pagination for list endpoints. Use streaming for large files. Consider compression (gzip) for response bodies.

---

## Checklist

- [ ] API endpoints versioned (/api/v1, /api/v2)
- [ ] Input validation at all boundaries (zod, yup, joi)
- [ ] Centralized error handling middleware
- [ ] Environment variables for all configuration
- [ ] Database migrations tested with rollback
- [ ] Authentication and authorization on protected routes
- [ ] Rate limiting implemented on public endpoints
- [ ] Logging with context (request ID, user ID, timestamp)
- [ ] API documentation up-to-date (OpenAPI, Swagger)
- [ ] Unit tests for business logic (>=80% coverage)
- [ ] Integration tests for critical flows
- [ ] CI/CD pipeline configured (build, test, deploy)
- [ ] Health check endpoint (/health, /ping)
- [ ] Monitoring and alerting configured (errors, performance)

---

## Workflow

**E2E Development Workflow**:

1. **brainstorming** → Evaluate alternatives, high-level planning
2. **writing-plans** → Break into executable tasks (2-5 min, file paths)
3. **backend-dev** (this skill) → Apply Backend Developer thinking and architecture
4. **plan-execution** → Execute in batches of 3 with checkpoints
5. **verification-protocol** → Verify each task (IDENTIFY → RUN → READ → VERIFY → CLAIM)
6. **code-review** → Two-stage review (spec compliance → code quality)

---

## Resources

- [code-conventions](../code-conventions/SKILL.md) - Code organization and naming
- [architecture-patterns](../architecture-patterns/SKILL.md) - Design patterns (Repository, Service Layer, Clean Architecture)
- [nodejs](../nodejs/SKILL.md) - Node.js runtime patterns
- [typescript](../typescript/SKILL.md) - Type-safe backend development
- [form-validation](../form-validation/SKILL.md) - Input validation with zod/yup
- [express](../express/SKILL.md) - Express.js framework patterns
- [nest](../nest/SKILL.md) - NestJS framework patterns
- [hono](../hono/SKILL.md) - Hono framework patterns
- https://restfulapi.net/ - REST API design best practices
- https://swagger.io/docs/ - OpenAPI/Swagger documentation
