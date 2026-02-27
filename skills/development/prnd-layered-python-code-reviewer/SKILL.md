---
name: prnd-layered-python-code-reviewer
description: Review Python code changes with a focus on layered architecture boundaries, DI/DIP, configuration validation, typing quality (mypy/flake8 readiness), naming/docstring accuracy, and testability. Use when asked to “code review”, “PR review”, or “리뷰해줘” to prevent repeat feedback on architecture regressions and low-signal changes.
---

# Layered Python Code Reviewer

## Overview

Provide a reusable code review workflow and checklist that applies across Python projects, especially ones that aim for layered architecture and dependency inversion. Produce actionable, concrete review comments (not vague style opinions).

## Review Workflow

1. Identify the change scope
   - Summarize touched modules and layers (example layers: `domain/`, `service/`, `dataset/`, `adapters/`, `infra/`).
   - Identify which changes are: (a) core policy/logic, (b) boundary I/O, (c) data shaping, (d) test-only.

2. Apply hard blockers first
   - **Dependency direction (DIP)**: low-level modules must not import high-level policy/service types.
   - **Public interfaces**: avoid leaking raw `dict` across layers; use `TypedDict`, dataclasses, or Pydantic models.
   - **Config correctness**: validate invariants on load (range continuity, required keys, value bounds).
   - **Type/lint hygiene**: fix `None` handling, unused variables, and any `Any` propagation caused by missing types.

3. Apply design-quality checks
   - **Composition root**: wiring (ENV/config/I/O/client construction) stays at the boundary (`create()`/entrypoint).
   - **Separation of concerns**: “build payload/message” vs “send request”, “decide” vs “orchestrate”.
   - **Test seams**: time/randomness/I/O are injectable; core logic is deterministic.

4. Write review output
   - Group comments by: `Architecture`, `DIP/DI`, `Typing`, `Config`, `Error handling`, `Tests`, `Naming/Docs`.
   - For each issue: state impact + preferred pattern + minimal change suggestion.

## References

Load and apply:
- `references/principles.md` for a project-agnostic checklist and comment templates.
- `references/review-themes.md` for common “review-driven refactor” patterns (kept generic on purpose).

## Optional Automation

If your project uses clear “layer folders” under a package root, run:

`python scripts/check_layered_imports.py --root <package_root_dir> --package-name <top_level_package>`

Use flags to match your project layout:
- `--layers domain,service,adapters,infra`
- `--forbid domain:service,adapters,infra`
- `--forbid adapters:service`

