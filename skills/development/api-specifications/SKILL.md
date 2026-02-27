---
name: api-specifications
description: Guidance for Splits Network REST API design, implementation, and documentation
alwaysApply: false
applyTo:
    - "services/**/routes/**"
    - "services/**/src/v2/**"
    - "apps/**/src/app/api/**"
    - "docs/guidance/api-*.md"
    - "docs/guidance/*-endpoints-*.md"
---

# API Specifications Skill

This skill provides guidance for designing, implementing, and documenting REST APIs in the Splits Network platform.

## Purpose

Help developers create consistent, well-documented REST APIs that follow Splits Network standards:

- **V2 Architecture**: Standardized 5-route CRUD pattern
- **Response Format**: Consistent `{ data, pagination }` envelope
- **Access Control**: Role-based filtering via shared access context
- **Event Publishing**: Domain events for state changes
- **Documentation**: Clear endpoint specs and examples

## When to Use This Skill

Use this skill when:

- Creating new REST API endpoints
- Migrating V1 endpoints to V2 architecture
- Documenting API contracts
- Implementing pagination, filtering, or search
- Adding validation or error handling
- Publishing domain events

## V2 API Architecture Standards

### 1. Standardized 5-Route Pattern

**Every V2 resource** follows this exact pattern:

```typescript
// 1. LIST - Role-scoped collection
GET /api/v2/:resource?search=X&status=Y&sort_by=Z&page=1&limit=25
Response: { data: [...], pagination: { total, page, limit, total_pages } }

// 2. GET BY ID - Single resource
GET /api/v2/:resource/:id?include=related1,related2
Response: { data: {...} }

// 3. CREATE - New resource
POST /api/v2/:resource
Body: { field1: value1, field2: value2, ... }
Response: { data: {...} }

// 4. UPDATE - Single method handles ALL updates
PATCH /api/v2/:resource/:id
Body: { field1: newValue1, status: newStatus, ... }
Response: { data: {...} }

// 5. DELETE - Soft delete
DELETE /api/v2/:resource/:id
Response: { data: { message: 'Deleted successfully' } }
```

### 2. Response Format Standard

**ALL responses MUST use the wrapped envelope**:

```typescript
// Success response
reply.send({ data: <payload> })

// List response with pagination
reply.send({
  data: [...],
  pagination: { total, page, limit, total_pages }
})

// Error response
reply.code(400).send({
  error: { code: "ERROR_CODE", message: "..." }
})
```

**NEVER return unwrapped data**: `reply.send(payload)` is incorrect.

### 3. Domain-Based Folder Structure

```
services/<service>/src/v2/
├── shared/                 # Shared V2 utilities
│   ├── events.ts           # EventPublisher class
│   ├── helpers.ts          # requireUserContext, validation
│   └── pagination.ts       # PaginationParams, PaginationResponse
├── <domain>/               # Domain folder (e.g., jobs, candidates)
│   ├── types.ts            # Domain-specific types
│   ├── repository.ts       # Data access with role-based filtering
│   └── service.ts          # Business logic, validation, events
└── routes.ts               # All V2 routes (imports from domains)
```

### 4. Repository Pattern with Access Context

```typescript
import { resolveAccessContext } from "@splits-network/shared-access-context";

export class ResourceRepository {
    constructor(private supabase: SupabaseClient) {}

    async list(clerkUserId: string, filters: ResourceFilters) {
        const context = await resolveAccessContext(clerkUserId, this.supabase);

        const query = this.supabase.from("resources").select("*");

        // Apply role-based filtering from access context
        if (context.role === "recruiter") {
            query.eq("user_id", context.userId);
        } else if (context.role === "company_admin") {
            query.in("company_id", context.accessibleCompanyIds);
        }
        // Platform admins see everything (no filter)

        // Apply search/sorting filters
        if (filters.search) {
            query.ilike("name", `%${filters.search}%`);
        }
        if (filters.sort_by) {
            query.order(filters.sort_by, {
                ascending: filters.sort_order !== "desc",
            });
        }

        return query;
    }
}
```

