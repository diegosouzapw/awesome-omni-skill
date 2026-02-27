---
name: better-auth-patterns
description: Better Auth authentication patterns for TypeScript applications. Use when implementing authentication with Better Auth, configuring OAuth providers, setting up session management, integrating with Next.js/Astro/Hono/Express/TanStack Start, or configuring Drizzle/Prisma adapters.
---

# Better Auth Patterns

## Purpose

Comprehensive patterns for implementing authentication with Better Auth across frameworks. Covers server and client setup, database adapters, OAuth providers, session management, middleware, plugins (2FA, admin, organization, magic link, passkey, API keys), and production security hardening.

## Server Setup

### Core Configuration

```typescript
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  appName: "My App",
  baseURL: process.env.BETTER_AUTH_URL,        // required in production
  basePath: "/api/auth",                        // default mount path
  secret: process.env.BETTER_AUTH_SECRET,       // min 32 chars, generate: openssl rand -base64 32
  database: /* adapter — see Database section */,
  trustedOrigins: ["https://example.com"],      // required in production
  emailAndPassword: { enabled: true },
  socialProviders: { /* see OAuth section */ },
  plugins: [],
});
```

Environment variables:

```bash
BETTER_AUTH_SECRET=<openssl rand -base64 32>   # required
BETTER_AUTH_URL=http://localhost:3000           # required
```

### Next.js (App Router)

```typescript
// app/api/auth/[...all]/route.ts
import { auth } from "@/lib/auth";
import { toNextJsHandler } from "better-auth/next-js";

export const { POST, GET } = toNextJsHandler(auth);
```

For server actions that set cookies, add the `nextCookies` plugin (must be last):

```typescript
import { betterAuth } from "better-auth";
import { nextCookies } from "better-auth/next-js";

export const auth = betterAuth({
  plugins: [nextCookies()],  // must be last plugin
});
```

### Next.js (Pages Router)

```typescript
// pages/api/auth/[...all].ts
import { toNodeHandler } from "better-auth/node";
import { auth } from "@/lib/auth";

export const config = { api: { bodyParser: false } };
export default toNodeHandler(auth.handler);
```

### Astro

```typescript
// pages/api/auth/[...all].ts
import { auth } from "~/auth";
import type { APIRoute } from "astro";

export const ALL: APIRoute = async (ctx) => {
  return auth.handler(ctx.request);
};
```

### Hono

```typescript
import { Hono } from "hono";
import { cors } from "hono/cors";
import { auth } from "./auth";

const app = new Hono();

// CORS must be registered before routes
app.use("/api/auth/*", cors({
  origin: "http://localhost:3001",
  allowHeaders: ["Content-Type", "Authorization"],
  allowMethods: ["POST", "GET", "OPTIONS"],
  credentials: true,
}));

app.on(["POST", "GET"], "/api/auth/*", (c) => auth.handler(c.req.raw));
```

### Express

```typescript
import express from "express";
import { toNodeHandler } from "better-auth/node";
import { auth } from "./auth";

const app = express();

// Better Auth handler MUST come before express.json()
app.all("/api/auth/*splat", toNodeHandler(auth));  // v5 syntax
app.use(express.json());
```

### TanStack Start

```typescript
// src/routes/api/auth/$.ts
import { auth } from "@/lib/auth";
import { createFileRoute } from "@tanstack/react-router";

export const Route = createFileRoute("/api/auth/$")({
  server: {
    handlers: {
      GET: async ({ request }: { request: Request }) => auth.handler(request),
      POST: async ({ request }: { request: Request }) => auth.handler(request),
    },
  },
});
```

Requires `tanstackStartCookies()` plugin (must be last):

```typescript
import { tanstackStartCookies } from "better-auth/tanstack-start";

export const auth = betterAuth({
  plugins: [tanstackStartCookies()],
});
```

### SolidStart

```typescript
// routes/api/auth/*auth.ts
import { auth } from "~/lib/auth";
import { toSolidStartHandler } from "better-auth/solid-start";

export const { GET, POST } = toSolidStartHandler(auth);
```

### Nuxt

```typescript
// server/api/auth/[...all].ts
import { auth } from "~/lib/auth";

export default defineEventHandler((event) => {
  return auth.handler(toWebRequest(event));
});
```

### Cloudflare Workers

Requires `compatibility_flags = ["nodejs_compat"]` in `wrangler.toml`.

