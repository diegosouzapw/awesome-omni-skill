---
name: e2e-testing
description: End-to-end testing patterns and best practices for web applications using Playwright, Cypress, Selenium, and Puppeteer. Covers Page Object Model, test fixtures, selector strategies, async handling, visual regression testing, and flaky test prevention. Includes QA expertise for acceptance testing, smoke testing, cross-browser testing, and test reliability. Use when setting up E2E tests, debugging test failures, improving test reliability, or implementing browser automation. Trigger keywords: e2e, e2e testing, end-to-end, end-to-end tests, Playwright, Cypress, Selenium, Puppeteer, Page Object Model, page object, test fixtures, selectors, locator, locators, data-testid, async tests, visual regression, visual testing, screenshot, flaky tests, flakiness, browser testing, browser automation, UI test, UI testing, acceptance test, acceptance testing, smoke test, smoke testing, integration test, wait, waits, assertion, assertions, test data, test isolation.
---

# E2E Testing

## Overview

End-to-end (E2E) testing validates complete user flows through the application, ensuring all components work together correctly. This skill covers modern E2E testing patterns using Playwright and Cypress, including architectural patterns, selector strategies, and techniques for building reliable, maintainable test suites.

## Instructions

### 1. Choose Your Framework

**Playwright vs Cypress Comparison:**

| Feature              | Playwright                    | Cypress               |
| -------------------- | ----------------------------- | --------------------- |
| Multi-browser        | Chrome, Firefox, Safari, Edge | Chrome, Firefox, Edge |
| Multi-tab/window     | Yes                           | Limited               |
| Network interception | Powerful                      | Good                  |
| Parallel execution   | Built-in                      | Requires Dashboard    |
| Language support     | JS, TS, Python, .NET, Java    | JS, TS                |
| iframes              | Full support                  | Limited               |
| Mobile emulation     | Excellent                     | Basic                 |

**Playwright Setup:**

```bash
npm init playwright@latest
```

```typescript
// playwright.config.ts
import { defineConfig, devices } from "@playwright/test";

export default defineConfig({
  testDir: "./e2e",
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [["html"], ["junit", { outputFile: "results.xml" }]],
  use: {
    baseURL: "http://localhost:3000",
    trace: "on-first-retry",
    screenshot: "only-on-failure",
  },
  projects: [
    { name: "chromium", use: { ...devices["Desktop Chrome"] } },
    { name: "firefox", use: { ...devices["Desktop Firefox"] } },
    { name: "webkit", use: { ...devices["Desktop Safari"] } },
    { name: "mobile", use: { ...devices["iPhone 13"] } },
  ],
  webServer: {
    command: "npm run dev",
    url: "http://localhost:3000",
    reuseExistingServer: !process.env.CI,
  },
});
```

**Cypress Setup:**

```bash
npm install cypress --save-dev
```

```typescript
// cypress.config.ts
import { defineConfig } from "cypress";

export default defineConfig({
  e2e: {
    baseUrl: "http://localhost:3000",
    viewportWidth: 1280,
    viewportHeight: 720,
    video: false,
    screenshotOnRunFailure: true,
    retries: { runMode: 2, openMode: 0 },
    setupNodeEvents(on, config) {
      // Task plugins
    },
  },
});
```

### 2. Implement Page Object Model (POM)

**Playwright Page Object:**

```typescript
// e2e/pages/LoginPage.ts
import { Page, Locator } from "@playwright/test";

export class LoginPage {
  readonly page: Page;
  readonly emailInput: Locator;
  readonly passwordInput: Locator;
  readonly submitButton: Locator;
  readonly errorMessage: Locator;

  constructor(page: Page) {
    this.page = page;
    this.emailInput = page.getByLabel("Email");
    this.passwordInput = page.getByLabel("Password");
    this.submitButton = page.getByRole("button", { name: "Sign in" });
    this.errorMessage = page.getByRole("alert");
  }

  async goto() {
    await this.page.goto("/login");
  }

  async login(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
  }

  async getErrorMessage(): Promise<string> {
    return (await this.errorMessage.textContent()) ?? "";
  }
}
```

**Cypress Page Object:**

```typescript
// cypress/pages/LoginPage.ts
export class LoginPage {
  visit() {
    cy.visit("/login");
    return this;
  }

  getEmailInput() {
    return cy.findByLabelText("Email");
  }

  getPasswordInput() {
    return cy.findByLabelText("Password");
  }

  getSubmitButton() {
    return cy.findByRole("button", { name: "Sign in" });
  }

  login(email: string, password: string) {
    this.getEmailInput().type(email);
    this.getPasswordInput().type(password);
    this.getSubmitButton().click();
    return this;
  }
}
```

