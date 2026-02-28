---
name: DevOps & Deployment
description: CI/CD pipelines, containerization, Kubernetes, and infrastructure as code patterns
version: 1.0.0
category: Infrastructure & Deployment
agents: [backend-system-architect, code-quality-reviewer, studio-coach]
keywords: [CI/CD, deployment, Docker, Kubernetes, pipeline, infrastructure, GitOps, container, automation, release]
---

# DevOps & Deployment Skill

Comprehensive frameworks for CI/CD pipelines, containerization, deployment strategies, and infrastructure automation.

## When to Use

- Setting up CI/CD pipelines
- Containerizing applications
- Deploying to Kubernetes or cloud platforms
- Implementing GitOps workflows
- Managing infrastructure as code
- Planning release strategies

## Pipeline Architecture

```
┌─────────────┐   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐
│    Code     │──▶│    Build    │──▶│    Test     │──▶│   Deploy    │
│   Commit    │   │   & Lint    │   │   & Scan    │   │  & Release  │
└─────────────┘   └─────────────┘   └─────────────┘   └─────────────┘
       │                 │                 │                 │
       ▼                 ▼                 ▼                 ▼
   Triggers         Artifacts          Reports          Monitoring
```

## Key Concepts

### CI/CD Pipeline Stages

1. **Lint & Type Check** - Code quality gates
2. **Unit Tests** - Test coverage with reporting
3. **Security Scan** - npm audit + Trivy vulnerability scanner
4. **Build & Push** - Docker image to container registry
5. **Deploy Staging** - Environment-gated deployment
6. **Deploy Production** - Manual approval or automated

> See `templates/github-actions-pipeline.yml` for complete GitHub Actions workflow

### Container Best Practices

**Multi-stage builds** minimize image size:
- Stage 1: Install production dependencies only
- Stage 2: Build application with dev dependencies
- Stage 3: Production runtime with minimal footprint

**Security hardening**:
- Non-root user (uid 1001)
- Read-only filesystem where possible
- Health checks for orchestrator integration

> See `templates/Dockerfile` and `templates/docker-compose.yml`

### Kubernetes Deployment

**Essential manifests**:
- Deployment with rolling update strategy
- Service for internal routing
- Ingress for external access with TLS
- HorizontalPodAutoscaler for scaling

**Security context**:
- `runAsNonRoot: true`
- `allowPrivilegeEscalation: false`
- `readOnlyRootFilesystem: true`
- Drop all capabilities

**Resource management**:
- Always set requests and limits
- Use `requests` for scheduling, `limits` for throttling

> See `templates/k8s-manifests.yaml` and `templates/helm-values.yaml`

### Deployment Strategies

| Strategy | Use Case | Risk |
|----------|----------|------|
| **Rolling** | Default, gradual replacement | Low - automatic rollback |
| **Blue-Green** | Instant switch, easy rollback | Medium - double resources |
| **Canary** | Progressive traffic shift | Low - gradual exposure |

**Rolling Update** (Kubernetes default):
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 25%
    maxUnavailable: 0  # Zero downtime
```

**Blue-Green**: Deploy to standby environment, switch service selector
**Canary**: Use Istio VirtualService for traffic splitting (10% → 50% → 100%)

### Infrastructure as Code

**Terraform patterns**:
- Remote state in S3 with DynamoDB locking
- Module-based architecture (VPC, EKS, RDS)
- Environment-specific tfvars files

> See `templates/terraform-aws.tf` for AWS VPC + EKS + RDS example

### GitOps with ArgoCD

ArgoCD watches Git repository and syncs cluster state:
- Automated sync with pruning
- Self-healing (drift detection)
- Retry policies for transient failures

> See `templates/argocd-application.yaml`

### Secrets Management

Use External Secrets Operator to sync from cloud providers:
- AWS Secrets Manager
- HashiCorp Vault
- Azure Key Vault
- GCP Secret Manager

> See `templates/external-secrets.yaml`

## Deployment Checklist

### Pre-Deployment
- [ ] All tests passing in CI
- [ ] Security scans clean
- [ ] Database migrations ready
- [ ] Rollback plan documented

### During Deployment
- [ ] Monitor deployment progress
- [ ] Watch error rates
- [ ] Verify health checks passing

### Post-Deployment
- [ ] Verify metrics normal
- [ ] Check logs for errors
- [ ] Update status page

## Helm Chart Structure

```
charts/app/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   ├── secret.yaml
│   ├── hpa.yaml
│   └── _helpers.tpl
└── values/
    ├── staging.yaml
    └── production.yaml