```typescript
export default {
  async fetch(request: Request) {
    const url = new URL(request.url);
    if (url.pathname.startsWith("/api/auth")) {
      return auth.handler(request);
    }
    return new Response("Not found", { status: 404 });
  },
};
```

## Client Setup

### Framework-Specific Imports

```typescript
import { createAuthClient } from "better-auth/client";   // Vanilla JS
import { createAuthClient } from "better-auth/react";     // React / Next.js
import { createAuthClient } from "better-auth/vue";       // Vue / Nuxt
import { createAuthClient } from "better-auth/svelte";    // Svelte / SvelteKit
import { createAuthClient } from "better-auth/solid";     // Solid / SolidStart
```

### Client Configuration

```typescript
export const authClient = createAuthClient({
  baseURL: "http://localhost:3000",   // only needed if server is on a different domain
  plugins: [],
});
```

### Core Client Methods

```typescript
// Sign up
const { data, error } = await authClient.signUp.email({
  email: "user@example.com",
  password: "password1234",
  name: "User Name",
  callbackURL: "/dashboard",
});

// Sign in (email)
const { data, error } = await authClient.signIn.email({
  email: "user@example.com",
  password: "password1234",
  rememberMe: true,
});

// Sign in (social — redirects to provider)
await authClient.signIn.social({
  provider: "google",
  callbackURL: "/dashboard",
  errorCallbackURL: "/error",
  newUserCallbackURL: "/welcome",
});

// Sign out
await authClient.signOut({
  fetchOptions: { onSuccess: () => router.push("/login") },
});

// Get session (one-shot)
const { data: session } = await authClient.getSession();

// Use session (reactive hook for React/Vue/Svelte/Solid)
const { data: session, isPending, error } = authClient.useSession();

// Update user
await authClient.updateUser({ name: "New Name" });
```

### Per-Request Callbacks

```typescript
await authClient.signIn.email({ email, password }, {
  onRequest: (ctx) => { /* show loading */ },
  onSuccess: (ctx) => { /* redirect */ },
  onError: (ctx) => { alert(ctx.error.message); },
});
```

### Server-Side API

All client endpoints are callable on the server via `auth.api`:

```typescript
// Get session (pass framework headers)
const session = await auth.api.getSession({ headers: await headers() });

// Sign in
const data = await auth.api.signInEmail({
  body: { email: "user@example.com", password: "password" },
  headers: await headers(),
});
```

Server-side calls are NOT subject to rate limiting.

## Database Configuration

### Drizzle Adapter

```typescript
import { drizzleAdapter } from "better-auth/adapters/drizzle";
import { db } from "./database";

export const auth = betterAuth({
  database: drizzleAdapter(db, {
    provider: "pg",  // "pg" | "mysql" | "sqlite"
  }),
});
```

Generate schema and migrate:

```bash
npx @better-auth/cli generate    # generates auth-schema.ts
npx drizzle-kit generate         # generates migration
npx drizzle-kit migrate          # applies migration
```

Custom table names:

```typescript
import * as schema from "./schema";

database: drizzleAdapter(db, {
  provider: "pg",
  schema: { ...schema, user: schema.users },  // map user -> users table
}),
```

### Prisma Adapter

```typescript
import { prismaAdapter } from "better-auth/adapters/prisma";
import { PrismaClient } from "@prisma/client";

const prisma = new PrismaClient();

export const auth = betterAuth({
  database: prismaAdapter(prisma, {
    provider: "postgresql",  // "postgresql" | "mysql" | "sqlite"
  }),
});
```

Generate schema and migrate:

```bash
npx @better-auth/cli generate          # adds models to schema.prisma
npx prisma migrate dev --name auth
npx prisma generate
```

### Direct Database Drivers

```typescript
// SQLite
import Database from "better-sqlite3";
database: new Database("./sqlite.db")

// PostgreSQL
import { Pool } from "pg";
database: new Pool({ connectionString: process.env.DATABASE_URL })

// MySQL
import { createPool } from "mysql2/promise";
database: createPool({ uri: process.env.DATABASE_URL })
```

### Core Schema (4 tables)

| Table | Key Fields |
|-------|-----------|
| **user** | id, name, email, emailVerified, image, createdAt, updatedAt |
| **session** | id, userId (FK), token, expiresAt, ipAddress, userAgent, createdAt, updatedAt |
| **account** | id, userId (FK), accountId, providerId, accessToken, refreshToken, password, createdAt, updatedAt |
| **verification** | id, identifier, value, expiresAt, createdAt, updatedAt |

