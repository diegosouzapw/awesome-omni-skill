---
name: docker-best-practices
description: Create production-grade Dockerfiles optimized for speed, security, and minimal size. Use when creating or reviewing Dockerfiles, docker-compose files, or when optimizing container images for Python, Node.js, or multi-runtime environments.
---

# Docker Best Practices

Production-grade Docker patterns optimized for fast builds, minimal size, and bulletproof security. Every container spins up in seconds, syncs efficiently, and never accumulates bloat.

## Core Principles

1. **Multi-stage builds** — Separate build dependencies from runtime
2. **Pinned versions** — Never use `:latest` tags
3. **Layer caching** — Order instructions from least to most frequently changing
4. **Minimal attack surface** — Distroless or slim base images, non-root users
5. **BuildKit features** — Cache mounts, secrets, multi-platform builds
6. **Fast startup** — Eager dependencies, lazy application code

## Quick Reference

### Python API Services (FastAPI, Flask)

```dockerfile
# syntax=docker/dockerfile:1.7
FROM python:3.11.11-slim AS base

WORKDIR /app

# Install system dependencies (cached unless changed)
RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Create non-root user
RUN groupadd -g 1000 app && \
    useradd -m -u 1000 -g app app

# ─── Build stage ───
FROM base AS builder

RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# Install dependencies with cache mount
COPY pyproject.toml ./
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install --no-cache-dir .

# ─── Production stage ───
FROM base AS production

COPY --from=builder /usr/local/lib/python3.11/site-packages /usr/local/lib/python3.11/site-packages
COPY --from=builder /usr/local/bin /usr/local/bin

COPY . .

USER app
EXPOSE 8000

HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
    CMD python -c "import httpx; httpx.get('http://localhost:8000/health', timeout=2)"

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Multi-Runtime Sandbox (Python + Node + Tools)

```dockerfile
# syntax=docker/dockerfile:1.7
FROM ubuntu:22.04.5 AS base

ENV DEBIAN_FRONTEND=noninteractive \
    LANG=C.UTF-8 \
    LC_ALL=C.UTF-8

