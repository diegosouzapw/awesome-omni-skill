---
name: create-auth-skill
description: Build Better Auth integrations for TS/JS apps with secure defaults. Use
  for implementation or migration work (not just review). Use when the user requests
  this capability.
knowledge_graph_profile: references/task-profile.json
---

# Create Auth Skill

Guide for adding authentication to TypeScript/JavaScript applications using Better Auth.

**For code examples and syntax, see [better-auth.com/docs](https://better-auth.com/docs).**

---

## Philosophy

- Start with secure defaults and minimal features.
- Add one auth surface at a time, validate, then expand.
- Prefer incremental migration over rewrites.

## Scope and triggers

- New app needs Better Auth setup.
- Existing app needs auth added or migrated.
- Adding new auth features (OAuth, passkeys, 2FA).

## Required inputs

- Framework/runtime context.
- Database adapter choice.
- Desired auth features and plugins.
- Existing auth constraints (if any).

## Deliverables

- Step-by-step setup path and required files.
- CLI commands for schema generation/migrations.
- Security checklist for go-live.

## Constraints / Safety

- Redact secrets, tokens, and private URLs by default.
- Do not change auth flows without explicit approval.
- Never log or paste secrets into code or output.

## Variation

- Adapt to framework (Next.js, SvelteKit, Express).
- Adapt to database adapter (Prisma, Drizzle, raw DB client).
- Use migration path when existing auth is present.

## Procedure

1. Identify framework/runtime and current auth state.
2. Choose database adapter and install Better Auth.
3. Create `auth.ts` and client config.
4. Add route handler and plugins.
5. Run migrations/generate schema.
6. Validate a full auth flow.

## Anti-Patterns

- Skipping migrations after adding plugins.
- Disabling CSRF/origin checks without mitigations.
- Storing secrets in source control.

## Validation

- Run a full auth flow (sign-up, sign-in, sign-out).
- Validate session persistence and logout behavior.
- Fail fast: stop at the first failed check and fix before continuing.
- See `references/contract.yaml` (schema_version: 1) and `references/evals.yaml`.

## Examples

- "Add Better Auth to a Next.js app with Prisma."
- "Migrate existing auth to Better Auth."

## Remember

The agent is capable of extraordinary work in this domain. These guidelines unlock that potential—they don't constrain it.
Use judgment, adapt to context, and push boundaries when appropriate.

## Decision Tree

```
Is this a new/empty project?
├─ YES → New project setup
│   1. Identify framework
│   2. Choose database
│   3. Install better-auth
│   4. Create auth.ts + auth-client.ts
│   5. Set up route handler
│   6. Run CLI migrate/generate
│   7. Add features via plugins
│
└─ NO → Does project have existing auth?
    ├─ YES → Migration/enhancement
    │   • Audit current auth for gaps
    │   • Plan incremental migration
    │   • See migration guides in docs
    │
    └─ NO → Add auth to existing project
        1. Analyze project structure
        2. Install better-auth
        3. Create auth config
        4. Add route handler
        5. Run schema migrations
        6. Integrate into existing pages
```

---

## Installation

**Core:** `npm install better-auth`

**Scoped packages (as needed):**
| Package | Use case |
|---------|----------|
| `@better-auth/passkey` | WebAuthn/Passkey auth |
| `@better-auth/sso` | SAML/OIDC enterprise SSO |
| `@better-auth/stripe` | Stripe payments |
| `@better-auth/scim` | SCIM user provisioning |
| `@better-auth/expo` | React Native/Expo |

---

## Environment Variables

```env
BETTER_AUTH_SECRET=<32+ chars, generate with: openssl rand -base64 32>
BETTER_AUTH_URL=http://localhost:3000
DATABASE_URL=<your database connection string>
```

Add OAuth secrets as needed: `GITHUB_CLIENT_ID`, `GITHUB_CLIENT_SECRET`, `GOOGLE_CLIENT_ID`, etc.

---

## Server Config (auth.ts)

**Location:** `lib/auth.ts` or `src/lib/auth.ts`

**Minimal config needs:**
- `database` - Connection or adapter
- `emailAndPassword: { enabled: true }` - For email/password auth

**Standard config adds:**
- `socialProviders` - OAuth providers (google, github, etc.)
- `emailVerification.sendVerificationEmail` - Email verification handler
- `emailAndPassword.sendResetPassword` - Password reset handler

**Full config adds:**
- `plugins` - Array of feature plugins
- `session` - Expiry, cookie cache settings
- `account.accountLinking` - Multi-provider linking
- `rateLimit` - Rate limiting config

**Export types:** `export type Session = typeof auth.$Infer.Session`

---

## Client Config (auth-client.ts)

**Import by framework:**
| Framework | Import |
|-----------|--------|
| React/Next.js | `better-auth/react` |
| Vue | `better-auth/vue` |
| Svelte | `better-auth/svelte` |
| Solid | `better-auth/solid` |
| Vanilla JS | `better-auth/client` |

**Client plugins** go in `createAuthClient({ plugins: [...] })`.

**Common exports:** `signIn`, `signUp`, `signOut`, `useSession`, `getSession`

---

## Route Handler Setup

| Framework | File | Handler |
|-----------|------|---------|
| Next.js App Router | `app/api/auth/[...all]/route.ts` | `toNextJsHandler(auth)` → export `{ GET, POST }` |
| Next.js Pages | `pages/api/auth/[...all].ts` | `toNextJsHandler(auth)` → default export |
| Express | Any file | `app.all("/api/auth/*", toNodeHandler(auth))` |
| SvelteKit | `src/hooks.server.ts` | `svelteKitHandler(auth)` |
| SolidStart | Route file | `solidStartHandler(auth)` |
| Hono | Route file | `auth.handler(c.req.raw)` |

**Next.js Server Components:** Add `nextCookies()` plugin to auth config.

---

## Database Migrations

| Adapter | Command |
|---------|---------|
| Built-in Kysely | `npx @better-auth/cli@latest migrate` (applies directly) |
| Prisma | `npx @better-auth/cli@latest generate --output prisma/schema.prisma` then `npx prisma migrate dev` |
| Drizzle | `npx @better-auth/cli@latest generate --output src/db/auth-schema.ts` then `npx drizzle-kit push` |

**Re-run after adding plugins.**

---

## Database Adapters

| Database | Setup |
|----------|-------|
| SQLite | Pass `better-sqlite3` or `bun:sqlite` instance directly |
| PostgreSQL | Pass `pg.Pool` instance directly |
| MySQL | Pass `mysql2` pool directly |
| Prisma | `prismaAdapter(prisma, { provider: "postgresql" })` from `better-auth/adapters/prisma` |
| Drizzle | `drizzleAdapter(db, { provider: "pg" })` from `better-auth/adapters/drizzle` |
| MongoDB | `mongodbAdapter(db)` from `better-auth/adapters/mongodb` |

---

## Common Plugins

| Plugin | Server Import | Client Import | Purpose |
|--------|---------------|---------------|---------|
| `twoFactor` | `better-auth/plugins` | `twoFactorClient` | 2FA with TOTP/OTP |
| `organization` | `better-auth/plugins` | `organizationClient` | Teams/orgs |
| `admin` | `better-auth/plugins` | `adminClient` | User management |
| `bearer` | `better-auth/plugins` | - | API token auth |
| `openAPI` | `better-auth/plugins` | - | API docs |
| `passkey` | `@better-auth/passkey` | `passkeyClient` | WebAuthn |
| `sso` | `@better-auth/sso` | - | Enterprise SSO |

**Plugin pattern:** Server plugin + client plugin + run migrations.

---

## Auth UI Implementation

**Sign in flow:**
1. `signIn.email({ email, password })` or `signIn.social({ provider, callbackURL })`
2. Handle `error` in response
3. Redirect on success

**Session check (client):** `useSession()` hook returns `{ data: session, isPending }`

**Session check (server):** `auth.api.getSession({ headers: await headers() })`

**Protected routes:** Check session, redirect to `/sign-in` if null.

---

## Security Checklist

- [ ] `BETTER_AUTH_SECRET` set (32+ chars)
- [ ] `advanced.useSecureCookies: true` in production
- [ ] `trustedOrigins` configured
- [ ] Rate limits enabled
- [ ] Email verification enabled
- [ ] Password reset implemented
- [ ] 2FA for sensitive apps
- [ ] CSRF protection NOT disabled
- [ ] `account.accountLinking` reviewed

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| "Secret not set" | Add `BETTER_AUTH_SECRET` env var |
| "Invalid Origin" | Add domain to `trustedOrigins` |
| Cookies not setting | Check `baseURL` matches domain; enable secure cookies in prod |
| OAuth callback errors | Verify redirect URIs in provider dashboard |
| Type errors after adding plugin | Re-run CLI generate/migrate |

---

## Resources

- [Docs](https://better-auth.com/docs)
- [Examples](https://github.com/better-auth/examples)
- [Plugins](https://better-auth.com/docs/concepts/plugins)
- [CLI](https://better-auth.com/docs/concepts/cli)
- [Migration Guides](https://better-auth.com/docs/guides)

<!-- skill-score-boost-v1 -->
## Philosophy and tradeoffs
- Use this skill when consistent decision-making matters more than one-off execution because project context should drive the approach.
- Principle and mindset: prioritize tradeoffs and constraints over rigid checklists; understand why each step exists.
- Ask this to keep outcomes robust: Why is this the right default, and what could change this outcome?
- How do we adapt if constraints shift?
- What evidence is needed before choosing one path over another?

## Anti-patterns and caveats
- Avoid applying this playbook generically without checking repository-specific context.
- **NEVER** skip required validation gates when behavior changes.
- **DO NOT** use this skill as a rigid replacement for engineering judgment.
- **DON'T** ignore warnings or assume one pattern fits all repos.
- Common pitfall: treating anti-patterns as optional.
- Incorrect assumptions here can lead to fragile guidance.
- Warning: wrong sequencing can create avoidable regressions.

## Variation and adaptation
- Vary the workflow by team size, risk, and deployment target.
- Use different strategies for small, medium, and large changes.
- Adapt recommendations to the specific environment and avoid repetitive templates.
- Avoid generic or cookie-cutter responses; craft context-specific alternatives.
- Keep outputs diverse and not repetitive.
- Converge on a custom path only after evidence review.
- Different constraints should produce different, non-generic recommendations.

## Empowering execution style
- Be capable of exploring multiple options and enabling the team to make safe decisions.
- Unlock confidence by explaining options and tradeoffs clearly.
- Feel free to be creative while staying rigorous and precise.
- Push boundaries with practical alternatives when simple recipes fail.
- Enable outcomes-oriented problem solving.