### Additional User Fields

```typescript
user: {
  additionalFields: {
    role: {
      type: "string",
      required: false,
      defaultValue: "user",
      input: false,  // prevents user-provided values during signup
    },
    lang: {
      type: "string",
      required: false,
      defaultValue: "en",
    },
  },
},
```

Field `type` options: `"string"`, `"number"`, `"boolean"`, `"date"`, or string array for enums (e.g., `["user", "admin"]`).

### Secondary Storage (Redis)

```typescript
secondaryStorage: {
  get: async (key) => await redis.get(key),
  set: async (key, value, ttl) => {
    if (ttl) await redis.set(key, value, { EX: ttl });
    else await redis.set(key, value);
  },
  delete: async (key) => await redis.del(key),
},
```

### Custom Table/Column Names

```typescript
user: {
  modelName: "users",
  fields: { name: "full_name", email: "email_address" },
},
session: {
  modelName: "user_sessions",
  fields: { userId: "user_id" },
},
```

## Email and Password Authentication

### Server Configuration

```typescript
emailAndPassword: {
  enabled: true,
  minPasswordLength: 8,
  maxPasswordLength: 128,
  autoSignIn: true,           // auto sign in after sign up
  requireEmailVerification: false,
  sendResetPassword: async ({ user, url, token }, request) => {
    // Don't await — prevents timing attacks
    void sendEmail({
      to: user.email,
      subject: "Reset your password",
      text: `Click to reset: ${url}`,
    });
  },
},
```

### Email Verification

```typescript
emailVerification: {
  sendVerificationEmail: async ({ user, url, token }, request) => {
    void sendEmail({
      to: user.email,
      subject: "Verify your email",
      text: `Click to verify: ${url}`,
    });
  },
  sendOnSignUp: true,
  autoSignInAfterVerification: true,
  expiresIn: 3600,
},
```

### Password Reset Flow

```typescript
// Client: request reset
await authClient.requestPasswordReset({
  email: "user@example.com",
  redirectTo: "/reset-password",
});

// Client: reset password (from reset page with token)
await authClient.resetPassword({
  newPassword: "newPassword1234",
  token: tokenFromUrl,
});

// Client: change password (authenticated)
await authClient.changePassword({
  currentPassword: "old1234",
  newPassword: "new1234",
  revokeOtherSessions: true,
});
```

### Custom Password Hashing (Argon2)

```typescript
import { hash, verify } from "@node-rs/argon2";

emailAndPassword: {
  enabled: true,
  password: {
    hash: (password) => hash(password, {
      memoryCost: 65536, timeCost: 3, parallelism: 4, outputLen: 32,
    }),
    verify: ({ password, hash: h }) => verify(h, password),
  },
},
```

## OAuth / Social Authentication

### Provider Configuration

```typescript
socialProviders: {
  google: {
    clientId: process.env.GOOGLE_CLIENT_ID as string,
    clientSecret: process.env.GOOGLE_CLIENT_SECRET as string,
    prompt: "select_account",
    accessType: "offline",  // for refresh tokens
  },
  github: {
    clientId: process.env.GITHUB_CLIENT_ID as string,
    clientSecret: process.env.GITHUB_CLIENT_SECRET as string,
  },
  discord: {
    clientId: process.env.DISCORD_CLIENT_ID as string,
    clientSecret: process.env.DISCORD_CLIENT_SECRET as string,
  },
},
```

Callback URL pattern: `{baseURL}/api/auth/callback/{provider}`

35+ built-in providers: Apple, Discord, Facebook, GitHub, GitLab, Google, LinkedIn, Microsoft, Slack, Spotify, TikTok, Twitch, Twitter, and more.

### Per-Provider Options

```typescript
google: {
  clientId: "...",
  clientSecret: "...",
  scope: ["https://www.googleapis.com/auth/drive.file"],
  mapProfileToUser: (profile) => ({
    name: profile.name,
    image: profile.picture,
  }),
  disableSignUp: false,
  overrideUserInfoOnSignIn: false,
},
```

### Client-Side Social Sign In

```typescript
// Redirect-based (default)
await authClient.signIn.social({
  provider: "google",
  callbackURL: "/dashboard",
});

// ID Token-based (no redirect — for mobile/native)
await authClient.signIn.social({
  provider: "google",
  idToken: { token: googleIdToken, accessToken: googleAccessToken },
});
```

