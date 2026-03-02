---
name: argocd-audit
description: Audits ArgoCD Application manifests and raw K8s resources for anti-patterns, security issues, and best practice violations. Use when asked to audit, review, or check ArgoCD/GitOps quality. Generates a comprehensive report under reports/YYYY-MM-DD/argocd-audit.md. (project)
---

# Purpose

Enforce ArgoCD and Kubernetes manifest quality and security standards across `flux/apps/` and `raw-manifests/` directories through automated checks.

**What it checks (11 checks):**
1. Application Source (targetRevision pinned, HTTPS) - HIGH
2. SyncPolicy Config (automated, prune, selfHeal) - HIGH
3. Hardcoded Secrets (no secrets in Git) - HIGH
4. RBAC Wildcards (no * permissions) - HIGH
5. Istio Gateway TLS (TLS enabled, credentials) - HIGH
6. Application Project (not default, restrictions) - MEDIUM
7. ApplicationSet (goTemplate, missingkey=error) - MEDIUM
8. VirtualService Security (gateway, hosts) - MEDIUM
9. Deprecated APIs (no v1beta1) - MEDIUM
10. Namespace Spec (explicit namespace) - LOW
11. Metadata Best Practices (labels, annotations) - LOW

# Running Checks

**Full audit (all checks):**
```bash
node .claude/skills/argocd-audit/scripts/run_all_checks.mjs
```

**Generate report (all checks + markdown report):**
```bash
node .claude/skills/argocd-audit/scripts/generate_report.mjs
```
Report saved to: `reports/YYYY-MM-DD/argocd-audit.md`

**Individual checks:**
```bash
node .claude/skills/argocd-audit/scripts/check_application_source.mjs
node .claude/skills/argocd-audit/scripts/check_sync_policy.mjs
node .claude/skills/argocd-audit/scripts/check_hardcoded_secrets.mjs
node .claude/skills/argocd-audit/scripts/check_rbac_wildcards.mjs
node .claude/skills/argocd-audit/scripts/check_istio_gateway_tls.mjs
node .claude/skills/argocd-audit/scripts/check_application_project.mjs
node .claude/skills/argocd-audit/scripts/check_applicationset.mjs
node .claude/skills/argocd-audit/scripts/check_virtualservice.mjs
node .claude/skills/argocd-audit/scripts/check_deprecated_apis.mjs
node .claude/skills/argocd-audit/scripts/check_namespace_spec.mjs
node .claude/skills/argocd-audit/scripts/check_metadata.mjs
```

# Quality Rules

## 1. Application Source Validation (HIGH)

**RULE**: Pin source references for reproducibility. HEAD/floating refs = unpredictable deployments.

**Violations**:
- `targetRevision: HEAD` - can change without notice
- `repoURL: http://...` - insecure, use HTTPS
- Missing `targetRevision` - defaults to HEAD
- Chart source without `targetRevision`

**Fix**: Use `targetRevision: main` or `targetRevision: v1.0.0`.

---

## 2. SyncPolicy Configuration (HIGH)

**RULE**: Configure automated sync properly. Manual sync = drift accumulation.

**Violations**:
- No `syncPolicy.automated` block - manual sync required
- `prune: false` - orphaned resources accumulate
- `selfHeal: false` - manual drift correction required
- No `retry` configuration - transient failures not handled

**Fix**: Add `automated: { prune: true, selfHeal: true }` with retry config.

---

## 3. Hardcoded Secrets (HIGH)

**RULE**: Never store secrets in Git. Base64 encoding is NOT encryption.

**Violations**:
- `kind: Secret` with `data:` field - secrets exposed in Git
- `kind: Secret` with `stringData:` field - plaintext secrets
- Hardcoded passwords, tokens, API keys in any YAML
- Base64-encoded credentials (easily decoded)

**Fix**: Use External Secrets, Sealed Secrets, or HashiCorp Vault.

---

## 4. RBAC Wildcards (HIGH)

**RULE**: Follow least-privilege principle. Wildcard permissions = cluster takeover risk.

**Violations**:
- `verbs: ["*"]` - grants all actions
- `resources: ["*"]` - access to all resource types
- `apiGroups: ["*"]` - access across all API groups
- `roleRef.name: cluster-admin` - full cluster access
- `verbs: [impersonate]` - can act as other users

