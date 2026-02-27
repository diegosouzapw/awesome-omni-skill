---
name: devops-infra-github
description: Expert guidance for containerization, orchestration, and CI/CD pipelines for Bun monorepo projects.
---

# DevOps & Infrastructure Skill

## Overview
Expert guidance for containerization, orchestration, and CI/CD pipelines for Bun monorepo projects.

## When to Use This Skill
- Creating Docker images for services
- Setting up Docker Compose for local development
- Configuring GitHub Actions for CI/CD
- Managing environment variables and secrets
- Automating deployments

## Stack Context
- **Runtime:** Bun
- **Containerization:** Docker (multi-stage builds with `oven/bun:alpine`)
- **Orchestration:** Docker Compose
- **CI/CD:** GitHub Actions
- **Version Management:** Changesets
- **Registry:** Docker Hub / GitHub Container Registry

## Project Structure

```
.
├── .github/
│   └── workflows/
│       ├── service-a-ci.yml
│       └── service-b-ci.yml
├── packages/
│   ├── service-a/
│   │   ├── Dockerfile
│   │   ├── .dockerignore
│   │   └── docker-compose.yml
│   └── service-b/
│       ├── Dockerfile
│       ├── .dockerignore
│       └── docker-compose.yml
├── .changeset/
└── package.json
```

## Workflows

### 1. Dockerfile for Bun Service

`packages/my-service/Dockerfile`:
```dockerfile
# Multi-stage build for optimized image size
FROM oven/bun:alpine AS base
WORKDIR /app

# Install dependencies stage
FROM base AS deps
COPY package.json bun.lockb ./
RUN bun install --frozen-lockfile --production

# Build stage (if needed)
FROM base AS builder
COPY package.json bun.lockb ./
COPY src ./src
RUN bun install --frozen-lockfile

# Production stage
FROM base AS runner
WORKDIR /app

# Copy dependencies from deps stage
COPY --from=deps /app/node_modules ./node_modules
COPY package.json ./
COPY src ./src

# Create non-root user
RUN addgroup -g 1001 -S bunuser && \
    adduser -S bunuser -u 1001
USER bunuser

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

# Start application
CMD ["bun", "src/index.js"]
```

`.dockerignore`:
```
node_modules
.env
.env.*
*.log
.git
.gitignore
README.md
CHANGELOG.md
.changeset
dist
coverage
.vscode
.idea
```

### 2. Docker Compose for Local Development

`packages/my-service/docker-compose.yml`:
```yaml
version: '3.9'

services:
  # Main service
  api:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: my-service-api
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
      - PORT=3000
      - MONGO_URI=mongodb://mongo:27017
      - DB_NAME=mydb
      - REDIS_URI=redis://redis:6379
    depends_on:
      mongo:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - app-network
    restart: unless-stopped
    volumes:
      # Hot reload for development
      - ./src:/app/src:ro
      - ./package.json:/app/package.json:ro

  # MongoDB
  mongo:
    image: mongo:7-jammy
    container_name: my-service-mongo
    ports:
      - "27017:27017"
    environment:
      - MONGO_INITDB_DATABASE=mydb
    volumes:
      - mongo-data:/data/db
      - ./mongo-init.js:/docker-entrypoint-initdb.d/mongo-init.js:ro
    networks:
      - app-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s

  # Redis
  redis:
    image: redis:7-alpine
    container_name: my-service-redis
    ports:
      - "6379:6379"
    command: redis-server --appendonly yes
    volumes:
      - redis-data:/data
    networks:
      - app-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5
      start_period: 5s

volumes:
  mongo-data:
    driver: local
  redis-data:
    driver: local

networks:
  app-network:
    driver: bridge
```

**Usage:**
```bash
# Start all services
docker compose up -d

# View logs
docker compose logs -f

# Stop services
docker compose down

# Rebuild and start
docker compose up --build -d

# Remove volumes (clean state)
docker compose down -v
```

### 3. MongoDB Initialization Script

`packages/my-service/mongo-init.js`:
```javascript
// Run on first container start
db = db.getSiblingDB('mydb')

// Create collections
db.createCollection('users')
db.createCollection('posts')

// Create indexes
db.users.createIndex({ email: 1 }, { unique: true })
db.posts.createIndex({ createdAt: -1 })

// Insert seed data
db.users.insertMany([
  {
    name: 'Admin User',
    email: 'admin@example.com',
    createdAt: new Date()
  }
])

print('MongoDB initialized successfully')
```

### 4. GitHub Actions CI/CD Pipeline

