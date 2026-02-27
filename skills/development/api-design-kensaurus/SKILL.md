---
name: api-design
description: Design and review REST/GraphQL APIs with best practices. Use when creating endpoints, reviewing API architecture, designing data contracts, route handlers, Server Actions, webhooks, or when user mentions "API", "endpoint", "REST", "GraphQL", "route handler", "request/response", or "HTTP methods".
---

# API Design Skill

Systematic methodology for designing, reviewing, and improving APIs following industry best practices.

## When to Use

- Designing new API endpoints
- Reviewing existing API architecture
- Creating data contracts between services
- Standardizing API patterns across a codebase
- Migrating or versioning APIs

## CRITICAL: Check Existing First

**Before creating ANY new endpoint, verify:**

1. **Search for existing endpoints:**
```bash
rg "export.*GET|POST|PATCH|DELETE" app/api/ --type ts
ls -la app/api/
```

2. **Check for existing Server Actions:**
```bash
rg "'use server'" --type ts -l
rg "export async function" src/features/*/server/
```

3. **Check existing RPC functions (Supabase):**
```sql
SELECT proname FROM pg_proc WHERE pronamespace = 'public'::regnamespace;
```

4. **Review existing patterns:**
- Check `app/api/` folder structure
- Look at existing error handling patterns
- Verify authentication patterns used

**Why:** Duplicate endpoints cause routing conflicts, inconsistent behavior, and maintenance nightmares. Always search before creating.

## API Design Principles

### 1. RESTful Resource Design

**URL Structure:**
```
GET    /resources          → List resources
GET    /resources/:id      → Get single resource
POST   /resources          → Create resource
PATCH  /resources/:id      → Partial update
PUT    /resources/:id      → Full replace
DELETE /resources/:id      → Delete resource

# Nested resources
GET    /users/:id/posts    → User's posts
POST   /users/:id/posts    → Create post for user

# Actions (when CRUD doesn't fit)
POST   /orders/:id/cancel  → Cancel order
POST   /users/:id/verify   → Verify user
```

**Naming Conventions:**
- Use plural nouns for resources (`/users`, not `/user`)
- Use kebab-case for multi-word (`/user-profiles`)
- Avoid verbs in URLs (let HTTP methods convey action)
- Keep URLs shallow (max 2-3 levels deep)

### 2. Request/Response Design

**Request Structure:**
```typescript
// Query params for filtering/sorting/pagination
GET /products?category=electronics&sort=-price&page=2&limit=20

// Body for mutations
POST /products
{
  "name": "Widget",
  "price": 29.99,
  "categoryId": "cat_123"
}

// Partial updates with PATCH
PATCH /products/123
{
  "price": 24.99  // Only changed fields
}
```

**Response Structure (Consistent Envelope):**
```typescript
// Success response
{
  "data": { /* resource or array */ },
  "meta": {
    "pagination": { "page": 1, "limit": 20, "total": 100 }
  }
}

// Error response
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input",
    "details": [
      { "field": "email", "message": "Invalid email format" }
    ]
  }
}
```

### 3. HTTP Status Codes

| Code | Meaning | When to Use |
|------|---------|-------------|
| 200 | OK | Successful GET, PATCH, PUT |
| 201 | Created | Successful POST |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Invalid input, validation error |
| 401 | Unauthorized | Missing/invalid auth |
| 403 | Forbidden | Auth valid but no permission |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Duplicate, state conflict |
| 422 | Unprocessable | Semantic validation error |
| 429 | Too Many Requests | Rate limited |
| 500 | Internal Error | Server error (never expose details) |

### 4. Pagination Patterns

**Offset-based (simple but slow at scale):**
```typescript
GET /posts?page=5&limit=20

{
  "data": [...],
  "meta": {
    "pagination": {
      "page": 5,
      "limit": 20,
      "total": 543,
      "totalPages": 28
    }
  }
}
```

**Cursor-based (better for large datasets):**
```typescript
GET /posts?cursor=eyJpZCI6MTIzfQ&limit=20

{
  "data": [...],
  "meta": {
    "pagination": {
      "nextCursor": "eyJpZCI6MTQzfQ",
      "prevCursor": "eyJpZCI6MTAzfQ",
      "hasMore": true
    }
  }
}
```

### 5. Filtering & Sorting

