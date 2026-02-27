---
name: cicd-pipeline-builder
description: Generate CI/CD pipelines for GitHub Actions, GitLab CI, Jenkins with best practices
allowed-tools: ["Read", "Write", "Glob", "Grep"]
version: 1.0.0
author: GLINCKER Team
license: Apache-2.0
keywords: [cicd, github-actions, gitlab-ci, jenkins, automation, devops]
---

# CI/CD Pipeline Builder

Generate complete CI/CD pipelines for GitHub Actions, GitLab CI, or Jenkins. Includes testing, building, security scanning, and deployment stages with caching and optimization.

## What This Skill Does

- Generates platform-specific CI/CD configs
- Includes testing, linting, building stages
- Adds security scanning (SAST, dependency checks)
- Implements caching for faster builds
- Creates deployment workflows
- Matrix testing for multiple versions

## Supported Platforms

- GitHub Actions (most popular)
- GitLab CI/CD
- Jenkins
- CircleCI

## Instructions

### GitHub Actions Example

**.github/workflows/ci.yml**:
```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  NODE_VERSION: '20'

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18, 20, 21]
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

      - name: Run tests
        run: npm test -- --coverage

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info

  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run security audit
        run: npm audit --audit-level=moderate

      - name: CodeQL Analysis
        uses: github/codeql-action/analyze@v3

  build:
    needs: [test, security]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build

      - name: Upload artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build-artifacts
          path: dist/

  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment: production
    steps:
      - uses: actions/checkout@v4

      - name: Download artifacts
        uses: actions/download-artifact@v4
        with:
          name: build-artifacts
          path: dist/

      - name: Deploy to production
        run: |
          echo "Deploying to production..."
          # Add your deployment commands here
```

### GitLab CI Example

**.gitlab-ci.yml**:
```yaml
stages:
  - test
  - build
  - deploy

variables:
  NODE_VERSION: "20"

cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - node_modules/
    - .npm/

test:
  stage: test
  image: node:${NODE_VERSION}
  script:
    - npm ci --cache .npm --prefer-offline
    - npm run lint
    - npm test -- --coverage
  coverage: '/Lines\s*:\s*(\d+\.\d+)%/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

security:
  stage: test
  image: node:${NODE_VERSION}
  script:
    - npm audit --audit-level=moderate
  allow_failure: true

build:
  stage: build
  image: node:${NODE_VERSION}
  script:
    - npm ci --cache .npm --prefer-offline
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 week

deploy:production:
  stage: deploy
  image: alpine:latest
  script:
    - echo "Deploying to production..."
    # Add deployment commands
  only:
    - main
  environment:
    name: production
    url: https://example.com
```

## Advanced Features

### Docker Build & Push

```yaml
build-docker:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3

    - name: Login to DockerHub
      uses: docker/login-action@v3
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}

    - name: Build and push
      uses: docker/build-push-action@v5
      with:
        context: .
        push: true
        tags: |
          myapp:latest
          myapp:${{ github.sha }}
        cache-from: type=gha
        cache-to: type=gha,mode=max
```

### Multi-Environment Deployments

```yaml
deploy-staging:
  if: github.ref == 'refs/heads/develop'
  environment:
    name: staging
    url: https://staging.example.com

deploy-production:
  if: github.ref == 'refs/heads/main'
  needs: [deploy-staging]
  environment:
    name: production
    url: https://example.com
```

## Best Practices

1. **Caching**: Cache dependencies for faster builds
2. **Matrix testing**: Test multiple versions
3. **Security scanning**: Include SAST tools
4. **Artifacts**: Save build outputs
5. **Branch protection**: Require CI pass before merge
6. **Environment secrets**: Use platform secrets management

## Tool Requirements

- **Read**: Analyze project structure
- **Write**: Generate workflow files
- **Glob**: Find project files
- **Grep**: Detect frameworks

## Examples

### Example 1: Node.js Project

**User**: "Generate GitHub Actions CI/CD"

**Output**:
- Test job with matrix (Node 18, 20, 21)
- Lint and test stages
- Security audit
- Build and deploy

### Example 2: Python Project

**User**: "Create GitLab CI for Python"

**Output**:
- Pytest with coverage
- Black formatting check
- Pylint static analysis
- Docker image build

## Changelog

### Version 1.0.0
- GitHub Actions support
- GitLab CI support
- Matrix testing
- Security scanning
- Docker build integration

## Author

**GLINCKER Team**
- Repository: [claude-code-marketplace](https://github.com/GLINCKER/claude-code-marketplace)