```

---

## CI/CD Pipeline Patterns

### Branch Strategy

**Recommended: Git Flow with Feature Branches**
```
main (production) ─────●────────●──────▶
                       ┃        ┃
dev (staging) ─────●───●────●───●──────▶
                   ┃        ┃
feature/* ─────────●────────┘
                   ▲
                   └─ PR required, CI checks, code review
```

**Branch protection rules:**
- `main`: Require PR + 2 approvals + all checks pass
- `dev`: Require PR + 1 approval + all checks pass
- Feature branches: No direct commits to main/dev

### GitHub Actions Caching Strategy

```yaml
- name: Cache Dependencies
  uses: actions/cache@v3
  with:
    path: |
      ~/.npm
      node_modules
      backend/.venv
    key: ${{ runner.os }}-deps-${{ hashFiles('**/package-lock.json', '**/poetry.lock') }}
    restore-keys: |
      ${{ runner.os }}-deps-
```

**Cache hit ratio impact:**
- Without cache: 2-3 min install time
- With cache: 10-20 sec install time
- **~85% time savings** on typical workflows

### Artifact Management

```yaml
# Build and upload artifact
- name: Build Application
  run: npm run build

- name: Upload Build Artifact
  uses: actions/upload-artifact@v3
  with:
    name: build-${{ github.sha }}
    path: dist/
    retention-days: 7

# Download in deployment job
- name: Download Build Artifact
  uses: actions/download-artifact@v3
  with:
    name: build-${{ github.sha }}
    path: dist/
```

**Benefits:**
- Avoid rebuilding in deployment job
- Deploy exact tested artifact (byte-for-byte match)
- Retention policies prevent storage bloat

### Matrix Testing

```yaml
strategy:
  matrix:
    node-version: [18, 20, 22]
    os: [ubuntu-latest, windows-latest]
jobs:
  test:
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm test
```

---

## Container Optimization Deep Dive

### Multi-Stage Build Example

```dockerfile
# ============================================================
# Stage 1: Dependencies (builder)
# ============================================================
FROM node:20-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

# ============================================================
# Stage 2: Build (with dev dependencies)
# ============================================================
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci  # Include dev dependencies
COPY . .
RUN npm run build && npm run test

# ============================================================
# Stage 3: Production runtime (minimal)
# ============================================================
FROM node:20-alpine AS runner
WORKDIR /app

# Security: Non-root user
RUN addgroup -g 1001 -S nodejs && adduser -S nodejs -u 1001

# Copy only production dependencies and built artifacts
COPY --from=deps --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --chown=nodejs:nodejs package*.json ./

USER nodejs
EXPOSE 3000
ENV NODE_ENV=production
HEALTHCHECK --interval=30s --timeout=3s CMD node healthcheck.js || exit 1
CMD ["node", "dist/main.js"]
```

**Image size comparison:**
- Single-stage: **850 MB** (includes dev dependencies, source files)
- Multi-stage: **180 MB** (only runtime + production deps)
- **78% reduction**

### Layer Caching Optimization

**Order matters for cache efficiency:**
```dockerfile
# ❌ BAD: Invalidates cache on any code change
COPY . .
RUN npm install

# ✅ GOOD: Cache package.json layer separately
COPY package*.json ./
RUN npm ci  # Cached unless package.json changes
COPY . .    # Source changes don't invalidate npm install
```

### Security Scanning with Trivy

```yaml
- name: Build Docker Image
  run: docker build -t myapp:${{ github.sha }} .

- name: Scan for Vulnerabilities
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: 'myapp:${{ github.sha }}'
    format: 'sarif'
    output: 'trivy-results.sarif'
    severity: 'CRITICAL,HIGH'

- name: Upload Scan Results
  uses: github/codeql-action/upload-sarif@v2
  with:
    sarif_file: 'trivy-results.sarif'

- name: Fail on Critical Vulnerabilities
  run: |
    trivy image --severity CRITICAL --exit-code 1 myapp:${{ github.sha }}
```

---

## Kubernetes Production Patterns

### Health Probes

**Three probe types with distinct purposes:**

```yaml
spec:
  containers:
  - name: app
    # Startup probe (gives slow-starting apps time to boot)
    startupProbe:
      httpGet:
        path: /health/startup
        port: 8080
      initialDelaySeconds: 0
      periodSeconds: 5
      failureThreshold: 30  # 30 * 5s = 150s max startup time

    # Liveness probe (restarts pod if failing)
    livenessProbe:
      httpGet:
        path: /health/liveness
        port: 8080
      initialDelaySeconds: 60
      periodSeconds: 10
      failureThreshold: 3  # 3 failures = restart

    # Readiness probe (removes from service if failing)
    readinessProbe:
      httpGet:
        path: /health/readiness
        port: 8080
      initialDelaySeconds: 10
      periodSeconds: 5
      failureThreshold: 2  # 2 failures = remove from load balancer
