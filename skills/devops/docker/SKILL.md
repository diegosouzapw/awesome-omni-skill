---
name: docker
description: Creates and optimizes Docker configurations including Dockerfiles, docker-compose files, and container orchestration. Covers multi-stage builds, layer optimization, security hardening, networking, volumes, and debugging. Trigger keywords: docker, container, dockerfile, image, compose, registry, build, layer, cache, multi-stage, volume, network, port, environment, containerize, orchestration, registry, push, pull, tag, alpine, slim, distroless.
allowed-tools: Read, Grep, Glob, Edit, Write, Bash
---

# Docker

## Overview

This skill provides comprehensive Docker expertise for all container-related tasks. It covers:

- **Strategy & Architecture**: Container strategy for applications, deployment patterns, orchestration decisions
- **Implementation**: Writing optimized Dockerfiles, docker-compose files, and CI/CD integration
- **Security**: Container hardening, vulnerability scanning, secrets management, runtime protection
- **Operations**: Image optimization, layer caching, debugging, registry management, performance tuning

Use this skill for anything related to containerization, from initial strategy through production deployment.

## Instructions

### 1. Analyze Application Requirements

Before creating container configurations:

- **Runtime Dependencies**: Identify language runtime, system libraries, native extensions
- **Build vs Runtime Separation**: Plan multi-stage builds to separate build tools from runtime
- **Configuration Management**: Plan environment variables, secrets, config files
- **Data Persistence**: Identify stateful vs stateless components, volume mount points
- **Network Requirements**: Document ports, service dependencies, external integrations
- **Resource Constraints**: Consider memory, CPU, disk requirements
- **Deployment Target**: Development, staging, production environments

### 2. Write Efficient Dockerfiles

#### Base Image Selection

- **Official Images**: Prefer official language/OS images from Docker Hub
- **Minimal Images**: Use Alpine (-alpine), Slim (-slim), or Distroless for production
- **Pin Versions**: Use specific tags (python:3.12-alpine) not latest
- **Multi-Stage Pattern**: Separate builder and runtime stages

#### Layer Optimization

- **Order Matters**: Place least-changing instructions first (base image, system deps)
- **Combine Commands**: Use && to chain related RUN commands into single layers
- **Clear Caches**: Clean package manager caches in same RUN command
- **Copy Strategically**: Copy dependency files before source code for better caching

#### Image Size Reduction

- **Multi-Stage Builds**: Only copy artifacts needed at runtime
- **Remove Build Tools**: Don't include compilers, dev headers in final image
- **Minimize Layers**: Combine RUN commands where logical
- **Use .dockerignore**: Exclude unnecessary files from build context

#### Security Best Practices

- **Non-Root User**: Create and switch to non-privileged user
- **Read-Only Filesystems**: Use read-only root filesystem where possible
- **No Secrets in Layers**: Use build secrets, not ENV or COPY for credentials
- **Scan Images**: Run vulnerability scanners (trivy, grype) in CI

### 3. Configure Docker Compose Files

#### Service Definition

- **Service Dependencies**: Use depends_on with conditions (service_healthy)
- **Health Checks**: Define health check commands for critical services
- **Restart Policies**: Set appropriate restart behavior (unless-stopped, on-failure)
- **Resource Limits**: Define memory and CPU limits for production

#### Networking

- **Custom Networks**: Define named networks for service isolation
- **Internal Services**: Mark databases, caches as internal (no external ports)
- **Service Discovery**: Use service names for DNS-based discovery
- **Port Mapping**: Map only necessary ports to host

#### Volumes and Persistence

- **Named Volumes**: Use named volumes for data persistence
- **Bind Mounts**: Use for development (code changes) but not production data
- **Volume Ownership**: Ensure correct user/group ownership in containers
- **Backup Strategy**: Document volume backup procedures

#### Environment Configuration

- **Environment Files**: Use .env files for local development
- **Secret Management**: Use docker secrets or external secret managers
- **Environment Inheritance**: Use environment block for shared config
- **Variable Interpolation**: Use ${VAR:-default} for defaults

### 4. Container Security Hardening

#### Runtime Security

- **Minimal Base Images**: Reduce attack surface with distroless or alpine
- **Drop Capabilities**: Remove unnecessary Linux capabilities
- **Read-Only Root**: Mount root filesystem as read-only
- **No Privileged Mode**: Never use --privileged unless absolutely necessary
- **AppArmor/SELinux**: Use security profiles where available

