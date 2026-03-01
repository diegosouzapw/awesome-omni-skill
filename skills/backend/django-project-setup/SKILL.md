---
name: django-project-setup
description: Set up a new Django 6.0 project with modern tooling (uv, direnv, HTMX, OAuth, DRF, testing). Use when the user wants to create a Django project from scratch with production-ready configuration.
user-invocable: true
argument-hint: "[project_name] [--with-supabase] - Name of the Django project to create. Use --with-supabase to use Supabase instead of Docker PostgreSQL."
context: fork
agent: django-expert
hooks:
  stop:
    - name: validate-django-setup
      command: forge validate django --skip-django-checks
      description: Validate Django project setup with type checking, linting, and tests
---

# Django 6.0 Project Setup

Automated setup for production-ready Django 6.0 projects with modern tooling.

## What This Skill Does

Creates a complete Django 6.0 project with:

### Modern Stack (2026)
- ✅ **uv** - Fast package manager (no pip/venv needed)
- ✅ **Django 6.0** - Latest version with Python 3.12+ support
- ✅ **PostgreSQL** - Docker Compose + Supabase support
- ✅ **Custom User Model** - UUID primary keys from day 1
- ✅ **direnv** - Automatic environment management
- ✅ **HTMX** - Modern frontend interactivity
- ✅ **OAuth** - Authentication ready (django-oauth-toolkit)
- ✅ **DRF + Pydantic** - Type-safe API schemas
- ✅ **pytest + factory_boy** - Modern testing infrastructure
- ✅ **mypy + ruff** - Type checking and linting
- ✅ **Docker Compose** - Local PostgreSQL container
- ✅ **Makefile** - Common development commands
- ✅ **Full Type Annotations** - AI tooling optimized (Copilot, Cursor, Cody)

## AI Tooling Compatibility

This project template is optimized for AI coding assistants:

- ✅ **Full Type Coverage**: All generated code has comprehensive type annotations
- ✅ **Strict mypy**: `disallow_untyped_defs = true` enforced from day 1
- ✅ **Better Completions**: AI tools understand code context and relationships
- ✅ **Fewer Errors**: Type checking catches issues before runtime
- ✅ **Improved Refactoring**: AI can suggest safer, type-aware refactorings
- ✅ **Self-Documenting**: Type hints serve as inline documentation

Compatible with: GitHub Copilot, Cursor, Cody, Continue, Tabnine, and other AI coding assistants.

### Why This Matters

When you provide full type annotations:
- AI assistants give more accurate code completions
- Auto-import suggestions are more precise
- Refactoring suggestions are type-safe
- Error detection happens as you type
- Generated documentation is more comprehensive

### Directory Structure
```
{project_name}/
├── config/              # Settings and core config
│   ├── settings/
│   │   ├── __init__.py
│   │   ├── base.py      # Core settings
│   │   ├── dev.py       # Development
│   │   └── prod.py      # Production
│   ├── urls.py
│   ├── asgi.py
│   └── wsgi.py
├── apps/                # Django apps
│   ├── __init__.py
│   └── core/            # Shared utilities
│       ├── models.py    # UUIDModel base class
│       └── ...
├── templates/           # Global templates
├── static/              # Global static files
├── pyproject.toml       # uv dependencies
├── pytest.ini           # Test configuration
├── conftest.py          # Test fixtures
├── docker-compose.yml   # PostgreSQL container
├── Makefile             # Development commands
├── .envrc               # Environment config
├── .env.example         # Environment template
├── manage.py
└── CLAUDE.md            # Project instructions
```

## Usage

```bash
# With Docker PostgreSQL (default)
/django-project-setup myproject

# With Supabase
/django-project-setup myproject --with-supabase
```

The skill will:
1. Parse arguments (project name and optional --with-supabase flag)
2. Ask for first app name (e.g., 'accounts', 'blog', 'api')
3. Create project structure with uv
4. Set up database (Docker Compose or Supabase instructions)
5. Configure direnv environment
6. Install all dependencies
7. Create custom User model with UUID
8. Run initial migrations (or show Supabase setup instructions)
9. Set up testing infrastructure
10. Configure HTMX, OAuth, and DRF
11. Create CLAUDE.md with project patterns

