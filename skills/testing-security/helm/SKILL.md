---
name: helm
description: "Helm chart authoring — library charts, values.schema.json, subcharts, _helpers.tpl, helm-unittest. Use when building complex Helm charts. NOT for basic helm install/upgrade."
---

# Helm — Advanced Patterns

Beyond basics — patterns that trip up experienced engineers and aren't well-captured in training data.

**Docs:** https://helm.sh/docs/ | **Sprig:** https://masterminds.github.io/sprig/

## Library Charts

Shared template logic across multiple charts. Not deployable on their own.

```yaml
# Chart.yaml
apiVersion: v2
name: mylib
type: library   # Not "application"
version: 0.1.0
```

### Merge Utility Pattern

The canonical way to let consumers override library-defined resources:

```yaml
# mylib/templates/_util.tpl
{{- define "mylib.util.merge" -}}
{{- $top := first . -}}
{{- $overrides := fromYaml (include (index . 1) $top) | default (dict ) -}}
{{- $base := fromYaml (include (index . 2) $top) | default (dict ) -}}
{{- toYaml (merge $overrides $base) -}}
{{- end -}}
```

Consumer chart uses it to override specific fields while inheriting defaults:

```yaml
# mychart/templates/deployment.yaml
{{- include "mylib.util.merge" (list . "mychart.deployment.overrides" "mylib.deployment") }}

# mychart/templates/_overrides.tpl
{{- define "mychart.deployment.overrides" -}}
spec:
  replicas: {{ .Values.replicaCount }}
{{- end -}}
```

### Shared Label/Annotation Templates

```yaml
# mylib/templates/_labels.tpl
{{- define "mylib.labels" -}}
app.kubernetes.io/name: {{ .Chart.Name }}
app.kubernetes.io/instance: {{ .Release.Name }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
helm.sh/chart: {{ printf "%s-%s" .Chart.Name .Chart.Version | replace "+" "_" | trunc 63 }}
{{- end -}}

{{- define "mylib.selectorLabels" -}}
app.kubernetes.io/name: {{ .Chart.Name }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end -}}
```

## Values Schema Validation

`values.schema.json` validates values at install/upgrade/lint/template time. Catches config errors before they hit the cluster.

```json
{
  "$schema": "https://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["image", "service"],
  "properties": {
    "replicaCount": {
      "type": "integer",
      "minimum": 1,
      "maximum": 50
    },
    "image": {
      "type": "object",
      "required": ["repository"],
      "properties": {
        "repository": { "type": "string", "minLength": 1 },
        "tag": { "type": "string" },
        "pullPolicy": { "type": "string", "enum": ["Always", "IfNotPresent", "Never"] }
      },
      "additionalProperties": false
    },
    "resources": {
      "type": "object",
      "properties": {
        "requests": { "$ref": "#/definitions/resourceBlock" },
        "limits": { "$ref": "#/definitions/resourceBlock" }
      }
    }
  },
  "definitions": {
    "resourceBlock": {
      "type": "object",
      "properties": {
        "cpu": { "type": "string", "pattern": "^[0-9]+m?$" },
        "memory": { "type": "string", "pattern": "^[0-9]+(Mi|Gi)$" }
      }
    }
  }
}
```

Generate from values files: `helm plugin install https://github.com/losisin/helm-values-schema-json && helm schema -input values.yaml`

## The `lookup` Function

Query live cluster state during rendering. Returns empty dict in `--dry-run=client`, works with `--dry-run=server`.

```yaml
# Only create Secret if it doesn't already exist (preserve generated passwords across upgrades)
{{- if not (lookup "v1" "Secret" .Release.Namespace "myapp-credentials") }}
apiVersion: v1
kind: Secret
metadata:
  name: myapp-credentials
type: Opaque
data:
  password: {{ randAlphaNum 32 | b64enc | quote }}
{{- end }}
```

```yaml
# Iterate over existing namespaces to deploy NetworkPolicies
{{- range (lookup "v1" "Namespace" "" "").items }}
{{- if hasKey .metadata.labels "app.kubernetes.io/managed-by" }}
# ... generate NetworkPolicy for {{ .metadata.name }}
{{- end }}
{{- end }}
```

