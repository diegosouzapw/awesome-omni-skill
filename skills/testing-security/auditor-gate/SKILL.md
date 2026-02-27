---
name: auditor-gate
description: Apply final governance and release-gate checks to a judged change set by reading `handoff.json`, `verdict.json`, optional eval evidence, and emitting machine-readable `audit.json` with `gate` status. Use when implementation already has a judge verdict and a separate auditor must decide landability (`pass`, `fail`, or `needs-human`) without modifying source files.
---

# Auditor Gate

Run final acceptance gating as a separate role after judge evaluation.

## Role Boundaries

- Audit process and risk posture.
- Validate traceability and policy signals.
- Produce `audit.json` only.
- Do not implement fixes and do not re-grade functional correctness from scratch.

## Required Inputs

- `task_id`
- `handoff.json`
- `verdict.json`
- output path for `audit.json`

Optional:
- `eval-results.json`
- explicit audit commands (policy/security/license/quality checks)

## Workflow

### 1) Run audit evidence collection

Run:

```bash
python3 <path-to-skill>/scripts/run_audit.py \
  --repo <repo-root> \
  --handoff <artifact-path>/handoff.json \
  --verdict <artifact-path>/verdict.json \
  --output <artifact-path>/audit-results.json \
  --command "python3 -m pip list --format=freeze"
```

This collects artifact-integrity checks and command outcomes.
It also checks task and requirement traceability between `handoff.json` and `verdict.json`.

### 2) Write gate artifact

Run:

```bash
python3 <path-to-skill>/scripts/write_audit.py \
  --task-id <task-id> \
  --audit-results <artifact-path>/audit-results.json \
  --output <artifact-path>/audit.json \
  --gate fail \
  --finding "policy::Dependency policy review missing" \
  --required-action "Add dependency review evidence" \
  --risk-level high
```

### 3) Validate gate artifact

Run:

```bash
python3 <path-to-skill>/scripts/validate_audit.py \
  --input <artifact-path>/audit.json
```

## Decision Rules

- `pass`:
  - no critical audit failures
  - traceability and policy checks pass or are explicitly accepted
- `fail`:
  - policy, traceability, or release constraints are violated
- `needs-human`:
  - missing evidence or ambiguous policy interpretation blocks deterministic decision

## Output Rules

- Always produce `audit.json`.
- Keep findings evidence-based and implementation-neutral.
- Provide concrete `required_actions` for `fail` and `needs-human`.
- Report requirement-coverage traceability explicitly in `audit.json`.

## Resources

### scripts/
- `scripts/run_audit.py`: collect artifact and command-level audit evidence.
- `scripts/write_audit.py`: synthesize `audit.json` gate decision.
- `scripts/validate_audit.py`: validate audit artifact schema.

### references/
- `references/artifact-contract.md`: canonical `audit.json` structure.
- `references/gate-rubric.md`: practical criteria for `pass`/`fail`/`needs-human`.
