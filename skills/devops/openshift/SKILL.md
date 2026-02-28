---
name: openshift
description: OpenShift enterprise Kubernetes platform. Use for enterprise K8s.
---

# OpenShift

Red Hat OpenShift is an enterprise-ready Kubernetes container platform with full-stack automated operations. In 2025, **OpenShift Virtualization** (running VMs side-by-side with containers) is a key feature.

## When to Use

- **Hybrid Cloud**: Consistent experience across On-Prem and Cloud.
- **Enterprise Requirements**: Built-in strict security (SCC), registry, monitoring, and CI/CD.
- **VM Migration**: Lift-and-shift VMs into K8s using OpenShift Virtualization (KubeVirt).

## Quick Start (OC CLI)

```bash
# Login
oc login -u developer -p developer https://api.crc.testing:6443

# Create Project (Namespace)
oc new-project my-app

# Deploy from Source (Source-to-Image)
oc new-app nodejs~https://github.com/sclorg/nodejs-ex.git
```

## Core Concepts

### Source-to-Image (S2I)

Build container images directly from source code without writing a Dockerfile. OpenShift detects the language (Node/Java/Python) and builds it.

### Routes

OpenShift's native ingress controller. Used long before K8s Ingress/Gateway API.

### Operators

First-class citizens. Everything in OpenShift is managed by an Operator.

## Best Practices (2025)

**Do**:

- **Use OpenShift GitOps**: ArgoCD is fully integrated.
- **Use `oc`**: It is a superset of `kubectl`. You rarely need `kubectl` on OpenShift.
- **Leverage Virtualization**: Run legacy Windows/Linux VMs as Pods to decommission old VMWare clusters.

**Don't**:

- **Don't run as root**: OpenShift forbids this by default. Don't disable SCCs (Security Context Constraints) just to make a bad image work. Fix the image.

## References

- [OpenShift Documentation](https://docs.openshift.com/)
