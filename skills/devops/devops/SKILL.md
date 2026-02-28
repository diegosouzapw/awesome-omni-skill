---
name: devops
description: "DevOps skills: environment config, Kubernetes, CI/CD pipelines, containerization."
license: MIT
metadata:
  category: devops
  contains: env-config-manager, kubernetes-deploy, devops-ci-cd, dockerize-app
---

# DevOps Skills

## 1. Environment & Config Management

Manage environment configuration, secrets, and deployment-safe config patterns.

### Config hierarchy
1. Defaults in code (safe fallback values).
2. Config files (environment-specific overrides).
3. Environment variables (deployment-injected).
4. Secret manager (Vault, AWS SSM, etc.).

### Rules
- Validate all required env vars at startup, fail fast if missing.
- Never commit real secrets; use `.env.example` with placeholders.
- Rotate secrets regularly (90 days keys, 30 days passwords).
- Same code artifact across all environments; only config changes.

---

## 2. Kubernetes Deployment

Write production-ready K8s manifests, Helm charts, and rollout strategies.

### Essentials
- Always set resource requests AND limits.
- Liveness probe (restart if unhealthy), readiness probe (remove if not ready).
- Run as non-root: `securityContext.runAsNonRoot: true`.
- Drop all capabilities: `drop: [ALL]`.
- Use NetworkPolicies to restrict pod-to-pod communication.
- Rolling update: `maxUnavailable: 0`, `maxSurge: 1` for zero-downtime.
- Pod Disruption Budget for maintenance operations.

---

## 3. CI/CD Pipelines

Design and configure CI/CD pipelines for automated build, test, and deployment.

### Pipeline stages
1. Lint and format check.
2. Unit tests with coverage.
3. Integration tests.
4. Security scanning (SAST, dependency audit).
5. Build artifact (container image, package).
6. Deploy to staging.
7. Smoke/acceptance tests.
8. Deploy to production (canary or rolling).

---

## 4. Containerization

Dockerize applications with multi-stage builds, security hardening, and optimization.

### Best practices
- Multi-stage builds to minimize image size.
- Use specific base image tags, not `latest`.
- Run as non-root user.
- Copy only necessary files.
- Layer ordering: dependencies first, code last (cache optimization).
- Health check instruction in Dockerfile.
- .dockerignore for build exclusions.
