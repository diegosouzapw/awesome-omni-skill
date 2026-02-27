---
name: Cost Analysis
description: Analyze infrastructure and operational costs with optimization recommendations
category: operations
version: 1.0.0
triggers:
  - monthly-review
  - cost-anomaly
  - budget-threshold
globs: "**/infrastructure/**,**/*.tf"
---

# Cost Analysis Skill

Analyze infrastructure and operational costs with optimization recommendations.

## Trigger Conditions
- Monthly/quarterly cost review cycle
- Budget threshold breach alert
- New resource provisioning request
- User invokes with "cost analysis" or "optimize spending"

## Input Contract
- **Required:** Resource inventory or billing data
- **Required:** Time period for analysis
- **Optional:** Budget targets, comparison baselines

## Output Contract
- Cost breakdown by service, team, and resource type
- Cost-per-transaction trending
- Optimization recommendations with savings estimates
- Reserved vs. on-demand vs. spot mix recommendation

## Tool Permissions
- **Read:** Billing data, resource configs, usage metrics
- **Write:** Cost analysis reports
- **Search:** Resource tags and attribution data

## Execution Steps
1. Collect billing data for the analysis period
2. Attribute costs by service and team using resource tags
3. Identify unattributed costs
4. Calculate cost-per-transaction trends
5. Identify top cost drivers and anomalies
6. Generate optimization recommendations (rightsizing, reservations, spot)
7. Estimate savings per recommendation

## Success Criteria
- 100% of costs attributed to service/team
- Cost anomalies flagged with explanation
- Optimization recommendations include estimated savings
- Cost-per-transaction trend reported

## Escalation Rules
- Escalate if daily spend exceeds baseline by >20%
- Escalate if unattributed costs exceed 10% of total
- Escalate if optimization requires architectural changes

## Example Invocations

**Input:** "Analyze our AWS costs for January and find savings"

**Output:** Total: $45,200. Top 3: RDS ($18K — oversized instances, recommend r6g.xlarge→r6g.large, save $4K/mo), EKS ($12K — 40% idle capacity, recommend cluster autoscaler tuning), S3 ($8K — no lifecycle policy, archive to Glacier after 30d, save $3K/mo). Total potential savings: $7K/month.
