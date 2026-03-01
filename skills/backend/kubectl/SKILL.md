---
name: kubectl
description: Manage the local kubernetes cluster for development. Use when the user asks to check pods, restart deployments, view logs, apply manifests, create or delete resources, or perform any kubectl/helm operation.
argument-hint: <operation>
disable-model-invocation: false
allowed-tools: Bash(kubectl *), Bash(helm *), Read
---

## Setup

1. **Read cluster config**: Read `dev_env/.env` to get:
   - `DATASPOKE_DEV_KUBE_CLUSTER` — kube context (e.g., `docker-desktop`)
   - `DATASPOKE_DEV_KUBE_DATAHUB_NAMESPACE` — DataHub namespace (e.g., `datahub-01`)
   - `DATASPOKE_DEV_KUBE_DATASPOKE_NAMESPACE` — DataSpoke namespace (e.g., `dataspoke-01`)
   - `DATASPOKE_DEV_KUBE_DUMMY_DATA_NAMESPACE` — Example sources namespace (e.g., `dummy-data1`)

2. **Verify prerequisites**:
```bash
kubectl version --client
kubectl config current-context   # confirm correct context
kubectl get nodes                 # confirm cluster access
```

If tools are missing or cluster is unreachable, stop and inform the user.

---

## Execution Strategy for `$ARGUMENTS`

**Before acting**: identify the operation type from `$ARGUMENTS` — read, create, modify, or delete.

**Read operations** (safe, execute immediately):
```bash
kubectl get <type> -n <namespace>
kubectl describe <type> <name> -n <namespace>
kubectl logs <pod> -n <namespace> --tail=100
kubectl get events -n <namespace> --sort-by='.lastTimestamp'
helm status <release> -n <namespace>
helm history <release> -n <namespace>
helm get values <release> -n <namespace>
```

**Create/apply operations** (use dry-run first):
```bash
# Validate first
kubectl apply -f <file.yaml> --dry-run=server

# Then apply
kubectl apply -f <file.yaml> -n <namespace>
```

**Modify operations** (confirm intent, then execute):
```bash
kubectl rollout restart deployment/<name> -n <namespace>
helm upgrade <release> <chart> -n <namespace> -f <values.yaml>
```

**Delete operations** — always follow this safety workflow:
```bash
# 1. Confirm you're in the right context
kubectl config current-context

# 2. Describe before deleting
kubectl describe <type> <name> -n <namespace>

# 3. Delete
kubectl delete <type> <name> -n <namespace>

# 4. Verify
kubectl get <type> -n <namespace>
```

**Never delete namespaces.** For scale-to-zero or namespace-level destructive operations, confirm with the user first.

---

## Report Results

After every operation, summarize:
- What was executed
- What changed (before/after state if relevant)
- Any warnings, errors, or events triggered
- Suggested next steps if issues are found

For errors, consult [reference.md](reference.md) for common causes and resolutions.

---

See [reference.md](reference.md) for helm chart management, resource creation patterns, capacity planning, and error reference.