**Page Object Composition:**

```typescript
// e2e/pages/index.ts
import { Page } from "@playwright/test";
import { LoginPage } from "./LoginPage";
import { DashboardPage } from "./DashboardPage";
import { CheckoutPage } from "./CheckoutPage";

export class App {
  readonly login: LoginPage;
  readonly dashboard: DashboardPage;
  readonly checkout: CheckoutPage;

  constructor(page: Page) {
    this.login = new LoginPage(page);
    this.dashboard = new DashboardPage(page);
    this.checkout = new CheckoutPage(page);
  }
}

// Usage in tests
test("user can complete purchase", async ({ page }) => {
  const app = new App(page);
  await app.login.goto();
  await app.login.login("user@example.com", "password");
  await app.dashboard.selectProduct("Widget");
  await app.checkout.completePayment();
});
```

### 3. Manage Test Fixtures and Data

**Playwright Fixtures:**

```typescript
// e2e/fixtures/auth.fixture.ts
import { test as base } from "@playwright/test";
import { LoginPage } from "../pages/LoginPage";

type AuthFixtures = {
  authenticatedPage: Page;
  loginPage: LoginPage;
};

export const test = base.extend<AuthFixtures>({
  loginPage: async ({ page }, use) => {
    const loginPage = new LoginPage(page);
    await use(loginPage);
  },

  authenticatedPage: async ({ page }, use) => {
    // Set up authenticated state
    await page.goto("/login");
    await page.getByLabel("Email").fill("test@example.com");
    await page.getByLabel("Password").fill("password123");
    await page.getByRole("button", { name: "Sign in" }).click();
    await page.waitForURL("/dashboard");

    await use(page);
  },
});

// Or use storage state for faster auth
export const test = base.extend<AuthFixtures>({
  authenticatedPage: async ({ browser }, use) => {
    const context = await browser.newContext({
      storageState: "e2e/.auth/user.json",
    });
    const page = await context.newPage();
    await use(page);
    await context.close();
  },
});
```

**Test Data Factories:**

```typescript
// e2e/fixtures/factories.ts
import { faker } from "@faker-js/faker";

export const UserFactory = {
  create(overrides = {}) {
    return {
      email: faker.internet.email(),
      password: faker.internet.password({ length: 12 }),
      firstName: faker.person.firstName(),
      lastName: faker.person.lastName(),
      ...overrides,
    };
  },

  createAdmin(overrides = {}) {
    return this.create({ role: "admin", ...overrides });
  },
};

export const ProductFactory = {
  create(overrides = {}) {
    return {
      name: faker.commerce.productName(),
      price: parseFloat(faker.commerce.price()),
      description: faker.commerce.productDescription(),
      sku: faker.string.alphanumeric(8).toUpperCase(),
      ...overrides,
    };
  },
};
```

**Database Seeding:**

```typescript
// e2e/fixtures/database.ts
import { test as base } from "@playwright/test";
import { prisma } from "../../src/lib/prisma";
import { UserFactory, ProductFactory } from "./factories";

export const test = base.extend({
  testUser: async ({}, use) => {
    const userData = UserFactory.create();
    const user = await prisma.user.create({ data: userData });

    await use(user);

    // Cleanup after test
    await prisma.user.delete({ where: { id: user.id } });
  },

  seededProducts: async ({}, use) => {
    const products = await Promise.all(
      Array.from({ length: 5 }, () =>
        prisma.product.create({ data: ProductFactory.create() }),
      ),
    );

    await use(products);

    await prisma.product.deleteMany({
      where: { id: { in: products.map((p) => p.id) } },
    });
  },
});
```

### 4. Apply Selector Strategies

**Selector Priority (Best to Worst):**

1. Accessibility roles and labels
2. data-testid attributes
3. Text content
4. CSS selectors
5. XPath (avoid)

**Playwright Selector Examples:**

```typescript
// Preferred: Accessibility-based selectors
page.getByRole("button", { name: "Submit" });
page.getByRole("textbox", { name: "Email" });
page.getByRole("link", { name: "Learn more" });
page.getByLabel("Password");
page.getByPlaceholder("Enter your email");
page.getByText("Welcome back");

// Good: Test IDs for complex elements
page.getByTestId("user-avatar");
page.getByTestId("product-card-123");

// Acceptable: CSS for structural selection
page.locator("table tbody tr:first-child");
page.locator(".modal-content");

// Chaining locators
page
  .getByTestId("product-list")
  .getByRole("listitem")
  .filter({ hasText: "Widget" })
  .getByRole("button", { name: "Add to cart" });
```

