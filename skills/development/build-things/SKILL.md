---
name: build-things
description: Build software features end to end in an existing repository. Use when the user asks to build, implement, add, create, wire up, or ship code changes, including backend, frontend, APIs, and automation tasks.
---

# Build Things

Execute implementation work from request to verified result.

## Workflow

1. Confirm scope from the user request.
2. Inspect the relevant code paths before editing.
3. Implement the smallest complete change that satisfies the request.
4. Prefer edits that match existing project patterns.
5. Run focused checks for the changed area (tests, lint, build, or app command).
6. Report what changed, what was validated, and any remaining risks.

## Implementation Rules

- Keep diffs minimal and avoid unrelated refactors.
- Reuse existing utilities and abstractions before adding new ones.
- Preserve backwards compatibility unless the user requested a breaking change.
- Add or update tests when behavior changes.
- If validation cannot run, state exactly what could not be run and why.

## File Review Pattern

- Read entrypoints and nearby modules first.
- Trace call sites and side effects before changing interfaces.
- Check config, environment assumptions, and platform-specific paths.

## Output Contract

Return a concise summary that includes:

- Changed files and intent.
- Verification commands executed and key outcomes.
- Open issues, follow-ups, or assumptions.