### Generic OAuth (Custom Providers)

```typescript
import { genericOAuth } from "better-auth/plugins";

plugins: [
  genericOAuth({
    config: [{
      providerId: "keycloak",
      clientId: "...",
      clientSecret: "...",
      discoveryUrl: "https://auth.example.com/.well-known/openid-configuration",
      scopes: ["openid", "profile", "email"],
    }],
  }),
],
```

### Account Linking

```typescript
account: {
  accountLinking: {
    enabled: true,
    trustedProviders: ["google", "github"],
    allowDifferentEmails: false,
  },
},
```

## Session Management

### Session Configuration

```typescript
session: {
  expiresIn: 60 * 60 * 24 * 7,     // 7 days (seconds)
  updateAge: 60 * 60 * 24,           // refresh after 1 day
  freshAge: 60 * 60 * 24,            // fresh for 1 day (sensitive ops require fresh)
  disableSessionRefresh: false,
},
```

### Cookie Cache (Performance)

```typescript
session: {
  cookieCache: {
    enabled: true,
    maxAge: 5 * 60,          // 5 minutes
    strategy: "jwt",          // "compact" | "jwt" | "jwe"
    refreshCache: true,       // auto-refresh on expiry
  },
},
```

| Strategy | Size | Security |
|----------|------|----------|
| `compact` | Smallest | Readable, HMAC-SHA256 signed |
| `jwt` | Medium | Readable, HS256 JWT, interoperable |
| `jwe` | Largest | Fully encrypted AES-256-GCM |

### Session Invalidation

Change `cookieCache.version` and redeploy to invalidate all sessions.

### Client Session Methods

```typescript
const sessions = await authClient.listSessions();
await authClient.revokeSession({ token: "session-token" });
await authClient.revokeOtherSessions();
await authClient.revokeSessions();  // revoke all
```

## Middleware Patterns

### Next.js Middleware (Cookie Check)

```typescript
// middleware.ts
import { NextRequest, NextResponse } from "next/server";
import { getSessionCookie } from "better-auth/cookies";

export async function middleware(request: NextRequest) {
  const sessionCookie = getSessionCookie(request);
  if (!sessionCookie) {
    return NextResponse.redirect(new URL("/login", request.url));
  }
  return NextResponse.next();
}

export const config = { matcher: ["/dashboard/:path*"] };
```

For full validation (not just cookie presence):

```typescript
import { auth } from "@/lib/auth";
import { headers } from "next/headers";

// In RSC or Server Action:
const session = await auth.api.getSession({ headers: await headers() });
if (!session) redirect("/login");
```

### Next.js Cookie Cache Middleware

```typescript
import { getCookieCache } from "better-auth/cookies";

export async function middleware(request: NextRequest) {
  const session = await getCookieCache(request);
  if (!session) {
    return NextResponse.redirect(new URL("/login", request.url));
  }
  return NextResponse.next();
}
```

### Astro Middleware

```typescript
// middleware.ts
import { auth } from "@/auth";
import { defineMiddleware } from "astro:middleware";

export const onRequest = defineMiddleware(async (context, next) => {
  const isAuthed = await auth.api.getSession({
    headers: context.request.headers,
  });
  context.locals.user = isAuthed?.user || null;
  context.locals.session = isAuthed?.session || null;
  return next();
});
```

### Hono Middleware (Type-Safe Context)

```typescript
const app = new Hono<{
  Variables: {
    user: typeof auth.$Infer.Session.user | null;
    session: typeof auth.$Infer.Session.session | null;
  };
}>();

app.use("*", async (c, next) => {
  const session = await auth.api.getSession({
    headers: c.req.raw.headers,
  });
  c.set("user", session?.user ?? null);
  c.set("session", session?.session ?? null);
  await next();
});

// In routes:
app.get("/api/me", (c) => {
  const user = c.get("user");
  if (!user) return c.body(null, 401);
  return c.json(user);
});
```

### Express Middleware

```typescript
import { fromNodeHeaders } from "better-auth/node";

app.get("/api/me", async (req, res) => {
  const session = await auth.api.getSession({
    headers: fromNodeHeaders(req.headers),
  });
  if (!session) return res.status(401).json({ error: "Unauthorized" });
  return res.json(session);
});
```