**Adding Test IDs to Components:**

```tsx
// React component with test IDs
function ProductCard({ product }: { product: Product }) {
  return (
    <div data-testid={`product-card-${product.id}`}>
      <h3 data-testid="product-name">{product.name}</h3>
      <span data-testid="product-price">${product.price}</span>
      <button data-testid="add-to-cart-btn">Add to Cart</button>
    </div>
  );
}

// Strip test IDs in production
// babel.config.js
module.exports = {
  env: {
    production: {
      plugins: [["react-remove-properties", { properties: ["data-testid"] }]],
    },
  },
};
```

### 5. Handle Async Operations and Waits

**Auto-waiting in Playwright:**

```typescript
// Playwright auto-waits for actionability
await page.getByRole("button").click(); // Waits for visible, enabled, stable

// Explicit waits when needed
await page.waitForURL("/dashboard");
await page.waitForResponse("/api/users");
await page.waitForLoadState("networkidle");

// Wait for specific conditions
await expect(page.getByTestId("loading")).toBeHidden();
await expect(page.getByRole("table")).toBeVisible();
```

**Network Request Handling:**

```typescript
// Wait for API response
const responsePromise = page.waitForResponse("/api/products");
await page.getByRole("button", { name: "Load Products" }).click();
const response = await responsePromise;
expect(response.status()).toBe(200);

// Mock API responses
await page.route("/api/products", async (route) => {
  await route.fulfill({
    status: 200,
    contentType: "application/json",
    body: JSON.stringify([{ id: 1, name: "Mocked Product" }]),
  });
});

// Intercept and modify
await page.route("/api/user", async (route) => {
  const response = await route.fetch();
  const json = await response.json();
  json.isAdmin = true;
  await route.fulfill({ response, json });
});
```

**Handling Loading States:**

```typescript
// Wait for loading to complete
async function waitForDataLoad(page: Page) {
  // Option 1: Wait for loading indicator to disappear
  await page.getByTestId("loading-spinner").waitFor({ state: "hidden" });

  // Option 2: Wait for data to appear
  await expect(page.getByRole("table")).toHaveCount(1);

  // Option 3: Wait for network idle
  await page.waitForLoadState("networkidle");
}
```

### 6. Implement Visual Regression Testing

**Playwright Visual Comparisons:**

```typescript
// Basic screenshot comparison
test("homepage visual", async ({ page }) => {
  await page.goto("/");
  await expect(page).toHaveScreenshot("homepage.png");
});

// Component screenshot
test("button states", async ({ page }) => {
  await page.goto("/components/button");

  const button = page.getByRole("button", { name: "Click me" });
  await expect(button).toHaveScreenshot("button-default.png");

  await button.hover();
  await expect(button).toHaveScreenshot("button-hover.png");
});

// Full page with options
test("full page visual", async ({ page }) => {
  await page.goto("/dashboard");
  await expect(page).toHaveScreenshot("dashboard.png", {
    fullPage: true,
    mask: [page.getByTestId("dynamic-timestamp")],
    maxDiffPixelRatio: 0.01,
  });
});
```

**Visual Testing Configuration:**

```typescript
// playwright.config.ts
export default defineConfig({
  expect: {
    toHaveScreenshot: {
      maxDiffPixels: 100,
      maxDiffPixelRatio: 0.01,
      threshold: 0.2,
      animations: "disabled",
    },
  },
  use: {
    // Consistent viewport for visual tests
    viewport: { width: 1280, height: 720 },
  },
});
```

**Handling Dynamic Content:**

```typescript
// Mask dynamic elements
await expect(page).toHaveScreenshot({
  mask: [
    page.getByTestId("current-date"),
    page.getByTestId("user-avatar"),
    page.locator(".advertisement"),
  ],
});

// Freeze animations and time
await page.emulateMedia({ reducedMotion: "reduce" });
await page.clock.setFixedTime(new Date("2024-01-15T10:00:00"));
```

### 7. Prevent Flaky Tests

**Common Flakiness Causes and Solutions:**