#### Vulnerability Management

- **Regular Scanning**: Scan images with trivy, grype, or Docker Scout
- **Dependency Updates**: Keep base images and dependencies current
- **CVE Monitoring**: Track and remediate known vulnerabilities
- **SBOM Generation**: Create software bill of materials for compliance

#### Secrets Management

- **Never in Dockerfile**: No secrets in ENV, ARG, or committed files
- **Build Secrets**: Use --mount=type=secret for build-time secrets
- **Runtime Secrets**: Use docker secrets, vault, or k8s secrets
- **Environment Variables**: Only for non-sensitive configuration

### 5. Image Optimization Techniques

#### Multi-Stage Build Patterns

- **Builder Stage**: Install build dependencies, compile code
- **Runtime Stage**: Copy only artifacts, minimal dependencies
- **Shared Stages**: Reuse common base stages across images
- **Parallel Builds**: Use BuildKit for parallel stage execution

#### Layer Caching Strategies

- **Dependency Layers**: Copy package files before code for cache reuse
- **Cache Mounts**: Use --mount=type=cache for package manager caches
- **Ordered Instructions**: Structure Dockerfile for maximum cache hits
- **Build Context**: Minimize context size with .dockerignore

#### Registry Management

- **Image Tagging**: Use semantic versioning and git SHA tags
- **Multi-Architecture**: Build for amd64 and arm64 with buildx
- **Registry Mirrors**: Use local registry mirrors for faster pulls
- **Garbage Collection**: Clean old images from registry regularly

### 6. Debugging Containers

#### Interactive Debugging

- **Shell Access**: docker exec -it `<container>` /bin/sh
- **Debug Images**: Temporarily use non-minimal base for troubleshooting
- **Ephemeral Containers**: Use debug sidecars in Kubernetes
- **Log Inspection**: docker logs -f --tail 100 `<container>`

#### Common Issues

- **Permission Errors**: Check user/group IDs match volume ownership
- **Network Issues**: Verify network configuration, DNS resolution
- **Missing Dependencies**: Check runtime dependencies in final stage
- **Build Failures**: Use --progress=plain for detailed build output
- **Layer Size**: Use dive tool to analyze layer sizes

#### Tools and Techniques

- **Dive**: Analyze image layers and find bloat
- **Hadolint**: Lint Dockerfiles for best practices
- **Docker Scout**: Scan for vulnerabilities and recommendations
- **Trivy**: Comprehensive vulnerability scanning
- **Ctop**: Container resource monitoring

## Best Practices

### Universal Principles

1. **Use Official Base Images**: Start from trusted sources (Docker Official Images)
2. **Multi-Stage Builds**: Separate build and runtime environments to minimize size
3. **Minimize Layers**: Combine related commands with && to reduce layer count
4. **Don't Run as Root**: Create and use non-root users for security
5. **Use .dockerignore**: Exclude unnecessary files from build context
6. **Pin Versions**: Use specific tags (python:3.12-alpine) not latest
7. **Health Checks**: Add HEALTHCHECK instructions for container health monitoring
8. **Scan Regularly**: Run vulnerability scanners in CI/CD pipeline
9. **Clear Caches**: Remove package manager caches in same layer they're created
10. **Document Thoroughly**: Add comments explaining non-obvious decisions

### Language-Specific Patterns

#### Python

- Use pip wheel to cache compiled dependencies
- Use --no-cache-dir to prevent pip cache in layers
- Consider uv for faster dependency resolution
- Use python:3.x-slim for balance of size and compatibility

#### Node.js

- Copy package\*.json before source for better caching
- Use npm ci instead of npm install for reproducible builds
- Remove devDependencies in production builds
- Consider node:alpine for minimal size

#### Rust

- Use cargo chef for dependency caching
- Build with --release for optimized binaries
- Copy only the binary to runtime stage, not entire target/
- Consider scratch or distroless base for minimal size

#### Go

- Use multi-stage with golang:alpine for build
- Build statically linked binaries (CGO_ENABLED=0)
- Use scratch or distroless for minimal runtime
- COPY only the binary, Go doesn't need runtime

### Development vs Production

#### Development

- Use bind mounts for live code reloading
- Include debugging tools in image
- Expose more ports for inspection
- Use verbose logging
- Mount source code as volume

