---
name: volcano
description: "Volcano batch scheduling for Kubernetes — gang scheduling, VolcanoJobs, queue management, GPU scheduling, and Kubeflow integration. Use when scheduling distributed training or batch workloads. NOT for simple single-pod jobs. See also: kueue."
---

# Volcano

CNCF incubating batch scheduling system for AI/ML, big data, and HPC on Kubernetes. Replaces the default scheduler with gang scheduling, queue-based resource management, and framework-native integrations.

**Docs:** https://volcano.sh/en/docs/
**GitHub:** https://github.com/volcano-sh/volcano
**Version:** v1.14.1 | **Requires:** Kubernetes ≥ 1.12

## Core Concepts

| Object | API Group | Purpose |
|---|---|---|
| **VolcanoJob (vcjob)** | `batch.volcano.sh/v1alpha1` | Job with tasks, gang scheduling, lifecycle policies |
| **Queue** | `scheduling.volcano.sh/v1beta1` | Resource quotas, weights, priority, multi-tenancy |
| **PodGroup** | `scheduling.volcano.sh/v1beta1` | Group of pods scheduled atomically (auto-created by vcjob) |
| **Command** | `bus.volcano.sh/v1alpha1` | Control commands for jobs (abort, restart) |

**Flow:** VolcanoJob → PodGroup + Pods → Queue → Volcano scheduler (gang check + plugins) → node binding.

## Installation

```bash
# Helm (recommended)
helm repo add volcano-sh https://volcano-sh.github.io/helm-charts
helm repo update
helm install volcano volcano-sh/volcano -n volcano-system --create-namespace

# kubectl
kubectl apply -f https://raw.githubusercontent.com/volcano-sh/volcano/release-1.14/installer/volcano-development.yaml
```

**Components deployed:** volcano-scheduler, volcano-controllers, volcano-admission (webhook).

Verify: `kubectl get deploy -n volcano-system`

## VolcanoJob (vcjob)

The primary workload CRD. Supports multiple task groups with independent replicas, images, and policies.

```yaml
apiVersion: batch.volcano.sh/v1alpha1
kind: Job
metadata:
  name: distributed-training
spec:
  minAvailable: 4        # Gang scheduling: all 4 pods must be schedulable
  schedulerName: volcano
  queue: training-queue
  priorityClassName: high-priority
  maxRetry: 3
  plugins:
    ssh: []               # Auto-configures SSH between pods
    svc: []               # Creates headless service for DNS discovery
    env: []               # Injects VK_TASK_INDEX, VK_TASK_NUM env vars
  policies:
    - event: PodEvicted
      action: RestartJob
    - event: TaskCompleted
      action: CompleteJob
  tasks:
    - name: master
      replicas: 1
      template:
        spec:
          containers:
            - name: trainer
              image: training:latest
              command: ["torchrun", "--nproc_per_node=1", "--nnodes=4",
                "--node_rank=$(VK_TASK_INDEX)", "train.py"]
              resources:
                requests:
                  nvidia.com/gpu: "1"
                limits:
                  nvidia.com/gpu: "1"
          restartPolicy: Never
    - name: worker
      replicas: 3
      policies:
        - event: TaskCompleted
          action: CompleteJob
      template:
        spec:
          containers:
            - name: trainer
              image: training:latest
              resources:
                requests:
                  nvidia.com/gpu: "1"
                limits:
                  nvidia.com/gpu: "1"
          restartPolicy: Never
```

### Key Fields

- **`minAvailable`** — Minimum pods schedulable simultaneously (gang scheduling). Set to total replicas for strict gang.
- **`schedulerName: volcano`** — Routes to Volcano scheduler instead of default.
- **`queue`** — Target queue (defaults to `default`).
- **`plugins`** — `ssh` (passwordless SSH), `svc` (headless service + DNS), `env` (task index/count injection).
- **`policies`** — Lifecycle actions: `RestartJob`, `CompleteJob`, `AbortJob`, `TerminateJob` triggered by events (`PodEvicted`, `PodFailed`, `TaskCompleted`, `JobUnknown`).
- **`maxRetry`** — Max job restart attempts.

### Job Lifecycle States

`Pending` → `Running` (≥ minAvailable pods running) → `Completing` → `Completed`
Failure path: → `Restarting` → `Running` (up to maxRetry) → `Failed`
External: → `Aborting` → `Aborted`

## Queue Configuration

Queues control multi-tenant resource allocation. Two plugin modes:

### Proportion Plugin (weight-based, auto-adjusts)

```yaml
apiVersion: scheduling.volcano.sh/v1beta1
kind: Queue
metadata:
  name: team-a
spec:
  weight: 3               # Gets 3/(3+1) = 75% of cluster resources
  reclaimable: true        # Allow other queues to reclaim excess
  capability:              # Hard upper limit
    cpu: "64"
    memory: 256Gi
    nvidia.com/gpu: "8"
```

### Capacity Plugin (explicit quotas)

```yaml
apiVersion: scheduling.volcano.sh/v1beta1
kind: Queue
metadata:
  name: team-b
spec:
  deserved:                # Expected allocation (reclaimable above this)
    cpu: "16"
    memory: 64Gi
  guarantee:               # Reserved minimum (exclusive to this queue)
    resource:
      cpu: "8"
      memory: 32Gi
  capability:              # Hard ceiling
    cpu: "32"
    memory: 128Gi
  priority: 100
  reclaimable: true
```

**Rule:** `guarantee ≤ deserved ≤ capability`

- **proportion** plugin: auto-calculates deserved from weights. Best with autoscaling clusters.
- **capacity** plugin: explicit deserved values. More predictable. Use one, not both.

## Scheduler Configuration

Configure via `volcano-scheduler-configmap`. Actions execute in order; plugins provide algorithms.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: volcano-scheduler-configmap
  namespace: volcano-system
data:
  volcano-scheduler.conf: |
    actions: "enqueue, allocate, preempt, reclaim, backfill"
    tiers:
    - plugins:
      - name: priority
      - name: gang
        enablePreemptable: true
      - name: conformance
    - plugins:
      - name: drf
      - name: predicates
      - name: proportion
      - name: nodeorder
      - name: binpack
        arguments:
          binpack.weight: 10
          binpack.cpu: 5
          binpack.memory: 1
          binpack.resources: nvidia.com/gpu
          binpack.resources.nvidia.com/gpu: 10
```

### Actions

| Action | Purpose |
|---|---|
| `enqueue` | Filter jobs into scheduling queue based on quota |
| `allocate` | Assign pods to nodes using plugin algorithms |
| `preempt` | Preempt lower-priority jobs within the same queue |
| `reclaim` | Reclaim resources between queues when over-deserved |
| `backfill` | Fill idle resources with pending small jobs |

### Key Plugins

| Plugin | Purpose |
|---|---|
| `gang` | Enforce minAvailable — all-or-nothing scheduling |
| `priority` | Order by PriorityClass |
| `drf` | Dominant Resource Fairness — fair multi-resource allocation |
| `binpack` | Pack pods tightly to maximize utilization |
| `proportion` | Weight-based queue resource division |
| `capacity` | Explicit queue quota management |
| `predicates` | Node filtering (affinity, taints, resources) |
| `nodeorder` | Node scoring for placement optimization |
| `conformance` | Protect kube-system pods from preemption |

## Using Volcano with Kubeflow Training Operator

Set `schedulerName: volcano` on PyTorchJob/MPIJob/TFJob pod templates:

```yaml
apiVersion: kubeflow.org/v1
kind: PyTorchJob
metadata:
  name: ddp-training
  annotations:
    scheduling.volcano.sh/queue-name: training-queue
spec:
  schedulingPolicy:
    queue: training-queue
    minAvailable: 4
    priorityClass: high-priority
  pytorchReplicaSpecs:
    Master:
      replicas: 1
      template:
        spec:
          schedulerName: volcano
          containers:
            - name: pytorch
              image: training:latest
              resources:
                requests:
                  nvidia.com/gpu: "1"
    Worker:
      replicas: 3
      template:
        spec:
          schedulerName: volcano
          containers:
            - name: pytorch
              image: training:latest
              resources:
                requests:
                  nvidia.com/gpu: "1"
```

## Key kubectl Commands

```bash
# List Volcano objects
kubectl get vcjob,queue,podgroup -A

# Queue status and usage
kubectl describe queue <name>

# Job details
kubectl describe vcjob -n <ns> <name>

# PodGroup for a job
kubectl get podgroup -n <ns> -l volcano.sh/job-name=<name>

# Scheduler logs
kubectl logs -n volcano-system deploy/volcano-scheduler --tail=200