```typescript
// BAD: Race condition with timing
await page.click("#submit");
await page.waitForTimeout(2000); // Arbitrary wait
expect(await page.textContent(".result")).toBe("Success");

// GOOD: Wait for actual condition
await page.click("#submit");
await expect(page.getByText("Success")).toBeVisible();
```

```typescript
// BAD: Dependent on element order
const items = await page.locator(".list-item").all();
await items[2].click(); // Index might change

// GOOD: Select by content
await page.getByRole("listitem").filter({ hasText: "Target Item" }).click();
```

```typescript
// BAD: Not waiting for navigation
await page.click('a[href="/dashboard"]');
await expect(page.locator(".dashboard")).toBeVisible();

// GOOD: Explicit navigation wait
await page.click('a[href="/dashboard"]');
await page.waitForURL("/dashboard");
await expect(page.locator(".dashboard")).toBeVisible();
```

**Test Isolation:**

```typescript
// Each test should start fresh
test.beforeEach(async ({ page }) => {
  // Clear storage
  await page.context().clearCookies();
  await page.evaluate(() => localStorage.clear());

  // Reset to known state
  await page.goto("/");
});

// Use unique data per test
test("create user", async ({ page }) => {
  const uniqueEmail = `test-${Date.now()}@example.com`;
  // ...
});
```

**Retry Strategies:**

```typescript
// playwright.config.ts
export default defineConfig({
  retries: process.env.CI ? 2 : 0,
  use: {
    trace: "on-first-retry", // Capture trace on retry
  },
});

// Test-specific retry
test("potentially flaky test", async ({ page }) => {
  test.info().annotations.push({ type: "retries", description: "3" });
  // ...
});
```

**Debugging Flaky Tests:**

```typescript
// Enable tracing
await context.tracing.start({ screenshots: true, snapshots: true });
// ... run test
await context.tracing.stop({ path: "trace.zip" });

// View trace
// npx playwright show-trace trace.zip

// Add debugging pauses
await page.pause(); // Opens inspector
```

### 8. Implement Playwright-Specific Patterns

**Playwright Advanced Features:**

```typescript
// Multiple contexts (parallel sessions)
test("multiple users", async ({ browser }) => {
  const userContext = await browser.newContext();
  const adminContext = await browser.newContext();

  const userPage = await userContext.newPage();
  const adminPage = await adminContext.newPage();

  await userPage.goto("/");
  await adminPage.goto("/admin");

  // Test interactions between users
  await adminPage.getByRole("button", { name: "Broadcast" }).click();
  await expect(userPage.getByRole("alert")).toBeVisible();

  await userContext.close();
  await adminContext.close();
});

// Mobile emulation
test("mobile navigation", async ({ page }) => {
  await page.setViewportSize({ width: 375, height: 667 });
  await page.goto("/");

  // Mobile menu should be visible
  await expect(page.getByRole("button", { name: "Menu" })).toBeVisible();
});

// Geolocation testing
test("location-based features", async ({ context, page }) => {
  await context.setGeolocation({ latitude: 37.7749, longitude: -122.4194 });
  await context.grantPermissions(["geolocation"]);

  await page.goto("/");
  await expect(page.getByText("San Francisco")).toBeVisible();
});

// Network offline mode
test("offline behavior", async ({ context, page }) => {
  await page.goto("/");
  await context.setOffline(true);

  await page.reload();
  await expect(page.getByText("You are offline")).toBeVisible();
});
```

**Playwright API Request Context:**

```typescript
// API-level authentication for faster setup
test.beforeAll(async ({ request }) => {
  // Create user via API
  const response = await request.post("/api/users", {
    data: { email: "test@example.com", password: "secure123" },
  });
  expect(response.ok()).toBeTruthy();
});

// Hybrid API + UI testing
test("order creation", async ({ page, request }) => {
  // Setup via API (fast)
  await request.post("/api/cart/add", {
    data: { productId: "123", quantity: 2 },
  });

  // Verify via UI (user-facing)
  await page.goto("/cart");
  await expect(page.getByTestId("cart-item")).toHaveCount(1);
  await expect(page.getByTestId("quantity")).toHaveText("2");
});
```

### 9. Apply QA Best Practices

**Test Pyramid Strategy:**

```text
         E2E (5-10%)      ← Smoke tests, critical paths
       Integration (20-30%) ← Component integration
      Unit Tests (60-75%)  ← Business logic, utilities
```

**Smoke Test Suite (Must-Pass Before Release):**

