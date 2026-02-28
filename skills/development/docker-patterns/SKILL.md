---
name: docker-patterns
description: Docker best practices, multi-stage builds, container security, Docker Compose orchestration, and deployment patterns. Use when containerizing applications, optimizing Docker images, setting up development environments, or deploying with Docker.
license: MIT
compatibility: opencode
---

# Docker Patterns Skill

## Overview

This skill provides guidelines for building production-ready Docker containers, multi-stage builds, security hardening, Docker Compose orchestration, and deployment best practices.

## Quick Reference

### Multi-Stage Build Example

```dockerfile
# Build stage
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine
RUN addgroup -g 1001 -S nodejs && adduser -S nodejs -u 1001
WORKDIR /app
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
USER nodejs
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"
CMD ["node", "dist/main.js"]
```

### Docker Compose Development

```yaml
version: '3.8'
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
```

## Core Principles

### 1. Multi-Stage Builds
- Separate build and runtime environments
- Minimize final image size
- Don't include build tools in production

### 2. Security Hardening
- Run as non-root user
- Use specific image tags (not `latest`)
- Scan images for vulnerabilities
- Drop unnecessary capabilities

### 3. Layer Caching
- Order instructions by change frequency
- Copy package files before source code
- Use `.dockerignore` to exclude unnecessary files

### 4. Health Checks
- Implement `/health` endpoints
- Configure Docker health checks
- Set appropriate intervals and timeouts

## Language-Specific Examples

See detailed guides in references/:

- **[Dockerfile Patterns](references/dockerfile-patterns.md)** - Multi-stage builds for Node.js, Python, Go, Rust
- **[Docker Compose Patterns](references/docker-compose-patterns.md)** - Development, production, and multi-environment setups
- **[Container Security](references/container-security.md)** - Security hardening, scanning, and best practices
- **[Deployment Patterns](references/deployment-patterns.md)** - Blue-green, rolling updates, Kubernetes

## Dockerfile Best Practices

```dockerfile
# ✅ DO: Use specific tags
FROM node:18.19.0-alpine3.18

# ✅ DO: Combine RUN commands
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    rm -rf /var/lib/apt/lists/*

# ✅ DO: Create non-root user
RUN addgroup -g 1001 -S appgroup && \
    adduser -S appuser -u 1001 -G appgroup
USER appuser

# ✅ DO: Copy with ownership
COPY --chown=appuser:appgroup . /app

# ✅ DO: Add health check
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost:3000/health || exit 1
```

## Docker Compose Best Practices

```yaml
# ✅ DO: Use environment files
services:
  app:
    env_file:
      - .env.production

# ✅ DO: Set resource limits
deploy:
  resources:
    limits:
      cpus: '0.5'
      memory: 512M

# ✅ DO: Configure health checks
healthcheck:
  test: ["CMD", "wget", "--spider", "http://localhost:3000/health"]
  interval: 30s
  timeout: 10s
  retries: 3
```

## Image Scanning

```bash
# Using Trivy
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy:latest image myapp:latest

# Using Docker Scout
docker scout cves myapp:latest

# Using Snyk
snyk container test myapp:latest
```

## When to Use This Skill

Use this skill when:
- Creating Dockerfiles for production applications
- Setting up development environments with Docker Compose
- Optimizing image sizes with multi-stage builds
- Hardening containers for security
- Setting up health checks and monitoring
- Managing multiple environments (dev/staging/prod)
- Implementing CI/CD pipelines with Docker
- Troubleshooting container issues

## Related Skills

- `@ci-cd-pipelines` - Continuous integration and deployment with GitHub Actions
- `@security-best-practices` - Container security and vulnerability scanning
- `@feature-development` - Development workflow
- `@python-patterns` - Python-specific patterns
- `@go-conventions` - Go-specific patterns
- `@ts-react-nextjs` - Node.js patterns
