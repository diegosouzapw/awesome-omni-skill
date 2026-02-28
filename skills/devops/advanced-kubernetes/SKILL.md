---
name: advanced-kubernetes
category: cloud-native
difficulty: advanced
estimated_time: 60 minutes
prerequisites:
- kubernetes
tags:
- operators
- crd
- kubebuilder
- controllers
- webhooks
description: Custom Resource Definitions (CRDs) extend Kubernetes API with custom
  object types. Operators are controllers that manage these custom resources using
  domain-specific logic.
---


# Advanced Kubernetes: Operators & CRDs

## Level 1: Quick Reference

### Core Concepts at a Glance

**Custom Resource Definitions (CRDs)** extend Kubernetes API with custom object types. **Operators** are controllers that manage these custom resources using domain-specific logic.

**CRD vs ConfigMap Comparison:**

| Aspect | CRD | ConfigMap |
|--------|-----|-----------|
| **API Integration** | Full Kubernetes API support (CRUD, watch, RBAC) | Simple key-value storage |
| **Validation** | OpenAPI v3 schema validation, admission webhooks | No built-in validation |
| **Versioning** | Multiple versions with conversion webhooks | Single version only |
| **Use Case** | Complex application state, declarative APIs | Configuration data, environment variables |
| **Controller Support** | Reconciliation loops, status tracking | Manual polling required |
| **Example** | Database instances, ML workflows, backup policies | App config files, feature flags |

### Operator Pattern Overview

```
┌─────────────────────────────────────────────────────────┐
│                    Kubernetes API Server                 │
│  (stores desired state in etcd)                         │
└────────────┬────────────────────────────┬───────────────┘
             │                            │
             │ Watch                      │ Update Status
             ↓                            ↑
    ┌────────────────┐         ┌─────────────────────┐
    │   Controller   │────────→│  External Resources │
    │  (Reconcile)   │  Manage │  (DBs, APIs, etc.)  │
    └────────────────┘         └─────────────────────┘
         ↑
         │ Compare
         │
    ┌────┴─────┐
    │ Desired  │
    │ vs Actual│
    └──────────┘
```

**Reconciliation Loop:**

1. **Watch** - Controller watches for changes to custom resources
2. **Compare** - Reconcile function compares desired vs actual state
3. **Act** - Controller takes actions to align actual state with desired
4. **Update Status** - Controller updates resource status with current state
5. **Requeue** - Schedule next reconciliation (periodic or event-driven)

### Controller Reconciliation Logic

```go
// Simplified reconciliation pattern
func (r *MyReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    // 1. Fetch the custom resource
    obj := &myapi.MyResource{}
    if err := r.Get(ctx, req.NamespacedName, obj); err != nil {
        return ctrl.Result{}, client.IgnoreNotFound(err)
    }

    // 2. Handle deletion (finalizers)
    if !obj.DeletionTimestamp.IsZero() {
        return r.handleDeletion(ctx, obj)
    }

    // 3. Reconcile external state
    if err := r.reconcileExternal(ctx, obj); err != nil {
        return ctrl.Result{}, err
    }

    // 4. Update status
    obj.Status.Ready = true
    if err := r.Status().Update(ctx, obj); err != nil {
        return ctrl.Result{}, err
    }

    return ctrl.Result{}, nil // Success, no requeue
}
```

### Essential Checklist

**Prerequisites:**

- [ ] Kubernetes cluster (v1.25+) - local (kind, minikube) or remote
- [ ] kubectl configured with admin access
- [ ] Go 1.21+ installed
- [ ] Docker/Podman for building operator images

**Development Tools:**

- [ ] `kubebuilder` (v3.12+) - scaffolding and code generation
- [ ] `operator-sdk` (optional) - alternative framework
- [ ] `controller-gen` - generates CRDs, RBACs, webhooks
- [ ] `kustomize` - manages Kubernetes manifests

**Testing Tools:**

- [ ] `envtest` - runs API server locally for unit tests
- [ ] `kind` - Kubernetes in Docker for integration tests
- [ ] `ginkgo` - BDD testing framework (optional)