```typescript
// e2e/smoke/critical-paths.spec.ts
test.describe("Smoke Tests", () => {
  test("homepage loads", async ({ page }) => {
    await page.goto("/");
    await expect(page).toHaveTitle(/Home/);
    await expect(page.getByRole("navigation")).toBeVisible();
  });

  test("user can sign in", async ({ page }) => {
    await page.goto("/login");
    await page.getByLabel("Email").fill("user@example.com");
    await page.getByLabel("Password").fill("password123");
    await page.getByRole("button", { name: "Sign in" }).click();
    await expect(page).toHaveURL("/dashboard");
  });

  test("critical API endpoints respond", async ({ request }) => {
    const endpoints = ["/api/health", "/api/products", "/api/user"];
    for (const endpoint of endpoints) {
      const response = await request.get(endpoint);
      expect(response.status()).toBeLessThan(500);
    }
  });
});
```

**Acceptance Testing Patterns:**

```typescript
// e2e/acceptance/user-stories.spec.ts
test.describe("User Story: Purchase Flow", () => {
  test("As a customer, I want to buy a product so I can receive it at home", async ({
    page,
  }) => {
    // Given I am on the product page
    await page.goto("/products/widget-123");

    // When I add the product to cart
    await page.getByRole("button", { name: "Add to Cart" }).click();

    // And I proceed to checkout
    await page.getByRole("link", { name: "Checkout" }).click();

    // And I fill in my shipping details
    await page.getByLabel("Address").fill("123 Main St");
    await page.getByLabel("City").fill("Anytown");

    // And I complete payment
    await page.getByLabel("Card number").fill("4242424242424242");
    await page.getByRole("button", { name: "Place Order" }).click();

    // Then I should see order confirmation
    await expect(page.getByText("Thank you for your order")).toBeVisible();
    await expect(page.getByTestId("order-number")).toBeVisible();
  });
});
```

**Cross-Browser Testing Strategy:**

```typescript
// playwright.config.ts
export default defineConfig({
  projects: [
    // Desktop browsers
    { name: "chromium", use: { ...devices["Desktop Chrome"] } },
    { name: "firefox", use: { ...devices["Desktop Firefox"] } },
    { name: "webkit", use: { ...devices["Desktop Safari"] } },

    // Mobile browsers
    { name: "mobile-chrome", use: { ...devices["Pixel 5"] } },
    { name: "mobile-safari", use: { ...devices["iPhone 13"] } },

    // Branded browsers (if needed)
    { name: "edge", use: { ...devices["Desktop Edge"], channel: "msedge" } },
    {
      name: "chrome",
      use: { ...devices["Desktop Chrome"], channel: "chrome" },
    },
  ],
});

// Run critical tests on all browsers, others on Chrome only
test.describe("Critical Flow", () => {
  test("checkout works", async ({ page, browserName }) => {
    // Runs on all browsers
  });
});

test.describe("Admin Panel", () => {
  test.skip(({ browserName }) => browserName !== "chromium");

  test("bulk operations", async ({ page }) => {
    // Only runs on Chrome for speed
  });
});
```

**Test Observability and Reporting:**

```typescript
// Custom test reporter for CI
// playwright.config.ts
export default defineConfig({
  reporter: [
    ["html", { outputFolder: "test-results/html" }],
    ["junit", { outputFile: "test-results/junit.xml" }],
    ["json", { outputFile: "test-results/results.json" }],
    ["./custom-reporter.ts"], // Custom Slack/Teams notifications
  ],

  use: {
    trace: "retain-on-failure", // Keep traces for failed tests
    video: "retain-on-failure",
    screenshot: "only-on-failure",
  },
});

// Custom reporter example
class CustomReporter {
  onTestEnd(test, result) {
    if (result.status === "failed") {
      // Send notification to Slack/Teams
      // Attach trace URL, screenshot
    }
  }

  onEnd(result) {
    const passRate = (result.passed / result.total) * 100;
    // Send summary dashboard
  }
}
```

## Best Practices

1. **Keep Tests Independent**
   - No shared state between tests
   - Each test sets up and tears down its own data
   - Tests can run in any order
   - Use database transactions or isolated test databases

2. **Use Descriptive Test Names**

   ```typescript
   // Good - describes user behavior and expected outcome
   test('user sees error message when submitting empty form', ...);
   test('admin can delete user from management panel', ...);

   // Bad - too vague
   test('form validation', ...);
   test('delete user', ...);
   ```

