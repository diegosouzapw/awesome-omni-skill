---
name: python-uv
description: Modern Python development with uv package manager. Use when working on Python projects using uv, pytest, FastAPI, or Django. Covers development workflow, testing, and EC2 deployment.
allowed-tools: Read, Edit, Write, Bash, Grep, Glob, Task
---

# Python/uv Development

Modern Python development workflow using uv instead of pip/poetry.

## Why uv?

- 10-100x faster than pip
- Drop-in replacement for pip, pip-tools, virtualenv
- Lockfile support (`uv.lock`)
- Built-in Python version management

## Project Setup

### New Project

```bash
# Initialize project
uv init my-project
cd my-project

# Or with specific Python version
uv init my-project --python 3.12
```

### Existing Project

```bash
# Create venv and sync dependencies
uv sync

# With dev dependencies
uv sync --dev
```

### pyproject.toml Structure

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "my-project"
version = "0.1.0"
description = "Project description"
readme = "README.md"
requires-python = ">=3.12"
authors = [{ name = "Your Name", email = "you@example.com" }]
dependencies = [
    "fastapi~=0.118",
    "uvicorn[standard]~=0.37",
    "pydantic>=2.12,<3",
]

[project.optional-dependencies]
dev = [
    "pytest",
    "pytest-asyncio",
    "pytest-cov",
    "ruff",
    "black",
]

[project.scripts]
myapp = "app.main:start"

[tool.pytest.ini_options]
asyncio_mode = "auto"
testpaths = ["tests"]
addopts = "-q --tb=short"

[tool.ruff]
target-version = "py312"
line-length = 88

[tool.black]
line-length = 88
target-version = ["py312"]
```

## Development Workflow

### Dependency Management

```bash
# Add dependency
uv add fastapi

# Add dev dependency
uv add --dev pytest

# Add with version constraint
uv add "pydantic>=2.0,<3"

# Remove dependency
uv remove package-name

# Update all dependencies
uv sync --upgrade

# Update specific package
uv add package-name --upgrade
```

### Running Code

```bash
# Run script
uv run python script.py

# Run module
uv run python -m app.main

# Run project script (from pyproject.toml)
uv run myapp

# Run with specific Python
uv run --python 3.12 python script.py
```

### Testing

```bash
# Run all tests
uv run pytest

# Run with coverage
uv run pytest --cov=app --cov-report=term-missing

# Run specific test file
uv run pytest tests/test_api.py

# Run tests in parallel
uv run pytest -n auto

# Run marked tests
uv run pytest -m "not slow"
```

### Linting and Formatting

```bash
# Format with black
uv run black .

# Check formatting
uv run black --check .

# Lint with ruff
uv run ruff check .

# Auto-fix ruff issues
uv run ruff check --fix .

# Sort imports
uv run isort .
```

## FastAPI Development

### Project Structure

```
my-project/
├── app/
│   ├── __init__.py
│   ├── main.py           # FastAPI app
│   ├── api/
│   │   ├── __init__.py
│   │   └── routes.py
│   ├── models/
│   │   └── __init__.py
│   └── tests/
│       └── test_api.py
├── pyproject.toml
├── uv.lock
└── README.md
```

### Running FastAPI

```bash
# Development with auto-reload
uv run uvicorn app.main:app --reload --port 8000

# Production
uv run uvicorn app.main:app --host 0.0.0.0 --port 8000 --workers 4
```

## Django Development

### Running Django

```bash
# Migrations
uv run python manage.py migrate

# Run server
uv run python manage.py runserver

# Create superuser
uv run python manage.py createsuperuser

# Shell
uv run python manage.py shell
```

## Database Migrations (Alembic)

```bash
# Generate migration
uv run alembic revision --autogenerate -m "Add users table"

# Apply migrations
uv run alembic upgrade head

# Rollback
uv run alembic downgrade -1
```

## EC2 Deployment

### Deploy Script Pattern (from rookery)

```bash
# GitHub Actions deploy pattern
SCRIPT_AFTER: |
  set -e
  # Install uv
  curl -LsSf https://astral.sh/uv/install.sh | sh
  export PATH="$HOME/.local/bin:$PATH"

  cd /opt/myapp

  # Sync dependencies
  uv sync

  # Run migrations
  uv run alembic upgrade head

  # Restart service
  sudo systemctl restart myapp

  # Health check with retries
  for i in {1..5}; do
    if curl -sf http://127.0.0.1:8000/api/health; then
      echo "Health check passed"
      exit 0
    fi
    sleep 5
  done
  exit 1
```

### Systemd Service

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My FastAPI App
After=network.target

[Service]
User=appuser
WorkingDirectory=/opt/myapp
ExecStart=/home/appuser/.local/bin/uv run uvicorn app.main:app --host 0.0.0.0 --port 8000 --workers 2
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

## CI/CD

### GitHub Actions

```yaml
name: Python Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install uv
        uses: astral-sh/setup-uv@v4
        with:
          version: "latest"

      - name: Set up Python
        run: uv python install 3.12

      - name: Install dependencies
        run: uv sync --dev

      - name: Run tests
        run: uv run pytest

      - name: Lint
        run: uv run ruff check .
```

## Common Patterns

### Version Stamping

```bash
# Stamp build version for deployments
printf "%s\n" "$(date -u +%Y%m%dT%H%M%SZ)_$(git rev-parse --short HEAD)" > VERSION
```

### Environment Variables

```python
# app/config.py
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    database_url: str
    secret_key: str
    debug: bool = False

    class Config:
        env_file = ".env"

settings = Settings()
```

## Quick Reference

```bash
# Essential commands
uv sync              # Install all dependencies
uv run pytest        # Run tests
uv run ruff check .  # Lint
uv run black .       # Format
uv add <pkg>         # Add dependency
uv add --dev <pkg>   # Add dev dependency

# FastAPI
uv run uvicorn app.main:app --reload

# Django
uv run python manage.py runserver

# Alembic
uv run alembic upgrade head
```
