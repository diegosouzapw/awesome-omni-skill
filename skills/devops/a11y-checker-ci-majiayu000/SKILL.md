---
name: a11y-checker-ci
description: Adds comprehensive accessibility testing to CI/CD pipelines using axe-core Playwright integration or pa11y-ci. Automatically generates markdown reports for pull requests showing WCAG violations with severity levels, affected elements, and remediation guidance. This skill should be used when implementing accessibility CI checks, adding a11y tests to pipelines, generating accessibility reports, enforcing WCAG compliance, automating accessibility scans, or setting up PR accessibility gates. Trigger terms include a11y ci, accessibility pipeline, wcag ci, axe-core ci, pa11y ci, accessibility reports, a11y automation, accessibility gate, compliance check.
---

# A11y Checker CI

Automated accessibility testing in CI/CD pipelines with comprehensive reporting.

## Overview

To enforce accessibility standards in continuous integration, this skill configures automated WCAG compliance checks using industry-standard tools and generates detailed reports for every pull request.

## When to Use

Use this skill when:
- Adding accessibility testing to CI/CD pipelines
- Enforcing WCAG compliance in automated builds
- Generating accessibility reports for pull requests
- Setting up quality gates based on accessibility
- Automating accessibility audits
- Tracking accessibility improvements over time
- Ensuring new features meet accessibility standards

## Supported Tools

### @axe-core/playwright

Industry-standard accessibility testing engine with Playwright integration.

**Advantages:**
- Comprehensive WCAG rule coverage
- Fast execution in parallel with E2E tests
- Detailed violation reporting
- Active maintenance and updates

### pa11y-ci

Command-line accessibility testing tool for multiple URLs.

**Advantages:**
- Simple configuration
- Standalone execution (no browser automation needed)
- Multiple URL scanning
- Custom rule configuration

## Implementation Steps

### 1. Choose Testing Approach

To select the appropriate tool:

**Use @axe-core/playwright when:**
- Already using Playwright for E2E tests
- Need integration with existing test suites
- Want to test dynamic/authenticated pages
- Require detailed test context

**Use pa11y-ci when:**
- Need simple URL-based scanning
- Want standalone accessibility checks
- Testing static pages or public URLs
- Prefer configuration-based approach

### 2. Install Dependencies

For @axe-core/playwright:
```bash
npm install -D @axe-core/playwright
```

For pa11y-ci:
```bash
npm install -D pa11y-ci
```

### 3. Create Test Configuration

#### Option A: @axe-core/playwright

Create test file using `assets/a11y-test.spec.ts`:

```typescript
import { test, expect } from '@playwright/test'
import AxeBuilder from '@axe-core/playwright'

test.describe('Accessibility Tests', () => {
  test('homepage meets WCAG standards', async ({ page }) => {
    await page.goto('/')

    const accessibilityScanResults = await new AxeBuilder({ page })
      .withTags(['wcag2a', 'wcag2aa', 'wcag21a', 'wcag21aa'])
      .analyze()

    expect(accessibilityScanResults.violations).toEqual([])
  })
})
```

#### Option B: pa11y-ci

Create configuration using `assets/pa11y-config.json`:

```json
{
  "defaults": {
    "timeout": 30000,
    "chromeLaunchConfig": {
      "executablePath": "/usr/bin/chromium-browser",
      "args": ["--no-sandbox"]
    },
    "standard": "WCAG2AA",
    "runners": ["axe", "htmlcs"],
    "ignore": []
  },
  "urls": [
    "http://localhost:3000",
    "http://localhost:3000/entities",
    "http://localhost:3000/timeline"
  ]
}
```

### 4. Generate Report Script

Create report generator using `scripts/generate_a11y_report.py`:

```bash
python scripts/generate_a11y_report.py \
  --input test-results/a11y-results.json \
  --output accessibility-report.md \
  --format github
```

The script generates markdown reports with:
- Executive summary with pass/fail status
- Violation count by severity (critical, serious, moderate, minor)
- Detailed violation list with:
  - Rule ID and description
  - WCAG criteria
  - Impact level
  - Affected elements
  - Remediation guidance
