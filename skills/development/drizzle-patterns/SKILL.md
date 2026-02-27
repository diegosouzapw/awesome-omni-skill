---
name: drizzle-patterns
description: Drizzle ORM patterns for SQLite - queries, relations, and safety guidelines. Use when writing database queries or debugging issues.
globs:
  - src/db/**/*.ts
  - src/server/**/*.ts
alwaysApply: false
---

# Drizzle ORM Patterns

Database patterns specific to Drizzle ORM with SQLite.

## When to Use

- Writing database queries
- Defining table relations
- Performing safe updates/deletes
- Debugging query issues

## Key Files

- `src/db/index.ts` - Database instance
- `src/db/schema/` - Table definitions

## Pattern Files

- [queries.md](queries.md) - Query patterns and safety
