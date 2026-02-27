---
name: better-auth-specialist
description: Expert implementation of user authentication and authorization using Better Auth library for Next.js 15+/React 18+ frontends and Node.js/FastAPI backends with SQL and NoSQL databases. Use when implementing authentication systems, user login/signup, session management, protected routes, role-based access control (RBAC), OAuth integration, or any auth-related tasks including email/password authentication, JWT tokens, permissions, and user management.
---

# Better Auth Specialist

Implement secure, production-ready authentication and authorization with Better Auth.

## Quick Start Workflow

1. **Determine requirements** - Authentication type, database, framework
2. **Install Better Auth** - `npm install better-auth` or `pip install better-auth-python`
3. **Setup database** - Create tables/collections (see [database-schemas.md](references/database-schemas.md))
4. **Configure auth** - Initialize Better Auth with settings (see [setup-guide.md](references/setup-guide.md))
5. **Implement patterns** - Add authentication flows (see references below)
6. **Protect routes** - Add middleware and guards
7. **Add RBAC** - Implement roles and permissions if needed (see [rbac-guide.md](references/rbac-guide.md))

## Decision Tree

### What are you building?

**Next.js 15+ Frontend**
- Email/password auth → See [client-patterns.md](references/client-patterns.md) - Signup/Signin Forms
- OAuth (Google, GitHub) → See [client-patterns.md](references/client-patterns.md) - OAuth Integration
- Protected pages → See [client-patterns.md](references/client-patterns.md) - Protected Routes
- Complete example → See [assets/examples/nextjs-auth/](assets/examples/nextjs-auth/)

**Node.js/Express Backend**
- API authentication → See [api-patterns.md](references/api-patterns.md) - Authentication Endpoints
- Middleware → See [api-patterns.md](references/api-patterns.md) - Middleware Patterns
- Session management → See [api-patterns.md](references/api-patterns.md) - Session Management

**FastAPI Backend**
- API authentication → See [api-patterns.md](references/api-patterns.md) - FastAPI sections
- Dependencies → See [api-patterns.md](references/api-patterns.md) - FastAPI Dependencies
- Protected routes → See [api-patterns.md](references/api-patterns.md) - FastAPI examples

**Role-Based Access Control**
- Roles and permissions → See [rbac-guide.md](references/rbac-guide.md)
- Middleware → See [rbac-guide.md](references/rbac-guide.md) - Middleware Implementation
- Frontend guards → See [rbac-guide.md](references/rbac-guide.md) - Frontend Implementation

## Installation

### Next.js / React

```bash
npm install better-auth
```

### Node.js Backend

```bash
npm install better-auth
```

### FastAPI Backend

```bash
pip install better-auth-python python-jose[cryptography] passlib[bcrypt]
```

## Basic Setup Patterns

### Pattern 1: Next.js Email/Password Auth

```typescript
// lib/auth.ts
import { betterAuth } from "better-auth";
import { prismaAdapter } from "better-auth/adapters/prisma";

export const auth = betterAuth({
  database: prismaAdapter(prisma, { provider: "postgresql" }),
  emailAndPassword: { enabled: true },
});

// lib/auth-client.ts
import { createAuthClient } from "better-auth/react";

export const { signIn, signUp, signOut, useSession } = createAuthClient({
  baseURL: process.env.NEXT_PUBLIC_APP_URL,
});
```

### Pattern 2: Node.js API with Auth Middleware

```typescript
// auth.ts
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  database: { provider: "postgresql", pool },
  emailAndPassword: { enabled: true },
});

// middleware/auth.ts
export async function requireAuth(req, res, next) {
  const session = await auth.api.getSession({ headers: req.headers });
  if (!session) return res.status(401).json({ error: "Unauthorized" });
  req.user = session.user;
  next();
}

// Usage
app.get("/api/protected", requireAuth, (req, res) => {
  res.json({ user: req.user });
});
```

### Pattern 3: FastAPI with Auth Dependency

```python
# auth.py
from better_auth import BetterAuth

auth = BetterAuth(
    database=SQLAlchemyAdapter(engine),
    email_and_password={"enabled": True},
)

# dependencies.py
async def get_current_user(request: Request):
    session = await auth.get_session(headers=dict(request.headers))
    if not session:
        raise HTTPException(status_code=401)
    return session["user"]

# Usage
@app.get("/api/protected")
async def protected(user: dict = Depends(get_current_user)):
    return {"user": user}
```

