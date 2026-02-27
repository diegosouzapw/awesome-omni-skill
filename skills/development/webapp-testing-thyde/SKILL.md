---
name: webapp-testing
description: Comprehensive testing toolkit for LucidData using Playwright (E2E) and Vitest (unit/integration). Use when verifying frontend functionality, testing user flows (auth, vault CRUD, consent management), running test suites, debugging test failures, capturing screenshots, checking browser console logs, or validating responsive behavior. Integrates with existing LucidData test infrastructure and follows TDD practices.
license: MIT
compatibility: Designed for Claude Code and GitHub Copilot. Requires Node.js 18+, Playwright 1.57.0, Vitest 3.2.4. Works with locally running dev server at localhost:3000.
metadata:
  author: LucidData Team
  version: "1.0"
  e2e_framework: Playwright 1.57.0
  unit_framework: Vitest 3.2.4
  coverage_target: 80%
  test_env: .env.test
allowed-tools: Bash(npm run test:*) Bash(npx playwright:*) Bash(npx vitest:*) Read Write Edit
---

# Web Application Testing

Comprehensive testing toolkit for the LucidData personal data bank MVP, integrating Playwright for end-to-end testing and Vitest for unit/integration testing.

## Overview

This skill enables test-driven development (TDD) workflows for the LucidData Next.js application. It provides patterns for writing, running, and debugging tests across all layers: unit tests for utilities and services, integration tests for React components and API routes, and end-to-end tests for complete user workflows.

## When to Use This Skill

Activate this skill when you need to:

- **Run test suites**: Execute unit, integration, or E2E tests
- **Debug test failures**: Investigate failing tests, review screenshots, check console logs
- **Write new tests**: Create test files following LucidData patterns
- **Verify functionality**: Test user flows like signup → vault creation → consent management
- **Check test coverage**: Generate and analyze coverage reports (80% target)
- **Test responsive behavior**: Validate UI across different viewports
- **Validate accessibility**: Ensure WCAG compliance through automated tests

## LucidData Testing Philosophy

The project follows a **test pyramid approach**:
- **Many unit tests**: Fast, isolated tests for utilities, services, and pure functions
- **Some integration tests**: Tests for React components with hooks, API route handlers
- **Few E2E tests**: Critical user journeys (auth, core workflows)

**Coverage targets**: 80% for lines, functions, branches, and statements.

**Test naming conventions**:
- Unit/Integration: `*.test.ts` or `*.test.tsx`
- E2E: `*.spec.ts`

## Test Infrastructure

### Vitest (Unit & Integration Testing)

**Configuration**: [vitest.config.ts](../../vitest.config.ts)

**Key settings**:
- Environment: `jsdom` (simulates browser DOM)
- Setup file: [test/setup.ts](../../test/setup.ts)
- Global test functions: `describe`, `it`, `expect` available everywhere
- Excludes E2E tests: `__tests__/e2e/**` directory excluded
- Coverage: v8 provider with 80% thresholds
- Timeouts: 10000ms for tests and hooks

**Test utilities**:
- **Mocks**: [test/mocks/prisma.ts](../../test/mocks/prisma.ts), [test/mocks/supabase.ts](../../test/mocks/supabase.ts)
- **Fixtures**: [test/fixtures/](../../test/fixtures/) (sample data factories)
- **Helpers**: [test/utils/](../../test/utils/) and [test/helpers/](../../test/helpers/)

**Libraries**:
- `@testing-library/react`: Component testing with queries and user events
- `@testing-library/jest-dom`: Custom matchers (e.g., `toBeInTheDocument()`)
- `@testing-library/user-event`: Simulate user interactions
- `vitest-mock-extended`: Advanced mocking with `DeepMockProxy`
- `msw`: Mock Service Worker for API request interception

### Playwright (E2E Testing)

**Configuration**: [playwright.config.ts](../../playwright.config.ts)

**Key settings**:
- Test directory: `__tests__/e2e`
- Global setup: [__tests__/e2e/global-setup.ts](../../__tests__/e2e/global-setup.ts)
- Global teardown: [__tests__/e2e/global-teardown.ts](../../__tests__/e2e/global-teardown.ts)
- Base URL: `http://localhost:3000`
- Parallel execution: Enabled
- Browsers: Chromium, Firefox, WebKit, Mobile Chrome (Pixel 5), Mobile Safari (iPhone 12)
- Web server: Auto-starts Next.js dev server before tests
- Screenshots: On failure
- Videos: On first retry
- Traces: On first retry

