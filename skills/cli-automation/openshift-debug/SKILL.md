---
name: openshift-debug
description: OpenShift (OCP) troubleshooting - Security Context Constraints (SCC), Routes, Projects, OperatorHub/OLM operator failures, DeploymentConfig, cluster operators, image streams, and oc CLI diagnostics.
metadata:
  emoji: "🔴"
  requires:
    bins: ["oc", "kubectl", "bash"]
---

# OpenShift Debug — OCP Troubleshooting Runbook

OpenShift-specific diagnostics covering the unique resources and constraints that differ from upstream Kubernetes. Uses the `oc` CLI and covers SCCs, Routes, OLM operators, cluster operators, and image streams.

Inspired by: Red Hat OpenShift documentation, OpenShift SRE runbooks, must-gather analysis, OpenShift 4.x operations guide.

## When to Activate

Activate when the user asks about:
- OpenShift SCC, Security Context Constraints, runAsUser
- OpenShift Route, edge/passthrough/reencrypt TLS
- OLM operator failure, OperatorHub, Subscription
- OpenShift cluster operator degraded
- DeploymentConfig vs Deployment in OpenShift
- oc commands, OpenShift-specific CLI
- OpenShift image stream, internal registry
- OpenShift Projects (namespaces)
- OpenShift BuildConfig, S2I (source-to-image)
- must-gather, oc adm commands
- OpenShift oauth, authentication, htpasswd
- OpenShift MachineConfig, MachineConfigPool

## Troubleshooting Runbook

### OpenShift vs Kubernetes Key Differences

| Concept | Kubernetes | OpenShift |
|---------|-----------|-----------|
| Namespace | `kubectl get namespace` | `oc get project` (Project = Namespace) |
| Ingress | Ingress resource | Route (OCP-native) |
| RBAC | Role/ClusterRole | Same + additional SCC layer |
| Operators | manual | OLM (Operator Lifecycle Manager) |
| Image pull | Any registry | Integrated image registry + image streams |
| Build | External CI | BuildConfig, S2I |
| Node config | manual | MachineConfig / MachineConfigPool |
| Auth | OIDC/webhook | OAuth proxy + HTPasswd/LDAP/GitHub |

---

## Step 1 — Cluster Operator Health

Cluster operators manage all OpenShift platform components. All must be `Available=True, Degraded=False`.

```bash
# All cluster operators status
oc get co

# Degraded operators (most important health signal)
oc get co | grep -E "False|True.*True"

# Detailed status for a degraded operator
oc describe co <operator-name>

# Get cluster version
oc get clusterversion

# Check upgrade progress
oc get clusterversion -o yaml | grep -A10 "conditions:"
```

---

## Failure Mode: SCC Violation (Pod Rejected)

**Security Context Constraints (SCCs)** are OpenShift's security admission layer — more powerful than Kubernetes PodSecurityAdmission. A pod that works in upstream k8s may be rejected in OpenShift because of SCC restrictions.

**Symptom:** Pod stuck in `ContainerCreating` or `Error`, events show SCC-related rejection

```bash
# Check SCC admission error
oc describe pod <pod-name> -n <project>
# Look for: "unable to validate against any security context constraint"

# List all SCCs (sorted by priority, highest first)
oc get scc
# Key SCCs: privileged > hostnetwork > anyuid > restricted (default)

# Check what SCC a running pod was admitted under
oc get pod <pod-name> -n <project> \
  -o jsonpath='{.metadata.annotations.openshift\.io/scc}'

# Simulate which SCC a ServiceAccount would get
oc adm policy who-can use scc anyuid
oc policy scc-subject-review -z <service-account> -n <project>

# What SCCs does a user/SA have access to?
oc adm policy who-can use scc restricted

# Grant SCC to ServiceAccount (most common fix)
oc adm policy add-scc-to-user anyuid -z <service-account> -n <project>
oc adm policy add-scc-to-user privileged -z <service-account> -n <project>

# Remove SCC from SA
oc adm policy remove-scc-from-user anyuid -z <service-account> -n <project>

# Common SCC requirements:
# - App needs to run as root (uid 0): grant anyuid
# - App needs host network: grant hostnetwork
# - App needs privileged container: grant privileged
# - App specifies arbitrary uid: check runAsUser field

# Check what uid ranges a project allows
oc get project <project> -o yaml | grep -A5 "annotations:"
# openshift.io/sa.scc.uid-range: 1000660000/10000
```