# Controller logs
kubectl logs -n volcano-system deploy/volcano-controllers --tail=200
```

## Volcano vs Kueue

| Aspect | Volcano | Kueue |
|---|---|---|
| **Approach** | Replaces scheduler (custom binary) | Admission controller (works with default scheduler) |
| **Gang scheduling** | Native, first-class via `minAvailable` + gang plugin | Via pod groups, less mature |
| **Job CRD** | Own `VolcanoJob` with tasks, plugins, lifecycle | No own job type — wraps existing K8s Jobs/JobSets |
| **Queue model** | `Queue` with capacity/proportion/hierarchical | `ClusterQueue` + `LocalQueue` + `ResourceFlavor` |
| **Fair sharing** | DRF plugin, weight-based proportion | DRF + usage-history-based admission |
| **Preemption** | Within-queue (preempt) + cross-queue (reclaim) | Configurable within/across ClusterQueues and cohorts |
| **GPU features** | MIG, vGPU sharing, binpacking built-in | Relies on ResourceFlavors for GPU types |
| **Maturity** | CNCF incubating, 5+ years, widely adopted | K8s SIG, newer, growing adoption |
| **Best for** | Gang scheduling–heavy, MPI, custom scheduler needs | Quota management, multi-tenant admission, K8s-native |

**Use Volcano when:** Gang scheduling is critical (MPI, multi-node DDP), need built-in GPU sharing, or want a full scheduler replacement with rich plugins.
**Use Kueue when:** You want admission-based quota without replacing the scheduler, need ResourceFlavors for heterogeneous hardware, or prefer the SIG-supported K8s-native approach.

## Volcano and LeaderWorkerSet

Volcano and [LeaderWorkerSet (LWS)](../leaderworkerset/) are **complementary, not competing**:

- **LWS** defines the workload primitive: leader + N workers managed as a cohesive group with all-or-nothing restarts, HPA scaling, and rolling updates. It is the standard K8s primitive for multi-node inference (vLLM, SGLang, NIM) and long-running training.
- **Volcano** provides the scheduling layer: gang scheduling, queue-based resource quotas, fair-share allocation, and preemption across jobs and tenants.

They can be used together — LWS manages the pod-group lifecycle while Volcano schedules it into a queue. The LWS `schedulerName: volcano` field routes its pods through Volcano's gang and capacity plugins. If you only need quota management without replacing the scheduler, use the [Kueue LWS integration](https://kueue.sigs.k8s.io/docs/tasks/run/leaderworkerset/) instead.

## References

- [`advanced-scheduling.md`](references/advanced-scheduling.md) — DRF fair-share, elastic jobs, SLA/TDM plugins, and GPU binpack tuning
- [`kubernetes-integration.md`](references/kubernetes-integration.md) — Kubeflow, Spark, Argo Workflows, RBAC, and monitoring integration
- [`operations.md`](references/operations.md) — Helm deployment, scheduler tuning, hierarchical queues, and GPU scheduling
- [`training-patterns.md`](references/training-patterns.md) — PyTorch DDP/FSDP, MPI, and Horovod distributed training patterns
- [`troubleshooting.md`](references/troubleshooting.md) — Common scheduling, networking, and job lifecycle issues

## Cross-References

- [kueue](../kueue/) — Alternative K8s-native job queueing (admission-based); compare Kueue LocalQueues with Volcano Queues
- [leaderworkerset](../leaderworkerset/) — Complementary pod-group primitive for multi-node inference and training; LWS pods can be scheduled through Volcano queues
- [nvidia-nim](../nvidia-nim/) — NIM inference microservices; use Volcano queue management when running NIM alongside training jobs in multi-tenant clusters
- [sglang](../sglang/) — SGLang inference serving; use Volcano to queue and gang-schedule batch inference SGLang jobs alongside training workloads
- [pytorch](../pytorch/) — PyTorch training fundamentals
- [fsdp](../fsdp/) — FSDP distributed training patterns
- [deepspeed](../deepspeed/) — DeepSpeed ZeRO integration
- [gpu-operator](../gpu-operator/) — NVIDIA GPU Operator for driver/MIG management
- [nccl](../nccl/) — NCCL tuning for multi-node GPU communication; see for IB/RoCE env vars and transport troubleshooting
- [kubeflow-trainer](../kubeflow-trainer/) — Volcano scheduler integration for training jobs
- [aws-efa](../aws-efa/) — EFA networking for Volcano-scheduled multi-node jobs
- [prometheus-grafana](../prometheus-grafana/) — Monitor Volcano queue and job metrics
- [minio](../minio/) — Checkpoint storage for Volcano-scheduled training jobs
