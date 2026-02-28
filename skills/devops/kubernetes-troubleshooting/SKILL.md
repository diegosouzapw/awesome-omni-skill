---
name: kubernetes-troubleshooting
description: Debug Kubernetes pods, services, networking, and scaling issues. Use this skill when troubleshooting K8s deployments, investigating pod failures, or diagnosing cluster problems.
alwaysApply: false
---

# Kubernetes Troubleshooting

You are a Kubernetes expert. Use these systematic debugging patterns when investigating K8s issues.

## Diagnostic Decision Tree

```
Pod not running?
├── Pending → Resource constraints or scheduling issues
│   ├── kubectl describe pod <name> → check Events
│   ├── Insufficient CPU/memory → scale cluster or reduce requests
│   ├── Node selector/affinity not matching → check node labels
│   └── PVC not bound → check storage class and PV availability
├── CrashLoopBackOff → Application crashing on startup
│   ├── kubectl logs <pod> → check application logs
│   ├── kubectl logs <pod> --previous → check last crash logs
│   ├── OOMKilled → increase memory limits
│   ├── Exit code 1 → application error (bad config, missing env)
│   └── Exit code 137 → killed by OOM or liveness probe
├── ImagePullBackOff → Can't pull container image
│   ├── Image name typo → verify image:tag exists
│   ├── Private registry → check imagePullSecrets
│   └── Rate limited → Docker Hub pull limit, use mirror
├── Running but not Ready → Readiness probe failing
│   ├── Check readiness probe config
│   ├── Application not listening on expected port
│   └── Dependency not available (database, cache)
└── Evicted → Node pressure
    ├── Disk pressure → clean up images, expand disk
    └── Memory pressure → reduce workload or add nodes
```

## Essential Debug Commands

### Pod Investigation
```bash
# Overview
kubectl get pods -A                          # All pods, all namespaces
kubectl get pods -o wide                     # With node and IP info
kubectl get pods --sort-by='.status.startTime' # Sorted by age

# Deep inspect
kubectl describe pod <name>                  # Events, conditions, volumes
kubectl logs <name>                          # Current logs
kubectl logs <name> --previous               # Previous crash logs
kubectl logs <name> -c <container>           # Specific container in multi-container pod
kubectl logs <name> --tail=100 -f            # Follow last 100 lines

# Interactive debug
kubectl exec -it <name> -- /bin/sh           # Shell into pod
kubectl exec -it <name> -- env               # Check environment
kubectl exec -it <name> -- cat /etc/resolv.conf  # Check DNS config

# Resource usage
kubectl top pods                             # CPU/memory per pod
kubectl top nodes                            # CPU/memory per node
```

### Service & Networking
```bash
# Check service endpoints
kubectl get endpoints <service>              # Are pods registered?
kubectl get svc <service> -o yaml            # Service config

# DNS resolution (from inside a pod)
kubectl exec -it <pod> -- nslookup <service>
kubectl exec -it <pod> -- wget -qO- http://<service>:<port>/health

# Test connectivity
kubectl run debug --image=nicolaka/netshoot -it --rm -- /bin/bash
# Then: curl, dig, nslookup, tcpdump, ping

# Ingress
kubectl get ingress -A
kubectl describe ingress <name>
```

### Cluster Health
```bash
kubectl get nodes                            # Node status
kubectl describe node <name>                 # Node conditions, allocatable resources
kubectl get events --sort-by='.lastTimestamp' # Recent cluster events
kubectl cluster-info                         # API server status
```

## Common Issues and Fixes

### CrashLoopBackOff
```bash
# 1. Check logs
kubectl logs <pod> --previous

# 2. Common causes:
# - Missing environment variable → check deployment env/configmap/secret
# - Database not reachable → check network policy, service DNS
# - Port conflict → check containerPort in deployment
# - Permissions → check SecurityContext, ServiceAccount

# 3. Debug with overridden command
kubectl run debug --image=<same-image> --command -- sleep 3600
kubectl exec -it debug -- /bin/sh
# Manually run the entrypoint to see errors
```

### OOMKilled (Exit Code 137)
```bash
# Check current limits
kubectl describe pod <name> | grep -A 5 "Limits"

# Fix: increase memory limit
# In deployment spec:
resources:
  requests:
    memory: "256Mi"
  limits:
    memory: "512Mi"  # Increase this

# Monitor actual usage first
kubectl top pod <name>
```

### Service Not Reachable
```bash
# Checklist:
# 1. Pod is Running and Ready?
kubectl get pods -l app=<name>

# 2. Service has endpoints?
kubectl get endpoints <service>
# If empty → labels don't match between Service and Pod

# 3. Port correct?
kubectl get svc <service> -o jsonpath='{.spec.ports[*]}'
# targetPort must match containerPort

# 4. NetworkPolicy blocking?
kubectl get networkpolicy -A
```

### Persistent Volume Issues
```bash
# PVC stuck in Pending
kubectl describe pvc <name>
# Common: no matching PV, storage class missing, capacity insufficient

# Check storage classes
kubectl get storageclass

# Check PVs
kubectl get pv
```

## Resource Right-Sizing

### Requests vs Limits
```yaml
resources:
  requests:          # Guaranteed minimum — scheduler uses this
    cpu: "100m"      # 0.1 CPU core
    memory: "128Mi"
  limits:            # Maximum allowed — killed if exceeded (memory), throttled (CPU)
    cpu: "500m"
    memory: "256Mi"
```

**Rules of thumb:**
- `requests` = average usage + 20% buffer
- `limits` = peak usage + 30% buffer
- Never set `limits` without `requests`
- CPU limits cause throttling — some teams only set requests for CPU
- Memory limits are hard — OOMKilled if exceeded

### HPA (Horizontal Pod Autoscaler)
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app
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
```

## Quick Reference

| Symptom | First Command | Likely Cause |
|---------|--------------|-------------|
| Pod pending | `kubectl describe pod` | Resource constraints |
| Pod crashing | `kubectl logs --previous` | App error or OOM |
| Service unreachable | `kubectl get endpoints` | Label mismatch or no ready pods |
| Slow response | `kubectl top pods` | CPU throttling or memory pressure |
| DNS not resolving | `kubectl exec -- nslookup` | CoreDNS issue or network policy |
| Storage error | `kubectl describe pvc` | No matching PV or storage class |
