---
name: error-handling
description: Comprehensive error handling patterns for Splits Network services and apps
alwaysApply: false
applyTo:
    - "services/**/*.ts"
    - "apps/**/*.ts"
    - "apps/**/*.tsx"
    - "packages/**/*.ts"
---

# Error Handling Skill

This skill provides guidance for consistent, user-friendly error handling across Splits Network.

## Purpose

Help developers implement robust error handling:

- **HTTP Status Codes**: Correct status codes for API responses
- **Error Response Format**: Standardized error structure
- **Error Classes**: Custom error types for different scenarios
- **Frontend Error Handling**: User-friendly error messages
- **Logging**: Error logging best practices

## When to Use This Skill

Use this skill when:

- Implementing API error responses
- Creating custom error classes
- Handling errors in frontend components
- Logging errors for debugging
- Displaying error messages to users

## Core Principles

### 1. HTTP Status Codes

Use correct HTTP status codes for API responses:

```typescript
// 400 Bad Request - Client error (validation, malformed request)
if (!isValidEmail(email)) {
  return reply.code(400).send({
    error: {
      code: 'VALIDATION_ERROR',
      message: 'Invalid email format',
      details: { field: 'email' }
    }
  });
}

// 401 Unauthorized - Missing or invalid authentication
if (!request.headers['x-clerk-user-id']) {
  return reply.code(401).send({
    error: {
      code: 'UNAUTHORIZED',
      message: 'Authentication required'
    }
  });
}

// 403 Forbidden - Valid auth but insufficient permissions
if (!canAccessResource(userId, resourceId)) {
  return reply.code(403).send({
    error: {
      code: 'FORBIDDEN',
      message: 'You do not have permission to access this resource'
    }
  });
}

// 404 Not Found - Resource doesn't exist
const job = await repository.getById(id);
if (!job) {
  return reply.code(404).send({
    error: {
      code: 'NOT_FOUND',
      message: 'Job not found'
    }
  });
}

// 409 Conflict - Resource state conflict
const existing = await repository.findByEmail(email);
if (existing) {
  return reply.code(409).send({
    error: {
      code: 'CONFLICT',
      message: 'User with this email already exists'
    }
  });
}

// 422 Unprocessable Entity - Semantic validation error
if (application.stage === 'closed') {
  return reply.code(422).send({
    error: {
      code: 'INVALID_STATE',
      message: 'Cannot update closed application'
    }
  });
}

// 429 Too Many Requests - Rate limit exceeded
if (rateLimitExceeded) {
  return reply.code(429).send({
    error: {
      code: 'RATE_LIMIT_EXCEEDED',
      message: 'Too many requests, please try again later',
      retryAfter: 60
    }
  });
}

// 500 Internal Server Error - Unexpected server error
catch (error) {
  console.error('Unexpected error:', error);
  return reply.code(500).send({
    error: {
      code: 'INTERNAL_SERVER_ERROR',
      message: 'An unexpected error occurred'
    }
  });
}

// 503 Service Unavailable - External dependency failure
if (!canConnectToDatabase) {
  return reply.code(503).send({
    error: {
      code: 'SERVICE_UNAVAILABLE',
      message: 'Database unavailable, please try again later'
    }
  });
}
```

See [references/http-status-codes.md](./references/http-status-codes.md).

### 2. Error Response Format

All error responses follow standard envelope:

```typescript
{
  "error": {
    "code": "ERROR_CODE",           // Machine-readable error code
    "message": "User-friendly message", // Human-readable description
    "details"?: { ... },             // Optional additional context
    "retryAfter"?: number            // Optional retry delay (seconds)
  }
}
```

**Examples**:

```typescript
// Validation error with field details
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed",
    "details": {
      "fields": {
        "email": "Invalid email format",
        "phone": "Phone number is required"
      }
    }
  }
}

// Not found error
{
  "error": {
    "code": "NOT_FOUND",
    "message": "Candidate not found"
  }
}

// Permission error
{
  "error": {
    "code": "FORBIDDEN",
    "message": "You do not have permission to delete this job"
  }
}

// Rate limit error
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many API requests",
    "retryAfter": 60
  }
}
```