```

**Probe implementation:**
```python
@app.get("/health/startup")
async def startup_check():
    # Check DB connection established
    if not db.is_connected():
        raise HTTPException(status_code=503, detail="DB not ready")
    return {"status": "ok"}

@app.get("/health/liveness")
async def liveness_check():
    # Basic "is process running" check
    return {"status": "alive"}

@app.get("/health/readiness")
async def readiness_check():
    # Check all dependencies healthy
    if not redis.ping() or not db.health_check():
        raise HTTPException(status_code=503, detail="Dependencies unhealthy")
    return {"status": "ready"}
```

### PodDisruptionBudget

Prevents too many pods from being evicted during node maintenance:

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: app-pdb
spec:
  minAvailable: 2  # Always keep at least 2 pods running
  selector:
    matchLabels:
      app: myapp
```

**Use cases:**
- Cluster upgrades (node drains)
- Autoscaler downscaling
- Manual evictions

### Resource Quotas

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-quota
  namespace: production
spec:
  hard:
    requests.cpu: "10"      # Total CPU requests
    requests.memory: 20Gi   # Total memory requests
    limits.cpu: "20"        # Total CPU limits
    limits.memory: 40Gi     # Total memory limits
    pods: "50"              # Max pods
```

### StatefulSets for Databases

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres
  replicas: 3
  selector:
    matchLabels:
      app: postgres
  template:
    # Pod spec here
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 100Gi
```

**Key differences from Deployment:**
- Stable pod names (`postgres-0`, `postgres-1`, `postgres-2`)
- Ordered deployment and scaling
- Persistent storage per pod

---

## Database Migration Strategies

### Zero-Downtime Migration Pattern

**Problem:** Adding a NOT NULL column breaks old application versions

**Solution: 3-phase migration**

**Phase 1: Add nullable column**
```sql
-- Migration v1 (deploy with old code still running)
ALTER TABLE users ADD COLUMN email VARCHAR(255);
```

**Phase 2: Deploy new code + backfill**
```python
# New code writes to both old and new schema
def create_user(name: str, email: str):
    # Write to new column
    db.execute("INSERT INTO users (name, email) VALUES (%s, %s)", (name, email))

# Backfill existing rows
async def backfill_emails():
    users_without_email = await db.fetch("SELECT id FROM users WHERE email IS NULL")
    for user in users_without_email:
        email = generate_email(user.id)
        await db.execute("UPDATE users SET email = %s WHERE id = %s", (email, user.id))
```

**Phase 3: Add constraint**
```sql
-- Migration v2 (after backfill complete)
ALTER TABLE users ALTER COLUMN email SET NOT NULL;
```

### Backward/Forward Compatibility

**Backward compatible changes (safe):**
- ✅ Add nullable column
- ✅ Add table
- ✅ Add index
- ✅ Rename column (with view alias)

**Backward incompatible changes (requires 3-phase):**
- ❌ Remove column
- ❌ Rename column (no alias)
- ❌ Add NOT NULL column
- ❌ Change column type

### Rollback Procedures

```yaml
# Helm rollback to previous revision
helm rollback myapp 3

# Kubernetes rollback
kubectl rollout undo deployment/myapp

# Database migration rollback (Alembic example)
alembic downgrade -1
```

**Critical: Test rollback procedures regularly!**

---

## Observability & Monitoring

### Prometheus Metrics Exposition

```python
from prometheus_client import Counter, Histogram, generate_latest

# Define metrics
http_requests_total = Counter(
    'http_requests_total',
    'Total HTTP requests',
    ['method', 'endpoint', 'status']
)

http_request_duration_seconds = Histogram(
    'http_request_duration_seconds',
    'HTTP request duration',
    ['method', 'endpoint']
)

@app.middleware("http")
async def prometheus_middleware(request: Request, call_next):
    start_time = time.time()
    response = await call_next(request)
    duration = time.time() - start_time

    # Record metrics
    http_requests_total.labels(
        method=request.method,
        endpoint=request.url.path,
        status=response.status_code
    ).inc()

    http_request_duration_seconds.labels(
        method=request.method,
        endpoint=request.url.path
    ).observe(duration)

    return response

@app.get("/metrics")
async def metrics():
    return Response(content=generate_latest(), media_type="text/plain")
```

