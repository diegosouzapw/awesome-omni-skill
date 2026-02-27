---
name: init-python
description: Initialize or refresh a root `AGENTS.md` for a Python project from repository inspection, then enforce typing and architecture rules tailored to the detected stack.
metadata:
  mode: execution
  approval_policy: on-request
  model_hint: precise-and-pragmatic
---

# init-python - Python AGENTS Bootstrap

You are a senior Python engineer setting up a repository-specific `AGENTS.md` at the project root.

Your goal is to create practical, enforceable instructions for future coding work, based on repository evidence.

---

## Required Baseline Rules

Always include these rules in the generated `AGENTS.md`:

- No `typing.Any` unless strictly required and justified
- Prefer explicit type hints for public functions, methods, and module boundaries
- Strong typing at boundaries and core domain paths
- Separation of concerns
- Minimal abstraction (no speculative layers)
- No comments unless truly required for non-obvious logic
- Maintainable code over clever code
- Evolutive code (easy to extend safely)

Do not weaken or remove these baseline rules.

---

## Safety and Merge Rules

- Do not delete files.
- Do not perform destructive rewrites.
- If `AGENTS.md` already exists, merge updates by section and preserve user-authored guidance.
- If a safe merge is ambiguous, stop and report the conflict clearly.
- Keep edits scoped to root `AGENTS.md` unless the user asked for broader changes.

---

## Execution Steps

### 1) Validate this is a Python repository

Inspect repository evidence before writing rules:

- `pyproject.toml`, `requirements*.txt`, `Pipfile`, `poetry.lock`, `uv.lock`
- Tooling configs (`ruff`, `mypy`, `pytest`, `tox`, `nox`)
- Source roots (`src/`, package directories, `app/`, `services/`)
- Runtime entrypoints and API surfaces (`main.py`, ASGI/WSGI apps, CLI modules)

If Python evidence is missing, stop and report what was checked.

### 2) Build a project profile

Collect concrete, repo-derived inputs for `AGENTS.md`:

- Project structure (top-level and key source directories)
- Primary commands (dev, test, lint, typecheck, format, build)
- Mainly used packages (framework/runtime + key tooling)
- Existing conventions (packaging style, test layout, dependency management)

Do not guess. If uncertain, mark as `Not detected`.

### 3) Add project-specific rules

After inspection, append rules that fit the detected structure. Examples:

- Framework boundaries (FastAPI, Django, Flask, CLI app, workers)
- Validation at IO boundaries (request parsing, serialization, config/env loading)
- Service/repository/module boundaries based on current layout
- Error handling policy and logging conventions
- Test structure and fixtures based on repository patterns

Keep rules concrete and enforceable.

### 4) Add package-usage research policy

Always include this policy in `AGENTS.md`:

- Before using an unfamiliar package or advanced API, consult Context7 or package-specific skills/docs first.
- Prefer repository-approved package patterns over ad-hoc usage.
- Record key package usage constraints in short bullets when they are known.

### 5) Create or update root `AGENTS.md`

Ensure `AGENTS.md` contains these sections (merge if existing):

1. Purpose and scope
2. Stack profile
3. Non-negotiable coding rules (baseline rules)
4. Project-specific architecture rules
5. Project structure
6. Commands
7. Mainly used packages
8. Package research policy (Context7/package skills)
9. Delivery and quality expectations

Recommended additional guidance to include when applicable:

- Keep modules cohesive and explicit.
- Validate input near system boundaries.
- Prefer explicit return types for exported/public functions.
- Avoid hidden side effects in shared helpers.
- Minimize cross-module coupling.

---

## Output Requirements

After updating `AGENTS.md`, return:

- Whether `AGENTS.md` was created or updated
- Project structure summary used for rule generation
- Command list discovered from repo manifests
- Mainly used packages and why they were considered primary
- Added project-specific rules
- Any unknowns or skipped items

No filler text.

---

## Completion Condition

Task is complete only when all are true:

- Root `AGENTS.md` exists
- Baseline rules are present exactly in intent
- Project-specific rules are added from repository evidence
- Project structure, commands, and mainly used packages are reported
- Package research policy (Context7/package skills) is included

---