**Key Files in Operator Project:**

```
my-operator/
├── api/v1/           # CRD definitions (Go structs)
├── config/
│   ├── crd/          # Generated CRD YAML
│   ├── rbac/         # Generated RBAC YAML
│   ├── manager/      # Operator deployment
│   └── webhook/      # Webhook configurations
├── controllers/      # Reconciliation logic
├── main.go           # Entrypoint (manager setup)
└── Dockerfile        # Container image build
```

**Quick Commands:**

```bash
# Initialize operator project
kubebuilder init --domain example.com --repo github.com/myorg/my-operator

# Create CRD + controller
kubebuilder create api --group apps --version v1 --kind MyApp

# Generate manifests
make manifests

# Run locally (connects to current kubeconfig cluster)
make install run

# Run tests
make test

# Build and deploy
make docker-build docker-push deploy IMG=myregistry/my-operator:v1.0.0
```

**Common Pitfalls:**

- ❌ Forgetting to update CRD when changing API structs → run `make manifests`
- ❌ Infinite reconciliation loops → use `ctrl.Result{RequeueAfter: time.Minute}`
- ❌ Not handling deletion properly → implement finalizers
- ❌ Blocking operations in reconcile → use background workers for long tasks
- ❌ Not setting owner references → orphaned resources on deletion

**When to Use Operators:**

- ✅ Managing complex stateful applications (databases, message queues)
- ✅ Automating operational tasks (backups, upgrades, scaling)
- ✅ Integrating with external systems (cloud APIs, SaaS platforms)
- ✅ Enforcing organizational policies (cost controls, security standards)
- ❌ Simple deployments (use Helm or plain manifests)
- ❌ One-time configuration changes (use Jobs or manual kubectl)

---

## Level 2: Implementation Guide

> **📚 Complete Examples**: See [REFERENCE.md](./REFERENCE.md) for full controller implementations, webhook code, test suites, and production-ready patterns.

### 2.1 Custom Resource Definitions (CRDs)

CRDs extend Kubernetes API with custom object types validated by OpenAPI v3 schemas.

**Key Components:**

- **Spec** - Desired state (user input)
- **Status** - Observed state (controller output, separate subresource)
- **Validation** - Markers like `+kubebuilder:validation:Minimum=1`
- **Versions** - Support multiple API versions with conversion webhooks

**Essential Kubebuilder Markers:**

```go
// +kubebuilder:validation:Minimum=1
// +kubebuilder:validation:Maximum=10
Size int32 `json:"size"`

// +kubebuilder:validation:Pattern=`^[a-z0-9.-]+/[a-z0-9.-]+:[a-z0-9.-]+$`
Image string `json:"image"`

// +optional
Port int32 `json:"port,omitempty"`
```

**Printcolumns for `kubectl get`:**

```go
// +kubebuilder:printcolumn:name="Ready",type=integer,JSONPath=`.status.readyReplicas`
// +kubebuilder:printcolumn:name="Phase",type=string,JSONPath=`.status.phase`
```

**Subresources:**

- `+kubebuilder:subresource:status` - Separate status endpoint
- `+kubebuilder:subresource:scale` - Enable `kubectl scale`

**Generate CRDs:** `make manifests` → outputs to `config/crd/bases/`

See [REFERENCE.md](./REFERENCE.md) for complete CRD definition, versioning, and conversion webhooks.

---

### 2.2 Operators and Controllers

**Reconciliation Loop:**

1. **Watch** - Controller watches for resource changes (via informers/caches)
2. **Compare** - Reconcile compares desired vs actual state
3. **Act** - Create/update/delete Kubernetes resources to match desired state
4. **Update Status** - Set status conditions (`Ready`, `Progressing`, `Degraded`)
5. **Requeue** - Schedule next reconciliation (event-driven or periodic)

**Controller Pattern:**

