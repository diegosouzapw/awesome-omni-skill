---
name: docker-optimize
description: "Audit and optimize Dockerfiles and docker-compose files for size, security, build speed, and best practices. Triggers on: optimize dockerfile, audit docker, fix dockerfile, docker best practices, docker compose security."
---

# Docker Optimization

Audit Dockerfiles and docker-compose files for common mistakes and security vulnerabilities. Based on patterns from hundreds of real-world Docker deployments, CWE-mapped security rules, and industry best practices.

---

## The Job

1. Find Dockerfile(s) and docker-compose files in the project
2. Run Hadolint for static analysis (if available)
3. Audit against security checklist (Critical → High → Medium)
4. Audit against optimization checklist
5. Generate prioritized findings report with CWE references
6. Apply fixes (with user confirmation)

---

## Part 1: Dockerfile Security Audit

Score: 🔴 Critical | 🟠 High | 🟡 Medium | 🟢 Good

### SEC-001: Docker Socket Exposure
**Severity**: 🔴 Critical | **CWE**: CWE-250
```dockerfile
# 🔴 CRITICAL: Full host compromise possible
VOLUME ["/var/run/docker.sock"]

# Also check for:
-v /var/run/docker.sock:/var/run/docker.sock
```
**Impact**: Grants full control over host Docker daemon. Complete container escape.
**Fix**: Never mount Docker socket. Use Docker API with TLS authentication if needed.

### SEC-002: Running as Root
**Severity**: 🟠 High | **CWE**: CWE-250
```dockerfile
# 🟠 BAD: No USER instruction (58% of images run as root!)
CMD ["node", "server.js"]

# 🟢 GOOD: Explicit non-root user
RUN adduser -D myuser && chown -R myuser /app
USER myuser
CMD ["node", "server.js"]
```
**Check**: No `USER` instruction before final `CMD`/`ENTRYPOINT`
**Note**: Some orchestrators (OpenShift) block root containers by default

### SEC-003: Secrets in ENV/ARG
**Severity**: 🔴 Critical | **CWE**: CWE-538
```dockerfile
# 🔴 CRITICAL: Visible in docker history forever
ENV AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
ARG DATABASE_PASSWORD=secret123

# 🟢 GOOD: BuildKit secrets (never persisted)
RUN --mount=type=secret,id=db_password,target=/run/secrets/db_password \
    cat /run/secrets/db_password | some-command
```
**Check**: ENV/ARG containing: key, secret, password, token, credential, api_key
**Warning**: ENV vars visible to child processes, logs, and linked containers
**Note**: ARG values visible in `docker history` even without default values

### SEC-004: Sudo in Dockerfile
**Severity**: 🟡 Medium | **CWE**: CWE-250
```dockerfile
# 🟡 BAD: Unnecessary privilege escalation
RUN sudo apt-get install -y curl

# 🟢 GOOD: Run as root before USER, then drop privileges
RUN apt-get install -y curl
USER appuser
```
**Fix**: Run privileged commands before `USER` instruction, not with sudo

### SEC-005: ADD vs COPY
**Severity**: 🟡 Medium
```dockerfile
# 🟡 BAD: ADD has hidden behaviors (auto-extract, remote fetch)
ADD https://example.com/file.tar.gz /app/
ADD archive.tar.gz /app/

# 🟢 GOOD: COPY is explicit and predictable
COPY archive.tar.gz /app/
RUN tar -xzf /app/archive.tar.gz && rm /app/archive.tar.gz
```
**Check**: Any `ADD` instruction → prefer `COPY` unless extraction needed

### SEC-006: Shell Form vs Exec Form
**Severity**: 🟡 Medium
```dockerfile
# 🟡 BAD: Shell form - no signal handling, extra shell process
CMD npm start
ENTRYPOINT /app/start.sh

# 🟢 GOOD: Exec form - proper PID 1, signal handling
CMD ["npm", "start"]
ENTRYPOINT ["/app/start.sh"]
```
**Impact**: Shell form prevents proper SIGTERM handling, graceful shutdown fails

