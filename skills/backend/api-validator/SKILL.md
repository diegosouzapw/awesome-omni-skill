---
name: api-validator
description: Validate REST API standards compliance (versioning, naming, HTTP methods, status codes, pagination, Swagger). Use when checking endpoints before deployment, reviewing API design, or ensuring documentation completeness (e.g., "Validate User API", "Check Product endpoints").
allowed-tools: Read, Glob, Grep
---

# API Standards Validator

Validate REST API design standards compliance including versioning, naming conventions, HTTP methods, status codes, pagination, and Swagger documentation.

## What This Skill Does

Performs comprehensive API standards validation:

- **Versioning**: Checks `/v1/` prefix on all routes
- **Resource Naming**: Validates plural nouns, lowercase-with-hyphens
- **HTTP Methods**: Validates proper method usage and status codes
- **Pagination**: Ensures list endpoints support pagination
- **Swagger Docs**: Verifies complete OpenAPI documentation
- **Error Mapping**: Checks proper status code usage

## When to Use This Skill

Use when you need to:
- Validate API endpoints before deployment
- Check new endpoints for standards compliance
- Review API design during code review
- Ensure Swagger documentation completeness

Examples:
- "Validate the User API endpoints"
- "Check if Product API follows design standards"
- "Review Order API for compliance"

## API Design Standards

### Versioning Rules

All controllers MUST be prefixed with `/v1/`:

```typescript
// ✅ CORRECT
@JsonController('/v1/users')
export class UserController {}

@JsonController('/v1/sms-messages')
export class SmsMessageController {}

// ❌ WRONG
@JsonController('/users')  // Missing version
@JsonController('/api/users')  // Wrong format
```

### Resource Naming Rules

**Must be:**
- Plural nouns: `/v1/users` not `/v1/user`
- Lowercase with hyphens: `/v1/sms-messages`
- No verbs: `/v1/users` not `/v1/getUsers`

```typescript
// ✅ CORRECT
'/v1/users'
'/v1/products'
'/v1/sms-messages'
'/v1/order-items'

// ❌ WRONG
'/v1/user'          // Singular
'/v1/getUsers'      // Verb
'/v1/Users'         // Wrong casing
'/v1/sms_messages'  // Underscores
```

### HTTP Methods and Status Codes

**Expected patterns:**

- **POST** (Create) → 201 Created
- **GET** (Read) → 200 OK
- **PATCH** (Partial Update) → 200 OK
- **PUT** (Full Update) → 200 OK
- **DELETE** → 204 No Content

**Error status codes:**

- **400** Bad Request → Validation errors
- **401** Unauthorized → Authentication required
- **403** Forbidden → Permission denied
- **404** Not Found → Resource not found
- **409** Conflict → Duplicate resource
- **422** Unprocessable Entity → Business rule violation
- **500** Internal Server Error → Unexpected errors

### Pagination Requirements

List endpoints MUST support pagination with DTOs:

```typescript
// ✅ CORRECT
export class QueryEntityDto {
  @IsOptional()
  @Type(() => Number)
  @IsNumber()
  @Min(1)
  limit?: number = 20;

  @IsOptional()
  @Type(() => Number)
  @IsNumber()
  @Min(0)
  offset?: number = 0;
  sortBy: z.enum(['createdAt', 'name']).optional(),
  order: z.enum(['asc', 'desc']).default('desc'),
});

// Response format
{
  items: Entity[],
  total: number,
  limit: number,
  offset: number
}
```

### Swagger Documentation Requirements

All endpoints MUST have:

```typescript
.post('/', controller.create.bind(controller), {
  body: CreateSchema,
  detail: {
    summary: 'Create entity',          // Required
    description: 'Detailed description', // Recommended
    tags: ['Entities'],                // Required
    responses: {                        // Required
      201: { description: 'Created' },
      400: { description: 'Invalid input' },
      409: { description: 'Duplicate' },
    },
  },
})
```

## Validation Checks

### Versioning Validation

**Checks:**
- All route prefixes include `/v1/`
- No routes without versioning
- Consistent version across all endpoints

**Reports:**
```
CRITICAL: Missing API version
File: src/contexts/user/presentation/user.routes.ts:5
Issue: Route prefix '/users' lacks /v1/
Fix: Change to '/v1/users'
```

### Resource Naming Validation

**Checks:**
- Resources are plural nouns
- Lowercase with hyphens (not underscores or camelCase)
- No verbs in resource names
- Proper REST conventions

**Reports:**
```
CRITICAL: Singular resource name
File: src/contexts/user/presentation/user.routes.ts:5
Issue: Resource '/v1/user' is singular
Fix: Change to '/v1/users'

CRITICAL: Verb in resource name
File: src/contexts/user/presentation/user.routes.ts:10
Issue: Resource '/v1/getUsers' contains verb
Fix: Use '/v1/users' with GET method
```

