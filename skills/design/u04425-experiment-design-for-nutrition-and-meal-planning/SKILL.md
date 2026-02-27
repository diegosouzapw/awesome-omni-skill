---
name: u04425-experiment-design-for-nutrition-and-meal-planning
description: Operate the "Experiment design for nutrition and meal planning" capability in production for nutrition and meal planning workflows. Use when mission execution explicitly requires this capability and outcomes must be reproducible, policy-gated, and handoff-ready.
---

# Experiment design for nutrition and meal planning

## Why This Skill Exists
This skill hardens a generated capability for production execution so nutrition and meal planning workflows remain deterministic, auditable, and fail-closed under risk.

## When To Use
Use this skill only when the request explicitly needs `Experiment design for nutrition and meal planning` in nutrition and meal planning and a downstream consumer requires contract-bound artifacts.

## Step-by-Step Implementation Guide
1. Validate production trigger criteria: explicit capability request, approved source-tagged inputs, and named downstream consumer.
2. Enforce deterministic normalization workflow with pinned mapping/ruleset versions and stable serialization order.
3. Apply explicit determinism tolerance checks (score delta <= 0.005 absolute; identical input must produce zero artifact hash drift).
4. Execute fail-closed validation gates (schema, determinism, policy-risk) and block output on any failure.
5. Require explicit human sign-off token for high-risk runs before publication or downstream routing.
6. Emit handoff envelope with artifact paths, gate results, risk tier, and approval state for the next stage.

## Deterministic Workflow Constraints
- Replay score variance: <= 0.005 absolute per item.
- Artifact hash drift for identical replay: 0 allowed.
- Time-dependent fields allowed only in metadata and excluded from scoring.

## Validation Gates
1. **schema-gate** — all required fields present and schema-valid; otherwise block and return error bundle.
2. **determinism-gate** — replay output within tolerance; otherwise quarantine and escalate.
3. **policy-risk-gate** — policy and risk checks pass; otherwise block routing.
4. **approval-gate-high-risk** — if risk is high, require human sign-off token; otherwise fail closed.

## Handoff Contract
- Inputs: source-tagged signals, claims, evidence, confidence traces, run context.
- Outputs: deterministic artifact, scorecard, and handoff envelope with approval metadata.
- Routing rule: forward only when every gate passes; high-risk requires explicit sign-off token.

## Immediate Hardening Additions
- Fixture: `fixtures/minimal-valid.json`
- Regression case: `tests/regression-case.md`
- Machine-readable summary: `hardening-summary.json`
