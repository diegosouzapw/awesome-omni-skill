---
name: deploy
description: Full deployment lifecycle including pre-checks, execution, verification, rollback, and documentation.
---

# OPS Deployment Skill

## Overview
The deployment skill covers the full lifecycle of deploying code to production: pre-checks, execution, verification, rollback, and documentation. Every deployment follows this process. No shortcuts.

## Pre-Deployment Checklist

Before any deployment, verify ALL of the following:

- [ ] **CQ Review passed:** Confirm a CQ approval comment exists on the ticket
- [ ] **QA Testing passed:** Confirm a QA PASS comment exists on the ticket
- [ ] **Ticket is in `rfp` status:** Only deploy tickets that have reached Ready for Production
- [ ] **Infrastructure health is green:** All existing services are healthy before introducing changes
- [ ] **Dependencies are in place:** Database migrations staged, environment variables set, external service dependencies confirmed
- [ ] **Rollback plan documented:** You know exactly how to revert this deployment
- [ ] **Deployment notes reviewed:** Any special instructions from DEV, CQ, or QA are understood
- [ ] **No conflicting deployments in progress:** One deployment at a time to isolate issues

If ANY item on this checklist is not satisfied, STOP. Do not deploy. Communicate the blocker in #standup and on the ticket.

## Docker Deployment Commands

### Build and Tag the Image

```bash
docker build -t ${SERVICE_NAME}:${VERSION} -f Dockerfile .
docker tag ${SERVICE_NAME}:${VERSION} ${REGISTRY}/${SERVICE_NAME}:${VERSION}
docker tag ${SERVICE_NAME}:${VERSION} ${REGISTRY}/${SERVICE_NAME}:latest
```

### Push to Registry

```bash
docker push ${REGISTRY}/${SERVICE_NAME}:${VERSION}
docker push ${REGISTRY}/${SERVICE_NAME}:latest
```

### Deploy with Docker Compose

```bash
# Pull latest images
docker compose -f docker-compose.prod.yml pull

# Deploy with zero-downtime (if configured)
docker compose -f docker-compose.prod.yml up -d --remove-orphans

# Verify containers are running
docker compose -f docker-compose.prod.yml ps
```

### Direct Container Deployment

```bash
# Stop the old container
docker stop ${SERVICE_NAME} || true
docker rm ${SERVICE_NAME} || true

# Run the new container
docker run -d \
  --name ${SERVICE_NAME} \
  --restart unless-stopped \
  --network ${NETWORK_NAME} \
  -p ${HOST_PORT}:${CONTAINER_PORT} \
  --env-file .env.production \
  ${REGISTRY}/${SERVICE_NAME}:${VERSION}

# Verify the container is healthy
docker inspect --format='{{.State.Health.Status}}' ${SERVICE_NAME}
```

## Kubernetes Deployment Commands

### Apply Deployment Manifest

```bash
# Apply the deployment
kubectl apply -f k8s/deployment.yaml -n ${NAMESPACE}

# Watch the rollout
kubectl rollout status deployment/${DEPLOYMENT_NAME} -n ${NAMESPACE} --timeout=300s
```

### Verify the Deployment

```bash
# Check pod status
kubectl get pods -n ${NAMESPACE} -l app=${APP_LABEL}

# Check pod logs for errors
kubectl logs -n ${NAMESPACE} -l app=${APP_LABEL} --tail=50

# Describe deployment for events
kubectl describe deployment/${DEPLOYMENT_NAME} -n ${NAMESPACE}
```

### Scale a Deployment

```bash
# Scale up/down
kubectl scale deployment/${DEPLOYMENT_NAME} -n ${NAMESPACE} --replicas=${REPLICA_COUNT}

# Verify scaling
kubectl get pods -n ${NAMESPACE} -l app=${APP_LABEL}
```

## Rollback Procedures

### Docker Rollback