# Single apt layer for all system packages
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && apt-get install -y --no-install-recommends \
    # Core utilities
    bash curl wget git jq unzip ca-certificates \
    # Build tools (for native extensions)
    build-essential libffi-dev libssl-dev \
    # FUSE for S3 mount
    fuse libfuse2 \
    # Process management
    tini \
    # Python
    python3.11 python3.11-venv python3.11-dev python3-pip \
    && ln -sf /usr/bin/python3.11 /usr/bin/python3 \
    && ln -sf /usr/bin/python3.11 /usr/bin/python \
    && rm -rf /var/lib/apt/lists/*

# Python package installs with cache mount
RUN --mount=type=cache,target=/root/.cache/pip \
    python3 -m pip install --no-cache-dir --upgrade pip setuptools wheel

# Node.js via official NodeSource script
RUN curl -fsSL https://deb.nodesource.com/setup_20.x | bash - && \
    apt-get install -y --no-install-recommends nodejs && \
    rm -rf /var/lib/apt/lists/* && \
    npm install -g npm@latest

# Create non-root user
RUN groupadd -g 1000 agent && \
    useradd -m -u 1000 -g agent -s /bin/bash agent

USER agent
WORKDIR /home/agent

ENTRYPOINT ["tini", "--"]
CMD ["/bin/bash"]
```

### Development vs Production Patterns

```dockerfile
# ─── Development stage (hot reload, debug tools) ───
FROM base AS development

RUN --mount=type=cache,target=/root/.cache/pip \
    pip install --no-cache-dir -e ".[dev]"

COPY . .

# Mount source as volume for hot reload
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--reload"]

# ─── Production stage (minimal, optimized) ───
FROM base AS production

COPY --from=builder /usr/local/lib/python3.11/site-packages /usr/local/lib/python3.11/site-packages
COPY . .

USER app

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0"]
```

## Essential Patterns

### Version Pinning

```dockerfile
# ❌ NEVER use latest
FROM python:3.11-slim

# ✅ ALWAYS pin to patch version
FROM python:3.11.11-slim

# ✅ Pin all base images with digest for immutability
FROM python:3.11.11-slim@sha256:abc123...
```

Check latest versions:
- Python: https://hub.docker.com/_/python/tags
- Node: https://hub.docker.com/_/node/tags
- Ubuntu: https://hub.docker.com/_/ubuntu/tags

### BuildKit Cache Mounts

```dockerfile
# ❌ Old way — downloads every build
RUN pip install -r requirements.txt

# ✅ New way — reuses download cache
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install -r requirements.txt

# ✅ npm cache mount
RUN --mount=type=cache,target=/root/.npm \
    npm ci --prefer-offline

# ✅ apt cache mount (shared across concurrent builds)
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && apt-get install -y package-name
```

### Layer Ordering for Optimal Caching

```dockerfile
# Install dependencies (rarely change) — cached ✅
COPY pyproject.toml ./
RUN pip install .

# Copy application code (changes often) — rebuilt on every change ✅
COPY . .
```

### Health Checks

```dockerfile
# Python API
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
    CMD python -c "import httpx; httpx.get('http://localhost:8000/health')"

# Node.js API
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
    CMD node -e "require('http').get('http://localhost:3000/health', (r) => process.exit(r.statusCode === 200 ? 0 : 1))"

# General process check
HEALTHCHECK --interval=30s --timeout=5s CMD pgrep -x python3 || exit 1
```

### Security Hardening

```dockerfile
# 1. Non-root user
RUN groupadd -g 1000 app && \
    useradd -m -u 1000 -g app app
USER app

# 2. Read-only root filesystem (in docker-compose or k8s)
# docker run --read-only --tmpfs /tmp ...

# 3. Drop capabilities (in docker-compose)
# security_opt:
#   - no-new-privileges:true
# cap_drop:
#   - ALL

# 4. Minimal base image (distroless for Python/Node)
FROM gcr.io/distroless/python3-debian12:latest
```

## docker-compose.yml Best Practices

```yaml
services:
  api:
    build:
      context: .
      dockerfile: Dockerfile
      target: production  # Multi-stage target
      cache_from:
        - type=registry,ref=ghcr.io/org/api:buildcache
    image: org/api:latest
    restart: unless-stopped
    
    # Security
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    read_only: true
    tmpfs:
      - /tmp
    
    # Resources
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
        reservations:
          cpus: '1'
          memory: 1G
    
    # Health check
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 10s
    
    # Env
    env_file: .env
    environment:
      - NODE_ENV=production
```

## .dockerignore

**CRITICAL**: Always create `.dockerignore` to exclude unnecessary files from build context.

```
# Version control
.git/
.github/
.gitignore

# Python
__pycache__/
*.pyc
*.pyo
*.pyd
.Python
*.so
*.egg-info/
dist/
build/
.venv/
venv/
*.pytest_cache/
.coverage

# Node
node_modules/
npm-debug.log*
.npm/

# IDE
.vscode/
.idea/
*.swp
*.swo
.DS_Store

# Env files (never copy .env to image)
.env
.env.*
!.env.example

# CI/CD
.github/
*.md
!README.md

# Docs
docs/
*.md
```

## Dockerfile Review Checklist

When creating or reviewing a Dockerfile, verify:

### Performance
- [ ] Multi-stage build for services with build dependencies
- [ ] `--mount=type=cache` for pip/npm/apt downloads
- [ ] Layer ordering: system packages → language runtime → dependencies → app code
- [ ] `.dockerignore` excludes unnecessary files
- [ ] Single `apt-get update && install && rm -rf` per stage (not multiple RUN commands)

### Security
- [ ] Base image pinned to specific version (no `:latest`)
- [ ] Non-root user for runtime
- [ ] Minimal base image (slim, alpine, or distroless)
- [ ] No secrets in ENV or build args (use `--mount=type=secret` instead)
- [ ] Health check defined

### Size
- [ ] `--no-install-recommends` on apt-get
- [ ] `rm -rf /var/lib/apt/lists/*` after apt-get
- [ ] `--no-cache-dir` on pip installs
- [ ] Multi-stage excludes build tools from final image
- [ ] Production stage only includes runtime dependencies

### Correctness
- [ ] `WORKDIR` set before COPY
- [ ] Correct user ownership for copied files if using non-root user
- [ ] `EXPOSE` matches actual port
- [ ] `CMD`/`ENTRYPOINT` is production-appropriate (no `--reload`)
- [ ] Environment variables have sensible defaults

## Common Mistakes to Avoid

### ❌ Installing dev dependencies in production
```dockerfile
FROM python:3.11-slim
RUN pip install -e ".[dev]"  # Includes pytest, debug tools, etc.
```

### ✅ Separate dev and prod stages
```dockerfile
FROM base AS development
RUN pip install -e ".[dev]"

FROM base AS production
RUN pip install .  # Prod deps only
```

### ❌ Running as root
```dockerfile
CMD ["uvicorn", "app:main"]  # Runs as root
```

### ✅ Create and use non-root user
```dockerfile
RUN useradd -m app
USER app
CMD ["uvicorn", "app:main"]
```

### ❌ Copying before installing dependencies
```dockerfile
COPY . .  # Invalidates cache on every code change
RUN pip install -r requirements.txt
```

### ✅ Install dependencies first
```dockerfile
COPY requirements.txt .
RUN pip install -r requirements.txt  # Cached unless requirements.txt changes
COPY . .
```

### ❌ Using `:latest` tags
```dockerfile
FROM python:3-slim  # Breaks reproducibility
```

### ✅ Pin to patch version
```dockerfile
FROM python:3.11.11-slim
```

### ❌ Multiple apt-get updates
```dockerfile
RUN apt-get update && apt-get install -y curl
RUN apt-get update && apt-get install -y git  # Second update is wasted
```

### ✅ Single apt layer
```dockerfile
RUN apt-get update && apt-get install -y \
    curl \
    git \
    && rm -rf /var/lib/apt/lists/*
```

## Building Images

### Enable BuildKit
```bash
export DOCKER_BUILDKIT=1

# Or in docker-compose.yml
export COMPOSE_DOCKER_CLI_BUILD=1
```

### Multi-platform builds
```bash
# For ARM64 Macs building x86_64 images
docker buildx build \
  --platform linux/amd64 \
  --tag org/app:latest \
  --push \
  .

# Build for both platforms
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  --tag org/app:latest \
  --push \
  .
```

### Cache optimization
```bash
# Use inline cache for CI
docker buildx build \
  --cache-from type=registry,ref=org/app:buildcache \
  --cache-to type=registry,ref=org/app:buildcache,mode=max \
  --tag org/app:latest \
  --push \
  .
```

## Additional Resources

- For detailed explanations and edge cases, see [reference.md](reference.md)
- For complete working examples, see [templates/](templates/)
- For image size comparison and benchmarks, see [optimization.md](optimization.md)

## When NOT to Use This Skill

- Creating `docker-compose.yml` for simple local dev (just use official images)
- Debugging running containers (use `docker logs`, `docker exec` instead)
- Container orchestration at scale (Kubernetes/ECS patterns differ significantly)
