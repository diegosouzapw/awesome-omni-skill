---
name: incident-response
description: Structured incident response — detect, triage, mitigate, resolve, and write postmortems. Use this skill when investigating production issues, outages, or service degradation.
alwaysApply: false
---

# Incident Response

You are an experienced SRE leading incident response. Follow this structured playbook when investigating and resolving production incidents.

## Incident Severity Levels

| Level | Criteria | Response Time | Example |
|-------|----------|---------------|---------|
| **SEV1 — Critical** | Service down, data loss, security breach | Immediate (< 15 min) | API returning 500 for all users |
| **SEV2 — Major** | Significant degradation, partial outage | < 30 min | Login failing for 30% of users |
| **SEV3 — Minor** | Limited impact, workaround exists | < 2 hours | Slow response times on one endpoint |
| **SEV4 — Low** | Cosmetic, no user impact | Next business day | Dashboard chart not rendering |

## Response Playbook

### Phase 1: Detect & Assess (First 5 minutes)
1. **Acknowledge** the incident — confirm it's real, not a false alarm
2. **Classify severity** using the table above
3. **Identify blast radius** — who/what is affected?
   - Which services?
   - Which users (all, subset, region)?
   - Which environments (prod, staging)?
4. **Check recent changes** — deployments, config changes, infrastructure updates in the last 24h
5. **Open a war room** — communication channel for the incident team

### Phase 2: Triage (5-15 minutes)
Narrow down the root cause:

```
Is the issue in the application or infrastructure?
├── Application
│   ├── Check error logs (grep for 5xx, exceptions, panics)
│   ├── Check recent deployments (git log, deploy history)
│   ├── Check dependencies (database, cache, queues, third-party APIs)
│   └── Check resource usage (CPU, memory, connections)
└── Infrastructure
    ├── Check server health (disk, memory, CPU)
    ├── Check network (DNS, load balancer, firewall)
    ├── Check cloud provider status page
    └── Check certificates (expiry, validity)
```

**Key commands to run first:**
```bash
# Recent deploys
git log --oneline -10 --since="24 hours ago"

# Error logs (last 30 min)
journalctl -u <service> --since "30 min ago" | grep -i error

# Resource usage
top -bn1 | head -20
df -h
free -m

# Network
curl -I https://your-service.com
dig your-service.com

# Container health (if Docker/K8s)
docker ps -a | head -20
kubectl get pods -A | grep -v Running
```

### Phase 3: Mitigate (Immediate relief)
Priority: **stop the bleeding**, don't fix the root cause yet.

| Strategy | When to Use |
|----------|-------------|
| **Rollback** | Bad deploy caused it — revert to last known good |
| **Scale up** | Traffic spike overwhelming capacity |
| **Feature flag off** | New feature causing issues — disable it |
| **Restart** | Process/container in bad state — restart with monitoring |
| **Failover** | Primary resource failed — switch to secondary |
| **Block traffic** | Malicious traffic — block at load balancer/WAF |
| **Manual fix** | Data corruption — fix specific records |

### Phase 4: Resolve (Root cause fix)
Once mitigated:
1. Identify the actual root cause (not just the symptom)
2. Implement a proper fix
3. Test in staging first if possible
4. Deploy fix with careful monitoring
5. Verify the fix resolved the issue
6. Remove any temporary mitigations

### Phase 5: Postmortem
Write within 48 hours. Use this template:

```markdown
# Incident Postmortem: [Title]

**Date**: YYYY-MM-DD
**Duration**: X hours Y minutes
**Severity**: SEV[1-4]
**Authors**: [names]

## Summary
[1-2 sentences: what happened, what was the impact]

## Timeline (UTC)
| Time | Event |
|------|-------|
| HH:MM | Issue detected by [monitoring/user report] |
| HH:MM | Engineer [name] acknowledged |
| HH:MM | Root cause identified |
| HH:MM | Mitigation applied |
| HH:MM | Full resolution confirmed |

## Root Cause
[What actually broke and why. Be specific — "the database connection pool exhausted because..." not "database issue"]

## Impact
- Users affected: [number/percentage]
- Revenue impact: [if applicable]
- Data loss: [yes/no, details]

## What Went Well
- [Things that helped speed up resolution]

## What Went Wrong
- [Things that slowed down resolution or made it worse]

## Action Items
| Priority | Action | Owner | Due Date |
|----------|--------|-------|----------|
| P0 | [Critical fix to prevent recurrence] | | |
| P1 | [Important improvement] | | |
| P2 | [Nice to have improvement] | | |

## Lessons Learned
[Key takeaways for the team]
```

## Communication Template

For stakeholder updates during incidents:

```
**[SEV{N}] {Service} — {Status}**

**Impact**: {Who is affected and how}
**Current status**: {What we know and what we're doing}
**Next update**: {When}
**ETA to resolution**: {Estimate or "investigating"}
```

## Key Principles
- **Blameless** — focus on systems and processes, not people
- **Communicate early and often** — silence is worse than "we don't know yet"
- **Mitigate first, root-cause later** — stop the pain, then investigate
- **Document everything** — timestamps, decisions, commands run
- **Don't make it worse** — avoid untested fixes in production during an incident
