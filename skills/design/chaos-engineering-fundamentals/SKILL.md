---
name: chaos-engineering-fundamentals
description: Use when implementing chaos engineering, designing fault injection experiments, or building resilience testing practices. Covers chaos principles and experiment design.
allowed-tools: Read, Glob, Grep
---

# Chaos Engineering Fundamentals

Principles and practices for chaos engineering - proactively discovering system weaknesses through controlled experiments.

## When to Use This Skill

- Implementing chaos engineering practices
- Designing fault injection experiments
- Building confidence in system resilience
- Discovering hidden failure modes
- Validating disaster recovery

## What is Chaos Engineering?

```text
Chaos Engineering = Proactive resilience testing

Traditional testing: "Does it work when everything is right?"
Chaos engineering: "Does it work when things go wrong?"

Principle:
Build confidence in the system's ability to withstand
turbulent conditions in production.

Not about breaking things randomly.
About controlled experiments to learn.
```

## The Chaos Engineering Loop

```text
┌─────────────────────────────────────────────────────────┐
│                CHAOS ENGINEERING LOOP                    │
│                                                          │
│   ┌─────────┐     ┌─────────┐     ┌─────────┐          │
│   │ Define  │────►│ Inject  │────►│ Observe │          │
│   │ Steady  │     │ Chaos   │     │ Results │          │
│   │ State   │     │         │     │         │          │
│   └─────────┘     └─────────┘     └────┬────┘          │
│        ▲                               │                │
│        │                               │                │
│        │          ┌─────────┐          │                │
│        └──────────│ Improve │◄─────────┘                │
│                   │ System  │                           │
│                   └─────────┘                           │
└─────────────────────────────────────────────────────────┘
```

## Core Principles

### 1. Build Hypothesis Around Steady State

```text
Steady State = Normal system behavior

Define measurable indicators:
- Request success rate: 99.9%
- Latency p99: < 200ms
- Orders processed/minute: > 100
- User sessions active: > 10,000

Hypothesis format:
"When [fault condition] occurs, the system will
maintain [steady state metrics] within [acceptable bounds]"

Example:
"When one database replica fails, request success rate
will remain above 99.5% and latency below 500ms"
```

### 2. Vary Real-World Events

```text
Inject realistic failures:

Infrastructure:
- Server crash
- Network partition
- Disk full
- CPU exhaustion
- Clock skew

Application:
- Service unavailable
- Slow responses
- Corrupted data
- Certificate expiry
- Resource exhaustion

Dependencies:
- Database failure
- Cache unavailable
- Third-party API down
- Message queue backup
```

### 3. Run Experiments in Production

```text
Why production?
- Real traffic patterns
- Real infrastructure
- Real dependencies
- Real monitoring

Start safe:
1. Begin in non-production
2. Graduate to canary
3. Progress to production
4. Expand blast radius gradually

Safety nets:
- Kill switch ready
- Rollback plan
- Limited blast radius
- Monitoring in place
```

### 4. Automate Experiments to Run Continuously

```text
One-time experiments find one-time bugs.
Continuous experiments catch regressions.

Automation goals:
- Run experiments regularly
- Integrate with CI/CD
- Catch new failure modes
- Validate changes

Example schedule:
- Critical paths: Daily
- Core services: Weekly
- Full system: Monthly
```

### 5. Minimize Blast Radius

```text
Control experiment impact:

Scope limitations:
- Single instance
- Percentage of traffic
- Specific region
- Test accounts only

Duration limits:
- Seconds to minutes
- Automatic termination
- Scheduled windows

Abort conditions:
- Error rate exceeds threshold
- Customer impact detected
- Manual kill switch
```

## Experiment Design

### Experiment Structure

```text
Experiment: [Name]
Date: [When]
Team: [Who]

## Hypothesis
When [fault is injected], the system will [expected behavior]
because [reasoning].

## Steady State Metrics
- [Metric 1]: [Expected value]
- [Metric 2]: [Expected value]

## Experiment Details
Fault Type: [What we're injecting]
Target: [Where we're injecting]
Magnitude: [How severe]
Duration: [How long]

## Blast Radius
- Affected services: [List]
- Affected users: [Percentage/count]
- Region/zone: [Scope]

## Abort Conditions
- [Condition 1] → Abort
- [Condition 2] → Abort

## Rollback Plan
1. [Step 1]
2. [Step 2]

## Results
Hypothesis: [Confirmed/Falsified]
Observations: [What we saw]
Action Items: [What to fix]
```

### Common Experiment Types

```text
1. Service Failure
   └── Kill instances, return errors

2. Network Failures
   └── Latency injection, packet loss, partitions

3. Resource Exhaustion
   └── CPU stress, memory pressure, disk full

4. Dependency Failures
   └── Database down, cache miss, API timeout

5. State Corruption
   └── Clock skew, data inconsistency

6. Traffic Surge
   └── Sudden load increase
```

## Fault Injection Patterns

### Infrastructure Faults

