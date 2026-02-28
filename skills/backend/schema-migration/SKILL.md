---
name: Schema Migration
description: Create safe, zero-downtime schema migrations with rollback procedures
category: database
version: 1.0.0
triggers:
  - schema-change
  - migration-file-created
  - database-evolution
globs: "**/migrations/**,**/models/**,**/*.sql"
---

# Schema Migration Skill

Create safe, zero-downtime schema migrations with rollback procedures.

## Trigger Conditions
- Migration file created or modified
- ORM model changes detected
- User invokes with "create migration" or "schema change"

## Input Contract
- **Required:** Current schema state
- **Required:** Desired schema change
- **Optional:** Data volume estimate, downtime tolerance

## Output Contract
- Migration SQL (up and down)
- Backward-compatibility assessment
- Lock time estimate for large tables
- Data backfill strategy (if applicable)

## Tool Permissions
- **Read:** Schema files, existing migrations, ORM models
- **Write:** Migration files
- **Execute:** Schema comparison tools, EXPLAIN ANALYZE

## Execution Steps
1. Analyze desired change against current schema
2. Determine if change is backward-compatible with N-1 app version
3. Apply expand-and-contract pattern if needed
4. Write UP migration with safety guards (IF NOT EXISTS, CONCURRENTLY)
5. Write DOWN migration (rollback)
6. Estimate lock time for table size
7. Design backfill strategy for data changes
8. Test both UP and DOWN against staging data

## Success Criteria
- Migration is backward-compatible with current app version
- Both UP and DOWN migrations tested
- Lock time estimated and acceptable
- Migration is idempotent (safe to re-run)

## Escalation Rules
- Escalate if lock time estimate >30 seconds
- Escalate if migration requires data backfill on >10M rows
- Escalate if migration affects multiple services

## Example Invocations

**Input:** "Add a non-null 'organization_id' column to the users table (5M rows)"

**Output:** 3-phase migration: Phase 1 (ADD COLUMN organization_id nullable, backfill from org_memberships, ~2min), Phase 2 (SET NOT NULL after backfill verified), Phase 3 (ADD INDEX CONCURRENTLY on organization_id, ~45s). Rollback: DROP COLUMN in each phase. Total estimated time: 4 minutes with zero downtime.