See [examples/error-responses.ts](./examples/error-responses.ts).

### 3. Custom Error Classes

Create typed error classes for different scenarios:

```typescript
// Base application error
export class AppError extends Error {
    constructor(
        public code: string,
        message: string,
        public statusCode: number = 500,
        public details?: any,
    ) {
        super(message);
        this.name = "AppError";
    }
}

// Validation errors (400)
export class ValidationError extends AppError {
    constructor(message: string, details?: any) {
        super("VALIDATION_ERROR", message, 400, details);
        this.name = "ValidationError";
    }
}

// Not found errors (404)
export class NotFoundError extends AppError {
    constructor(resource: string) {
        super("NOT_FOUND", `${resource} not found`, 404);
        this.name = "NotFoundError";
    }
}

// Permission errors (403)
export class ForbiddenError extends AppError {
    constructor(message: string = "Access denied") {
        super("FORBIDDEN", message, 403);
        this.name = "ForbiddenError";
    }
}

// Conflict errors (409)
export class ConflictError extends AppError {
    constructor(message: string) {
        super("CONFLICT", message, 409);
        this.name = "ConflictError";
    }
}

// State errors (422)
export class InvalidStateError extends AppError {
    constructor(message: string) {
        super("INVALID_STATE", message, 422);
        this.name = "InvalidStateError";
    }
}

// Usage
throw new NotFoundError("Job");
throw new ValidationError("Invalid email", { field: "email" });
throw new ForbiddenError("Only recruiters can submit candidates");
throw new ConflictError("Application already exists");
throw new InvalidStateError("Cannot reopen closed job");
```

See [examples/error-classes.ts](./examples/error-classes.ts).

### 4. Error Handler Middleware

Fastify error handler catches all errors:

```typescript
// services/ats-service/src/index.ts
app.setErrorHandler((error, request, reply) => {
    // Log error with context
    console.error("Error handling request:", {
        method: request.method,
        url: request.url,
        error: error.message,
        stack: error.stack,
        userId: request.headers["x-clerk-user-id"],
    });

    // Handle custom AppError
    if (error instanceof AppError) {
        return reply.code(error.statusCode).send({
            error: {
                code: error.code,
                message: error.message,
                details: error.details,
            },
        });
    }

    // Handle Fastify validation errors
    if (error.validation) {
        return reply.code(400).send({
            error: {
                code: "VALIDATION_ERROR",
                message: "Request validation failed",
                details: error.validation,
            },
        });
    }

    // Handle Supabase errors
    if (error.code?.startsWith("PGRST")) {
        return reply.code(500).send({
            error: {
                code: "DATABASE_ERROR",
                message: "Database operation failed",
            },
        });
    }

    // Fallback to 500 for unexpected errors
    return reply.code(500).send({
        error: {
            code: "INTERNAL_SERVER_ERROR",
            message: "An unexpected error occurred",
        },
    });
});
```

See [examples/error-middleware.ts](./examples/error-middleware.ts).

### 5. Frontend Error Handling

Handle API errors gracefully in frontend:

```typescript
'use client';

import { useState } from 'react';
import { apiClient } from '@/lib/api-client';

export default function JobForm() {
  const [error, setError] = useState<string | null>(null);
  const [fieldErrors, setFieldErrors] = useState<Record<string, string>>({});
  const [submitting, setSubmitting] = useState(false);

  async function handleSubmit(data: any) {
    setError(null);
    setFieldErrors({});
    setSubmitting(true);

    try {
      await apiClient.post('/jobs', data);
      // Success handling...
    } catch (err: any) {
      // Network error
      if (!err.response) {
        setError('Network error. Please check your connection.');
        return;
      }

      const { error } = err.response.data;

      // Validation error with field details
      if (error.code === 'VALIDATION_ERROR' && error.details?.fields) {
        setFieldErrors(error.details.fields);
        setError('Please fix the validation errors below.');
      }
      // Permission error
      else if (error.code === 'FORBIDDEN') {
        setError('You do not have permission to create jobs.');
      }
      // Generic error
      else {
        setError(error.message || 'An unexpected error occurred.');
      }
    } finally {
      setSubmitting(false);
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      {/* Global error alert */}
      {error && (
        <div className="alert alert-error mb-4">
          <i className="fa-duotone fa-regular fa-circle-exclamation"></i>
          <span>{error}</span>
        </div>
      )}

      {/* Field with error */}
      <fieldset className="fieldset">
        <legend className="fieldset-legend">Job Title *</legend>
        <input
          type="text"
          className={`input w-full ${fieldErrors.title ? 'input-error' : ''}`}
          name="title"
        />
        {fieldErrors.title && (
          <p className="fieldset-label text-error">{fieldErrors.title}</p>
        )}
      </fieldset>

      <button type="submit" className="btn btn-primary" disabled={submitting}>
        {submitting ? 'Creating...' : 'Create Job'}
      </button>
    </form>
  );
}
```

