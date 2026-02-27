---
name: authentication
description: Auth flows, session management, OAuth integration, domain-restricted access, and role-based access control for TopNetworks properties. Primary implementation is Better Auth 1.x with Google OAuth in route-genius. Use when implementing login, session checks, protected routes, or any access control logic.
---

# Authentication — TopNetworks, Inc.

This skill governs all authentication and authorization work. Derived from route-genius (Better Auth 1.x + Google OAuth + Supabase sessions) as the canonical full-stack auth implementation in the workspace.

---

## Scope

**Use for:** Login flows, session management, protected route middleware, OAuth integration, domain-restricted access, Google Drive OAuth (secondary), and access control patterns.

**Not for:** Database RLS configuration (see `database` skill), general API route design (see `backend` skill), or AdZep ad network integration (that is a separate concern — see project instruction files).

---

## Auth Stack by Project

| Project                              | Auth Method       | Provider                   | Session Storage       |
| ------------------------------------ | ----------------- | -------------------------- | --------------------- |
| route-genius                         | Better Auth 1.x   | Google OAuth 2.0           | PostgreSQL (Supabase) |
| topfinanzas-\* (financial platforms) | No authentication | N/A — public content sites | N/A                   |
| emailgenius-broadcasts-generator     | Internal tool     | GCP service account        | N/A                   |
| arbitrage-manager-dashboard          | Internal tool     | GCP IAM / service accounts | N/A                   |

Financial content platforms (topfinanzas-us-next, uk-topfinanzas-com, budgetbee-next, kardtrust) are **public-facing and have no user authentication**. They do not implement login, sessions, or protected routes.

---

## Better Auth — Route-Genius Implementation

### Installation & Setup

```bash
npm install better-auth pg
```

```typescript
// lib/auth.ts — server-side auth configuration
import { betterAuth } from "better-auth";
import { Pool } from "pg";

export const auth = betterAuth({
  database: new Pool({
    connectionString: process.env.DATABASE_URL,
  }),

  socialProviders: {
    google: {
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
    },
  },

  // Domain restriction — only @topnetworks.co and @topfinanzas.com allowed
  user: {
    additionalFields: {},
  },

  // Trusted origins (must match deployment environments)
  trustedOrigins: [
    "http://localhost:3070",
    "https://route-genius.vercel.app",
    "https://route.topnetworks.co",
  ],

  session: {
    expiresIn: 60 * 60 * 24 * 7, // 7 days
    updateAge: 60 * 60 * 24, // Refresh daily
    cookieCache: {
      enabled: true,
      maxAge: 5 * 60, // 5-minute cookie cache
    },
  },
});
```

```typescript
// lib/auth-client.ts — client-side auth configuration
import { createAuthClient } from "better-auth/react";

export const authClient = createAuthClient({
  baseURL: process.env.NEXT_PUBLIC_APP_URL ?? "http://localhost:3070",
});

export const { signIn, signOut, useSession } = authClient;
```

### API Route Handler

```typescript
// app/api/auth/[...all]/route.ts
import { auth } from "@/lib/auth";
import { toNextJsHandler } from "better-auth/next-js";

export const { GET, POST } = toNextJsHandler(auth);
```

---

## Domain Restriction

Only `@topnetworks.co` and `@topfinanzas.com` email domains are permitted. This is enforced at the application level in `lib/auth.ts` using Better Auth's callback hooks:

```typescript
// lib/auth.ts — domain restriction implementation
export const auth = betterAuth({
  // ...base config...

  hooks: {
    after: [
      {
        matcher(context) {
          return context.path === "/sign-in/social/callback";
        },
        handler: async (ctx) => {
          const email = ctx.context?.session?.user?.email ?? "";
          const allowedDomains = ["@topnetworks.co", "@topfinanzas.com"];
          const isAllowed = allowedDomains.some((domain) =>
            email.endsWith(domain),
          );

          if (!isAllowed) {
            // Sign out the user and redirect to login with error
            await auth.api.signOut({ headers: ctx.request.headers });
            return Response.redirect(
              `${process.env.NEXT_PUBLIC_APP_URL}/login?error=unauthorized_domain`,
            );
          }
        },
      },
    ],
  },
});
```

---

## Session Management

### Server-Side Session Access

```typescript
// lib/auth-session.ts
import { auth } from "./auth";
import { headers } from "next/headers";
import { redirect } from "next/navigation";

// Use in Server Components and Server Actions
export async function getServerSession() {
  const session = await auth.api.getSession({
    headers: await headers(),
  });
  return session;
}

// Auth guard — use at the top of every Server Action
export async function requireUserId(): Promise<string> {
  const session = await getServerSession();
  if (!session?.user?.id) {
    redirect("/login");
  }
  return session.user.id;
}

// Auth guard — use in Server Components (doesn't redirect, returns null)
export async function getOptionalSession() {
  const session = await getServerSession();
  return session?.user ?? null;
}
```