```bash
# Stop the current (broken) container
docker stop ${SERVICE_NAME}
docker rm ${SERVICE_NAME}

# Run the previous version
docker run -d \
  --name ${SERVICE_NAME} \
  --restart unless-stopped \
  --network ${NETWORK_NAME} \
  -p ${HOST_PORT}:${CONTAINER_PORT} \
  --env-file .env.production \
  ${REGISTRY}/${SERVICE_NAME}:${PREVIOUS_VERSION}

# Verify rollback
docker inspect --format='{{.State.Health.Status}}' ${SERVICE_NAME}
```

### Docker Compose Rollback

```bash
# Revert to previous image tags in compose file, then:
docker compose -f docker-compose.prod.yml up -d --remove-orphans
docker compose -f docker-compose.prod.yml ps
```

### Kubernetes Rollback

```bash
# Rollback to the previous revision
kubectl rollout undo deployment/${DEPLOYMENT_NAME} -n ${NAMESPACE}

# Watch the rollback
kubectl rollout status deployment/${DEPLOYMENT_NAME} -n ${NAMESPACE} --timeout=300s

# Verify pods are healthy after rollback
kubectl get pods -n ${NAMESPACE} -l app=${APP_LABEL}

# Check rollout history
kubectl rollout history deployment/${DEPLOYMENT_NAME} -n ${NAMESPACE}
```

### Rollback Decision Criteria
Execute a rollback immediately if any of the following occur:
- Health checks fail after deployment
- Error rates spike above baseline threshold
- Response times degrade significantly
- Critical functionality is broken
- Data integrity issues detected

Do NOT attempt to "fix forward" if the issue is unclear. Roll back first, then investigate.

## Post-Deployment Verification

After every deployment, verify ALL of the following:

### Health Checks

```bash
# HTTP health check
curl -sf ${SERVICE_URL}/health || echo "HEALTH CHECK FAILED"

# Detailed health with dependencies
curl -s ${SERVICE_URL}/health/detailed | jq .
```

### Smoke Tests

```bash
# Verify critical endpoints are responding
curl -sf -o /dev/null -w "%{http_code}" ${SERVICE_URL}/api/status
curl -sf -o /dev/null -w "%{http_code}" ${SERVICE_URL}/api/ping
```

### Monitoring Verification

```bash
# Check error rates (should not spike)
# Check response times (should not degrade)
# Check resource usage (should be within normal bounds)
# Check logs for new error patterns
docker logs ${SERVICE_NAME} --since 5m 2>&1 | grep -i error || echo "No errors found"
```

## Monitoring Setup

Every deployed service must have:
- **Health check endpoint:** Returns 200 when healthy, non-200 when degraded
- **Metrics exposure:** CPU, memory, request count, error rate, response time
- **Log aggregation:** Structured logs shipped to central logging
- **Alerting rules:** Alerts on health check failure, error rate spike, resource exhaustion

## Deployment Documentation Template

Every deployment gets a comment on the ticket with this structure:

```markdown
## Deployment Record

**Ticket:** #[TICKET_ID]
**Deployed by:** ops
**Timestamp:** [ISO 8601 timestamp]
**Environment:** [production/staging]
**Version:** [version tag or commit hash]
**Previous Version:** [what was running before]

### Changes Deployed
[Brief description of what this deployment includes]

### Deployment Method
[Docker Compose / Kubernetes / Direct Container]

### Pre-Deployment Checks
- [x] CQ review confirmed
- [x] QA pass confirmed
- [x] Infrastructure health verified
- [x] Rollback plan prepared
- [x] Dependencies in place

### Post-Deployment Verification
- [x] Health checks passing
- [x] Smoke tests passing
- [x] Error rates nominal
- [x] Response times nominal
- [x] Monitoring dashboards green

### Rollback Plan
To rollback this deployment:
1. [Exact rollback command or steps]
2. [Verification after rollback]
3. [Notify team in #standup]

### Notes
[Any observations, warnings, or follow-up items]
```
