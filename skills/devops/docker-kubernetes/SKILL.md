---
name: Docker & Kubernetes
description: Containerization, orchestration, and deployment with Docker and K8s
---

# Docker & Kubernetes Skill

## Docker Basics

### Dockerfile for Next.js
```dockerfile
# Build stage
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV production

# Copy necessary files
COPY --from=builder /app/next.config.js ./
COPY --from=builder /app/public ./public
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static

EXPOSE 3000
ENV PORT 3000
CMD ["node", "server.js"]
```

### Docker Compose
```yaml
version: '3.8'
services:
  web:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=${DATABASE_URL}
    depends_on:
      - db
      - redis
  
  db:
    image: postgres:16-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
  
  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

## Docker Commands

```bash
# Build image
docker build -t smarter-poker .

# Run container
docker run -p 3000:3000 smarter-poker

# View running containers
docker ps

# View logs
docker logs -f container_id

# Enter container shell
docker exec -it container_id sh

# Stop all containers
docker stop $(docker ps -q)

# Clean up
docker system prune -a
```

## Kubernetes Deployment

### deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: smarter-poker
spec:
  replicas: 3
  selector:
    matchLabels:
      app: smarter-poker
  template:
    metadata:
      labels:
        app: smarter-poker
    spec:
      containers:
      - name: web
        image: smarter-poker:latest
        ports:
        - containerPort: 3000
        resources:
          limits:
            memory: "512Mi"
            cpu: "500m"
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: database-url
---
apiVersion: v1
kind: Service
metadata:
  name: smarter-poker-service
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 3000
  selector:
    app: smarter-poker
```

## Useful Tools
- `lazydocker` - TUI for Docker
- `dive` - Analyze Docker image layers
- `hadolint` - Dockerfile linter
