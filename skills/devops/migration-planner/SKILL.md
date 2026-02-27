---
name: Migration Planner
description: Plan safe, zero-downtime migrations for schemas, services, and infrastructure
category: database
version: 1.0.0
triggers:
  - schema-migration
  - service-migration
  - infrastructure-migration
globs: "**/migrations/**,**/infrastructure/**"
---

# Migration Planner Skill

Plan safe, zero-downtime migrations for schemas, services, and infrastructure.

## Trigger Conditions
- Schema migration files are created or modified
- A service migration or infrastructure change is planned
- User invokes with "plan migration" or "migration strategy"

## Input Contract
- **Required:** Current state (schema, service, infrastructure)
- **Required:** Target state (desired end state)
- **Optional:** Constraints (downtime window, data volume, timeline)

## Output Contract
- Phased migration plan with rollback steps
- Risk assessment per phase
- Data validation checkpoints
- Estimated duration and resource requirements

## Tool Permissions
- **Read:** Schema files, migration history, service configs
- **Write:** Migration plan documents, migration scripts
- **Execute:** Schema comparison tools

## Execution Steps
1. Analyze current and target state; identify delta
2. Decompose into phases using expand-and-contract pattern
3. Identify backward-compatibility requirements per phase
4. Design rollback procedure for each phase
5. Estimate data volume and migration duration
6. Create validation checkpoints between phases
7. Document the plan with risk scores

## Success Criteria
- Every phase is independently rollback-able
- No phase requires downtime
- Data validation checkpoints defined between phases
- Risk assessment includes worst-case scenarios

## Escalation Rules
- Escalate if migration affects >100M rows
- Escalate if cross-service coordination is required
- Escalate if estimated lock time exceeds 30 seconds

## Example Invocations

**Input:** "Rename the users.email column to users.email_address"

**Output:** 3-phase plan: Phase 1 (add email_address column, backfill, dual-write), Phase 2 (switch readers to email_address, verify), Phase 3 (drop email column). Estimated: 2 deploys, 45min backfill on 5M rows. Rollback at each phase documented.