**Helpers**:
- **Auth helper**: [__tests__/e2e/helpers/auth.ts](../../__tests__/e2e/helpers/auth.ts)
  - `login()`, `signup()`, `logout()`, `isAuthenticated()`, `getUniqueEmail()`, `clearSession()`
- **Data generators**: [__tests__/e2e/helpers/data-generators.ts](../../__tests__/e2e/helpers/data-generators.ts)
  - `generateVaultEntry()`, `generateConsent()`, `generateVaultEntries(count)`

## Common Testing Patterns

### Unit Testing Pattern (Vitest)

```typescript
import { describe, it, expect, vi } from 'vitest';

describe('UtilityFunction', () => {
  it('should perform expected operation', () => {
    const result = utilityFunction('input');
    expect(result).toBe('expected output');
  });

  it('should handle edge cases', () => {
    const result = utilityFunction('');
    expect(result).toBe('default value');
  });
});
```

**Example from LucidData** ([lib/crypto/__tests__/hashing.test.ts](../../lib/crypto/__tests__/hashing.test.ts)):
```typescript
describe('hash', () => {
  it('should produce SHA-256 hex string', () => {
    const hashed = hash('test data');
    expect(typeof hashed).toBe('string');
    expect(hashed).toHaveLength(64);
    expect(hashed).toMatch(/^[a-f0-9]{64}$/);
  });

  it('should be deterministic', () => {
    const hash1 = hash('data');
    const hash2 = hash('data');
    expect(hash1).toBe(hash2);
  });
});
```

### Integration Testing Pattern (React Components)

```typescript
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { vi } from 'vitest';

vi.mock('@/lib/db/prisma', () => ({
  prisma: prismaMock,
}));

describe('ComponentName', () => {
  it('should render correctly', () => {
    render(<ComponentName />);
    expect(screen.getByText('Expected Text')).toBeInTheDocument();
  });

  it('should handle user interaction', async () => {
    const user = userEvent.setup();
    render(<ComponentName />);

    await user.click(screen.getByRole('button', { name: 'Submit' }));
    expect(mockFunction).toHaveBeenCalled();
  });
});
```

**Example from LucidData** (repository tests):
```typescript
import { prismaMock } from '@/test/mocks/prisma';

describe('VaultRepository', () => {
  beforeEach(() => {
    repository = new VaultRepository();
    vi.clearAllMocks();
  });

  it('should return vault entries for a user', async () => {
    prismaMock.vaultData.findMany.mockResolvedValue(userEntries);
    const result = await repository.findByUserId(userId);

    expect(result).toEqual(userEntries);
    expect(prismaMock.vaultData.findMany).toHaveBeenCalledWith({
      where: { userId },
      orderBy: { createdAt: 'desc' },
    });
  });
});
```

### E2E Testing Pattern (Playwright)

```typescript
import { test, expect } from '@playwright/test';

test.describe('Feature Flow', () => {
  test.beforeEach(async ({ page }) => {
    await clearSession(page);
  });

  test('should complete user workflow', async ({ page }) => {
    await page.goto('/feature-page');

    // Fill form
    await page.fill('input[name="field"]', 'value');
    await page.click('button[type="submit"]');

    // Verify result
    await expect(page.locator('text=Success')).toBeVisible();
  });
});
```

**Example from LucidData** ([__tests__/e2e/auth/signup.spec.ts](../../__tests__/e2e/auth/signup.spec.ts)):
```typescript
test.describe('Signup Flow', () => {
  test('should successfully create account', async ({ page }) => {
    await page.goto('/signup');
    const email = getUniqueEmail();

    await page.fill('input[name="email"]', email);
    await page.fill('input[name="password"]', TEST_USER.password);
    await page.click('button[type="submit"]');

    await expect(page).toHaveURL('/dashboard', { timeout: 10000 });
    await expect(page.locator(`text=${email}`)).toBeVisible();
  });
});
```

## Test Commands

### Unit & Integration Tests (Vitest)

```bash
# Watch mode (re-runs on file changes)
npm run test

# Single run (for CI/CD)
npm run test:run

# With coverage report
npm run test:coverage

# UI mode (visual test runner)
npm run test:ui

# Run specific test file
npx vitest run path/to/test.test.ts

# Run tests matching pattern
npx vitest run --grep "VaultRepository"
```

### E2E Tests (Playwright)

```bash
# Headless mode (no browser UI)
npm run test:e2e

# Headed mode (see browser)
npm run test:e2e:headed

# UI mode (interactive)
npm run test:e2e:ui

# Debug mode (step through tests)
npm run test:e2e:debug

# Run specific test file
npx playwright test __tests__/e2e/auth/signup.spec.ts

# Run specific browser
npx playwright test --project=chromium

# Run specific test by name
npx playwright test --grep "should successfully create account"
```