### HTTP Method Validation

**Checks:**
- GET for reads (200 OK)
- POST for creates (201 Created)
- PATCH/PUT for updates (200 OK)
- DELETE for deletions (204 No Content)
- Swagger responses match expected codes

**Reports:**
```
CRITICAL: Wrong status code for POST
File: src/contexts/user/presentation/user.routes.ts:15
Issue: POST endpoint documents 200, should be 201
Fix: Change response code to 201 in Swagger detail
```

### Error Mapping Validation

**Checks:**
- Controllers map domain errors to HTTP exceptions
- Correct status codes used:
  - EntityNotFoundError → 404
  - DuplicateError → 409
  - ValidationError → 400
  - UnauthorizedError → 401
  - ForbiddenError → 403
  - BusinessRuleViolation → 422

**Reports:**
```
WARNING: Missing error mapping
File: src/contexts/user/presentation/user.controller.ts:20
Issue: EntityNotFoundError not mapped to HttpException
Fix: Add error mapping:
  if (error instanceof EntityNotFoundError) {
    throw new HttpException(404, error.message, error.code);
  }
```

### Pagination Validation

**Checks:**
- List endpoints have `limit` parameter
- List endpoints have `offset` parameter
- Defaults defined for pagination
- Response includes total count

**Reports:**
```
CRITICAL: Missing pagination
File: src/contexts/user/presentation/schemas/query-user.schema.ts
Issue: Query schema lacks limit and offset
Fix: Add pagination fields:
  limit: z.coerce.number().min(1).max(100).default(10),
  offset: z.coerce.number().min(0).default(0),
```

### Swagger Documentation Validation

**Checks:**
- All endpoints have `detail` object
- `summary` field present
- `tags` array present
- `responses` object with status codes
- Response descriptions provided

**Reports:**
```
WARNING: Incomplete Swagger documentation
File: src/contexts/user/presentation/user.routes.ts:15
Issue: POST endpoint missing 'responses' in detail
Fix: Add responses object:
  responses: {
    201: { description: 'User created' },
    400: { description: 'Invalid input' },
    409: { description: 'Email exists' },
  }
```

### Controller Binding Validation

**Checks:**
- Controller methods use `.bind(controller)`
- Controller resolved from DI container

**Reports:**
```
CRITICAL: Controller method not bound
File: src/contexts/user/presentation/user.routes.ts:12
Issue: .post('/', controller.create) missing .bind()
Fix: Add .bind(controller): .post('/', controller.create.bind(controller))
```

## Common Violations

### Missing Versioning
```typescript
// ❌ WRONG
new Elysia({ prefix: '/users' })

// ✅ CORRECT
new Elysia({ prefix: '/v1/users' })
```

### Wrong Resource Naming
```typescript
// ❌ WRONG - Singular
new Elysia({ prefix: '/v1/user' })

// ❌ WRONG - Verb
.get('/getUser', ...)

// ✅ CORRECT
new Elysia({ prefix: '/v1/users' })
.get('/:id', ...)
```

### Wrong Status Code
```typescript
// ❌ WRONG
.post('/', ..., {
  detail: {
    responses: {
      200: {} // Should be 201 for POST
    }
  }
})

// ✅ CORRECT
.post('/', ..., {
  detail: {
    responses: {
      201: { description: 'Created' }
    }
  }
})
```

### Missing Pagination
```typescript
// ❌ WRONG
export const QuerySchema = z.object({
  search: z.string().optional(),
});

// ✅ CORRECT
export const QuerySchema = z.object({
  limit: z.coerce.number().min(1).max(100).default(10),
  offset: z.coerce.number().min(0).default(0),
  search: z.string().optional(),
});
```

### Incomplete Swagger
```typescript
// ❌ WRONG
.post('/', controller.create.bind(controller), {
  body: CreateSchema,
})

// ✅ CORRECT
.post('/', controller.create.bind(controller), {
  body: CreateSchema,
  detail: {
    summary: 'Create user',
    tags: ['Users'],
    responses: {
      201: { description: 'User created' },
      400: { description: 'Invalid input' },
    },
  },
})
```

## Report Format

1. **Summary**: Endpoints validated
2. **Endpoint Inventory**: List all found endpoints
3. **Critical Issues**: Must fix immediately
4. **Warnings**: Should fix before deployment
5. **Suggestions**: Nice-to-have improvements
6. **Compliance Score**: Percentage of standards met

## Integration

The validator is read-only - it never modifies code. After review:

1. Fix critical issues (versioning, naming)
2. Address warnings (Swagger docs, pagination)
3. Consider suggestions (additional docs)
4. Re-run validator to verify compliance

## Related Skills

- **ddd-validator**: Validate DDD architecture
- **ddd-api-generator**: Generate compliant APIs
- **di-helper**: Check DI setup