**Creating a custom SCC (preferred over using built-in privileged):**
```yaml
apiVersion: security.openshift.io/v1
kind: SecurityContextConstraints
metadata:
  name: my-app-scc
allowPrivilegedContainer: false
allowPrivilegeEscalation: false
runAsUser:
  type: RunAsAny   # or MustRunAs, MustRunAsNonRoot
seLinuxContext:
  type: MustRunAs
fsGroup:
  type: RunAsAny
users: []
groups: []
```

---

## Failure Mode: Route Not Working

Routes expose services externally in OpenShift (analogous to Ingress + LoadBalancer).

```bash
# List all routes
oc get routes -n <project>
oc get routes -A

# Describe a route
oc describe route <route-name> -n <project>

# Check router pods (HAProxy)
oc get pods -n openshift-ingress
oc get pods -n openshift-ingress-operator

# Router logs
oc logs -n openshift-ingress \
  $(oc get pod -n openshift-ingress -l ingresscontroller.operator.openshift.io/deployment-ingresscontroller=default -o name | head -1) \
  --tail=30

# Test route
curl -v https://<route-hostname>/

# Route TLS termination types:
# Edge:        TLS terminated at router, plain HTTP to pod
# Passthrough: TLS passes through to pod (pod handles TLS)
# Re-encrypt:  TLS terminated at router, re-encrypted to pod

# Create a route from CLI
oc expose svc/<service-name> --hostname=myapp.apps.mycluster.example.com

# Create edge TLS route
oc create route edge <route-name> \
  --service=<service-name> \
  --hostname=myapp.apps.mycluster.example.com \
  -n <project>

# Get ingress domain
oc get ingresses.config cluster -o jsonpath='{.spec.domain}'
```

---

## Failure Mode: OLM Operator Installation Failure

The Operator Lifecycle Manager (OLM) manages operator installation via Subscriptions, CSVs, and CatalogSources.

```bash
# Check OLM components
oc get pods -n openshift-operator-lifecycle-manager
oc get pods -n openshift-marketplace

# List operator subscriptions
oc get subscription -A

# Check subscription status (shows install plan)
oc describe subscription <name> -n <namespace>

# Check ClusterServiceVersion (CSV) status — this is the actual operator
oc get csv -n <namespace>
# Status: Succeeded = operator installed OK
# Status: Failed / InstallComponentFailed = problem

oc describe csv <csv-name> -n <namespace>

# Check InstallPlan
oc get installplan -n <namespace>
oc describe installplan <name> -n <namespace>

# Check CatalogSource (operator source)
oc get catalogsource -n openshift-marketplace

# CatalogSource pod logs (pulls index from registry)
oc logs -n openshift-marketplace \
  $(oc get pod -n openshift-marketplace -l olm.catalogSource=<cs-name> -o name)

# Approve a pending InstallPlan (if manual approval mode)
oc patch installplan <name> -n <namespace> \
  --type='json' -p='[{"op":"replace","path":"/spec/approved","value":true}]'

# Delete and recreate stuck subscription
oc delete subscription <name> -n <namespace>
oc delete csv <csv-name> -n <namespace>
# Recreate the Subscription YAML
```

**Common OLM error: CatalogSource not reachable**
```bash
# Check if catalog pod can reach registry
oc logs -n openshift-marketplace \
  $(oc get pod -n openshift-marketplace -l olm.catalogSource=redhat-operators -o name) | tail -20

# If behind proxy: check proxy config
oc get proxy cluster -o yaml
# spec.httpProxy, spec.httpsProxy, spec.noProxy must be set correctly
```