**Gotcha:** `lookup` returns nil in dry-run client mode. Guard with `{{- if .Release.IsInstall }}` or use `--dry-run=server`.

## Template Pitfalls & Solutions

### Nil Map Panics

```yaml
# WRONG — panics if .Values.ingress is nil
{{- if .Values.ingress.enabled }}

# CORRECT — safe navigation
{{- if ((.Values.ingress).enabled) }}

# Also works with dig (Sprig)
{{- if (dig "ingress" "enabled" false .Values) }}
```

### Whitespace Control

```yaml
# Produces clean YAML without extra blank lines
{{- if .Values.podAnnotations }}
      annotations:
        {{- toYaml .Values.podAnnotations | nindent 8 }}
{{- end }}
```

`-` on `{{-` trims left whitespace; `-}}` trims right. Without them, blank lines accumulate.

### Type Coercion

```yaml
# Values from --set are always strings. "true" != true
# Use quote + explicit type handling
{{- if eq (toString .Values.debug) "true" }}

# Or in schema, coerce with: {"type": ["boolean", "string"]}
```

### Range Scope

```yaml
# WRONG — $ is the global scope, . changes inside range
{{- range .Values.hosts }}
  host: {{ . }}
  service: {{ $.Values.serviceName }}   # Use $ to access root
{{- end }}
```

## CRD Management

Two approaches, both with tradeoffs:

### `crds/` Directory (Helm 3)

- CRDs in `crds/` are installed **once** on first install
- **Never updated or deleted** by Helm (by design)
- Simple but no lifecycle management

### Hook-Based CRDs

```yaml
# templates/crds.yaml
{{- if .Values.installCRDs }}
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  annotations:
    "helm.sh/hook": pre-install,pre-upgrade
    "helm.sh/hook-weight": "-5"
  name: myresources.example.com
spec:
  # ...
{{- end }}
```

Gives upgrade control but risks deleting CRDs (and all CRs) on uninstall. Add `"helm.sh/resource-policy": keep` annotation.

## Post-Renderers

Mutate rendered manifests before apply. Useful for injecting sidecars, labels, or policy annotations without modifying chart templates.

**⚠️ Helm 4 breaking change:** Post-renderers must be installed Helm plugins — passing an executable path (`--post-renderer ./script.sh`) no longer works. Use `--post-renderer <plugin-name>` with a properly installed plugin.

**Helm 3 (executable path — still works in Helm 3.x):**
```bash
helm upgrade --install myapp ./mychart --post-renderer ./inject-labels.sh
```

```bash
#!/bin/bash
# inject-labels.sh — add labels via kustomize
cat > /tmp/kustomization.yaml <<EOF
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - rendered.yaml
commonLabels:
  company.io/cost-center: platform
EOF
cat <&0 > /tmp/rendered.yaml
kustomize build /tmp
```

**Helm 4 (plugin name — required in Helm 4, also works in Helm 3.14+):**
```bash
# Install a post-renderer plugin, then reference it by name
helm plugin install https://github.com/example/helm-post-renderer-kustomize
helm upgrade --install myapp ./mychart --post-renderer kustomize
```

