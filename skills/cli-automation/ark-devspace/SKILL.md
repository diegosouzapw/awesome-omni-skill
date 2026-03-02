---
name: Ark DevSpace
description: Run Ark from cloned source using devspace
---

# Ark DevSpace

Run Ark from a cloned repository using devspace for development/PR testing.

## CRITICAL: Failure Behavior

**If devspace fails, you MUST STOP and report the error. Do NOT try alternative approaches like ark-cli or npm install. DevSpace failures must be diagnosed and fixed.**

## When to use

- Testing Ark pull requests
- Running Ark from source code

## Prerequisites

- Ark repo cloned (e.g., `/workspace/ark` or `/workspace/agents-at-scale-ark`)
- Kind cluster running (use kind-setup skill if needed)
- Verify cluster: `kubectl cluster-info`

## Steps

1. **Navigate to ark repo**
   ```bash
   cd /workspace/ark  # or /workspace/agents-at-scale-ark
   ```

2. **Deploy with devspace (non-interactive)**
   ```bash
   devspace deploy
   ```

   This deploys all components and exits. Check the exit code - if non-zero, STOP and troubleshoot.

3. **Verify deployment succeeded**
   ```bash
   kubectl get pods -n ark-system
   kubectl get pods -n cert-manager
   ```

4. **Wait for pods to be ready**
   ```bash
   kubectl wait --for=condition=Ready pods --all -n ark-system --timeout=300s
   ```

5. **Check services**
   ```bash
   kubectl get svc -n ark-system
   ```

## Troubleshooting

### Check devspace logs
```bash
ls -la .devspace/logs/
cat .devspace/logs/default.log
```

### Common failures

1. **"namespace not found"** - Set namespace first
   ```bash
   devspace use namespace ark-system
   ```

2. **"image pull error"** - Build images locally
   ```bash
   devspace build
   ```

3. **Cert-manager not ready** - Wait for it
   ```bash
   kubectl wait --for=condition=Available deployment --all -n cert-manager --timeout=120s
   ```

4. **Cluster connectivity** - Re-export kubeconfig
   ```bash
   kind export kubeconfig --name ark-cluster --internal
   ```

### If devspace exits with error

1. Check the exit code and error message
2. Check `.devspace/logs/` for detailed logs
3. Check pod status: `kubectl get pods -A`
4. Check events: `kubectl get events -A --sort-by='.lastTimestamp'`
5. **STOP and report the specific error - do not try workarounds**

## Cleanup

```bash
devspace purge
kubectl delete ns ark-system --ignore-not-found
```
