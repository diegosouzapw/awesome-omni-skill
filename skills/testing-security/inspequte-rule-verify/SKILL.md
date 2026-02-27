---
name: inspequte-rule-verify
description: Perform isolated, file-based verification of an inspequte rule change using verify-input/. Use when producing a go/no-go verification report from spec.md, patch/diff, and report files without reading plan.md or chat logs.
---

# inspequte rule verify

## Required Input Directory
- `verify-input/`

Required files:
- `verify-input/spec.md`
- `verify-input/diff.patch` (or equivalent change set)
- `verify-input/reports/*` (test/build/audit evidence)

Optional but recommended:
- `verify-input/changes/*`
- `verify-input/changed-files.txt`

## Isolation Policy
- Verify must only use `spec.md`, change set (`diff.patch`), and report files.
- Do not read `src/rules/<rule-id>/plan.md`.
- Do not use implementation discussion logs, chat context, or author intent.
- If required input files are missing, fail with a clear blocked report.

## Output
- Print a verification report with these sections:
1. `## Spec compliance findings`
2. `## FP/noise risks`
3. `## Determinism/stability risks`
4. `## Performance and regression concerns`
5. `## Recommendation (Go/No-Go)`
- Save the same report to `verify-input/verify-report.md`.

## Minimal Context Loading
1. Read only files under `verify-input/`.
2. Avoid reading the broader repository unless a missing required file blocks verification.

## Definition of Done
- All required sections are present.
- Every finding cites concrete evidence from files inside `verify-input/`.
- Recommendation is explicit: `Go` or `No-Go`.
- Report does not reference `plan.md` or discussion history.
- Report calls out deviations from policy, including any `@Suppress` suppression behavior or non-JSpecify annotation semantics introduced without an explicit spec change.
