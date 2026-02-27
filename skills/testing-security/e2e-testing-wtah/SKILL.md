---
name: e2e-testing
description: "Guidelines for planning and implementing end-to-end tests using browser automation (Playwright). Covers test case planning from user journeys/requirements and individual test development with MCP server access."
---

# E2E Testing Skill

This skill provides guidance for **planning and implementing end-to-end tests** that validate fullstack integration (frontend + backend) using browser automation tools like Playwright.

**IMPORTANT**: E2E tests run in **LOCAL_RUN=true mode by default**, bypassing authentication to enable efficient test development without infrastructure dependencies.

---

## Core Concept

E2E tests validate complete user journeys through the application stack:

```
User Journey (from .product/)
         ↓
   E2E Test Planner
         ↓
   Test Cases (.e2e-tests/test-plan.md)
         ↓
   E2E Test Developers (parallel, max 4)
         ↓
   Playwright Tests (e2e-tests/)
```

---

## LOCAL_RUN Mode (Default)

E2E tests **always run with LOCAL_RUN=true by default** to:
- Bypass authentication infrastructure
- Use the test user (`localdev@test.local`)
- Enable rapid test development without external dependencies
- Allow developers to focus on user journey validation

### When to Use Each Mode

| Mode | LOCAL_RUN | Use Case |
|------|-----------|----------|
| **Local Development** | `true` (default) | Writing tests, debugging, CI pipeline |
| **Infrastructure Testing** | `false` | Explicit testing of real auth flows |

### Infrastructure Mode

Only use `LOCAL_RUN=false` when explicitly testing:
- Real authentication flows (OAuth, SSO)
- Token validation logic
- Session management
- Auth error handling

This must be **explicitly requested** and is NOT the default.

---

## Two Roles

### 1. E2E Test Planner

Analyzes product requirements and architecture specs to define comprehensive test cases.

**Input**:
- `.product/USER_JOURNEYS.md` - User journey definitions
- `.product/REQUIREMENTS.md` - Functional requirements
- `.arch-registry/README.md` - System architecture
- `{container}/.specs/` - Container specifications

**Output**:
- `.e2e-tests/test-plan.md` - Comprehensive test plan with individual test cases

### 2. E2E Test Developer

Implements individual test cases using Playwright via MCP server.

**Input**:
- `.e2e-tests/test-plan.md` - Test case specification
- Access to Playwright MCP server for browser automation

**Output**:
- `e2e-tests/{test-id}.spec.ts` - Playwright test file

---

## Test Plan Structure

### `.e2e-tests/test-plan.md`

```markdown
# E2E Test Plan - {Product Name}

## Overview

| Metric | Value |
|--------|-------|
| Total Test Cases | {n} |
| Coverage | User journeys: {n}/{total}, Requirements: {n}/{total} |
| Priority Distribution | Critical: {n}, High: {n}, Medium: {n}, Low: {n} |

## Test Environment

| Component | URL/Config |
|-----------|------------|
| Frontend | {url} |
| Backend API | {url} |
| Test Data | {setup instructions} |

---

## Test Cases

### E2E-001: {Test Case Title}

| Attribute | Value |
|-----------|-------|
| **Priority** | Critical / High / Medium / Low |
| **User Journey** | UJ-{xxx} |
| **Requirements** | FR-{xxx}, FR-{yyy} |
| **Preconditions** | {what must be true before test} |

#### Test Steps

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | {user action} | {expected outcome} |
| 2 | {user action} | {expected outcome} |
| 3 | {user action} | {expected outcome} |

#### Test Data

```json
{
  "input": { ... },
  "expected": { ... }
}
```

#### Assertions

- [ ] {assertion 1}
- [ ] {assertion 2}
- [ ] {assertion 3}

#### Status

- [ ] Planned
- [ ] In Development
- [ ] Implemented
- [ ] Passing

---

### E2E-002: {Next Test Case}

{same structure}

---

## Coverage Matrix

### User Journey Coverage

| Journey | Test Cases | Status |
|---------|------------|--------|
| UJ-001 | E2E-001, E2E-003 | Covered |
| UJ-002 | E2E-002, E2E-004, E2E-005 | Covered |

### Requirement Coverage

| Requirement | Test Cases | Status |
|-------------|------------|--------|
| FR-001 | E2E-001 | Covered |
| FR-002 | E2E-002, E2E-003 | Covered |
```

---

## Local Development Setup

Before developing E2E tests, the application must be running locally with `LOCAL_RUN=true`.

### Discovering Startup Commands (Technology-Agnostic)

E2E test agents must **discover** the actual startup commands from existing project artifacts:

1. **Container README**: `{container}/README.md` - contains dev startup instructions
2. **Container scripts**: `{container}/scripts/` - start, dev, setup scripts from integration-developer
3. **Technology specs**: `{container}/.specs/technology.md` - framework and runtime info
4. **Global constraints**: `.constraints/TECHNOLOGY.md` - tech stack definition
5. **Local setup skill**: `.claude/skills/local-setup/SKILL.md` - LOCAL_RUN patterns per framework

### Environment Variable Patterns by Technology

Read `.claude/skills/local-setup/SKILL.md` for the correct pattern:

| Technology | Backend Variable | Frontend Variable |
|------------|------------------|-------------------|
| Node.js/Express | `LOCAL_RUN=true` | N/A |
| Python/FastAPI/Flask | `LOCAL_RUN=true` | N/A |
| Go | `LOCAL_RUN=true` | N/A |
| Vite | N/A | `VITE_LOCAL_RUN=true` |
| Create React App | N/A | `REACT_APP_LOCAL_RUN=true` |
| Next.js | N/A | `NEXT_PUBLIC_LOCAL_RUN=true` |
| Vue CLI | N/A | `VUE_APP_LOCAL_RUN=true` |
| Angular | N/A | Set `localRun: true` in `environment.ts` |

The test plan must include **actual commands** discovered from the project.

### Test User in LOCAL_RUN Mode

When `LOCAL_RUN=true`, you are automatically authenticated as:

| Field | Value |
|-------|-------|
| ID | `test-user-local-dev` |
| Email | `localdev@test.local` |
| Name | `Local Developer` |
| Role | `admin` |
| Permissions | `*` (all) |

Tests should **not** include login steps when running in LOCAL_RUN mode. The test user is pre-authenticated.

---

## Test Development Guidelines

### Playwright Test Structure

```typescript
import { test, expect } from '@playwright/test';

test.describe('E2E-001: {Test Case Title}', () => {
  test.beforeEach(async ({ page }) => {
    // Setup: navigate to starting point
    await page.goto('/');
  });

  test('should {expected behavior}', async ({ page }) => {
    // Step 1: {action}
    await page.click('[data-testid="button"]');

    // Step 2: {action}
    await page.fill('[data-testid="input"]', 'value');

    // Assertion
    await expect(page.locator('[data-testid="result"]')).toBeVisible();
  });
});
```

### Selector Strategy (Priority Order)

1. **data-testid** - Most reliable, add to source if needed
2. **role + name** - `getByRole('button', { name: 'Submit' })`
3. **label** - `getByLabel('Email')`
4. **placeholder** - `getByPlaceholder('Enter email')`
5. **text** - `getByText('Welcome')` (last resort)

### Wait Strategies

```typescript
// Prefer explicit waits over fixed timeouts
await page.waitForSelector('[data-testid="loaded"]');
await expect(page.locator('.result')).toBeVisible({ timeout: 10000 });

// Avoid
await page.waitForTimeout(3000); // Anti-pattern
```

### Test Data Patterns

```typescript
// Use fixtures for reusable test data
const testUser = {
  email: 'test@example.com',
  password: 'TestPass123!'
};

// Generate unique data for isolation
const uniqueId = `test-${Date.now()}`;
```

---

## MCP Server Integration

E2E Test Developers have access to the **Playwright MCP server** for browser automation.

### Available MCP Tools

The Playwright MCP server provides tools for:

| Tool | Purpose |
|------|---------|
| `browser_navigate` | Navigate to URLs |
| `browser_click` | Click elements |
| `browser_type` | Type into inputs |
| `browser_screenshot` | Capture screenshots |
| `browser_get_text` | Extract text content |
| `browser_wait` | Wait for elements |

### Using MCP for Test Development

1. **Explore the UI** - Use MCP to navigate and understand the application
2. **Identify selectors** - Find reliable element selectors
3. **Validate interactions** - Test actions before writing assertions
4. **Debug failures** - Use screenshots to understand failures

---

## Test Planning Process

### Step 1: Analyze User Journeys

Read `.product/USER_JOURNEYS.md` and identify:
- Happy path scenarios (must test)
- Alternative flows (should test)
- Exception flows (could test)
- Edge cases

### Step 2: Map Requirements

Cross-reference with `.product/REQUIREMENTS.md`:
- Which requirements are covered by each journey?
- Are there requirements not covered by journeys?
- What acceptance criteria need E2E validation?

### Step 3: Review Architecture

Check `.arch-registry/README.md` and container specs:
- Frontend/backend communication patterns
- API endpoints involved
- Authentication flows
- Data flow between containers

### Step 4: Define Test Cases

For each user journey step:
1. Identify testable actions
2. Define expected outcomes
3. Specify test data requirements
4. Note preconditions and dependencies

### Step 5: Prioritize

