---
name: gke-deployment
description: Deploy, configure, and manage Kubernetes workloads on GKE with Deployments, Services, Ingress, HPA, health probes, ConfigMaps, and Secrets. Use when deploying containers to GKE, configuring load balancers, setting up autoscaling, writing health checks, managing environment configs, or troubleshooting pod issues.
license: MIT
metadata:
  version: 1.0.0
  audience: developers
  workflow: infrastructure
---

# GKE Deployment

Production-ready Kubernetes deployment patterns for Google Kubernetes Engine.

## What I Do

- Write Kubernetes Deployments with proper update strategies
- Configure Services (ClusterIP, NodePort, LoadBalancer) and Ingress
- Implement HPA with CPU, memory, and custom metrics
- Define resource requests/limits and health probes
- Manage ConfigMaps, Secrets, and Workload Identity

## When to Use Me

- Deploy applications or microservices to GKE
- Configure Ingress with HTTPS and managed certificates
- Set up autoscaling based on metrics
- Write health check endpoints and probe configurations
- Troubleshoot pod crashes, restarts, or scheduling issues
- Implement blue-green or canary deployment strategies

## Deployment Patterns

### Rolling Update (Zero Downtime)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: my-app
  template:
    spec:
      containers:
      - name: my-app
        image: us-docker.pkg.dev/PROJECT/REPO/my-app:TAG  # Artifact Registry (preferred)
        # Or legacy: gcr.io/PROJECT/my-app:TAG
        resources:
          requests:
            cpu: 100m
            memory: 256Mi
          limits:
            cpu: 500m
            memory: 512Mi
```

## Service Configuration

| Type | Use Case | External |
|------|----------|----------|
| ClusterIP | Internal services | No |
| NodePort | Dev, custom LB | Via node |
| LoadBalancer | Direct external | GCP L4 LB |

## Health Probes

```yaml
containers:
- name: my-app
  startupProbe:
    httpGet: {path: /healthz, port: 8080}
    periodSeconds: 10
    failureThreshold: 30
  livenessProbe:
    httpGet: {path: /healthz, port: 8080}
    periodSeconds: 15
    failureThreshold: 3
  readinessProbe:
    httpGet: {path: /ready, port: 8080}
    periodSeconds: 5
    failureThreshold: 3
```

| Probe | Purpose | On Failure |
|-------|---------|------------|
| Startup | Wait for slow apps | Block other probes |
| Liveness | Detect deadlocks | Restart container |
| Readiness | Control traffic | Remove from Service |

## Horizontal Pod Autoscaler

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
```

## Context7 Integration

Use Context7 MCP server for up-to-date Kubernetes docs:

```
context7_resolve-library-id("kubernetes", "HPA configuration")
context7_query-docs("/kubernetes/website", "Ingress path types")
```

## Quick Decision Matrix

| Need | Solution |
|------|----------|
| Zero-downtime deploy | `maxUnavailable: 0` |
| External HTTPS | Ingress + ManagedCertificate |
| Auto-scale on load | HPA with CPU target |
| Slow app startup | startupProbe, high failureThreshold |
| Pod spread across zones | topologySpreadConstraints |
| GCP API access | Workload Identity |

## Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| `CrashLoopBackOff` | App crashes | Check logs, verify probes |
| `ImagePullBackOff` | Can't pull image | Verify path, imagePullSecrets |
| `Pending` pod | No resources | Check capacity, adjust requests |
| `OOMKilled` | Memory exceeded | Increase limit or fix leak |
| `Unhealthy` backend | Health check fails | Ensure `/healthz` returns 200 |

## Resource Guidelines

| Workload | CPU | Memory |
|----------|-----|--------|
| Web API | 100m-500m | 256Mi-512Mi |
| Worker | 250m-1000m | 512Mi-1Gi |
| Sidecar | 10m-50m | 32Mi-64Mi |

## Security Checklist

- [ ] `runAsNonRoot: true`
- [ ] `readOnlyRootFilesystem: true`
- [ ] Drop all capabilities
- [ ] Workload Identity for GCP access
- [ ] NetworkPolicies applied
- [ ] PodDisruptionBudgets configured

## GKE-Specific Patterns

### Workload Identity (GCP API Access)
```yaml
# ServiceAccount annotation
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-app
  annotations:
    iam.gke.io/gcp-service-account: my-app@PROJECT.iam.gserviceaccount.com
```

```bash
# Bind KSA to GSA
gcloud iam service-accounts add-iam-policy-binding \
  my-app@PROJECT.iam.gserviceaccount.com \
  --role roles/iam.workloadIdentityUser \
  --member "serviceAccount:PROJECT.svc.id.goog[NAMESPACE/my-app]"
```

### GKE Ingress with Managed Certificate
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: "gce"
    networking.gke.io/managed-certificates: "my-cert"
spec:
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /*
        pathType: ImplementationSpecific
        backend:
          service:
            name: my-app
            port:
              number: 80
---
apiVersion: networking.gke.io/v1
kind: ManagedCertificate
metadata:
  name: my-cert
spec:
  domains:
  - api.example.com
```

### Container-Native Load Balancing (NEG)
```yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    cloud.google.com/neg: '{"ingress": true}'  # Enable NEGs
spec:
  type: ClusterIP  # Not NodePort
```

> See `references/research.md` for detailed examples and advanced patterns.

## Related Skills

| Skill | Use When |
|-------|----------|
| kubernetes-debugging | Troubleshooting pod issues |
| helm-charts | Packaging deployments as charts |
| github-actions | CI/CD pipeline setup |

## Resources

- [GKE Docs](https://cloud.google.com/kubernetes-engine/docs)
- [K8s API Reference](https://kubernetes.io/docs/reference/kubernetes-api/)