`.github/workflows/service-a-ci.yml`:
```yaml
name: Service A CI/CD

on:
  push:
    branches: [main, develop]
    paths:
      - 'packages/service-a/**'
      - '.changeset/**'
  pull_request:
    branches: [main]
    paths:
      - 'packages/service-a/**'

env:
  SERVICE_NAME: service-a
  SERVICE_PATH: packages/service-a
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}/service-a

jobs:
  # Lint and test
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Bun
        uses: oven-sh/setup-bun@v1
        with:
          bun-version: latest
      
      - name: Install dependencies
        run: bun install --frozen-lockfile
        working-directory: ${{ env.SERVICE_PATH }}
      
      - name: Lint
        run: bun run lint
        working-directory: ${{ env.SERVICE_PATH }}
      
      - name: Test
        run: bun test
        working-directory: ${{ env.SERVICE_PATH }}

  # Check for version changes
  version-check:
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    outputs:
      has-changeset: ${{ steps.changeset.outputs.hasChangesets }}
      version: ${{ steps.get-version.outputs.version }}
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      
      - name: Setup Bun
        uses: oven-sh/setup-bun@v1
      
      - name: Install dependencies
        run: bun install --frozen-lockfile
      
      - name: Check for changesets
        id: changeset
        uses: changesets/action@v1
        with:
          version: bun run version
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Get package version
        id: get-version
        run: |
          VERSION=$(node -p "require('./package.json').version")
          echo "version=$VERSION" >> $GITHUB_OUTPUT
        working-directory: ${{ env.SERVICE_PATH }}

  # Build and push Docker image
  build-and-push:
    needs: [test, version-check]
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v4
      
      - name: Log in to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=semver,pattern={{version}},value=${{ needs.version-check.outputs.version }}
            type=semver,pattern={{major}}.{{minor}},value=${{ needs.version-check.outputs.version }}
            type=raw,value=latest,enable={{is_default_branch}}
            type=sha,prefix={{branch}}-
      
      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: ${{ env.SERVICE_PATH }}
          file: ${{ env.SERVICE_PATH }}/Dockerfile
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}

  # Create GitHub Release
  release:
    needs: [version-check, build-and-push]
    if: needs.version-check.outputs.has-changeset == 'true'
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
      
      - name: Create Release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{ env.SERVICE_NAME }}@v${{ needs.version-check.outputs.version }}
          release_name: ${{ env.SERVICE_NAME }} v${{ needs.version-check.outputs.version }}
          body_path: ${{ env.SERVICE_PATH }}/CHANGELOG.md
          draft: false
          prerelease: false

  # Deploy (placeholder)
  deploy:
    needs: [build-and-push]
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to Production
        run: |
          echo "Deployment step - integrate with your deployment platform"
          echo "Image: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ needs.version-check.outputs.version }}"
          # Add deployment commands here (e.g., kubectl, SSH, etc.)
```

### 5. Multi-Service CI/CD with Matrix

`.github/workflows/monorepo-ci.yml`:
```yaml
name: Monorepo CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  # Detect changed services
  detect-changes:
    runs-on: ubuntu-latest
    outputs:
      services: ${{ steps.filter.outputs.changes }}
    steps:
      - uses: actions/checkout@v4
      
      - uses: dorny/paths-filter@v2
        id: filter
        with:
          filters: |
            service-a:
              - 'packages/service-a/**'
            service-b:
              - 'packages/service-b/**'
            web:
              - 'packages/web/**'

  # Test changed services
  test:
    needs: detect-changes
    if: ${{ needs.detect-changes.outputs.services != '[]' }}
    runs-on: ubuntu-latest
    strategy:
      matrix:
        service: ${{ fromJSON(needs.detect-changes.outputs.services) }}
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Bun
        uses: oven-sh/setup-bun@v1
      
      - name: Install dependencies
        run: bun install --frozen-lockfile
      
      - name: Test ${{ matrix.service }}
        run: |
          cd packages/${{ matrix.service }}
          bun run lint
          bun test
```

### 6. Environment Variables Management

**Development (.env):**
```bash
# Service
PORT=3000
NODE_ENV=development

# MongoDB
MONGO_URI=mongodb://localhost:27017
DB_NAME=mydb_dev

# Redis
REDIS_URI=redis://localhost:6379

# Auth
API_TOKEN=dev-token-12345
```

**Production (GitHub Secrets):**
- `MONGO_URI`: Production MongoDB connection string
- `REDIS_URI`: Production Redis connection string
- `API_TOKEN`: Production API token
- `DOCKER_USERNAME`: Docker registry username
- `DOCKER_PASSWORD`: Docker registry password