- Historical comparison (if available)

### 5. Configure CI Pipeline

#### GitHub Actions

Use template from `assets/github-actions-a11y.yml`:

```yaml
name: Accessibility Tests

on:
  pull_request:
    branches: [main, master]
  push:
    branches: [main, master]

jobs:
  a11y:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build application
        run: npm run build

      - name: Start server
        run: npm start &

      - name: Wait for server
        run: npx wait-on http://localhost:3000 -t 60000

      - name: Run accessibility tests
        run: npm run test:a11y

      - name: Generate report
        if: always()
        run: |
          python scripts/generate_a11y_report.py \
            --input test-results/a11y-results.json \
            --output accessibility-report.md \
            --format github

      - name: Comment PR
        if: github.event_name == 'pull_request' && always()
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs')
            const report = fs.readFileSync('accessibility-report.md', 'utf8')

            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: report
            })

      - name: Upload report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: accessibility-report
          path: |
            accessibility-report.md
            test-results/

      - name: Fail on violations
        if: failure()
        run: exit 1
```

#### GitLab CI

Use template from `assets/gitlab-ci-a11y.yml`:

```yaml
accessibility-test:
  stage: test
  image: mcr.microsoft.com/playwright:v1.40.0-focal
  script:
    - npm ci
    - npm run build
    - npm start &
    - npx wait-on http://localhost:3000 -t 60000
    - npm run test:a11y
    - python scripts/generate_a11y_report.py
      --input test-results/a11y-results.json
      --output accessibility-report.md
      --format gitlab
  artifacts:
    when: always
    paths:
      - accessibility-report.md
      - test-results/
    reports:
      junit: test-results/junit.xml
  only:
    - merge_requests
    - main
```

### 6. Add Package Scripts

Add to package.json:

```json
{
  "scripts": {
    "test:a11y": "playwright test a11y.spec.ts",
    "test:a11y:ci": "playwright test a11y.spec.ts --reporter=json",
    "pa11y": "pa11y-ci --config .pa11yci.json"
  }
}
```

## Report Format

### Executive Summary

```markdown
# Accessibility Test Report

**Status:** [ERROR] Failed
**Total Violations:** 12
**Pages Tested:** 5
**WCAG Level:** AA
**Date:** 2025-01-15

## Summary by Severity

- [CRITICAL] Critical: 2
- [SERIOUS] Serious: 5
- [MODERATE] Moderate: 3
- [MINOR] Minor: 2
```

### Violation Details

```markdown
## Violations

### [CRITICAL] Critical (2)

#### 1. Form elements must have labels (form-field-multiple-labels)

**WCAG Criteria:** 3.3.2 (Level A)
**Impact:** Critical
**Occurrences:** 3 elements

**Description:**
Form fields should have exactly one associated label element.

**Affected Elements:**
- Line 45: `<input type="text" name="entity-name">`
- Line 67: `<input type="email" name="user-email">`
- Line 89: `<select name="entity-type">`

**How to Fix:**
Add a `<label>` element with a `for` attribute matching the input's `id`:

\`\`\`html
<label for="entity-name">Entity Name</label>
<input id="entity-name" type="text" name="entity-name">
\`\`\`

**More Info:** https://dequeuniversity.com/rules/axe/4.7/label

---
```

### Historical Comparison

```markdown
## Progress

| Metric | Previous | Current | Change |
|--------|----------|---------|--------|
| Total Violations | 15 | 12 | [OK] -3 |
| Critical | 3 | 2 | [OK] -1 |
| Serious | 7 | 5 | [OK] -2 |
| Moderate | 4 | 3 | [OK] -1 |
| Minor | 1 | 2 | [ERROR] +1 |
```

## Quality Gates

### Blocking Violations

To fail builds on specific violations, configure thresholds:

```typescript
const results = await new AxeBuilder({ page }).analyze()

// Fail on any critical violations
const critical = results.violations.filter(v => v.impact === 'critical')
expect(critical).toHaveLength(0)

// Allow up to 5 moderate violations
const moderate = results.violations.filter(v => v.impact === 'moderate')
expect(moderate.length).toBeLessThanOrEqual(5)
```