### Client-Side Session Access

```typescript
"use client";
import { useSession } from "@/lib/auth-client";

export function UserAvatar() {
  const { data: session, isPending } = useSession();

  if (isPending) return <Skeleton />;
  if (!session) return null;

  return (
    <img
      src={session.user.image ?? "/default-avatar.png"}
      alt={session.user.name}
    />
  );
}
```

---

## Middleware Protection (proxy.ts)

Route-genius uses `proxy.ts` (Next.js 16 middleware convention). Next.js 15 projects use `middleware.ts`.

```typescript
// proxy.ts (route-genius — Next.js 16)
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

// Routes accessible without authentication
const PUBLIC_PATHS = [
  "/login",
  "/api/auth", // Better Auth endpoints
  "/api/redirect", // Public redirect endpoint
  "/api/analytics", // Public analytics API
  "/analytics", // Public analytics pages
  "/privacy",
  "/terms",
];

export function middleware(request: NextRequest) {
  const pathname = request.nextUrl.pathname;

  const isPublic = PUBLIC_PATHS.some((p) => pathname.startsWith(p));
  if (isPublic) return NextResponse.next();

  // Cookie name varies by protocol:
  // HTTPS: __Secure-better-auth.session_token
  // HTTP (localhost): better-auth.session_token
  const sessionCookie =
    request.cookies.get("__Secure-better-auth.session_token") ||
    request.cookies.get("better-auth.session_token");

  if (!sessionCookie) {
    const loginUrl = new URL("/login", request.url);
    loginUrl.searchParams.set("callbackUrl", pathname);
    return NextResponse.redirect(loginUrl);
  }

  // Redirect authenticated users away from /login
  if (pathname === "/login") {
    return NextResponse.redirect(new URL("/dashboard", request.url));
  }

  return NextResponse.next();
}

export const config = {
  matcher: ["/((?!_next/static|_next/image|favicon.ico|.*\\.png$).*)"],
};
```

---

## Login Page Implementation

```typescript
// app/login/page.tsx
"use client";
import { signIn } from "@/lib/auth-client";
import { Button } from "@/components/ui/button";
import Image from "next/image";

export default function LoginPage() {
  async function handleGoogleSignIn() {
    await signIn.social({
      provider: "google",
      callbackURL: "/dashboard",
    });
  }

  return (
    <main className="min-h-screen flex items-center justify-center bg-gradient-to-br from-lime-50 via-cyan-50 to-blue-100">
      <div className="bg-white rounded-2xl shadow-lg p-8 w-full max-w-md text-center">
        <Image
          src="https://storage.googleapis.com/media-topfinanzas-com/images/topnetworks-positivo-sinfondo.webp"
          alt="TopNetworks"
          width={180}
          height={40}
          className="mx-auto mb-6"
        />
        <h1 className="text-2xl font-bold text-gray-800 mb-2">Sign in</h1>
        <p className="text-gray-600 mb-8 text-sm">
          Access is restricted to @topnetworks.co and @topfinanzas.com accounts.
        </p>
        <Button
          onClick={handleGoogleSignIn}
          className="w-full bg-blue-600 hover:bg-blue-700 text-white font-semibold py-3"
        >
          Continue with Google
        </Button>
      </div>
    </main>
  );
}
```

---

## Better Auth Database Tables

Better Auth auto-manages these tables — **never modify them manually**:

| Table          | Purpose                                                   |
| -------------- | --------------------------------------------------------- |
| `user`         | User profiles (id, name, email, image, createdAt)         |
| `session`      | Active sessions (id, userId, token, expiresAt)            |
| `account`      | OAuth account links (userId, provider, providerAccountId) |
| `verification` | Email verification tokens                                 |

These tables are created/migrated automatically when Better Auth initializes with a database connection.

### Better Auth Migration (initial setup)

```typescript
// Run once to create Better Auth tables
import { auth } from "@/lib/auth";
await auth.api.migrate(); // Creates all Better Auth tables
```

---

## Google Drive OAuth (Secondary — Backup Feature)

Route-genius uses a separate OAuth flow for Google Drive access, stored in HTTP-only cookies:

