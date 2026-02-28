---
name: add-node
description: Add a new node to the Kubernetes cluster. Use when connecting new hardware, expanding cluster capacity, or setting up worker nodes.
---

# Add New Node to Cluster

Complete workflow to add a new node to the Kubernetes cluster via Tailscale.

## Steps Overview

1. Discover the node on Tailscale
2. Add to Ansible inventory
3. Run bootstrap playbook
4. Run preflight checks
5. Add to cluster

## Instructions

### Step 1: Discover the Node

Check if the node is visible on Tailscale:

```bash
tailscale status | grep -i <node_name>
```

If not found, the user needs to install Tailscale on the target:
```bash
# On target node:
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

### Step 2: Get Node Details

Required information:
- **Node name**: Hostname (should match Tailscale name)
- **Role**: worker (default) or control-plane
- **Reserved CPU**: CPU cores for local use (default: "2")
- **Reserved Memory**: Memory to reserve (default: "4Gi")
- **GPU**: Has GPU? (default: false)
- **Labels**: Custom labels (optional)

### Step 3: Update Inventory

Add to `ansible/inventory/hosts.yml`:

```yaml
workers:
  hosts:
    <node_name>:
      ansible_host: <tailscale_ip>
      tailscale_ip: <tailscale_ip>
      reserved_cpu: "2"
      reserved_memory: "4Gi"
      node_labels:
        node-role: worker
        workstation: "true"

bootstrap:
  hosts:
    <node_name>: {}
```

### Step 4: Setup SSH Access

Test connectivity:
```bash
ssh -o BatchMode=yes -o ConnectTimeout=5 <tailscale_ip> echo "SSH OK" 2>/dev/null
```

If fails:
```bash
ssh-copy-id <user>@<tailscale_ip>
```

### Step 5: Run Bootstrap Playbook

```bash
cd /home/al/git/kubani
ansible-playbook -i ansible/inventory/hosts.yml ansible/playbooks/bootstrap_node.yml --limit <node_name>
```

### Step 6: Run Preflight Checks

```bash
ansible-playbook -i ansible/inventory/hosts.yml ansible/playbooks/preflight_checks.yml --limit <node_name>
```

### Step 7: Add Node to Cluster

Include control plane node to fetch join token:
```bash
ansible-playbook -i ansible/inventory/hosts.yml ansible/playbooks/add_node.yml --limit "<node_name>,sparky"
```

### Step 8: Verify

```bash
KUBECONFIG=/home/al/.kube/config kubectl get nodes
```

## Troubleshooting

- **SSH failed**: Run `ssh-copy-id <user>@<ip>` manually
- **Bootstrap failed**: Usually package or network issue
- **Preflight failed**: Missing Tailscale auth or network issues
- **Add node failed**: Check control plane health