## Requirements

- **Python 3.12+** (Django 6.0 requirement - automatically installed by uv if missing)
- **uv** (will be installed if missing - handles Python version management)
- **direnv** (recommended, will prompt if missing)
- **Docker** (for local PostgreSQL)

## After Setup

### With Docker PostgreSQL (default)

```bash
# Start PostgreSQL
/usr/bin/make start-docker

# Run migrations
/usr/bin/make migrate

# Create superuser (set password in .envrc.local first)
/usr/bin/make createsuperuser

# Start development server
/usr/bin/make runserver

# Run tests
/usr/bin/make test

# Type checking
/usr/bin/make typecheck

# Linting
/usr/bin/make lint
```

Visit http://localhost:8000/admin/ to verify setup.

### With Supabase

```bash
# 1. Complete Supabase setup (see SUPABASE_SETUP.md)
# 2. Add DATABASE_URL to .env.local
# 3. Allow direnv
direnv allow

# Run migrations
uv run python manage.py migrate

# Create superuser
uv run python manage.py createsuperuser

# Start development server
uv run python manage.py runserver

# Run tests
uv run pytest

# Type checking
uv run mypy .

# Linting
uv run ruff check .
```

Visit http://localhost:8000/admin/ to verify setup.

## Execution Instructions

When this skill is invoked with a project name (e.g., `/django-project-setup myproject` or `/django-project-setup myproject --with-supabase`), follow these steps:

### Prerequisites Check

1. **Parse Arguments**:
   ```bash
   # Extract project name (first non-flag argument)
   # Check if --with-supabase flag is present
   # Set use_supabase=true if flag found, false otherwise
   ```

2. Check if uv is installed, install if missing:
   ```bash
   which uv || curl -LsSf https://astral.sh/uv/install.sh | sh
   ```

3. Check if Python 3.12+ is available, install via uv if missing:
   ```bash
   # Check current Python version
   python3 --version 2>/dev/null | grep -q "Python 3.1[2-9]" || \
   # If not found, use uv to install Python 3.12
   uv python install 3.12
   ```

4. Verify current directory or use /tmp if not specified

### Execution Steps

**Step 1: Ask for First App Name**

Use AskUserQuestion to ask:
```
Question: "What should be the first Django app to create?"
Options:
  - "blog" (description: "Blog/content management app")
  - "accounts" (description: "User accounts and profiles")
  - "api" (description: "API endpoints")
  - "core" (description: "Core functionality only")
Header: "First App"
```

**Step 2: Initialize uv Project**

```bash
# Initialize project with Python 3.12+
uv init --python 3.12 {project_name}
builtin cd {project_name}

# Verify .python-version file was created
[ -f .python-version ] && echo "✅ Python version pinned" || echo "3.12" > .python-version
```

**Step 3: Add Core Dependencies**

```bash
uv add django psycopg[binary] django-environ
uv add djangorestframework django-oauth-toolkit django-htmx pydantic
uv add --group dev pytest pytest-django pytest-cov pytest-asyncio
uv add --group dev factory-boy pytest-factoryboy
uv add --group dev mypy django-stubs types-requests
uv add --group dev ruff django-extensions ipython
```

**Step 4: Create Django Project Structure**

```bash
uv run django-admin startproject config .
```

**Step 5: Create Directory Structure**

```bash
mkdir -p apps/core/{migrations,tests}
mkdir -p apps/{first_app_name}/{migrations,tests}
mkdir -p templates/partials
mkdir -p static
mkdir -p config/settings
```

**Step 6: Create Configuration Files**

Use the templates in `templates/` directory to create:

1. `Makefile` - Use Makefile.template (conditionally add Docker targets if use_supabase == false)
2. `.envrc` - Use .envrc.template, configure DATABASE_URL based on use_supabase
3. `.env` - Use .env.template, configure database variables based on use_supabase
4. `.env.example` - Use .env.example.template, configure based on use_supabase
5. `.envrc.local` - Use .envrc.local.template
6. `.gitignore` - Use .gitignore.template
7. `pytest.ini` - Use pytest.ini.template
8. `conftest.py` - Use conftest.py.template
9. `CLAUDE.md` - Use CLAUDE.md.template, replace {project_name} and add database-specific notes

**If use_supabase == false (Docker):**
- Create `docker-compose.yml` - Use docker-compose.yml.template, replace {project_name}

**If use_supabase == true:**
- Create `SUPABASE_SETUP.md` with instructions for Supabase project creation
- Display message: "⚠️  Supabase setup required - see SUPABASE_SETUP.md for instructions"

**Step 7: Split Settings Files**

1. Delete `config/settings.py`
2. Create `config/settings/__init__.py` (empty)
3. Create `config/settings/base.py` with core settings (see implementation guide below)
4. Create `config/settings/dev.py` with development settings
5. Create `config/settings/prod.py` with production settings

**Step 8: Create Core App**

1. Create `apps/__init__.py` (empty)
2. Create `apps/core/__init__.py` (empty)
3. Create `apps/core/apps.py` with CoreConfig
4. Create `apps/core/models.py` with UUIDModel and User
5. Create `apps/core/admin.py` with UserAdmin
6. Create `apps/core/factories.py` with UserFactory
7. Create `apps/core/tests/__init__.py` (empty)
8. Create `apps/core/tests/test_models.py` with User tests
9. Create `apps/core/migrations/__init__.py` (empty)

**Step 9: Create First App**

```bash
mkdir -p apps/{first_app_name}
uv run python manage.py startapp {first_app_name} apps/{first_app_name}
```

Update `apps/{first_app_name}/apps.py` to use UUID default.

**Step 10: Update Config Files**

1. Update `config/settings/base.py` to include:
   - apps.core
   - apps.{first_app_name}
   - All third-party apps

2. Update `config/urls.py` to include OAuth and API auth URLs

**Step 11: Create Initial Migrations**

```bash
uv run python manage.py makemigrations core
uv run python manage.py makemigrations {first_app_name}
```

**Step 12: Update pyproject.toml**

Add tool configurations for mypy, ruff, and pytest at the end of pyproject.toml.

**Step 13: Database Setup**

**If use_supabase == false (Docker):**
```bash
# Start PostgreSQL container
/usr/bin/make start-docker

# Wait for database to be ready
sleep 3

# Run migrations
uv run python manage.py migrate
```

**If use_supabase == true:**
```bash
# Display setup instructions
echo ""
echo "=========================================="
echo "⚠️  Supabase Setup Required"
echo "=========================================="
echo ""
echo "Next steps:"
echo "1. Create a Supabase project at https://supabase.com"
echo "2. Go to Settings > Database"
echo "3. Copy the 'Connection string' (URI format)"
echo "4. Add to .env.local:"
echo "   DATABASE_URL=postgresql://postgres.[PROJECT-REF]:[PASSWORD]@aws-0-[REGION].pooler.supabase.com:6543/postgres"
echo "5. Run: direnv allow"
echo "6. Run migrations: uv run python manage.py migrate"
echo ""
echo "See SUPABASE_SETUP.md for detailed instructions"
echo ""
```

**Step 14: Verify Setup**

Run verification checks:

```bash
# Check if files exist
[ -f Makefile ] && echo "✅ Makefile"
[ -f .envrc ] && echo "✅ direnv config"
[ -f pytest.ini ] && echo "✅ pytest config"
[ -f CLAUDE.md ] && echo "✅ Documentation"

# Database-specific checks
if [ "$use_supabase" = "true" ]; then
  [ -f SUPABASE_SETUP.md ] && echo "✅ Supabase setup instructions"
else
  [ -f docker-compose.yml ] && echo "✅ Docker Compose"
fi

# Check Django setup (only if DATABASE_URL is configured for Supabase)
if [ "$use_supabase" = "false" ] || [ -n "$DATABASE_URL" ]; then
  uv run python manage.py check
fi

# Check if tests can be collected
uv run pytest --collect-only
```