### Grafana Dashboard Queries

```promql
# Request rate (requests per second)
rate(http_requests_total[5m])

# Error rate (4xx/5xx as percentage)
sum(rate(http_requests_total{status=~"4..|5.."}[5m])) /
sum(rate(http_requests_total[5m])) * 100

# p95 latency
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))

# Pod CPU usage
sum(rate(container_cpu_usage_seconds_total{pod=~"myapp-.*"}[5m])) by (pod)
```

### Alerting Rules

```yaml
groups:
- name: app-alerts
  rules:
  - alert: HighErrorRate
    expr: |
      sum(rate(http_requests_total{status=~"5.."}[5m])) /
      sum(rate(http_requests_total[5m])) > 0.05
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "High error rate detected"
      description: "Error rate is {{ $value | humanizePercentage }}"

  - alert: HighLatency
    expr: |
      histogram_quantile(0.95,
        rate(http_request_duration_seconds_bucket[5m])
      ) > 2
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "High p95 latency detected"
      description: "p95 latency is {{ $value }}s"
```

---

## Real-World Examples

### Example 1: E-commerce Platform with Docker Compose

**Full-stack e-commerce development environment:**
```yaml
version: '3.8'
services:
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: shopuser
      POSTGRES_PASSWORD: dev_password
      POSTGRES_DB: shop_dev
    ports:
      - "5437:5432"  # Avoid conflict with host postgres
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U shopuser"]
      interval: 5s
      timeout: 3s
      retries: 5

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    command: redis-server --appendonly yes --maxmemory 512mb --maxmemory-policy allkeys-lru
    volumes:
      - redisdata:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile.dev
    ports:
      - "8000:8000"
    environment:
      DATABASE_URL: postgresql://shopuser:dev_password@postgres:5432/shop_dev
      REDIS_URL: redis://redis:6379
      STRIPE_SECRET_KEY: ${STRIPE_SECRET_KEY}
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    volumes:
      - ./backend:/app  # Hot reload
      - /app/.venv  # Persist virtualenv

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    environment:
      VITE_API_URL: http://localhost:8000
      VITE_STRIPE_PUBLIC_KEY: ${VITE_STRIPE_PUBLIC_KEY}
    volumes:
      - ./frontend:/app
      - /app/node_modules  # Avoid overwriting node_modules

  worker:
    build:
      context: ./backend
      dockerfile: Dockerfile.dev
    command: celery -A app.worker worker --loglevel=info
    environment:
      DATABASE_URL: postgresql://shopuser:dev_password@postgres:5432/shop_dev
      REDIS_URL: redis://redis:6379
    depends_on:
      - postgres
      - redis
    volumes:
      - ./backend:/app

volumes:
  pgdata:
  redisdata:
```

**Key patterns:**
- Port mapping to avoid host conflicts (5437:5432)
- Health checks before dependent services start
- Volume mounts for hot reload during development
- Named volumes for data persistence

### Example 2: GitHub Actions CI/CD Pipeline (2025)

**Modern FastAPI backend with Ruff and uv:**
```yaml
name: Backend CI/CD

on:
  push:
    branches: [main, dev]
    paths: ['backend/**']
  pull_request:
    branches: [main, dev]
    paths: ['backend/**']

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  lint-and-test:
    runs-on: ubuntu-24.04-latest  # Latest LTS runner
    defaults:
      run:
        working-directory: backend

    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_PASSWORD: test
          POSTGRES_DB: test_db
        ports: ["5432:5432"]
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

      redis:
        image: redis:7-alpine
        ports: ["6379:6379"]
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s

    steps:
      - uses: actions/checkout@v4

      - name: Setup Python 3.12
        uses: actions/setup-python@v5
        with:
          python-version: '3.12'

      - name: Install uv
        uses: astral-sh/setup-uv@v4
        with:
          version: "latest"

      - name: Cache dependencies
        uses: actions/cache@v4
        with:
          path: ~/.cache/uv
          key: ${{ runner.os }}-uv-${{ hashFiles('backend/uv.lock') }}

      - name: Install dependencies
        run: uv sync --frozen --no-dev

      - name: Lint - Ruff format check
        run: uv run ruff format --check app/

      - name: Lint - Ruff code check
        run: uv run ruff check app/

      - name: Type check with mypy
        run: uv run mypy app/ --strict --ignore-missing-imports

      - name: Run tests with coverage
        env:
          DATABASE_URL: postgresql://postgres:test@localhost:5432/test_db
          REDIS_URL: redis://localhost:6379
        run: uv run pytest tests/ --cov=app --cov-report=xml --cov-report=term -v

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v4
        with:
          file: ./backend/coverage.xml
          flags: backend
          token: ${{ secrets.CODECOV_TOKEN }}

  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: 'backend/'
          severity: 'CRITICAL,HIGH'
          format: 'sarif'
          output: 'trivy-results.sarif'

      - name: Upload Trivy results to GitHub Security
        uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: 'trivy-results.sarif'

  docker-build:
    runs-on: ubuntu-latest
    needs: [lint-and-test, security-scan]
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: ./backend
          push: true
          tags: |
            ghcr.io/${{ github.repository }}/backend:latest
            ghcr.io/${{ github.repository }}/backend:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

**Key features:**
- Latest GitHub Actions runners (ubuntu-24.04)
- uv for fast Python package management
- Service containers for integration tests
- Concurrency control to cancel outdated runs
- Docker Buildx with GitHub Actions cache
- SARIF upload for security findings

### Example 3: Database Migration with Alembic (2025)

**E-commerce order tracking migration:**
```python
# backend/alembic/versions/2025_12_29_add_order_tracking.py
"""Add order tracking with shipment provider

Revision ID: a1b2c3d4e5f6
Revises: prev_revision_id
Create Date: 2025-12-29 10:30:00
"""
from alembic import op
import sqlalchemy as sa
from sqlalchemy.dialects import postgresql

