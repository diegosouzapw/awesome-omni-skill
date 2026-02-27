---
name: backend-dev-guidelines
description: Comprehensive backend development guide for Node.js/NestJS/TypeScript microservices. Use when creating controllers, services, database access, middleware, DTOs, or working with NestJS APIs, dependency injection, or async patterns. Covers layered architecture (controllers → services → dbservice), error handling, performance monitoring, testing strategies.
---

# Backend Development Guidelines

## Purpose

Establish consistency and best practices across backend microservices (web-server) using modern Node.js/NestJS/TypeScript patterns.

## When to Use

Auto-activates when working on:

- Creating or modifying routes, endpoints, APIs
- Building controllers, services, repositories
- Implementing middleware (auth, validation, error handling)
- DTOs with class-validator
- Database operations (via data-access-layer)
- Swagger documentation
- Authentication/authorization (login, logout, session management, RBAC)
- Guards and decorators (@Authorize, @IgnoreAuthorization)
- Logging and error handling
- Backend testing and refactoring

---

## CRITICAL: Implementation Workflow

**Before writing ANY code, you MUST:**

1. ✅ Read the relevant resource guides for your task (see Navigation Guide below)
2. ✅ Use the "New Backend Feature Checklist" as your TODO list
   **DO NOT:**

- ❌ Start by reading existing implementations
- ❌ Copy-paste from existing files without reading the guides
- ❌ Create an abbreviated version of the checklist

---

## Quick Start

### New Backend Feature Checklist

- [ ] **Dto**: Create [Entity]Dto in packages/types/src/dto/[entity].dto.ts (see [types-guide.md](resources/types-guide.md))
- [ ] **Entity**: Create [Entity] entity in packages/data-access-layer/src/features/[entity]/entities/[entity].entity.ts
- [ ] **DbService**: [Entity]DbService in packages/data-access-layer/src/features/[entity]/services/[entity]-db.service.ts (see [database-patterns-guide.md](resources/database-patterns-guide.md))
- [ ] **Controller**: [Entity]Controller in apps/web-server/src/app/features/[entity]/[entity].controller.ts (see [controllers-guide.md](resources/controllers-guide.md))
- [ ] **Service**: Create [Entity]Service in apps/web-server/src/app/features/[entity]/[entity].service.ts (see [services-guide.md](resources/services-guide.md))
- [ ] **Mapper**: [Entity]Mapper in apps/web-server/src/app/features/[entity]/[entity].mapper.ts (see [services-guide.md](resources/services-guide.md))
- [ ] **Register**: register new DbServices and Entities in `data-access.module` (see [database-patterns-guide.md](resources/database-patterns-guide.md))
- [ ] **Unit Tests**: Write comprehensive unit tests for all components with business logic (see [testing-guide.md](resources/testing-guide.md))
- [ ] **Verify Tests Pass**: Run: `npm run test`. 100% of tests should pass. If not fix until all pass
- [ ] **Code Coverage**: Run: `npm run test:coverage`. Review coverage report and write tests for any untested files with business logic. Achieve 80% coverage (statements, lines, branches 60%, functions 60%) (see [code-coverage-guide.md](resources/code-coverage-guide.md))
- [ ] **API Documentation**: Write API Endpoint documentation (see [api-documentation-guide.md](resources/api-documentation-guide.md))
- [ ] **Client API Generate**: Run: `npm run gen-api-client`
- [ ] **Verify Format**: Run: `npm run format:fix`
- [ ] **Verify Lint**: Run: `npm run lint`
- [ ] **Verify Build**: Run: `npm run build`

### New Microservice Checklist

- [ ] **Directory structure**: Create feature modules pattern (see [architecture-overview.md](resources/architecture-overview.md))
- [ ] **App Module**: Configure root module with imports, controllers, providers
- [ ] **Data Access Module**: Set up TypeORM connection and register entities
- [ ] **Exception Filter**: Register global `AllExceptionsFilter` in main.ts
- [ ] **Logging**: Configure `nestjs-pino` with proper log levels
- [ ] **Swagger**: Set up OpenAPI documentation in main.ts
- [ ] **Validation**: Enable global `ValidationPipe` with class-validator
- [ ] **Auth Guard**: Configure `@Authorize()` decorator and JWT validation
- [ ] **Testing**: Set up Jest with test-setup.ts configuration
- [ ] **Health Check**: Add `/health` endpoint for monitoring

---

## Architecture

### Layered Flow

```
HTTP Request → Controller → Service → DbService → TypeORM → Database
```

**Key Principle:** Each layer has ONE responsibility.