```typescript
// app/api/auth/google-drive/callback/route.ts
import { cookies } from "next/headers";
import type { NextRequest } from "next/server";

export async function GET(request: NextRequest) {
  const { searchParams } = new URL(request.url);
  const code = searchParams.get("code");

  if (!code) {
    return Response.redirect(
      `${process.env.NEXT_PUBLIC_APP_URL}/dashboard/settings?error=drive_auth_failed`,
    );
  }

  // Exchange code for tokens
  const tokenResponse = await fetch("https://oauth2.googleapis.com/token", {
    method: "POST",
    body: new URLSearchParams({
      code,
      client_id: process.env.GOOGLE_DRIVE_CLIENT_ID!,
      client_secret: process.env.GOOGLE_DRIVE_CLIENT_SECRET!,
      redirect_uri: process.env.GOOGLE_DRIVE_REDIRECT_URI!,
      grant_type: "authorization_code",
    }),
  });

  const tokens = await tokenResponse.json();

  // Store in HTTP-only cookie — 30 days
  const cookieStore = await cookies();
  cookieStore.set("rg_gdrive_tokens", JSON.stringify(tokens), {
    httpOnly: true,
    secure: process.env.NODE_ENV === "production",
    maxAge: 60 * 60 * 24 * 30,
    path: "/",
  });

  return Response.redirect(
    `${process.env.NEXT_PUBLIC_APP_URL}/dashboard/settings?drive=connected`,
  );
}
```

Google Drive OAuth scope: `https://www.googleapis.com/auth/drive.file` (only files created by the app — principle of least privilege).

---

## Cookie Naming Convention

Better Auth uses different cookie name prefixes based on protocol:

| Environment        | Cookie Name                          |
| ------------------ | ------------------------------------ |
| Production (HTTPS) | `__Secure-better-auth.session_token` |
| Development (HTTP) | `better-auth.session_token`          |

Always check for both when reading session cookies in middleware.

---

## Environment Variables Required

```bash
# Better Auth core
DATABASE_URL=                           # PostgreSQL connection string for session storage
NEXT_PUBLIC_APP_URL=                    # Canonical app URL
BETTER_AUTH_URL=                        # Optional — overrides NEXT_PUBLIC_APP_URL for auth

# Google OAuth (user authentication)
GOOGLE_CLIENT_ID=                       # Google OAuth 2.0 Client ID
GOOGLE_CLIENT_SECRET=                   # Google OAuth 2.0 Client Secret

# Google Drive OAuth (backup feature — separate OAuth client)
GOOGLE_DRIVE_CLIENT_ID=
GOOGLE_DRIVE_CLIENT_SECRET=
GOOGLE_DRIVE_REDIRECT_URI=              # e.g. http://localhost:3070/api/auth/google-drive/callback
```

---

## Access Control Patterns

### Server Action Guard

```typescript
// Pattern: always first line in any Server Action
export async function mutateDataAction(data: InputType) {
  const userId = await requireUserId(); // Redirects to /login if not authenticated

  // All subsequent code runs only for authenticated users
  const result = await performOperation(data, userId);
  return { success: true, data: result };
}
```

### Server Component Guard

```typescript
// Pattern: in Server Components where you want to conditionally render
import { getServerSession } from "@/lib/auth-session";
import { redirect } from "next/navigation";

export default async function DashboardPage() {
  const session = await getServerSession();
  if (!session) redirect("/login");

  // Render authenticated UI
  return <Dashboard user={session.user} />;
}
```

### Client Component Session Check

```typescript
"use client";
import { useSession } from "@/lib/auth-client";
import { useRouter } from "next/navigation";
import { useEffect } from "react";

export function ProtectedClientComponent() {
  const { data: session, isPending } = useSession();
  const router = useRouter();

  useEffect(() => {
    if (!isPending && !session) {
      router.push("/login");
    }
  }, [session, isPending, router]);

  if (isPending) return <LoadingSkeleton />;
  if (!session) return null;

  return <div>Authenticated content</div>;
}
```

---

## Sign Out

```typescript
"use client";
import { signOut } from "@/lib/auth-client";
import { useRouter } from "next/navigation";

export function SignOutButton() {
  const router = useRouter();

  async function handleSignOut() {
    await signOut({
      fetchOptions: {
        onSuccess: () => router.push("/login"),
      },
    });
  }

  return <button onClick={handleSignOut}>Sign out</button>;
}
```

---

## Constraints

- **Never** implement custom session storage — use Better Auth's PostgreSQL adapter
- **Never** store sensitive tokens (OAuth tokens, session data) in `localStorage` — use HTTP-only cookies
- **Never** manually modify Better Auth tables (`user`, `session`, `account`, `verification`)
- **Never** skip `requireUserId()` in Server Actions that access user data
- **Never** bypass domain restriction for convenience — `@topnetworks.co` and `@topfinanzas.com` only
- **Never** use the same Google OAuth client ID for user authentication and Google Drive backup — these are separate OAuth credentials
- The `SUPABASE_SERVICE_ROLE_KEY` bypasses RLS — this is intentional for server-side operations but must never be used for auth validation
- Session cookie prefix (`__Secure-`) is auto-applied on HTTPS — do not hard-code the prefix; always check for both variants in middleware
- Do not add `auth.uid()` to Supabase RLS policies — Better Auth does not use Supabase Auth, so `auth.uid()` will always return null
