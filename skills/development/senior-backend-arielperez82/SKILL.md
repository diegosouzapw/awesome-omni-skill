---

# === CORE IDENTITY ===
name: senior-backend
title: Senior Backend Skill Package
description: Comprehensive backend development skill for building scalable backend systems using Node.js, Express, Go, Python, PostgreSQL, GraphQL, REST APIs. Includes API scaffolding, database optimization, security implementation, and performance tuning. Use when designing APIs, optimizing database queries, implementing business logic, handling authentication/authorization, or reviewing backend code.
domain: engineering
subdomain: backend-development

# === WEBSITE DISPLAY ===
difficulty: advanced
time-saved: "TODO: Quantify time savings"
frequency: "TODO: Estimate usage frequency"
use-cases:
  - Building robust API services with proper authentication and authorization
  - Designing database schemas and optimizing query performance
  - Implementing microservices patterns and service communication
  - Setting up CI/CD pipelines for backend applications

# === RELATIONSHIPS ===
related-agents: []
related-skills: []
related-commands: []
orchestrated-by: []

# === TECHNICAL ===
dependencies:
  scripts: []
  references: []
  assets: []
compatibility:
  python-version: 3.8+
  platforms: [macos, linux, windows]
tech-stack:
  - Node.js
  - Express
  - Go
  - Python
  - PostgreSQL
  - GraphQL
  - REST APIs
  - Docker
  - Kubernetes
  - Redis

# === EXAMPLES ===
examples:
  -
    title: Example Usage
    input: "TODO: Add example input for senior-backend"
    output: "TODO: Add expected output"

# === ANALYTICS ===
stats:
  downloads: 0
  stars: 0
  rating: 0.0
  reviews: 0

# === VERSIONING ===
version: v1.0.0
author: Claude Skills Team
contributors: []
created: 2025-10-19
updated: 2025-11-08
license: MIT

# === DISCOVERABILITY ===
tags:
  - api
  - backend
  - data
  - database
  - design
  - development
  - engineering
  - optimization
  - performance
  - security
featured: false
verified: true
---

# Senior Backend

## Core Capabilities

- **[Capability 1]** - [Description]
- **[Capability 2]** - [Description]
- **[Capability 3]** - [Description]
- **[Capability 4]** - [Description]


## Key Workflows

### Workflow 1: [Workflow Name]

**Time:** [Duration estimate]

**Steps:**
1. [Step 1]
2. [Step 2]
3. [Step 3]

**Expected Output:** [What success looks like]

### Workflow 2: [Workflow Name]

**Time:** [Duration estimate]

**Steps:**
1. [Step 1]
2. [Step 2]
3. [Step 3]

**Expected Output:** [What success looks like]


Expert backend development skill with comprehensive tools for building scalable, secure, and performant backend systems using modern tech stacks and architectural patterns.

## Overview

This skill provides production-ready backend development capabilities through three Python automation tools and extensive reference documentation. Whether building REST APIs, implementing GraphQL servers, designing database schemas, or optimizing performance, this skill ensures best practices and scalable architecture.

**What This Skill Provides:**
- API scaffolding for REST and GraphQL projects
- Database migration management and optimization
- Load testing and performance benchmarking
- Architecture patterns (layered, microservices, CQRS)
- Security best practices and authentication
- Comprehensive code templates and examples

**Use this skill when:**
- Designing and building backend APIs
- Implementing authentication and authorization
- Optimizing database queries and schemas
- Setting up microservices architecture
- Performance tuning and load testing
- Reviewing backend code for quality and security

## Quick Start

### API Scaffolding

```bash
# Create REST API with Express + TypeScript + PostgreSQL
python scripts/api_scaffolder.py my-api --type rest --stack express-typescript

# Create GraphQL API with Apollo Server
python scripts/api_scaffolder.py graphql-api --type graphql --stack apollo-typescript

# Start development
cd my-api
npm install
docker-compose up -d
npm run migrate
npm run dev
```

### Database Migrations

```bash
# Create new migration
python scripts/database_migration_tool.py create "add_user_table"

# Run pending migrations
python scripts/database_migration_tool.py migrate

# Rollback last migration
python scripts/database_migration_tool.py rollback

# Check migration status
python scripts/database_migration_tool.py status
```

### Load Testing

```bash
# Test API endpoint
python scripts/api_load_tester.py http://localhost:3000/api/users --users 50 --requests 1000

# Generate HTML report
python scripts/api_load_tester.py http://localhost:3000/api/users --users 100 --requests 1000 --output html --save report.html
```

## Core Workflows

### 1. New Backend Project Setup

**Steps:**
1. Scaffold project structure with api_scaffolder.py
   - Choose REST or GraphQL
   - Select tech stack (Express, Fastify, NestJS)
   - Enable authentication, Docker, CI/CD
2. Configure environment variables (.env)
3. Start services with Docker Compose
4. Run database migrations
5. Implement business logic in service layer
6. Add tests (unit and integration)