**Using secrets in GitHub Actions:**
```yaml
- name: Deploy
  env:
    MONGO_URI: ${{ secrets.MONGO_URI }}
    REDIS_URI: ${{ secrets.REDIS_URI }}
    API_TOKEN: ${{ secrets.API_TOKEN }}
```

### 7. Docker Build Optimization

**Multi-architecture builds:**
```yaml
- name: Set up QEMU
  uses: docker/setup-qemu-action@v3

- name: Set up Docker Buildx
  uses: docker/setup-buildx-action@v3

- name: Build and push
  uses: docker/build-push-action@v5
  with:
    platforms: linux/amd64,linux/arm64
    push: true
    tags: ${{ steps.meta.outputs.tags }}
    cache-from: type=gha
    cache-to: type=gha,mode=max
```

### 8. Docker Compose for Production

`docker-compose.prod.yml`:
```yaml
version: '3.9'

services:
  api:
    image: ghcr.io/org/service-a:latest
    container_name: service-a-api
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - PORT=3000
    env_file:
      - .env.production
    depends_on:
      - mongo
      - redis
    networks:
      - app-network
    restart: always
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 512M
        reservations:
          cpus: '0.5'
          memory: 256M

  mongo:
    image: mongo:7-jammy
    container_name: service-a-mongo
    environment:
      - MONGO_INITDB_ROOT_USERNAME=${MONGO_USER}
      - MONGO_INITDB_ROOT_PASSWORD=${MONGO_PASSWORD}
    volumes:
      - mongo-data:/data/db
    networks:
      - app-network
    restart: always

  redis:
    image: redis:7-alpine
    container_name: service-a-redis
    command: redis-server --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis-data:/data
    networks:
      - app-network
    restart: always

volumes:
  mongo-data:
  redis-data:

networks:
  app-network:
    driver: bridge
```

### 9. Deployment Script

`deploy.sh`:
```bash
#!/bin/bash
set -e

SERVICE_NAME=$1
VERSION=$2

if [ -z "$SERVICE_NAME" ] || [ -z "$VERSION" ]; then
  echo "Usage: ./deploy.sh <service-name> <version>"
  exit 1
fi

echo "Deploying $SERVICE_NAME version $VERSION..."

# Pull latest image
docker pull ghcr.io/org/$SERVICE_NAME:$VERSION

# Stop current container
docker compose -f packages/$SERVICE_NAME/docker-compose.prod.yml down

# Start new version
docker compose -f packages/$SERVICE_NAME/docker-compose.prod.yml up -d

# Check health
sleep 5
if curl -f http://localhost:3000/health; then
  echo "✅ Deployment successful"
else
  echo "❌ Health check failed"
  exit 1
fi
```

### 10. Monitoring and Logging

**Docker logging configuration:**
```yaml
services:
  api:
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

**View logs:**
```bash
# Follow logs for all services
docker compose logs -f

# Follow logs for specific service
docker compose logs -f api

# View last 100 lines
docker compose logs --tail=100 api
```

## Best Practices

1. **Docker:**
   - Use multi-stage builds to reduce image size
   - Use `oven/bun:alpine` for smallest images
   - Run as non-root user
   - Include health checks
   - Use `.dockerignore` effectively

2. **Docker Compose:**
   - Use health checks for dependencies
   - Define restart policies
   - Use volumes for persistent data
   - Network isolation with custom networks
   - Set resource limits in production

3. **CI/CD:**
   - Run tests before building
   - Use path filters to only build changed services
   - Tag images with semantic versions
   - Cache dependencies for faster builds
   - Use GitHub Secrets for sensitive data

4. **Versioning:**
   - Use Changesets for version management
   - Automate version bumps on merge to main
   - Create GitHub Releases automatically
   - Tag format: `service-name@vX.X.X`

5. **Secrets:**
   - Never commit `.env` files
   - Use GitHub Secrets for CI/CD
   - Use environment-specific `.env` files
   - Rotate secrets regularly

6. **Deployment:**
   - Use health checks before routing traffic
   - Implement zero-downtime deployments
   - Keep rollback capability
   - Monitor logs and metrics

## Common Commands

**Docker:**
```bash
# Build image
docker build -t my-service:latest .

# Run container
docker run -p 3000:3000 my-service:latest

# View running containers
docker ps

# Stop container
docker stop <container-id>

# Remove container
docker rm <container-id>

# View logs
docker logs -f <container-id>
```

**Docker Compose:**
```bash
# Start services
docker compose up -d

# Stop services
docker compose down

# Rebuild services
docker compose build

# View logs
docker compose logs -f

# Execute command in container
docker compose exec api bun run migrate
```

## References

- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Changesets Documentation](https://github.com/changesets/changesets)
- [Bun Docker Image](https://hub.docker.com/r/oven/bun)