### TanStack Start Middleware

```typescript
// src/middleware/auth.ts
import { redirect } from "@tanstack/react-router";
import { createMiddleware } from "@tanstack/react-start";
import { getRequestHeaders } from "@tanstack/react-start/server";
import { auth } from "@/lib/auth";

export const authMiddleware = createMiddleware().server(
  async ({ next }) => {
    const headers = getRequestHeaders();
    const session = await auth.api.getSession({ headers });
    if (!session) throw redirect({ to: "/login" });
    return await next();
  },
);
```

### Nuxt Route Middleware

```typescript
// middleware/auth.global.ts
import { authClient } from "~/lib/auth-client";

export default defineNuxtRouteMiddleware(async (to) => {
  const { data: session } = await authClient.useSession(useFetch);
  if (!session.value && to.path === "/dashboard") {
    return navigateTo("/login");
  }
});
```

## Two-Factor Authentication Plugin

### Setup

```typescript
// Server
import { twoFactor } from "better-auth/plugins";

plugins: [
  twoFactor({
    issuer: "My App",   // shown in authenticator apps
    otpOptions: {
      async sendOTP({ user, otp }) {
        await sendEmail(user.email, `Your code: ${otp}`);
      },
    },
  }),
]

// Client
import { twoFactorClient } from "better-auth/client/plugins";

plugins: [
  twoFactorClient({
    onTwoFactorRedirect: () => { window.location.href = "/2fa"; },
  }),
]
```

### Enable/Disable

```typescript
await authClient.twoFactor.enable({ password: "user-password" });
await authClient.twoFactor.disable({ password: "user-password" });
```

### Sign In Flow

```typescript
await authClient.signIn.email({ email, password }, {
  onSuccess: (ctx) => {
    if (ctx.data.twoFactorRedirect) {
      router.push("/2fa");  // redirect to 2FA verification page
    }
  },
});
```

### Verify TOTP

```typescript
await authClient.twoFactor.verifyTotp({
  code: "123456",
  trustDevice: true,  // skip 2FA for 30 days on this device
});
```

### Backup Codes

```typescript
const { data } = await authClient.twoFactor.generateBackupCodes({ password });
// data.backupCodes — display to user for safekeeping

await authClient.twoFactor.verifyBackupCode({ code: "abc123" });
```

### Database Schema

Adds `twoFactorEnabled` (boolean) to user table. Creates `twoFactor` table with `id`, `userId`, `secret`, `backupCodes`.

## Admin Plugin

### Setup

```typescript
// Server
import { admin } from "better-auth/plugins";

plugins: [
  admin({
    defaultRole: "user",
    adminRoles: ["admin"],
    impersonationSessionDuration: 3600,
  }),
]

// Client
import { adminClient } from "better-auth/client/plugins";
plugins: [adminClient()]
```

### Admin Operations

```typescript
// Create user
await authClient.admin.createUser({
  email: "new@example.com",
  password: "password",
  name: "New User",
  role: "admin",
});

// List users (with filtering)
const { data } = await authClient.admin.listUsers({
  searchValue: "john",
  searchField: "email",
  searchOperator: "contains",
  limit: 50,
  sortBy: "createdAt",
  sortDirection: "desc",
});

// Ban/unban
await authClient.admin.banUser({ userId: "...", banReason: "Spam" });
await authClient.admin.unbanUser({ userId: "..." });

// Impersonate
await authClient.admin.impersonateUser({ userId: "..." });
await authClient.admin.stopImpersonating();

// Set role
await authClient.admin.setRole({ userId: "...", role: "admin" });
```

### Custom Access Control

```typescript
import { createAccessControl } from "better-auth/plugins/access";
import { defaultStatements, adminAc } from "better-auth/plugins/admin/access";

const statement = {
  ...defaultStatements,
  project: ["create", "update", "delete"],
} as const;

const ac = createAccessControl(statement);

const adminRole = ac.newRole({
  project: ["create", "update"],
  ...adminAc.statements,
});

// Server
plugins: [admin({ ac, roles: { admin: adminRole } })]

// Check permission
const can = await authClient.admin.hasPermission({
  permission: { project: ["create"] },
});
```

### Database Schema

Adds to user table: `role` (string), `banned` (boolean), `banReason` (string), `banExpires` (date).

## Organization Plugin

### Setup

