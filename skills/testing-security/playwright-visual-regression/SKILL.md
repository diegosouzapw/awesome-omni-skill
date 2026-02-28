---
name: playwright-visual-regression
description: Playwright browser automation patterns for E2E testing, visual regression, screenshot capture, and accessibility validation. Use when implementing or debugging browser-based tests.
---

# Playwright Visual Regression Testing

This skill documents patterns for E2E testing with Playwright, including visual regression, cross-browser testing, and accessibility automation for OSCAR Export Analyzer.

## Setup and Configuration

### Installation

```bash
npm install -D @playwright/test
npx playwright install  # Install browsers
```

### Configuration (`playwright.config.js`)

```javascript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests/e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',

  use: {
    baseURL: 'http://localhost:5173',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },

  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
  ],

  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:5173',
    reuseExistingServer: !process.env.CI,
  },
});
```

## Basic E2E Test

```javascript
import { test, expect } from '@playwright/test';

test.describe('CSV Upload Flow', () => {
  test('uploads CSV and displays charts', async ({ page }) => {
    await page.goto('/');

    // Upload CSV file
    const fileInput = page.locator('input[type="file"]');
    await fileInput.setInputFiles('tests/fixtures/sample-cpap-data.csv');

    // Wait for parsing to complete
    await expect(page.locator('text=Parsing complete')).toBeVisible();

    // Verify charts are rendered
    await expect(page.locator('[data-testid="usage-chart"]')).toBeVisible();
    await expect(page.locator('[data-testid="ahi-chart"]')).toBeVisible();
  });
});
```

## Visual Regression Testing

### Capture Visual Baseline

```javascript
import { test, expect } from '@playwright/test';

test.describe('Visual Regression', () => {
  test('chart layout matches baseline', async ({ page }) => {
    // Load test data
    await page.goto('/');
    await page
      .locator('input[type="file"]')
      .setInputFiles('tests/fixtures/test-data.csv');
    await page.waitForLoadState('networkidle');

    // Take screenshot and compare with baseline
    await expect(page).toHaveScreenshot('usage-chart.png', {
      maxDiffPixels: 100, // Allow small differences
    });
  });

  test('dark mode appearance', async ({ page }) => {
    await page.goto('/');

    // Enable dark mode
    await page.locator('[aria-label="Toggle dark mode"]').click();

    // Wait for theme transition
    await page.waitForTimeout(300);

    // Compare with dark mode baseline
    await expect(page).toHaveScreenshot('dark-mode.png');
  });
});
```

### Update Baselines

```bash
# Update all baselines
npx playwright test --update-snapshots

# Update specific test baselines
npx playwright test visual-regression --update-snapshots
```

### Baseline Management

- **Store baselines in git**: `tests/e2e/*.spec.js-snapshots/`
- **Review visual diffs**: Check Playwright HTML report
- **Approve changes**: Commit updated baselines after verifying correctness

## Responsive Testing

```javascript
test.describe('Responsive Design', () => {
  test('mobile layout', async ({ page }) => {
    // Set mobile viewport
    await page.setViewportSize({ width: 375, height: 667 });
    await page.goto('/');

    // Verify mobile layout
    await expect(page.locator('[data-testid="mobile-menu"]')).toBeVisible();
    await expect(page).toHaveScreenshot('mobile-layout.png');
  });

  test('tablet layout', async ({ page }) => {
    await page.setViewportSize({ width: 768, height: 1024 });
    await page.goto('/');

    await expect(page).toHaveScreenshot('tablet-layout.png');
  });

  test('desktop layout', async ({ page }) => {
    await page.setViewportSize({ width: 1920, height: 1080 });
    await page.goto('/');

    await expect(page).toHaveScreenshot('desktop-layout.png');
  });
});
```

## Chart Interaction Testing

