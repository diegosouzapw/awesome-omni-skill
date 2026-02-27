---
name: webapp-testing
description: Test local web applications using Playwright for UI verification, debugging, screenshot capture, and end-to-end testing. Use when testing web apps, verifying UI behavior, or capturing screenshots of running applications.
---

# Webapp Testing Skill

## Purpose
Test local and deployed web applications using Playwright for comprehensive UI verification, debugging, and automated testing.

## Setup

### Install Playwright
```bash
npm init playwright@latest
# or
pnpm create playwright
```

### Project Structure
```
tests/
├── e2e/
│   ├── auth.spec.ts
│   ├── dashboard.spec.ts
│   └── orders.spec.ts
├── fixtures/
│   └── test-data.ts
└── playwright.config.ts
```

## Configuration

```typescript
// playwright.config.ts
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
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
    { name: 'webkit', use: { ...devices['Desktop Safari'] } },
    { name: 'mobile', use: { ...devices['iPhone 14'] } },
  ],
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

## Test Patterns

### Authentication Tests
```typescript
import { test, expect } from '@playwright/test';

test.describe('Authentication', () => {
  test('should login successfully', async ({ page }) => {
    await page.goto('/login');
    
    await page.getByLabel('Email').fill('user@example.com');
    await page.getByLabel('Password').fill('password123');
    await page.getByRole('button', { name: 'Sign In' }).click();
    
    await expect(page).toHaveURL('/dashboard');
    await expect(page.getByRole('heading', { name: 'Dashboard' })).toBeVisible();
  });

  test('should show error for invalid credentials', async ({ page }) => {
    await page.goto('/login');
    
    await page.getByLabel('Email').fill('invalid@example.com');
    await page.getByLabel('Password').fill('wrongpassword');
    await page.getByRole('button', { name: 'Sign In' }).click();
    
    await expect(page.getByText('Invalid credentials')).toBeVisible();
  });

  test('should logout successfully', async ({ page }) => {
    // Login first
    await page.goto('/login');
    await page.getByLabel('Email').fill('user@example.com');
    await page.getByLabel('Password').fill('password123');
    await page.getByRole('button', { name: 'Sign In' }).click();
    
    // Logout
    await page.getByRole('button', { name: 'Logout' }).click();
    await expect(page).toHaveURL('/login');
  });
});
```

### Dashboard Tests
```typescript
test.describe('Dashboard', () => {
  test.beforeEach(async ({ page }) => {
    // Login before each test
    await page.goto('/login');
    await page.getByLabel('Email').fill('user@example.com');
    await page.getByLabel('Password').fill('password123');
    await page.getByRole('button', { name: 'Sign In' }).click();
  });

  test('should display stats cards', async ({ page }) => {
    await expect(page.getByTestId('stats-card-sales')).toBeVisible();
    await expect(page.getByTestId('stats-card-orders')).toBeVisible();
    await expect(page.getByTestId('stats-card-customers')).toBeVisible();
  });

  test('should filter data by date range', async ({ page }) => {
    await page.getByRole('button', { name: 'Date Range' }).click();
    await page.getByRole('option', { name: 'Last 7 days' }).click();
    
    // Wait for data to reload
    await page.waitForResponse(resp => resp.url().includes('/api/stats'));
    
    await expect(page.getByTestId('stats-card-sales')).toContainText('$');
  });

  test('should navigate to orders page', async ({ page }) => {
    await page.getByRole('link', { name: 'Orders' }).click();
    await expect(page).toHaveURL('/orders');
  });
});
```

### Form Tests
```typescript
test.describe('Order Form', () => {
  test('should create a new order', async ({ page }) => {
    await page.goto('/orders/new');
    
    // Fill form
    await page.getByLabel('Customer Name').fill('John Doe');
    await page.getByLabel('Phone').fill('+1234567890');
    await page.getByRole('combobox', { name: 'Product' }).click();
    await page.getByRole('option', { name: 'Pizza Margherita' }).click();
    await page.getByLabel('Quantity').fill('2');
    
    // Submit
    await page.getByRole('button', { name: 'Create Order' }).click();
    
    // Verify success
    await expect(page.getByText('Order created successfully')).toBeVisible();
  });

  test('should validate required fields', async ({ page }) => {
    await page.goto('/orders/new');
    await page.getByRole('button', { name: 'Create Order' }).click();
    
    await expect(page.getByText('Customer name is required')).toBeVisible();
  });
});
```

### Table Tests
```typescript
test.describe('Orders Table', () => {
  test('should sort by column', async ({ page }) => {
    await page.goto('/orders');
    
    // Click on Date column to sort
    await page.getByRole('columnheader', { name: 'Date' }).click();
    
    // Verify sort indicator
    await expect(page.getByRole('columnheader', { name: 'Date' }))
      .toHaveAttribute('aria-sort', 'ascending');
  });

  test('should paginate results', async ({ page }) => {
    await page.goto('/orders');
    
    await page.getByRole('button', { name: 'Next' }).click();
    await expect(page.getByText('Page 2')).toBeVisible();
  });

  test('should search orders', async ({ page }) => {
    await page.goto('/orders');
    
    await page.getByPlaceholder('Search orders...').fill('ORD-001');
    await page.keyboard.press('Enter');
    
    await expect(page.getByText('ORD-001')).toBeVisible();
  });
});
```

## Screenshot Capture

```typescript
test('capture dashboard screenshot', async ({ page }) => {
  await page.goto('/dashboard');
  await page.waitForLoadState('networkidle');
  
  // Full page screenshot
  await page.screenshot({ 
    path: 'screenshots/dashboard-full.png',
    fullPage: true 
  });
  
  // Element screenshot
  await page.getByTestId('sales-chart').screenshot({ 
    path: 'screenshots/sales-chart.png' 
  });
});
```

## Visual Regression

```typescript
test('visual regression - dashboard', async ({ page }) => {
  await page.goto('/dashboard');
  await expect(page).toHaveScreenshot('dashboard.png', {
    maxDiffPixels: 100,
  });
});
```

## Mobile Testing

```typescript
test.describe('Mobile', () => {
  test.use({ viewport: { width: 375, height: 667 } });

  test('should show mobile menu', async ({ page }) => {
    await page.goto('/dashboard');
    await page.getByRole('button', { name: 'Menu' }).click();
    await expect(page.getByRole('navigation')).toBeVisible();
  });
});
```

## API Mocking

```typescript
test('should handle API errors gracefully', async ({ page }) => {
  // Mock API to return error
  await page.route('**/api/orders', route => {
    route.fulfill({
      status: 500,
      body: JSON.stringify({ error: 'Server error' }),
    });
  });

  await page.goto('/orders');
  await expect(page.getByText('Failed to load orders')).toBeVisible();
});
```

## Running Tests

```bash
# Run all tests
npx playwright test

# Run specific test file
npx playwright test tests/e2e/dashboard.spec.ts

# Run in headed mode (see browser)
npx playwright test --headed

# Run with UI mode
npx playwright test --ui

# Generate report
npx playwright show-report
```

## Instructions

1. **Identify test scenarios**: What user flows need testing?
2. **Set up test fixtures**: Create reusable login, data setup
3. **Write tests**: Use page object pattern for maintainability
4. **Add screenshots**: Capture visual state for debugging
5. **Run in CI**: Add to GitHub Actions or similar
6. **Review report**: Analyze failures and fix
