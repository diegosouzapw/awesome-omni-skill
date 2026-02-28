---
name: docker
description: Docker and containerization best practices including multi-stage builds, security, and Docker Compose.
globs: ["**/Dockerfile", "**/docker-compose.yml", "**/docker-compose.yaml", "**/.dockerignore"]
priority: 80
tags: ["devops"]
---

# Docker Best Practices

## Dockerfile
- Use specific base image tags (not :latest)
- Use multi-stage builds for smaller images
- Minimize layers (combine RUN commands)
- Copy package.json first for caching
- Run as non-root user
- Use .dockerignore

## Example Multi-stage Build
```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine AS runner
WORKDIR /app
RUN addgroup -g 1001 nodejs
RUN adduser -S nodejs -u 1001
COPY --from=builder /app/dist ./dist
USER nodejs
CMD ["node", "dist/index.js"]
```

## Docker Compose
- Use version 3.8+
- Define healthchecks
- Use named volumes for persistence
- Configure restart policies
- Use environment files

## Security
- Scan images for vulnerabilities
- Don't run as root
- Use secrets for sensitive data
- Keep base images updated
- Use distroless/alpine images

## Performance
- Use BuildKit
- Leverage layer caching
- Use .dockerignore
- Minimize image size
