---
name: python-setup-dev-environment
description: Set up and run a reproducible Python dev environment with uv, ruff, mypy, and VSCode.
metadata:
  author: RyoMurakami1983
  tags: [python, uv, ruff, mypy, vscode]
  invocable: false
---

# Set Up a Python Dev Environment

Single-workflow guide for setting up and operating a reproducible Python development environment with `uv`, `ruff`, `mypy`, and VSCode save-time guardrails.

## When to Use This Skill

Use this skill when:
- Setting up a new Python project and standardizing execution with `uv run`
- Running lint/format/type-check in a consistent order before commit
- Troubleshooting VSCode save-time formatting behavior with Ruff
- Reproducing the same Python environment across team members and machines
- Migrating from ad-hoc `python` execution to explicit `uv`-based workflows

---

## Related Skills

- **`skills-validate-skill`** — Validate this skill document after edits
- **`git-commit-practices`** — Commit environment changes as atomic, reviewable units
- **`github-pr-workflow`** — Ship setup changes through PR workflow

---

## Dependencies

- `uv` (required)
- `ruff` and `mypy` as dev dependencies
- VSCode + Ruff extension (recommended)

---

## Core Principles

1. **Single runtime entrypoint** — Execute Python-related tasks through `uv run` to reduce environment drift (基礎と型)
2. **Fast feedback before commit** — Run `ruff` and `mypy` in a repeatable sequence (成長の複利)
3. **Reproducible dependency state** — Track environment state via `pyproject.toml` and `uv.lock` (温故知新)
4. **Safe editor automation** — Use explicit save-time actions to avoid destructive auto-fixes (ニュートラル)
5. **Incremental adoption** — Start minimal (`uv + ruff + mypy`) and add tools only when necessary (継続は力)

---

## Workflow: Set Up and Operate Python Dev Environment

### Step 1: Initialize and Pin Environment

Create project metadata and lock reproducible dependencies.

```powershell
# Initialize (run at repository/project root)
uv init .

# Verify managed Python runtime
uv run python --version

# Install dependencies
uv add --dev ruff mypy
```

Use when starting new Python work or normalizing an existing project.

**Values**: 基礎と型 / 継続は力

### Step 2: Standardize Daily Command Entry

Use `uv run` as the default command prefix for Python tooling.

```powershell
# ✅ CORRECT - Run commands through uv-managed environment
# Python execution
uv run python path\to\script.py

# Lint check
uv run ruff check .

# Format
uv run ruff format .

# Type check
uv run mypy .

# ❌ WRONG - Bypasses uv-managed runtime/dependencies
python path\to\script.py
```

Use when avoiding local interpreter/version mismatch.

**Values**: ニュートラル / 基礎と型

### Step 3: Run Quality Checks in a Safe Order

Follow a predictable order that minimizes churn and review noise. The order matters because it explains why each step comes next.

Why this sequence works:
- It clarifies why formatting must happen before lint checks.
- It clarifies why lint checks should run before type checks.
- It clarifies why type checks are the final gate before review.

| Phase | Command | Why |
|------|---------|-----|
| 1 | `uv run ruff format .` | Normalize formatting first |
| 2 | `uv run ruff check .` | Catch lint issues after formatting |
| 3 | `uv run mypy .` | Validate type-level correctness last |

Use when preparing changes for commit or pull request.

**Values**: 継続は力 / 成長の複利

### Step 4: Configure VSCode Save-Time Guardrails

Use Ruff formatter with explicit code actions to prevent unexpected rewrite behavior, and document why explicit actions are safer in mixed environments.

```json
{
  "[python]": {
    "editor.formatOnSave": true,
    "editor.codeActionsOnSave": {
      "source.fixAll": "explicit",
      "source.organizeImports": "explicit"
    },
    "editor.defaultFormatter": "charliermarsh.ruff"
  }
}
```

Use when save-time behavior causes unstable or surprising diffs.

**Values**: ニュートラル / 継続は力

### Step 5: Verify Reproducibility

Confirm that another machine/session can recreate the same environment.

```powershell
# Recreate environment from lock file
uv sync

# Re-run baseline checks
uv run ruff check .
uv run mypy .
```

Use when onboarding collaborators or validating CI parity.

**Values**: 温故知新 / 基礎と型

### Step 6: Define Terms Before Team Rollout

Use explicit definitions so everyone reads commands and settings the same way.

- **UV**: A fast Python package and project manager used as the runtime entrypoint.
- **LSP (Language Server Protocol)**: Editor protocol used for diagnostics and code actions.
- **CI**: Continuous Integration pipeline that should reproduce local checks.

Use when writing onboarding docs or handing off to another contributor.

**Values**: ニュートラル / 成長の複利

---

## Best Practices

- Use `uv run` for every Python-related command in docs and scripts.
- Define dependency changes as atomic commits with `pyproject.toml` and `uv.lock`.
- Apply `ruff format` before lint and type checks to reduce noisy diffs.
- Avoid automatic broad save-time fixes; keep save-time actions explicit.
- Consider adding project-specific exceptions only after baseline rules stabilize.

---

## Common Pitfalls

1. **Running bare `python` instead of `uv run python`**  
Fix: Replace command examples and scripts with `uv run ...` consistently.

2. **Using aggressive save-time auto-fixes**  
Fix: Keep `source.fixAll` and `source.organizeImports` as `explicit`.

3. **Skipping lockfile updates after dependency changes**  
Fix: Use `uv add`/`uv remove` and commit resulting `uv.lock` changes.

---

## Anti-Patterns

- Adding convenience tools before team baseline is stable (e.g., early task-runner sprawl)
- Mixing multiple formatters for Python in the same repository
- Treating type checks as optional after lint passes
- Using `uv pip install` as a default workflow command without documentation

---

## Quick Reference

### Setup

```powershell
uv init .
uv add --dev ruff mypy
uv run python --version
```

### Daily checks

```powershell
uv run ruff format .
uv run ruff check .
uv run mypy .
```

### Reproducibility

```powershell
uv sync
```

### Decision Table

| Situation | Action | Why |
|-----------|--------|-----|
| Need quick local check | `uv run ruff check .` | Catch style/lint issues fast |
| Need commit-ready diff | `uv run ruff format .` then `uv run ruff check .` | Format first, then enforce rules |
| Need confidence before PR | `uv run mypy .` | Catch type-level regressions early |

---

## FAQ

**Q: Should we include poethepoet in this skill?**  
A: No. This workflow intentionally stays minimal; add task runners in a separate issue if needed.

**Q: Why keep `codeActionsOnSave` as `explicit`?**  
A: It prevents unintended broad rewrites while still allowing controlled fixes.

**Q: Is `uv pip install` allowed?**  
A: As a rule, avoid it for normal workflow; prefer `uv add` / `uv remove` to keep project state reproducible. If an exception is unavoidable, document the reason and command in `README.md` (or equivalent project docs).

---

## Resources

- [uv documentation](https://docs.astral.sh/uv/)
- [Ruff documentation](https://docs.astral.sh/ruff/)
- [mypy documentation](https://mypy.readthedocs.io/)

---

## Changelog

### Version 1.0.0 (2026-02-14)
- Initial release for Issue #29