```go
func (r *Reconciler) Reconcile(ctx, req) (ctrl.Result, error) {
    // 1. Fetch custom resource
    obj := &MyResource{}
    if err := r.Get(ctx, req.NamespacedName, obj); err != nil {
        return ctrl.Result{}, client.IgnoreNotFound(err)
    }

    // 2. Handle deletion (finalizers)
    if !obj.DeletionTimestamp.IsZero() {
        return r.handleDeletion(ctx, obj)
    }

    // 3. Reconcile external state
    if err := r.reconcileDeployment(ctx, obj); err != nil {
        return ctrl.Result{RequeueAfter: 30*time.Second}, err
    }

    // 4. Update status
    obj.Status.Ready = true
    return ctrl.Result{}, r.Status().Update(ctx, obj)
}
```

**Key Functions:**

- `controllerutil.CreateOrUpdate()` - Idempotent create/update
- `controllerutil.SetControllerReference()` - Automatic garbage collection
- `controllerutil.AddFinalizer()` - Cleanup before deletion

**Error Handling:**

- **Transient errors** - Requeue with delay: `ctrl.Result{RequeueAfter: 30s}`
- **Permanent errors** - Set degraded condition, don't requeue
- **Unknown errors** - Return error for exponential backoff

See [REFERENCE.md](./REFERENCE.md) for complete controller implementation with finalizers, owner references, and error handling.

---

### 2.3 Admission Webhooks

Webhooks intercept API requests before persistence for validation/mutation.

**Types:**

- **Validating** - Accept/reject requests (JWT validation, cross-field checks)
- **Mutating** - Modify requests (inject sidecars, set defaults)

**Implementation:**

```go
// Validating webhook
func (r *MyApp) ValidateCreate() (admission.Warnings, error) {
    if r.Spec.Size < 1 || r.Spec.Size > 100 {
        return nil, fmt.Errorf("size must be 1-100")
    }
    return nil, nil
}

// Mutating webhook (Defaulter)
func (r *MyApp) Default() {
    if r.Spec.Port == 0 {
        r.Spec.Port = 8080
    }
}
```

**Setup:**

1. Implement `webhook.Validator` or `webhook.Defaulter` interface
2. Add kubebuilder marker: `// +kubebuilder:webhook:path=/validate-...,mutating=false,...`
3. `make manifests` generates webhook config
4. Deploy with cert-manager for TLS certificates

**Requirements:**

- TLS certificates (use cert-manager)
- Service routing webhook traffic to operator
- `failurePolicy: fail` (default) - reject on webhook errors

See [REFERENCE.md](./REFERENCE.md) for complete webhook examples, cert-manager setup, and validation patterns.

---

### 2.4 Leader Election & High Availability

Leader election ensures only one controller instance reconciles at a time (prevents race conditions).

**Configuration:**

```go
mgr, err := ctrl.NewManager(ctrl.GetConfigOrDie(), ctrl.Options{
    LeaderElection:          true,
    LeaderElectionID:        "myapp-controller.example.com",
    LeaderElectionNamespace: "myapp-system",
})
```

**How It Works:**

- Uses Kubernetes `Lease` resource for coordination
- One replica acquires lease, becomes leader
- Other replicas standby, ready to take over on leader failure
- Leader renews lease every 10s (default)

**Deployment:**

```yaml
spec:
  replicas: 3  # High availability
  containers:
  - args:
    - --leader-elect
```

See [REFERENCE.md](./REFERENCE.md) for RBAC requirements and lease configuration tuning.

---

### 2.5 Testing Operators

**Unit Testing with envtest:**

- Runs local API server (no kubelet, no containers)
- Fast tests (milliseconds per test)
- Full CRD validation

**Setup:**

```go
testEnv = &envtest.Environment{
    CRDDirectoryPaths: []string{filepath.Join("..", "config", "crd", "bases")},
}
cfg, _ := testEnv.Start()
k8sClient, _ = client.New(cfg, client.Options{Scheme: scheme.Scheme})
```

**Test Pattern:**

```go
It("Should create Deployment", func() {
    myApp := &MyApp{...}
    Expect(k8sClient.Create(ctx, myApp)).Should(Succeed())

    deployment := &Deployment{}
    Eventually(func() error {
        return k8sClient.Get(ctx, namespacedName, deployment)
    }, timeout, interval).Should(Succeed())

    Expect(*deployment.Spec.Replicas).To(Equal(int32(3)))
})
```