**Step 15: Display Summary**

Show the user:
- Project structure created
- Database configuration (Docker PostgreSQL or Supabase)
- List of installed dependencies
- Next steps to start using the project
- Available commands (Make targets for Docker, or direct uv commands for Supabase)

### Error Handling

- If Python < 3.12: Use `uv python install 3.12` to automatically download and install Python 3.12
- If uv installation fails: Show installation instructions
- If Django project creation fails: Show Django error and suggest fixes
- If migrations fail: Show error and suggest checking models
- If `uv python install` fails: Check network connectivity and suggest manual Python installation

### Success Criteria

- All configuration files created
- Dependencies installed
- Initial migrations created (or Supabase setup instructions provided)
- Tests can be collected
- Django check passes (if database is configured)
- CLAUDE.md documentation exists
- Database setup complete (Docker Compose or SUPABASE_SETUP.md)

## Implementation Guide

### Step 0: Docker Compose Setup

Create `docker-compose.yml` for local PostgreSQL:

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:16-alpine
    container_name: ${PROJECT_NAME:-django}_db
    environment:
      POSTGRES_DB: ${POSTGRES_DB:-django_dev}
      POSTGRES_USER: ${POSTGRES_USER:-postgres}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-postgres}
    ports:
      - "${POSTGRES_PORT:-5432}:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:
```

Create `Makefile` for common operations:

```makefile
.PHONY: start-docker stop-docker restart-docker logs shell migrate makemigrations test runserver createsuperuser lint format typecheck

start-docker:
	docker-compose up -d

stop-docker:
	docker-compose down

restart-docker:
	docker-compose restart

logs:
	docker-compose logs -f postgres

shell:
	uv run python manage.py shell_plus

migrate:
	uv run python manage.py migrate

makemigrations:
	uv run python manage.py makemigrations

test:
	uv run pytest

runserver:
	uv run python manage.py runserver

createsuperuser:
	uv run python manage.py createsuperuser

lint:
	uv run ruff check .

format:
	uv run ruff format .

typecheck:
	uv run mypy .
```

### Step 1: Initial Project Setup

```bash
# Ensure Python 3.12+ is available
uv python install 3.12

# Initialize uv project with Python 3.12+
uv init --python 3.12 {project_name}
cd {project_name}

# Verify Python version is pinned
echo "3.12" > .python-version

# Add Django and core dependencies
uv add django psycopg[binary] django-environ

# Create Django project structure
uv run django-admin startproject config .
```

### Step 2: Environment Setup (direnv)

**For Docker PostgreSQL (default):**

Create `.envrc`:

```bash
# Load environment from .env files
dotenv_if_exists .env
dotenv_if_exists .env.local

# Docker Database (local development)
export POSTGRES_DB="${POSTGRES_DB:-django_dev}"
export POSTGRES_USER="${POSTGRES_USER:-postgres}"
export POSTGRES_PASSWORD="${POSTGRES_PASSWORD:-postgres}"
export POSTGRES_PORT="${POSTGRES_PORT:-5432}"
export DATABASE_URL="${DATABASE_URL:-postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@localhost:${POSTGRES_PORT}/${POSTGRES_DB}}"

# Django
export DJANGO_SETTINGS_MODULE="${DJANGO_SETTINGS_MODULE:-config.settings.dev}"
export DJANGO_SECRET_KEY="${DJANGO_SECRET_KEY:-dev-secret-key-change-in-production}"
export DEBUG="${DEBUG:-true}"

# Allow local overrides
source_env_if_exists .envrc.local
```

Create `.env.example`:

```bash
# Database Configuration
POSTGRES_DB=django_dev
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
POSTGRES_PORT=5432
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/django_dev

