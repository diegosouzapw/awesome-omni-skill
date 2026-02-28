---
name: Capacity Planning
description: Forecast resource needs and design scaling strategies based on growth projections
category: operations
version: 1.0.0
triggers:
  - utilization-threshold
  - quarterly-review
  - scaling-strategy
globs: "**/infrastructure/**,**/scaling/**"
---

# Capacity Planning Skill

Forecast resource needs and design scaling strategies based on growth projections.

## Trigger Conditions
- Traffic spikes or utilization exceeds 75% threshold
- Quarterly capacity review cycle
- User invokes with "capacity plan" or "scaling strategy"

## Input Contract
- **Required:** Current resource utilization metrics
- **Required:** Growth forecast (user acquisition, traffic patterns)
- **Optional:** Budget constraints, SLA requirements

## Output Contract
- Capacity model with projections
- Scaling recommendations (horizontal/vertical, auto/manual)
- Cost forecast per growth scenario
- Headroom analysis for failover scenarios

## Tool Permissions
- **Read:** Metrics, infrastructure configs, billing data
- **Write:** Capacity planning documents
- **Search:** Historical usage patterns

## Execution Steps
1. Collect current utilization across compute, storage, network
2. Model demand from business metrics, not just historical extrapolation
3. Define target utilization bands (60-75%)
4. Calculate headroom for failover (lose 1 of N replicas)
5. Project costs for 3, 6, 12 month horizons
6. Recommend scaling triggers and strategies
7. Document the capacity model

## Success Criteria
- Utilization targets defined per resource type
- Cost projections for multiple growth scenarios
- Scaling triggers calibrated for scale-up latency
- Failover headroom verified

## Escalation Rules
- Escalate if projected costs exceed budget by >20%
- Escalate if current utilization exceeds 85%
- Escalate if no scaling strategy can meet SLA targets

## Example Invocations

**Input:** "Plan capacity for Black Friday (expected 5x normal traffic)"

**Output:** Current: 3 pods, 45% CPU avg. For 5x: need 12 pods minimum (15 recommended for headroom). Auto-scaler trigger at 60% CPU. Pre-scale 2 hours before peak. Estimated additional cost: $2,400 for 48h burst. DB read replicas: add 2 in us-east, 1 in eu-west.