### Pattern 4: Protected Next.js Page

```typescript
// app/dashboard/page.tsx
import { redirect } from "next/navigation";
import { auth } from "@/lib/auth";
import { headers } from "next/headers";

export default async function DashboardPage() {
  const session = await auth.api.getSession({ headers: await headers() });
  if (!session) redirect("/auth");

  return <div>Welcome, {session.user.name}!</div>;
}
```

## Database Setup

Choose your database and see schemas:

**PostgreSQL/MySQL** → [database-schemas.md](references/database-schemas.md) - SQL Schemas
**MongoDB** → [database-schemas.md](references/database-schemas.md) - NoSQL Schemas

Key tables/collections:
- `users` - User accounts
- `sessions` - Active sessions
- `accounts` - OAuth provider accounts
- `passwords` - Hashed passwords (if using email/password)
- `roles` / `permissions` / `user_roles` - RBAC (optional)

## Common Use Cases

### Implementing Login/Signup

**Frontend:**
1. Create auth page with forms (see [client-patterns.md](references/client-patterns.md) - Authentication Forms)
2. Use `signIn.email()` and `signUp.email()` methods
3. Handle errors and redirects
4. Complete example: [assets/examples/nextjs-auth/auth-page.tsx](assets/examples/nextjs-auth/auth-page.tsx)

**Backend:**
1. Create `/auth/signup` and `/auth/signin` endpoints (see [api-patterns.md](references/api-patterns.md))
2. Validate input
3. Return session tokens
4. Handle errors appropriately

### Protecting Routes/Endpoints

**Next.js Middleware:**
```typescript
// middleware.ts
import { auth } from "@/lib/auth";

export async function middleware(request: NextRequest) {
  const session = await auth.api.getSession({ headers: request.headers });
  if (!session) return NextResponse.redirect(new URL("/auth", request.url));
  return NextResponse.next();
}

export const config = { matcher: ["/dashboard/:path*"] };
```

**Express Middleware:**
```typescript
export async function requireAuth(req, res, next) {
  const session = await auth.api.getSession({ headers: req.headers });
  if (!session) return res.status(401).json({ error: "Unauthorized" });
  req.user = session.user;
  next();
}
```

**FastAPI Dependency:**
```python
async def get_current_user(request: Request):
    session = await auth.get_session(headers=dict(request.headers))
    if not session:
        raise HTTPException(status_code=401)
    return session["user"]
```

### Session Management

**Get current session:**
```typescript
// Client-side
const { data: session } = useSession();

// Server-side
const session = await auth.api.getSession({ headers });
```

**Sign out:**
```typescript
// Client-side
await signOut();

// Server-side
await auth.api.signOut({ headers: req.headers });
```

**Refresh session:**
```typescript
await auth.api.refreshSession({ headers });
```

### OAuth Integration

```typescript
// Setup in auth config
export const auth = betterAuth({
  socialProviders: {
    google: {
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
    },
    github: {
      clientId: process.env.GITHUB_CLIENT_ID!,
      clientSecret: process.env.GITHUB_CLIENT_SECRET!,
    },
  },
});

// Client-side usage
await signIn.social({ provider: "google", callbackURL: "/dashboard" });
```

See [client-patterns.md](references/client-patterns.md) - OAuth Integration for complete implementation.

### Role-Based Access Control

**Setup roles and permissions:**
1. Create database tables (see [database-schemas.md](references/database-schemas.md) - RBAC schemas)
2. Assign roles to users
3. Implement permission checks

**Backend middleware:**
```typescript
export function requireRole(...roles: string[]) {
  return async (req, res, next) => {
    const userRoles = await getUserRoles(req.user.id);
    if (!userRoles.some(r => roles.includes(r.name))) {
      return res.status(403).json({ error: "Insufficient permissions" });
    }
    next();
  };
}

// Usage
router.delete("/users/:id", requireAuth, requireRole("admin"), handler);
```

**Frontend role guard:**
```typescript
<RoleGuard roles={["admin", "moderator"]}>
  <AdminPanel />
</RoleGuard>
```

See [rbac-guide.md](references/rbac-guide.md) for complete RBAC implementation.

## Environment Variables