# Django Configuration
DJANGO_SETTINGS_MODULE=config.settings.dev
DJANGO_SECRET_KEY=dev-secret-key-change-in-production
DJANGO_SUPERUSER_PASSWORD=admin
DEBUG=true
```

**For Supabase (--with-supabase):**

Create `.envrc`:

```bash
# Load environment from .env files
dotenv_if_exists .env
dotenv_if_exists .env.local

# Supabase Database
# Set DATABASE_URL in .env.local (see SUPABASE_SETUP.md)
export DATABASE_URL="${DATABASE_URL:-}"

# Django
export DJANGO_SETTINGS_MODULE="${DJANGO_SETTINGS_MODULE:-config.settings.dev}"
export DJANGO_SECRET_KEY="${DJANGO_SECRET_KEY:-dev-secret-key-change-in-production}"
export DEBUG="${DEBUG:-true}"

# Allow local overrides
source_env_if_exists .envrc.local
```

Create `.env.example`:

```bash
# Supabase Configuration
# Get these values from your Supabase project:
# https://supabase.com/dashboard/project/[PROJECT-REF]/settings/database
DATABASE_URL=postgresql://postgres.[PROJECT-REF]:[PASSWORD]@aws-0-[REGION].pooler.supabase.com:6543/postgres

# Django Configuration
DJANGO_SETTINGS_MODULE=config.settings.dev
DJANGO_SECRET_KEY=dev-secret-key-change-in-production
DJANGO_SUPERUSER_PASSWORD=admin
DEBUG=true
```

Create `SUPABASE_SETUP.md`:

```markdown
# Supabase Setup Instructions

This project is configured to use Supabase as the PostgreSQL database.

## Steps

1. **Create a Supabase Project**
   - Go to https://supabase.com
   - Click "New Project"
   - Choose your organization and set a database password

2. **Get Database Connection String**
   - Go to Settings > Database
   - Find "Connection string" section
   - Copy the "URI" format connection string
   - It looks like: `postgresql://postgres.[PROJECT-REF]:[PASSWORD]@aws-0-[REGION].pooler.supabase.com:6543/postgres`

3. **Configure Local Environment**
   - Create `.env.local` file (gitignored):
     ```bash
     DATABASE_URL=postgresql://postgres.[PROJECT-REF]:[PASSWORD]@aws-0-[REGION].pooler.supabase.com:6543/postgres
     ```
   - Replace `[PROJECT-REF]`, `[PASSWORD]`, and `[REGION]` with your actual values
   - Run `direnv allow` to load the environment

4. **Run Migrations**
   ```bash
   uv run python manage.py migrate
   ```

5. **Create Superuser**
   ```bash
   uv run python manage.py createsuperuser
   ```

## Notes

- Use the "Session pooler" connection string (port 6543) for development
- Use the "Direct connection" (port 5432) for production/long-running connections
- Connection strings are in Settings > Database > Connection string
- Don't commit `.env.local` (it's in .gitignore)
```

### Step 3: Settings Configuration

Split `config/settings.py` into modular files:

**config/settings/base.py** (core settings):

```python
"""Django settings for {project_name} project."""
import os
from pathlib import Path
import environ

# Build paths
BASE_DIR = Path(__file__).resolve().parent.parent.parent

# Environment variables
env = environ.Env()
environ.Env.read_env(os.path.join(BASE_DIR, '.env'))

# Security
SECRET_KEY = env('DJANGO_SECRET_KEY')

# Application definition
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    # Third-party apps
    'rest_framework',
    'oauth2_provider',
    'django_htmx',

    # Local apps
    'apps.core',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    'django_htmx.middleware.HtmxMiddleware',
]

ROOT_URLCONF = 'config.urls'

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [BASE_DIR / 'templates'],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

WSGI_APPLICATION = 'config.wsgi.application'

# Database
DATABASES = {
    'default': env.db('DATABASE_URL'),
}

# Custom User model
AUTH_USER_MODEL = 'core.User'

# Password validation
AUTH_PASSWORD_VALIDATORS = [
    {'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator'},
    {'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator'},
    {'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator'},
    {'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator'},
]

# Internationalization
LANGUAGE_CODE = 'en-us'
TIME_ZONE = 'UTC'
USE_I18N = True
USE_TZ = True

