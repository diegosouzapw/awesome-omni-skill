---
name: ADR (Architecture Decision Record)
description: Create and maintain Architecture Decision Records following a consistent template
category: architecture
version: 1.0.0
triggers:
  - architecture-decision
  - adr-supersede
  - document-decision
globs: "**/docs/adr/**,**/architecture/**"
---

# ADR (Architecture Decision Record) Skill

Create and maintain Architecture Decision Records following a consistent template.

## Trigger Conditions
- A new architecture decision needs to be documented
- An existing ADR needs to be superseded or updated
- User invokes with "create ADR" or "document decision"

## Input Contract
- **Required:** Decision context (what problem are we solving?)
- **Required:** Options considered (at least 2 alternatives)
- **Optional:** Constraints, evaluation criteria, stakeholders

## Output Contract
- ADR document in markdown format with: Status, Date, Context, Decision, Consequences, Alternatives Considered
- Numbered sequentially (ADR-NNN) in the `docs/adr/` directory
- Links to related ADRs (supersedes, relates-to)

## Tool Permissions
- **Read:** All project files, existing ADRs, architecture docs
- **Write:** `docs/adr/` directory
- **Search:** Codebase for implementation evidence

## Execution Steps
1. Gather the decision context from user or triggering event
2. Search existing ADRs for related decisions
3. Determine the next ADR number
4. Draft the ADR using the standard template
5. Link to related ADRs and relevant code/config
6. Write the ADR file to `docs/adr/ADR-NNN-<title>.md`

## Success Criteria
- ADR has all required sections filled in
- Status is set (Proposed, Accepted, Deprecated, Superseded)
- Consequences section includes both positive and negative impacts
- File is written to the correct location with proper naming

## Escalation Rules
- Escalate if the decision has >$1K/month cost implications
- Escalate if the decision affects regulatory compliance
- Escalate if no clear winner among alternatives

## Example Invocations

**Input:** "We need to decide between PostgreSQL and DynamoDB for the order service"

**Output:** ADR-005-order-service-database.md with context (high write volume, need transactions), decision (PostgreSQL for ACID guarantees), consequences (operational overhead, need connection pooling), alternatives (DynamoDB rejected due to transaction limitations).