See [examples/frontend-error-handling.tsx](./examples/frontend-error-handling.tsx).

### 6. Error Logging

Log errors with context for debugging:

```typescript
// Backend error logging
function logError(
    error: Error,
    context: {
        service: string;
        method: string;
        userId?: string;
        resourceId?: string;
    },
): void {
    console.error("Error:", {
        service: context.service,
        method: context.method,
        userId: context.userId,
        resourceId: context.resourceId,
        error: {
            name: error.name,
            message: error.message,
            stack: error.stack,
        },
        timestamp: new Date().toISOString(),
    });
}

// Usage
try {
    await repository.update(id, data);
} catch (error) {
    logError(error as Error, {
        service: "ats-service",
        method: "JobRepository.update",
        userId: clerkUserId,
        resourceId: id,
    });
    throw error;
}
```

**Logging Rules**:

- ✅ Log all 500 errors with full context
- ✅ Include user ID and resource ID
- ✅ Include timestamp
- ✅ Include stack trace
- ⚠️ Log 400-level errors at info/warn level (not error)
- ❌ Don't log sensitive data (passwords, tokens)

See [examples/error-logging.ts](./examples/error-logging.ts).

### 7. Async Error Handling

Handle errors in async operations:

```typescript
// Try-catch for async functions
async function fetchJob(id: string): Promise<Job> {
    try {
        const { data, error } = await supabase
            .from("jobs")
            .select("*")
            .eq("id", id)
            .single();

        if (error) throw error;
        if (!data) throw new NotFoundError("Job");

        return data;
    } catch (error) {
        // Log error
        console.error("Failed to fetch job:", error);
        throw error; // Re-throw for caller to handle
    }
}

// Promise.allSettled for parallel operations
async function fetchMultipleJobs(ids: string[]): Promise<Job[]> {
    const results = await Promise.allSettled(ids.map((id) => fetchJob(id)));

    const jobs = results
        .filter(
            (r): r is PromiseFulfilledResult<Job> => r.status === "fulfilled",
        )
        .map((r) => r.value);

    const errors = results
        .filter((r): r is PromiseRejectedResult => r.status === "rejected")
        .map((r) => r.reason);

    if (errors.length > 0) {
        console.warn(`Failed to fetch ${errors.length} jobs:`, errors);
    }

    return jobs;
}
```

See [examples/async-error-handling.ts](./examples/async-error-handling.ts).

### 8. Database Error Handling

Handle Supabase/PostgreSQL errors:

```typescript
async function createJob(data: JobCreate): Promise<Job> {
    try {
        const { data: job, error } = await supabase
            .from("jobs")
            .insert(data)
            .select()
            .single();

        if (error) {
            // Handle specific error codes
            switch (error.code) {
                case "23505": // Unique constraint violation
                    throw new ConflictError(
                        "Job with this title already exists",
                    );

                case "23503": // Foreign key violation
                    throw new ValidationError("Invalid company ID");

                case "23502": // Not null violation
                    throw new ValidationError("Missing required field");

                case "PGRST116": // Not found
                    throw new NotFoundError("Job");

                default:
                    console.error("Database error:", error);
                    throw new AppError(
                        "DATABASE_ERROR",
                        "Database operation failed",
                    );
            }
        }

        return job;
    } catch (error) {
        if (error instanceof AppError) throw error;

        console.error("Unexpected database error:", error);
        throw new AppError("DATABASE_ERROR", "Database operation failed");
    }
}
```

