---
name: deployment-safety
description: Pre-deployment checklists, rollback strategies, and post-deploy verification. Use this skill when preparing to deploy code, reviewing deployment processes, or setting up CI/CD pipelines.
alwaysApply: false
---

# Deployment Safety

You are a senior DevOps engineer reviewing deployments. Apply these checklists and strategies to ensure safe, reliable releases.

## Pre-Deployment Checklist

Run through before every production deployment:

### Code Readiness
- [ ] All tests passing (unit, integration, e2e)
- [ ] Code reviewed and approved
- [ ] No unresolved merge conflicts
- [ ] Feature flags in place for risky changes
- [ ] Database migrations tested on staging
- [ ] API backward compatibility verified (no breaking changes without versioning)
- [ ] Dependencies updated and locked (`package-lock.json`, `go.sum`, etc.)

### Infrastructure Readiness
- [ ] Staging deployment successful and verified
- [ ] Resource limits configured (CPU, memory, replicas)
- [ ] Health check endpoints responding
- [ ] Monitoring and alerting configured for new features
- [ ] Log collection working for new components
- [ ] Secrets and environment variables configured in production
- [ ] SSL/TLS certificates valid and not expiring soon

### Rollback Plan
- [ ] Previous version tagged and accessible
- [ ] Rollback procedure documented and tested
- [ ] Database rollback plan if migrations are involved
- [ ] Feature flags that can disable new features quickly
- [ ] Communication plan if rollback is needed

### Timing
- [ ] Not deploying on Friday afternoon (unless critical)
- [ ] Not deploying during peak traffic hours
- [ ] Team available to monitor post-deploy
- [ ] No conflicting deployments from other teams

## Deployment Strategies

### Rolling Update (Default)
```
Old: [v1] [v1] [v1] [v1]
     [v2] [v1] [v1] [v1]  ← replace one at a time
     [v2] [v2] [v1] [v1]
     [v2] [v2] [v2] [v1]
     [v2] [v2] [v2] [v2]  ← done
```
**Use when**: Standard releases, stateless services
**Risk**: Mixed versions serve traffic during rollout

### Blue-Green
```
Blue  (current): [v1] [v1] [v1] ← all traffic
Green (new):     [v2] [v2] [v2] ← ready, no traffic

Switch: Blue → standby, Green → active
```
**Use when**: Zero-downtime required, easy rollback needed
**Risk**: Requires 2x infrastructure temporarily

### Canary
```
[v1] [v1] [v1] [v1] [v1]  ← 100% traffic
[v2] [v1] [v1] [v1] [v1]  ← 20% to v2, monitor
[v2] [v2] [v1] [v1] [v1]  ← 40% to v2, monitor
[v2] [v2] [v2] [v2] [v2]  ← 100% after validation
```
**Use when**: High-risk changes, gradual confidence building
**Risk**: Slower rollout, users may see inconsistent behavior

### Feature Flags
```
v2 deployed to all instances with flag OFF
Flag ON for internal team → test
Flag ON for 5% of users → canary
Flag ON for 100% → full release
```
**Use when**: Decoupling deploy from release, A/B testing
**Risk**: Flag complexity, flag cleanup debt

## Post-Deployment Verification

### Immediate (First 5 minutes)
- [ ] Health check endpoints returning 200
- [ ] No spike in error rates (4xx, 5xx)
- [ ] Response times within normal range
- [ ] Logs show successful startup
- [ ] No crash loops or OOM kills

### Short-term (First 30 minutes)
- [ ] Key business metrics stable (orders, sign-ups, API calls)
- [ ] No increase in support tickets
- [ ] Memory/CPU usage stable (no leaks)
- [ ] Database connections stable
- [ ] Queue depth not growing unexpectedly

### Long-term (First 24 hours)
- [ ] No slow degradation patterns
- [ ] Scheduled jobs completing successfully
- [ ] No edge case errors accumulating
- [ ] Resource usage trending normally

## Database Migration Safety

### DO
- Add new columns as nullable or with defaults
- Create new tables before referencing them in code
- Add indexes concurrently (`CREATE INDEX CONCURRENTLY` in PostgreSQL)
- Test rollback of every migration on staging
- Run migrations before deploying new code (expand-then-contract)

### DON'T
- Drop columns or tables in the same deploy that removes the code using them
- Add NOT NULL constraints without a default value on existing columns
- Run long-running migrations during peak traffic
- Combine schema changes with large data migrations

### Expand-Contract Pattern
```
Deploy 1: Add new column (nullable)      ← expand
Deploy 2: Code writes to both old + new  ← dual-write
Deploy 3: Backfill old data to new column ← migrate
Deploy 4: Code reads from new column     ← switch
Deploy 5: Drop old column                ← contract
```

## Rollback Procedures

### Application Rollback
```bash
# Docker/K8s
kubectl rollout undo deployment/<name>
# or
kubectl set image deployment/<name> <container>=<previous-image>

# Git-based (Heroku, Render, etc.)
git revert HEAD && git push

# Blue-Green
# Switch load balancer back to blue environment
```

### Database Rollback
```bash
# If using migration tool
migrate down 1

# If manual
# Run the DOWN migration SQL script
# Verify data integrity
```

### When NOT to Roll Back
- Data has been written in new format (would lose data)
- External systems already received new-format data
- Rollback would cause more disruption than the bug
→ Instead: **fix forward** with a hotfix
