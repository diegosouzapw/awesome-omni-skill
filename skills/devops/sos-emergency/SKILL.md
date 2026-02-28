---
name: sos-emergency
description: 'Ship Operating System: Emergency Kubernetes cluster recovery, Talos reset procedures, Synology Container Manager recovery, and graceful shutdown protocols. Trigger with /sos'
allowed-tools: ['read_file', 'run_in_terminal', 'grep_search', 'semantic_search', 'get_terminal_output', 'fetch_webpage', 'create_file']
---

# Ship Operating System (SOS) - Emergency Recovery

**"When everything fails, this skill has your back."**

I provide emergency recovery procedures for Sunkworks home lab infrastructure. This skill embraces the reality that systems fail—often spectacularly, often live on stream—and focuses on getting you back to operational status.

**Enterprise Warning**: These are emergency procedures. Some operations are destructive. Always verify you're targeting the correct cluster/node before executing recovery commands.

## Slash Command

### `/sos`
Runs emergency diagnostic workflow:
1. Identify cluster connectivity status
2. Check node health and etcd quorum
3. Assess recovery options based on failure mode
4. Recommend appropriate recovery procedure

**Usage**: Type `/sos` when systems are unresponsive or in critical failure state.

**Script Verification**: Before executing, verify the script integrity:
```bash
sha256sum .github/skills/sos-emergency/scripts/diagnose.sh
# Expected: 8691dce29cdc2a6ab7a02ff71f04fb35d1da30a44e8f0e794eef5c9afe7826c2
```

**Execute diagnostics**:
```bash
bash .github/skills/sos-emergency/scripts/diagnose.sh
```

## When I Activate
- `/sos` (slash command)
- "Everything is broken"
- "Cluster unreachable"
- "etcd quorum lost"
- "Talos node hung"
- "Synology Container Manager crashed"
- "Power failure protocol"
- "Emergency shutdown"
- "Abandon ship"

## Expected Failure Modes

### Kubernetes Cluster Failures
| Failure Mode | Symptoms | Est. Recovery Time |
|--------------|----------|-------------------|
| Single control plane down | API server unreachable, etcd errors | 15-30 min |
| etcd quorum loss | All control planes down, cluster frozen | 1-2 hours |
| Node kubelet crash | Pods stuck Terminating, node NotReady | 10-20 min |
| CNI failure | Pods running but no network connectivity | 30-60 min |

### Talos-Specific Failures
| Failure Mode | Symptoms | Est. Recovery Time |
|--------------|----------|-------------------|
| Extension deadlock | Node boots but services hang (Tailscale subnet router) | 30-45 min |
| Config drift | talosctl commands rejected, authentication failures | 15-30 min |
| Disk pressure | Node evicting pods, kubelet OOM | 20-40 min |

### Synology Infrastructure Failures
| Failure Mode | Symptoms | Est. Recovery Time |
|--------------|----------|-------------------|
| Container Manager crash | Docker daemon unresponsive, WebUI timeout | 10-20 min |
| Volume mount failures | Containers crash loop, permission errors | 15-30 min |
| DSM update breakage | Services fail post-update | 30-60 min |

## Core Recovery Procedures

### 1. Kubernetes Cluster Recovery

#### Check Cluster Connectivity
```bash
# Test API server reachability
kubectl cluster-info --request-timeout=5s

# If unreachable, check kubeconfig context
kubectl config current-context
kubectl config get-contexts
```

#### etcd Health Check
```bash
# For Talos clusters
talosctl -n <control-plane-ip> etcd status

# Check etcd member list
talosctl -n <control-plane-ip> etcd members

# Verify etcd alarm status (capacity/corruption)
talosctl -n <control-plane-ip> etcd alarm list
```

#### etcd Quorum Recovery (DESTRUCTIVE - Single Node)
**Use only when quorum is lost and cannot be recovered normally.**

```bash
# 1. Identify the node with most recent etcd data
talosctl -n <node1-ip> etcd status
talosctl -n <node2-ip> etcd status
talosctl -n <node3-ip> etcd status

# 2. Remove failed members from the surviving node
talosctl -n <surviving-node> etcd remove-member <failed-member-id>

# 3. If single node remains, force new cluster
talosctl -n <surviving-node> etcd forfeit-leadership
```

