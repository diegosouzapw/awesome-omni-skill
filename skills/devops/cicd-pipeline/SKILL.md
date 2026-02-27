---
name: cicd-pipeline
description: Use when setting up GitHub Actions, automated testing, build checks, or deployment workflows. Triggers on "CI/CD", "pipeline", "GitHub Actions", "deploy", "automated testing", "build check".
---

# CI/CD Pipeline

## Overview

Set up continuous integration and deployment with GitHub Actions, build verification, and deployment gates.

## When to Use

- Setting up CI/CD from scratch
- Adding build checks to PRs
- Configuring automated test runs
- Production deployment automation

## Workflow

### Phase 1: CI Workflow

`.github/workflows/ci.yml`:
```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npm run lint
      - run: npm run build
      - run: npm run test

  edge-functions:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: denoland/setup-deno@v2
      - run: deno check supabase/functions/*/index.ts
```

### Phase 2: Deploy Workflow

```yaml
  deploy:
    needs: [build, edge-functions]
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: npm }
      - run: npm ci && npm run build
      # Deploy to hosting + edge functions
```

### Phase 3: Branch Protection

1. Require CI pass before merge
2. Require PR review
3. No force push to main

## Checklist

- [ ] CI runs on push + PRs
- [ ] Build, lint, test steps
- [ ] Edge function type check
- [ ] Deploy only on main after checks pass
- [ ] Secrets in GitHub Actions secrets
- [ ] Branch protection rules enabled