### All Tests

```bash
# Run all test suites (unit, integration, E2E)
npm run test:all
```

## Debugging Test Failures

### Step-by-Step Debugging Process

1. **Read test output**: Identify the failing test and assertion error
2. **Check screenshots**: E2E failures auto-save screenshots in `test-results/`
3. **Review browser logs**: Playwright captures `console.log`, `console.error` messages
4. **Use interactive debugging**:
   - Vitest: Add `vi.debug()` to inspect values
   - Playwright: Add `await page.pause()` for interactive debugging
5. **Verify test fixtures/mocks**: Ensure mock data matches expected structure
6. **Check environment variables**: Verify `.env.test` has correct configuration
7. **Isolate the issue**: Run only the failing test with `--grep` flag

### Common Issues & Solutions

**Issue**: "Element not found" in E2E test
- **Cause**: Selector changed or element not rendered yet
- **Solution**: Use `await expect(element).toBeVisible()` or increase timeout

**Issue**: "React Query mutation not called" in integration test
- **Cause**: Mock setup timing issue
- **Solution**: Use `await waitFor()` from @testing-library/react

**Issue**: "Prisma method not mocked"
- **Cause**: Missing mock setup for database call
- **Solution**: Add `prismaMock.model.method.mockResolvedValue()` in test setup

**Issue**: Test timeout
- **Cause**: Async operation taking too long
- **Solution**: Increase timeout in test config or use `{ timeout: 15000 }` in specific test

## LucidData-Specific Test Scenarios

### Authentication Tests
- Signup flow: Create account → Redirect to dashboard
- Login flow: Enter credentials → Successful login
- Logout flow: Click logout → Redirect to landing page
- Protected routes: Access dashboard without auth → Redirect to login

**Location**: `__tests__/e2e/auth/`

### Vault CRUD Tests
- Create vault entry: Fill form → Submit → Entry appears in list
- Read vault entry: Click entry → View dialog shows data
- Update vault entry: Edit form → Save → Changes reflected
- Delete vault entry: Click delete → Confirm → Entry removed

**Location**: `__tests__/e2e/vault/`

### Consent Management Tests
- Grant consent: Create consent → Set permissions → Save
- View consent: Open consent list → See active/revoked consents
- Revoke consent: Click revoke → Confirm → Status updated
- Filter consents: Apply filters → List updates

**Location**: `__tests__/e2e/consent/` (when implemented)

### Audit Log Tests
- View audit log: Navigate to audit page → See log entries
- Filter by event type: Select filter → Entries update
- Verify hash chain: Check hash integrity across entries

**Location**: `__tests__/e2e/audit/` (when implemented)

### Responsive Design Tests
Test UI across viewports:
- Mobile: 375px, 390px (iPhone SE, iPhone 12)
- Tablet: 768px, 1024px (iPad, iPad Pro)
- Desktop: 1920px, 2560px (HD, QHD)

```typescript
test('should be responsive on mobile', async ({ page }) => {
  await page.setViewportSize({ width: 375, height: 667 });
  await page.goto('/dashboard');
  // Verify mobile layout
});
```

### Accessibility Tests
- Keyboard navigation: Tab through forms, activate buttons with Enter
- Screen reader labels: Check ARIA attributes, alt text
- Color contrast: Automated checks with axe-core (future integration)

## Test Data Management

### Using Fixtures

Import pre-built data from `test/fixtures/`:

```typescript
import { mockVaultEntry, createMockVaultEntries } from '@/test/fixtures/vault-data';

// Use single entry
const entry = mockVaultEntry;

// Generate multiple entries
const entries = createMockVaultEntries(5, 'user-id-123');
```

### E2E Data Generators

For E2E tests, use helper functions to create unique test data:

```typescript
import { generateVaultEntry } from '@/__tests__/e2e/helpers/data-generators';

const entry = generateVaultEntry({
  label: 'Test Entry',
  category: 'Identity',
});
```

### Cleanup After E2E Tests

Global teardown ([__tests__/e2e/global-teardown.ts](../../__tests__/e2e/global-teardown.ts)) handles cleanup. For manual cleanup:

```typescript
test.afterEach(async ({ page }) => {
  // Delete test data created during test
  await clearTestData(page);
});
```

## Writing New Tests

### 1. Decide Test Type

- **Unit test**: Testing a single function/utility in isolation
- **Integration test**: Testing components with hooks, API routes with database calls
- **E2E test**: Testing complete user workflows across multiple pages

### 2. Choose Location

