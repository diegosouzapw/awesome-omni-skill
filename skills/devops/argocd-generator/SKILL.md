---
name: argocd-generator
description: Generates ArgoCD Application manifests following the App-of-Apps pattern.
---

# ArgoCD Application Generator

## YAML RULES (CRITICAL)
- Each key MUST appear EXACTLY ONCE in its parent map
- NEVER repeat configuration blocks
- Keep values MINIMAL - only necessary overrides

## File Location
```
platform/stacks/{NN}-{category}/overlays/home/{app-name}.yaml
```

## Stack Categories
| Stack | Path | Purpose |
|-------|------|---------|
| 00-core | Core infra | cert-manager, cloudflared |
| 01-platforms | Platform services | argo-workflows, harbor |
| 02-o11y | Observability | grafana, tempo |
| 03-data | Databases | postgres, redis |
| 04-ml | ML services | feast, ray, mlflow |
| 05-workloads | Applications | |
| 06-labs | Experimental | |

---

## IMPORTANT: Always Use Combined Pattern

**For any app that needs secrets (database passwords, API keys, etc.), ALWAYS create TWO apps in ONE file:**

1. **`{app}-raw`** (syncWave: "5") - InfisicalSecret to fetch secrets
2. **`{app}`** (syncWave: "10") - Main Helm chart with dependency on raw

---

## Complete Example: App with Secrets

This is the **standard pattern** for most applications:

```yaml
apps:
  # Step 1: Deploy InfisicalSecret first (lower syncWave)
  - name: superset-raw
    namespace: superset
    project: dev
    syncWave: "5"
    source:
      repoURL: https://bedag.github.io/helm-charts
      chart: raw
      targetRevision: "2.0.2"
      helm:
        releaseName: superset-raw
        values: |
          resources:
            - apiVersion: secrets.infisical.com/v1alpha1
              kind: InfisicalSecret
              metadata:
                name: superset-managed-secrets
                namespace: superset
              spec:
                hostAPI: https://app.infisical.com/api
                authentication:
                  universalAuth:
                    secretsScope:
                      projectSlug: home-lab
                      envSlug: "hme"
                      secretsPath: /superset
                    credentialsRef:
                      secretName: infisical-secrets
                      secretNamespace: infisical
                managedKubeSecretReferences:
                  - secretName: superset-managed-secrets
                    secretNamespace: superset
                    creationPolicy: Owner

  # Step 2: Deploy Helm chart with dependency
  - name: superset
    namespace: superset
    project: dev
    syncWave: "10"
    dependencies:
      - superset-raw
    source:
      repoURL: http://apache.github.io/superset/
      chart: superset
      targetRevision: "0.15.0"
      helm:
        releaseName: superset
        values: |
          replicaCount: 1
          postgresql:
            enabled: false
          redis:
            enabled: false
          persistence:
            enabled: true
            size: 10Gi
          resources:
            requests:
              cpu: 500m
              memory: 1Gi
          # Reference the managed secret
          extraEnvVarsSecret: superset-managed-secrets
```

---

## Simple Example: App without Secrets

Only use this pattern if the app truly needs NO secrets:

```yaml
apps:
  - name: myapp
    namespace: myapp
    project: dev
    syncWave: "10"
    source:
      repoURL: https://charts.example.io
      chart: myapp
      targetRevision: "1.0.0"
      helm:
        releaseName: myapp
        values: |
          replicaCount: 1
          resources:
            requests:
              cpu: 500m
              memory: 1Gi
```

---

## InfisicalSecret Template

Replace `{app}` with actual app name:

```yaml
- apiVersion: secrets.infisical.com/v1alpha1
  kind: InfisicalSecret
  metadata:
    name: {app}-managed-secrets
    namespace: {app}
  spec:
    hostAPI: https://app.infisical.com/api
    authentication:
      universalAuth:
        secretsScope:
          projectSlug: home-lab
          envSlug: "hme"
          secretsPath: /{app}
        credentialsRef:
          secretName: infisical-secrets
          secretNamespace: infisical
    managedKubeSecretReferences:
      - secretName: {app}-managed-secrets
        secretNamespace: {app}
        creationPolicy: Owner
```

---

## Notes
- **project**: always `dev`
- **syncWave**: 5 for secrets, 10 for apps (lower = earlier)
- **Bedag raw chart**: always version `2.0.2`
- **InfisicalSecret**: required for any app with database/API credentials
- **dependencies**: ensures secrets are created before the app