# Static files
STATIC_URL = 'static/'
STATIC_ROOT = BASE_DIR / 'staticfiles'
STATICFILES_DIRS = [BASE_DIR / 'static']

# Default primary key field type
DEFAULT_AUTO_FIELD = 'django.db.models.UUIDField'

# REST Framework
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'oauth2_provider.contrib.rest_framework.OAuth2Authentication',
        'rest_framework.authentication.SessionAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 100,
}

# OAuth2
OAUTH2_PROVIDER = {
    'SCOPES': {'read': 'Read scope', 'write': 'Write scope'},
    'ACCESS_TOKEN_EXPIRE_SECONDS': 36000,
}
```

**config/settings/dev.py** (development):

```python
"""Development settings."""
from .base import *

DEBUG = True

ALLOWED_HOSTS = ['localhost', '127.0.0.1', '[::1]']

# Development apps
INSTALLED_APPS += [
    'django_extensions',
]

# Console email backend
EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'
```

**config/settings/prod.py** (production):

```python
"""Production settings."""
from .base import *

DEBUG = False

ALLOWED_HOSTS = env.list('ALLOWED_HOSTS', default=[])

# Security settings
SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
SECURE_BROWSER_XSS_FILTER = True
SECURE_CONTENT_TYPE_NOSNIFF = True
X_FRAME_OPTIONS = 'DENY'

# Static files
STATICFILES_STORAGE = 'django.contrib.staticfiles.storage.ManifestStaticFilesStorage'
```

### Step 4: Create Custom User Model

**apps/core/models.py** (with full type annotations):

```python
"""Core models with UUID base class."""
import uuid
from django.contrib.auth.models import AbstractUser
from django.db import models


class UUIDModel(models.Model):
    """Abstract base class for models with UUID primary keys."""

    id: models.UUIDField = models.UUIDField(
        primary_key=True,
        default=uuid.uuid4,
        editable=False
    )
    created_at: models.DateTimeField = models.DateTimeField(auto_now_add=True)
    updated_at: models.DateTimeField = models.DateTimeField(auto_now=True)

    class Meta:
        abstract = True

    def __str__(self) -> str:
        return str(self.id)


class User(AbstractUser, UUIDModel):
    """Custom user model with UUID primary key."""

    email: models.EmailField = models.EmailField(unique=True)

    class Meta:
        db_table = 'users'
        ordering = ['-created_at']

    def __str__(self) -> str:
        return self.email
```

**apps/core/admin.py** (with type annotations):

```python
"""Admin configuration for core models."""
from typing import Any
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin as BaseUserAdmin
from django.http import HttpRequest
from .models import User


@admin.register(User)
class UserAdmin(BaseUserAdmin):
    """Admin for custom User model."""

    list_display: list[str] = ['email', 'username', 'is_staff', 'is_active', 'created_at']
    list_filter: list[str] = ['is_staff', 'is_active', 'created_at']
    search_fields: list[str] = ['email', 'username']
    ordering: list[str] = ['-created_at']
```

### Step 5: Create First App

Ask user: "What should be the first app to create?" (e.g., 'accounts', 'blog', 'api')

Then:

```bash
mkdir -p apps/{app_name}
uv run python manage.py startapp {app_name} apps/{app_name}
```

Update `apps/{app_name}/apps.py`:

```python
from django.apps import AppConfig


class {AppName}Config(AppConfig):
    default_auto_field = 'django.db.models.UUIDField'
    name = 'apps.{app_name}'
```

### Step 6: HTMX Integration

Add HTMX example view in first app:

**apps/{app_name}/views.py**:

```python
"""Example HTMX views."""
from django.shortcuts import render


def index(request):
    """Index view with HTMX support."""
    if request.htmx:
        return render(request, 'partials/content.html')
    return render(request, 'index.html')
