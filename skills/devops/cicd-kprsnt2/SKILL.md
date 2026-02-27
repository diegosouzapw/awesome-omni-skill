---
name: cicd
description: CI/CD pipeline best practices including GitHub Actions, testing, and deployment strategies.
globs: ["**/.github/workflows/*.yml", "**/.gitlab-ci.yml", "**/Jenkinsfile", "**/.circleci/config.yml"]
priority: 75
tags: ["devops"]
---

# CI/CD Best Practices

## Pipeline Design
- Keep pipelines fast (< 10 min)
- Fail fast (lint/test first)
- Cache dependencies
- Use parallel jobs
- Make builds reproducible

## GitHub Actions
- Use specific action versions
- Use composite actions for reuse
- Store secrets in GitHub Secrets
- Use matrix for multi-version testing
- Use artifacts for build outputs

## Testing in CI
- Run unit tests on every push
- Run integration tests on PR
- Run E2E tests before deploy
- Generate coverage reports
- Fail on coverage drops

## Deployment
- Use blue/green or canary deployments
- Automate staging deployments
- Require approval for production
- Implement rollback procedures
- Use feature flags

## Security
- Scan dependencies (Dependabot, Snyk)
- Scan Docker images
- Run SAST/DAST
- Rotate secrets regularly
- Use OIDC for cloud auth

## Artifacts
- Version artifacts semantically
- Sign artifacts
- Store in artifact registry
- Clean up old artifacts
