---
name: django_devops
description: Enterprise Django 6 DevOps — containerization, CI/CD, environment management, database deployment, monitoring, and production readiness checklist.
---

# Django DevOps Skill

## Purpose

Define and enforce DevOps standards for building, deploying, monitoring, and operating enterprise Django 6 systems with Strawberry GraphQL, SSE, Celery, Redis, and PostgreSQL.

## Scope

- Containerization (Docker)
- CI/CD pipeline design
- Environment management
- Database migration deployment
- Health checks and readiness probes
- Logging and monitoring
- Secrets management in deployments
- Scaling and resource allocation
- Backup and disaster recovery

## Responsibilities

1. ENFORCE containerized deployments.
2. ENFORCE CI/CD with automated testing and migration validation.
3. ENFORCE environment parity across development, staging, and production.
4. ENFORCE structured logging and monitoring.
5. ENFORCE health check endpoints for orchestrators.
6. PREVENT manual deployments without CI/CD.
7. PREVENT unreviewed migrations in production.
8. PREVENT secret leakage in CI/CD logs or Docker images.

---

## Mandatory Rules

### ALWAYS

1. ALWAYS containerize the application with a multi-stage Dockerfile:
   ```dockerfile
   # Stage 1: Builder
   FROM python:3.12-slim AS builder
   WORKDIR /app
   COPY requirements.txt .
   RUN pip install --no-cache-dir --prefix=/install -r requirements.txt

   # Stage 2: Runtime
   FROM python:3.12-slim
   WORKDIR /app
   COPY --from=builder /install /usr/local
   COPY . .
   RUN python manage.py collectstatic --noinput
   EXPOSE 8000
   CMD ["gunicorn", "CMS.asgi:application", "-k", "uvicorn.workers.UvicornWorker", "--bind", "0.0.0.0:8000"]
   ```
2. ALWAYS use ASGI server (Uvicorn/Daphne/Hypercorn) for production. Never use Django's `runserver`.
3. ALWAYS implement health check endpoints:
   ```python
   # urls.py
   path('health/', HealthCheckView.as_view(), name='health-check'),
   path('health/ready/', ReadinessCheckView.as_view(), name='readiness-check'),
   ```
   - **Liveness** (`/health/`): Returns 200 if the process is alive.
   - **Readiness** (`/health/ready/`): Returns 200 if DB, Redis, and Celery are reachable.
4. ALWAYS run migrations as a separate step in deployment, BEFORE starting new application instances:
   ```yaml
   # CI/CD Pipeline
   steps:
     - name: Run migrations
       run: python manage.py migrate --noinput
     - name: Deploy new version
       run: kubectl rollout restart deployment/cms-api
   ```
5. ALWAYS validate migrations in CI before deployment:
   ```bash
   python manage.py makemigrations --check --dry-run  # Fail if unapplied migrations exist
   python manage.py migrate --plan                     # Show migration plan
   ```
6. ALWAYS use environment variables for all configuration. Never bake configuration into Docker images.
7. ALWAYS use `.dockerignore` to exclude `.git`, `__pycache__`, `.env`, `node_modules`, media files.
8. ALWAYS tag Docker images with the Git commit SHA, not `latest`:
   ```bash
   docker build -t cms-backend:$(git rev-parse --short HEAD) .
   ```
9. ALWAYS use structured JSON logging in production:
   ```python
   LOGGING = {
       'version': 1,
       'formatters': {
           'json': {
               '()': 'pythonjsonlogger.jsonlogger.JsonFormatter',
               'format': '%(asctime)s %(name)s %(levelname)s %(message)s',
           },
       },
       'handlers': {
           'console': {
               'class': 'logging.StreamHandler',
               'formatter': 'json',
           },
       },
       'root': {
           'handlers': ['console'],
           'level': 'INFO',
       },
   }
   ```
10. ALWAYS monitor these metrics:
    - Request latency (p50, p95, p99)
    - Error rate (5xx responses)
    - Database query count and latency
    - Redis connection pool usage
    - Celery task queue depth and processing time
    - SSE active connection count
    - Memory and CPU usage per container
11. ALWAYS define resource limits for containers:
    ```yaml
    resources:
      requests:
        memory: "256Mi"
        cpu: "250m"
      limits:
        memory: "512Mi"
        cpu: "500m"
    ```
12. ALWAYS implement graceful shutdown — handle SIGTERM to finish in-flight requests:
    ```python
    # Uvicorn handles this natively with --timeout-graceful-shutdown
    CMD ["uvicorn", "CMS.asgi:application", "--host", "0.0.0.0", "--port", "8000", "--timeout-graceful-shutdown", "30"]
    ```

### NEVER

