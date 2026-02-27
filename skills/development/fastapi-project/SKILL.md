---
name: fastapi-project
description: Scaffold and evolve FastAPI projects with uv-based tooling, structured settings, and production-ready observability, resilience, availability, and security patterns aligned with python.instructions.md.
---

# FastAPI Project Skill 🧩

This skill scaffolds new FastAPI projects or upgrades existing ones with a production-ready layout and operational practices suitable for large systems.

## Scope and alignment 🧭

Mandatory reads (must be loaded before using this skill):

- [Python instructions](../../instructions/python.instructions.md) — use its identifiers when describing compliance.
- [Local-first dev baseline](../../instructions/includes/local-first-dev-baseline.include.md)
- [Quality gates baseline](../../instructions/includes/quality-gates-baseline.include.md)
- [Observability logging baseline](../../instructions/includes/observability-baseline.include.md)
- [AI-assisted change baseline](../../instructions/includes/ai-assisted-change-baseline.include.md)

## Inputs to confirm ✅

- Project name and Python version
- API surface (REST only, GraphQL, or mixed)
- Database (SQLite/PostgreSQL) and cache (Redis/none)
- Background jobs (Celery/Arq/RQ/none) and async worker needs
- Deployment target (container, PaaS, serverless)
- Environment split (local/staging/prod) and secrets strategy

## Quick reference 🧠

| Capability                  | Purpose                                                              | Key Outputs                                           |
| --------------------------- | -------------------------------------------------------------------- | ----------------------------------------------------- |
| Project scaffold            | Create a working FastAPI foundation aligned with Python instructions | `pyproject.toml`, `src/` layout, app core, routers    |
| Large-system structure      | Keep growth manageable with clear boundaries                         | Domain routers, service layers, adapters              |
| Observability baseline      | Operational visibility from day one                                  | Structured logs, request IDs, health endpoints        |
| Security baseline           | Protect data and reduce risk                                         | Secure defaults, auth boundaries, dependency scanning |
| Resilience and availability | Survive failures and scale safely                                    | Timeouts, retries, graceful degradation               |
| Quality gates               | Enforce fast feedback                                                | `make` targets or uv commands for lint/typecheck/test |

---

## Capabilities 🧰

### 1. Project scaffold (foundation)

Use this for new projects or to align an existing project with the standard layout.

Core requirements:

- Use `pyproject.toml` as the single source of truth. If available, start from [the template](../../instructions/templates/pyproject.toml) and set:
  - project metadata (name, version, requires-python)
  - dependency groups for dev tooling
  - ruff, mypy, pytest configuration
- Use `uv` for deterministic installs and lockfile management.
- Pin Python in `.python-version` and `requires-python`.
- Scaffold with a `src/` layout so imports are explicit and testable.
- Define app settings with `pydantic-settings` and environment variables.
- Keep the ASGI entrypoint thin; wire routers and dependencies in `main.py`.

Recommended layout:

```text
.
├── .python-version
├── pyproject.toml
├── src/
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py
│   │   ├── api/
│   │   │   ├── __init__.py
│   │   │   ├── v1/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── routes/
│   │   │   │   └── schemas/
│   │   ├── core/
│   │   │   ├── config.py
│   │   │   ├── logging.py
│   │   │   └── observability.py
│   │   ├── services/
│   │   ├── adapters/
│   │   └── health.py
└── tests/
```

Dependency defaults (adjust to requirements):

- `fastapi`
- `uvicorn` (or `gunicorn` + `uvicorn` workers for production)
- `pydantic-settings`
- `httpx` (for outbound HTTP)
- `pytest`
- `ruff`, `mypy`

### 2. Large-system structure and boundaries

Use these patterns when the system is expected to grow:

- Group routers by domain in `api/v1/routes/<domain>.py`; keep routing thin.
- Put business logic in services; keep I/O inside adapters/selectors.
- Use dependency injection for shared concerns (auth, DB sessions, clients).
- Use explicit schemas at boundaries; avoid leaking ORM models.
- Separate infrastructure wiring (clients, pools) from request handling.

### 3. Observability baseline

Observability is non-negotiable for production workloads:

- Configure structured logging and include required fields from the structured logging baseline.
- Add request ID/correlation ID middleware and propagate IDs into logs.
- Provide health endpoints (`/healthz`, `/readyz`) with clear dependency checks.
- Add optional hooks for metrics and tracing (Prometheus or OpenTelemetry) but keep them toggled by configuration.
- Never log secrets or raw personal data.

### 4. Security baseline

Protect data and enforce secure defaults:

- Load secrets from environment or a secret manager; never commit or log secrets.
- Configure CORS narrowly; never use `*` in production for credentialed routes.
- Enforce authn/authz at router boundaries; keep public vs private APIs explicit.
- Disable or restrict interactive docs in production when required.
- Use dependency pinning and vulnerability scanning; keep lock files updated.

### 5. Resilience and availability baseline

Build for failure and recovery:

- Set explicit timeouts on outbound HTTP/DB calls; never rely on defaults.
- Use retries with bounded backoff and jitter for transient failures; avoid retrying non-idempotent operations without safeguards.
- Use connection pooling and sensible concurrency limits for the ASGI server.
- Provide graceful shutdown and ensure background tasks are interruptible.
- Use caching for hot paths and provide safe fallbacks when caches fail.

### 6. Quality gates and verification

Align with Python quality gates:

- Prefer `make format`, `make lint`, `make typecheck`, `make test` when Makefile targets exist.
- Otherwise use `uv run ruff format .`, `uv run ruff check .`, `uv run mypy .`, `uv run pytest`.
- Add a lightweight ASGI startup check (import app, load settings) before shipping.

---

## Output expectations 📦

When executing this skill, produce:

- A scaffolded FastAPI project or a concrete refactor plan for an existing codebase
- A list of decisions made for observability, security, resilience, and availability
- A short validation checklist using the canonical quality gates

When information is missing, record **Unknown from code – {suggested action}** instead of guessing.

---

> **Version**: 1.0.0
> **Last Amended**: 2026-01-18