#### Production

- Use named volumes for persistence
- Minimal images with only runtime dependencies
- Restricted ports and capabilities
- Structured logging to stdout/stderr
- No SSH, debugging tools, or shells in final image

## Examples

### Example 1: Multi-Stage Python Dockerfile

```dockerfile
# Build stage
FROM python:3.12-slim AS builder

WORKDIR /app

# Install build dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY requirements.txt .
RUN pip wheel --no-cache-dir --no-deps --wheel-dir /app/wheels -r requirements.txt

# Runtime stage
FROM python:3.12-slim AS runtime

# Create non-root user
RUN groupadd --gid 1000 appgroup && \
    useradd --uid 1000 --gid appgroup --shell /bin/bash --create-home appuser

WORKDIR /app

# Copy wheels from builder
COPY --from=builder /app/wheels /wheels
RUN pip install --no-cache-dir /wheels/* && rm -rf /wheels

# Copy application code
COPY --chown=appuser:appgroup . .

# Switch to non-root user
USER appuser

# Expose port
EXPOSE 8000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1

# Run application
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "--workers", "4", "app:create_app()"]
```

### Example 2: Node.js Dockerfile with Security

```dockerfile
FROM node:20-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy source code
COPY . .

# Build application
RUN npm run build

# Production stage
FROM node:20-alpine AS production

# Add security updates
RUN apk update && apk upgrade && rm -rf /var/cache/apk/*

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nextjs -u 1001

WORKDIR /app

# Copy built assets
COPY --from=builder --chown=nextjs:nodejs /app/dist ./dist
COPY --from=builder --chown=nextjs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nextjs:nodejs /app/package.json ./

USER nextjs

EXPOSE 3000

ENV NODE_ENV=production

CMD ["node", "dist/server.js"]
```

### Example 3: Docker Compose for Development

```yaml
version: "3.8"

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      - .:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgres://user:pass@db:5432/myapp
      - REDIS_URL=redis://cache:6379
    depends_on:
      db:
        condition: service_healthy
      cache:
        condition: service_started
    networks:
      - app-network

  db:
    image: postgres:16-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: myapp
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d myapp"]
      interval: 5s
      timeout: 5s
      retries: 5
    networks:
      - app-network

  cache:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data
    networks:
      - app-network

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./certs:/etc/nginx/certs:ro
    depends_on:
      - app
    networks:
      - app-network

volumes:
  postgres_data:
  redis_data:

networks:
  app-network:
    driver: bridge
```

### Example 4: Rust Dockerfile with Cargo Chef

```dockerfile
# Chef stage - plan dependencies
FROM rust:1.75-alpine AS chef
RUN apk add --no-cache musl-dev && \
    cargo install cargo-chef
WORKDIR /app

# Planner stage - analyze dependencies
FROM chef AS planner
COPY Cargo.toml Cargo.lock ./
COPY src ./src
RUN cargo chef prepare --recipe-path recipe.json

# Builder stage - build dependencies then app
FROM chef AS builder
COPY --from=planner /app/recipe.json recipe.json

# Build dependencies (cached layer)
RUN cargo chef cook --release --recipe-path recipe.json

# Copy source and build application
COPY . .
RUN cargo build --release && \
    strip target/release/myapp

# Runtime stage - minimal distroless image
FROM gcr.io/distroless/cc-debian12

# Copy only the binary
COPY --from=builder /app/target/release/myapp /usr/local/bin/myapp

# Run as non-root (distroless provides nonroot user)
USER nonroot:nonroot

EXPOSE 8080

ENTRYPOINT ["/usr/local/bin/myapp"]
```

### Example 5: Go Dockerfile with Static Binary

```dockerfile
# Builder stage
FROM golang:1.22-alpine AS builder

WORKDIR /app

# Install build dependencies
RUN apk add --no-cache git ca-certificates tzdata

# Copy go mod files for caching
COPY go.mod go.sum ./
RUN go mod download

# Copy source code
COPY . .

# Build static binary
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build \
    -ldflags='-w -s -extldflags "-static"' \
    -o /app/bin/server ./cmd/server

# Runtime stage - scratch (empty base)
FROM scratch

# Copy CA certificates for HTTPS
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/

# Copy timezone data
COPY --from=builder /usr/share/zoneinfo /usr/share/zoneinfo

# Copy binary
COPY --from=builder /app/bin/server /server

# Expose port
EXPOSE 8080

# Run as non-root (must be defined)
USER 65534:65534

ENTRYPOINT ["/server"]
```

