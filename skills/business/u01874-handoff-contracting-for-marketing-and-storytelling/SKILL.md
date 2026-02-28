---
name: u01874-handoff-contracting-for-marketing-and-storytelling
description: Operate the "Handoff Contracting for marketing and storytelling" capability in production for marketing and storytelling workflows. Use when mission execution explicitly requires this capability and outcomes must be reproducible, policy-gated, and handoff-ready.
---

# Handoff Contracting for marketing and storytelling

## Why This Skill Exists
Use handoff contracting in marketing and storytelling with emphasis on throughput, reliability, leverage, and execution speed.

## Step-by-Step Implementation Guide
1. Define measurable outcomes for Handoff Contracting for marketing and storytelling, including baseline and target metrics for marketing and storytelling.
2. Specify structured inputs/outputs for handoff contracting and validate schema contract edge cases.
3. Implement the core handoff contracting logic with deterministic scoring and reproducible execution traces.
4. Integrate orchestration policy, routing, approval gates, retries, and rollback for autonomous execution.
5. Run unit, integration, simulation, and regression suites for Handoff Contracting for marketing and storytelling under hyper-productive autonomy conditions.
6. Roll out behind a feature flag, monitor telemetry, and refine thresholds using observed operational outcomes.

## Metadata
- **Skill ID:** `1874`
- **Skill Name:** `u01874-handoff-contracting-for-marketing-and-storytelling`
- **Domain:** `marketing and storytelling`
- **Domain Slug:** `marketing-and-storytelling`
- **Archetype:** `generalist-engine`
- **Core Method:** `handoff contracting`
- **Primary Artifact:** `handoff-contracting-artifact-marketing-and-storytelling`
- **Routing Tag:** `marketing-and-storytelling:generalist-engine`
- **Feature Flag:** `skill_01874_handoff-contracting`
- **Release Cycles:** `2`

## Allowed Tools
- `read`, `write`, `edit` for contract maintenance, runbook updates, and handoff documentation.
- `exec`, `process` for deterministic execution, validation suites, and regression checks.
- `web_search`, `web_fetch` only when fresh external evidence is required for claims/evidence inputs.
- Use messaging or publishing tools only after policy approval gates are satisfied.

## Inputs (formatted)
| name | type | required | source |
|---|---|---|---|
| (none) | - | - | - |

## Outputs (formatted)
| name | type | guaranteed | consumer |
|---|---|---|---|
| (none) | - | - | - |

## Guidelines
1. Validate required inputs before execution and reject non-conforming payloads early.
2. Run `handoff contracting` with deterministic settings and trace capture enabled.
3. Produce `handoff-contracting-artifact-marketing-and-storytelling` outputs in machine-readable form for orchestrator/operator use.
4. Keep routing aligned with `marketing-and-storytelling:generalist-engine` and include approval context.
5. Tune thresholds incrementally based on observed KPI drift and incident learnings.

## Musts
- Enforce approval gates: `policy-constraint-check`, `human-approval-router`.
- Apply retry policy: maxAttempts=`4`, baseDelayMs=`750`, backoff=`exponential`.
- Run validation suites before release: `unit`, `integration`, `simulation`, `regression-baseline`.
- Fail closed when validation gates fail and execute rollback strategy `rollback-to-last-stable-baseline`.
- Preserve reproducible evidence artifacts for audits and downstream handoff.

## Targets (day/week/month operating cadence)
- **Day:** Validate new upstream signals, execute deterministic run, and hand off outputs for active decisions.
- **Week:** Review KPI focus (`cycle time reduction`, `throughput gain`, `automation leverage in marketing and storytelling`), failure trends, and approval/retry performance.
- **Month:** Re-baseline deterministic expectations, confirm policy alignment, and refresh feature-flag/rollout posture.

## Common Actions
1. **Intake Check:** Confirm all required signals are present and schema-valid.
2. **Core Execution:** Run the capability pipeline and generate report + scorecard artifacts.
3. **Gate Review:** Evaluate validation and approval gates before publish-level handoff.
4. **Recovery:** Retry transient failures, then rollback to stable baseline on persistent errors.
5. **Handoff:** Send artifacts with risk/confidence metadata and downstream routing hints.

## External Tool Calls Needed
- None required by default.
- If external systems are introduced for a run, record the dependency, timeout budget, and retry behavior in execution notes.

## Validation & Handoff
### Validation Gates
- (none declared)

### Validation Suites
- `unit`
- `integration`
- `simulation`
- `regression-baseline`

### Failure Handling
- (none declared)

### Handoff Contract
- **Produces:** (none declared)
- **Consumes:** (none declared)
- **Downstream Hint:** (none declared)
