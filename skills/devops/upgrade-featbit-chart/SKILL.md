---
name: upgrade-featbit-chart
description: Guides safe FeatBit Helm chart upgrades on Kubernetes and AKS. Use when user asks to upgrade FeatBit, update FeatBit version, deploy new FeatBit release, check FeatBit migrations, or troubleshoot FeatBit upgrade issues.
license: MIT
metadata:
  author: FeatBit
  version: 1.0.0
  category: kubernetes-deployment
---

# Upgrade FeatBit Deployment

Guides safe, three-step upgrades of FeatBit deployments: check database migrations, test locally, deploy to production.

## Quick Start

```bash
# 1. Check migrations
ls migration/ && cat migration/RELEASE-v<version>.md

# 2. Test locally
kubectl config use-context docker-desktop
helm upgrade featbit-test charts/featbit \
  -f charts/featbit/examples/standard/featbit-standard-local-pg.yaml

# 3. Deploy to AKS
az aks get-credentials --resource-group <rg> --name <cluster>
helm upgrade <release> featbit/featbit \
  -f charts/featbit/examples/aks/featbit-aks-values.local.yaml \
  --namespace <ns>
```

## Three-Step Upgrade Workflow

### Step 1: Check Database Migrations

⚠️ **CRITICAL**: FeatBit does NOT auto-execute migrations.

```bash
# List migration files
ls migration/

# Read migration for target version
cat migration/RELEASE-v<target-version>.md
```

**Actions required:**
- [ ] Review database schema changes
- [ ] Backup database
- [ ] Execute migrations manually (DBA)
- [ ] Verify schema changes applied

📄 **Complete Guide**: [references/database-migrations.md](references/database-migrations.md)

### Step 2: Test Locally on Docker Desktop

Test upgrades locally before production:

```bash
# Switch to Docker Desktop K8s
kubectl config use-context docker-desktop

# Dry-run upgrade
helm upgrade featbit-test charts/featbit \
  -f charts/featbit/examples/standard/featbit-standard-local-pg.yaml \
  --dry-run

# Apply if dry-run succeeds
helm upgrade featbit-test charts/featbit \
  -f charts/featbit/examples/standard/featbit-standard-local-pg.yaml

# Monitor rollout
kubectl get pods -w
```

**Verification:**
```bash
kubectl get pods                               # All Running
kubectl logs -l app.kubernetes.io/name=featbit-api --tail=50
kubectl port-forward service/featbit-ui 8081:8081
curl http://localhost:5000/health              # API health check
```

📄 **Complete Guide**: [references/local-testing-guide.md](references/local-testing-guide.md)

### Step 3: Deploy to Production (AKS)

After successful local testing:

```bash
# Azure login and AKS access
az login
az aks get-credentials --resource-group <rg> --name <cluster>
kubectl config use-context <aks-context>

# Verify current release
helm list -A | grep featbit

# Update Helm repo
helm repo update featbit

# Dry-run with AKS values
helm upgrade <release-name> featbit/featbit \
  -f charts/featbit/examples/aks/featbit-aks-values.local.yaml \
  --namespace <namespace> \
  --dry-run

# Apply if dry-run succeeds
helm upgrade <release-name> featbit/featbit \
  -f charts/featbit/examples/aks/featbit-aks-values.local.yaml \
  --namespace <namespace>

# Monitor deployment
kubectl get pods -n <namespace> -w
```

**Post-upgrade verification:**
```bash
kubectl rollout status deployment/featbit-api -n <ns>
curl https://api.yourdomain.com/health
curl https://els.yourdomain.com/health
```

📄 **Complete Guide**: [references/aks-deployment-guide.md](references/aks-deployment-guide.md)

## Critical Concepts

**Database Migrations:**
- NOT automated by Helm chart
- Located in `migration/RELEASE-v{version}.md`
- Must be executed manually before upgrade
- Always backup database first

**Chart Sources:**
- Local testing: `charts/featbit` (local path)
- AKS/Production: `featbit/featbit` (published from Helm repo)
- Never mix local and published charts

**External URLs (Production):**
```yaml
apiExternalUrl: "https://api.featbit.com"
evaluationServerExternalUrl: "https://els.featbit.com"
```
⚠️ Without these, client SDKs cannot connect.

## Pre-Upgrade Checklist

**Database:**
- [ ] Migration scripts reviewed and executed
- [ ] Database backup completed and verified
- [ ] Connection strings validated

**Kubernetes:**
- [ ] kubectl access to target cluster
- [ ] Helm >= 3.7.0, kubectl >= 1.23 installed
- [ ] Sufficient cluster resources

**Configuration:**
- [ ] External URLs configured (production)
- [ ] Managed services validated (PostgreSQL/Redis)
- [ ] Secrets and ConfigMaps updated
- [ ] TLS certificates valid

**Operational:**
- [ ] Current Helm revision noted (rollback plan)
- [ ] Team notified
- [ ] Monitoring ready

## Troubleshooting

### Quick Rollback
```bash
helm history <release> -n <namespace>
helm rollback <release> <revision> -n <namespace>
```

### Debug Commands
```bash
kubectl get pods -n <namespace>
kubectl describe pod <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace>
kubectl get events -n <namespace> --sort-by='.lastTimestamp'
```

📄 **Complete Guide**: [references/troubleshooting.md](references/troubleshooting.md)

## Common Issues

| Issue | Quick Fix |
|-------|-----------|
| Migration not executed | Execute migration scripts manually |
| Pods not starting | Check resources: `kubectl describe pod` |
| Connection errors | Verify ConfigMaps and secrets |
| Ingress not working | Check ingress controller and TLS |
| Database connection failure | Verify firewall rules and credentials |

See [references/troubleshooting.md](references/troubleshooting.md) for detailed solutions.

## Best Practices

1. ✅ **Always test locally first** - Catch issues early
2. ✅ **Use dry-run extensively** - Simulate before applying
3. ✅ **Review migrations** - Understand database changes
4. ✅ **Backup before upgrade** - Database and configurations
5. ✅ **Monitor during upgrade** - Watch logs and status
6. ✅ **Have rollback plan** - Note current revision
7. ✅ **Staged rollout** - Dev → Staging → Production
8. ✅ **External URLs** - Required for production
9. ✅ **Managed services** - Use external providers for production
10. ✅ **Document changes** - Track customizations

## Reference Guides

- [references/database-migrations.md](references/database-migrations.md) - Migration workflow, backup, rollback
- [references/local-testing-guide.md](references/local-testing-guide.md) - Docker Desktop K8s testing
- [references/aks-deployment-guide.md](references/aks-deployment-guide.md) - AKS deployment, Azure CLI
- [references/troubleshooting.md](references/troubleshooting.md) - Common issues and solutions

## Tools Required

- Helm >= 3.7.0
- kubectl >= 1.23
- Azure CLI (for AKS)
- Docker Desktop with Kubernetes (for local testing)

## Support

- **Repository**: https://github.com/featbit/featbit-charts
- **Documentation**: https://docs.featbit.co/
- **Issues**: https://github.com/featbit/featbit-charts/issues