---

## Failure Mode: MachineConfig Issues

MachineConfig applies OS-level configuration to nodes (files, systemd units, kernel args).

```bash
# MachineConfigPool status
oc get mcp

# A pool stuck in "Updating" or "Degraded" blocks node updates
oc describe mcp <pool-name>

# Which nodes are being updated
oc get nodes | grep -E "SchedulingDisabled|NotReady"

# Check MachineConfigDaemon on a specific node
oc get pod -n openshift-machine-config-operator \
  -l k8s-app=machine-config-daemon \
  --field-selector="spec.nodeName=<node-name>"

oc logs -n openshift-machine-config-operator \
  <machine-config-daemon-pod> -c machine-config-daemon --tail=50

# Pause a MachineConfigPool (to prevent rolling updates during maintenance)
oc patch mcp <pool-name> --type='json' \
  -p='[{"op":"replace","path":"/spec/paused","value":true}]'

# Resume
oc patch mcp <pool-name> --type='json' \
  -p='[{"op":"replace","path":"/spec/paused","value":false}]'
```

---

## OpenShift Image Streams and Internal Registry

```bash
# List image streams
oc get imagestream -n <project>
oc describe imagestream <name> -n <project>

# Check internal registry pods
oc get pods -n openshift-image-registry

# Registry logs
oc logs -n openshift-image-registry \
  $(oc get pod -n openshift-image-registry -l docker-registry=default -o name | head -1) \
  --tail=30

# Registry storage
oc get configs.imageregistry.operator.openshift.io cluster -o yaml | grep -A5 "storage:"

# Pull an image from internal registry
docker pull \
  $(oc get route default-route -n openshift-image-registry -o jsonpath='{.spec.host}')/<project>/<image>:<tag>

# Login to internal registry
oc registry login
```

---

## OpenShift must-gather (Diagnostic Data Collection)

`must-gather` collects full diagnostic data — use when opening a Red Hat support case.

```bash
# Collect must-gather (dumps logs, configs, events)
oc adm must-gather

# Output goes to: must-gather.local.<timestamp>/

# Collect must-gather for specific operator
oc adm must-gather --image=<operator-must-gather-image>

# Inspect must-gather (offline)
kubectl --kubeconfig=must-gather.local.<ts>/registry-.../<cluster>/kubeconfig get pods -A
```

---

## Quick oc Command Reference

```bash
oc get co                        # cluster operators
oc get mcp                       # machine config pools
oc get nodes                     # nodes
oc get pods -A | grep -v Running # unhealthy pods
oc get routes -A                 # all routes
oc get csv -A                    # installed operators
oc get subscription -A           # operator subscriptions
oc get scc                       # security context constraints

# Project/namespace operations
oc new-project <name>
oc project <name>                # switch project
oc get project

# Resource investigation
oc describe node <node>
oc adm top nodes
oc adm top pods -A

# OAuth/auth
oc get oauth cluster -o yaml
oc get oauthaccesstoken
oc whoami
oc whoami --show-token

# Debugging
oc debug node/<node-name>        # debug shell on node (via privileged pod)
oc debug deployment/<name> -n <project>  # debug copy of pod
oc rsh <pod-name>                # remote shell into pod
oc rsync <pod>:/remote/path /local/path  # copy files
```

---

## References

- [OpenShift: SCC](https://docs.openshift.com/container-platform/latest/authentication/managing-security-context-constraints.html)
- [OpenShift: Routes](https://docs.openshift.com/container-platform/latest/networking/routes/route-configuration.html)
- [OpenShift: OLM](https://docs.openshift.com/container-platform/latest/operators/understanding/olm/olm-understanding-olm.html)
- [OpenShift: MachineConfig](https://docs.openshift.com/container-platform/latest/post_installation_configuration/machine-configuration-tasks.html)
- [must-gather](https://github.com/openshift/must-gather)
