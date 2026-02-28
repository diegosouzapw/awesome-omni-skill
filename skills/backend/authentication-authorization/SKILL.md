---
name: authentication-authorization
description: Authentication and authorization patterns using Clerk and RBAC
alwaysApply: false
applyTo:
    - "services/api-gateway/src/rbac.ts"
    - "services/**/v2/shared/access.ts"
    - "apps/**/middleware.ts"
    - "apps/**/src/lib/auth/**"
---

# Authentication & Authorization Skill

This skill provides guidance for implementing authentication and authorization in Splits Network.

## Purpose

Help developers implement secure, role-based access control:

- **Clerk Integration**: Authentication with Clerk
- **V2 Access Context**: Role-based data filtering
- **API Gateway RBAC**: Route-level authorization
- **Frontend Auth**: Protected routes and role checks
- **User Identification**: Proper ID handling across services

## When to Use This Skill

Use this skill when:

- Implementing protected API endpoints
- Creating repository methods with role-based filtering
- Building protected frontend pages
- Checking user permissions
- Debugging authorization issues

## Core Principles

### 1. V2 Access Context Pattern (Recommended)

**All V2 services** use shared access context for data-level authorization:

```typescript
// packages/shared-access-context/src/index.ts
import { SupabaseClient } from "@supabase/supabase-js";

export interface AccessContext {
    userId: string; // Internal UUID (users.id)
    clerkUserId: string; // Clerk user ID
    role: UserRole; // Primary role
    memberships: Membership[]; // Organization memberships
    isCompanyUser: boolean; // Has company affiliation
    isRecruiter: boolean; // Is recruiter (independent or affiliated)
    accessibleCompanyIds: string[]; // Companies user can access
}

export async function resolveAccessContext(
    clerkUserId: string,
    supabase: SupabaseClient,
): Promise<AccessContext> {
    // 1. Lookup internal user ID
    const { data: user } = await supabase
        .from("users")
        .select("id")
        .eq("clerk_user_id", clerkUserId)
        .single();

    if (!user) {
        throw new Error("User not found");
    }

    // 2. Get memberships
    const { data: memberships } = await supabase
        .from("memberships")
        .select("*, organizations(*)")
        .eq("user_id", user.id);

    // 3. Determine role and permissions
    const isPlatformAdmin = memberships?.some(
        (m) => m.role === "platform_admin",
    );
    const isCompanyAdmin = memberships?.some((m) => m.role === "company_admin");
    const isHiringManager = memberships?.some(
        (m) => m.role === "hiring_manager",
    );

    // 4. Check recruiter status
    const { data: recruiter } = await supabase
        .from("recruiters")
        .select("id, status")
        .eq("user_id", user.id)
        .single();

    const isRecruiter = !!recruiter && recruiter.status === "active";

    // 5. Get accessible company IDs
    const accessibleCompanyIds =
        memberships?.map((m) => m.organization.id).filter(Boolean) || [];

    return {
        userId: user.id,
        clerkUserId,
        role: isPlatformAdmin
            ? "platform_admin"
            : isCompanyAdmin
              ? "company_admin"
              : isHiringManager
                ? "hiring_manager"
                : isRecruiter
                  ? "recruiter"
                  : "user",
        memberships: memberships || [],
        isCompanyUser: isCompanyAdmin || isHiringManager,
        isRecruiter,
        accessibleCompanyIds,
    };
}
```

See [examples/access-context.ts](./examples/access-context.ts).

### 2. Repository with Access Context

Apply role-based filtering in repository queries:

```typescript
// services/ats-service/src/v2/jobs/repository.ts
import { resolveAccessContext } from "@splits-network/shared-access-context";

export class JobRepository {
    constructor(private supabase: SupabaseClient) {}

    async list(clerkUserId: string, filters: JobFilters): Promise<Job[]> {
        const context = await resolveAccessContext(clerkUserId, this.supabase);

        const query = this.supabase.from("jobs").select("*");

        // Apply role-based filtering
        if (context.role === "platform_admin") {
            // Platform admins see all jobs (no filter)
        } else if (context.isCompanyUser) {
            // Company users see their organization's jobs
            query.in("company_id", context.accessibleCompanyIds);
        } else if (context.isRecruiter) {
            // Recruiters see jobs they're assigned to
            const { data: assignments } = await this.supabase
                .from("role_assignments")
                .select("job_id")
                .eq("recruiter_user_id", context.userId);

            query.in("id", assignments?.map((a) => a.job_id) || []);
        } else {
            // Regular users see no jobs
            query.eq("id", "impossible"); // Force empty result
        }

        // Apply additional filters
        if (filters.status) {
            query.eq("status", filters.status);
        }

        const { data, error } = await query;
        if (error) throw error;

        return data || [];
    }

    async getById(id: string, clerkUserId: string): Promise<Job | null> {
        const context = await resolveAccessContext(clerkUserId, this.supabase);

        const { data, error } = await this.supabase
            .from("jobs")
            .select("*")
            .eq("id", id)
            .single();

        if (error || !data) return null;

        // Check access
        if (context.role === "platform_admin") {
            return data; // Admins can see all
        } else if (context.isCompanyUser) {
            // Check company ownership
            if (!context.accessibleCompanyIds.includes(data.company_id)) {
                throw new ForbiddenError("Access denied to this job");
            }
        } else if (context.isRecruiter) {
            // Check assignment
            const { data: assignment } = await this.supabase
                .from("role_assignments")
                .select("id")
                .eq("job_id", id)
                .eq("recruiter_user_id", context.userId)
                .single();

            if (!assignment) {
                throw new ForbiddenError("Access denied to this job");
            }
        } else {
            throw new ForbiddenError("Access denied");
        }

        return data;
    }
}
```

**Key Rules**:

- ✅ Always call `resolveAccessContext` at start of repository method
- ✅ Apply role-based filtering to ALL queries
- ✅ Platform admins see everything (no filter)
- ✅ Company users see their organization's data
- ✅ Recruiters see assigned data
- ✅ Regular users see minimal data (or none)
- ❌ Never skip access context checks

See [examples/repository-with-access-context.ts](./examples/repository-with-access-context.ts).

### 3. API Gateway RBAC (V1 Pattern - Legacy)

API Gateway enforces route-level authorization:

```typescript
// services/api-gateway/src/rbac.ts
import { FastifyRequest, FastifyReply } from "fastify";

export type UserRole =
    | "platform_admin"
    | "company_admin"
    | "hiring_manager"
    | "recruiter"
    | "candidate"
    | "user";

export function requireRoles(
    allowedRoles: UserRole[],
    services?: ServiceRegistry,
) {
    return async (request: FastifyRequest, reply: FastifyReply) => {
        const clerkUserId = request.auth?.clerkUserId;

        if (!clerkUserId) {
            return reply.code(401).send({
                error: {
                    code: "UNAUTHORIZED",
                    message: "Authentication required",
                },
            });
        }

        // Check memberships first (fast path)
        const memberships = request.auth.memberships || [];

        // Platform admin always allowed
        if (isPlatformAdmin(memberships)) {
            return;
        }

        // Check company roles
        if (
            allowedRoles.includes("company_admin") &&
            isCompanyAdmin(memberships)
        ) {
            return;
        }

        if (
            allowedRoles.includes("hiring_manager") &&
            isHiringManager(memberships)
        ) {
            return;
        }

        // Check recruiter status (requires network service call)
        if (allowedRoles.includes("recruiter") && services) {
            const isRecruiterActive = await isRecruiter(
                memberships,
                clerkUserId,
                services.network,
            );

            if (isRecruiterActive) {
                return;
            }
        }

        // Check candidate status (requires ATS service call)
        if (allowedRoles.includes("candidate") && services) {
            const { data: candidates } = await services.ats.get(
                `/api/v2/candidates?limit=1`,
                { headers: { "x-clerk-user-id": clerkUserId } },
            );

            if (candidates?.data?.length > 0) {
                return;
            }
        }

        // No matching roles
        return reply.code(403).send({
            error: {
                code: "FORBIDDEN",
                message: "You do not have permission to access this resource",
            },
        });
    };
}

// Helper functions
export function isPlatformAdmin(memberships: Membership[]): boolean {
    return memberships.some((m) => m.role === "platform_admin");
}

export function isCompanyAdmin(
    memberships: Membership[],
    orgId?: string,
): boolean {
    return memberships.some(
        (m) =>
            m.role === "company_admin" &&
            (!orgId || m.organization_id === orgId),
    );
}

export async function isRecruiter(
    memberships: Membership[],
    userId: string,
    networkService: NetworkServiceClient,
): Promise<boolean> {
    // Check memberships first
    if (memberships.some((m) => m.role === "recruiter")) {
        return true;
    }

    // Check independent recruiter status
    try {
        const recruiter = await networkService.getRecruiterByUserId(userId);
        return recruiter?.status === "active";
    } catch (error) {
        return false;
    }
}
```

