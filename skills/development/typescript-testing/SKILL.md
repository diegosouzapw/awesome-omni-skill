---
name: typescript-testing
description: Comprehensive testing guidance for TypeScript projects including unit testing patterns, mocking strategies, and test organization best practices
version: 1.0.0
priority: 25
tags:
  - testing
  - typescript
  - vitest
  - jest
  - builtin
triggers:
  - type: keyword
    pattern: test
  - type: keyword
    pattern: vitest
  - type: keyword
    pattern: jest
  - type: glob
    pattern: "**/*.test.ts"
  - type: glob
    pattern: "**/*.spec.ts"
globs:
  - "**/*.test.ts"
  - "**/*.spec.ts"
  - "**/vitest.config.ts"
  - "**/jest.config.ts"
---

# TypeScript Testing

Guidance for writing effective tests in TypeScript projects using Vitest or Jest.

## Rules

- **Arrange-Act-Assert**: Structure tests with clear setup, execution, and verification phases
- **Single Assertion Focus**: Each test should verify one specific behavior
- **Descriptive Names**: Use `describe` blocks for grouping and `it`/`test` with clear descriptions
- **Test Isolation**: Tests must not depend on execution order or shared mutable state
- **Mock External Dependencies**: Always mock I/O, network calls, and system interactions
- **Coverage Targets**: Aim for 80%+ line coverage, 70%+ branch coverage
- **No Logic in Tests**: Avoid conditionals and loops in test code
- **Clean Up**: Use `beforeEach`/`afterEach` for setup/teardown, never leave test artifacts

## Patterns

### Test Structure

```typescript
import { describe, it, expect, vi, beforeEach, afterEach } from "vitest";
import { MyService } from "./my-service.js";

describe("MyService", () => {
  let service: MyService;

  beforeEach(() => {
    service = new MyService();
    vi.useFakeTimers();
  });

  afterEach(() => {
    vi.useRealTimers();
    vi.restoreAllMocks();
  });

  describe("methodName", () => {
    it("should return expected result when given valid input", () => {
      // Arrange
      const input = "test";

      // Act
      const result = service.methodName(input);

      // Assert
      expect(result).toBe("expected");
    });

    it("should throw error when given invalid input", () => {
      expect(() => service.methodName(null)).toThrow("Invalid input");
    });
  });
});
```markdown

### Mocking Dependencies

```typescript
import { vi, type Mock } from "vitest";
import type { Database } from "./database.js";

// Factory mock
vi.mock("./database.js", () => ({
  Database: vi.fn().mockImplementation(() => ({
    query: vi.fn(),
    close: vi.fn(),
  })),
}));

// Inline mock with type safety
const mockFetch = vi.fn() as Mock<typeof fetch>;
vi.stubGlobal("fetch", mockFetch);

mockFetch.mockResolvedValue(
  new Response(JSON.stringify({ data: "test" }), {
    status: 200,
    headers: { "Content-Type": "application/json" },
  })
);
```markdown

### Testing Async Code

```typescript
it("should handle async operations", async () => {
  const promise = service.asyncMethod();
  
  // Advance timers if needed
  await vi.runAllTimersAsync();
  
  const result = await promise;
  expect(result).toBeDefined();
});

it("should reject with error on failure", async () => {
  mockDependency.mockRejectedValueOnce(new Error("Network error"));
  
  await expect(service.asyncMethod()).rejects.toThrow("Network error");
});
```markdown

### Snapshot Testing

```typescript
it("should match snapshot for complex output", () => {
  const result = service.generateConfig();
  expect(result).toMatchSnapshot();
});

it("should match inline snapshot", () => {
  const result = service.formatOutput({ key: "value" });
  expect(result).toMatchInlineSnapshot(`"{ key: 'value' }"`);
});
```markdown

## Anti-Patterns

```typescript
// ❌ Testing implementation details
it("should call internal method", () => {
  const spy = vi.spyOn(service, "_privateMethod");
  service.publicMethod();
  expect(spy).toHaveBeenCalled(); // Don't test internals
});

// ❌ Multiple unrelated assertions
it("should work", () => {
  expect(service.method1()).toBe("a");
  expect(service.method2()).toBe("b");
  expect(service.method3()).toBe("c"); // Split into separate tests
});

// ❌ Test interdependence
let sharedState: string;
it("first test", () => { sharedState = "set"; });
it("depends on first", () => { expect(sharedState).toBe("set"); }); // Never do this

// ❌ Not awaiting async code
it("broken async test", () => {
  service.asyncMethod().then(r => expect(r).toBe("x")); // Missing await/return
});

// ❌ Overspecified mocks
mockFn.mockReturnValueOnce("a")
      .mockReturnValueOnce("b")
      .mockReturnValueOnce("c"); // Fragile, breaks if call order changes
```markdown

## Examples

### Testing a Service Class

```typescript
import { describe, it, expect, vi, beforeEach } from "vitest";
import { UserService } from "./user-service.js";
import type { UserRepository } from "./user-repository.js";

describe("UserService", () => {
  let service: UserService;
  let mockRepo: UserRepository;

  beforeEach(() => {
    mockRepo = {
      findById: vi.fn(),
      save: vi.fn(),
      delete: vi.fn(),
    };
    service = new UserService(mockRepo);
  });

  describe("getUser", () => {
    it("should return user when found", async () => {
      const user = { id: "1", name: "Test" };
      vi.mocked(mockRepo.findById).mockResolvedValue(user);

      const result = await service.getUser("1");

      expect(result).toEqual(user);
      expect(mockRepo.findById).toHaveBeenCalledWith("1");
    });

    it("should return null when user not found", async () => {
      vi.mocked(mockRepo.findById).mockResolvedValue(null);

      const result = await service.getUser("999");

      expect(result).toBeNull();
    });
  });
});
```markdown

### Testing Error Handling

```typescript
describe("error handling", () => {
  it("should wrap repository errors", async () => {
    const dbError = new Error("Connection failed");
    vi.mocked(mockRepo.findById).mockRejectedValue(dbError);

    await expect(service.getUser("1")).rejects.toThrow("Failed to fetch user");
  });

  it("should handle validation errors gracefully", () => {
    expect(() => service.validateInput("")).toThrow(ValidationError);
    expect(() => service.validateInput("")).toThrow("Input cannot be empty");
  });
});
```

## References

- [Vitest Documentation](https://vitest.dev/)
- [Jest Documentation](https://jestjs.io/)
- [Testing Library](https://testing-library.com/)
- [MSW (Mock Service Worker)](https://mswjs.io/)