```typescript
// Filtering
GET /products?status=active&minPrice=10&maxPrice=100
GET /products?category[in]=electronics,books
GET /products?createdAt[gte]=2024-01-01

// Sorting
GET /products?sort=price         // Ascending
GET /products?sort=-price        // Descending
GET /products?sort=-createdAt,name  // Multiple

// Field selection (reduce payload)
GET /users?fields=id,name,email
```

### 6. Versioning Strategies

**URL versioning (recommended for breaking changes):**
```
/api/v1/users
/api/v2/users
```

**Header versioning (cleaner URLs):**
```
Accept: application/vnd.api+json; version=2
```

**When to version:**
- Breaking changes to response structure
- Removing fields or endpoints
- Changing field types
- NOT for additive changes (new optional fields)

### 7. Authentication Patterns

**Bearer Token (JWT):**
```typescript
headers: {
  'Authorization': 'Bearer eyJhbGciOiJIUzI1NiIs...'
}
```

**API Key (service-to-service):**
```typescript
headers: {
  'X-API-Key': 'sk_live_abc123...'
}
```

**Security Checklist:**
- [ ] Auth on every endpoint (except public)
- [ ] Rate limiting per user/IP
- [ ] Request validation before processing
- [ ] No sensitive data in URLs (use headers/body)
- [ ] HTTPS only

### 8. Error Handling

**Consistent Error Schema:**
```typescript
interface ApiError {
  error: {
    code: string           // Machine-readable (VALIDATION_ERROR)
    message: string        // Human-readable
    details?: ErrorDetail[] // Field-level errors
    requestId?: string     // For debugging
  }
}

interface ErrorDetail {
  field: string
  message: string
  code?: string
}
```

**Implementation:**
```typescript
// Zod validation errors → API errors
function formatZodError(error: ZodError): ApiError {
  return {
    error: {
      code: 'VALIDATION_ERROR',
      message: 'Invalid request data',
      details: error.errors.map(e => ({
        field: e.path.join('.'),
        message: e.message,
      }))
    }
  }
}
```

### 9. Next.js Route Handler Patterns

```typescript
// app/api/products/route.ts
import { z } from 'zod'
import { NextRequest, NextResponse } from 'next/server'

const CreateProductSchema = z.object({
  name: z.string().min(1).max(200),
  price: z.number().positive(),
})

export async function POST(request: NextRequest) {
  try {
    const body = await request.json()
    const validated = CreateProductSchema.parse(body)
    
    const product = await db.product.create({ data: validated })
    
    return NextResponse.json(
      { data: product },
      { status: 201 }
    )
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        formatZodError(error),
        { status: 400 }
      )
    }
    return NextResponse.json(
      { error: { code: 'INTERNAL_ERROR', message: 'Something went wrong' } },
      { status: 500 }
    )
  }
}
```

### 10. Server Actions vs Route Handlers (Next.js 15+)

| Use Case | Recommendation |
|----------|----------------|
| Internal mutations | Server Actions |
| Webhooks from external services | Route Handlers |
| Third-party API proxy | Route Handlers |
| File uploads | Route Handlers (or Server Actions with `unstable_after`) |
| Streaming responses | Route Handlers |
| Form submissions | Server Actions |
| Background tasks | Server Actions + `after()` from next/server |

**Server Action Example (Next.js 15):**
```tsx
// app/actions.ts
'use server'

import { revalidatePath } from 'next/cache'
import { after } from 'next/server'

export async function createProduct(formData: FormData) {
  const product = await db.product.create({
    data: { name: formData.get('name') }
  })
  
  // Run after response (Next.js 15)
  after(async () => {
    await sendNotification(product.id)
  })
  
  revalidatePath('/products')
  return { success: true, data: product }
}
```

## API Review Checklist

- [ ] URLs follow RESTful conventions
- [ ] HTTP methods used correctly
- [ ] Status codes are appropriate
- [ ] Request/response schemas defined (Zod)
- [ ] Error responses consistent
- [ ] Pagination implemented for lists
- [ ] Authentication required where needed
- [ ] Rate limiting in place
- [ ] Input validation on all endpoints
- [ ] No sensitive data in URLs
- [ ] Versioning strategy defined
- [ ] OpenAPI/Swagger docs generated

## API Documentation Template

```yaml
# openapi.yaml
openapi: 3.0.0
info:
  title: Product API
  version: 1.0.0

paths:
  /products:
    get:
      summary: List products
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
      responses:
        200:
          description: Success
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ProductList'
```