See [architecture-overview.md](resources/architecture-overview.md) for complete details.

---

## Directory Structure

```
  apps/web-server/src/
  ├── app/
  │   ├── auth/                          # Authentication feature
  │   │   ├── auth.controller.ts         # Auth endpoints (login, logout, etc.)
  │   │   ├── auth.service.ts            # Auth business logic
  │   │   ├── auth-mapper.service.ts     # DTO ↔ Entity mapping
  │   │   └── auth.module.ts             # Auth module
  │   │
  │   ├── features/                      # Feature modules
  │   │   ├── user/                      # User management feature
  │   │   │   ├── user.controller.ts     # User CRUD endpoints
  │   │   │   ├── user.service.ts        # User business logic
  │   │   │   ├── user.mapper.ts         # User DTO ↔ Entity mapping
  │   │   │   └── user.module.ts         # User module
  │   │   │
  │   │   ├── example/                   # Example feature (reference)
  │   │   │   ├── example.controller.ts
  │   │   │   ├── example.service.ts
  │   │   │   └── example.module.ts
  │   │   │
  │   │   ├── sync-events/               # Server-Sent Events feature
  │   │   │   ├── sync-events.controller.ts
  │   │   │   └── sync-events.module.ts
  │   │   │
  │   │   └── exceptions/                # Exception testing endpoints
  │   │       ├── exceptions.controller.ts
  │   │       └── exceptions.module.ts
  │   │
  │   ├── health/                        # Health check
  │   │   └── health.controller.ts       # Health endpoint
  │   │
  │   ├── app.module.ts                  # Root application module
  │   ├── app-initializer-service.ts     # App initialization logic
  │   └── data-access.module.ts          # Data access layer module setup
  │
  ├── common/                            # Shared utilities
  │   ├── base.mapper.ts                 # Base mapper class
  │   └── all-exceptions.filter.ts       # Global exception filter
  │
  ├── assets/                            # Static assets
  │
  ├── main.ts                            # Application entry point
  └── test-setup.ts                      # Jest test configuration
```

**Naming Conventions:**

- Files: `kebab-case` - `user.controller.ts`, `user.service.ts`, `user.mapper.ts`
- Classes: `PascalCase` - `UserController`, `UserService`, `UserMapper`

---

## Core Principles (8 Key Rules)

### 1. Let Services Throw, Controllers Delegate

```typescript
// ✅ Service throws business exceptions
async findOne(id: string): Promise<ClientUserDto> {
  const user = await this.userDbService.findById(id);
  if (!user) {
    throw new NotFoundException(`User ${id} not found`);
  }
  return this.userMapper.toDto(user);
}

// ✅ Controller just delegates - no try/catch needed
@Get(':id')
async findOne(@Param('id') id: string): Promise<ClientUserDto> {
  return this.userService.findOne(id);
}
```

**Key points:**

- ✅ Services throw `NotFoundException`, `ConflictException`, `BadRequestException`
- ✅ Controllers delegate to services - no try/catch blocks
- ✅ Global exception filter handles all errors automatically
- ❌ No manual `handleError()` methods needed

→ See [services-guide.md](resources/services-guide.md) and [controllers-guide.md](resources/controllers-guide.md)

### 2. Authorize requests using guard middleware

```typescript
  @Authorize(Role.Admin)
  async create(@Body(ValidationPipe) createUserDto: CreateUserDto): Promise<ClientUserDto> {
    return await this.userService.create(createUserDto);
  }
```

→ See [auth-session-guide.md](resources/auth-session-guide.md)

### 3. Validate input using class-validator

```typescript
  async create(@Body(ValidationPipe) createUserDto: CreateUserDto): Promise<ClientUserDto> {
    return this.userService.create(createUserDto);
  }
```

→ See [types-guide.md](resources/types-guide.md) and [controllers-guide.md](resources/controllers-guide.md)

### 4. Use Swagger decorators for API documentation

```typescript
     @ApiOperation({ summary: 'Create user' })
     @ApiResponse({ status: 201, type: ClientUserDto })
     async create(@Body(ValidationPipe) createUserDto: CreateUserDto): Promise<ClientUserDto> {
       return this.userService.create(createUserDto);
     }
```

→ See [api-documentation-guide.md](resources/api-documentation-guide.md)

### 5. Use Mapper to transform an entity to DTO

In the service use toDto() to return a DTO object

```typescript
     async create(createUserDto: CreateUserDto): Promise<ClientUserDto> {
       // Create via DbService
       const user = await this.userDbService.create(createUserDto);

       // Map Entity → DTO
       return this.userMapper.toDto(user); 
     }
```