# Revision identifiers
revision = 'a1b2c3d4e5f6'
down_revision = 'prev_revision_id'
branch_labels = None
depends_on = None

def upgrade():
    # Add tracking column as nullable first (backward compatible)
    op.add_column('orders',
        sa.Column('tracking_number', sa.String(100), nullable=True)
    )
    op.add_column('orders',
        sa.Column('shipping_provider', sa.String(50), nullable=True)
    )
    op.add_column('orders',
        sa.Column('shipped_at', sa.DateTime(timezone=True), nullable=True)
    )

    # Add composite index for tracking lookups
    op.create_index(
        'idx_orders_tracking',
        'orders',
        ['tracking_number', 'shipping_provider'],
        unique=False
    )

    # Add check constraint for valid providers
    op.create_check_constraint(
        'ck_orders_shipping_provider',
        'orders',
        "shipping_provider IN ('USPS', 'UPS', 'FedEx', 'DHL')"
    )

def downgrade():
    op.drop_constraint('ck_orders_shipping_provider', 'orders', type_='check')
    op.drop_index('idx_orders_tracking', 'orders')
    op.drop_column('orders', 'shipped_at')
    op.drop_column('orders', 'shipping_provider')
    op.drop_column('orders', 'tracking_number')
```

**Migration workflow with uv (2025):**
```bash
# Create new migration (auto-generate from models)
uv run alembic revision --autogenerate -m "Add order tracking with shipment provider"

# ALWAYS review the generated migration!
cat alembic/versions/*_add_order_tracking.py

# Test migration in development
uv run alembic upgrade head

# Verify schema changes
uv run alembic current
psql $DATABASE_URL -c "\d orders"

# Rollback if issues found
uv run alembic downgrade -1

# Check migration history
uv run alembic history --verbose
```

**Production migration checklist:**
```bash
# 1. Backup database
pg_dump $DATABASE_URL > backup_$(date +%Y%m%d_%H%M%S).sql

# 2. Test migration on staging with production data snapshot
uv run alembic upgrade head

# 3. Monitor query performance
EXPLAIN ANALYZE SELECT * FROM orders WHERE tracking_number = 'TRACK123';

# 4. Deploy with zero downtime (nullable columns)
# Phase 1: Add nullable columns
# Phase 2: Backfill data
# Phase 3: Add NOT NULL constraint (if needed)
```

---

## Extended Thinking Triggers

Use Opus 4.5 extended thinking for:
- **Architecture decisions** - Kubernetes vs serverless, multi-region setup
- **Migration planning** - Moving between cloud providers
- **Incident response** - Complex deployment failures
- **Security design** - Zero-trust architecture

## Templates Reference

| Template | Purpose |
|----------|---------|
| `github-actions-pipeline.yml` | Full CI/CD workflow with 6 stages |
| `Dockerfile` | Multi-stage Node.js build |
| `docker-compose.yml` | Development environment |
| `k8s-manifests.yaml` | Deployment, Service, Ingress |
| `helm-values.yaml` | Helm chart values |
| `terraform-aws.tf` | VPC, EKS, RDS infrastructure |
| `argocd-application.yaml` | GitOps application |
| `external-secrets.yaml` | Secrets Manager integration |