### 5. Service Layer with Events

```typescript
export class ResourceServiceV2 {
    constructor(
        private repository: ResourceRepository,
        private events: EventPublisher,
    ) {}

    async create(clerkUserId: string, data: ResourceCreate) {
        // Validate input
        this.validateResourceData(data);

        // Create via repository
        const resource = await this.repository.create(clerkUserId, data);

        // Publish event after successful creation
        await this.events.publish("resource.created", {
            resourceId: resource.id,
            createdBy: clerkUserId,
        });

        return resource;
    }

    async update(id: string, clerkUserId: string, updates: ResourceUpdate) {
        // Smart validation based on what's being updated
        if (updates.status) {
            this.validateStatusTransition(updates.status);
        }

        const updated = await this.repository.update(id, clerkUserId, updates);

        // Publish event
        await this.events.publish("resource.updated", {
            resourceId: id,
            changes: Object.keys(updates),
            updatedBy: clerkUserId,
        });

        return updated;
    }
}
```

### 6. Route Implementation

```typescript
import { FastifyInstance } from "fastify";

export async function resourceRoutes(
    app: FastifyInstance,
    service: ResourceServiceV2,
) {
    // LIST
    app.get("/api/v2/resources", async (request, reply) => {
        const clerkUserId = request.headers["x-clerk-user-id"] as string;
        const {
            search,
            status,
            sort_by,
            sort_order,
            page = 1,
            limit = 25,
        } = request.query as any;

        const result = await service.list(clerkUserId, {
            search,
            status,
            sort_by,
            sort_order,
            page,
            limit,
        });

        return reply.send(result); // Service returns { data, pagination }
    });

    // GET BY ID
    app.get("/api/v2/resources/:id", async (request, reply) => {
        const clerkUserId = request.headers["x-clerk-user-id"] as string;
        const { id } = request.params as { id: string };
        const { include } = request.query as { include?: string };

        const resource = await service.getById(id, clerkUserId, include);
        return reply.send({ data: resource });
    });

    // CREATE
    app.post("/api/v2/resources", async (request, reply) => {
        const clerkUserId = request.headers["x-clerk-user-id"] as string;
        const data = request.body as ResourceCreate;

        const resource = await service.create(clerkUserId, data);
        return reply.code(201).send({ data: resource });
    });

    // UPDATE
    app.patch("/api/v2/resources/:id", async (request, reply) => {
        const clerkUserId = request.headers["x-clerk-user-id"] as string;
        const { id } = request.params as { id: string };
        const updates = request.body as ResourceUpdate;

        const resource = await service.update(id, clerkUserId, updates);
        return reply.send({ data: resource });
    });

    // DELETE
    app.delete("/api/v2/resources/:id", async (request, reply) => {
        const clerkUserId = request.headers["x-clerk-user-id"] as string;
        const { id } = request.params as { id: string };

        await service.delete(id, clerkUserId);
        return reply.send({ data: { message: "Deleted successfully" } });
    });
}
```

## Query Parameters

### List Endpoints

Standard query parameters for list endpoints:

```typescript
interface StandardListParams {
    page?: number; // Page number (1-based)
    limit?: number; // Items per page (default 25, max 100)
    search?: string; // Search term (service-specific fields)
    sort_by?: string; // Field to sort by
    sort_order?: "asc" | "desc"; // Sort direction
    filters?: Record<string, any>; // Domain-specific filters
    include?: string; // Comma-separated related resources
}
```

**Examples**:

- `/api/v2/jobs?page=1&limit=25&search=engineer&status=active`
- `/api/v2/applications?candidate_id=123&stage=screen&sort_by=created_at&sort_order=desc`
- `/api/v2/candidates?include=documents,applications`

### Get By ID Endpoints

Support `include` parameter for related data:

```typescript
// GET /api/v2/applications/:id?include=candidate,job,ai_review
if (includes.includes("candidate")) {
    application.candidate = await this.getCandidateData(
        application.candidate_id,
    );
}
```