### Example 6: Docker Compose with Secrets

```yaml
version: "3.8"

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
      args:
        - APP_VERSION=${APP_VERSION:-latest}
    image: myapp:${APP_VERSION:-latest}
    ports:
      - "8080:8080"
    environment:
      - DATABASE_URL_FILE=/run/secrets/db_url
      - API_KEY_FILE=/run/secrets/api_key
    secrets:
      - db_url
      - api_key
    depends_on:
      db:
        condition: service_healthy
    networks:
      - backend
    restart: unless-stopped
    deploy:
      resources:
        limits:
          cpus: "1.0"
          memory: 512M
        reservations:
          cpus: "0.5"
          memory: 256M

  db:
    image: postgres:16-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
      POSTGRES_USER: myapp
      POSTGRES_DB: myapp
    secrets:
      - db_password
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U myapp"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

secrets:
  db_url:
    file: ./secrets/db_url.txt
  api_key:
    file: ./secrets/api_key.txt
  db_password:
    file: ./secrets/db_password.txt

volumes:
  postgres_data:

networks:
  backend:
    driver: bridge
    ipam:
      config:
        - subnet: 172.28.0.0/16
```

### Example 7: Build Cache Optimization with BuildKit

```dockerfile
# syntax=docker/dockerfile:1.4

FROM node:20-alpine AS base

# Enable BuildKit cache mounts
WORKDIR /app

# Install dependencies with cache mount
FROM base AS deps
COPY package*.json ./
RUN --mount=type=cache,target=/root/.npm \
    npm ci --only=production

# Build stage with separate dev dependencies
FROM base AS builder
COPY package*.json ./
RUN --mount=type=cache,target=/root/.npm \
    npm ci
COPY . .
RUN npm run build

# Production stage
FROM base AS production

# Copy production node_modules
COPY --from=deps /app/node_modules ./node_modules

# Copy built application
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package.json ./

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nextjs -u 1001

USER nextjs

EXPOSE 3000

CMD ["node", "dist/index.js"]
```

### Example 8: .dockerignore

```gitignore
# Git
.git
.gitignore

# Dependencies
node_modules
__pycache__
*.pyc
.venv
venv

# Build artifacts
dist
build
*.egg-info

# IDE
.idea
.vscode
*.swp

# Testing
coverage
.pytest_cache
.nyc_output

# Environment files
.env
.env.local
.env.*.local

# Documentation
docs
*.md
!README.md

# Docker
Dockerfile*
docker-compose*
.docker

# Misc
.DS_Store
*.log
tmp
```

## Common Patterns and Solutions

### Pattern: Handling Configuration

#### Environment-based Config

```dockerfile
# Support multiple environments
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .

# Default to production, can override at runtime
ENV NODE_ENV=production

# Use config files per environment
CMD ["sh", "-c", "node server.js --config /app/config/${NODE_ENV}.json"]
```

#### Config from Files (12-Factor)

```dockerfile
# Read config from files mounted at runtime
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

# Application reads from /config at runtime
VOLUME /config

CMD ["python", "app.py", "--config", "/config/settings.yaml"]
```

### Pattern: Multi-Architecture Builds

```bash
# Build for multiple architectures with buildx
docker buildx create --name multiarch --use
docker buildx build --platform linux/amd64,linux/arm64 \
  -t myapp:latest \
  --push \
  .
```

```dockerfile
# Handle architecture differences in Dockerfile
FROM --platform=${BUILDPLATFORM} golang:1.22-alpine AS builder

ARG TARGETARCH
ARG TARGETOS

WORKDIR /app
COPY . .

RUN CGO_ENABLED=0 GOOS=${TARGETOS} GOARCH=${TARGETARCH} \
    go build -o /app/bin/server ./cmd/server

FROM alpine:latest
COPY --from=builder /app/bin/server /server
ENTRYPOINT ["/server"]
```

### Pattern: Database Migrations

