---
name: docker-k8s
description: Docker and Kubernetes containerization best practices. Use when writing Dockerfiles, docker-compose files, or Kubernetes manifests. Triggers on mentions of Docker, container, Dockerfile, docker-compose, Kubernetes, k8s, pods, deployments, helm.
---

# Docker and Kubernetes Best Practices

## Dockerfile Best Practices

### Multi-Stage Builds
```dockerfile
# Build stage
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

# Production stage
FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
USER node
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

### Layer Optimization
```dockerfile
# Bad - cache invalidation on any file change
COPY . .
RUN npm install

# Good - dependencies cached separately
COPY package*.json ./
RUN npm ci
COPY . .
```

### Security
```dockerfile
# Use specific version tags
FROM python:3.11-slim

# Run as non-root user
RUN useradd -m -r appuser
USER appuser

# Don't store secrets in image
# Use runtime environment variables or secrets

# Minimize attack surface
RUN apt-get update && apt-get install -y --no-install-recommends \
    package1 \
    package2 \
    && rm -rf /var/lib/apt/lists/*
```

### Python Dockerfile
```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install dependencies first for caching
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

# Non-root user
RUN useradd -m appuser && chown -R appuser:appuser /app
USER appuser

EXPOSE 8000
CMD ["gunicorn", "-w", "4", "-b", "0.0.0.0:8000", "app:app"]
```

## Docker Compose

### Development Setup
```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    volumes:
      - .:/app
      - /app/node_modules  # Preserve node_modules
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
    depends_on:
      - db
      - redis

  db:
    image: postgres:15-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

### Production Setup
```yaml
version: '3.8'

services:
  app:
    image: myapp:${VERSION:-latest}
    restart: unless-stopped
    ports:
      - "8080:8080"
    environment:
      - DATABASE_URL
      - REDIS_URL
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 512M
```

## Kubernetes Basics

### Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  labels:
    app: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:1.0.0
        ports:
        - containerPort: 8080
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "500m"
            memory: "512Mi"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: myapp-secrets
              key: database-url
```

### Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP
```

### Ingress
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp
            port:
              number: 80
```

### ConfigMap
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-config
data:
  LOG_LEVEL: "info"
  MAX_CONNECTIONS: "100"
```

### Secret
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: myapp-secrets
type: Opaque
stringData:
  database-url: "postgres://user:pass@host/db"
  api-key: "secret-key"
```

## Resource Management

### Requests and Limits
```yaml
resources:
  requests:
    cpu: "100m"      # 0.1 CPU
    memory: "128Mi"  # 128 MiB
  limits:
    cpu: "500m"      # 0.5 CPU
    memory: "512Mi"  # 512 MiB
```

### Horizontal Pod Autoscaler
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

## Health Checks

### Liveness vs Readiness
```yaml
# Liveness - Is the container healthy?
# Restart if unhealthy
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
  failureThreshold: 3

# Readiness - Can it accept traffic?
# Remove from service if not ready
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5

# Startup - For slow-starting containers
startupProbe:
  httpGet:
    path: /health
    port: 8080
  failureThreshold: 30
  periodSeconds: 10
```

## Helm Charts

### Chart Structure
```
mychart/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   └── _helpers.tpl
```

### values.yaml
```yaml
replicaCount: 3

image:
  repository: myapp
  tag: "1.0.0"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 100m
    memory: 128Mi
```

### Template Example
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "mychart.fullname" . }}
spec:
  replicas: {{ .Values.replicaCount }}
  template:
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        resources:
          {{- toYaml .Values.resources | nindent 12 }}
```

## Common Commands

### Docker
```bash
docker build -t myapp:1.0 .
docker run -d -p 8080:8080 myapp:1.0
docker logs -f container_id
docker exec -it container_id sh
docker system prune -a
```

### Kubernetes
```bash
kubectl apply -f deployment.yaml
kubectl get pods
kubectl describe pod myapp-xxx
kubectl logs myapp-xxx
kubectl exec -it myapp-xxx -- sh
kubectl port-forward svc/myapp 8080:80
kubectl rollout restart deployment/myapp
kubectl scale deployment myapp --replicas=5
```

## Anti-Patterns to Avoid

- Using `latest` tag in production
- Running as root
- Storing secrets in images
- Not setting resource limits
- Missing health checks
- Large base images
- Not using .dockerignore
- Single container per pod (when multiple make sense)
