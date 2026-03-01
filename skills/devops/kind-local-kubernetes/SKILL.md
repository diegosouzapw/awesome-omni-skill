---
name: Kind Local Kubernetes
description: This skill should be used when the user asks to "setup Kind", "local Kubernetes", "Kind cluster", "multi-node cluster", "Kubernetes development", "k8s local environment", or works with local Kubernetes clusters using Kind.
version: 0.2.0
---

# Kind Local Kubernetes

Kind (Kubernetes IN Docker) runs local Kubernetes clusters using Docker container nodes.

## Quick Start

```bash
# Install (macOS)
brew install kind

# Create cluster
kind create cluster --name dev-cluster --config=kind-config.yaml

# Load local image
kind load docker-image myapp:latest --name dev-cluster

# Delete cluster
kind delete cluster --name dev-cluster
```

## Configuration

**Basic multi-node config (`kind-config.yaml`):**
```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
    extraPortMappings:
      - containerPort: 30000
        hostPort: 30000
  - role: worker
```

## Go Integration (Best Practice)

For Go projects, use `client-go` and Kind Go library instead of kubectl.

### Create Kind Client

```go
import (
    "k8s.io/client-go/kubernetes"
    "k8s.io/client-go/tools/clientcmd"
    "sigs.k8s.io/kind/pkg/cluster"
)

func NewKindClient(clusterName string) (*kubernetes.Clientset, error) {
    provider := cluster.NewProvider()
    
    // Get kubeconfig directly from Kind (not ~/.kube/config)
    kubeconfig, err := provider.KubeConfig(clusterName, false)
    if err != nil {
        return nil, err
    }
    
    config, err := clientcmd.RESTConfigFromKubeConfig([]byte(kubeconfig))
    if err != nil {
        return nil, err
    }
    
    return kubernetes.NewForConfig(config)
}
```

### Deploy Resources

```go
import (
    "context"
    "time"
    
    apierrors "k8s.io/apimachinery/pkg/api/errors"
    metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
    "k8s.io/apimachinery/pkg/util/wait"
)

// Create resource
_, err := client.CoreV1().Namespaces().Create(ctx, ns, metav1.CreateOptions{})

// Wait for deployment ready
wait.PollUntilContextTimeout(ctx, 2*time.Second, timeout, true,
    func(ctx context.Context) (bool, error) {
        dep, err := client.AppsV1().Deployments(ns).Get(ctx, name, metav1.GetOptions{})
        if err != nil {
            return false, err
        }
        return dep.Status.ReadyReplicas == *dep.Spec.Replicas, nil
    })

// Cleanup
client.CoreV1().Namespaces().Delete(ctx, ns, metav1.DeleteOptions{})
```

### E2E Test Pattern

```go
func TestWithKind(t *testing.T) {
    provider := cluster.NewProvider()
    
    // Create cluster
    err := provider.Create("test", cluster.CreateWithConfigFile("kind-config.yaml"))
    require.NoError(t, err)
    defer provider.Delete("test", "")
    
    // Get client
    client, err := NewKindClient("test")
    require.NoError(t, err)
    
    // Run tests with client-go...
}
```

## Best Practices

1. **Use client-go instead of kubectl** - Type-safe, testable K8s operations
2. **Get kubeconfig from Kind directly** - `provider.KubeConfig()` avoids accidental production access
3. **Use wait.PollUntilContextTimeout** - Proper polling for resource readiness
4. **Define manifests in Go** - Type-safe resource definitions
5. **Name your clusters** - Avoid conflicts with default cluster
6. **Clean up** - Delete unused clusters to save resources

## Go Dependencies

```go
import (
    "k8s.io/client-go/kubernetes"
    "k8s.io/client-go/tools/clientcmd"
    appsv1 "k8s.io/api/apps/v1"
    corev1 "k8s.io/api/core/v1"
    metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
    apierrors "k8s.io/apimachinery/pkg/api/errors"
    "k8s.io/apimachinery/pkg/util/wait"
    "sigs.k8s.io/kind/pkg/cluster"
)