### SEC-007: Missing pipefail
**Severity**: 🟡 Medium
```dockerfile
# 🟡 BAD: Pipe failure ignored
RUN curl -s https://example.com/script.sh | bash

# 🟢 GOOD: Fail on any pipe component failure
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
RUN curl -s https://example.com/script.sh | bash
```
**Check**: RUN with pipes (`|`) without `set -o pipefail` or SHELL override

### SEC-008: Hardcoded UID
**Severity**: 🟡 Medium
```dockerfile
# 🟡 BAD: Hardcoded UID breaks some orchestrators
RUN useradd -u 1000 appuser
RUN chown -R 1000:1000 /app

# 🟢 GOOD: Flexible UID, write to /tmp for portability
RUN adduser -D appuser
USER appuser
WORKDIR /tmp/app-workspace
```
**Why**: OpenShift and some Kubernetes configs use random UIDs. Don't assume UID 1000.

### SEC-009: Writable Binaries
**Severity**: 🟡 Medium
```dockerfile
# 🟡 BAD: App user owns executable (can be modified at runtime)
COPY --chown=appuser:appuser myapp /app/myapp

# 🟢 GOOD: Root owns binary, non-writable (immutable)
COPY myapp /app/myapp
RUN chmod 555 /app/myapp
USER appuser
```
**Why**: Prevents runtime code modification attacks. Enforces container immutability.

### SEC-010: Secrets "Deleted" in Later Layer
**Severity**: 🔴 Critical
```dockerfile
# 🔴 CRITICAL: Secret still accessible in earlier layer!
COPY credentials.json /app/
RUN configure-app.sh && rm credentials.json

# 🟢 GOOD: Use BuildKit secrets
RUN --mount=type=secret,id=creds,target=/tmp/creds \
    configure-app.sh /tmp/creds
```
**Impact**: Files removed in later layers can still be extracted from image history.

---

## Part 2: Docker Compose Security Audit

### COMPOSE-SEC-001: Privileged Mode
**Severity**: 🔴 Critical | **CWE**: CWE-250
```yaml
# 🔴 CRITICAL: Disables ALL container isolation
services:
  app:
    privileged: true
```
**Impact**: Access to all devices, can load kernel modules, complete host escape
**Fix**: Remove `privileged: true`. Use specific capabilities if needed.

### COMPOSE-SEC-002: Docker Socket Mount
**Severity**: 🔴 Critical | **CWE**: CWE-250
```yaml
# 🔴 CRITICAL: Full Docker daemon access
services:
  app:
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```
**Fix**: Remove socket mount. Use Docker API with TLS authentication.

### COMPOSE-SEC-003: Seccomp Disabled
**Severity**: 🟠 High | **CWE**: CWE-284
```yaml
# 🟠 BAD: Removes syscall filtering
services:
  app:
    security_opt:
      - seccomp:unconfined
```
**Fix**: Remove or use custom seccomp profile: `seccomp:./custom-seccomp.json`

### COMPOSE-SEC-004: Host Network Mode
**Severity**: 🟠 High | **CWE**: CWE-250
```yaml
# 🟠 BAD: Bypasses network isolation
services:
  app:
    network_mode: host
```
**Impact**: Container shares host network stack, can access all host ports
**Fix**: Use bridge network with explicit port mappings

### COMPOSE-SEC-005: Dangerous Capabilities
**Severity**: 🟠 High | **CWE**: CWE-250
```yaml
# 🟠 BAD: Overly permissive capabilities
services:
  app:
    cap_add:
      - SYS_ADMIN    # Near-root powers
      - NET_ADMIN    # Network manipulation
      - SYS_PTRACE   # Process debugging
      - ALL          # Everything

# 🟢 GOOD: Drop all, add only what's needed
services:
  app:
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE  # Only if binding <1024
```

### COMPOSE-SEC-006: Host PID/IPC Namespace
**Severity**: 🟠 High | **CWE**: CWE-250
```yaml
# 🟠 BAD: Shares host process/IPC namespace
services:
  app:
    pid: host
    ipc: host
```
**Impact**: Can see/signal all host processes, access shared memory
**Fix**: Remove `pid: host` and `ipc: host`

