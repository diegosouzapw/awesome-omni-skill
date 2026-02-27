---
name: scaffold-docker
description: Generate production-grade Docker configuration with multi-stage builds and health check module
disable-model-invocation: true
arguments:
  - name: components
    description: "Comma-separated list: 'dockerfile,health-checks,compose' or 'all'. If omitted, you will be asked."
    required: false
---

# Scaffold Docker & Health Checks

You are setting up production-grade containerization for a mission-critical TypeScript project. This skill generates a hardened Dockerfile, health check module, and Docker Compose configuration.

## Available Components

| Key | Component | Description |
|-----|-----------|-------------|
| `dockerfile` | Dockerfile + .dockerignore | Multi-stage build with security hardening |
| `health-checks` | Health Check Module | `src/health/` module with liveness, readiness, and startup probes |
| `compose` | Docker Compose | Development and production compose files |

## Instructions

### 1. Determine components

If `$ARGUMENTS` is provided, parse it as a comma-separated list. Otherwise, ask the user which components to set up. Default: `all`.

### 2. Load context

- Read `package.json` for project name, main entry point, scripts
- Read `tsconfig.json` for `outDir` (default: `dist/`)
- Check if `src/index.ts` or `src/main.ts` exists (application entry point)
- Check for existing `Dockerfile`, `docker-compose.yml`, `src/health/`

---

## Component: `dockerfile` — Dockerfile + .dockerignore

Ask the user which base image they prefer for the production stage:
- **`node:22-slim`** (recommended for most cases) — smaller than full image, includes shell for debugging
- **`gcr.io/distroless/nodejs22-debian12`** — no shell, no package manager, minimal attack surface (best for production)

### Generate `Dockerfile`:

```dockerfile
# =============================================================================
# Build Stage
# =============================================================================
FROM node:22-slim AS builder

WORKDIR /app

# Copy dependency files first for layer caching
COPY package.json package-lock.json ./

# Use npm ci for reproducible installs (Rule 3.4)
RUN npm ci

# Copy source and build
COPY tsconfig.json ./
COPY src/ ./src/
RUN npm run build

# Prune dev dependencies
RUN npm ci --omit=dev

# =============================================================================
# Production Stage
# =============================================================================
FROM <chosen-base-image> AS production

# Security: run as non-root user
USER node

WORKDIR /app

# Copy only production artifacts
COPY --from=builder --chown=node:node /app/dist ./dist
COPY --from=builder --chown=node:node /app/node_modules ./node_modules
COPY --from=builder --chown=node:node /app/package.json ./

# Environment
ENV NODE_ENV=production

# Health check (adjust port and path as needed)
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD ["node", "-e", "fetch('http://localhost:3000/healthz').then(r => process.exit(r.ok ? 0 : 1)).catch(() => process.exit(1))"]

EXPOSE 3000

# Use node directly (not npm start) for proper signal handling
CMD ["node", "dist/index.js"]
```

If the user chose distroless:
- Remove `HEALTHCHECK` (no shell available; rely on Kubernetes probes)
- `CMD` uses JSON array format only
- No `USER` directive needed (distroless defaults to non-root)

If the user chose slim:
- Add `dumb-init` for PID 1 signal handling:
  ```dockerfile
  RUN apt-get update && apt-get install -y --no-install-recommends dumb-init && rm -rf /var/lib/apt/lists/*
  ENTRYPOINT ["dumb-init", "--"]
  ```

### Generate `.dockerignore`:

```
# Source and tests (built artifacts are in dist/)
src/
tests/
**/*.test.ts
**/*.spec.ts

# Documentation
docs/
*.md
!README.md

# Development
.git/
.github/
.claude/
.vscode/
.idea/
coverage/
.stryker-tmp/
reports/

# Environment and secrets
.env
.env.*
*.pem
*.key

# Build tools
stryker.config.*
vitest.config.*
tsconfig.json
.eslintrc*
eslint.config.*
commitlint.config.*
.husky/

# OS files
.DS_Store
Thumbs.db

# Dependencies (rebuilt in Docker)
node_modules/
```

### Add npm scripts:
```json
{
  "docker:build": "docker build -t <project-name> .",
  "docker:run": "docker run -p 3000:3000 <project-name>"
}
```

### Suggest Trivy scanning:
```bash
# Scan the built image for vulnerabilities
trivy image --severity HIGH,CRITICAL --exit-code 1 <project-name>
```