**See:** [templates.md](references/templates.md) for complete project templates and configuration examples.

### 2. API Design and Implementation

**REST API Design:**
- Resource-based URL structure
- Proper HTTP methods and status codes
- Request/response validation with Zod
- Error handling with custom error classes
- Rate limiting and authentication middleware

**GraphQL API Design:**
- Type-safe schema definitions
- Efficient resolver implementation
- DataLoader for N+1 query prevention
- Error handling and validation
- Field-level authorization

**See:** [frameworks.md](references/frameworks.md) for complete API design patterns, GraphQL schemas, and resolver examples.

### 3. Database Schema Design and Optimization

**Schema Design:**
1. Design entities and relationships (Prisma schema)
2. Create migration: `python scripts/database_migration_tool.py create "schema_name"`
3. Define indexes for query optimization
4. Add constraints and triggers
5. Run migration: `python scripts/database_migration_tool.py migrate`

**Query Optimization:**
- N+1 query prevention
- Proper indexing strategies
- Connection pooling
- Cursor-based pagination
- Caching with Redis

**See:** [frameworks.md](references/frameworks.md) for database optimization patterns and [templates.md](references/templates.md) for Prisma schema examples.

### 4. Authentication and Authorization

**Implementation:**
1. JWT-based authentication with bcrypt password hashing
2. Login/register endpoints
3. Authentication middleware for protected routes
4. Role-based authorization (RBAC)
5. Token refresh mechanism
6. Password reset flow with email

**See:** [templates.md](references/templates.md) for complete authentication service implementation and middleware examples.

### 5. Performance Testing and Optimization

**Load Testing Workflow:**
1. Establish baseline: `python scripts/api_load_tester.py <endpoint> --users 10 --requests 100`
2. Test under load: Increase concurrent users gradually
3. Identify bottlenecks (response times, error rates)
4. Optimize (caching, query optimization, connection pooling)
5. Re-test and compare results
6. Document capacity limits

**Optimization Techniques:**
- Redis caching (cache-aside pattern)
- Database query optimization
- Connection pooling
- Response compression
- API rate limiting

**See:** [frameworks.md](references/frameworks.md) for caching strategies and performance patterns.

## Python Tools

### api_scaffolder.py

Production-ready API project generator with complete infrastructure.

**Key Features:**
- REST and GraphQL API scaffolding
- Multiple tech stacks (Express, Fastify, NestJS, Apollo)
- Authentication setup (JWT + bcrypt)
- Database integration (Prisma ORM)
- Docker and docker-compose configuration
- Testing infrastructure (Jest + Supertest)
- CI/CD pipelines (GitHub Actions)
- Middleware (auth, validation, rate limiting, logging)

**Usage:**
```bash
# Express + TypeScript + PostgreSQL
python scripts/api_scaffolder.py my-api --type rest --stack express-typescript

# GraphQL with Apollo Server
python scripts/api_scaffolder.py gql-api --type graphql --auth

# Minimal setup without optional features
python scripts/api_scaffolder.py simple-api --minimal
```

**Generated Structure:** Controllers, services, repositories, middleware, routes, error handling, testing setup, Docker configuration.

**See:** [tools.md](references/tools.md) for complete documentation, options, and generated project structure.

### database_migration_tool.py

Comprehensive database migration management for schema versioning and rollbacks.

**Key Features:**
- Create, run, and rollback migrations
- Version tracking and migration history
- Transaction support for atomic operations
- Dry-run mode for testing changes
- Multi-database support (PostgreSQL, MySQL, MongoDB)
- Migration status reporting

**Usage:**
```bash
# Create migration
python scripts/database_migration_tool.py create "add_user_table"

# Run migrations
python scripts/database_migration_tool.py migrate

# Rollback
python scripts/database_migration_tool.py rollback --steps 2

# Check status
python scripts/database_migration_tool.py status
```

**Features:** UP/DOWN migrations, checksum validation, confirmation prompts, detailed logging.

**See:** [tools.md](references/tools.md) for migration file format, safety features, and workflow examples.

### api_load_tester.py

Advanced load testing tool for API performance benchmarking.

**Key Features:**
- Concurrent user simulation
- Customizable request scenarios
- Performance metrics (response times, RPS, throughput)
- Success/failure rates and error categorization
- Multiple output formats (text, JSON, HTML)
- Authentication and custom headers support

**Usage:**
```bash
# Simple load test
python scripts/api_load_tester.py http://localhost:3000/api/users --users 50 --requests 1000

# POST request test
python scripts/api_load_tester.py http://localhost:3000/api/users --method POST --data user.json

# Generate HTML report
python scripts/api_load_tester.py http://localhost:3000/api/users --users 100 --requests 1000 --output html --save report.html
```

**Metrics:** Min/max/avg/median/p95/p99 response times, requests per second, error rates, throughput.

**See:** [tools.md](references/tools.md) for testing scenarios, output examples, and capacity planning workflows.