**Integration Testing with kind:**

```bash
kind create cluster
make docker-build docker-push deploy IMG=operator:test
kubectl wait --for=condition=available deployment/operator
kubectl apply -f test-cr.yaml
```

See [REFERENCE.md](./REFERENCE.md) for complete test suites, ginkgo patterns, and E2E test scripts.

---

### 2.6 Best Practices & Anti-Patterns

**✅ Best Practices:**

- **Idempotent reconciliation** - Same result on multiple calls
- **Use `CreateOrUpdate`** - Simplifies create/update logic
- **Set owner references** - Automatic garbage collection
- **Finalizers for cleanup** - External resources (cloud APIs, databases)
- **Status conditions** - `Ready`, `Progressing`, `Degraded` with detailed messages
- **Structured logging** - JSON format with consistent key-value pairs

**❌ Anti-Patterns:**

- **Blocking operations** - Don't make sync calls that block reconcile
- **Infinite loops** - Updating spec in reconcile triggers another reconcile
- **Hardcoded values** - Use env vars/ConfigMaps
- **Missing watches** - Ensure RBAC allows watching dependent resources
- **No health checks** - Implement `/healthz` and `/readyz` endpoints

**Requeue Strategies:**

```go
// Immediate requeue (rate-limited)
return ctrl.Result{Requeue: true}, nil

// Requeue after delay
return ctrl.Result{RequeueAfter: 30 * time.Second}, nil

// No requeue (wait for watch event)
return ctrl.Result{}, nil

// Error (exponential backoff)
return ctrl.Result{}, fmt.Errorf("transient error")
```

See [REFERENCE.md](./REFERENCE.md) for advanced patterns, multi-cluster operators, and OLM integration.

---

## Level 3: Deep Dive Resources


### Advanced Operator Patterns


**State Machine Operators**


- Model complex workflows as finite state machines

- Use status phases to track progression through states

- Implement state transition validations and guards


**Multi-Tenancy Operators**


- Namespace isolation strategies

- Shared vs dedicated operator deployments

- RBAC scoping for tenant-specific resources


**GitOps Integration**


- Reconcile against Git repository state

- Implement drift detection and auto-remediation

- Use annotations to track source commits


**External Secret Management**


- Integrate with Vault, AWS Secrets Manager, or Azure Key Vault

- Implement secret rotation without downtime

- Use external-secrets operator pattern


### Multi-Cluster Operators


**Architecture Patterns:**


1. **Hub-Spoke Model** - Central operator manages multiple clusters

2. **Federated Model** - Operators in each cluster coordinate via shared state

3. **Active-Active** - Operators in multiple clusters handle same resources


**Implementation Considerations:**


- Use cluster-api for cluster lifecycle management

- Implement cross-cluster service discovery (e.g., Submariner)

- Handle network partitions and split-brain scenarios

- Use consensus protocols for distributed state


**Tools:**


- **KubeFed** (deprecated) - Kubernetes Federation v2

- **OCM (Open Cluster Management)** - CNCF sandbox project

- **Argo CD ApplicationSet** - Multi-cluster GitOps

- **Crossplane** - Universal control plane for multi-cloud


### Operator Lifecycle Manager (OLM)


**What is OLM?**


- Package manager for Kubernetes operators

- Handles installation, upgrades, and dependency management

- Used by OpenShift and available as CNCF project


**OLM Components:**


- **Catalog** - Repository of operator metadata (CSV, CRD)

- **Subscription** - Declarative operator installation

- **InstallPlan** - Execution plan for operator installation

- **ClusterServiceVersion (CSV)** - Operator metadata and deployment info


**Creating an OLM Bundle:**