**Gateway RBAC Rules**:

- ✅ Gateway enforces route-level authorization
- ✅ Backend services apply data-level filtering
- ✅ Always pass `services` parameter when allowing recruiters/candidates
- ✅ Check memberships first (fast), then external services
- ❌ Backend services should NOT duplicate authorization checks

See [examples/gateway-rbac.ts](./examples/gateway-rbac.ts).

### 4. Protected API Routes

Use `requireRoles` middleware on protected endpoints:

```typescript
// services/api-gateway/src/routes/jobs/routes.ts
import { requireRoles } from "../../rbac";

export async function jobsRoutes(
    app: FastifyInstance,
    services: ServiceRegistry,
) {
    // Public endpoint - no auth required
    app.get("/api/v2/jobs/public", async (request, reply) => {
        // Return public job listings
    });

    // Protected endpoint - requires recruiter or company user
    app.get(
        "/api/v2/jobs",
        {
            preHandler: requireRoles(
                [
                    "recruiter",
                    "company_admin",
                    "hiring_manager",
                    "platform_admin",
                ],
                services,
            ),
        },
        async (request, reply) => {
            const clerkUserId = request.auth.clerkUserId;

            // Forward to ATS service with auth headers
            const response = await services.ats.get("/api/v2/jobs", {
                headers: buildAuthHeaders(request),
            });

            return reply.send(response.data);
        },
    );

    // Admin-only endpoint
    app.delete(
        "/api/v2/jobs/:id",
        {
            preHandler: requireRoles(["platform_admin"]),
        },
        async (request, reply) => {
            // Only platform admins can delete
        },
    );
}
```

**Route Protection Rules**:

- ✅ Use `requireRoles` on all protected endpoints
- ✅ Pass `services` when allowing recruiters/candidates
- ✅ Forward auth headers to backend services
- ⚠️ Use minimum required roles (don't over-restrict)

See [examples/protected-routes.ts](./examples/protected-routes.ts).

### 5. Frontend Protected Routes

Protect Next.js pages with Clerk:

```typescript
// apps/portal/src/middleware.ts
import { clerkMiddleware, createRouteMatcher } from "@clerk/nextjs/server";

const isPublicRoute = createRouteMatcher([
    "/sign-in(.*)",
    "/sign-up(.*)",
    "/",
    "/about",
]);

export default clerkMiddleware((auth, req) => {
    if (!isPublicRoute(req)) {
        auth().protect(); // Require authentication
    }
});

export const config = {
    matcher: [
        "/((?!_next|[^?]*\\.(?:html?|css|js(?!on)|jpe?g|webp|png|gif|svg|ttf|woff2?|ico|csv|docx?|xlsx?|zip|webmanifest)).*)",
        "/(api|trpc)(.*)",
    ],
};
```

```typescript
// apps/portal/app/portal/jobs/page.tsx
import { auth } from "@clerk/nextjs/server";
import { redirect } from "next/navigation";

export default async function JobsPage() {
    const { userId } = await auth();

    if (!userId) {
        redirect("/sign-in");
    }

    // Page content...
}
```

**Frontend Auth Rules**:

- ✅ Use Clerk middleware for route protection
- ✅ Check auth in page components
- ✅ Redirect unauthenticated users to sign-in
- ✅ Show loading states during auth checks

See [examples/protected-frontend-routes.tsx](./examples/protected-frontend-routes.tsx).

### 6. Role-Based UI

Show/hide UI based on user role:

```typescript
'use client';

import { useUser } from '@clerk/nextjs';
import { useEffect, useState } from 'react';
import { apiClient } from '@/lib/api-client';

export function RoleBasedActions() {
  const { user } = useUser();
  const [userRole, setUserRole] = useState<string | null>(null);

  useEffect(() => {
    async function fetchRole() {
      const { data } = await apiClient.get('/users/me/role');
      setUserRole(data.role);
    }

    if (user) {
      fetchRole();
    }
  }, [user]);

  if (!userRole) return null;

  return (
    <div className="flex gap-2">
      {/* All authenticated users */}
      <button className="btn">View Jobs</button>

      {/* Recruiters only */}
      {(userRole === 'recruiter' || userRole === 'platform_admin') && (
        <button className="btn btn-primary">Submit Candidate</button>
      )}

      {/* Company admins only */}
      {(userRole === 'company_admin' || userRole === 'platform_admin') && (
        <button className="btn btn-primary">Create Job</button>
      )}

      {/* Platform admins only */}
      {userRole === 'platform_admin' && (
        <button className="btn btn-error">Delete</button>
      )}
    </div>
  );
}
```

**UI Role Rules**:

- ✅ Fetch user role from API
- ✅ Show/hide actions based on role
- ✅ Always enforce permissions on backend (UI is not security)
- ⚠️ Handle loading states gracefully

See [examples/role-based-ui.tsx](./examples/role-based-ui.tsx).

### 7. User Identification Standards

**Critical**: Proper user ID handling across the stack:

```typescript
// ❌ WRONG - Frontend setting user ID header
fetch("/api/jobs", {
    headers: {
        "x-clerk-user-id": userId, // NEVER do this!
    },
});

// ✅ CORRECT - Frontend sends only auth token
fetch("/api/jobs", {
    headers: {
        Authorization: `Bearer ${token}`, // Clerk JWT
    },
});

// ✅ API Gateway extracts user ID from verified JWT
app.addHook("onRequest", async (request, reply) => {
    const token = request.headers.authorization?.replace("Bearer ", "");
    const verified = await verifyClerkJWT(token);
    request.auth = {
        clerkUserId: verified.sub, // ← From verified token, not header
        userId: verified.userId,
    };
});

// ✅ Gateway forwards to backend services
const response = await services.ats.get("/api/v2/jobs", {
    headers: {
        "x-clerk-user-id": request.auth.clerkUserId, // ← From verified JWT
    },
});

// ✅ Backend service uses header (trusts gateway)
export async function jobsRoutes(app: FastifyInstance) {
    app.get("/api/v2/jobs", async (request, reply) => {
        const clerkUserId = request.headers["x-clerk-user-id"] as string;
        const jobs = await repository.list(clerkUserId, filters);
        return reply.send({ data: jobs });
    });
}
```

**User ID Flow**:

1. Frontend: Send `Authorization: Bearer <token>` (Clerk JWT)
2. Gateway: Extract `clerkUserId` from verified JWT
3. Gateway: Forward `x-clerk-user-id: <clerkUserId>` to services
4. Service: Use `x-clerk-user-id` header (trust gateway)

See [references/user-identification-flow.md](./references/user-identification-flow.md).

## Role Hierarchy

```
platform_admin (highest)
  ↓
company_admin
  ↓
hiring_manager
  ↓
recruiter
  ↓
candidate
  ↓
user (lowest)
```

**Role Capabilities**:

- **platform_admin**: Full system access, manage all organizations
- **company_admin**: Manage organization, create jobs, view all org data
- **hiring_manager**: Create jobs, view applications for org jobs
- **recruiter**: Submit candidates, view assigned jobs
- **candidate**: View own profile, manage applications
- **user**: Authenticated user with no special permissions

See [references/role-capabilities.md](./references/role-capabilities.md).

## Testing Authorization

Test access control:

```typescript
describe("JobRepository", () => {
    it("should filter jobs by role", async () => {
        // Mock access context for recruiter
        vi.mocked(resolveAccessContext).mockResolvedValue({
            userId: "user_123",
            clerkUserId: "clerk_123",
            role: "recruiter",
            isRecruiter: true,
            accessibleCompanyIds: [],
        });

        const jobs = await repository.list("clerk_123", {});

        // Should only see assigned jobs
        expect(jobs).toHaveLength(2);
    });

    it("should throw ForbiddenError for unauthorized access", async () => {
        vi.mocked(resolveAccessContext).mockResolvedValue({
            userId: "user_123",
            role: "user",
            isRecruiter: false,
        });

        await expect(
            repository.getById("job_456", "clerk_123"),
        ).rejects.toThrow(ForbiddenError);
    });
});

describe("API Gateway", () => {
    it("should require authentication", async () => {
        const response = await app.inject({
            method: "GET",
            url: "/api/v2/jobs",
            // No auth headers
        });

        expect(response.statusCode).toBe(401);
    });

    it("should enforce role-based access", async () => {
        const response = await app.inject({
            method: "DELETE",
            url: "/api/v2/jobs/123",
            headers: {
                Authorization: "Bearer recruiter_token", // Not admin
            },
        });

        expect(response.statusCode).toBe(403);
    });
});
```

See [examples/authorization-testing.ts](./examples/authorization-testing.ts).

## Anti-Patterns to Avoid

### ❌ Client-Side Only Auth

```typescript
// WRONG - No backend verification
if (user.role === "admin") {
    await deleteJob(id); // Anyone can call this!
}

// CORRECT - Backend enforces auth
await apiClient.delete(`/jobs/${id}`);
// Backend checks role and throws 403 if not authorized
```

### ❌ Skipping Access Context

```typescript
// WRONG - No role filtering
async list(): Promise<Job[]> {
  return await this.supabase.from('jobs').select('*');
}

// CORRECT - Role-based filtering
async list(clerkUserId: string): Promise<Job[]> {
  const context = await resolveAccessContext(clerkUserId, this.supabase);
  // Apply role-based filtering...
}
```

### ❌ Hardcoding Permissions

```typescript
// WRONG - Hardcoded user IDs
if (userId === "specific-user-id") {
    // Grant special access
}

// CORRECT - Role-based
if (context.role === "platform_admin") {
    // Grant access
}
```

## References

- [Access Context Example](./examples/access-context.ts)
- [Repository with Access Context](./examples/repository-with-access-context.ts)
- [Gateway RBAC](./examples/gateway-rbac.ts)
- [Protected Routes](./examples/protected-routes.ts)
- [Protected Frontend Routes](./examples/protected-frontend-routes.tsx)
- [Role-Based UI](./examples/role-based-ui.tsx)
- [Authorization Testing](./examples/authorization-testing.ts)
- [User Identification Flow](./references/user-identification-flow.md)
- [Role Capabilities](./references/role-capabilities.md)

## Related Skills

- `api-specifications` - V2 API patterns with access context
- `database-patterns` - Repository with role-based filtering
- `error-handling` - Auth error responses (401, 403)
