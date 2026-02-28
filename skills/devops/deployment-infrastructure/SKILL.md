---
name: deployment-infrastructure
description: Kubernetes deployment and infrastructure patterns
alwaysApply: false
applyTo:
  - "infra/**/*.yaml"
  - "**/Dockerfile"
  - ".github/workflows/**"
---

# Deployment & Infrastructure Skill

Guide for deploying Splits Network services and apps.

## Purpose

- **Kubernetes**: Manifest patterns for services
- **Docker**: Multi-stage build optimization
- **CI/CD**: GitHub Actions workflows
- **Configuration**: Environment variables and secrets

## When to Use

- Deploying new services
- Creating Docker images
- Setting up CI/CD pipelines
- Managing Kubernetes resources

## Core Patterns

### 1. Kubernetes Deployment

```yaml
# infra/k8s/ats-service/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ats-service
  namespace: splits-network
spec:
  replicas: 2
  selector:
    matchLabels:
      app: ats-service
  template:
    metadata:
      labels:
        app: ats-service
    spec:
      containers:
      - name: ats-service
        image: ghcr.io/splits-network/ats-service:latest
        ports:
        - containerPort: 3002
        env:
        - name: PORT
          value: "3002"
        - name: SUPABASE_URL
          valueFrom:
            secretKeyRef:
              name: supabase-credentials
              key: url
        - name: SUPABASE_ANON_KEY
          valueFrom:
            secretKeyRef:
              name: supabase-credentials
              key: anon-key
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3002
          initialDelaySeconds: 10
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 3002
          initialDelaySeconds: 5
          periodSeconds: 5
```

### 2. Kubernetes Service

```yaml
# infra/k8s/ats-service/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: ats-service
  namespace: splits-network
spec:
  selector:
    app: ats-service
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3002
  type: ClusterIP
```

### 3. Multi-Stage Dockerfile

```dockerfile
# Build stage
FROM node:20-alpine AS builder
WORKDIR /app

# Copy package files
COPY package.json pnpm-lock.yaml ./
COPY packages ./packages
COPY services/ats-service ./services/ats-service

# Install dependencies
RUN npm install -g pnpm
RUN pnpm install --frozen-lockfile

# Build
WORKDIR /app/services/ats-service
RUN pnpm build

# Production stage
FROM node:20-alpine AS runner
WORKDIR /app

# Copy built files
COPY --from=builder /app/services/ats-service/dist ./dist
COPY --from=builder /app/services/ats-service/package.json ./
COPY --from=builder /app/node_modules ./node_modules

# Run as non-root
USER node

EXPOSE 3002
CMD ["node", "dist/index.js"]
```

### 4. GitHub Actions Workflow

```yaml
# .github/workflows/deploy-ats-service.yml
name: Deploy ATS Service

on:
  push:
    branches: [main]
    paths:
      - 'services/ats-service/**'
      - 'packages/**'

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Build Docker image
        run: |
          docker build -t ghcr.io/splits-network/ats-service:${{ github.sha }} \
            -f services/ats-service/Dockerfile .
      
      - name: Push to registry
        run: |
          echo "${{ secrets.GITHUB_TOKEN }}" | docker login ghcr.io -u ${{ github.actor }} --password-stdin
          docker push ghcr.io/splits-network/ats-service:${{ github.sha }}
      
      - name: Deploy to Kubernetes
        run: |
          kubectl set image deployment/ats-service \
            ats-service=ghcr.io/splits-network/ats-service:${{ github.sha }} \
            -n splits-network
```

### 5. Health Check Endpoints

```typescript
// services/ats-service/src/health.ts
export async function healthRoutes(app: FastifyInstance) {
  // Liveness probe - is service running?
  app.get('/health', async (request, reply) => {
    return reply.send({ status: 'ok' });
  });

  // Readiness probe - can service handle traffic?
  app.get('/ready', async (request, reply) => {
    try {
      // Check database connection
      const { error } = await supabase.from('jobs').select('id').limit(1);
      if (error) throw error;
      
      return reply.send({ status: 'ready' });
    } catch (error) {
      return reply.code(503).send({ status: 'not ready' });
    }
  });
}
```

### 6. Environment Configuration

```typescript
// Kubernetes Secret
apiVersion: v1
kind: Secret
metadata:
  name: supabase-credentials
  namespace: splits-network
type: Opaque
stringData:
  url: "https://einhgkqmxbkgdohwfayv.supabase.co"
  anon-key: "eyJhbGc..."
  service-role-key: "eyJhbGc..."
```

### 7. Ingress Configuration

```yaml
# infra/k8s/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: splits-network-ingress
  namespace: splits-network
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
  - hosts:
    - api.splits.network
    secretName: api-tls
  rules:
  - host: api.splits.network
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-gateway
            port:
              number: 80
```

See [examples/](./examples/) and [references/](./references/).
