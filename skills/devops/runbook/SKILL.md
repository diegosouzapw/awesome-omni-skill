---
name: runbook
description: Generate operational runbooks for services, procedures, or incident response with step-by-step procedures, troubleshooting guides, and escalation paths
license: MIT
compatibility:
  - runtime:any
allowed-tools:
  - Read
  - Glob
  - Grep
  - Write
metadata:
  author: thoreinstein
  version: 1.0.0
---

# Runbook

Generate operational runbooks for services, procedures, or incident response. Investigates the codebase and infrastructure to produce accurate, actionable procedures.

## When to Use

- Creating operational documentation for a service
- Documenting deployment, scaling, or maintenance procedures
- Building incident response playbooks
- Standardizing operational procedures across teams

## Input

- **Topic**: Service name, operation type, or incident scenario
- **Scope**: deployment, scaling, failover, maintenance, troubleshooting
- **Optional**: Specific scenarios to cover

## Investigation Strategy

Launch parallel investigation tracks to gather comprehensive information:

### Track 1: Codebase Exploration

- Identify service entry points and configuration
- Find health check endpoints
- Map dependencies (databases, caches, external services)
- Locate logging and metrics instrumentation
- Find existing scripts or automation

### Track 2: Infrastructure Analysis

- Review deployment manifests (Kubernetes, Terraform, etc.)
- Identify scaling configuration
- Map service dependencies
- Find monitoring and alerting setup
- Review backup and recovery procedures

### Track 3: External Research

- Find operational best practices for the service type
- Research common failure modes
- Identify industry-standard procedures

## Output

Generate the runbook document using the template at `references/templates/runbook.md`.

The runbook should include:
- Service overview and architecture
- Dependencies with failure impact
- Step-by-step procedures with actual commands
- Troubleshooting guides for common issues
- Escalation paths and contacts

## Behavior

1. Parse topic to identify service and operation scope
2. Launch parallel investigation tracks
3. Extract configuration, endpoints, and dependencies from codebase
4. Identify common operations and failure modes
5. Generate step-by-step procedures with actual commands
6. Document troubleshooting steps and escalation paths

## Constraints

- **Accuracy**: All commands must be verified against actual codebase/infrastructure
- **Actionable**: Every procedure must have concrete, executable steps
- **Complete**: Include prerequisites, verification, and rollback for each procedure
- **Maintainable**: Note dependencies that may change and require updates

## Example

```
Input: "Generate runbook for the payment-service"

Investigation:
- Found deployment at k8s/payment-service/
- Found health endpoints: /health, /ready
- Dependencies: PostgreSQL (critical), Redis (cache), Stripe API
- Scaling: HPA configured, min 3, max 10 replicas
- Alerts: Prometheus rules in monitoring/

Generated Runbook: payment-service-runbook.md

## Overview
- Service: payment-service
- Owner: payments-team
- Criticality: P1

## Dependencies
| Dependency | Type | Criticality | Failure Impact |
|------------|------|-------------|----------------|
| PostgreSQL | Database | Critical | Full outage |
| Redis | Cache | High | Degraded latency |
| Stripe API | External | Critical | Payment failures |

## Procedures

### Deployment
1. Verify no active transactions
   ```bash
   kubectl exec -it payment-service-0 -- curl localhost:8080/metrics | grep active_transactions
   ```
2. Apply new deployment
   ```bash
   kubectl apply -f k8s/payment-service/deployment.yaml
   ```
3. Monitor rollout
   ```bash
   kubectl rollout status deployment/payment-service
   ```

### Scaling
```bash
kubectl scale deployment payment-service --replicas=5
```

## Troubleshooting

### High Latency
**Symptoms**: p99 latency > 500ms
**Diagnosis**:
```bash
kubectl top pods -l app=payment-service
kubectl logs -l app=payment-service --tail=100 | grep -i slow
```
**Resolution**: Check Redis connection, scale if CPU > 80%
```

Begin by identifying the service or operation to document and launching investigation tracks.