```javascript
test.describe('Chart Interactions', () => {
  test('zoom chart with mouse wheel', async ({ page }) => {
    await page.goto('/');
    await loadTestData(page);

    const chart = page.locator('[data-testid="ahi-chart"]');

    // Get initial axis range
    const initialRange = await page.evaluate(() => {
      const plotly = window.Plotly;
      const chartDiv = document.querySelector('[data-testid="ahi-chart"]');
      return chartDiv.layout.xaxis.range;
    });

    // Zoom in with mouse wheel
    await chart.hover();
    await page.mouse.wheel(0, -100);

    // Wait for zoom animation
    await page.waitForTimeout(500);

    // Verify axis range changed
    const newRange = await page.evaluate(() => {
      const chartDiv = document.querySelector('[data-testid="ahi-chart"]');
      return chartDiv.layout.xaxis.range;
    });

    expect(newRange).not.toEqual(initialRange);
  });

  test('toggle legend series', async ({ page }) => {
    await page.goto('/');
    await loadTestData(page);

    // Click legend item to hide series
    await page.locator('.legend .traces:has-text("AHI")').click();

    // Verify series hidden
    const isVisible = await page.evaluate(() => {
      const trace = document.querySelector('.trace.scatter');
      return trace.style.opacity === '0.5';
    });

    expect(isVisible).toBe(true);
  });
});
```

## Accessibility Testing

### Built-in Accessibility Checks

```javascript
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test.describe('Accessibility', () => {
  test('page has no accessibility violations', async ({ page }) => {
    await page.goto('/');
    await loadTestData(page);

    // Run axe accessibility scan
    const accessibilityScanResults = await new AxeBuilder({ page }).analyze();

    expect(accessibilityScanResults.violations).toEqual([]);
  });
});
```

### Keyboard Navigation

```javascript
test('keyboard navigation works', async ({ page }) => {
  await page.goto('/');

  // Tab to file input
  await page.keyboard.press('Tab');
  await expect(page.locator('input[type="file"]')).toBeFocused();

  // Tab to upload button
  await page.keyboard.press('Tab');
  await expect(page.locator('button:has-text("Upload")')).toBeFocused();

  // Activate with Enter
  await page.keyboard.press('Enter');
});
```

### Screen Reader Testing

```javascript
test('has proper ARIA labels', async ({ page }) => {
  await page.goto('/');
  await loadTestData(page);

  // Check chart has accessible name
  const chartLabel = await page
    .locator('[data-testid="ahi-chart"]')
    .getAttribute('aria-label');

  expect(chartLabel).toContain('AHI Trends');

  // Check form labels
  await expect(page.locator('label[for="start-date"]')).toHaveText(
    'Start Date',
  );
});
```

## Print Layout Testing

```javascript
test.describe('Print Layout', () => {
  test('print stylesheet applied', async ({ page }) => {
    await page.goto('/');
    await loadTestData(page);

    // Emulate print media
    await page.emulateMedia({ media: 'print' });

    // Verify print-specific styles
    await expect(page).toHaveScreenshot('print-layout.png');

    // Check elements hidden in print
    const navVisible = await page.locator('nav').isVisible();
    expect(navVisible).toBe(false);
  });

  test('PDF export quality', async ({ page }) => {
    await page.goto('/');
    await loadTestData(page);

    // Trigger print dialog
    await page.locator('button:has-text("Export PDF")').click();

    // Generate PDF
    const pdf = await page.pdf({
      format: 'A4',
      printBackground: true,
    });

    // Verify PDF generated
    expect(pdf.length).toBeGreaterThan(0);
  });
});
```

## Cross-Browser Testing

```javascript
// Run same test across browsers
test.describe('Cross-Browser Compatibility', () => {
  test('works in all browsers', async ({ page, browserName }) => {
    test.skip(browserName === 'webkit', 'Known webkit issue #123');

    await page.goto('/');
    await loadTestData(page);

    // Verify core functionality works
    await expect(page.locator('[data-testid="usage-chart"]')).toBeVisible();
  });
});
```

## Large File Stress Testing

```javascript
test.describe('Performance', () => {
  test('handles large CSV upload', async ({ page }) => {
    // Increase timeout for large file
    test.setTimeout(60000);

    await page.goto('/');

    // Upload 30MB CSV
    await page
      .locator('input[type="file"]')
      .setInputFiles('tests/fixtures/large-cpap-data.csv');

    // Wait for parsing (with progress updates)
    await expect(page.locator('text=Parsing')).toBeVisible();
    await expect(page.locator('text=Parsing complete')).toBeVisible({
      timeout: 30000,
    });

    // Verify app still responsive
    await page.locator('button:has-text("Filter")').click();
    await expect(page.locator('[data-testid="date-filter"]')).toBeVisible();
  });

  test('memory usage stays reasonable', async ({ page }) => {
    await page.goto('/');
    await loadTestData(page);

    // Measure memory usage
    const metrics = await page.evaluate(() => {
      return (performance as any).memory
        ? {
            usedJSHeapSize: (performance as any).memory.usedJSHeapSize,
            totalJSHeapSize: (performance as any).memory.totalJSHeapSize,
          }
        : null;
    });

    if (metrics) {
      // Ensure memory usage under 200MB
      expect(metrics.usedJSHeapSize).toBeLessThan(200 * 1024 * 1024);
    }
  });
});
```