```env
# Database
DATABASE_URL="postgresql://user:pass@localhost:5432/db"

# Auth
BETTER_AUTH_SECRET="your-secret-key-min-32-chars"
BETTER_AUTH_URL="http://localhost:3000"

# OAuth (optional)
GOOGLE_CLIENT_ID="your-google-client-id"
GOOGLE_CLIENT_SECRET="your-google-client-secret"
GITHUB_CLIENT_ID="your-github-client-id"
GITHUB_CLIENT_SECRET="your-github-client-secret"
```

## Reference Files

- **[setup-guide.md](references/setup-guide.md)** - Complete installation and configuration for Next.js, Node.js, and FastAPI
- **[database-schemas.md](references/database-schemas.md)** - SQL and NoSQL database schemas, migration scripts, TypeScript types
- **[api-patterns.md](references/api-patterns.md)** - Backend authentication endpoints, middleware, session management, security
- **[client-patterns.md](references/client-patterns.md)** - Frontend forms, protected routes, session hooks, OAuth, role-based UI
- **[rbac-guide.md](references/rbac-guide.md)** - Complete role-based access control implementation and best practices

## Example Files

- **[nextjs-auth/auth-page.tsx](assets/examples/nextjs-auth/auth-page.tsx)** - Complete authentication page with signup/signin forms and OAuth
- **[nextjs-auth/protected-dashboard.tsx](assets/examples/nextjs-auth/protected-dashboard.tsx)** - Protected dashboard page example

## Security Best Practices

1. **Always use HTTPS in production**
2. **Validate and sanitize user input**
3. **Use strong passwords** - Min 8 chars, uppercase, lowercase, numbers, special chars
4. **Implement rate limiting** - Prevent brute force attacks (see [api-patterns.md](references/api-patterns.md) - Rate Limiting)
5. **Use secure session settings** - HttpOnly cookies, SameSite, Secure flag
6. **Enable CSRF protection** - For state-changing operations
7. **Store secrets securely** - Use environment variables, never commit to git
8. **Implement email verification** - For production applications
9. **Add 2FA support** - For sensitive applications
10. **Audit and monitor** - Log authentication events, monitor suspicious activity

## Common Issues and Solutions

**"Unauthorized" errors:**
- Check session token is being sent correctly
- Verify database connection
- Check BETTER_AUTH_SECRET is set
- Ensure session hasn't expired

**CORS errors:**
- Configure CORS to allow credentials
- Set correct origin in CORS settings
- Ensure cookies are sent with credentials: true

**Session not persisting:**
- Check cookie settings (secure, sameSite, domain)
- Verify BETTER_AUTH_URL matches your app URL
- Check browser cookie settings

**OAuth not working:**
- Verify client IDs and secrets
- Check redirect URLs in provider dashboard
- Ensure callback URL is correctly configured

See [setup-guide.md](references/setup-guide.md) - Troubleshooting for more solutions.

## General Guidelines

1. **Start with email/password** - Simplest to implement and test
2. **Add OAuth later** - Once basic auth is working
3. **Implement RBAC only if needed** - Not all apps need complex permissions
4. **Use middleware/dependencies** - Centralize auth logic
5. **Handle errors gracefully** - Provide clear error messages to users
6. **Test authentication flows** - Test signup, signin, signout, protected routes
7. **Secure sessions** - Use appropriate expiry times and refresh tokens
8. **Follow the principle of least privilege** - Grant minimum necessary permissions

## Example Workflow

User request: "Implement user authentication with email/password and protect the dashboard"

1. **Setup database** (see database-schemas.md)
   - Create users, sessions, passwords tables
   - Run migrations

2. **Install Better Auth**
   ```bash
   npm install better-auth
   ```

3. **Configure auth** (see setup-guide.md - Next.js Setup)
   - Create auth configuration in lib/auth.ts
   - Setup auth API route
   - Create auth client

4. **Create auth page** (see assets/examples/nextjs-auth/auth-page.tsx)
   - Implement signup form
   - Implement signin form
   - Handle errors and redirects

5. **Protect dashboard** (see client-patterns.md - Protected Routes)
   - Add server-side session check
   - Redirect to /auth if not authenticated
   - Display user information

6. **Add middleware** (see client-patterns.md - Protected Route Middleware)
   - Create middleware.ts
   - Protect /dashboard routes
   - Redirect unauthenticated users

7. **Test the flow**
   - Sign up new user
   - Sign in
   - Access dashboard
   - Sign out
   - Verify protection works

Result: Fully functional authentication system with protected dashboard.
