---
name: backend-patterns
description: Backend patterns for ORPC routers, Drizzle schemas, and server-side code. Use when creating API endpoints, database tables, or services.
globs:
  - src/server/**/*.ts
  - src/db/**/*.ts
  - src/app/api/**/*.ts
alwaysApply: false
---

# Backend Patterns

Use these patterns when creating API endpoints, database schemas, or server-side logic.

## When to Use

- Creating new ORPC router procedures
- Adding database tables or schema changes
- Implementing server-side business logic
- Adding authentication/authorization logic

## Key Files

- `src/server/create-api.ts` - API factory with dependency injection
- `src/server/context.ts` - Request context with session and db
- `src/server/index.ts` - Procedure definitions (publicProcedure, protectedProcedure)
- `src/server/routers/index.ts` - Router definitions
- `src/db/schema/` - Table definitions

## Pattern Files

- [schema.md](schema.md) - SQLite schema patterns
- [router.md](router.md) - ORPC router patterns
- [service.md](service.md) - Service factory patterns