See [examples/database-error-handling.ts](./examples/database-error-handling.ts) and [references/supabase-error-codes.md](./references/supabase-error-codes.md).

## Error Code Catalog

### Client Errors (4xx)

- `VALIDATION_ERROR` (400) - Request validation failed
- `UNAUTHORIZED` (401) - Authentication required
- `FORBIDDEN` (403) - Insufficient permissions
- `NOT_FOUND` (404) - Resource not found
- `CONFLICT` (409) - Resource state conflict
- `INVALID_STATE` (422) - Invalid resource state
- `RATE_LIMIT_EXCEEDED` (429) - Rate limit exceeded

### Server Errors (5xx)

- `INTERNAL_SERVER_ERROR` (500) - Unexpected server error
- `DATABASE_ERROR` (500) - Database operation failed
- `SERVICE_UNAVAILABLE` (503) - External service unavailable

See [references/error-codes.md](./references/error-codes.md).

## Testing Error Handling

Test error scenarios:

```typescript
describe("JobRepository", () => {
    it("should throw NotFoundError for non-existent job", async () => {
        mockSupabase.single.mockResolvedValue({ data: null, error: null });

        await expect(repository.getById("999")).rejects.toThrow(NotFoundError);
    });

    it("should throw ConflictError for duplicate job", async () => {
        mockSupabase.insert.mockResolvedValue({
            data: null,
            error: { code: "23505" },
        });

        await expect(repository.create(jobData)).rejects.toThrow(ConflictError);
    });

    it("should return 404 for non-existent job", async () => {
        const response = await app.inject({
            method: "GET",
            url: "/api/v2/jobs/non-existent-id",
        });

        expect(response.statusCode).toBe(404);
        expect(JSON.parse(response.body)).toMatchObject({
            error: {
                code: "NOT_FOUND",
                message: expect.any(String),
            },
        });
    });
});
```

See [examples/error-testing.ts](./examples/error-testing.ts).

## Anti-Patterns to Avoid

### ❌ Swallowing Errors

```typescript
// WRONG - Silent failure
try {
    await saveData();
} catch (error) {
    // Do nothing - error is lost!
}

// CORRECT - Log and handle
try {
    await saveData();
} catch (error) {
    console.error("Failed to save data:", error);
    throw error; // Or handle appropriately
}
```

### ❌ Generic Error Messages

```typescript
// WRONG - Unhelpful
throw new Error("Something went wrong");

// CORRECT - Specific
throw new ValidationError("Email format is invalid");
```

### ❌ Exposing Stack Traces to Users

```typescript
// WRONG - Security risk
return reply.code(500).send({
    error: error.stack, // Exposes internal details!
});

// CORRECT - Generic message
return reply.code(500).send({
    error: {
        code: "INTERNAL_SERVER_ERROR",
        message: "An unexpected error occurred",
    },
});
```

### ❌ Not Using Status Codes

```typescript
// WRONG - Always 200
return reply.send({
    success: false,
    error: "Not found",
});

// CORRECT - Use status code
return reply.code(404).send({
    error: {
        code: "NOT_FOUND",
        message: "Job not found",
    },
});
```

## References

- [Error Classes](./examples/error-classes.ts)
- [Error Responses](./examples/error-responses.ts)
- [Error Middleware](./examples/error-middleware.ts)
- [Frontend Error Handling](./examples/frontend-error-handling.tsx)
- [Database Error Handling](./examples/database-error-handling.ts)
- [Async Error Handling](./examples/async-error-handling.ts)
- [Error Testing](./examples/error-testing.ts)
- [HTTP Status Codes](./references/http-status-codes.md)
- [Error Codes Catalog](./references/error-codes.md)
- [Supabase Error Codes](./references/supabase-error-codes.md)

## Related Skills

- `api-specifications` - API response format standards
- `database-patterns` - Database error handling
- `testing-patterns` - Testing error scenarios