```yaml
# docker-compose with migration service
version: "3.8"

services:
  migrate:
    image: myapp:latest
    command: ["./migrate", "up"]
    environment:
      DATABASE_URL: postgres://user:pass@db:5432/myapp
    depends_on:
      db:
        condition: service_healthy
    networks:
      - backend

  app:
    image: myapp:latest
    depends_on:
      migrate:
        condition: service_completed_successfully
    environment:
      DATABASE_URL: postgres://user:pass@db:5432/myapp
    ports:
      - "8080:8080"
    networks:
      - backend

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: myapp
    volumes:
      - db_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d myapp"]
      interval: 5s
      timeout: 5s
      retries: 5
    networks:
      - backend

volumes:
  db_data:

networks:
  backend:
```

### Pattern: Build-time Secrets

```dockerfile
# syntax=docker/dockerfile:1.4

FROM node:20-alpine AS builder

WORKDIR /app

# Use secret at build time without leaving it in layers
RUN --mount=type=secret,id=npmrc,target=/root/.npmrc \
    npm ci

COPY . .
RUN npm run build

FROM node:20-alpine AS production
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/server.js"]
```

```bash
# Build with secret
docker build --secret id=npmrc,src=$HOME/.npmrc -t myapp .
```

### Pattern: Health Check with Dependencies

```dockerfile
FROM python:3.12-slim

# Install health check tool
RUN apt-get update && apt-get install -y --no-install-recommends curl && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

# Comprehensive health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
    CMD curl -f http://localhost:8000/health && \
        python -c "import redis; redis.Redis(host='cache').ping()" || exit 1

CMD ["gunicorn", "--bind", "0.0.0.0:8000", "app:app"]
```

## Troubleshooting Guide

### Issue: Build Context Too Large

**Symptoms**: Slow builds, "sending build context" takes minutes

**Solutions**:

1. Add comprehensive .dockerignore file
2. Check for large files accidentally included (node_modules, .git, build artifacts)
3. Use .dockerignore patterns like `**/.git` to exclude nested git repos
4. Build from subdirectory: `docker build -f Dockerfile.app ./app`

### Issue: Layer Caching Not Working

**Symptoms**: Rebuilds everything despite no changes

**Solutions**:

1. Ensure dependency files are copied before source code
2. Don't use wildcard COPY at the start of Dockerfile
3. Check if commands have dynamic output (dates, random values)
4. Use BuildKit cache mounts for package managers
5. Verify build context hasn't changed (check .dockerignore)

### Issue: Image Size Too Large

**Symptoms**: Image is hundreds of MB when it should be small

**Solutions**:

1. Use multi-stage builds to exclude build dependencies
2. Choose smaller base images (alpine, slim, distroless)
3. Clean package manager caches in same RUN command: `apt-get install ... && rm -rf /var/lib/apt/lists/*`
4. Remove unnecessary files before COPY to runtime stage
5. Use `docker image history myapp:latest` to find large layers
6. Use `dive` tool to analyze layer contents

### Issue: Permission Denied in Container

**Symptoms**: Cannot write files, "permission denied" errors

**Solutions**:

1. Check volume ownership matches container user
2. Use `--chown=user:group` in COPY instructions
3. Create user with specific UID/GID matching host: `useradd -u 1000`
4. Initialize volumes with correct ownership in entrypoint
5. For bind mounts, ensure host directory permissions are correct

### Issue: Container Cannot Connect to Other Services

**Symptoms**: "connection refused", "name not found" errors

**Solutions**:

1. Verify services are on same Docker network
2. Use service names for DNS (not localhost)
3. Check `depends_on` in docker-compose
4. Ensure ports are exposed (EXPOSE in Dockerfile)
5. Check if service is actually listening on correct interface (0.0.0.0 not 127.0.0.1)
6. Verify firewall rules on host

### Issue: Changes Not Reflected in Container

**Symptoms**: Code changes don't appear, old version running

**Solutions**:

1. Check if using volume mounts for development
2. Rebuild image if not using bind mounts: `docker-compose build`
3. Remove containers and recreate: `docker-compose up --force-recreate`
4. Clear build cache if needed: `docker builder prune`
5. Verify correct image is running: `docker ps` and check IMAGE column

### Issue: Container Exits Immediately

**Symptoms**: Container starts then exits, no error message

**Solutions**:

1. Check logs: `docker logs <container>`
2. Run interactively: `docker run -it myapp /bin/sh`
3. Verify CMD/ENTRYPOINT is correct and executable
4. Check if application requires environment variables
5. Ensure dependencies are available (missing shared libraries)
6. Add `tail -f /dev/null` temporarily to keep container running for debugging