## Fixture Management

```javascript
// tests/e2e/fixtures/index.ts
import { test as base } from '@playwright/test';

type Fixtures = {
  loadTestData: () => Promise<void>;
};

export const test = base.extend<Fixtures>({
  loadTestData: async ({ page }, use) => {
    const loader = async () => {
      await page.goto('/');
      await page
        .locator('input[type="file"]')
        .setInputFiles('tests/fixtures/sample-cpap-data.csv');
      await page.waitForLoadState('networkidle');
    };

    await use(loader);
  },
});
```

**Usage:**

```javascript
import { test, expect } from './fixtures';

test('uses fixture', async ({ page, loadTestData }) => {
  await loadTestData();

  // Test with data already loaded
  await expect(page.locator('[data-testid="usage-chart"]')).toBeVisible();
});
```

## Resilient Selectors

### Good Selectors (Stable)

```javascript
// ✅ Data test IDs (most stable)
page.locator('[data-testid="usage-chart"]');

// ✅ Accessible roles and names
page.getByRole('button', { name: 'Upload' });
page.getByRole('textbox', { name: 'Start Date' });

// ✅ Text content (user-facing)
page.locator('text=Usage Patterns');
```

### Bad Selectors (Brittle)

```javascript
// ❌ CSS classes (change frequently)
page.locator('.chart-container-wrapper-inner');

// ❌ Deeply nested selectors
page.locator('div > div > div:nth-child(3) > span.label');

// ❌ Positional selectors
page.locator('.chart').nth(2);
```

## Documentation Automation

### Screenshot Generation for README

```javascript
test.describe('Documentation Screenshots', () => {
  test('generate README screenshots', async ({ page }) => {
    await page.goto('/');
    await loadTestData(page);

    // Full page screenshot for README hero
    await page.screenshot({
      path: 'screenshots/app-overview.png',
      fullPage: true,
    });

    // Specific chart screenshots
    await page.locator('[data-testid="usage-chart"]').screenshot({
      path: 'screenshots/usage-chart.png',
    });

    await page.locator('[data-testid="ahi-chart"]').screenshot({
      path: 'screenshots/ahi-chart.png',
    });

    // Dark mode screenshot
    await page.locator('[aria-label="Toggle dark mode"]').click();
    await page.waitForTimeout(300);
    await page.screenshot({
      path: 'screenshots/dark-mode.png',
    });
  });
});
```

## CI Integration

### GitHub Actions

```yaml
# .github/workflows/playwright.yml
name: Playwright Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: 20
      - name: Install dependencies
        run: npm ci
      - name: Install Playwright Browsers
        run: npx playwright install --with-deps
      - name: Run Playwright tests
        run: npx playwright test
      - uses: actions/upload-artifact@v3
        if: always()
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30
```

## Debugging Tests

### Interactive Mode

```bash
# Open Playwright Inspector
npx playwright test --debug

# Run in headed mode (see browser)
npx playwright test --headed

# Run specific test
npx playwright test tests/e2e/upload.spec.js --headed
```

### Trace Viewer

```bash
# Generate trace on failure (configured in playwright.config.js)
# View trace after test failure:
npx playwright show-trace trace.zip
```

### Pause Execution

```javascript
test('debug test', async ({ page }) => {
  await page.goto('/');

  // Pause execution, open inspector
  await page.pause();

  // Continue manually in inspector
});
```

## Resources

- **Playwright docs**: https://playwright.dev/
- **Axe accessibility**: https://github.com/dequelabs/axe-core-npm
- **Test fixtures**: `tests/e2e/fixtures/`
- **Config**: `playwright.config.js`
- **CI workflow**: `.github/workflows/playwright.yml`