```typescript
// Server
import { organization } from "better-auth/plugins";

plugins: [
  organization({
    allowUserToCreateOrganization: true,
    organizationLimit: 5,
    creatorRole: "owner",
    membershipLimit: 100,
    sendInvitationEmail: async ({ email, organization, inviter, url }) => {
      await sendEmail(email, `Join ${organization.name}: ${url}`);
    },
  }),
]

// Client
import { organizationClient } from "better-auth/client/plugins";
plugins: [organizationClient()]
```

### Organization CRUD

```typescript
await authClient.organization.create({ name: "Acme Inc", slug: "acme" });
await authClient.organization.update({ data: { name: "Acme Corp" } });
await authClient.organization.delete({ organizationId: "..." });
await authClient.organization.setActive({ organizationSlug: "acme" });

const { data } = authClient.useActiveOrganization();
const orgs = await authClient.organization.list({});
```

### Member Management

```typescript
await authClient.organization.inviteMember({
  email: "member@example.com",
  role: "member",
});
await authClient.organization.acceptInvitation({ invitationId: "..." });
await authClient.organization.removeMember({ memberIdOrEmail: "member@example.com" });
await authClient.organization.updateMemberRole({
  memberId: "...",
  role: "admin",
});
```

Default roles: `owner` (full control), `admin` (no delete org/change owner), `member` (read-only).

### Teams (Sub-groups)

```typescript
// Enable in config
organization({ teams: { enabled: true, maximumTeams: 10 } })

await authClient.organization.createTeam({ name: "Engineering" });
await authClient.organization.addTeamMember({ teamId: "...", userId: "..." });
```

### Database Schema

Creates tables: `organization`, `member`, `invitation`, optionally `team` and `teamMember`.

## Magic Link Plugin

### Setup

```typescript
// Server
import { magicLink } from "better-auth/plugins";

plugins: [
  magicLink({
    sendMagicLink: async ({ email, url, token }) => {
      await sendEmail(email, `Sign in: ${url}`);
    },
    expiresIn: 300,  // 5 minutes
  }),
]

// Client
import { magicLinkClient } from "better-auth/client/plugins";
plugins: [magicLinkClient()]
```

### Usage

```typescript
await authClient.signIn.magicLink({
  email: "user@example.com",
  callbackURL: "/dashboard",
});

// Verify (on callback page)
await authClient.magicLink.verify({ token: tokenFromUrl });
```

## Passkey Plugin

### Setup

```bash
npm install @better-auth/passkey
```

```typescript
// Server
import { passkey } from "@better-auth/passkey";

plugins: [
  passkey({
    rpID: "example.com",
    rpName: "My App",
    origin: "https://example.com",
  }),
]

// Client
import { passkeyClient } from "@better-auth/passkey/client";
plugins: [passkeyClient()]
```

### Usage

```typescript
// Register passkey (must be authenticated)
await authClient.passkey.addPasskey({ name: "My Passkey" });

// Sign in with passkey
await authClient.signIn.passkey();

// Conditional UI (autofill)
await authClient.signIn.passkey({ autoFill: true });

// List and delete
const passkeys = await authClient.passkey.listUserPasskeys({});
await authClient.passkey.deletePasskey({ id: "..." });
```

## API Key Plugin

### Setup

```typescript
// Server
import { apiKey } from "better-auth/plugins";

plugins: [
  apiKey({
    defaultPrefix: "sk_",
    defaultKeyLength: 64,
    enableMetadata: true,
    rateLimit: {
      enabled: true,
      timeWindow: 1000 * 60 * 60 * 24,
      maxRequests: 1000,
    },
  }),
]

// Client
import { apiKeyClient } from "better-auth/client/plugins";
plugins: [apiKeyClient()]
```

### Usage

```typescript
const { data } = await authClient.apiKey.create({
  name: "Production Key",
  expiresIn: 86400 * 30,  // 30 days
  prefix: "sk_live_",
});
// data.key — show ONCE, then it's hashed

const keys = await authClient.apiKey.list({});
await authClient.apiKey.delete({ keyId: "..." });
```

### Verify in API routes

```typescript
// Keys are sent in x-api-key header by default
const session = await auth.api.getSession({ headers: req.headers });
```

## Bearer Token Plugin

