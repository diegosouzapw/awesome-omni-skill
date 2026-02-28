---
name: testing-patterns
description: Testing patterns and best practices for Splits Network services and apps
alwaysApply: false
applyTo:
    - "services/**/tests/**"
    - "services/**/*.test.ts"
    - "services/**/*.spec.ts"
    - "apps/**/tests/**"
    - "apps/**/__tests__/**"
---

# Testing Patterns Skill

This skill provides guidance for writing comprehensive tests in Splits Network.

## Purpose

Help developers write maintainable, reliable tests following Splits Network standards:

- **Unit Tests**: Repository and service layer testing
- **Integration Tests**: API endpoint testing
- **Mocking**: Supabase and external service mocks
- **Test Data**: Fixture management
- **Coverage**: What and how to test

## When to Use This Skill

Use this skill when:

- Writing unit tests for repositories or services
- Writing integration tests for API endpoints
- Mocking Supabase or external APIs
- Creating test fixtures
- Debugging test failures

## Core Testing Principles

### 1. Repository Unit Tests

Test repository methods with mocked Supabase client:

```typescript
import { describe, it, expect, vi, beforeEach } from "vitest";
import { JobRepository } from "../repository";

describe("JobRepository", () => {
    let mockSupabase: any;
    let repository: JobRepository;

    beforeEach(() => {
        // Mock Supabase client
        mockSupabase = {
            from: vi.fn().mockReturnThis(),
            select: vi.fn().mockReturnThis(),
            eq: vi.fn().mockReturnThis(),
            in: vi.fn().mockReturnThis(),
            single: vi.fn(),
        };

        repository = new JobRepository(mockSupabase);
    });

    it("should fetch job by ID", async () => {
        const mockJob = { id: "123", title: "Engineer" };
        mockSupabase.single.mockResolvedValue({ data: mockJob, error: null });

        const result = await repository.getById("123");

        expect(result).toEqual(mockJob);
        expect(mockSupabase.from).toHaveBeenCalledWith("jobs");
        expect(mockSupabase.eq).toHaveBeenCalledWith("id", "123");
    });

    it("should throw error when job not found", async () => {
        mockSupabase.single.mockResolvedValue({
            data: null,
            error: { code: "PGRST116" },
        });

        await expect(repository.getById("999")).rejects.toThrow(
            "Job not found",
        );
    });
});
```

See [examples/repository-test.spec.ts](./examples/repository-test.spec.ts).

### 2. Service Layer Tests

Test business logic with mocked repository:

```typescript
import { describe, it, expect, vi } from "vitest";
import { JobServiceV2 } from "../service";

describe("JobServiceV2", () => {
    let mockRepository: any;
    let mockEventPublisher: any;
    let service: JobServiceV2;

    beforeEach(() => {
        mockRepository = {
            create: vi.fn(),
            getById: vi.fn(),
            update: vi.fn(),
        };

        mockEventPublisher = {
            publish: vi.fn(),
        };

        service = new JobServiceV2(mockRepository, mockEventPublisher);
    });

    it("should create job and publish event", async () => {
        const jobData = { title: "Engineer", company_id: "123" };
        const createdJob = { id: "456", ...jobData };

        mockRepository.create.mockResolvedValue(createdJob);

        const result = await service.create("clerk_123", jobData);

        expect(result).toEqual(createdJob);
        expect(mockRepository.create).toHaveBeenCalledWith(
            "clerk_123",
            jobData,
        );
        expect(mockEventPublisher.publish).toHaveBeenCalledWith(
            "job.created",
            expect.objectContaining({ jobId: "456" }),
        );
    });

    it("should validate job data before creation", async () => {
        const invalidData = { title: "" }; // Missing required fields

        await expect(service.create("clerk_123", invalidData)).rejects.toThrow(
            "Validation error",
        );

        expect(mockRepository.create).not.toHaveBeenCalled();
    });
});
```

See [examples/service-test.spec.ts](./examples/service-test.spec.ts).

### 3. API Integration Tests

Test complete request/response cycle:

```typescript
import { describe, it, expect, beforeAll, afterAll } from "vitest";
import { buildServer } from "../index";
import { FastifyInstance } from "fastify";

describe("Jobs API", () => {
    let app: FastifyInstance;

    beforeAll(async () => {
        app = await buildServer();
        await app.ready();
    });

    afterAll(async () => {
        await app.close();
    });

    it("should list jobs with pagination", async () => {
        const response = await app.inject({
            method: "GET",
            url: "/api/v2/jobs?page=1&limit=25",
            headers: {
                "x-clerk-user-id": "test_user_123",
            },
        });

        expect(response.statusCode).toBe(200);
        const body = JSON.parse(response.body);
        expect(body).toHaveProperty("data");
        expect(body).toHaveProperty("pagination");
        expect(Array.isArray(body.data)).toBe(true);
    });

    it("should return 404 for non-existent job", async () => {
        const response = await app.inject({
            method: "GET",
            url: "/api/v2/jobs/non-existent-id",
            headers: {
                "x-clerk-user-id": "test_user_123",
            },
        });

        expect(response.statusCode).toBe(404);
        const body = JSON.parse(response.body);
        expect(body).toHaveProperty("error");
    });

    it("should create job with valid data", async () => {
        const jobData = {
            title: "Senior Engineer",
            company_id: "company_123",
            location: "San Francisco",
        };

        const response = await app.inject({
            method: "POST",
            url: "/api/v2/jobs",
            headers: {
                "x-clerk-user-id": "test_user_123",
            },
            payload: jobData,
        });

        expect(response.statusCode).toBe(201);
        const body = JSON.parse(response.body);
        expect(body.data).toMatchObject(jobData);
    });
});
```

See [examples/api-integration-test.spec.ts](./examples/api-integration-test.spec.ts).

### 4. Test Fixtures

Create reusable test data:

```typescript
// tests/fixtures/jobs.ts
export const mockJobs = {
    engineer: {
        id: "job_123",
        title: "Senior Software Engineer",
        company_id: "company_123",
        status: "active",
        location: "San Francisco, CA",
        created_at: "2026-01-01T00:00:00Z",
    },

    designer: {
        id: "job_456",
        title: "Product Designer",
        company_id: "company_123",
        status: "active",
        location: "Remote",
        created_at: "2026-01-02T00:00:00Z",
    },
};

export const mockApplications = {
    pending: {
        id: "app_123",
        candidate_id: "candidate_123",
        job_id: "job_123",
        stage: "screen",
        created_at: "2026-01-01T00:00:00Z",
    },
};
```

See [examples/test-fixtures.ts](./examples/test-fixtures.ts).

### 5. Mocking External Services

Mock Clerk, Stripe, Resend, etc.:

```typescript
// Mock Clerk auth
vi.mock("@clerk/nextjs", () => ({
    auth: () => ({
        userId: "test_user_123",
        sessionId: "test_session",
    }),
    currentUser: () => ({
        id: "test_user_123",
        emailAddresses: [{ emailAddress: "test@example.com" }],
    }),
}));

// Mock Stripe
vi.mock("stripe", () => ({
    default: vi.fn().mockImplementation(() => ({
        subscriptions: {
            create: vi.fn().mockResolvedValue({ id: "sub_123" }),
            retrieve: vi.fn().mockResolvedValue({ status: "active" }),
        },
    })),
}));

// Mock Resend
vi.mock("resend", () => ({
    Resend: vi.fn().mockImplementation(() => ({
        emails: {
            send: vi.fn().mockResolvedValue({ id: "email_123" }),
        },
    })),
}));
```

See [examples/external-service-mocks.ts](./examples/external-service-mocks.ts).

## Test Organization

### File Structure

```
services/ats-service/
├── src/
│   └── v2/
│       └── jobs/
│           ├── repository.ts
│           ├── service.ts
│           └── __tests__/
│               ├── repository.spec.ts
│               ├── service.spec.ts
│               └── integration.spec.ts
└── tests/
    ├── fixtures/
    │   ├── jobs.ts
    │   └── applications.ts
    └── helpers/
        └── test-utils.ts
```

### Naming Conventions

- **Unit tests**: `repository.spec.ts`, `service.spec.ts`
- **Integration tests**: `integration.spec.ts`, `api.spec.ts`
- **Test fixtures**: `fixtures/*.ts`
- **Test helpers**: `helpers/*.ts`

See [references/test-organization.md](./references/test-organization.md).

## Coverage Guidelines

### What to Test

✅ **Always Test**:

- Repository methods (all CRUD operations)
- Service business logic
- Validation rules
- Error handling
- Access control logic
- Event publishing

⚠️ **Consider Testing**:

- Complex utility functions
- Data transformations
- State machines

❌ **Don't Test**:

- Third-party library internals
- Simple getters/setters
- Type definitions

### Coverage Targets

- **Repository Layer**: 90%+ coverage
- **Service Layer**: 85%+ coverage
- **API Routes**: 80%+ coverage
- **Overall**: 80%+ coverage

See [references/coverage-guidelines.md](./references/coverage-guidelines.md).

## Test Utilities

### Custom Matchers

```typescript
// tests/helpers/matchers.ts
expect.extend({
    toBeUUID(received: string) {
        const uuidRegex =
            /^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$/i;
        const pass = uuidRegex.test(received);
        return {
            pass,
            message: () => `expected ${received} to be a valid UUID`,
        };
    },
});

// Usage
expect(job.id).toBeUUID();
```

### Test Helpers

```typescript
// tests/helpers/test-utils.ts
export function createMockSupabase() {
    return {
        from: vi.fn().mockReturnThis(),
        select: vi.fn().mockReturnThis(),
        eq: vi.fn().mockReturnThis(),
        single: vi.fn(),
    };
}

export function createMockAccessContext(role: string) {
    return {
        userId: "user_123",
        role,
        accessibleCompanyIds: ["company_123"],
        isCompanyUser: role !== "recruiter",
    };
}
```

See [examples/test-utilities.ts](./examples/test-utilities.ts).

## Running Tests

```bash
# Run all tests
pnpm test

# Run tests in watch mode
pnpm test:watch

# Run tests with coverage
pnpm test:coverage

# Run specific test file
pnpm test repository.spec.ts

# Run tests matching pattern
pnpm test --grep "JobRepository"
```

See [references/running-tests.md](./references/running-tests.md).

## Common Testing Patterns

### Testing Async Functions

```typescript
it("should handle async operations", async () => {
    const result = await service.create(data);
    expect(result).toBeDefined();
});
```

### Testing Error Cases

```typescript
it("should throw error for invalid data", async () => {
    await expect(service.create(invalidData)).rejects.toThrow(
        "Validation error",
    );
});
```

### Testing Event Publishing

```typescript
it("should publish event after creation", async () => {
    await service.create(data);

    expect(mockEventPublisher.publish).toHaveBeenCalledWith(
        "job.created",
        expect.objectContaining({ jobId: expect.any(String) }),
    );
});
```

See [examples/common-test-patterns.ts](./examples/common-test-patterns.ts).

## Anti-Patterns to Avoid

### ❌ Testing Implementation Details

```typescript
// WRONG - Tests internal implementation
it("should call private method", () => {
    expect(service["privateMethod"]).toHaveBeenCalled();
});

// CORRECT - Tests public behavior
it("should return processed data", () => {
    expect(service.process(data)).toEqual(expected);
});
```

### ❌ Brittle Tests

```typescript
// WRONG - Too specific, breaks easily
expect(result).toEqual({
    id: "123",
    created_at: "2026-01-13T10:00:00.000Z", // Exact timestamp
    title: "Engineer",
});

// CORRECT - Flexible assertions
expect(result).toMatchObject({
    id: expect.any(String),
    created_at: expect.any(String),
    title: "Engineer",
});
```

### ❌ Not Cleaning Up

```typescript
// WRONG - State leaks between tests
let sharedState = {};

it("test 1", () => {
    sharedState.value = 1;
});

it("test 2", () => {
    // Fails if test 1 didn't run
    expect(sharedState.value).toBe(1);
});

// CORRECT - Clean state for each test
beforeEach(() => {
    sharedState = {};
});
```

## References

- [Repository Test Example](./examples/repository-test.spec.ts)
- [Service Test Example](./examples/service-test.spec.ts)
- [API Integration Test](./examples/api-integration-test.spec.ts)
- [Test Fixtures](./examples/test-fixtures.ts)
- [Test Utilities](./examples/test-utilities.ts)
- [Coverage Guidelines](./references/coverage-guidelines.md)
- [Running Tests Guide](./references/running-tests.md)

## Related Skills

- `database-patterns` - Repository layer patterns
- `api-specifications` - API endpoint patterns
- `error-handling` - Error handling to test
