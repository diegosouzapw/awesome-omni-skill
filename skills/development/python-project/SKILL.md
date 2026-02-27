---
name: python-project
description: Scaffold and harden Python projects using vpngw-aligned defaults (pyproject/setuptools-scm, src layout, Ruff, pytest, Typer, Pydantic) plus best practices for CLI tools, systemd services, APIs/UI apps, IaC/automation, security/networking, and AI/ML workflows.
---

# Python Project

Scaffold production-grade Python repositories with conservative defaults inspired by `services/vpngw`.

## Use This Skill For

- Creating a new Python project from scratch.
- Standardizing an existing Python repo layout and tooling.
- Adding or improving:
  - CLI applications
  - systemd services/timers
  - API services and UI apps
  - IaC and automation integration
  - security and networking controls
  - AI/ML pipelines and model-serving structure

## Defaults (vpngw-aligned)

- `pyproject.toml` with PEP 621 metadata.
- `setuptools` + `setuptools-scm` for build/versioning.
- `src/` package layout.
- `ruff` for linting/formatting checks.
- `pytest` for tests.
- `Typer` + `Rich` for CLI UX.
- `Pydantic` for config/schema validation.
- `Makefile` with `.DEFAULT_GOAL := all` and aggregate `all` target (for example `all: check build`).
- Optional packaged systemd assets in `src/<package>/systemd/`.

## Workflow

1. Gather missing essentials only:
   - Project name (distribution) and import package name.
   - Python version range (default: `>=3.11,<3.14`).
   - Workload profile(s): `cli`, `systemd`, `api`, `ui`, `iac`, `automation`, `security`, `networking`, `ai-ml`.
   - Runtime target (local VM, container, Kubernetes, hybrid).
2. Start from `references/base-layout.md` and `assets/pyproject.toml.template`.
3. Load only relevant profile references:
   - `references/cli-systemd.md`
   - `references/api-ui.md`
   - `references/iac-automation-security-networking.md`
   - `references/ai-ml.md`
4. Generate scaffolding and output in this order:
   - Directory tree
   - Full file contents (one file at a time)
   - Exact bootstrap/lint/test/build/run commands
   - Security + operations checklist
5. Keep placeholders (`TODO`) for environment-specific values and never invent secrets.

## Output Contract

Always include:

- `pyproject.toml`
- `.gitignore`
- `README.md`
- `src/<package>/__init__.py`
- `src/<package>/__main__.py`
- `tests/`

Add these when selected:

- `cli`: `src/<package>/cli.py` and `[project.scripts]`.
- `systemd`: `src/<package>/systemd/*.service` and optional `*.timer`, plus package-data configuration.
- `api`: `src/<package>/api.py` (ASGI app) and production run guidance.
- `ui`: `src/<package>/ui.py` and auth/network boundary notes.
- `iac`: `infra/terraform/` (or point to `$terraform` skill for full scaffolding).
- `automation`: `Makefile`, `.github/workflows/ci.yml`, `.pre-commit-config.yaml`.
  - `Makefile` must set `.DEFAULT_GOAL := all`.
  - `Makefile` must include an `all` target that aggregates primary checks/build.
- `ai-ml`: `src/<package>/ml/` split for train/eval/infer pipelines.

## Non-Negotiable Guardrails

- No secret material in repo, samples, logs, tests, or docs.
- Ignore runtime, build, cache, credential, and local config artifacts.
- Prefer bounded dependency ranges (`>=x,<y`) and avoid unconstrained pins unless required.
- Keep commands idempotent where practical.
- Use typed boundaries for external inputs (Pydantic models/dataclasses).
- Return explicit non-zero exit codes for CLI failures.
- Avoid shelling out when a Python API exists; if shell is required, set explicit timeouts and sanitize args.
- Keep networking code timeout-safe and retry-safe.

## Versioning and Release Pattern

Default to `setuptools-scm` with SemVer tags:

- Tag format: `<project>-vMAJOR.MINOR.PATCH`
- Generated runtime version file: `src/<package>/_version.py`
- Do not manually edit the version in `pyproject.toml` when SCM versioning is enabled.

For containerized/Helm-delivered apps, keep version layers related but independent:

- App version (source of functional behavior):
  - SemVer from tags.
- Image version (source of deployed artifact):
  - publish immutable tags (`sha-<shortsha>`, and release tags like `X.Y.Z` + `X.Y.Z-g<shortsha>`).
  - prefer production deploys pinned by digest.
- Helm chart versioning:
  - `Chart.yaml.version` tracks chart packaging changes.
  - `Chart.yaml.appVersion` tracks the default app/image SemVer.
  - do not force chart `version` to equal app SemVer.

## Templates and References

- `assets/pyproject.toml.template`: baseline vpngw-style project metadata and tooling.
- `assets/cli.py.template`: Typer-based CLI starter.
- `assets/api.py.template`: FastAPI starter with health endpoint.
- `assets/systemd.service.template`: hardened service unit baseline.
- `assets/github-actions-ci.yml.template`: minimal CI for lint/test/build.

Use detailed references only when needed:

- `references/base-layout.md`
- `references/cli-systemd.md`
- `references/api-ui.md`
- `references/iac-automation-security-networking.md`
- `references/ai-ml.md`
