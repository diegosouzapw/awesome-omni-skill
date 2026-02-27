---
name: Feature Flag Strategy
description: Design and manage feature flag lifecycles for safe, gradual rollouts
category: deployment
version: 1.0.0
triggers:
  - new-feature-rollout
  - flag-lifecycle
  - gradual-release
globs: "**/flags/**,**/features/**"
---

# Feature Flag Strategy Skill

Design and manage feature flag lifecycles for safe, gradual rollouts.

## Trigger Conditions
- New feature flag is created
- Existing flag is older than 90 days
- User invokes with "feature flag strategy" or "rollout plan"

## Input Contract
- **Required:** Feature being flagged
- **Required:** Rollout target and timeline
- **Optional:** Risk assessment, rollback criteria

## Output Contract
- Flag configuration with rollout stages
- Monitoring criteria for each stage
- Cleanup plan with ticket and deadline
- A/B test configuration (if applicable)

## Tool Permissions
- **Read:** Flag configs, feature code, monitoring dashboards
- **Write:** Flag configurations, cleanup tickets
- **Search:** Codebase for flag usage

## Execution Steps
1. Define the flag with name, owner, and expiration date
2. Design rollout stages (internal → canary 1% → 10% → 50% → 100%)
3. Define monitoring criteria and auto-halt thresholds
4. Configure flag evaluation (centralized, not scattered if/else)
5. Create cleanup ticket with 90-day deadline
6. Document in flag inventory

## Success Criteria
- Flag has owner, expiration, and cleanup ticket
- Rollout stages defined with monitoring criteria
- Auto-halt configured for error rate/latency anomalies
- Both flag-on and flag-off paths tested

## Escalation Rules
- Escalate if flag has been active >90 days without cleanup
- Escalate if rollout shows >5% error rate increase
- Escalate if flag cleanup has cross-team dependencies

## Example Invocations

**Input:** "Create a feature flag strategy for the new checkout flow"

**Output:** Flag: ENABLE_NEW_CHECKOUT, owner: payments-team, expiration: 2026-05-01. Rollout: internal (week 1) → 5% canary (week 2, monitor error rate/conversion) → 25% (week 3) → 100% (week 4). Auto-halt: >2% error rate increase or >10% conversion drop. Cleanup PR template created.