→ See [services-guide.md](resources/services-guide.md)

### 6. Use DbService for database access

```typescript
// ✅ ALWAYS
const entity = await this.userDbService.create(createUserDto);
```

→ See [database-patterns-guide.md](resources/database-patterns-guide.md)

### 7. Constructor Conventions and Logger Setup

```typescript
@Injectable()
export class UserService {
  constructor(
    private readonly userDbService: UserDbService,
    private readonly userMapper: UserMapper,
    private readonly logger: PinoLogger,
  ) {
    this.logger.setContext(UserService.name); // REQUIRED
  }
}
```

**Key points:**

- ✅ Use `private readonly` for all injected dependencies
- ✅ Set logger context in constructor: `this.logger.setContext(ClassName.name)`
- ✅ Register providers in module's `providers` array

→ See [services-guide.md](resources/services-guide.md) and [logging-guide.md](resources/logging-guide.md)

### 8. Generate API Client After Controller Changes

**After modifying controllers, ALWAYS run:**

```bash
npm run gen-api-client
```

**What it does:**

- Reads NestJS controllers and decorators
- Extracts HTTP methods, paths, parameters, return types
- Generates type-safe Angular services in `@ai-nx-starter/api-client`
- Ensures frontend always matches backend API contract

**Example:**

```typescript
// Backend: UserController
@Post()
@ApiOperation({ summary: 'Create user' })
@ApiResponse({ status: 201, type: ClientUserDto })
async create(@Body() dto: CreateUserDto): Promise<ClientUserDto> { ... }

// ↓ Auto-generates ↓

// Frontend: ApiUserService
create(dto: CreateUserDto): Observable<ClientUserDto> {
  return this.http.post<ClientUserDto>(`${this.BASE_URL}/users`, dto);
}
```

**Why this matters:**

- ✅ Type-safe API calls in Angular
- ✅ Breaking changes caught at compile time
- ✅ No manual HTTP service writing
- ❌ Never create manual HTTP services - always auto-generate

---

## Navigation Guide

| Need to...                             | Read this                                                          |
| -------------------------------------- | ------------------------------------------------------------------ |
| Understand architecture                | [architecture-overview.md](resources/architecture-overview.md)     |
| Types Package, DTOs, enums, constants  | [types-guide.md](resources/types-guide.md)                         |
| Create controllers                     | [controllers-guide.md](resources/controllers-guide.md)             |
| Organize business logic and exceptions | [services-guide.md](resources/services-guide.md)                   |
| Database access                        | [database-patterns-guide.md](resources/database-patterns-guide.md) |
| Authentication & authorization         | [auth-session-guide.md](resources/auth-session-guide.md)           |
| Logging with Pino                      | [logging-guide.md](resources/logging-guide.md)                     |
| Security best practices                | [security-guide.md](resources/security-guide.md)                   |
| API Documentation                      | [api-documentation-guide.md](resources/api-documentation-guide.md) |
| Write tests                            | [testing-guide.md](resources/testing-guide.md)                     |
| Coverage exclusions                    | [code-coverage-guide.md](resources/code-coverage-guide.md)         |

---

## Resource Files

### [architecture-overview.md](resources/architecture-overview.md)

Layered architecture, request lifecycle, separation of concerns

### [types-guide.md](resources/types-guide.md)

Types Package & DTOs - Complete guide for DTOs, enums, constants, validation, naming conventions, and organization

### [controllers-guide.md](resources/controllers-guide.md)

Controller definitions, error handling, examples

### [services-guide.md](resources/services-guide.md)

Service patterns, Mapper integration, error handling, async patterns

### [database-patterns-guide.md](resources/database-patterns-guide.md)

DbService, TypeORM entities, MongoDB queries, data access patterns

### [auth-session-guide.md](resources/auth-session-guide.md)

Authentication, authorization, session management, RBAC, Redis sessions

### [logging-guide.md](resources/logging-guide.md)

Structured logging with Pino, PinoLogger usage, log levels, best practices

### [security-guide.md](resources/security-guide.md)

Input validation, authorization, sensitive data protection, password security, CORS, rate limiting

### [api-documentation-guide.md](resources/api-documentation-guide.md)

Swagger documentation guidelines for API endpoints

### [testing-guide.md](resources/testing-guide.md)

AI testing guidelines, coverage decisions, proactive test generation policy

### [code-coverage-guide.md](resources/code-coverage-guide.md)

Coverage exclusion guidelines, decision framework, what to test vs exclude

---

**Skill Status**: COMPLETE ✅
**Line Count**: < 500 ✅
**Progressive Disclosure**: 11 resource files ✅