```typescript
// Server
import { bearer } from "better-auth/plugins";
plugins: [bearer()]

// Client: capture token after sign-in
const authClient = createAuthClient({
  fetchOptions: {
    onSuccess: (ctx) => {
      const token = ctx.response.headers.get("set-auth-token");
      if (token) localStorage.setItem("bearer_token", token);
    },
    auth: {
      type: "Bearer",
      token: () => localStorage.getItem("bearer_token") || "",
    },
  },
});
```

## JWT Plugin

```typescript
// Server
import { jwt } from "better-auth/plugins";

plugins: [
  jwt({
    jwt: {
      issuer: "https://example.com",
      audience: "https://example.com",
      expirationTime: "1h",
      definePayload: ({ user }) => ({
        id: user.id,
        email: user.email,
        role: user.role,
      }),
    },
  }),
]

// Client
import { jwtClient } from "better-auth/client/plugins";
plugins: [jwtClient()]

// Get JWT token
const { data } = await authClient.token();
```

JWKS endpoint exposed at `/api/auth/jwks` for token verification:

```typescript
import { jwtVerify, createRemoteJWKSet } from "jose";

const JWKS = createRemoteJWKSet(new URL("https://example.com/api/auth/jwks"));
const { payload } = await jwtVerify(token, JWKS);
```

## Email OTP Plugin

```typescript
// Server
import { emailOTP } from "better-auth/plugins";

plugins: [
  emailOTP({
    async sendVerificationOTP({ email, otp, type }) {
      await sendEmail(email, `Your code: ${otp}`);
    },
    otpLength: 6,
    expiresIn: 300,
  }),
]

// Client
import { emailOTPClient } from "better-auth/client/plugins";
plugins: [emailOTPClient()]

// Sign in flow
await authClient.emailOtp.sendVerificationOtp({ email, type: "sign-in" });
await authClient.signIn.emailOtp({ email, otp: "123456" });
```

## Username Plugin

```typescript
// Server
import { username } from "better-auth/plugins";

plugins: [
  username({
    minUsernameLength: 3,
    maxUsernameLength: 30,
  }),
]

// Client
import { usernameClient } from "better-auth/client/plugins";
plugins: [usernameClient()]

// Sign in by username
await authClient.signIn.username({ username: "john", password: "..." });

// Check availability
await authClient.isUsernameAvailable({ username: "john" });
```

## Phone Number Plugin

```typescript
// Server
import { phoneNumber } from "better-auth/plugins";

plugins: [
  phoneNumber({
    sendOTP: async ({ phoneNumber, code }) => {
      await twilioClient.messages.create({
        body: `Your code: ${code}`,
        to: phoneNumber,
        from: "+1234567890",
      });
    },
  }),
]

// Client
import { phoneNumberClient } from "better-auth/client/plugins";
plugins: [phoneNumberClient()]

await authClient.phoneNumber.sendOtp({ phoneNumber: "+1234567890" });
await authClient.phoneNumber.verify({ phoneNumber: "+1234567890", code: "123456" });
```

## Hooks and Lifecycle

### Server Hooks (Before/After)

```typescript
import { createAuthMiddleware, APIError } from "better-auth/api";

hooks: {
  before: createAuthMiddleware(async (ctx) => {
    // Access: ctx.path, ctx.body, ctx.query, ctx.headers
    if (ctx.path === "/sign-up/email" && blockedDomains.has(getDomain(ctx.body.email))) {
      throw new APIError("FORBIDDEN", { message: "Domain blocked" });
    }
  }),
  after: createAuthMiddleware(async (ctx) => {
    // ctx.context.newSession available after sign-up
    // ctx.context.returned for previous return value
  }),
},
```

### Database Hooks

```typescript
databaseHooks: {
  user: {
    create: {
      before: async (user) => ({
        data: { ...user, role: "user" },
      }),
      after: async (user) => {
        await analytics.track("user_created", { userId: user.id });
      },
    },
    delete: {
      before: async (user) => {
        if (user.role === "admin") return false;  // prevent deletion
        return true;
      },
    },
  },
},
```

## Rate Limiting

```typescript
rateLimit: {
  enabled: true,
  window: 60,
  max: 100,
  storage: "secondary-storage",  // use Redis for multi-instance
  customRules: {
    "/sign-in/email": { window: 10, max: 3 },
    "/two-factor/*": { window: 10, max: 3 },
    "/get-session": false,  // disable for session checks
  },
},
```

Built-in stricter limits: `/sign-in/email` (3/10s), `/two-factor/verify` (3/10s).

