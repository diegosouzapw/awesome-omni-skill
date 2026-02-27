---
name: repo-a-policy-selftest-gate
description: Enforce Repo A DDC policy and acceptance gates before PRs. Use when changing policy files, node runtime behavior, guardrail-sensitive config, or validation tooling that must satisfy AGENTS.md acceptance commands.
---

# Repo A DDC Policy and Selftest Gate

Use this skill to run required pre-PR validation in `<PRIVATE_REPO_A>`.

## Workflow

1. Review `AGENTS.md`, `INSTRUCTIONS.md`, and policy manifests before coding.
2. Run lint/type gates.
3. Run mesh-ready selftest against `config/device_policy.json`.
4. Run focused tests for changed modules.
5. Ensure policy/config changes remain documented and intentional.

## Scope Boundary

Use this skill for `<PRIVATE_REPO_A>` policy/selftest acceptance gates only.

Do not use this skill for:

1. `<PRIVATE_REPO_C>` policy/schema or trace/ranking contracts.
2. General boundary-governance checks in other repos.
3. Runtime bridge or hardware troubleshooting lanes.

## Required Commands

Run from `<PRIVATE_REPO_A>` root:

```bash
ruff check .
mypy repo_a_node
python -m repo_a_node --policy config/device_policy.json --selftest config/mesh_ready_selftest.yaml
pytest -q
```

## Policy Safety Notes

- Never hardcode secrets, URLs, or policy overrides.
- Do not relax canary/redundancy defaults without docs and policy updates.
- Keep governance boundaries aligned with `BOUNDARIES.md`.

## Reference

- `references/gate-checklist.md`

## Loopback

If this lane is unresolved, blocked, or ambiguous:

1. Capture current evidence and failure context.
2. Route back through `$skill-hub` for chain recalculation.
3. Resume only after the updated chain returns a deterministic next step.