## Pagination

All list endpoints must return pagination metadata:

```typescript
interface PaginationResponse {
  total: number;        // Total items across all pages
  page: number;         // Current page number (1-based)
  limit: number;        // Items per page
  total_pages: number;  // Total number of pages
}

// Example response
{
  "data": [...],
  "pagination": {
    "total": 1000,
    "page": 1,
    "limit": 25,
    "total_pages": 40
  }
}
```

## Error Handling

Return structured error responses:

```typescript
// 400 Bad Request - Validation error
reply.code(400).send({
    error: {
        code: "VALIDATION_ERROR",
        message: "Invalid input data",
        details: { field: "email", reason: "Invalid email format" },
    },
});

// 401 Unauthorized - Missing/invalid auth
reply.code(401).send({
    error: {
        code: "UNAUTHORIZED",
        message: "Authentication required",
    },
});

// 403 Forbidden - Insufficient permissions
reply.code(403).send({
    error: {
        code: "FORBIDDEN",
        message: "Insufficient permissions to access this resource",
    },
});

// 404 Not Found - Resource doesn't exist
reply.code(404).send({
    error: {
        code: "NOT_FOUND",
        message: "Resource not found",
        details: { resourceId: id },
    },
});

// 409 Conflict - Duplicate or constraint violation
reply.code(409).send({
    error: {
        code: "CONFLICT",
        message: "Resource already exists",
        details: { constraint: "unique_email" },
    },
});

// 500 Internal Server Error - Unexpected error
reply.code(500).send({
    error: {
        code: "INTERNAL_ERROR",
        message: "An unexpected error occurred",
    },
});
```

## Event Publishing

Publish domain events for significant state changes:

```typescript
// After successful create
await this.events.publish("resource.created", {
    resourceId: resource.id,
    companyId: resource.company_id,
    createdBy: clerkUserId,
});

// After successful update
await this.events.publish("resource.updated", {
    resourceId: id,
    changes: Object.keys(updates),
    updatedBy: clerkUserId,
});

// After successful delete
await this.events.publish("resource.deleted", {
    resourceId: id,
    deletedBy: clerkUserId,
});

// Domain-specific events
await this.events.publish("application.stage_changed", {
    applicationId: id,
    oldStage: oldStage,
    newStage: newStage,
    changedBy: clerkUserId,
});
```

## API Documentation

Document each endpoint with:

1. **Purpose**: What the endpoint does
2. **Path**: Full path with parameters
3. **Method**: HTTP method (GET, POST, PATCH, DELETE)
4. **Auth**: Required authentication (Clerk JWT)
5. **Access**: Who can access (roles, scoping rules)
6. **Query Params**: Optional parameters
7. **Request Body**: Expected payload structure
8. **Response**: Success response format
9. **Errors**: Possible error codes and meanings
10. **Example**: Full request/response example

### Example Documentation

````markdown
### List Jobs

Returns a paginated list of jobs based on role-based access control.

**Endpoint**: `GET /api/v2/jobs`

**Authentication**: Required (Clerk JWT)

**Access Control**:

- **Recruiters**: See assigned jobs only
- **Company Users**: See jobs from their organization
- **Platform Admins**: See all jobs

**Query Parameters**:

- `page` (number, optional): Page number (default: 1)
- `limit` (number, optional): Items per page (default: 25, max: 100)
- `search` (string, optional): Search in job title
- `status` (string, optional): Filter by status (active, paused, closed)
- `company_id` (uuid, optional): Filter by company
- `sort_by` (string, optional): Sort field (created_at, title)
- `sort_order` (string, optional): Sort direction (asc, desc)

**Response**: `200 OK`

