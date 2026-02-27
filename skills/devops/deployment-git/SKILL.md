---
name: deployment-git
description: "Deployment and Git workflow guides for Kailash applications including Docker deployment, Kubernetes orchestration, and Git workflows. Use when asking about 'deployment', 'Docker deployment', 'Kubernetes deployment', 'containerization', 'K8s', 'Git workflow', 'Git branching', 'CI/CD', 'production deployment', 'Docker compose', or 'container orchestration'."
---

# Deployment & Git Workflows

Comprehensive guides for deploying Kailash applications with Docker and Kubernetes, plus Git workflow best practices.

## Overview

Production deployment patterns for:
- Docker containerization
- Kubernetes orchestration
- Git workflows and branching strategies
- CI/CD integration
- Environment management

## Reference Documentation

### Docker Deployment
- **[deployment-docker-quick](deployment-docker-quick.md)** - Docker deployment quick start
  - Dockerfile setup for Kailash apps
  - Docker Compose configurations
  - Multi-stage builds
  - Environment variables
  - Volume management
  - Health checks
  - Production optimizations

### Kubernetes Deployment
- **[deployment-kubernetes-quick](deployment-kubernetes-quick.md)** - Kubernetes deployment guide
  - Deployment manifests
  - Service configuration
  - ConfigMaps and Secrets
  - Persistent volumes
  - Health probes
  - Scaling strategies
  - Ingress setup

### Git Workflow
- **[git-workflow-quick](git-workflow-quick.md)** - Git workflow best practices
  - Branching strategies
  - Commit conventions
  - Pull request workflow
  - Code review process
  - Release management
  - Hotfix procedures

## Docker Patterns

### Basic Dockerfile
```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY . .

# Use AsyncLocalRuntime for Docker
ENV RUNTIME_TYPE=async

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:8000/health || exit 1

# Run with Nexus
CMD ["python", "-m", "app.main"]
```

### Docker Compose
```yaml
version: '3.8'
services:
  nexus:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/mydb
      - RUNTIME_TYPE=async
    depends_on:
      - db

  db:
    image: postgres:15
    environment:
      - POSTGRES_PASSWORD=pass
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

## Kubernetes Patterns

### Deployment Manifest
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kailash-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: kailash
  template:
    metadata:
      labels:
        app: kailash
    spec:
      containers:
      - name: app
        image: my-kailash-app:latest
        ports:
        - containerPort: 8000
        env:
        - name: RUNTIME_TYPE
          value: "async"
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 10
          periodSeconds: 30
```

## Git Workflow Patterns

### Branch Strategy
```
main (production)
  ↓
develop (integration)
  ↓
feature/* (new features)
hotfix/* (urgent fixes)
release/* (release prep)
```

### Commit Conventions
```
feat: Add user authentication workflow
fix: Resolve async runtime threading issue
docs: Update DataFlow integration guide
test: Add cycle workflow test cases
chore: Bump version to 0.9.25
```

## Critical Rules

### Docker
- ✅ Use AsyncLocalRuntime for Docker/FastAPI
- ✅ Implement health checks
- ✅ Use multi-stage builds for smaller images
- ✅ Set proper resource limits
- ✅ Use secrets for sensitive data
- ❌ NEVER use LocalRuntime in Docker (causes hangs)
- ❌ NEVER commit secrets to images
- ❌ NEVER run as root user

### Kubernetes
- ✅ Define resource requests and limits
- ✅ Use ConfigMaps for configuration
- ✅ Implement readiness and liveness probes
- ✅ Use Horizontal Pod Autoscaling
- ✅ Set up proper monitoring
- ❌ NEVER store secrets in plain text
- ❌ NEVER skip health checks
- ❌ NEVER use latest tag in production

### Git
- ✅ Use feature branches for development
- ✅ Write descriptive commit messages
- ✅ Squash commits before merging
- ✅ Use pull requests for code review
- ✅ Tag releases semantically
- ❌ NEVER commit directly to main
- ❌ NEVER force push to shared branches
- ❌ NEVER commit sensitive data

## Runtime Selection

| Environment | Runtime | Reason |
|-------------|---------|--------|
| **Docker** | AsyncLocalRuntime | No threading, async-first |
| **K8s** | AsyncLocalRuntime | Container-optimized |
| **FastAPI** | AsyncLocalRuntime | Native async support |
| **CLI** | LocalRuntime | Synchronous execution |
| **Scripts** | LocalRuntime | Simple sync context |

## When to Use This Skill

Use this skill when you need to:
- Deploy Kailash apps with Docker
- Set up Kubernetes deployments
- Configure CI/CD pipelines
- Establish Git workflows
- Containerize workflows
- Scale applications in production
- Manage environments and secrets

## Environment Management

### Development
```bash
# Local development
python -m app.main

# Docker development
docker-compose up
```

### Production
```bash
# Docker production
docker build -t app:prod .
docker run -d -p 8000:8000 app:prod

# Kubernetes production
kubectl apply -f k8s/
kubectl scale deployment kailash-app --replicas=5
```

## Related Skills

- **[03-nexus](../../03-nexus/SKILL.md)** - Application deployment
- **[02-dataflow](../../02-dataflow/SKILL.md)** - Database in containers
- **[01-core-sdk](../../01-core-sdk/SKILL.md)** - Runtime selection
- **[17-gold-standards](../../17-gold-standards/SKILL.md)** - Deployment best practices

## Support

For deployment help, invoke:
- `deployment-specialist` - Docker and Kubernetes expertise
- `git-release-specialist` - Git workflows and releases
- `nexus-specialist` - Application configuration