3. **Follow AAA Pattern (Arrange-Act-Assert)**

   ```typescript
   test("product added to cart", async ({ page }) => {
     // Arrange - setup initial state
     await page.goto("/products");

     // Act - perform user action
     await page
       .getByTestId("product-1")
       .getByRole("button", { name: "Add" })
       .click();

     // Assert - verify expected outcome
     await expect(page.getByTestId("cart-count")).toHaveText("1");
   });
   ```

4. **Minimize Test Scope**
   - Test one user flow per test
   - Break complex flows into smaller tests
   - Use fixtures for common setup
   - Avoid testing multiple scenarios in one test

5. **Handle Flakiness Proactively**
   - Review and fix flaky tests immediately (broken window theory)
   - Use proper waits, never arbitrary timeouts
   - Isolate tests from external dependencies
   - Mock unstable third-party services
   - Use auto-retry only as a temporary measure

6. **Maintain Test Data**
   - Use factories for consistent test data
   - Clean up after tests (avoid polluting database)
   - Avoid hardcoded IDs or values
   - Use unique identifiers (timestamps, UUIDs) when needed

7. **Prioritize Test Maintenance**
   - Refactor tests when code changes
   - Remove obsolete tests
   - Keep Page Objects in sync with UI changes
   - Review test failures in CI immediately

8. **Optimize Test Execution Speed**
   - Run tests in parallel when possible
   - Use API setup instead of UI for test data
   - Skip unnecessary navigation between tests
   - Use storage state for authentication
   - Group similar tests to share setup

## Examples

### Example: Complete E2E Test Suite

```typescript
// e2e/checkout.spec.ts
import { test, expect } from "@playwright/test";
import { App } from "./pages";
import { UserFactory, ProductFactory } from "./fixtures/factories";

test.describe("Checkout Flow", () => {
  let app: App;

  test.beforeEach(async ({ page }) => {
    app = new App(page);
  });

  test("guest user can complete checkout", async ({ page }) => {
    // Navigate to product
    await page.goto("/products");
    await page
      .getByTestId("product-card")
      .first()
      .getByRole("button", { name: "Add to Cart" })
      .click();

    // Verify cart updated
    await expect(page.getByTestId("cart-count")).toHaveText("1");

    // Go to checkout
    await page.getByRole("link", { name: "Checkout" }).click();
    await page.waitForURL("/checkout");

    // Fill shipping info
    await page.getByLabel("Email").fill("guest@example.com");
    await page.getByLabel("Address").fill("123 Test St");
    await page.getByLabel("City").fill("Test City");
    await page.getByRole("button", { name: "Continue" }).click();

    // Fill payment (test mode)
    await page.getByLabel("Card number").fill("4242424242424242");
    await page.getByLabel("Expiry").fill("12/25");
    await page.getByLabel("CVC").fill("123");

    // Complete order
    await page.getByRole("button", { name: "Place Order" }).click();

    // Verify success
    await expect(
      page.getByRole("heading", { name: "Order Confirmed" }),
    ).toBeVisible();
    await expect(page.getByTestId("order-number")).toBeVisible();
  });

  test("shows validation errors for invalid payment", async ({ page }) => {
    // Setup: Add item and go to payment
    await page.goto("/checkout?items=product-1");
    await page.getByLabel("Email").fill("test@example.com");
    await page.getByRole("button", { name: "Continue" }).click();

    // Enter invalid card
    await page.getByLabel("Card number").fill("1234567890123456");
    await page.getByRole("button", { name: "Place Order" }).click();

    // Verify error
    await expect(page.getByRole("alert")).toContainText("Invalid card number");
  });
});
```

### Example: API Mocking for Edge Cases

```typescript
// e2e/error-handling.spec.ts
import { test, expect } from "@playwright/test";

test.describe("Error Handling", () => {
  test("shows friendly error when API fails", async ({ page }) => {
    // Mock API failure
    await page.route("/api/products", (route) =>
      route.fulfill({ status: 500, body: "Internal Server Error" }),
    );

    await page.goto("/products");

    await expect(page.getByRole("alert")).toContainText(
      "Unable to load products. Please try again.",
    );
    await expect(page.getByRole("button", { name: "Retry" })).toBeVisible();
  });

  test("handles network timeout gracefully", async ({ page }) => {
    // Simulate slow network
    await page.route("/api/products", async (route) => {
      await new Promise((resolve) => setTimeout(resolve, 30000));
      await route.continue();
    });

    await page.goto("/products");

    await expect(page.getByText("Loading...")).toBeVisible();
    // Verify timeout handling after reasonable wait
  });
});
```
