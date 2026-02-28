---
name: docker-setup
description: Dockerfile and Docker Compose patterns with multi-stage builds, layer optimization, security hardening, and health checks. Use when containerizing applications, writing Dockerfiles, or setting up Docker Compose environments.
compatibility: Requires Docker
metadata:
  version: "1.0"
---

# Docker Setup

## Decision Tree

```
New container needed → What stack?
    ├─ Node/TypeScript → Use assets/Dockerfile.node.template
    ├─ Python → Use assets/Dockerfile.python.template
    └─ Other → Follow multi-stage pattern below
```

## Multi-Stage Build Pattern

```dockerfile
# Stage 1: Build
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Production
FROM node:20-alpine
RUN addgroup -S app && adduser -S app -G app
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
USER app
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=3s CMD wget -qO- http://localhost:3000/health || exit 1
CMD ["node", "dist/index.js"]
```

## Layer Optimization Rules

1. **COPY package files before source** - Dependency layer caches separately
2. **Use .dockerignore** - Exclude node_modules, .git, .env, tests
3. **Combine RUN commands** - Fewer layers = smaller image
4. **Use alpine bases** - 5MB vs 100MB+ for full images
5. **Pin versions** - `node:20.11-alpine` not `node:latest`

## Security Checklist

- [ ] Non-root USER in final stage
- [ ] No secrets in build args or ENV
- [ ] .dockerignore excludes .env, .git, node_modules, tests
- [ ] Base image pinned to specific version
- [ ] HEALTHCHECK defined
- [ ] No unnecessary packages installed

## For Compose patterns see:
- [Docker Compose patterns](references/compose-patterns.md)
