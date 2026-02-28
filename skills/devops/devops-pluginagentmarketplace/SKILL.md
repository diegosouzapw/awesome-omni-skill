---
name: devops
description: Deploy applications with Docker and Kubernetes, automate with CI/CD, manage infrastructure with code, and configure cloud platforms and networking.
sasmp_version: "2.0.0"
bonded_agent: 06-devops-infrastructure
bond_type: PRIMARY_BOND

# === PRODUCTION-GRADE SKILL CONFIG (SASMP v2.0.0) ===

atomic_operations:
  - CONTAINERIZATION
  - KUBERNETES_DEPLOYMENT
  - CI_CD_PIPELINE
  - INFRASTRUCTURE_PROVISIONING

parameter_validation:
  query:
    type: string
    required: true
    minLength: 5
    maxLength: 2500
  platform:
    type: string
    enum: [aws, gcp, azure, on-prem]
    required: false
  orchestrator:
    type: string
    enum: [kubernetes, docker-swarm, ecs]
    required: false

retry_logic:
  max_attempts: 3
  backoff: exponential
  initial_delay_ms: 2000

logging_hooks:
  on_invoke: "skill.devops.invoked"
  on_success: "skill.devops.completed"
  on_error: "skill.devops.failed"

exit_codes:
  SUCCESS: 0
  INVALID_INPUT: 1
  DEPLOYMENT_FAILED: 2
  SECURITY_ISSUE: 3
---

# DevOps & Infrastructure Skill

**Bonded to:** `devops-infrastructure-agent`

---

## Quick Start

```bash
# Invoke devops skill
"Containerize my Python application with Docker"
"Set up a CI/CD pipeline with GitHub Actions"
"Deploy my app to Kubernetes"
```

---

## Instructions

1. **Containerize**: Create optimized Docker images
2. **Orchestrate**: Configure Kubernetes or Docker Swarm
3. **Automate**: Set up CI/CD pipelines
4. **Provision**: Deploy infrastructure with Terraform
5. **Monitor**: Configure observability stack

---

## Docker Best Practices

### Optimized Dockerfile
```dockerfile
# Multi-stage build for smaller images
FROM python:3.12-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

FROM python:3.12-slim
RUN useradd --create-home appuser
USER appuser
WORKDIR /app
COPY --from=builder /root/.local /home/appuser/.local
ENV PATH=/home/appuser/.local/bin:$PATH
COPY --chown=appuser:appuser . .
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:8000/health || exit 1
EXPOSE 8000
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Image Checklist
- [ ] Use slim/alpine base images
- [ ] Multi-stage builds
- [ ] Non-root user
- [ ] Health checks
- [ ] .dockerignore file

---

## Kubernetes Essentials

### Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-server
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api-server
  template:
    metadata:
      labels:
        app: api-server
    spec:
      containers:
      - name: api-server
        image: myapp:v1.0.0
        ports:
        - containerPort: 8000
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
        readinessProbe:
          httpGet:
            path: /ready
            port: 8000
```

---

## CI/CD Pipeline (GitHub Actions)

```yaml
name: CI/CD

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
      - run: pip install -r requirements.txt
      - run: pytest --cov=app tests/

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: docker/build-push-action@v5
        with:
          push: true
          tags: myregistry/myapp:${{ github.sha }}

  deploy:
    needs: build
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - run: kubectl set image deployment/api-server api-server=myregistry/myapp:${{ github.sha }}
```

---

## Decision Tree

```
Deployment need?
    │
    ├─→ Simple app → Docker Compose
    │
    ├─→ Production, scaling needed
    │     ├─→ Cloud-native → Kubernetes
    │     └─→ AWS only → ECS
    │
    └─→ Serverless → AWS Lambda / Cloud Functions
```

---

## Troubleshooting

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Pod CrashLoopBackOff | App crash | Check logs: `kubectl logs <pod>` |
| ImagePullBackOff | Registry auth | Verify imagePullSecrets |
| OOMKilled | Memory limit | Increase limits or optimize |
| Pending pods | No resources | Scale cluster |

### Debug Commands

```bash
# Pod status
kubectl get pods -o wide

# Pod logs
kubectl logs <pod> --previous

# Describe pod
kubectl describe pod <pod>

# Events
kubectl get events --sort-by='.lastTimestamp'

# Exec into pod
kubectl exec -it <pod> -- /bin/sh
```

---

## Test Template

```python
# tests/test_docker.py
import subprocess

class TestDockerBuild:
    def test_dockerfile_builds_successfully(self):
        result = subprocess.run(
            ["docker", "build", "-t", "test-image", "."],
            capture_output=True, text=True
        )
        assert result.returncode == 0

    def test_container_starts_and_healthy(self):
        subprocess.run(["docker", "run", "-d", "--name", "test", "test-image"])
        # Wait for health check
        result = subprocess.run(
            ["docker", "inspect", "--format", "{{.State.Health.Status}}", "test"],
            capture_output=True, text=True
        )
        assert "healthy" in result.stdout
```

---

## Resources

- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Docker Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
