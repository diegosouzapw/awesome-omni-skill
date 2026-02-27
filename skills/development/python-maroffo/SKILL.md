---
name: python
description: "Python development with uv, type checking, linting, testing, and Docker deployment. Use when working with .py files, pyproject.toml, or user asks about pytest, mypy, ruff, FastAPI, Django."
allowed-tools: [mcp__acp__Read, mcp__acp__Edit, mcp__acp__Write, mcp__acp__Bash]
---

# ABOUTME: Complete Python development with uv package manager, quality tools, and Docker
# ABOUTME: Modern workflow with uv, ruff, ty, pytest, Pydantic, and containerization

# Python Development

## What's New (2025-2026)

### Python 3.14 (Oct 2025)
- **Template strings (PEP 750)**: `t"Hello {name}"` - safe SQL/HTML interpolation
- **Incremental GC**: 10ms pauses (was 100ms)
- **uuid7**: Time-ordered UUIDs (better for DBs)
- **Remote pdb**: `python -p PID`

### Tooling
| Tool | Notes |
|------|-------|
| ty | Astral's type checker, 10-60x faster than mypy |
| Ruff 0.8+ | Type-aware linting, 800+ rules |
| uv 0.5+ | Stable, production-ready |

## Quick Reference

```bash
uv init myproject && cd myproject              # New project
uv add requests pydantic httpx                 # Add deps
uv add --dev pytest ruff                       # Dev deps
uv sync --locked                               # CI-safe sync
uv run python main.py                          # Run code
uv run ruff check . && uv run ruff format --check . && uvx ty check && uv run pytest  # Quality
```

**Target: Python 3.13** | **See also:** `_AST_GREP.md`, `_PATTERNS.md`, `source-control`

---

## Package Management (uv)

**UV is the ONLY way. Do NOT use pip/poetry/pipenv.** 10-100x faster, universal lockfile.

### pyproject.toml
```toml
[project]
name = "myproject"
version = "0.1.0"
requires-python = ">=3.13"
dependencies = ["httpx>=0.27.0", "pydantic>=2.10.0"]

[dependency-groups]
dev = ["pytest>=8.0.0", "ruff>=0.8.0"]

[tool.pytest.ini_options]
testpaths = ["tests"]
asyncio_mode = "auto"
```

---

## Code Quality

### Type Checking (ty)
```bash
uvx ty check          # Run
```

### Linting (Ruff)
```bash
uv run ruff check . && uv run ruff format --check .
```

```toml
[tool.ruff]
line-length = 100
target-version = "py313"

[tool.ruff.lint]
select = ["E", "F", "I", "N", "W", "UP", "B", "C4", "SIM", "TCH", "RUF", "PERF"]
```

### Testing (pytest)
```bash
uv run pytest --cov=myproject --cov-report=html
```

```python
@pytest.fixture
def user_service(mock_repo): return UserService(repo=mock_repo)

def test_find_user(user_service, mock_repo):
    mock_repo.find.return_value = User(id="1", name="Max")
    assert user_service.find("1").name == "Max"

@pytest.mark.parametrize("email,expected", [("test@x.com", True), ("bad", False)])
def test_email(email, expected): assert validate_email(email) == expected
```

---

## Pydantic v2

```python
from pydantic import BaseModel, Field, field_validator

class User(BaseModel):
    id: int
    name: str = Field(min_length=1, max_length=100)
    email: str

    @field_validator("email")
    @classmethod
    def validate_email(cls, v): return v.lower() if "@" in v else raise ValueError("Invalid")

# Performance
user = User.model_validate_json('{"id":1,"name":"Max","email":"m@x.com"}')  # Fast JSON
```

---

## Async Patterns

```python
async def fetch_all(ids: list[str]) -> list[User]:
    return await asyncio.gather(*[fetch_user(uid) for uid in ids])

# Rate limiting
async def fetch_limited(urls: list[str], max_concurrent: int = 10):
    sem = asyncio.Semaphore(max_concurrent)
    async def fetch(url):
        async with sem:
            async with httpx.AsyncClient() as c: return await c.get(url)
    return await asyncio.gather(*[fetch(u) for u in urls])
```

---

## Docker

```dockerfile
FROM ghcr.io/astral-sh/uv:python3.13-bookworm-slim AS builder
WORKDIR /app
COPY pyproject.toml uv.lock ./
RUN uv sync --frozen --no-cache --no-dev

FROM python:3.13-slim-bookworm
COPY --from=ghcr.io/astral-sh/uv:latest /uv /bin/uv
RUN useradd --create-home --uid 1000 app
WORKDIR /app
COPY --from=builder /app/.venv /app/.venv
COPY --chown=app:app src/ ./src/
USER app
ENV PATH="/app/.venv/bin:$PATH" PYTHONDONTWRITEBYTECODE=1 PYTHONUNBUFFERED=1
CMD ["python", "-m", "myapp"]
```

---

## CI/CD (GitHub Actions)

```yaml
name: CI
on: [push, pull_request]
jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v5
        with: { enable-cache: true }
      - run: uv python install 3.13
      - run: uv sync --locked
      - run: uv run ruff format --check . && uv run ruff check . && uvx ty check && uv run pytest --cov
```

---

## Code Review Checklist

- [ ] Public functions have type hints, no unjustified `Any`
- [ ] Pydantic for external data, `X | None` not `Optional[X]`
- [ ] Ruff/type checker pass, no bare `except:`
- [ ] Context managers, `asyncio.gather()` for concurrency
- [ ] AAA tests, parametrized

---

## Resources

**Astral**: [uv](https://docs.astral.sh/uv/), [Ruff](https://docs.astral.sh/ruff/), [ty](https://docs.astral.sh/ty/)
**Python**: [pytest](https://docs.pytest.org/), [Pydantic](https://docs.pydantic.dev/)