## Reference Documentation

### Architecture Frameworks ([frameworks.md](references/frameworks.md))

Comprehensive architectural patterns and best practices:

- **API Design Patterns:** REST principles, GraphQL schemas, HTTP status codes, error responses
- **Architecture Patterns:** Layered architecture, dependency injection, repository pattern, CQRS
- **Microservices Patterns:** Service communication (sync/async), circuit breaker, API gateway
- **Performance Patterns:** Caching strategies (Redis), query optimization, connection pooling
- **Error Handling:** Custom error classes, global error handler, validation errors
- **Testing Strategies:** Unit testing with Jest, integration testing with Supertest
- **Documentation Standards:** Swagger/OpenAPI specification

### Implementation Templates ([templates.md](references/templates.md))

Production-ready code templates and examples:

- **Project Setup:** Express + TypeScript + Prisma starter, package.json, Dockerfile, docker-compose
- **Database Schema:** Prisma schema templates, migration files, indexes and constraints
- **Middleware:** Authentication, validation (Zod), rate limiting, logging (Winston)
- **Service Implementation:** Auth service (JWT + bcrypt), email service, CRUD services
- **Testing Templates:** Unit test examples, integration test setup, API testing patterns
- **API Routes:** RESTful route definitions, GraphQL resolvers, authentication integration
- **Environment Configuration:** .env.example with all required variables

### Python Tools Guide ([tools.md](references/tools.md))

Complete tool documentation:

- **api_scaffolder.py:** All options, supported stacks, generated structure, workflow examples
- **database_migration_tool.py:** Migration commands, file format, version tracking, rollback procedures
- **api_load_tester.py:** Testing scenarios, performance metrics, output formats, best practices

## Tech Stack

**Languages:** TypeScript, JavaScript, Python, Go
**Runtime:** Node.js 18+
**Frameworks:** Express, Fastify, NestJS, Apollo Server
**Database:** PostgreSQL, MySQL, MongoDB
**ORM:** Prisma, TypeORM, Sequelize
**Caching:** Redis
**Authentication:** JWT, bcrypt
**Testing:** Jest, Supertest
**Validation:** Zod
**API Docs:** Swagger/OpenAPI
**DevOps:** Docker, docker-compose, GitHub Actions
**Monitoring:** Winston (logging), Prometheus, Grafana

## Best Practices Summary

### Code Organization
- Use layered architecture (controllers → services → repositories)
- Implement dependency injection for testability
- Separate business logic from HTTP layer
- Use DTOs for request/response validation

### Security
- Hash passwords with bcrypt (10+ rounds)
- Implement JWT with proper expiration
- Validate all inputs with Zod schemas
- Use parameterized queries (Prisma prevents SQL injection)
- Rate limit authentication endpoints
- Use Helmet.js for security headers
- Never expose sensitive data in error messages

### Performance
- Cache frequently accessed data with Redis
- Optimize database queries (avoid N+1 problems)
- Use connection pooling
- Implement pagination (cursor-based for large datasets)
- Add database indexes on frequently queried fields
- Use compression middleware

### Testing
- Write unit tests for business logic
- Write integration tests for API endpoints
- Aim for 80%+ code coverage
- Mock external dependencies
- Test error scenarios
- Use factories for test data

### Database
- Use migrations for all schema changes
- Never modify migrations after deployment
- Always write DOWN migrations for rollback
- Use transactions for multi-step operations
- Add indexes on foreign keys
- Use appropriate data types (UUID vs integer IDs)

## Common Commands

```bash
# Development
npm run dev          # Start development server
npm run build        # Build for production
npm run start        # Start production server

# Database
npm run migrate      # Run migrations
npm run migrate:rollback  # Rollback last migration
npm run db:seed      # Seed database

# Testing
npm test             # Run all tests
npm run test:watch   # Run tests in watch mode
npm run test:coverage  # Generate coverage report

# Quality
npm run lint         # Run ESLint
npm run format       # Format code with Prettier
npm run type-check   # TypeScript type checking

# Docker
docker-compose up -d     # Start services
docker-compose down      # Stop services
docker-compose logs -f   # View logs
```

## Integration Points

This skill integrates with:
- **Frontend Skills:** REST/GraphQL API consumption
- **DevOps Skills:** Docker deployment, CI/CD pipelines
- **QA Skills:** API testing, integration testing
- **Security Skills:** Authentication, authorization, vulnerability scanning
- **Database Skills:** Schema design, query optimization

## Getting Help

1. **Architecture patterns:** See [frameworks.md](references/frameworks.md)
2. **Code templates:** See [templates.md](references/templates.md)
3. **Tool usage:** See [tools.md](references/tools.md) or run tools with `--help` flag
4. **Project setup:** Use api_scaffolder.py to generate boilerplate

---

**Version:** 1.0.0
**Last Updated:** 2025-11-08
**Documentation Structure:** Progressive disclosure with references/
