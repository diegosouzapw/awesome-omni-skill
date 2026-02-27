---
name: lang-python
description: This skill should be used when the user asks to "write Python code", "debug a Python script", "set up a Python project", "explain Python concepts", or mentions Python-specific libraries like FastAPI, Django, or Pandas.
credit: modu-ai/moai-adk
---
<skill_doc>
<trigger_keywords>
## Trigger Keywords

Activate this skill when the user mentions any of:

**File Extensions**: `.py`, `pyproject.toml`, `requirements.txt`, `pytest.ini`, `setup.py`

**Python Core**: Python, python3, pip, poetry, uv, asyncio, async/await, type hints, dataclass, Protocol, TypeVar, ParamSpec, match/case, pattern matching

**Web Frameworks**: FastAPI, Django, Flask, Starlette, ASGI, WSGI, Depends, APIRouter, @app.get, @app.post

**Validation**: Pydantic, BaseModel, model_validate, ConfigDict, Field, validator, model_validator

**ORM/Database**: SQLAlchemy, AsyncSession, create_async_engine, Alembic, migrations, ORM, query builder

**Testing**: pytest, pytest-asyncio, @pytest.fixture, @pytest.mark.parametrize, mock, conftest.py, coverage

**Data Science**: numpy, pandas, polars, DataFrame, ndarray, data pipeline, ETL

**Package Management**: uv (preferred), poetry, pip install, virtual environment, venv
</trigger_keywords>

## ⛔ Forbidden Patterns

1.  **NO Mutable Default Arguments**: Never use `def foo(x=[])`. Use `None` as default and initialize inside the function.
2.  **NO Sync I/O in Async Functions**: Blocking operations (like `requests.get` or `time.sleep`) inside `async def` kill performance. Use `httpx` or `asyncio.sleep`.
3.  **NO Wildcard Imports**: Avoid `from module import *`. Explicit imports prevent namespace pollution and make code readable.
4.  **NO Bare Excepts**: Never use `except:`. Always catch specific exceptions (e.g., `except ValueError:`).
5.  **NO Pip/Poetry (unless forced)**: ALWAYS prefer `uv` for package management. It is significantly faster and more reliable. Only use `pip` or `poetry` if explicitly requested or if `uv` is unavailable.

## 🤖 Agent Tool Strategy

1.  **Discovery**: Check for `justfile` first. If it exists, use `just -l`. Read `pyproject.toml` or `requirements.txt`.
2.  **Package Management**: ALWAYS use `uv` for all package operations (`uv add`, `uv pip install`, `uv venv`). Only fallback to `pip`/`poetry` if `uv` fails.
3.  **Virtual Environment**: `uv venv` creates environments instantly. Use it.
4.  **Code Analysis**: Use `search_files` to find definitions. Avoid `grep` if possible.
5.  **Testing**: Use `pytest` for running tests. Prefer `pytest` over `unittest`.
6.  **Linting/Formatting**: Respect `ruff.toml` or `pyproject.toml` [tool.ruff] settings.

## Quick Reference (30 seconds)

Python 3.13+ Development Specialist - FastAPI, Django, async patterns, pytest, and modern Python features.

Auto-Triggers: `.py` files, `pyproject.toml`, `requirements.txt`, `pytest.ini`, FastAPI/Django discussions

Core Capabilities:
- Python 3.13 Features: JIT compiler (PEP 744), GIL-free mode (PEP 703), pattern matching
- Web Frameworks: FastAPI 0.115+, Django 5.2 LTS
- Data Validation: Pydantic v2.9 with model_validate patterns
- ORM: SQLAlchemy 2.0 async patterns
- Testing: pytest with fixtures, async testing, parametrize
- Package Management: poetry, uv, pip with pyproject.toml
- Type Hints: Protocol, TypeVar, ParamSpec, modern typing patterns
- Async: asyncio, async generators, task groups
- Data Science: numpy, pandas, polars basics

### Quick Patterns

FastAPI Endpoint:
```python
from fastapi import FastAPI, Depends
from pydantic import BaseModel

app = FastAPI()

class UserCreate(BaseModel):
    name: str
    email: str

@app.post("/users/")
async def create_user(user: UserCreate) -> User:
    return await UserService.create(user)
```

Pydantic v2.9 Validation:
```python
from pydantic import BaseModel, ConfigDict

class User(BaseModel):
    model_config = ConfigDict(from_attributes=True, str_strip_whitespace=True)

    id: int
    name: str
    email: str

user = User.model_validate(orm_obj)  # from ORM object
user = User.model_validate_json(json_data)  # from JSON
```

pytest Async Test:
```python
import pytest

@pytest.mark.asyncio
async def test_create_user(async_client):
    response = await async_client.post("/users/", json={"name": "Test"})
    assert response.status_code == 201
```

---

## Implementation Guide (5 minutes)

See [patterns.md](references/patterns.md) for the detailed implementation guide covering:
- Python 3.13 New Features (JIT, GIL-Free)
- FastAPI 0.115+ Patterns (Async DI, Class Dependencies)
- Django 5.2 LTS Features
- Pydantic v2.9 Deep Patterns
- SQLAlchemy 2.0 Async Patterns
- pytest Advanced Patterns
- Type Hints Modern Patterns
- Package Management (Poetry, uv)

---

## Advanced Implementation (10+ minutes)

For comprehensive coverage including:
- Production deployment patterns (Docker, Kubernetes)
- Advanced async patterns (task groups, semaphores)
- Data science integration (numpy, pandas, polars)
- Performance optimization techniques
- Security best practices (OWASP patterns)
- CI/CD integration patterns

See:
- [patterns.md](references/patterns.md) - Implementation patterns (FastAPI, Pydantic, SQLAlchemy)
- [reference.md](references/reference.md) - Complete reference documentation
- [examples.md](examples/examples.md) - Production-ready code examples

---

## Context7 Library Mappings

```
/docs/astral.sh/uv
/tiangolo/fastapi - FastAPI async web framework
/django/django - Django web framework
/pydantic/pydantic - Data validation with type annotations
/sqlalchemy/sqlalchemy - SQL toolkit and ORM
/pytest-dev/pytest - Testing framework
/numpy/numpy - Numerical computing
/pandas-dev/pandas - Data analysis library
/pola-rs/polars - Fast DataFrame library
```

---

## Troubleshooting

Common Issues:

Python Version Check:
```bash
python --version  # Should be 3.13+
python -c "import sys; print(sys.version_info)"
```

Async Session Detached Error:
- Solution: Set `expire_on_commit=False` in session config
- Or: Use `await session.refresh(obj)` after commit

pytest asyncio Mode Warning:
```toml
# pyproject.toml
[tool.pytest.ini_options]
asyncio_mode = "auto"
asyncio_default_fixture_loop_scope = "function"
```

Pydantic v2 Migration:
- `parse_obj()` is now `model_validate()`
- `parse_raw()` is now `model_validate_json()`
- `from_orm()` requires `from_attributes=True` in ConfigDict
</skill_doc>
