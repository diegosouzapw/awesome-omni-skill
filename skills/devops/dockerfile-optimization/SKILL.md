---
name: dockerfile-optimization
description: Optimize Dockerfiles for smaller images, faster builds, better caching, and security. Use this skill when writing, reviewing, or debugging Dockerfiles.
alwaysApply: false
---

# Dockerfile Optimization

You are a Docker expert. When writing or reviewing Dockerfiles, apply these best practices for size, speed, caching, and security.

## Multi-Stage Build Pattern

Always use multi-stage builds for compiled languages:

```dockerfile
# Stage 1: Build
FROM node:22-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --production=false
COPY . .
RUN npm run build

# Stage 2: Production
FROM node:22-alpine AS runtime
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package*.json ./
USER node
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

**Go example (even smaller — scratch base):**
```dockerfile
FROM golang:1.23-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w" -o /server ./cmd/server

FROM scratch
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
COPY --from=builder /server /server
EXPOSE 8080
ENTRYPOINT ["/server"]
```

## Layer Caching Rules

Docker caches each layer. When a layer changes, all layers after it are rebuilt.

### Maximize Cache Hits
```dockerfile
# BAD — any source change invalidates npm install cache
COPY . .
RUN npm ci

# GOOD — only re-install if package.json changes
COPY package*.json ./
RUN npm ci
COPY . .
```

### Order: Least-changing → Most-changing
1. Base image (rarely changes)
2. System packages (rarely changes)
3. Dependency manifests (changes occasionally)
4. Dependency install (changes with manifests)
5. Source code (changes frequently)
6. Build step (changes with source)

## Image Size Optimization

### Use Alpine or Distroless Base Images
| Base | Size | Use When |
|------|------|----------|
| `scratch` | 0 MB | Static Go binaries |
| `distroless` | ~2 MB | Java, Go, Python without shell |
| `alpine` | ~7 MB | Need a shell, package manager |
| `slim` | ~80 MB | Need Debian packages |
| `full` | ~200+ MB | Development only, never production |

### Reduce Layer Count
```dockerfile
# BAD — 3 layers
RUN apt-get update
RUN apt-get install -y curl
RUN rm -rf /var/lib/apt/lists/*

# GOOD — 1 layer, cleaned up
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    rm -rf /var/lib/apt/lists/*
```

### Use .dockerignore
Create a `.dockerignore` to exclude unnecessary files:
```
.git
node_modules
dist
*.md
.env*
.vscode
.idea
__pycache__
*.pyc
coverage
.next
```

## Security Best Practices

### Run as Non-Root
```dockerfile
# Create a non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

# Or use built-in user
USER node          # Node.js images
USER nobody:nobody # Generic
```

### Pin Versions
```dockerfile
# BAD — unpredictable
FROM node:latest
RUN apt-get install python3

# GOOD — reproducible
FROM node:22.12-alpine3.19
RUN apk add --no-cache python3=3.11.6-r0
```

### Don't Store Secrets in Images
```dockerfile
# BAD — secret baked into image
ENV API_KEY=sk-secret123
COPY .env .

# GOOD — pass at runtime
# docker run -e API_KEY=sk-secret123 myapp
# or use Docker secrets / mount
```

### Scan for Vulnerabilities
```bash
# Docker Scout
docker scout cves myimage:latest

# Trivy
trivy image myimage:latest

# Grype
grype myimage:latest
```

## Common Anti-Patterns

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| `FROM ubuntu:latest` | Large, unpredictable | Use `alpine` or `distroless`, pin version |
| `COPY . .` before `npm install` | Breaks caching | Copy manifests first, install, then copy source |
| `RUN apt-get update` alone | Stale package list cached | Combine with `install` in one `RUN` |
| `USER root` in production | Security risk | Create and switch to non-root user |
| No `.dockerignore` | Bloated context, slow builds | Add `.dockerignore` with exclusions |
| `ENTRYPOINT` without `exec` form | Signals not forwarded | Use `["executable", "arg"]` form |
| Not cleaning up in same layer | Larger image | Combine install + cleanup in one `RUN` |

## Health Checks
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1
```

## Debugging Dockerfiles
```bash
# Build with progress output
docker build --progress=plain .

# Build up to a specific stage
docker build --target builder .

# Inspect image layers
docker history myimage:latest

# Run shell in failed build
docker run -it --entrypoint /bin/sh myimage:latest

# Check image size breakdown
docker image inspect myimage:latest --format='{{.Size}}'
dive myimage:latest  # Interactive layer explorer
```