```

**templates/index.html**:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{ project_name }}</title>
    <script src="https://unpkg.com/htmx.org@1.9.10"></script>
</head>
<body>
    <h1>Welcome to {{ project_name }}</h1>
    <div hx-get="/" hx-trigger="load" hx-swap="innerHTML">
        Loading...
    </div>
</body>
</html>
```

**templates/partials/content.html**:

```html
<div>
    <p>This content was loaded via HTMX!</p>
</div>
```

### Step 7: Testing Infrastructure

**pytest.ini**:

```ini
[pytest]
DJANGO_SETTINGS_MODULE = config.settings.dev
python_files = tests.py test_*.py *_tests.py
addopts = --reuse-db --cov=apps --cov-report=html --cov-report=term
testpaths = apps
```

**conftest.py** (with type annotations):

```python
"""Pytest configuration and fixtures."""
import pytest
from pytest_factoryboy import register
from rest_framework.test import APIClient
from apps.core.factories import UserFactory
from apps.core.models import User

# Register factories
register(UserFactory)


@pytest.fixture
def api_client() -> APIClient:
    """DRF API client."""
    return APIClient()


@pytest.fixture
def authenticated_client(api_client: APIClient, user: User) -> APIClient:
    """Authenticated API client."""
    api_client.force_authenticate(user=user)
    return api_client
```

**apps/core/factories.py** (with type annotations):

```python
"""Factory Boy factories for core models."""
from typing import Any
import factory
from factory.django import DjangoModelFactory
from .models import User


class UserFactory(DjangoModelFactory):
    """Factory for User model."""

    class Meta:
        model = User

    email: factory.Faker = factory.Faker('email')
    username: factory.Faker = factory.Faker('user_name')
    is_active: bool = True
    is_staff: bool = False
    is_superuser: bool = False

    @factory.post_generation
    def password(
        self,
        create: bool,
        extracted: str | None,
        **kwargs: Any
    ) -> None:
        """Set user password after generation."""
        if not create:
            return

        password = extracted or 'testpass123'
        self.set_password(password)
        self.save()
```

**apps/core/tests/test_models.py** (with type annotations):

```python
"""Tests for core models."""
from typing import Any
from uuid import UUID
import pytest
from apps.core.models import User


@pytest.mark.django_db
class TestUserModel:
    """Tests for User model."""

    def test_user_creation(self, user_factory: Any) -> None:
        """Test creating a user."""
        user: User = user_factory()
        assert user.id is not None
        assert user.email
        assert user.username

    def test_user_str(self, user_factory: Any) -> None:
        """Test user string representation."""
        user: User = user_factory(email="test@example.com")
        assert str(user) == "test@example.com"

    def test_user_uuid_primary_key(self, user_factory: Any) -> None:
        """Test user has UUID primary key."""
        user: User = user_factory()
        assert isinstance(user.id, UUID)
```

### Step 8: Type Checking & Linting

Add to **pyproject.toml** (with strict typing enabled):

```toml
[tool.mypy]
plugins = ["mypy_django_plugin.main"]
python_version = "3.12"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true  # Strict: all functions must have type annotations
warn_redundant_casts = true
warn_unused_ignores = true
strict_equality = true
check_untyped_defs = true
exclude = [
    "migrations/",
    "venv/",
    ".venv/",
]

[[tool.mypy.overrides]]
module = [
    "factory.*",
    "environ.*",
    "oauth2_provider.*",
    "rest_framework.*",
]
ignore_missing_imports = true

[tool.django-stubs]
django_settings_module = "config.settings.dev"

[tool.ruff]
line-length = 100
target-version = "py312"
exclude = [
    "migrations",
    ".venv",
    "venv",
]

[tool.ruff.lint]
select = ["E", "F", "I", "N", "W", "UP"]
ignore = ["E501"]

[tool.ruff.lint.per-file-ignores]
"__init__.py" = ["F401"]
"*/migrations/*" = ["E501", "N806"]
"*/tests/*" = ["F401", "F811"]
"*/admin.py" = ["F401"]
"*/models.py" = ["F401"]
"*/views.py" = ["F401"]

[tool.pytest.ini_options]
DJANGO_SETTINGS_MODULE = "config.settings.dev"
python_files = ["tests.py", "test_*.py", "*_tests.py"]
addopts = "--reuse-db --cov=apps --cov-report=html --cov-report=term"
testpaths = ["apps"]
```