#### etcd Snapshot Restore
```bash
# 1. Take snapshot from healthy node (if available)
talosctl -n <healthy-node> etcd snapshot /tmp/etcd-backup.db

# 2. Copy snapshot and restore (cluster down scenario)
talosctl -n <node-ip> bootstrap --recover-from=/path/to/etcd-backup.db
```

### 2. Talos Reset Procedures

#### The Tailscale Subnet Router Deadlock
*From Moon and Back Episode: Node hangs during boot when Tailscale extension can't reach coordination server.*

**Symptoms**:
- Node boots but never becomes Ready
- `talosctl dmesg` shows Tailscale initialization timeout
- Network extension blocks kubelet startup

**Recovery**:
```bash
# Option 1: Wait for timeout (may take 10+ minutes)
# Tailscale extension has built-in timeout, node may recover

# Option 2: Force reset to maintenance mode
talosctl -n <node-ip> reset --graceful=false --reboot

# Option 3: Remove extension config temporarily
# Edit machine config to comment out Tailscale extension
talosctl -n <node-ip> edit machineconfig
# Then apply:
talosctl -n <node-ip> apply-config --insecure
```

#### Complete Node Reset
**DESTRUCTIVE: Wipes node and reinstalls from config.**

```bash
# Graceful reset (preserves etcd data if possible)
talosctl -n <node-ip> reset --graceful

# Hard reset (complete wipe)
talosctl -n <node-ip> reset --graceful=false --reboot

# Reset with system disk wipe
talosctl -n <node-ip> reset --system-labels-to-wipe STATE --system-labels-to-wipe EPHEMERAL
```

#### Recover Unresponsive Node
```bash
# 1. Check if maintenance mode is available
talosctl -n <node-ip> --endpoints <node-ip> version --insecure

# 2. If responsive in maintenance, reapply config
talosctl -n <node-ip> apply-config -f machineconfig.yaml --insecure

# 3. If completely unresponsive, physical intervention required
# Boot from Talos ISO and reinstall
```

### 3. Synology Emergency Access

#### Container Manager Crash Recovery
**When DSM Container Manager (Docker) becomes unresponsive:**

```bash
# SSH to Synology (enable SSH in Control Panel first)
ssh admin@<synology-ip>

# Check Docker daemon status
sudo synosystemctl status pkgctl-Docker

# Restart Docker service
sudo synosystemctl restart pkgctl-Docker

# If restart fails, stop and check logs
sudo synosystemctl stop pkgctl-Docker
sudo cat /var/log/synopkg.log | grep -i docker

# Force kill if hung
sudo killall -9 dockerd
sudo synosystemctl start pkgctl-Docker
```

#### Emergency SSH Access Setup
**Prepare before you need it:**

```bash
# Enable SSH in DSM Control Panel > Terminal & SNMP
# Set non-standard port for security

# Add SSH key for passwordless access
ssh-copy-id -i ~/.ssh/id_ed25519 admin@<synology-ip>

# Test access
ssh admin@<synology-ip> "sudo whoami"
```

#### Volume Mount Recovery
```bash
# SSH to Synology and check mounts
ssh admin@<synology-ip>

# List current mounts
mount | grep volume

# Check disk status
sudo cat /proc/mdstat

# If volume not mounted, check for errors
sudo dmesg | tail -50

# Remount volume (CAREFUL: verify volume number)
sudo mount /volume1
```

### 4. Abandon Ship Protocol

#### Graceful Shutdown Sequence
**For planned power outages or emergency situations:**

```bash
#!/bin/bash
# abandon-ship.sh - Graceful home lab shutdown

echo "=== ABANDON SHIP PROTOCOL INITIATED ==="
echo "Time: $(date)"

# Phase 1: Suspend Flux reconciliation (prevent drift during shutdown)
echo "Phase 1: Suspending Flux reconciliation..."
kubectl annotate fluxinstance flux -n flux-system \
  kustomize.toolkit.fluxcd.io/suspend=true

# Phase 2: Scale down workloads
echo "Phase 2: Scaling down deployments..."
kubectl get deployments -A -o name | \
  xargs -I{} kubectl scale {} --replicas=0

# Phase 3: Drain nodes gracefully
echo "Phase 3: Draining worker nodes..."
for node in $(kubectl get nodes -l node-role.kubernetes.io/control-plane!= -o name); do
  kubectl drain $node --ignore-daemonsets --delete-emptydir-data --force
done

# Phase 4: Drain control planes
echo "Phase 4: Draining control planes..."
for node in $(kubectl get nodes -l node-role.kubernetes.io/control-plane -o name); do
  kubectl drain $node --ignore-daemonsets --delete-emptydir-data --force
done

# Phase 5: Shutdown Talos nodes
echo "Phase 5: Shutting down Talos nodes..."
for node_ip in <worker-ips>; do
  talosctl -n $node_ip shutdown
done

# Wait for workers
sleep 30

for node_ip in <control-plane-ips>; do
  talosctl -n $node_ip shutdown
done

# Phase 6: Shutdown Synology (if integrated)
echo "Phase 6: Shutting down Synology NAS..."
ssh admin@<synology-ip> "sudo shutdown -h now"

echo "=== SHUTDOWN COMPLETE ==="
echo "Safe to remove power in 60 seconds"
```

