---
name: bun-testing
description: Testing guidelines for Bun/TypeScript projects using bun:test framework. Use when writing tests, creating test files, debugging test failures, setting up mocks, or reviewing test code. Triggers on *.test.ts files, test-related questions, mocking patterns, and coverage discussions.
---

# Bun Testing Skill

## Quick Reference

- **Framework**: bun:test
- **File pattern**: `*.test.ts` inside `__tests__` directories
- **Module mocking**: Use `ModuleMocker` from `@/__tests__` (see [patterns](references/mocking-patterns.md))
- **Coverage target**: 60–80% (focus on important logic, not 100%)
- **Config**: `.env.test` for test environment variables

## Test Utilities (src/__tests__/)

Before writing custom test helpers, check existing utilities:

- **`createTestApp(basePath, route, middleware[])`** - Creates test Hono app with error handler, logger, and optional middleware
- **`ModuleMocker(import.meta.url)`** - Module mocking utility (see [mocking patterns](references/mocking-patterns.md))
- **`post(app, url, body, headers)`** - POST request helper
- **`get(app, url, headers)`** - GET request helper
- **`doRequest(app, url, method, body, headers)`** - Generic request helper

Example:

```typescript
import { createTestApp, post } from '@/__tests__'

const app = createTestApp('/api/v1/auth', signupRoute, [captchaMiddleware()])
const response = await post(app, '/api/v1/auth/signup', {
  email: 'test@example.com',
  password: 'SecurePass123!'
})
```

## Test Types

- **Unit tests**: Mock all dependencies (repositories, services, APIs)
- **Integration tests**: Use real database, mock external APIs only
- **Endpoint tests**: Use `createTestApp()` with mocked services

## Test Priorities

1. Correct scenario(s)
2. Error handling
3. Boundary inputs
4. Failure scenarios

## Environment Setup

- Use `.env.test` for test-specific variables
- Bun handles env loading natively — no manual dotenv needed
- Minimize mocking `@/env` — only mock for special/invalid configs

## Mocking Strategy

- Mock only business logic dependencies (repositories, external APIs)
- Use global mocks for shared services (CAPTCHA, email) — don't redefine per test
- No real database or network calls — all I/O must be mocked
- Don't mock encapsulated dependencies — mock the public API/wrapper only

## Type Safety

- Minimize `any` — prefer proper TypeScript types
- Use type inference when possible
- Use `Partial<T>` for mock objects
- Exception: Use `any` only for complex mocks where full typing adds unnecessary complexity

## Test Structure

Use arrange → act → assert pattern with descriptive test names:

```typescript
describe('UserService', () => {
  it('should return user when found by email', async () => {
    // Arrange
    const mockUser = { id: 1, email: 'test@example.com' }
    mockUserRepo.findByEmail.mockResolvedValue(mockUser)

    // Act
    const result = await userService.findByEmail('test@example.com')

    // Assert
    expect(result).toEqual(mockUser)
  })
})
```

## Mocking Patterns

For detailed mocking patterns including variable ordering, `beforeEach` setup, and ModuleMocker usage, see [references/mocking-patterns.md](references/mocking-patterns.md).

## Cleanup

Always clean up side effects after each test:

```typescript
afterEach(async () => {
  await moduleMocker.clear() // restore mocked modules
  vi.clearAllMocks() // or mock.mockClear() for individual mocks
})
```
