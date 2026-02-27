---
name: webapp-testing
description: Toolkit for interacting with and testing local web applications using Playwright. Supports verifying frontend functionality, debugging UI behavior, capturing browser screenshots, and viewing browser logs. [MANDATORY] Before saying "implementation complete", you MUST use this skill to run tests and verify functionality. Completion reports without verification are PROHIBITED.
license: Complete terms in LICENSE.txt
---

# Web Application Testing

To test local web applications, write TypeScript E2E tests using **Playwright Test** (`@playwright/test`).

**CRITICAL: E2E Test File Placement**
- **ALWAYS** place E2E test files in `tests/e2e/` or `e2e/` directory at the project root
- **NEVER** place test scripts in `.artifacts/` - that's for evidence only (screenshots, videos)
- E2E tests should be permanent project assets, not disposable artifacts

## Decision Tree: Choosing Your Approach

```
User task → Is it static HTML?
    ├─ Yes → Read HTML file directly to identify selectors
    │         ├─ Success → Write Playwright test using selectors
    │         └─ Fails/Incomplete → Treat as dynamic (below)
    │
    └─ No (dynamic webapp) → Is the server already running?
        ├─ No → Use webServer config in playwright.config.ts
        │        to auto-start the dev server
        │
        └─ Yes → Reconnaissance-then-action:
            1. Navigate and wait for networkidle
            2. Take screenshot or inspect DOM
            3. Identify selectors from rendered state
            4. Execute actions with discovered selectors
```

## Playwright Test Setup

### playwright.config.ts
```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests/e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
  ],
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
    timeout: 120 * 1000,
  },
});
```

### Example Test: tests/e2e/login.spec.ts
```typescript
import { test, expect } from '@playwright/test';

test.describe('Login Flow', () => {
  test('should login successfully with valid credentials', async ({ page }) => {
    await page.goto('/login');
    await page.waitForLoadState('networkidle');

    await page.getByLabel('Email').fill('test@example.com');
    await page.getByLabel('Password').fill('password123');
    await page.getByRole('button', { name: 'Login' }).click();

    await expect(page).toHaveURL('/dashboard');
    await expect(page.getByRole('heading', { name: 'Welcome' })).toBeVisible();
  });

  test('should show error for invalid credentials', async ({ page }) => {
    await page.goto('/login');

    await page.getByLabel('Email').fill('invalid@example.com');
    await page.getByLabel('Password').fill('wrong');
    await page.getByRole('button', { name: 'Login' }).click();

    await expect(page.getByText('Invalid credentials')).toBeVisible();
  });
});
```

## Running Tests

```bash
# Run all E2E tests
npx playwright test

# Run specific test file
npx playwright test tests/e2e/login.spec.ts

# Run with UI mode (interactive debugging)
npx playwright test --ui

# Run headed (visible browser)
npx playwright test --headed

# Generate test code interactively
npx playwright codegen http://localhost:3000
```

## Reconnaissance-Then-Action Pattern

When you don't know the page structure:

```typescript
import { test, expect } from '@playwright/test';

test('discover and interact with page elements', async ({ page }) => {
  await page.goto('/');
  await page.waitForLoadState('networkidle');

  // 1. Take screenshot for reconnaissance
  await page.screenshot({ path: '/tmp/inspect.png', fullPage: true });

  // 2. Log all buttons for analysis
  const buttons = await page.getByRole('button').all();
  for (const button of buttons) {
    console.log('Button:', await button.textContent());
  }

  // 3. Get page content for selector discovery
  const content = await page.content();
  console.log(content);
});
```

## Evidence Collection for PR Reviews

When collecting evidence (screenshots/videos) for PR reviews, use this pattern:

```bash
# Run tests with evidence collection
FEATURE=${FEATURE:-feature}
mkdir -p .artifacts/$FEATURE/{images,videos}

npx playwright test tests/e2e/your-feature.spec.ts \
  --headed \
  --output=.artifacts/$FEATURE \
  --trace=retain-on-failure \
  --reporter=line
```

### Test with Built-in Evidence Collection

