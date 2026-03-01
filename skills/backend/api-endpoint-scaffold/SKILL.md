---
name: api-endpoint-scaffold
description: Scaffold new Next.js API endpoints with authentication, rate limiting, and tests. Use when creating new API routes, adding endpoints, or building API features.
allowed-tools: Read, Grep, Glob, Write, Edit
---

# API Endpoint Scaffold

Creates production-ready API endpoints following this repo's patterns.

## When to Use

- "Create an endpoint for..."
- "Add API route for..."
- "I need a POST/GET endpoint"
- "Build an API for {feature}"

## Prerequisites

Before creating an endpoint, confirm:

1. Endpoint path (e.g., `/api/users/profile`)
2. HTTP methods needed (GET, POST, PUT, DELETE)
3. Authentication required? (default: yes)
4. Rate limiting config (requests/window)
5. Request/response schema

## Procedure

### Step 1: Create Route File

Path: `apps/web/app/api/{path}/route.ts`

Use the template in [templates.md](./templates.md).

### Step 2: Add Rate Limiting (if needed)

Import from existing pattern:

```typescript
import { withUserRateLimit } from '../_lib/withUserRateLimit';
import { createRateLimiter } from '@acme/security';
```

### Step 3: Add Request Validation

Use Zod for schema validation:

```typescript
import { z } from 'zod';

const RequestSchema = z.object({
  field: z.string().min(1),
});
```

### Step 4: Create Integration Test

Path: `packages/tests/src/{feature}.test.ts`

See [templates.md](./templates.md) for test template.

### Step 5: Verify

Run these commands:

1. `pnpm typecheck` - Type check
2. `pnpm lint` - Lint check
3. `pnpm test:integration` - Run tests

## Checklist

- [ ] Route file created at correct path
- [ ] Authentication check using `getCurrentUser()`
- [ ] Rate limiting applied via `withUserRateLimit`
- [ ] Request validation with Zod
- [ ] Proper error responses (400, 401, 403, 429, 500)
- [ ] Integration test created
- [ ] TypeScript types pass
- [ ] ESLint passes

## Guardrails

- ALWAYS use `getCurrentUser()` from `@acme/auth` for auth
- ALWAYS apply rate limiting to user-facing endpoints
- NEVER expose internal errors to clients
- NEVER skip request validation
- If unsure about rate limit config, ask user
