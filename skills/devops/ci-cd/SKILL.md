---
name: ci-cd
description: Use when setting up or modifying CI/CD pipelines with GitHub Actions. Covers workflow syntax, triggers, caching, matrix builds, secrets, deployment jobs, and status checks.
---

# CI/CD with GitHub Actions

## Basic Workflow Structure

```yaml
# .github/workflows/ci.yml
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
          node-version: 20
          cache: 'npm'
      - run: npm ci
      - run: npm run lint
      - run: npm run typecheck

  test:
    runs-on: ubuntu-latest
    needs: lint
    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: otto_test
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 5s
          --health-timeout 3s
          --health-retries 5
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
      - run: npm ci
      - run: npm test
        env:
          DATABASE_URL: postgresql://test:test@localhost:5432/otto_test

  build:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: build-output
          path: dist/
```

## Caching Strategies

```yaml
# Node modules caching (built into setup-node)
- uses: actions/setup-node@v4
  with:
    node-version: 20
    cache: 'npm'

# Custom caching for other dependencies
- uses: actions/cache@v4
  with:
    path: |
      ~/.cache/playwright
    key: playwright-${{ runner.os }}-${{ hashFiles('package-lock.json') }}
    restore-keys: |
      playwright-${{ runner.os }}-

# Turborepo or Nx caching for monorepos
- uses: actions/cache@v4
  with:
    path: .turbo
    key: turbo-${{ runner.os }}-${{ hashFiles('**/turbo.json') }}-${{ github.sha }}
    restore-keys: |
      turbo-${{ runner.os }}-${{ hashFiles('**/turbo.json') }}-
```

## Matrix Builds

```yaml
test:
  runs-on: ubuntu-latest
  strategy:
    fail-fast: false
    matrix:
      node-version: [18, 20, 22]
      shard: [1, 2, 3]
  steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-node@v4
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'
    - run: npm ci
    - run: npx vitest --shard=${{ matrix.shard }}/3
```

## Secrets Management

```yaml
# Use GitHub Secrets for sensitive values
- run: npm run deploy
  env:
    DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}
    DATABASE_URL: ${{ secrets.DATABASE_URL }}

# Use environments for per-environment secrets
deploy-staging:
  runs-on: ubuntu-latest
  environment: staging
  steps:
    - run: npm run deploy
      env:
        API_URL: ${{ vars.API_URL }}          # non-sensitive config
        DEPLOY_KEY: ${{ secrets.DEPLOY_KEY }}  # sensitive

# NEVER echo secrets or use them in job names/titles
# BAD: run: echo ${{ secrets.TOKEN }}
```

## Deployment Jobs

```yaml
deploy:
  runs-on: ubuntu-latest
  needs: [lint, test, build]
  if: github.ref == 'refs/heads/main' && github.event_name == 'push'
  environment:
    name: production
    url: https://otto.example.com
  steps:
    - uses: actions/checkout@v4
    - uses: actions/download-artifact@v4
      with:
        name: build-output
        path: dist/
    - name: Deploy to production
      run: |
        npx vercel deploy --prod --token=${{ secrets.VERCEL_TOKEN }}
```

## Reusable Workflows

```yaml
# .github/workflows/reusable-deploy.yml
on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string
    secrets:
      deploy_token:
        required: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}
    steps:
      - uses: actions/checkout@v4
      - run: ./deploy.sh
        env:
          DEPLOY_TOKEN: ${{ secrets.deploy_token }}

# Calling the reusable workflow
# .github/workflows/deploy-prod.yml
jobs:
  deploy:
    uses: ./.github/workflows/reusable-deploy.yml
    with:
      environment: production
    secrets:
      deploy_token: ${{ secrets.DEPLOY_TOKEN }}
```

## PR Status Checks

```yaml
# Required checks should be fast and reliable
# Separate optional checks from required ones
e2e:
  runs-on: ubuntu-latest
  if: github.event_name == 'pull_request'
  steps:
    - uses: actions/checkout@v4
    - run: npm ci
    - run: npx playwright install --with-deps chromium
    - run: npx playwright test
    - uses: actions/upload-artifact@v4
      if: failure()
      with:
        name: playwright-report
        path: playwright-report/
```

## Anti-Patterns

- NEVER hardcode secrets in workflow files. Always use `secrets` context.
- NEVER use `pull_request_target` with `actions/checkout@v4` on PR head. This grants write access to untrusted code.
- NEVER skip `concurrency` groups. Without them, stale deployments can overwrite newer ones.
- NEVER use `continue-on-error: true` on required checks. Fix the flakiness instead.
- NEVER install all Playwright browsers when you only need one. Use `--with-deps chromium`.