1. NEVER use `runserver` in production.
2. NEVER bake secrets into Docker images.
3. NEVER deploy without running the test suite in CI.
4. NEVER run migrations and application startup in the same process simultaneously.
5. NEVER use `DEBUG=True` in production.
6. NEVER expose database ports to the public internet.
7. NEVER use `latest` tag for Docker images in production deployments.
8. NEVER skip health checks in orchestrator configuration.
9. NEVER log request/response bodies in production (PII risk).
10. NEVER store persistent data in container filesystems (ephemeral).

---

## CI/CD Pipeline

### Pipeline Stages

```
1. Lint        → ruff check, ruff format --check
2. Test        → pytest with coverage
3. Migration   → makemigrations --check, migrate --plan
4. Build       → Docker image build
5. Security    → Dependency vulnerability scan (pip-audit, safety)
6. Push        → Push image to registry
7. Deploy      → Apply migrations → Rolling update
8. Smoke Test  → Hit health endpoints on new deployment
```

### CI Configuration Example

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_DB: cms_test
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
        ports: ["5432:5432"]
      redis:
        image: redis:7
        ports: ["6379:6379"]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
      - run: pip install -r requirements.txt
      - run: ruff check .
      - run: ruff format --check .
      - run: python manage.py makemigrations --check --dry-run
      - run: pytest --cov --cov-fail-under=80
```

---

## Database Deployment Rules

1. Run migrations in a dedicated init container or CI step, not in the application process.
2. Use `--plan` to preview migrations before applying.
3. For zero-downtime migrations, follow the expand-contract pattern:
   - Expand: Add new column (nullable), deploy code that writes to both old and new.
   - Migrate: Backfill data.
   - Contract: Remove old column, deploy code that uses only new.
4. Never run destructive migrations (RemoveField, DeleteModel) without a maintenance window or expand-contract strategy.

---

## Environment Management

```
.env.example        # Template with all required vars (committed)
.env                 # Local development values (NOT committed)
.env.staging         # Staging values (NOT committed, in CI secrets)
.env.production      # Production values (NOT committed, in secret manager)
```

Required environment variables:
```
SECRET_KEY=
DEBUG=
DATABASE_URL=
REDIS_URL=
ALLOWED_HOSTS=
CORS_ALLOWED_ORIGINS=
CELERY_BROKER_URL=
```

---

## Security Considerations

1. Scan dependencies for vulnerabilities in CI: `pip-audit`, `safety check`.
2. Use non-root user in Docker containers.
3. Run containers with read-only filesystem where possible.
4. Use network policies to restrict container-to-container communication.
5. Rotate secrets regularly.

---

## Performance Considerations

1. Use Gunicorn with Uvicorn workers for ASGI: `gunicorn -k uvicorn.workers.UvicornWorker -w 4`.
2. Worker count formula: `(2 × CPU cores) + 1` for CPU-bound, higher for I/O-bound.
3. Use PgBouncer for database connection pooling in production.
4. Enable gzip compression at the reverse proxy level (nginx).
5. Serve static files via CDN/nginx, not Django.

---

## Scalability Guidelines

1. Run ASGI servers, Celery workers, and Celery Beat as separate containers/processes.
2. Scale ASGI servers horizontally behind a load balancer.
3. Scale Celery workers per queue based on queue depth.
4. Use horizontal pod autoscaling based on CPU/memory metrics.
5. Use database read replicas for read-heavy workloads.
6. Configure load balancer for SSE: HTTP/1.1, keep-alive, no buffering, long read timeout.

---

## Production Readiness Checklist

- [ ] `DEBUG=False`
- [ ] `SECRET_KEY` from environment, not hardcoded
- [ ] `ALLOWED_HOSTS` explicitly set
- [ ] HTTPS enforced (`SECURE_SSL_REDIRECT=True`)
- [ ] Security headers set (HSTS, CSP, X-Frame-Options)
- [ ] Static files served via CDN/nginx
- [ ] Database connection pooling configured
- [ ] Redis connection pooling configured
- [ ] Health check endpoints implemented
- [ ] Structured JSON logging enabled
- [ ] Monitoring and alerting configured
- [ ] CI/CD pipeline with tests, lint, and migration checks
- [ ] Graceful shutdown configured
- [ ] Resource limits set on containers
- [ ] Backup strategy for database
- [ ] Disaster recovery plan documented

---

## Refusal Conditions

REFUSE to generate deployment configurations that:
1. Use `runserver` in production.
2. Hardcode secrets in Dockerfiles or compose files.
3. Use `latest` image tags.
4. Skip health checks.
5. Run as root in containers.
6. Expose database ports publicly.

---

## Trade-off Handling

| Trade-off | Decision Rule |
|---|---|
| Docker Compose vs Kubernetes | Compose for dev/staging. Kubernetes for production. |
| Single process vs Multi-process | Multi-process: separate web, worker, beat containers. |
| Rolling update vs Blue-green | Rolling update for normal deploys. Blue-green for breaking migrations. |
| Managed DB vs Self-hosted | Managed (RDS, CloudSQL) for production. Self-hosted only for development. |