Client handling:

```typescript
const authClient = createAuthClient({
  fetchOptions: {
    onError: (ctx) => {
      if (ctx.response.status === 429) {
        const retryAfter = ctx.response.headers.get("X-Retry-After");
      }
    },
  },
});
```

## TypeScript Integration

### Type Inference

```typescript
// Server-side session type
type Session = typeof auth.$Infer.Session;  // { session, user }

// Client-side
type Session = typeof authClient.$Infer.Session;
```

### Client-Side Additional Field Inference

```typescript
// Same project (monorepo):
import { inferAdditionalFields } from "better-auth/client/plugins";
import type { auth } from "./auth";

const authClient = createAuthClient({
  plugins: [inferAdditionalFields<typeof auth>()],
});

// Separate projects:
const authClient = createAuthClient({
  plugins: [
    inferAdditionalFields({
      user: { role: { type: "string" } },
    }),
  ],
});
```

### Error Codes

```typescript
const errorCodes = authClient.$ERROR_CODES;  // all possible error codes
```

### TSConfig Requirements

```json
{ "compilerOptions": { "strict": true } }
```

Do NOT enable both `declaration` and `composite` simultaneously.

## Production Security Hardening

### Required Configuration

```typescript
export const auth = betterAuth({
  secret: process.env.BETTER_AUTH_SECRET,
  baseURL: process.env.BETTER_AUTH_URL,
  trustedOrigins: ["https://example.com"],
  advanced: {
    useSecureCookies: true,
    defaultCookieAttributes: { httpOnly: true, secure: true },
    ipAddress: { ipAddressHeaders: ["cf-connecting-ip"] },
  },
  rateLimit: {
    enabled: true,
    storage: "secondary-storage",  // never use "memory" in production multi-instance
  },
  account: {
    encryptOAuthTokens: true,
  },
  session: {
    cookieCache: { enabled: true, strategy: "jwe" },  // encrypted cookie cache
  },
});
```

### Serverless Background Tasks

```typescript
import { waitUntil } from "@vercel/functions";

advanced: {
  backgroundTasks: { handler: waitUntil },
},
```

### Cross-Subdomain Cookies

```typescript
advanced: {
  crossSubDomainCookies: {
    enabled: true,
    domain: ".example.com",
  },
},
```

### Production Checklist

1. Set `BETTER_AUTH_SECRET` with high entropy (min 32 chars)
2. Set `BETTER_AUTH_URL` explicitly (never infer from request)
3. Configure `trustedOrigins` for all valid domains
4. Enable rate limiting with Redis/database storage
5. Never disable CSRF or origin checks
6. Enable `encryptOAuthTokens` for OAuth token storage
7. Use `"jwe"` cookie cache for maximum security
8. Configure IP headers for your CDN/proxy
9. Avoid awaiting email sends (timing attacks) — use `void` or `waitUntil`
10. Use `sessionCookie` check in middleware, full `getSession` for protected operations

## CLI Reference

```bash
npx @better-auth/cli init                      # initialize in project
npx @better-auth/cli generate --output ./db     # generate ORM schema
npx @better-auth/cli migrate                    # run migrations (Kysely only)
npx @better-auth/cli secret                     # generate a secret
npx @better-auth/cli info                       # diagnostic info
```

## Best Practices

- Always set `BETTER_AUTH_SECRET` and `BETTER_AUTH_URL` via environment variables, never hardcode
- Use `void sendEmail()` in email callbacks to prevent timing attacks that reveal user existence
- On serverless platforms, use `waitUntil` for background tasks like email delivery
- Place `express.json()` middleware AFTER the Better Auth handler to avoid request body conflicts
- Cookie-only middleware checks are fast but insufficient for sensitive operations; always call `auth.api.getSession()` for protected data
- Use `"secondary-storage"` (Redis) for rate limiting and sessions in multi-instance production deployments
- Enable `encryptOAuthTokens` when storing OAuth tokens to protect against database breaches
- The `nextCookies` and `tanstackStartCookies` plugins must always be the LAST plugin in the array
- Run `npx @better-auth/cli generate` after adding plugins to update your database schema
- Use `input: false` on `additionalFields` for server-only fields like `role` to prevent user manipulation during signup
- Configure `accountLinking.trustedProviders` to auto-link accounts only from verified OAuth providers
- Set `session.freshAge` to control how recently a user must have authenticated for sensitive operations