### COMPOSE-SEC-007: Missing no-new-privileges
**Severity**: 🟡 Medium | **CWE**: CWE-732
```yaml
# 🟢 GOOD: Prevent privilege escalation
services:
  app:
    security_opt:
      - no-new-privileges:true
```
**Check**: Missing `no-new-privileges:true` in security_opt

### COMPOSE-SEC-008: SELinux/AppArmor Disabled
**Severity**: 🟡 Medium | **CWE**: CWE-732
```yaml
# 🟡 BAD: Disables mandatory access controls
services:
  app:
    security_opt:
      - label:disable
      - apparmor:unconfined
```

### COMPOSE-SEC-009: Writable Root Filesystem
**Severity**: 🟡 Medium
```yaml
# 🟢 GOOD: Read-only root, explicit writable paths
services:
  app:
    read_only: true
    tmpfs:
      - /tmp
      - /var/run
    volumes:
      - app-data:/app/data
```
**Check**: Missing `read_only: true`

---

## Part 3: Optimization Checklist

### OPT-001: Base Image Tags
```dockerfile
# 🔴 BAD: Moving target, "your build can suddenly break"
FROM node:latest

# 🟡 BETTER: Pinned version
FROM node:20.11.1-alpine3.19

# 🟢 BEST: Immutable digest (prevents tag mutation attacks)
FROM node@sha256:abc123...
```
**Why**: Tags can change. Digests are immutable. Use tags for dev, digests for prod.

### OPT-002: Image Size
```dockerfile
# 🟡 BAD: Full OS - "more than 100 vulnerabilities" (Sysdig)
FROM ubuntu:22.04

# 🟢 GOOD: Minimal base
FROM node:20-alpine      # ~180MB
FROM python:3.12-slim    # ~150MB
FROM gcr.io/distroless/static  # ~2MB (best for Go/Rust)
```
**Prefer**: distroless > alpine > slim > full

### OPT-003: Layer Cache Order
```dockerfile
# 🔴 BAD: Cache busted on every code change
COPY . .
RUN npm install

# 🟢 GOOD: Dependencies cached separately
COPY package*.json ./
RUN npm ci
COPY . .
```
**Rule**: Order by change frequency. Rarely changed → frequently changed.

### OPT-004: Package Manager Flags
```dockerfile
# 🟡 BAD: Missing optimization flags
RUN apt-get update && apt-get install -y curl
RUN apk add curl
RUN pip install requests

# 🟢 GOOD: With cache/size optimization
RUN apt-get update && apt-get install -y --no-install-recommends curl \
    && rm -rf /var/lib/apt/lists/*
RUN apk add --no-cache curl
RUN pip install --no-cache-dir requests
```

### OPT-005: apt-get upgrade
```dockerfile
# 🟡 BAD: Non-reproducible, breaks immutability
RUN apt-get update && apt-get upgrade -y

# 🟢 GOOD: Pin base image version instead
FROM debian:12.4-slim
```
**Why**: Upgrades are unpredictable. Pin your base image for reproducibility.

### OPT-006: Multi-Stage Builds
```dockerfile
# 🟢 GOOD: Build tools don't ship to production
FROM node:20-alpine AS builder
RUN npm ci && npm run build

FROM node:20-alpine AS production
COPY --from=builder /app/dist ./dist
```

### OPT-007: .dockerignore
**Check**: No `.dockerignore` file
**Impact**: Bloated context, slower builds, potential secret leakage
**Risk**: Build context can accidentally include credentials, backups, lock files

### OPT-008: Health Checks
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1
```

### OPT-009: ENV/EXPOSE Placement
```dockerfile
# 🟡 BAD: ENV early breaks cache for everything after
FROM node:20-alpine
ENV APP_VERSION=1.0.0
COPY . .
RUN npm ci