### Configuration File

Use `assets/a11y-thresholds.json`:

```json
{
  "thresholds": {
    "critical": 0,
    "serious": 0,
    "moderate": 5,
    "minor": 10
  },
  "allowedViolations": [
    "color-contrast"
  ],
  "ignoreSelectors": [
    "#third-party-widget",
    "[data-testid='external-embed']"
  ]
}
```

## Advanced Configuration

### Custom Rules

To disable or configure specific rules:

```typescript
const results = await new AxeBuilder({ page })
  .disableRules(['color-contrast'])
  .withRules({
    'custom-rule': { enabled: true }
  })
  .analyze()
```

### Page-Specific Tests

Test different page types:

```typescript
const pages = [
  { url: '/', name: 'Homepage' },
  { url: '/entities', name: 'Entity List' },
  { url: '/timeline', name: 'Timeline View' }
]

for (const { url, name } of pages) {
  test(`${name} accessibility`, async ({ page }) => {
    await page.goto(url)
    const results = await new AxeBuilder({ page }).analyze()
    expect(results.violations).toEqual([])
  })
}
```

### Authenticated Pages

Test pages requiring authentication:

```typescript
test.use({ storageState: 'auth.json' })

test('dashboard accessibility', async ({ page }) => {
  await page.goto('/dashboard')
  const results = await new AxeBuilder({ page }).analyze()
  expect(results.violations).toEqual([])
})
```

## Report Customization

### Custom Templates

Create custom report templates in `assets/report-templates/`:

- `github-template.md` - GitHub PR comments
- `gitlab-template.md` - GitLab MR comments
- `slack-template.md` - Slack notifications
- `html-template.html` - HTML reports

### Report Destinations

Configure report distribution:

```python
python scripts/generate_a11y_report.py \
  --input results.json \
  --output-dir reports/ \
  --formats github gitlab slack html \
  --slack-webhook $SLACK_WEBHOOK \
  --github-token $GITHUB_TOKEN
```

## Monitoring and Tracking

### Historical Data

Store results for trend analysis:

```bash
# Save results with timestamp
python scripts/save_a11y_results.py \
  --input test-results/a11y-results.json \
  --database a11y-history.db

# Generate trend report
python scripts/generate_trend_report.py \
  --database a11y-history.db \
  --days 30 \
  --output a11y-trends.md
```

### Metrics Dashboard

Generate metrics for dashboards:

```json
{
  "timestamp": "2025-01-15T10:30:00Z",
  "commit": "abc123",
  "branch": "feature/new-ui",
  "violations": {
    "critical": 2,
    "serious": 5,
    "moderate": 3,
    "minor": 2
  },
  "wcagCompliance": {
    "a": false,
    "aa": false,
    "aaa": false
  },
  "pagesTested": 5,
  "totalElements": 1247,
  "testedElements": 1247
}
```

## Resources

Consult the following resources for detailed information:

- `scripts/generate_a11y_report.py` - Report generator
- `scripts/save_a11y_results.py` - Historical data storage
- `scripts/generate_trend_report.py` - Trend analysis
- `assets/a11y-test.spec.ts` - Playwright test template
- `assets/pa11y-config.json` - pa11y-ci configuration
- `assets/github-actions-a11y.yml` - GitHub Actions workflow
- `assets/gitlab-ci-a11y.yml` - GitLab CI configuration
- `assets/a11y-thresholds.json` - Violation thresholds
- `references/wcag-criteria.md` - WCAG standards reference
- `references/common-violations.md` - Common issues and fixes

## Best Practices

- Run accessibility tests on every pull request
- Set appropriate thresholds for violations
- Generate readable reports for developers
- Track accessibility metrics over time
- Test authenticated and dynamic pages
- Include accessibility in definition of done
- Review and update ignored rules periodically
- Provide remediation guidance in reports
- Celebrate accessibility improvements