### Issue: Build Fails with "No Space Left on Device"

**Symptoms**: Build fails midway with disk space error

**Solutions**:

1. Clean unused containers: `docker container prune`
2. Remove unused images: `docker image prune -a`
3. Clean build cache: `docker builder prune`
4. Remove unused volumes: `docker volume prune`
5. Check Docker disk usage: `docker system df`
6. Increase Docker Desktop disk limit in settings

### Issue: Slow Container Startup

**Symptoms**: Container takes long time to become healthy

**Solutions**:

1. Optimize application startup (lazy loading, parallel initialization)
2. Use smaller base images to reduce initial I/O
3. Implement health check with appropriate start period
4. Reduce number of layers to speed up image pull
5. Use local registry or registry mirror to cache images
6. Pre-pull images: `docker-compose pull`

## CI/CD Integration

### GitHub Actions Example

```yaml
name: Build and Push Docker Image

on:
  push:
    branches: [main]
    tags: ["v*"]

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Log in to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ghcr.io/${{ github.repository }}
          tags: |
            type=ref,event=branch
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          platforms: linux/amd64,linux/arm64
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

      - name: Scan image for vulnerabilities
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ steps.meta.outputs.tags }}
          format: "sarif"
          output: "trivy-results.sarif"

      - name: Upload scan results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: "trivy-results.sarif"
```

### GitLab CI Example

```yaml
stages:
  - build
  - test
  - push

variables:
  DOCKER_DRIVER: overlay2
  DOCKER_TLS_CERTDIR: "/certs"
  IMAGE_TAG: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA

build:
  stage: build
  image: docker:24-cli
  services:
    - docker:24-dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build --pull -t $IMAGE_TAG .
    - docker push $IMAGE_TAG
  only:
    - main
    - tags

security-scan:
  stage: test
  image: aquasec/trivy:latest
  script:
    - trivy image --exit-code 1 --severity HIGH,CRITICAL $IMAGE_TAG
  needs:
    - build

push-latest:
  stage: push
  image: docker:24-cli
  services:
    - docker:24-dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker pull $IMAGE_TAG
    - docker tag $IMAGE_TAG $CI_REGISTRY_IMAGE:latest
    - docker push $CI_REGISTRY_IMAGE:latest
  only:
    - main
  needs:
    - security-scan
```

## Quick Reference

### Essential Commands

```bash
# Build
docker build -t myapp:latest .
docker build -f Dockerfile.prod -t myapp:prod .
docker buildx build --platform linux/amd64,linux/arm64 -t myapp:latest --push .

# Run
docker run -d -p 8080:8080 --name myapp myapp:latest
docker run -it --rm myapp:latest /bin/sh
docker run --env-file .env myapp:latest

# Compose
docker-compose up -d
docker-compose down -v
docker-compose logs -f app
docker-compose exec app /bin/sh
docker-compose build --no-cache

# Inspect
docker ps
docker logs -f <container>
docker inspect <container>
docker exec -it <container> /bin/sh
docker stats

# Clean
docker system prune -a
docker volume prune
docker builder prune
docker image prune -a

# Debug
docker run --rm -it --entrypoint /bin/sh myapp:latest
docker cp <container>:/app/logfile.log ./local-logfile.log
docker diff <container>
```

### Performance Optimization Checklist

- [ ] Using multi-stage builds
- [ ] Minimal base image (alpine, slim, distroless)
- [ ] Dependencies copied before source code
- [ ] Package manager caches cleared in same layer
- [ ] .dockerignore excludes unnecessary files
- [ ] Build cache mounts for package managers
- [ ] Layer count minimized (combined RUN commands)
- [ ] Health checks defined
- [ ] Non-root user configured
- [ ] Specific version tags (not latest)
- [ ] Security scanning in CI/CD
- [ ] Image size < 200MB (adjust per use case)

### Security Checklist

- [ ] Using official/trusted base images
- [ ] Base images pinned to specific versions
- [ ] No secrets in Dockerfile or environment variables
- [ ] Running as non-root user
- [ ] Read-only root filesystem where possible
- [ ] Minimal packages installed
- [ ] Security updates applied
- [ ] Vulnerability scanning enabled
- [ ] No unnecessary capabilities
- [ ] AppArmor/SELinux profiles applied
- [ ] Secrets managed via Docker secrets or external vault
- [ ] HTTPS certificates validated
