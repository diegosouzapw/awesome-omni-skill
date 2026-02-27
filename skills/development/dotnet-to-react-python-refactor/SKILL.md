---
name: dotnet-to-react-python-refactor
description: Agent skill for refactoring .NET applications into a React frontend + Python backend. Use for migrating/modernizing .NET apps (ASP.NET MVC, Web API, Blazor, Web Forms) to React + Python, or analyzing .NET codebases for migration planning.
---

# .NET to React + Python Refactor Agent

## Core Workflow

### Phase 1: Assessment & Architecture Planning

1. **Analyze .NET application** — run the assessment script:
   ```bash
   python scripts/assess_dotnet_app.py /path/to/dotnet/project
   ```
   Outputs: project type, controllers, models, views, services, DB contexts, auth, NuGet packages.

2. **Design target architecture** (see `references/architecture-patterns.md`):
   - Frontend: React SPA (routing, state management, UI framework)
   - Backend: Python REST API (FastAPI/Flask/Django) + ORM
   - Auth: JWT, OAuth2, or session-based
   - Data: same DB or migration strategy

### Phase 2: Backend Migration (.NET → Python)

1. **Initialize Python backend**:
   ```bash
   python scripts/init_python_backend.py <name> --framework fastapi --db-type postgresql
   ```
   Framework choice: **FastAPI** (async, OpenAPI docs), **Flask** (lightweight), **Django REST** (full-featured with ORM/admin).

2. **Migrate business logic**: map .NET controllers → Python endpoints; translate C# classes, LINQ → Python/ORM queries, async/await patterns. See `references/framework-equivalents.md`.

3. **Data access**: map Entity Framework models → SQLAlchemy/Django ORM; convert queries and stored procedures. See `references/orm-migration.md`.

4. **Auth**: map ASP.NET Identity → JWT tokens or session auth; preserve role-based/policy authorization. See `references/authentication-patterns.md`.

### Phase 3: Frontend Migration (.NET Views → React)

1. **Initialize React** (Vite recommended):
   ```bash
   npm create vite@latest frontend -- --template react-ts
   ```
   See `assets/react-project-template/` for recommended structure.

2. **Convert Razor views** to JSX (semi-automated):
   ```bash
   python scripts/convert_razor_to_jsx.py ./Views --output ./frontend/src/components
   ```
   Use `assets/react-component-templates/` for production-ready Button, Input, and Modal components.

3. **State management**: Context API (small apps), Zustand (medium), Redux Toolkit (large); convert ViewBag/ViewData to component or global state.

4. **Routing**: replace ASP.NET routing with React Router; implement protected routes for authenticated pages.

5. **API integration**: React Query + axios with auth interceptors. See `references/api-integration.md`.

6. **Forms**: React Hook Form or Formik; client-side validation matching backend rules.

### Phase 4: Data Layer Migration

1. **Generate ORM models**:
   ```bash
   python scripts/generate_migration.py /path/to/models --framework sqlalchemy --output ./migrations
   ```

2. **Preserve** relationships, constraints, indexes, and custom type mappings.

3. **Apply migrations**: Alembic (SQLAlchemy) or Django migrations.

### Phase 5: Testing & Validation

- **Backend** (pytest): unit tests, endpoint integration tests, auth flows, error handling.
- **Frontend** (Jest + RTL): component tests; Cypress/Playwright for E2E.
- **Data integrity**: compare query results between .NET and Python; validate CRUD and cascading deletes.

### Phase 6: Deployment

1. **Environment**: configure env vars, secrets, and per-environment configs (dev/staging/prod).
2. **Backend**: Gunicorn/Uvicorn + Nginx reverse proxy; health checks and DB connection pooling.
3. **Frontend**: production build → static hosting (Netlify, Vercel, S3+CloudFront); CDN for assets.
4. **Cutover strategy**: Big Bang (fast, higher risk) | Strangler Fig (incremental, lower risk) | Parallel Run.
5. See `references/deployment-guides/` for step-by-step Docker Compose and AWS guides. Use `assets/docker-compose.yml` and `assets/nginx.conf`.

## Execution Guidelines

**Order**: backend first → frontend as endpoints become available → staging validation → cutover with rollback plan.

**Key pitfalls**:
- N+1 queries: Python ORM lazy loading differs from Entity Framework
- Async/await: C# and Python semantics differ
- Timezones: standardize on UTC
- Decimal precision: map carefully for financial calculations

**Standards**: PEP 8 + type hints + mypy (Python); ESLint + Prettier + TypeScript (React); 80%+ test coverage.

**Performance**: profile DB queries early; add Redis caching; paginate large datasets; use React.memo/useMemo and lazy loading.

## Quick Reference

### Scripts

| Script | Purpose |
|--------|---------|
| `assess_dotnet_app.py` | Generate migration inventory from .NET project |
| `init_python_backend.py` | Scaffold Python backend (FastAPI/Flask/Django) |
| `generate_migration.py` | Create ORM models from .NET Entity Framework models |
| `convert_razor_to_jsx.py` | Convert Razor views to React JSX |

### Reference Docs (`references/`)

| File | Use When |
|------|---------|
| `architecture-patterns.md` | Planning architecture or choosing tech stack |
| `framework-equivalents.md` | Mapping .NET → Python technologies and packages |
| `orm-migration.md` | Entity Framework → SQLAlchemy/Django ORM |
| `authentication-patterns.md` | Implementing auth migration |
| `api-integration.md` | React–Python API client patterns |
| `testing-strategies.md` | Setting up tests and coverage |
| `security-hardening-checklist.md` | Pre-production security audit |
| `troubleshooting-guide.md` | Debugging migration and deployment issues |
| `deployment-guides/docker-compose-deployment.md` | Docker Compose deployment |
| `deployment-guides/aws-deployment.md` | AWS deployment (Elastic Beanstalk, EC2, ECS) |

### Assets (`assets/`)

| Asset | Purpose |
|-------|---------|
| `react-project-template/` | React project structure guide |
| `react-component-templates/` | Button, Input, Modal components with CSS |
| `docker-compose.yml` | Full-stack Docker setup (PostgreSQL, Redis, Nginx) |
| `nginx.conf` | Production Nginx reverse proxy |
| `Dockerfile.dev` | Development frontend Dockerfile with hot-reload |
| `.env.example` | Environment variables template |
| `cicd-templates/` | GitHub Actions and GitLab CI/CD pipelines |