```json
{
    "data": [
        {
            "id": "123e4567-e89b-12d3-a456-426614174000",
            "title": "Senior Software Engineer",
            "company_id": "123e4567-e89b-12d3-a456-426614174001",
            "status": "active",
            "location": "San Francisco, CA",
            "created_at": "2026-01-13T10:00:00Z"
        }
    ],
    "pagination": {
        "total": 100,
        "page": 1,
        "limit": 25,
        "total_pages": 4
    }
}
```
````

**Errors**:

- `401 Unauthorized`: Missing or invalid authentication
- `403 Forbidden`: Insufficient permissions
- `500 Internal Server Error`: Server error

**Example Request**:

```bash
curl -X GET "https://api.splits.network/api/v2/jobs?page=1&limit=25&status=active" \
  -H "Authorization: Bearer <clerk-jwt>"
```

````

## Common Patterns

### Current User Access Pattern

For user-specific singleton resources, use the `/me` alias pattern:

```typescript
// ✅ RECOMMENDED - Use /me alias on existing GET by ID endpoint
GET /api/v2/candidates/me
// Resolves to user's actual ID, returns { data: {...} }

// Implementation in route handler
app.get('/api/v2/candidates/:id', async (request, reply) => {
  const clerkUserId = request.headers['x-clerk-user-id'] as string;
  let { id } = request.params;

  // Resolve "me" to actual user ID
  if (id === 'me') {
    const context = await resolveAccessContext(clerkUserId, supabase);
    id = context.userId; // Actual UUID
  }

  // Standard getById logic - no special handling needed
  const candidate = await service.getById(id, clerkUserId);
  return reply.send({ data: candidate });
});
````

**Benefits**:

- **Intuitive**: Clear what `/me` means
- **Performant**: Direct ID lookup (not filtered list query)
- **Correct Shape**: Returns `{ data: {...} }` (singleton, not array)
- **No New Endpoint**: Just an alias within existing GET by ID route
- **Still Secure**: Access context validates user can access resolved ID

### Single Update Method

One update method handles ALL updates with smart validation:

```typescript
// ❌ WRONG - Multiple update endpoints
PATCH /api/v2/jobs/:id/status
PATCH /api/v2/jobs/:id/title
PATCH /api/v2/jobs/:id/close

// ✅ CORRECT - Single update endpoint
PATCH /api/v2/jobs/:id
Body: { status: "closed" } or { title: "New Title" } or any field
```

### Include Parameters vs Child Endpoints

Use include parameters for related data, NOT child endpoints:

```typescript
// ❌ WRONG - Child endpoints
GET /api/v2/applications/:id/documents
GET /api/v2/applications/:id/ai-review

// ✅ CORRECT - Include parameters
GET /api/v2/applications/:id?include=documents,ai_review
GET /api/v2/documents?application_id=:id
GET /api/v2/ai-reviews?application_id=:id
```

### Data Enrichment with JOINs

Services share the same database (all tables in `public` schema) and can enrich data with JOINs:

```typescript
// Enrich applications with candidate and job data
const enrichedApplications = await this.supabase.from("applications").select(`
    *,
    candidate:candidates(*),
    job:jobs(*),
    recruiter:recruiters(*)
  `);

// Note: All domain tables (applications, candidates, jobs, recruiters, etc.)
// are in the public schema. No cross-schema queries needed.
```

## References

- **API Response Format**: `docs/guidance/api-response-format.md`
- **Pagination Standard**: `docs/guidance/pagination.md`
- **V2 Architecture Guide**: `docs/migration/v2/V2-ARCHITECTURE-IMPLEMENTATION-GUIDE.md`
- **Service Architecture**: `docs/guidance/service-architecture-pattern.md`
- **Access Context**: `packages/shared-access-context/README.md`

## Examples from Production

**V2 Services**:

- Identity Service: `services/identity-service/src/v2/`
- ATS Service: `services/ats-service/src/v2/`
- Network Service: `services/network-service/src/v2/`
- Billing Service: `services/billing-service/src/v2/`
- Notification Service: `services/notification-service/src/v2/`

**API Gateway Routes**:

- Gateway V2 Proxy: `services/api-gateway/src/routes/v2/`
