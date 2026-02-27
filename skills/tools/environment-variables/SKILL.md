---
name: environment-variables
description: Manage and validate environment variables using t3-env with Zod. Consult this skill whenever adding, modifying, or referencing environment variables, working with .env or .env.local files, configuring t3-env schemas, dealing with VITE_ prefixed variables, or separating client vs. server variables.
---

# Environment Variables

Patterns for managing and validating environment variables using t3-env with Zod.

Configuration file: `src/lib/env.ts`

## Variable Types

### Client Variables (`VITE_` prefix)

Variables the browser can read. Must have `VITE_` prefix to be exposed to client code.

- Public API endpoints
- Public keys (Stripe publishable key, analytics ID)
- Feature flags for UI

### Server Variables (no prefix)

Variables only available on the server. Never exposed to the browser.

- API secrets and private keys
- Database connection strings
- Internal service URLs

### Shared Variables

Values derived from the build environment, available in both contexts.

- `DEV` - Whether running in development mode
- `NODE_ENV` - Current environment

## Vite Exposure Rules

Vite only exposes variables prefixed with `VITE_` to client code via `import.meta.env`. Variables without this prefix are only available server-side.

| Prefix | Client | Server | Example |
|--------|--------|--------|---------|
| `VITE_*` | Yes | Yes | `VITE_API_URL` |
| No prefix | No | Yes | `DATABASE_URL` |

## Usage

```ts
import { env } from "@/lib/env";

console.log("Is dev?", env.DEV);
```

## Environment Files

The project uses two environment files with different purposes:

| File | Purpose | Git |
|------|---------|-----|
| `.env` | Development defaults (local URLs, non-sensitive config) | Committed |
| `.env.local` | Secrets, API keys, overrides | Ignored |

**Load order (highest priority first):**
1. `.env.local` - Overrides, secrets (never committed)
2. `.env` - Development defaults (committed)

This pattern allows:
- Easy onboarding - clone and run with sensible defaults
- Secrets stay out of git
- Production values set in deployment environment

## Adding New Variables

1. Add to `src/lib/env.ts` with Zod validation in appropriate section (`client`, `server`, or `shared`)
2. If it's a sensible development default (local URL, etc.): add to `.env`
3. If it's a secret or API key: add to `.env.local`
4. Configure in deployment environment
5. Add a test in the co-located test file (`src/lib/env.test.ts`) verifying the validation accepts correct values and rejects invalid ones.

## Quick Reference

| Task | Action |
|------|--------|
| Access env var | `import { env } from "@/lib/env"` |
| Client variable | Add to `client` with `VITE_` prefix |
| Server variable | Add to `server` (no prefix, never exposed to browser) |
| Shared variable | Add to `shared` (build-time values) |
| Validation | Use Zod schemas (`z.string()`, `z.boolean()`, etc.) |
