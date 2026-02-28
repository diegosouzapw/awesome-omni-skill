---
name: containers-skill
description: Docker and Kubernetes - containerization, orchestration, and production deployment.
sasmp_version: "1.3.0"
bonded_agent: 03-containers-orchestration
bond_type: PRIMARY_BOND

parameters:
  - name: platform
    type: string
    required: false
    enum: ["docker", "kubernetes", "both"]
    default: "both"
  - name: operation
    type: string
    required: true
    enum: ["build", "deploy", "debug", "optimize", "secure"]

retry_config:
  strategy: exponential_backoff
  initial_delay_ms: 1000
  max_retries: 3

observability:
  logging: structured
  metrics: enabled
---

# Containers & Orchestration Skill

## Overview
Master Docker and Kubernetes for production deployments.

## Parameters
| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| platform | string | No | both | docker/kubernetes |
| operation | string | Yes | - | Operation type |

## Core Topics

### MANDATORY
- Docker fundamentals (images, containers, volumes)
- Dockerfile best practices (multi-stage, security)
- Docker Compose
- Kubernetes architecture
- Deployments, Services, Ingress
- Health checks

### OPTIONAL
- Helm charts
- ConfigMaps and Secrets
- Persistent storage
- Network policies

### ADVANCED
- Custom operators
- Service mesh
- Multi-cluster strategies

## Quick Reference

```bash
# Docker
docker build -t app:v1 .
docker run -d -p 8080:80 --name app app:v1
docker logs -f container
docker exec -it container sh
docker system prune -af

# Docker Compose
docker compose up -d
docker compose logs -f
docker compose down -v

# Kubernetes
kubectl get pods -A
kubectl describe pod pod-name
kubectl logs -f pod-name
kubectl exec -it pod-name -- sh
kubectl apply -f manifest.yaml
kubectl rollout status deployment/app
kubectl rollout undo deployment/app

# Debugging
kubectl get events --sort-by='.lastTimestamp'
kubectl top pods
kubectl run debug --rm -it --image=busybox -- sh

# Helm
helm install release chart
helm upgrade release chart
helm rollback release 1
```

## Troubleshooting

### Common Failures
| Symptom | Root Cause | Solution |
|---------|------------|----------|
| ImagePullBackOff | Image not found | Verify name, check creds |
| CrashLoopBackOff | Container crashing | Check logs, verify CMD |
| Pending | Cannot schedule | Check resources, selectors |
| OOMKilled | Out of memory | Increase limits |

### Debug Checklist
1. Pod status: `kubectl get pods -o wide`
2. Events: `kubectl describe pod`
3. Logs: `kubectl logs pod --previous`
4. Resources: `kubectl top pods`

### Recovery Procedures

#### CrashLoopBackOff
1. Get logs: `kubectl logs pod --previous`
2. Check events: `kubectl describe pod`
3. Test locally: `docker run -it image sh`

## Resources
- [Docker Docs](https://docs.docker.com)
- [Kubernetes Docs](https://kubernetes.io/docs)