```typescript
import { test, expect } from '@playwright/test';

test.describe('Feature Demo', () => {
  test('demonstrate feature workflow', async ({ page }, testInfo) => {
    const feature = process.env.FEATURE || 'feature';
    const timestamp = new Date().toISOString().slice(0, 10).replace(/-/g, '');

    await page.goto('/feature');
    await page.waitForLoadState('networkidle');

    // Capture before state
    await page.screenshot({
      path: `.artifacts/${feature}/images/${timestamp}-before.png`,
      fullPage: true
    });

    // Perform actions
    await page.getByRole('button', { name: 'Enable Feature' }).click();
    await expect(page.getByText('Feature Enabled')).toBeVisible();

    // Capture after state
    await page.screenshot({
      path: `.artifacts/${feature}/images/${timestamp}-after.png`,
      fullPage: true
    });
  });
});
```

## Quick One-liner for Ad-hoc Verification

For quick checks without writing a full test file (use TypeScript via tsx):

```bash
npx tsx -e "
import { chromium } from 'playwright';

const browser = await chromium.launch();
const page = await browser.newPage();
await page.goto(process.env.BASE_URL || 'http://localhost:3000', { waitUntil: 'networkidle' });
await page.screenshot({ path: '/tmp/webapp.png', fullPage: true });
await browser.close();
console.log('saved: /tmp/webapp.png');
"
```

## Common Pitfalls

❌ **Don't** inspect the DOM before waiting for `networkidle` on dynamic apps
✅ **Do** wait for `page.waitForLoadState('networkidle')` before inspection

❌ **Don't** place E2E test files in `.artifacts/`
✅ **Do** place E2E tests in `tests/e2e/` as permanent project assets

❌ **Don't** write tests in Python
✅ **Do** write tests in TypeScript using `@playwright/test`

❌ **Don't** use Vitest for E2E tests
✅ **Do** use Playwright Test for E2E, Vitest for unit/integration tests

## Best Practices

- **Use Playwright Test runner** - Not Vitest or other runners for E2E
- **Always use TypeScript** - Never Python for Playwright tests
- **Place tests in `tests/e2e/`** - Permanent project assets, not disposable
- **Use role-based selectors** - `getByRole`, `getByLabel`, `getByText` over CSS
- **Always close browser** - Playwright Test handles this automatically
- **Add appropriate waits** - `waitForLoadState`, `waitForSelector`, `expect().toBeVisible()`
- **Use `webServer` config** - Auto-start dev server in `playwright.config.ts`

## File Structure Convention

```
project/
├── playwright.config.ts      # Playwright configuration
├── tests/
│   └── e2e/                  # E2E tests (permanent)
│       ├── login.spec.ts
│       ├── checkout.spec.ts
│       └── settings.spec.ts
├── .artifacts/               # Evidence only (temporary)
│   └── <feature>/
│       ├── images/           # Screenshots
│       ├── videos/           # Recorded videos
│       └── REPORT.md         # Review report
└── test-results/             # Playwright auto-generated (gitignored)
```

**Key distinction:**
- `tests/e2e/` = Permanent E2E test code (committed to repo)
- `.artifacts/` = Temporary evidence for PR review (gitignored or LFS)
- `test-results/` = Playwright's auto-generated output (gitignored)

## Console/Error Collection

```typescript
test('collect console logs', async ({ page }) => {
  const consoleLogs: string[] = [];
  const errors: string[] = [];

  page.on('console', msg => consoleLogs.push(`${msg.type()}: ${msg.text()}`));
  page.on('pageerror', err => errors.push(err.message));
  page.on('requestfailed', req => errors.push(`Request failed: ${req.url()}`));

  await page.goto('/');
  await page.waitForLoadState('networkidle');

  console.log('Console logs:', consoleLogs);
  if (errors.length > 0) {
    console.error('Errors:', errors);
  }
});
```

## Vitest vs Playwright Test: When to Use Each

| Test Type | Tool | Reason |
|-----------|------|--------|
| Unit tests | Vitest | Fast, no browser needed |
| Integration tests | Vitest | Fast, mock external deps |
| Component tests | Vitest + browser mode | or Storybook |
| **E2E tests** | **Playwright Test** | Full browser, real flows |

**Do NOT use Vitest for E2E tests.** Playwright Test has:
- Built-in parallelization for browser tests
- Automatic retries and trace collection
- Screenshot/video on failure
- Web server management
- Test generator (`codegen`)
