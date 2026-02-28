---
model: haiku
created: 2025-12-16
modified: 2026-02-11
reviewed: 2025-12-16
description: Check and configure UX testing infrastructure (Playwright, accessibility, visual regression)
allowed-tools: Glob, Grep, Read, Write, Edit, Bash, AskUserQuestion, TodoWrite, WebSearch, WebFetch
args: "[--check-only] [--fix] [--a11y] [--visual]"
argument-hint: "[--check-only] [--fix] [--a11y] [--visual]"
name: configure-ux-testing
---

# /configure:ux-testing

Check and configure UX testing infrastructure with Playwright as the primary tool for E2E, accessibility, and visual regression testing.

## When to Use This Skill

| Use this skill when... | Use another approach when... |
|------------------------|------------------------------|
| Setting up Playwright E2E testing infrastructure for a project | Running existing Playwright tests (use `bun test:e2e` or test-runner agent) |
| Adding accessibility testing with axe-core to a project | Performing manual accessibility audits on a live site |
| Configuring visual regression testing with screenshot assertions | Debugging a specific failing E2E test (use system-debugging agent) |
| Setting up Playwright MCP server for Claude browser automation | Writing individual test cases (use playwright-testing skill) |
| Creating CI/CD workflows for E2E and accessibility test execution | Configuring unit or integration tests (use `/configure:tests`) |

## Context

- Package manager: !`find . -maxdepth 1 \( -name 'package.json' -o -name 'bun.lockb' \)`
- Playwright config: !`find . -maxdepth 1 -name 'playwright.config.*'`
- Playwright installed: !`grep -l '@playwright/test' package.json`
- Axe-core installed: !`grep -l '@axe-core/playwright' package.json`
- E2E test dir: !`find . -maxdepth 2 -type d \( -name 'e2e' -o -name 'tests' \)`
- Visual snapshots: !`find . -maxdepth 4 -type d -name '__snapshots__'`
- MCP config: !`find . -maxdepth 1 -name '.mcp.json'`
- CI workflow: !`find .github/workflows -maxdepth 1 -name 'e2e*'`

**UX Testing Stack:**
- **Playwright** - Cross-browser E2E testing (primary tool)
- **axe-core** - Automated accessibility testing (WCAG compliance)
- **Playwright screenshots** - Visual regression testing
- **Playwright MCP** - Browser automation via MCP integration

## Parameters

Parse from command arguments:

- `--check-only`: Report status without offering fixes
- `--fix`: Apply all fixes automatically without prompting
- `--a11y`: Focus on accessibility testing configuration
- `--visual`: Focus on visual regression testing configuration

## Execution

Execute this UX testing configuration check:

### Step 1: Fetch latest tool versions

Verify latest versions before configuring:

1. **@playwright/test**: Check [playwright.dev](https://playwright.dev/) or [npm](https://www.npmjs.com/package/@playwright/test)
2. **@axe-core/playwright**: Check [npm](https://www.npmjs.com/package/@axe-core/playwright)
3. **playwright MCP**: Check [npm](https://www.npmjs.com/package/@anthropic/mcp-server-playwright)

Use WebSearch or WebFetch to verify current versions.

### Step 2: Detect existing UX testing infrastructure

Check for each component:

| Indicator | Component | Status |
|-----------|-----------|--------|
| `playwright.config.*` | Playwright | Installed |
| `@axe-core/playwright` in package.json | Accessibility testing | Configured |
| `@playwright/test` in package.json | Playwright Test | Installed |
| `tests/e2e/` or `e2e/` directory | E2E tests | Present |
| `*.spec.ts` files with toHaveScreenshot | Visual regression | Configured |
| `.mcp.json` with playwright server | Playwright MCP | Configured |

### Step 3: Analyze current testing state

Check for complete UX testing setup across four areas:

**Playwright Core:**
- `@playwright/test` installed
- `playwright.config.ts` exists
- Browser projects configured (Chromium, Firefox, WebKit)
- Mobile viewports configured (optional)
- WebServer configuration for local dev
- Trace/screenshot/video on failure

**Accessibility Testing:**
- `@axe-core/playwright` installed
- Accessibility tests created
- WCAG level configured (A, AA, AAA)
- Custom rules/exceptions documented

**Visual Regression:**
- Screenshot assertions configured
- Snapshot directory configured
- Update workflow documented
- CI snapshot handling configured

**MCP Integration:**
- Playwright MCP server in `.mcp.json`
- Browser automation available to Claude

### Step 4: Generate compliance report

Print a formatted compliance report showing status for Playwright core, accessibility testing, visual regression, and MCP integration.

If `--check-only` is set, stop here.

For the compliance report format, see [REFERENCE.md](REFERENCE.md).

### Step 5: Install dependencies (if --fix or user confirms)

```bash
# Core Playwright
bun add --dev @playwright/test

# Accessibility testing
bun add --dev @axe-core/playwright

# Install browsers
bunx playwright install
```

### Step 6: Create Playwright configuration

Create `playwright.config.ts` with:
- Desktop browser projects (Chromium, Firefox, WebKit)
- Mobile viewport projects (Pixel 5, iPhone 13)
- Dedicated a11y test project (Chromium only)
- WebServer auto-start for local dev
- Trace/screenshot/video on failure settings
- JSON and JUnit reporters for CI

For the complete `playwright.config.ts` template, see [REFERENCE.md](REFERENCE.md).

### Step 7: Create accessibility test helper

Create `tests/e2e/helpers/a11y.ts` with:
- `expectNoA11yViolations(page, options)` - Assert no WCAG violations
- `getA11yReport(page, options)` - Generate detailed a11y report
- Configurable WCAG level (wcag2a, wcag2aa, wcag21aa, wcag22aa)
- Rule include/exclude support
- Formatted violation output

For the complete a11y helper code, see [REFERENCE.md](REFERENCE.md).

### Step 8: Create example test files

Create example tests:

1. **`tests/e2e/homepage.a11y.spec.ts`** - Homepage accessibility tests (WCAG 2.1 AA violations, post-interaction checks, full report)
2. **`tests/e2e/visual.spec.ts`** - Visual regression tests (full page screenshots, component screenshots, responsive layouts, dark mode)

For complete example test files, see [REFERENCE.md](REFERENCE.md).

### Step 9: Add npm scripts

Update `package.json` with test scripts:

```json
{
  "scripts": {
    "test:e2e": "playwright test",
    "test:e2e:headed": "playwright test --headed",
    "test:e2e:debug": "playwright test --debug",
    "test:e2e:ui": "playwright test --ui",
    "test:a11y": "playwright test --project=a11y",
    "test:visual": "playwright test visual.spec.ts",
    "test:visual:update": "playwright test visual.spec.ts --update-snapshots",
    "playwright:codegen": "playwright codegen http://localhost:3000",
    "playwright:report": "playwright show-report"
  }
}
```

### Step 10: Configure MCP integration (optional)

Add to `.mcp.json`:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "bunx",
      "args": ["-y", "@playwright/mcp@latest"]
    }
  }
}
```

This enables Claude to navigate web pages, take screenshots, fill forms, click elements, and capture accessibility snapshots.

### Step 11: Create CI/CD workflow

Create `.github/workflows/e2e.yml` with parallel jobs for:
- E2E tests (all browsers)
- Accessibility tests (Chromium only)
- Artifact upload for reports and failure screenshots

For the complete CI workflow template, see [REFERENCE.md](REFERENCE.md).

### Step 12: Update standards tracking

Update `.project-standards.yaml`:

```yaml
components:
  ux_testing: "2025.1"
  ux_testing_framework: "playwright"
  ux_testing_a11y: true
  ux_testing_a11y_level: "wcag21aa"
  ux_testing_visual: true
  ux_testing_mcp: true
```

### Step 13: Report configuration results

Print a summary of configuration applied, scripts added, and CI/CD setup. Include next steps for starting the dev server, running tests, updating snapshots, and opening the interactive UI.

For the results report format, see [REFERENCE.md](REFERENCE.md).

## Agentic Optimizations

| Context | Command |
|---------|---------|
| Quick compliance check | `/configure:ux-testing --check-only` |
| Auto-fix all issues | `/configure:ux-testing --fix` |
| Accessibility focus only | `/configure:ux-testing --a11y` |
| Visual regression focus only | `/configure:ux-testing --visual` |
| Run E2E tests compact | `bunx playwright test --reporter=line` |
| Run a11y tests only | `bunx playwright test --project=a11y --reporter=dot` |

## Flags

| Flag | Description |
|------|-------------|
| `--check-only` | Report status without offering fixes |
| `--fix` | Apply all fixes automatically without prompting |
| `--a11y` | Focus on accessibility testing configuration |
| `--visual` | Focus on visual regression testing configuration |

## Error Handling

- **No package manager found**: Cannot install dependencies, provide manual steps
- **Dev server not configured**: Warn about manual baseURL configuration
- **Browsers not installed**: Prompt to run `bunx playwright install`
- **Existing config conflicts**: Preserve user config, suggest merge

## See Also

- `/configure:tests` - Unit and integration testing configuration
- `/configure:all` - Run all compliance checks
- **Skills**: `playwright-testing`, `accessibility-implementation`
- **Agents**: `ux-implementation` for implementing UX designs
- **Playwright documentation**: https://playwright.dev
- **axe-core documentation**: https://www.deque.com/axe