| Priority | Criteria |
|----------|----------|
| **Critical** | Core functionality, main happy paths |
| **High** | Important features, common use cases |
| **Medium** | Edge cases, alternative flows |
| **Low** | Rare scenarios, nice-to-have |

---

## Test File Organization

```
e2e-tests/
├── playwright.config.ts       # Playwright configuration
├── package.json               # Dependencies
├── fixtures/                  # Test fixtures and helpers
│   ├── auth.ts               # Authentication helpers
│   └── test-data.ts          # Test data generators
├── tests/
│   ├── auth/                 # Authentication tests
│   │   ├── e2e-001-login.spec.ts
│   │   └── e2e-002-logout.spec.ts
│   ├── products/             # Product-related tests
│   │   ├── e2e-003-import.spec.ts
│   │   └── e2e-004-search.spec.ts
│   └── compatibility/        # Compatibility check tests
│       └── e2e-005-check.spec.ts
└── .e2e-tests/
    └── test-plan.md          # Test plan document
```

---

## Playwright Configuration

### `playwright.config.ts` (Template)

The actual configuration must be generated based on the project's technology stack.
Read from `.constraints/TECHNOLOGY.md` and container specs to determine:
- Base URLs (frontend, backend health endpoint)
- Startup commands (from container README/scripts)
- Environment variables (from local-setup skill)

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',

  use: {
    // Discover from container specs
    baseURL: process.env.BASE_URL || '{frontend-url-from-specs}',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },

  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
  ],

  // Web server config - commands discovered from container README/scripts
  // Environment variables from local-setup skill for the technology
  webServer: [
    {
      command: '{backend-start-command-from-readme}',
      cwd: '{api-container-path}',
      url: '{backend-health-url}',
      reuseExistingServer: !process.env.CI,
      env: { LOCAL_RUN: 'true' },  // Backend always uses LOCAL_RUN
    },
    {
      command: '{frontend-start-command-from-readme}',
      cwd: '{frontend-container-path}',
      url: '{frontend-url}',
      reuseExistingServer: !process.env.CI,
      env: { '{FRONTEND_LOCAL_RUN_VAR}': 'true' },  // From local-setup skill
    },
  ],
});
```

### Infrastructure Mode Configuration

For explicit auth testing (when requested), create `playwright.infra.config.ts`:
- Same structure but with `LOCAL_RUN` variables set to `'false'`
- Only used when explicitly testing authentication flows

Run infrastructure tests:
```
{test-runner-command} --config=playwright.infra.config.ts
```

---

## Quality Criteria

### Test Case Quality

| Criterion | Measure |
|-----------|---------|
| **Independent** | Tests don't depend on each other |
| **Repeatable** | Same result on every run |
| **Clear assertions** | Obvious what's being verified |
| **Fast feedback** | Minimal wait times |
| **Meaningful** | Tests real user scenarios |

### Coverage Quality

| Criterion | Target |
|-----------|--------|
| User journey coverage | 100% of critical paths |
| Requirement coverage | All testable requirements |
| No duplication | Each scenario tested once |

---

## Anti-Patterns to Avoid

| Anti-Pattern | Problem | Solution |
|--------------|---------|----------|
| **Fixed sleeps** | Slow, flaky | Use explicit waits |
| **Brittle selectors** | Break on UI changes | Use data-testid |
| **Coupled tests** | Order-dependent failures | Make tests independent |
| **No cleanup** | Data pollution | Reset state after tests |
| **Testing internals** | Breaks on refactoring | Test user behavior |
| **Too many assertions** | Hard to debug | One concept per test |

---

## Coordination with Architecture

### Reading Specs for Test Planning

```
.product/USER_JOURNEYS.md     → Primary source of test scenarios
.product/REQUIREMENTS.md      → Acceptance criteria to verify
.arch-registry/README.md      → Understand system structure
{frontend}/.specs/            → UI component behavior
{api}/.specs/interfaces/      → API contracts to test
```

### What to Test E2E vs Unit/Integration

| E2E Tests | Unit/Integration Tests |
|-----------|----------------------|
| Complete user flows | Individual functions |
| Cross-container integration | Single container behavior |
| UI + API interaction | API only |
| Browser behavior | Logic only |
| Real user scenarios | Edge cases |

---

## Parallel Execution Limits

When developing tests in parallel:

| Setting | Value |
|---------|-------|
| Max parallel test developers | 4 |
| Test execution parallelism | Configurable in playwright.config.ts |
| Resource isolation | Each developer works on separate test file |

---

## Status Tracking

### In Test Plan

```markdown
#### Status

- [x] Planned
- [x] In Development
- [ ] Implemented
- [ ] Passing
```

### After Development

Update `.e2e-tests/test-plan.md` with:
- Status changes
- Actual file locations
- Any deviations from plan
- Known issues or limitations