**Note**: `disallow_untyped_defs = true` enforces that all functions and methods must have type annotations. This is intentional to ensure maximum compatibility with AI coding assistants.

### Step 9: Update URLs

**config/urls.py**:

```python
"""URL configuration."""
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('oauth/', include('oauth2_provider.urls', namespace='oauth2_provider')),
    path('api-auth/', include('rest_framework.urls')),
]
```

### Step 10: Create .gitignore

```gitignore
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
*.egg-info/
.installed.cfg
*.egg

# Virtual environments
.venv/
venv/
ENV/
env/

# Django
*.log
db.sqlite3
db.sqlite3-journal
/static/
/staticfiles/
/media/

# Environment variables
.env
.envrc.local

# Testing
.pytest_cache/
.coverage
htmlcov/
.tox/

# IDEs
.vscode/
.idea/
*.swp
*.swo
*~

# OS
.DS_Store
Thumbs.db

# Docker
postgres_data/
```

### Step 11: Create CLAUDE.md

See template in `templates/CLAUDE.md.template`.

### Step 12: Install Dependencies

```bash
# Core dependencies
uv add django psycopg[binary] django-environ

# Third-party apps
uv add djangorestframework django-oauth-toolkit django-htmx pydantic

# Development tools
uv add --group dev pytest pytest-django pytest-cov pytest-asyncio
uv add --group dev factory-boy pytest-factoryboy
uv add --group dev mypy django-stubs types-requests
uv add --group dev ruff
uv add --group dev django-extensions ipython
```

### Step 13: Run Initial Migrations

```bash
# Create migrations for core app
uv run python manage.py makemigrations core

# Run migrations
uv run python manage.py migrate

# Create superuser
export DJANGO_SUPERUSER_PASSWORD=admin
uv run python manage.py createsuperuser --noinput --username admin --email admin@example.com
```

## Verification Steps

After setup, verify:

1. ✅ **Project Structure**: Correct `config/` and `apps/` layout
2. ✅ **Environment**: `direnv allow` succeeds
3. ✅ **Database**:
   - **Docker**: Container running (`docker ps` shows postgres)
   - **Supabase**: DATABASE_URL configured in `.env.local`
4. ✅ **Migrations**: Applied successfully (or Supabase setup instructions provided)
5. ✅ **Superuser**: Created and can log in to admin
6. ✅ **Dev Server**: `uv run python manage.py runserver` starts
7. ✅ **Admin**: http://localhost:8000/admin/ accessible
8. ✅ **Tests**: `uv run pytest` passes
9. ✅ **Type Check**: `uv run mypy .` passes with strict mode (`disallow_untyped_defs = true`)
10. ✅ **Linting**: `uv run ruff check .` passes
11. ✅ **HTMX**: Basic view works
12. ✅ **OAuth**: `/oauth/` endpoint accessible
13. ✅ **AI Tooling**: All generated files have comprehensive type annotations

## Related Skills

- `/python-experts:django-dev` - Django development patterns
- `/python-experts:django-api` - API development with DRF
- `/devops-data:direnv` - Environment management
- `/git-workflow:commit` - Commit the initial project

## References

- [Django 6.0 Documentation](https://docs.djangoproject.com/en/6.0/)
- [uv Package Manager](https://github.com/astral-sh/uv)
- [uv Python Version Management](https://docs.astral.sh/uv/concepts/python-versions/)
- [Supabase Database](https://supabase.com/docs/guides/database)
- [django-htmx](https://django-htmx.readthedocs.io/)
- [django-oauth-toolkit](https://django-oauth-toolkit.readthedocs.io/)
- [Django REST Framework](https://www.django-rest-framework.org/)
- [Factory Boy](https://factoryboy.readthedocs.io/)
- [pytest-django](https://pytest-django.readthedocs.io/)