#### UPS Integration Check
```bash
# Check UPS status via NUT (Network UPS Tools)
upsc <ups-name>@localhost

# Monitor battery level
upsc <ups-name>@localhost battery.charge

# Check if on battery power
upsc <ups-name>@localhost ups.status
# OL = Online (power good)
# OB = On Battery (power lost)
# LB = Low Battery (critical)
```

#### Emergency Power Script
```bash
#!/bin/bash
# emergency-power.sh - Triggered by UPS on low battery

BATTERY_LEVEL=$(upsc <ups-name>@localhost battery.charge 2>/dev/null)

if [ "$BATTERY_LEVEL" -lt 20 ]; then
  echo "CRITICAL: Battery at ${BATTERY_LEVEL}%"
  echo "Initiating emergency shutdown..."
  bash /path/to/abandon-ship.sh
fi
```

## Recovery Playbooks

### Playbook 1: "I Can't Reach My Cluster" (15-30 min)
1. Check network connectivity: `ping <control-plane-ip>`
2. Check VPN/Tailscale: `tailscale status`
3. Check kubeconfig: `kubectl config current-context`
4. Test direct connection: `kubectl --server=https://<ip>:6443 cluster-info`
5. SSH to node and check kubelet: `talosctl -n <ip> service kubelet`

### Playbook 2: "Node Won't Come Back Up" (30-60 min)
1. Check node status: `talosctl -n <ip> version --insecure`
2. Review boot logs: `talosctl -n <ip> dmesg | tail -100`
3. Check etcd membership: `talosctl -n <other-node> etcd members`
4. If extension hang: Apply config without problematic extension
5. If total failure: Reset and reinstall from known-good config

### Playbook 3: "Everything Is Down" (1-2 hours)
1. Identify last known good state (Terraform state, git history)
2. Check physical infrastructure (power, network switches)
3. Boot minimal cluster (single control plane if needed)
4. Restore etcd from most recent snapshot
5. Gradually bring up additional nodes
6. Verify Flux sync before re-enabling reconciliation

## MCP Server Integration

**After initial recovery, the Flux Operator MCP Server accelerates verification:**

### Post-Recovery Health Check
Once cluster connectivity is restored, use MCP tools instead of manual kubectl:
```
MCP Tool: get_flux_instance
→ Comprehensive Flux controller status and CRD health
→ Faster than iterative /flux-status diagnosis

MCP Tool: get_kubernetes_resources  
→ Query all pods, check for CrashLoopBackOff across namespaces
→ Identify remaining issues post-recovery
```

**Setup**: See flux-operator skill Step 10 for MCP server configuration.

## Integration Points

- **Flux Operator** → **Use MCP server** for comprehensive post-recovery verification
- **Prometheus Observer**: Check `/prometheus-status` for alert state after cluster recovery
- **Post-Mortem Author**: After recovery, use `/postmortem` to document what happened

## Read-Only Diagnostic Commands

Safe commands for assessment without modification:

```bash
# Cluster state
kubectl get nodes -o wide
kubectl get pods -A | grep -v Running
kubectl get events -A --sort-by=.lastTimestamp | tail -20

# Talos diagnostics
talosctl -n <node-ip> version
talosctl -n <node-ip> service
talosctl -n <node-ip> dmesg | tail -50

# Network diagnostics
talosctl -n <node-ip> netstat -tulpn

# etcd diagnostics
talosctl -n <node-ip> etcd status
```

## Sunkworks Episode Notes

*"The best debugging happens live, with an audience watching you fail."*

This skill is designed for live-stream troubleshooting where:
- Mistakes are educational
- Recovery time matters for audience engagement
- Documentation happens in real-time
- Iterative debugging is expected and embraced