# 🟢 GOOD: ENV/EXPOSE at end (cache optimization)
FROM node:20-alpine
COPY package*.json ./
RUN npm ci
COPY . .
ENV APP_VERSION=1.0.0
EXPOSE 3000
```
**Why**: Changing ENV invalidates all subsequent layers. Put at end unless needed for build.

### OPT-010: Multiple FROM Pitfall
```dockerfile
# ⚠️ WARNING: Only LAST FROM is used (unless multi-stage)
FROM ubuntu:22.04
FROM node:20-alpine  # This is the actual base image!
```
**Check**: Multiple `FROM` without stage names (`AS builder`) may be a mistake

### OPT-011: VOLUME Timing
```dockerfile
# 🟡 BAD: Files written before VOLUME are lost at runtime
WORKDIR /app/data
RUN echo "config" > settings.json
VOLUME /app/data  # settings.json won't persist!

# 🟢 GOOD: Write files after container starts, or use COPY
COPY settings.json /app/data/
VOLUME /app/data
```
**Why**: VOLUME is runtime, not build time. Pre-existing files are shadowed.

### OPT-012: Layer Consolidation
```dockerfile
# 🟡 BAD: 4 layers, cleanup doesn't save space
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y wget
RUN apt-get clean

# 🟢 GOOD: 1 layer, cleanup actually works
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl wget && \
    rm -rf /var/lib/apt/lists/*
```

### OPT-013: OCI Labels
```dockerfile
# 🟢 GOOD: Metadata for lifecycle management
LABEL org.opencontainers.image.title="MyApp" \
      org.opencontainers.image.version="1.0.0" \
      org.opencontainers.image.vendor="MyCompany" \
      org.opencontainers.image.source="https://github.com/myorg/myapp"
```
**Why**: Helps with image inventory, vulnerability tracking, and compliance.

---

## Report Format

```markdown
# Docker Security & Optimization Audit

## Summary
- 🔴 Critical: X issues (container escape risk)
- 🟠 High: X issues (privilege escalation risk)
- 🟡 Medium: X issues (best practice violations)
- 🟢 Good: X checks passed

## Security Findings

### 🔴 [CRITICAL] Docker Socket Exposed (SEC-001)
**File**: docker-compose.yml:12
**CWE**: CWE-250 (Execution with Unnecessary Privileges)
**Current**: `- /var/run/docker.sock:/var/run/docker.sock`
**Impact**: Complete host compromise possible
**Fix**: Remove socket mount, use Docker API with TLS

### 🔴 [CRITICAL] Secret Persisted in Layer (SEC-010)
**File**: Dockerfile:15-16
**Current**: `COPY secrets.json ... && rm secrets.json`
**Impact**: Secret extractable from image layers
**Fix**: Use BuildKit --mount=type=secret

## Optimization Findings

### 🟡 [MEDIUM] Unpinned Base Image (OPT-001)
**File**: Dockerfile:1
**Current**: `FROM node:latest`
**Fix**: `FROM node:20.11.1-alpine3.19`
**Best**: `FROM node@sha256:...` (immutable digest)

## Recommended Actions (Priority Order)
1. 🔴 Remove Docker socket mount
2. 🔴 Fix secret in layer (use BuildKit)
3. 🟠 Disable privileged mode
4. 🟠 Add non-root user
5. 🟡 Pin base image versions
```

---

## Secure docker-compose.yml Template

```yaml
version: "3.8"

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    image: myapp:${VERSION:-latest}

    # Security hardening
    read_only: true
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE  # Only if needed

    # Resource limits
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 512M
        reservations:
          memory: 256M

    # Networking
    networks:
      - app-network
    ports:
      - "3000:3000"

    # Writable paths only where needed
    tmpfs:
      - /tmp:size=100M
    volumes:
      - app-data:/app/data:rw

    # Health check
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 3s
      retries: 3
      start_period: 10s

    # Environment (no secrets here)
    environment:
      - NODE_ENV=production
    env_file:
      - .env  # Secrets in .env, not committed

networks:
  app-network:
    driver: bridge

volumes:
  app-data:
```

---

## Quick Reference: Dockerfile Template

```dockerfile
# syntax=docker/dockerfile:1
FROM node:20.11.1-alpine AS builder

WORKDIR /app

# Dependencies first (cache optimization)
COPY package*.json ./
RUN npm ci

# Source code
COPY . .
RUN npm run build

# Production stage
FROM node:20.11.1-alpine AS production

WORKDIR /app

# Non-root user (flexible UID)
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Production dependencies only
COPY package*.json ./
RUN npm ci --omit=dev && npm cache clean --force

# Built artifacts (root-owned, non-writable for security)
COPY --from=builder /app/dist ./dist
RUN chown -R appuser:appgroup /app && chmod -R 555 /app/dist

USER appuser

# Metadata
LABEL org.opencontainers.image.title="MyApp" \
      org.opencontainers.image.version="1.0.0"

# Runtime config at end (cache optimization)
ENV NODE_ENV=production
EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
  CMD wget -qO- http://localhost:3000/health || exit 1

CMD ["node", "dist/index.js"]
```

---

## Workflow

1. **Find files**
   ```bash
   find . -name "Dockerfile*" -o -name "docker-compose*.yml" -o -name "compose*.yml"
   ```

2. **Run Hadolint** (static analysis)
   ```bash
   hadolint Dockerfile
   # Or with Docker:
   docker run --rm -i hadolint/hadolint < Dockerfile
   ```

3. **Run security audit** (Critical → High → Medium)

4. **Run optimization audit**

5. **Scan for CVEs**
   ```bash
   docker scout cves myapp:latest
   trivy image myapp:latest
   ```

6. **Apply fixes** (with user confirmation)

7. **Verify**
   ```bash
   docker build -t audit-after .
   docker images audit-after
   docker history audit-after
   ```

---

## Important Rules

### Security (Never Violate)
- **NEVER** mount Docker socket
- **NEVER** use `privileged: true`
- **NEVER** put secrets in Dockerfile/docker-compose (even if "deleted" later)
- **NEVER** run as root in production
- **NEVER** disable seccomp/AppArmor without justification
- **NEVER** make binaries writable by the app user

### Optimization (Best Practice)
- **ALWAYS** pin base image versions (digests for production)
- **ALWAYS** use minimal base images (distroless > alpine > slim)
- **ALWAYS** copy dependency files before source code
- **ALWAYS** use exec form for CMD/ENTRYPOINT
- **ALWAYS** include .dockerignore
- **ALWAYS** use `--no-install-recommends` / `--no-cache` / `--no-cache-dir`
- **ALWAYS** put ENV/EXPOSE at end unless needed for build
- **ALWAYS** add OCI labels for image metadata

---

## Standard .dockerignore

```
# Git
.git
.gitignore

# Dependencies (reinstalled in container)
node_modules
vendor
.venv
__pycache__
*.pyc

# Build artifacts
dist
build
coverage
.nyc_output

# IDE/Editor
.vscode
.idea
*.swp
*.swo

# Environment/Secrets (CRITICAL)
.env
.env.*
*.pem
*.key
credentials.json
secrets/

# Documentation
*.md
docs
LICENSE

# Tests (unless needed)
tests
__tests__
*.test.js
*.spec.js

# Docker (don't include in context)
Dockerfile*
docker-compose*
.dockerignore

# CI/CD
.github
.gitlab-ci.yml
.circleci

# Backups and temp files
*.bak
*.tmp
*.log
*~
```

---

## Size Targets

| Stack | Target | Base Image |
|-------|--------|------------|
| Node.js | < 200MB | node:XX-alpine |
| Python | < 300MB | python:XX-slim |
| Go | < 20MB | distroless/static |
| Rust | < 20MB | distroless/cc |
| Java | < 300MB | eclipse-temurin:XX-jre-alpine |
| Bun | < 200MB | oven/bun:XX-slim |

---

## References

- [Sysdig: Top 20 Dockerfile Best Practices](https://www.sysdig.com/learn-cloud-native/dockerfile-best-practices)
- [Atlassian: Common Dockerfile Mistakes](https://www.atlassian.com/blog/developer/common-dockerfile-mistakes)
- [CodePathfinder: Docker Compose Security Rules](https://codepathfinder.dev/blog/announcing-docker-compose-security-rules)
- [Hadolint: Dockerfile Linter](https://github.com/hadolint/hadolint)
