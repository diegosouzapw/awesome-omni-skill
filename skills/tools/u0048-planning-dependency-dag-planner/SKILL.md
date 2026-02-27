---
name: u0048-planning-dependency-dag-planner
description: Build and operate the "Planning Dependency DAG Planner" capability for Strategic Planning and Decomposition. Use when outcomes in this capability family are required for production execution.
---

# Planning Dependency DAG Planner

## Why This Skill Exists
We need this skill because large goals fail when decomposition is inconsistent or incomplete. This specific skill prevents sequencing errors and hidden blockers.

## When To Use
Use this skill when you need "Planning Dependency DAG Planner" outcomes for the Strategic Planning and Decomposition domain with measurable, production-facing outputs.

## Step-by-Step Implementation Guide
1. Define the scope and success metrics for `Planning Dependency DAG Planner`, including at least three measurable KPIs tied to execution stalls and hidden dependency failures.
2. Design and version the input/output contract for goals, dependencies, milestones, and constraints, then add schema validation and failure-mode handling.
3. Implement the core capability using dependency graph compilation, and produce validated workflow DAGs with deterministic scoring.
4. Integrate the skill into swarm orchestration: task routing, approval gates, retry strategy, and rollback controls.
5. Add unit, integration, and simulation tests that explicitly cover execution stalls and hidden dependency failures, then run regression baselines.
6. Deploy behind a feature flag, monitor telemetry/alerts for two release cycles, and iterate thresholds based on observed outcomes.

## Deterministic Workflow Notes
- Core method: dependency graph compilation
- Archetype: planning-router
- Routing tag: strategic-planning-and-decomposition:planning-router

## Input Contract
- `goals` (signal, source=upstream, required=true)
- `dependencies` (signal, source=upstream, required=true)
- `milestones` (signal, source=upstream, required=true)
- `constraints` (signal, source=upstream, required=true)
- `claims` (signal, source=upstream, required=true)
- `evidence` (signal, source=upstream, required=true)
- `confidence traces` (signal, source=upstream, required=true)

## Output Contract
- `validated_workflow_dags_report` (structured-report, consumer=orchestrator, guaranteed=true)
- `validated_workflow_dags_scorecard` (scorecard, consumer=operator, guaranteed=true)

## Validation Gates
1. **schema-contract-check** — All required input signals present and schema-valid (on fail: quarantine)
2. **determinism-check** — Repeated run on same inputs yields stable scoring and artifacts within tolerance <= 1% (on fail: escalate)
3. **policy-approval-check** — Approval gates satisfied before publish-level outputs; high-risk changes require human sign-off (on fail: block)

## Failure Handling
- `E_INPUT_SCHEMA`: Missing or malformed required signals → Reject payload, emit validation error, request corrected payload
- `E_NON_DETERMINISM`: Determinism delta exceeds allowed threshold → Freeze output, escalate to human approval router
- `E_DEPENDENCY_TIMEOUT`: Downstream or external dependency timeout → Apply retry policy then rollback to last stable baseline
- Rollback strategy: rollback-to-last-stable-baseline

## Handoff Contract
- Produces: Planning Dependency DAG Planner normalized artifacts; execution scorecard; risk posture
- Consumes: goals; dependencies; milestones; constraints; claims; evidence; confidence traces
- Downstream routing hint: Route next to strategic-planning-and-decomposition:planning-router consumers with approval-gate context

## Required Deliverables
- Capability contract: input schema, deterministic scoring, output schema, and failure modes.
- Orchestration integration: task routing, approval gates, retries, and rollback controls.
- Validation evidence: unit tests, integration tests, simulation checks, and rollout telemetry.

## Immediate Hardening Additions
- Add golden test fixtures for at least 5 representative payloads.
- Add regression test covering the highest-risk failure mode for this capability.
- Emit machine-readable run summary (`status`, `risk_score`, `confidence`, `next_handoff`).
- Fail closed on schema or policy gate violations; never emit publish-level output on gate failure.

## Production Trigger Clarity
- Use only when this capability produces production-facing outcomes with measurable acceptance criteria.
- Do not invoke for exploratory brainstorming or unrelated domains; route those requests to the correct capability family.

## Deterministic Tolerances
- Repeated runs on identical inputs must remain within **<=1% output variance** for scoring fields and preserve schema-identical artifact shape.
- Any variance beyond tolerance is a hard failure and must trigger escalation.

## Fail-Closed Validation Gates
1. Schema validity gate (required inputs present and valid).
2. Determinism gate (variance within tolerance).
3. Policy/approval gate (required approvals satisfied).

If any gate fails: **block output publication and fail closed**.

## High-Risk Human Sign-Off
- Any high-risk change, policy-impacting output, or publish-level action requires explicit human sign-off before release.
- Missing sign-off is a blocking condition.

## Explicit Handoff Contract
- **Produces:** normalized artifacts, decision scorecard, risk/confidence metadata.
- **Consumes:** validated upstream inputs for this capability.
- **Next hop:** route only to declared downstream consumers with gate/approval context attached.