- **Unit**: `lib/**/__tests__/utility.test.ts`
- **Integration**: `components/**/__tests__/component.test.tsx` or `lib/**/__tests__/service.test.ts`
- **E2E**: `__tests__/e2e/feature/flow.spec.ts`

### 3. Follow Existing Patterns

Look at similar existing tests for structure, mock setup, and assertion patterns.

### 4. Use Descriptive Test Names

```typescript
// Good
it('should encrypt data with AES-256-GCM')
it('should redirect to login when not authenticated')

// Bad
it('works')
it('test encryption')
```

### 5. Arrange-Act-Assert (AAA) Pattern

```typescript
it('should create vault entry', async () => {
  // Arrange: Set up test data and mocks
  const input = { label: 'Test', category: 'Identity' };
  prismaMock.vaultData.create.mockResolvedValue(mockEntry);

  // Act: Execute the function under test
  const result = await vaultService.create(input);

  // Assert: Verify the outcome
  expect(result).toEqual(mockEntry);
  expect(prismaMock.vaultData.create).toHaveBeenCalledWith({
    data: expect.objectContaining(input),
  });
});
```

## Best Practices

1. **Isolate tests**: Each test should be independent, not relying on other tests' state
2. **Use realistic data**: Fixtures should resemble production data (but never use real PII)
3. **Mock external dependencies**: Mock Prisma, Supabase, API calls for unit/integration tests
4. **Test error paths**: Don't just test happy paths; verify error handling
5. **Keep tests fast**: Unit tests should run in milliseconds, integration tests in seconds
6. **Avoid test pollution**: Clean up mocks with `vi.clearAllMocks()` in `beforeEach()`
7. **Use semantic queries**: Prefer `getByRole`, `getByLabelText` over `getByTestId`
8. **Wait for async updates**: Use `await waitFor()` or `await expect().toBeVisible()` for async operations

## Coverage Analysis

Generate coverage report:

```bash
npm run test:coverage
```

Coverage report location: `coverage/index.html` (open in browser)

**Interpreting coverage**:
- **Green**: Above 80% threshold
- **Yellow**: 50-80% coverage
- **Red**: Below 50% coverage

**Focus on**:
- **Lines**: Percentage of code lines executed
- **Functions**: Percentage of functions called
- **Branches**: Percentage of if/else paths tested
- **Statements**: Percentage of statements executed

**Note**: 100% coverage doesn't guarantee bug-free code. Focus on testing critical paths and edge cases.

## Environment Configuration

Test environment uses `.env.test` file:

```env
NEXT_PUBLIC_SUPABASE_URL=http://localhost:54321
SUPABASE_SERVICE_ROLE_KEY=<test-key>
DATABASE_URL=postgresql://postgres:postgres@localhost:54322/postgres
ENCRYPTION_KEY=eugsb9sWIyEOcEg5AzHazv0k7CMTjSsGfL1lbLp8duU=
NEXT_PUBLIC_APP_URL=http://localhost:3000
NODE_ENV=test
```

**Important**: Never commit real keys or sensitive data to version control.

## References

For more detailed information, see:

- [Playwright Patterns](references/playwright-patterns.md) - Page objects, selectors, authentication helpers
- [Vitest Patterns](references/vitest-patterns.md) - Component testing, mocking strategies, async patterns
- [Test Data](references/test-data.md) - Fixture structure, sample data, test credentials

## Unified Test Runner Script

For convenience, use the [run-tests.sh](scripts/run-tests.sh) script:

```bash
# Run unit tests
./scripts/run-tests.sh unit

# Run E2E tests
./scripts/run-tests.sh e2e

# Run all tests
./scripts/run-tests.sh all

# With coverage
./scripts/run-tests.sh unit --coverage
```

## Quick Reference

| Task | Command | Notes |
|------|---------|-------|
| Run unit tests (watch) | `npm run test` | Re-runs on file changes |
| Run unit tests (once) | `npm run test:run` | For CI/CD |
| Coverage report | `npm run test:coverage` | 80% threshold |
| Vitest UI | `npm run test:ui` | Visual test runner |
| Run E2E tests | `npm run test:e2e` | Headless mode |
| E2E headed mode | `npm run test:e2e:headed` | See browser |
| E2E UI mode | `npm run test:e2e:ui` | Interactive debugging |
| E2E debug mode | `npm run test:e2e:debug` | Step through tests |
| Run all tests | `npm run test:all` | Unit + E2E |
| Specific test file | `npx vitest run path/to/test.test.ts` | Single file |
| Specific E2E test | `npx playwright test path/to/test.spec.ts` | Single E2E file |

---

**Version**: 1.0
**Last Updated**: 2026-01-13
**Maintained by**: LucidData Team