**Fix**: Use explicit verbs like `[get, list, watch]`, explicit resources like `[pods, services]`.

---

## 5. Istio Gateway TLS (HIGH)

**RULE**: Enable TLS for production traffic. No TLS = unencrypted traffic.

**Violations**:
- HTTP server without TLS counterpart - plaintext traffic
- No `credentialName` for HTTPS - certificate not specified
- Missing `tls.mode` - TLS not configured
- `hosts: ["*"]` - overly broad, security risk
- HTTP-only gateway for production hosts

**Fix**: Add HTTPS server with `tls.mode: SIMPLE` and `credentialName: secret-name`.

---

## 6. Application Project Configuration (MEDIUM)

**RULE**: Use dedicated projects for isolation. `default` project = no restrictions.

**Violations**:
- `project: default` - no source/destination restrictions
- No project-level RBAC configured
- `sourceNamespaces` unrestricted
- `destinations` allowing all clusters

**Fix**: Create dedicated AppProject with proper sourceRepos and destinations restrictions.

---

## 7. ApplicationSet Best Practices (MEDIUM)

**RULE**: Use goTemplate with error handling. Silent template failures = hidden bugs.

**Violations**:
- Missing `goTemplate: true` - using legacy template engine
- Missing `goTemplateOptions: ["missingkey=error"]` - silent failures
- Generator without proper configuration
- ApplicationSet without preserveResourcesOnDeletion consideration

**Fix**: Enable `goTemplate: true` with `goTemplateOptions: ["missingkey=error"]`.

---

## 8. VirtualService Security (MEDIUM)

**RULE**: Validate gateway associations. Orphan VirtualService = unreachable services.

**Violations**:
- No `gateways` field specified - orphan VirtualService
- `hosts: ["*"]` - overly broad matching
- Missing destination `host` or `port`
- No route timeout configured

**Fix**: Associate with valid Gateway, use specific hosts like `["app.example.com"]`.

---

## 9. Deprecated Kubernetes APIs (MEDIUM)

**RULE**: Use stable Kubernetes APIs. Deprecated APIs = upgrade failures.

**Violations**:
- `extensions/v1beta1` - removed in K8s 1.22
- `networking.k8s.io/v1beta1` Ingress - removed in K8s 1.22
- `apps/v1beta1`, `apps/v1beta2` - removed in K8s 1.16
- `batch/v1beta1` CronJob - removed in K8s 1.25
- `policy/v1beta1` PodSecurityPolicy - removed in K8s 1.25

**Fix**: Update to stable APIs: `apps/v1`, `networking.k8s.io/v1`, `batch/v1`.

---

## 10. Namespace Specification (LOW)

**RULE**: Explicitly specify namespaces. Implicit namespace = deployment confusion.

**Violations**:
- Missing `metadata.namespace` - relies on kubectl context
- Inconsistent namespace usage across related resources
- Cluster-scoped resources without proper scope handling

**Fix**: Add explicit `namespace` field to all namespaced resources.

---

## 11. Metadata Best Practices (LOW)

**RULE**: Add labels for observability and management. No labels = hard to query/manage.

**Violations**:
- Missing `app` or `app.kubernetes.io/name` label
- Missing `component` or `app.kubernetes.io/component` label
- No annotations for documentation
- Missing `part-of` for application grouping

**Fix**: Add standard labels: `app.kubernetes.io/name`, `app.kubernetes.io/component`, `app.kubernetes.io/part-of`.

---

# Detection Philosophy

This skill uses **VALUE-BASED detection**:
- Detects issues by actual values and patterns, not by variable/field names
- Future-proof: new manifests with issues are automatically detected
- No need to update scripts when new applications are added

# Target Directories

- **flux/apps/** - ArgoCD Application/ApplicationSet manifests, Helm values
- **raw-manifests/** - Raw K8s resources (Istio, MetalLB, Secrets, RBAC, Flux CRDs)

# Parsing Strategy

- **YAML files**: Regex-based parsing for Go template compatibility
- **Multi-document YAML**: Handles `---` separators
- **File extensions**: `.yaml`, `.yml`

# Safety

- Read-only operation (except report generation)
- No ArgoCD resources modified
- No cluster changes
- No Git modifications