```text
Instance termination:
- Kill random instances
- Verify auto-scaling/recovery
- Netflix Chaos Monkey style

Zone/region failure:
- Simulate full zone outage
- Test failover to other zones
- Verify data consistency

Network partition:
- Split brain scenarios
- Cross-region communication failure
- Consensus algorithm behavior
```

### Application Faults

```text
Latency injection:
- Add artificial delay
- Test timeout handling
- Verify circuit breakers

Error injection:
- Return 500 errors
- Throw exceptions
- Test error handling paths

Resource leaks:
- Memory leaks
- Connection pool exhaustion
- File handle exhaustion
```

### Dependency Faults

```text
Database failures:
- Primary failover
- Replica lag
- Connection pool exhaustion

Cache failures:
- Cache miss scenarios
- Cache cluster failure
- Stampede protection

External API failures:
- Timeout
- Rate limiting
- Malformed responses
```

## Chaos Engineering Tools

### Open Source Tools

```text
Chaos Monkey (Netflix)
- Random instance termination
- AWS focused
- Part of Simian Army

Gremlin
- Comprehensive chaos platform
- Multiple attack types
- Enterprise features

Litmus
- Kubernetes native
- ChaosHub experiment library
- GitOps friendly

Chaos Mesh
- Kubernetes native
- Various fault types
- Dashboard included

Pumba
- Docker chaos testing
- Container-level faults
- CI/CD integration
```

### Cloud Provider Tools

```text
AWS:
- Fault Injection Simulator
- Native integration

Azure:
- Chaos Studio
- Azure-native experiments

GCP:
- No native tool (use Gremlin/Litmus)
```

## Implementation Strategy

### Maturity Model

```text
Level 0: Ad-hoc
- Manual testing
- No chaos practice
- Reactive to failures

Level 1: Beginning
- First experiments
- Non-production only
- Manual execution

Level 2: Intermediate
- Regular experiments
- Production experiments
- Some automation

Level 3: Advanced
- Continuous chaos
- Automated experiments
- Broad coverage

Level 4: Expert
- Chaos as code
- Integrated in CI/CD
- GameDays regular
```

### Getting Started

```text
Week 1-2: Foundation
- Identify critical paths
- Define steady state metrics
- Set up monitoring

Week 3-4: First Experiments
- Start with known failures
- Run in non-production
- Document learnings

Month 2: Expand
- Add more experiment types
- Move to production (carefully)
- Automate basic experiments

Month 3+: Mature
- Regular GameDays
- Continuous experiments
- Integrate with CI/CD
```

## GameDays

### What is a GameDay?

```text
GameDay = Planned chaos exercise

Like a fire drill for systems:
- Scheduled in advance
- Multiple failure scenarios
- Practice incident response
- Learn and improve
```

### GameDay Structure

```text
Before:
- Define objectives
- Plan scenarios
- Notify stakeholders
- Prepare runbooks
- Set up monitoring

During:
- Run scenarios
- Observe system behavior
- Practice incident response
- Document findings

After:
- Debrief meeting
- Document learnings
- Create action items
- Plan next GameDay
```

### GameDay Scenarios

```text
Scenario categories:

1. Infrastructure
   - Region failure
   - Network partition
   - Scaling limits

2. Application
   - Service outage
   - Deployment failure
   - Configuration error

3. Data
   - Database corruption
   - Backup restoration
   - Data center switch

4. Security
   - Credential rotation
   - Certificate expiry
   - Access revocation
```

## Safety and Guardrails

### Experiment Safety

```text
Before running chaos:

1. Have a hypothesis
   Know what you're testing

2. Limit blast radius
   Start small, expand gradually

3. Have abort conditions
   Automatic stops

4. Have rollback plan
   Know how to undo

5. Monitor everything
   Can't learn what you can't see

6. Communicate
   Team knows experiment is running
```

### Kill Switch

```text
Every experiment needs:

Manual kill switch:
- Instant termination
- Accessible to multiple people
- Tested before experiment

Automatic abort:
- Error rate threshold
- Latency threshold
- Customer impact detection

Notification:
- Alert when abort triggered
- Log reason for abort
```

## Measuring Success

```text
Chaos engineering success metrics:

1. Experiments run
   Are we doing chaos regularly?

2. Issues discovered
   Are we finding problems?

3. MTTR improvement
   Are we recovering faster?

4. Incident prevention
   Did chaos prevent production incidents?

5. Confidence level
   Does team trust the system more?
```

## Best Practices

```text
1. Start small
   Begin with simple experiments

2. Hypothesis first
   Know what you're testing

3. Automate gradually
   Manual first, then automate

4. Production eventually
   That's where real chaos lives

5. Blameless culture
   Findings are learnings, not failures

6. Regular GameDays
   Practice makes prepared

7. Share learnings
   Spread knowledge across teams
```

## Related Skills

- `resilience-patterns` - Building resilient systems
- `gameday-planning` - Detailed GameDay planning
- `incident-response` - Handling discovered issues