```bash

# Generate bundle manifests

operator-sdk generate bundle --version 1.0.0



# Validate bundle

operator-sdk bundle validate ./bundle



# Build and push bundle image

docker build -f bundle.Dockerfile -t myregistry/myapp-operator-bundle:v1.0.0 .

docker push myregistry/myapp-operator-bundle:v1.0.0



# Add to catalog

opm index add --bundles myregistry/myapp-operator-bundle:v1.0.0 \

  --tag myregistry/myapp-catalog:latest

```


**OLM Best Practices:**


- Define proper upgrade paths in CSV

- Test upgrade scenarios (skip versions, downgrades)

- Use semantic versioning

- Document breaking changes in release notes


### Advanced Testing Strategies


**Property-Based Testing:**


- Use tools like `gopter` for property-based tests

- Test invariants across state transitions

- Generate random valid/invalid inputs


**Chaos Testing:**


- Use Chaos Mesh or Litmus to inject failures

- Test operator resilience to node failures, network partitions

- Verify recovery from partial updates


**Performance Testing:**


- Benchmark reconciliation loop latency

- Test with 1000+ custom resources

- Measure memory/CPU usage under load

- Use profiling tools (pprof) for bottleneck analysis


### Production Readiness Checklist


**Observability:**


- [ ] Metrics exported via Prometheus endpoint

- [ ] Structured logging with levels (info, warn, error)

- [ ] Distributed tracing (OpenTelemetry)

- [ ] Custom metrics for business logic (e.g., backup success rate)


**Security:**


- [ ] RBAC follows least-privilege principle

- [ ] Secrets encrypted at rest and in transit

- [ ] Pod Security Standards enforced

- [ ] Network policies restrict traffic

- [ ] Image vulnerability scanning in CI/CD


**Reliability:**


- [ ] Leader election enabled for HA

- [ ] Graceful shutdown with finalizers

- [ ] Rate limiting to prevent API server overload

- [ ] Circuit breakers for external dependencies

- [ ] Backup/restore procedures documented


**Operational:**


- [ ] Runbooks for common failure scenarios

- [ ] SLO/SLI definitions (e.g., 99.9% reconciliation success)

- [ ] Alerting rules for critical conditions

- [ ] Upgrade/rollback procedures tested

- [ ] Capacity planning documented


### Resources and Further Learning


**Official Documentation:**


- [Kubebuilder Book](https://book.kubebuilder.io/)

- [Operator SDK Documentation](https://sdk.operatorframework.io/)

- [Kubernetes Controller Runtime](https://github.com/kubernetes-sigs/controller-runtime)


**Bundled Resources in This Directory:**


1. `templates/crd-definition.yaml` - Complete CRD with OpenAPI schema

2. `templates/operator-scaffold.go` - Controller with reconcile logic

3. `templates/webhook.go` - Validating and mutating webhooks

4. `templates/rbac.yaml` - RBAC manifests for operator deployment

5. `scripts/setup-operator-dev.sh` - Development environment setup

6. `resources/operator-patterns.md` - Common patterns and anti-patterns


**Community Resources:**


- [CNCF Operator White Paper](https://github.com/cncf/tag-app-delivery/blob/main/operator-wg/whitepaper/Operator-WhitePaper_v1-0.md)

- [Awesome Operators](https://github.com/operator-framework/awesome-operators)

- [Kubernetes Slack #kubebuilder](https://kubernetes.slack.com/)


**Example Production Operators:**


- [etcd-operator](https://github.com/coreos/etcd-operator)

- [prometheus-operator](https://github.com/prometheus-operator/prometheus-operator)

- [cert-manager](https://github.com/cert-manager/cert-manager)

- [strimzi-kafka-operator](https://github.com/strimzi/strimzi-kafka-operator)


### Next Steps


1. **Build a Simple Operator** - Start with a basic CRD and controller

2. **Add Validation** - Implement admission webhooks

3. **Test Thoroughly** - Write unit tests with envtest, integration tests with kind

4. **Observe in Production** - Deploy with metrics, logging, and tracing

5. **Iterate** - Add features based on operational experience


**Advanced Topics to Explore:**


- Custom admission plugins

- API aggregation and extension API servers

- Operator Hub and OLM

- Multi-cluster federation

- Operator performance optimization