See [Helm 4 plugin system (HIP-0026)](https://github.com/helm/community/blob/main/hips/hip-0026.md) and [example plugins](https://github.com/scottrigby/h4-example-plugins) for WebAssembly-based post-renderer plugins.

## Multi-Tenant / Subchart Patterns

### Global Values

```yaml
# Parent values.yaml
global:
  imageRegistry: 123456789.dkr.ecr.us-east-1.amazonaws.com
  storageClass: longhorn

# Accessible in subcharts as .Values.global.imageRegistry
```

### Conditional Subcharts

```yaml
# Chart.yaml
dependencies:
  - name: postgresql
    version: "15.x"
    repository: "oci://registry-1.docker.io/bitnamicharts"
    condition: postgresql.enabled
    import-values:
      - child: primary.service
        parent: database
```

`import-values` pulls subchart values into parent scope — avoids deeply nested `postgresql.primary.service.port` paths.

### Per-Environment Overrides

```bash
# Layered values: base → environment → secrets
helm upgrade --install myapp ./mychart \
  -f values.yaml \
  -f values-production.yaml \
  -f secrets-production.yaml.dec \
  --set image.tag=${GIT_SHA}
```

## Secrets Handling

Charts should **never** contain secrets. Integration patterns:

```yaml
# External Secrets Operator — reference in templates
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: {{ include "myapp.fullname" . }}-db
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets-manager
    kind: ClusterSecretStore
  target:
    name: {{ include "myapp.fullname" . }}-db
  data:
    - secretKey: password
      remoteRef:
        key: {{ .Values.externalSecrets.dbPasswordKey }}
```

```yaml
# SOPS-encrypted values (decrypt before helm)
# values-secrets.yaml.enc → decrypt with sops
helm secrets upgrade myapp ./mychart -f values.yaml -f values-secrets.yaml
```

## Testing with helm-unittest

```yaml
# tests/deployment_test.yaml
suite: deployment
templates:
  - templates/deployment.yaml
tests:
  - it: should not render when disabled
    set:
      enabled: false
    asserts:
      - hasDocuments:
          count: 0

  - it: should set resource limits from values
    set:
      resources:
        limits:
          memory: 1Gi
    asserts:
      - equal:
          path: spec.template.spec.containers[0].resources.limits.memory
          value: 1Gi

  - it: should fail without required values
    set:
      image.repository: ""
    asserts:
      - failedTemplate: {}
```

```bash
helm plugin install https://github.com/helm-unittest/helm-unittest
helm unittest ./mychart --color
```

## Helm 4 CLI Flag Renames

Helm 4 (released November 2025) renamed two common flags. The old names still work but emit deprecation warnings — update scripts and CI pipelines:

| Old flag (Helm 3) | New flag (Helm 4) | Notes |
|---|---|---|
| `--atomic` | `--rollback-on-failure` | Auto-rollback on failed install/upgrade |
| `--force` | `--force-replace` | Delete and re-create resources on conflict |

```bash
# Helm 4 — use new flag names
helm upgrade --install myapp ./mychart --rollback-on-failure --timeout 10m
helm upgrade myapp ./mychart --force-replace
```

## Debugging

```bash
# Render one template with debug output
helm template myapp ./mychart -s templates/deployment.yaml --debug 2>&1

# Diff what would change (requires helm-diff plugin)
helm diff upgrade myapp ./mychart -f values-production.yaml

# Validate against cluster without applying
helm upgrade --install myapp ./mychart --dry-run=server --debug

# Check computed values after all overlays
helm get values myapp -n production -a  # -a = all (computed)
```

## Common Advanced Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `nil pointer evaluating interface {}` | Accessing nested key on nil parent | Use `dig` or `((.Values.x).y)` safe access |
| `schema validation failed` | values.schema.json rejects input | Run `helm lint` with your values files to see which field fails |
| CRDs not updated on upgrade | `crds/` directory is install-only | Move CRDs to templates with pre-upgrade hooks |
| `lookup` returns empty in CI | Client-side dry-run has no cluster | Use `--dry-run=server` or guard with `{{- if not .Release.IsInstall }}` |
| Post-renderer output invalid | Script must read stdin, write stdout | Ensure script outputs valid YAML manifests to stdout |
| Subchart values ignored | Wrong nesting in parent values.yaml | Values must be under subchart name key: `postgresql.auth.password` |

## References

- [`troubleshooting.md`](references/troubleshooting.md) — Install and upgrade failures, template rendering, release management, and repository issues

## Cross-References

- [argocd](../argocd/) — GitOps deployment of Helm releases via ArgoCD Applications
- [crossplane](../crossplane/) — Helm provider for deploying charts as Crossplane managed resources
- [external-secrets](../external-secrets/) — Inject secrets into Helm releases without embedding them in values
- [cert-manager](../cert-manager/) — TLS certificate management commonly deployed via Helm
- [kueue](../kueue/) — Job queueing — Helm charts for ML workloads often need Kueue integration
- [operator-sdk](../operator-sdk/) — Building operators that use Helm charts as the reconciliation engine
- [github-actions](../github-actions/) — CI/CD for Helm chart linting, packaging, and publishing
- [docker-buildx](../docker-buildx/) — Build container images referenced by Helm charts
- [prometheus-grafana](../prometheus-grafana/) — kube-prometheus-stack deployed via Helm
- [gpu-operator](../gpu-operator/) — GPU Operator deployed and configured via Helm
