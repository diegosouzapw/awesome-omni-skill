---
name: Chaos Engineering
description: Design and execute controlled failure experiments to validate system resilience
category: reliability
version: 1.0.0
triggers:
  - pre-release-validation
  - post-deploy-verification
  - resilience-test
globs: "**/chaos/**,**/resilience/**"
---

# Chaos Engineering Skill

Design and execute controlled failure experiments to validate system resilience.

## Trigger Conditions
- Pre-release resilience validation needed
- Post-deploy verification of fault tolerance
- User invokes with "chaos experiment" or "resilience test"

## Input Contract
- **Required:** System under test
- **Required:** Steady-state hypothesis (measurable)
- **Optional:** Blast radius constraints, failure types to inject

## Output Contract
- Experiment definition with hypothesis
- Results report with pass/fail
- Findings and remediation recommendations
- Updated resilience scorecard

## Tool Permissions
- **Read:** Service configs, circuit breaker configs, monitoring dashboards
- **Write:** Experiment logs, findings reports
- **Execute:** Failure injection tools (network, compute, storage)

## Execution Steps
1. Define steady-state hypothesis with measurable metrics
2. Select failure injection type (network, pod kill, CPU, disk, dependency)
3. Constrain blast radius (start small: single pod, single AZ)
4. Execute experiment while monitoring steady state
5. Observe and record system behavior
6. Compare actual behavior against hypothesis
7. Document findings and remediation

## Success Criteria
- Hypothesis clearly defined before experiment
- Blast radius contained as planned
- Monitoring remained functional during experiment
- Findings documented with severity and remediation

## Escalation Rules
- Escalate if experiment causes unexpected customer impact
- Escalate if monitoring fails during the experiment
- Escalate if recovery takes longer than MTTR target

## Example Invocations

**Input:** "Test what happens when the Redis cache becomes unavailable"

**Output:** Hypothesis: API latency stays <500ms p99 with cache miss fallback to DB. Experiment: kill Redis pod. Result: FAIL — latency spiked to 3.2s, circuit breaker did not trip (misconfigured threshold). Remediation: lower circuit breaker threshold from 50% to 20% error rate, add cache stampede protection.