---

## Component: `health-checks` — Health Check Module

Generate a health check module under `src/health/` following the project's coding standard (Result pattern, Zod schemas, branded types, readonly, TSDoc).

### `src/health/schema.ts`:
- Zod schema for health check responses
- Status type using `as const` object (not enum — Rule 3.5):
  ```typescript
  const HealthStatus = {
    HEALTHY: 'HEALTHY',
    DEGRADED: 'DEGRADED',
    UNHEALTHY: 'UNHEALTHY',
  } as const;
  type HealthStatusValue = typeof HealthStatus[keyof typeof HealthStatus];
  ```
- Response types for liveness, readiness (with per-component status), and startup
- All properties `readonly` (Rule 7.1)
- Exhaustive switch with `assertUnreachable` (Rule 8.3)

### `src/health/health.ts`:
- **`checkLiveness()`**: Returns `Result<LivenessResponse>`. Checks:
  - Process is running
  - Event loop is responsive (measure event loop lag)
  - Does NOT check downstream dependencies (liveness should not trigger restarts due to external failures)

- **`checkReadiness(options)`**: Returns `Promise<Result<ReadinessResponse>>`. Checks each dependency:
  - Accept a `ReadonlyArray<HealthDependency>` (dependency name + check function)
  - Each check has an individual timeout using `AbortController` + `Promise.race` (Rule 4.2)
  - Default timeout: 5 seconds per dependency
  - Returns per-component status with overall status
  - Bounded parallelism: check dependencies concurrently but with a limit (Rule 4.3)

- **`checkStartup()`**: Returns `Result<StartupResponse>`. Validates initialization prerequisites.

- All functions <= 40 lines (Rule 8.4), use Result pattern (Rule 6.2), have TSDoc (Rule 10.1)

### `src/health/health.test.ts`:
- Unit tests for all three health check functions
- Tests for: all healthy, one degraded, one unhealthy, timeout scenario, all down
- Property-based test with fast-check for dependency check arrays
- No `any` in tests

### `src/health/index.ts`:
- Barrel export of public types and functions

### Kubernetes probe configuration:
Generate a `docs/kubernetes-probes.yaml` reference:
```yaml
livenessProbe:
  httpGet:
    path: /livez
    port: 3000
  initialDelaySeconds: 30
  periodSeconds: 15
  failureThreshold: 3
  timeoutSeconds: 5
readinessProbe:
  httpGet:
    path: /readyz
    port: 3000
  initialDelaySeconds: 10
  periodSeconds: 10
  failureThreshold: 3
  timeoutSeconds: 5
startupProbe:
  httpGet:
    path: /healthz
    port: 3000
  initialDelaySeconds: 5
  periodSeconds: 5
  failureThreshold: 30
  timeoutSeconds: 5
```

### Integration guidance:
Show how to wire the health module into common frameworks:
- Plain `http.createServer`: route handler example
- Express: `app.get('/healthz', ...)` example
- Fastify: route example

---

## Component: `compose` — Docker Compose

### Generate `docker-compose.yml` for development:
```yaml
services:
  app:
    build:
      context: .
      target: builder  # Use build stage for development
    ports:
      - "3000:3000"
      - "9229:9229"  # Node.js debugger
    volumes:
      - ./src:/app/src
      - ./dist:/app/dist
    environment:
      - NODE_ENV=development
      - LOG_LEVEL=debug
    command: npm run dev
    # Add dependencies as needed:
    # depends_on:
    #   - postgres
    #   - redis
```

### If the user has databases or caches:
Ask which services to include and add them:
- PostgreSQL: with health check, volume for persistence
- Redis: with health check, memory limits
- MongoDB: with health check, volume

### Generate `docker-compose.prod.yml` (production overrides):
```yaml
services:
  app:
    build:
      context: .
      target: production
    restart: unless-stopped
    environment:
      - NODE_ENV=production
    deploy:
      resources:
        limits:
          memory: 512M
```

---

## Summary

After generating all selected components:
1. List all created files
2. Note the Kubernetes probe configuration if health checks were generated
3. Remind the user to:
   - Run `docker build` to verify the Dockerfile works
   - Run Trivy scan on the built image
   - Wire health check endpoints into their HTTP server
   - Add `src/health/` to the parent barrel export if applicable
