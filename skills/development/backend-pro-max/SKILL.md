---
name: backend-pro-max
description: "Backend development intelligence. Multi-stack (Node.js, PHP, Python) and Multi-DB (PostgreSQL, MySQL). Actions: build, create, design, implement, review, fix, improve, optimize backend. Topics: authentication, database, API, error handling, testing, security, architecture."
---

# Backend Pro Max - Backend Development Intelligence

Comprehensive backend guide for modern web applications. Supports multiple languages (Node.js, PHP, Python) and databases (PostgreSQL, MySQL). Contains 15+ architecture patterns, 25+ security rules, 20+ language-specific patterns, 15+ database patterns, 20+ API design patterns, and 20+ project-specific reasoning rules.

## When to Apply

Reference these guidelines when:
- Building new backend APIs or services in Node.js, PHP, or Python
- Designing and optimizing database schemas (PostgreSQL, MySQL)
- Implementing authentication/authorization
- Reviewing code for security issues
- Structuring backend project architecture

## Rule Categories by Priority

| Priority | Category | Impact | Domain |
|----------|----------|--------|--------|
| 1 | Security | CRITICAL | `security`, `auth` |
| 2 | Architecture | HIGH | `architecture`, `language` |
| 3 | Database | HIGH | `database` |
| 4 | API Design | MEDIUM | `api` |
| 5 | Error Handling | MEDIUM | `error` |
| 6 | Testing | MEDIUM | `testing` |

## Quick Reference

### 1. Security (CRITICAL)

- `input-validation` - Validate and sanitize all user inputs
- `jwt-secret` - Use strong, random JWT secrets (256-bit)
- `sql-injection` - Use parameterized queries, never concatenate SQL
- `rate-limiting` - Implement rate limiting on all public endpoints
- `secrets-management` - Never hardcode secrets, use environment variables

### 2. Architecture (HIGH)

- `clean-architecture` - Separate domain, application, infrastructure layers
- `dependency-injection` - Use DI for testability and maintainability
- `repository-pattern` - Abstract data access logic
- `service-layer` - Keep business logic in service layer

### 3. Database (HIGH)

- `connection-pooling` - Reuse connections with connection pool
- `transactions` - Use transactions for atomic operations
- `indexes` - Add indexes on frequently queried columns
- `n-plus-one` - Avoid N+1 query problem with eager loading

### 4. API Design (MEDIUM)

- `restful-resources` - Design resource-based endpoints
- `versioning` - Version your API (URL or header)
- `pagination` - Paginate large result sets
- `error-responses` - Use consistent error response format

### 5. Error Handling (MEDIUM)

- `custom-errors` - Create custom error classes
- `centralized-handler` - Use centralized error handling middleware
- `logging` - Log errors with context, don't log sensitive data

### 6. Testing (MEDIUM)

- `unit-tests` - Test business logic in isolation
- `integration-tests` - Test API endpoints with database
- `coverage` - Aim for 80%+ code coverage

## How to Use

When user requests backend work (build API, design database, implement auth, review security, optimize queries), follow this workflow:

### Step 1: Analyze Requirements

Extract key information:
- **Project type**: E-commerce, chat, CMS, etc.
- **Language**: nodejs, php, python (default: nodejs)
- **Database**: postgresql, mysql, mongodb (default: postgresql)
- **Features**: Authentication, real-time, file upload, etc.
- **Scale**: MVP, production-ready, enterprise-scale

### Step 2: Generate Backend System

Always start with `--backend-system`:

```bash
python3 .github/skills/backend-pro-max/scripts/search.py \
  "e-commerce API nodejs postgresql" \
  --backend-system \
  -p "My Shop API" \
  --lang nodejs \
  --db postgresql
```

This generates:
- Architecture recommendation
- Security checklist
- Database patterns
- API design patterns
- Error handling strategy
- Authentication method

### Step 3: Domain-Specific Searches

```bash
# Security patterns
python3 .github/skills/backend-pro-max/scripts/search.py \
  "jwt authentication" --domain auth --lang nodejs --db postgresql

# Database optimization
python3 .github/skills/backend-pro-max/scripts/search.py \
  "postgresql indexes" --domain database --lang nodejs --db postgresql

# API design
python3 .github/skills/backend-pro-max/scripts/search.py \
  "pagination filtering" --domain api --lang nodejs --db postgresql
```

## 🔴 HOW TO USE THE SYSTEM GENERATOR (CRITICAL)

Before generating ANY architecture, you MUST analyze the user's prompt to detect the requested tech stack.

1. Detect Language: Is the user asking for Node.js, PHP, or Python? (Default: nodejs)
2. Detect Database: Is the user asking for PostgreSQL or MySQL? (Default: postgresql)

You MUST run the generator with the detected stack using the `--lang` and `--db` flags:

### Examples:
If user asks: "Build a PHP e-commerce backend with MySQL"
```bash
python3 .github/skills/backend-pro-max/scripts/search.py "e-commerce API" --backend-system -p "E-commerce" --lang php --db mysql
```

If user asks: "Create a chat app" (No stack mentioned, use defaults)
```bash
python3 .github/skills/backend-pro-max/scripts/search.py "chat app" --backend-system -p "Chat API" --lang nodejs --db postgresql
```

## Example Prompts

- Build an e-commerce API with authentication and payment processing
- Create a REST API for a chat application with PostgreSQL
- Design database schema for a multi-tenant SaaS application
- Implement JWT authentication with refresh tokens
- Optimize slow PostgreSQL queries in user service
- Review this code for security vulnerabilities
