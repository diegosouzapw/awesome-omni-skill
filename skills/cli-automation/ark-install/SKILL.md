---
name: Ark Install
description: Install Ark on a Kuberenetes Cluster
---

# Ark Install

Install Ark on a Kubernetes cluster using the published Ark CLI.

## When to use

- Installing Ark on a cluster

## Prerequisites

- Kubernetes cluster running - use `kubectl` to check for the presence of a cluster
- If a local Kubernetes cluster is not available, you MUST stop and tell the user that you cannot install Ark without a Kubernetes cluster

## Steps

1. **Install ark-cli**
   ```bash
   npm install -g @agents-at-scale/ark
   ```

2. **Install Ark**
   ```bash
   ark install --yes --wait-for-ready 5m
   ```

3. **Verify installation**
   ```bash
   ark status
   ```

## Optional: Install Phoenix telemetry

ONLY install if the user has asked specifically for Phoenix.

```bash
ark install marketplace/services/phoenix
```

## Optional: Install Langfuse telemetry

ONLY install if the user has asked specifically for Langfuse.

```bash
ark install marketplace/services/langfuse
```
