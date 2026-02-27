---
name: github-actions
description: Debug, optimize, and secure GitHub Actions workflows. Use this skill when writing CI/CD pipelines, fixing failing workflows, or improving build times.
alwaysApply: false
---

# GitHub Actions

You are a CI/CD expert specializing in GitHub Actions. Apply these patterns when writing, debugging, or optimizing workflows.

## Workflow Structure Best Practices

### Standard CI Workflow
```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm
      - run: npm ci
      - run: npm run lint

  test:
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm
      - run: npm ci
      - run: npm test

  build:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm
      - run: npm ci
      - run: npm run build
```

## Speed Optimization

### 1. Cache Dependencies
```yaml
# Node.js — built-in cache
- uses: actions/setup-node@v4
  with:
    node-version: 22
    cache: npm

# Go — built-in cache
- uses: actions/setup-go@v5
  with:
    go-version: '1.23'
    cache: true

# Generic caching
- uses: actions/cache@v4
  with:
    path: ~/.cache/pip
    key: ${{ runner.os }}-pip-${{ hashFiles('**/requirements.txt') }}
    restore-keys: ${{ runner.os }}-pip-
```

### 2. Parallel Jobs
```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
  test-unit:
    runs-on: ubuntu-latest
  test-integration:
    runs-on: ubuntu-latest
  # All three run in parallel

  deploy:
    needs: [lint, test-unit, test-integration]
    # Only runs after all three succeed
```

### 3. Skip Unnecessary Runs
```yaml
on:
  push:
    paths-ignore:
      - '**.md'
      - 'docs/**'
      - '.github/ISSUE_TEMPLATE/**'
    branches: [main]
```

### 4. Use Concurrency
```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
# Cancels older runs when new commits push to same branch
```

### 5. Matrix Builds (When Needed)
```yaml
strategy:
  matrix:
    node: [20, 22]
    os: [ubuntu-latest, macos-latest]
  fail-fast: true  # Stop all if one fails
```

## Debugging Failed Workflows

### Common Failures and Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| `Permission denied` | Missing `permissions` block | Add `permissions: contents: read` |
| `Node.js 16 deprecation` | Old action version | Update to `@v4` |
| `npm ci` fails | `package-lock.json` out of sync | Run `npm install` locally, commit lock file |
| `GITHUB_TOKEN` unauthorized | Insufficient permissions | Add `permissions:` block with needed scopes |
| Cache miss every time | Bad cache key | Use `hashFiles()` on lock files |
| Timeout | Long-running tests | Add `timeout-minutes: 15` to job |

### Debug Techniques
```yaml
# Enable debug logging
# Set repository secret: ACTIONS_STEP_DEBUG = true

# Print environment
- run: env | sort

# Print context
- run: echo '${{ toJSON(github) }}'

# SSH into runner (for emergency debugging)
- uses: mxschmitt/action-tmate@v3
  if: failure()
```

## Security

### Pin Action Versions by SHA
```yaml
# BAD — mutable tag
- uses: actions/checkout@v4

# GOOD — immutable SHA
- uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11 # v4.1.1
```

### Limit Permissions
```yaml
permissions:
  contents: read      # Only what's needed
  pull-requests: write # Only if needed

# Never use:
# permissions: write-all
```

### Protect Secrets
```yaml
# Use environment protection rules for production secrets
jobs:
  deploy:
    environment: production  # Requires approval
    steps:
      - run: deploy.sh
        env:
          API_KEY: ${{ secrets.PROD_API_KEY }}
```

### Don't Trust PR Input
```yaml
# BAD — PR title could contain injection
- run: echo "PR: ${{ github.event.pull_request.title }}"

# GOOD — use environment variable
- run: echo "PR: $PR_TITLE"
  env:
    PR_TITLE: ${{ github.event.pull_request.title }}
```

## Reusable Workflows

### Create a Reusable Workflow
```yaml
# .github/workflows/reusable-test.yml
name: Reusable Test
on:
  workflow_call:
    inputs:
      node-version:
        type: string
        default: '22'
    secrets:
      NPM_TOKEN:
        required: false

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
      - run: npm ci
      - run: npm test
```

### Use a Reusable Workflow
```yaml
jobs:
  test:
    uses: ./.github/workflows/reusable-test.yml
    with:
      node-version: '22'
    secrets: inherit
```

## Cost Optimization
- Use `ubuntu-latest` (cheapest runner) unless macOS/Windows needed
- Set `timeout-minutes` to prevent runaway jobs
- Use `concurrency` to cancel stale runs
- Cache aggressively — every `npm ci` without cache costs time
- Use `paths` filters to skip irrelevant builds
- Consider self-hosted runners for heavy workloads
