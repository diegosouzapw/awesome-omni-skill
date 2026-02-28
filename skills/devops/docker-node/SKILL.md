---
name: docker-node
description: Containerization for TypeScript/Node.js applications. Use when deploying Node.js backends, need consistent dev environments, or setting up CI/CD pipelines. Covers multi-stage builds, docker-compose for development, and production optimization. Choose this skill for containerizing tRPC/Express APIs with Prisma.
allowed-tools: Read, Edit, Write, Bash (*)
---

# Docker (Node.js Containerization)

## Overview

Docker enables consistent environments for Node.js applications across development, testing, and production. Multi-stage builds reduce image size, docker-compose simplifies local development.

**Base Image**: `node:20-alpine` (recommended for small size)  
**Use case**: Deploy TypeScript APIs, ensure consistent environments

**Key Benefit**: "Works on my machine" → "Works everywhere"

## When to Use This Skill

✅ **Use Docker when:**
- Deploying to cloud (AWS, GCP, Azure)
- Need consistent dev environment across team
- Running CI/CD pipelines
- Deploying with Kubernetes
- Need isolated PostgreSQL/Redis for development

❌ **Skip Docker when:**
- Simple scripts or CLI tools
- Serverless deployments (use platform's build)
- Early prototyping (adds complexity)

---

## Multi-Stage Dockerfile (Production)

```dockerfile
# Dockerfile

# Stage 1: Dependencies
FROM node:20-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

# Stage 2: Build
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY tsconfig.json ./
COPY prisma ./prisma/
COPY src ./src/
RUN npx prisma generate
RUN npm run build

# Stage 3: Production
FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production

# Security: non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001 -G nodejs

# Copy production artifacts
COPY --from=deps --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --from=builder --chown=nodejs:nodejs /app/prisma ./prisma
COPY --from=builder --chown=nodejs:nodejs /app/node_modules/.prisma ./node_modules/.prisma

USER nodejs
EXPOSE 3000

# Run migrations then start
CMD ["sh", "-c", "npx prisma migrate deploy && node dist/index.js"]
```

---

## Docker Compose (Development)

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build:
      context: .
      target: builder  # Use builder stage for dev
    ports:
      - "3000:3000"
    environment:
      NODE_ENV: development
      DATABASE_URL: postgresql://postgres:postgres@postgres:5432/myapp
    volumes:
      - ./src:/app/src:delegated    # Hot reload
      - ./prisma:/app/prisma:delegated
    depends_on:
      postgres:
        condition: service_healthy
    command: npm run dev

  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: myapp
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres -d myapp"]
      interval: 5s
      timeout: 5s
      retries: 10

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

---

## .dockerignore

```dockerignore
# Dependencies
node_modules

# Build outputs
dist
.next

# Git
.git
.gitignore

# Environment
.env
.env.*
!.env.example

# IDE
.vscode
.idea

# Docker
docker-compose*
Dockerfile*

# Tests & docs
coverage
*.md
**/*.test.ts
**/*.spec.ts

# OS
.DS_Store
Thumbs.db
```

---

## Commands

### Development

```bash
# Start all services
docker-compose up

# Start with rebuild
docker-compose up --build

# Start in background
docker-compose up -d

# View logs
docker-compose logs -f app

# Stop all
docker-compose down

# Stop and remove volumes (reset DB)
docker-compose down -v
```

### Production Build

```bash
# Build production image
docker build -t myapp:latest .

# Run container
docker run -p 3000:3000 \
  -e DATABASE_URL="postgresql://..." \
  -e JWT_SECRET="..." \
  myapp:latest

# Run with env file
docker run -p 3000:3000 --env-file .env.production myapp:latest
```

### Debug

```bash
# Shell into running container
docker-compose exec app sh

# Shell into new container
docker run -it myapp:latest sh

# View container processes
docker-compose top

# Check image size
docker images myapp
```

---

## Database Migrations in Docker

### Development (Auto-migrate)

```yaml
# docker-compose.yml
app:
  command: sh -c "npx prisma migrate dev && npm run dev"
```

### Production (Deploy migrations)

```dockerfile
# In Dockerfile CMD
CMD ["sh", "-c", "npx prisma migrate deploy && node dist/index.js"]
```

### CI/CD Pipeline

```bash
# Run migrations in separate step
docker run --rm \
  -e DATABASE_URL="$PROD_DATABASE_URL" \
  myapp:latest \
  npx prisma migrate deploy
```

---

## Health Checks

### Application Health

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1
```

### Health Endpoint

```typescript
// src/routes/health.ts
app.get('/health', async (req, res) => {
  try {
    await prisma.$queryRaw`SELECT 1`;
    res.json({ status: 'healthy', db: 'connected' });
  } catch {
    res.status(503).json({ status: 'unhealthy', db: 'disconnected' });
  }
});
```

---

## Environment Variables

### Development

```yaml
# docker-compose.yml
services:
  app:
    environment:
      NODE_ENV: development
      DATABASE_URL: postgresql://postgres:postgres@postgres:5432/myapp
      JWT_SECRET: dev-secret-not-for-production
```

### Production

```bash
# Pass at runtime
docker run -e DATABASE_URL="..." -e JWT_SECRET="..." myapp

# Or use env file
docker run --env-file .env.production myapp
```

---

## Optimization Tips

### Reduce Image Size

```dockerfile
# Use alpine base
FROM node:20-alpine

# Clean npm cache
RUN npm ci && npm cache clean --force

# Don't install devDependencies in production
RUN npm ci --only=production
```

### Layer Caching

```dockerfile
# Copy package files first (changes less often)
COPY package*.json ./
RUN npm ci

# Then copy source (changes more often)
COPY . .
RUN npm run build
```

### Security

```dockerfile
# Run as non-root
RUN adduser -S nodejs
USER nodejs

# Don't expose unnecessary ports
EXPOSE 3000

# Use specific versions
FROM node:20.10-alpine
```

---

## Rules

### Do ✅

- Use multi-stage builds for production
- Run containers as non-root user
- Use `.dockerignore` to exclude unnecessary files
- Add health checks
- Pin base image versions
- Use `docker-compose` for local development

### Avoid ❌

- Running as root in production
- Storing secrets in Dockerfile
- Using `latest` tag in production
- Including `node_modules` in image
- Skipping `.dockerignore`

---

## Troubleshooting

```yaml
"Prisma client not generated":
  → Add RUN npx prisma generate in builder stage
  → Copy node_modules/.prisma to runner stage

"Permission denied":
  → Check file ownership with --chown
  → Ensure USER matches file owner

"Container exits immediately":
  → Check logs: docker-compose logs app
  → Verify CMD is blocking (not backgrounded)

"Can't connect to database":
  → Use service name as host (postgres, not localhost)
  → Check depends_on with healthcheck
  → Verify DATABASE_URL uses container network

"Image too large":
  → Use alpine base image
  → Add .dockerignore
  → Use multi-stage builds
  → Clean npm cache
```

---

## File Structure

```
project/
├── Dockerfile
├── docker-compose.yml
├── .dockerignore
├── .env.example
├── package.json
├── tsconfig.json
├── prisma/
│   └── schema.prisma
└── src/
    └── index.ts
```

## References

- https://docs.docker.com — Official documentation
- https://docs.docker.com/compose/ — Docker Compose
- https://hub.docker.com/_/node — Node.js images
