---
name: deployment-engineer
description: Expert deployment automation for cloud platforms. Handles CI/CD pipelines, container orchestration, infrastructure setup, and production deployments with battle-tested configurations. Specializes in GitHub Actions, Docker, HuggingFace Spaces, and GitHub Pages.
category: devops
version: 1.0.0
---

# Deployment Engineer Skill

## Purpose

Automate and manage production deployments across multiple platforms with zero-downtime, proper monitoring, and rollback capabilities. This skill encapsulates hard-won lessons from real-world deployment scenarios.

## When to Use This Skill

Use this skill when:
- Setting up CI/CD pipelines for web applications
- Deploying to HuggingFace Spaces, Vercel, Netlify, or GitHub Pages
- Configuring Docker containers and orchestration
- Implementing environment-specific configurations
- Troubleshooting deployment failures
- Setting up monitoring and health checks

## Core Deployment Patterns

### 1. Multi-Platform Deployment Strategy

**Lesson Learned**: Always verify platform-specific requirements before deployment.

```yaml
# .github/workflows/deploy-backend.yml
# Critical patterns discovered through painful debugging:

# 1. Branch Name Consistency
on:
  push:
    # NEVER assume 'main' - always verify actual branch name
    branches: [master]  # Fixed from 'main' after repo inspection

# 2. Authentication for External Services
- name: Deploy to HuggingFace
  env:
    HF_TOKEN: ${{ secrets.HF_TOKEN }}
  run: |
    # Pattern: Use credential helper for Git auth
    git config credential.helper store
    echo "https://hf:$HF_TOKEN@huggingface.co" > ~/.git-credentials
    git remote set-url origin https://hf:$HF_TOKEN@huggingface.co/spaces/${{ env.HF_SPACE_NAME }}

# 3. Error Handling and Verification
- name: Verify Deployment
  run: |
    # Always add post-deployment verification
    curl -f "${{ env.DEPLOY_URL }}/health" || echo "Health check failed - space might still be starting"
```

### 2. Docker Configuration Best Practices

**Lesson Learned**: Order of operations in Dockerfile is critical for build success.

```dockerfile
# backend/Dockerfile - Battle-tested pattern

# 1. Use specific Python version
FROM python:3.11-slim

# 2. Install system dependencies FIRST
RUN apt-get update && apt-get install -y \
    gcc \
    curl \
    && rm -rf /var/lib/apt/lists/*

# 3. Set working directory early
WORKDIR /app

# 4. Copy requirements BEFORE source code (leverages Docker cache)
COPY pyproject.toml requirements.txt README.md ./

# 5. Install Python dependencies
RUN pip install uv
RUN uv pip install --system -e .

# 6. Copy application code
COPY . .

# 7. Create non-root user AFTER installation
RUN useradd -m -u 1000 user && chown -R user:user /app
USER user

# 8. Expose port and health check
EXPOSE 7860
HEALTHCHECK --interval=30s --timeout=30s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:7860/health || exit 1

# 9. CMD must be last
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "7860", "--workers", "1"]
```

### 3. Environment Variables Management

**Lesson Learned**: Different platforms require different environment variable strategies.

```python
# backend/main.py - Environment loading pattern

from dotenv import load_dotenv

# Load .env for local development
load_dotenv()

class Settings(BaseSettings):
    """Always provide defaults for critical settings"""

    # OpenAI Configuration
    openai_api_key: str = os.getenv("OPENAI_API_KEY", "")
    openai_model: str = os.getenv("OPENAI_MODEL", "gpt-4o-mini")  # Default to stable model

    # Platform Detection
    is_hf_spaces: bool = os.getenv("SPACE_ID") is not None
    is_production: bool = os.getenv("NODE_ENV") == "production"

    @property
    def api_endpoint(self) -> str:
        """Auto-detect API endpoint based on platform"""
        if self.is_hf_spaces:
            # HuggingFace Spaces
            space_name = os.getenv("SPACE_ID", "")
            return f"https://{space_name.replace(' ', '-').lower()}.hf.space"
        elif self.is_production:
            # Production environment
            return os.getenv("API_URL", "")
        else:
            # Local development
            return "http://localhost:7860"
```

### 4. CORS Configuration for Cross-Origin Requests

**Lesson Learned**: Frontend and backend on different domains require explicit CORS setup.

```python
# backend/main.py - CORS configuration

app = FastAPI()

# Dynamic CORS origins based on environment
cors_origins = []
if os.getenv("NODE_ENV") == "production":
    cors_origins = [
        "https://yourusername.github.io",
        "https://yourdomain.com"
    ]
else:
    cors_origins = ["http://localhost:3000", "http://localhost:7860"]

app.add_middleware(
    CORSMiddleware,
    allow_origins=cors_origins,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

### 5. Frontend Configuration Pattern

**Lesson Learned**: Frontend must adapt to different deployment environments.

```typescript
// src/theme/Root.tsx - Dynamic API endpoint detection
const getChatkitEndpoint = () => {
  // Check environment variable first
  if (process.env.REACT_APP_CHAT_API_URL) {
    return process.env.REACT_APP_CHAT_API_URL;
  }

  const hostname = window.location.hostname;
  if (hostname === 'localhost' || hostname === '127.0.0.1') {
    return 'http://localhost:7860/chat';
  }

  // Production URLs
  if (hostname.includes('github.io')) {
    // GitHub Pages
    return 'https://your-space.hf.space/chat';
  } else if (hostname.includes('hf.space')) {
    // HuggingFace Spaces
    return `https://${hostname}/chat`;
  }

  return '/chat'; // Same domain deployment
};
```

## Common Pitfalls & Solutions

### 1. Branch Name Mismatch
**Problem**: GitHub Actions configured for 'main' but repo uses 'master'
```yaml
# NEVER hard-code branch names
branches: [master]  # Verify with `git branch` first
```

### 2. Docker Build Failures
**Problem**: Permission errors during package installation
```dockerfile
# Install dependencies BEFORE switching to non-root user
RUN uv pip install --system -e .  # As root
USER user  # Switch AFTER installation
```

### 3. Model Compatibility Issues
**Problem**: Using models that require different APIs
```python
# Wrong: gpt-5-nano requires Responses API, not Chat Completions
# Correct: Use compatible models
openai_model: str = os.getenv("OPENAI_MODEL", "gpt-4o-mini")
```

### 4. Query Validation Errors
**Problem**: Backend crashes on short queries like "hi"
```python
# Allow single character queries
if not query or len(query.strip()) < 1:
    raise ValueError("Query must be at least 1 character long")
```

### 5. Missing Health Checks
**Problem**: No way to verify deployment success
```python
@app.get("/health")
async def health_check():
    """Always implement health endpoints"""
    return {
        "status": "healthy",
        "version": "1.0.0",
        "timestamp": datetime.utcnow().isoformat(),
        "services": {
            "database": await check_database(),
            "openai": bool(os.getenv("OPENAI_API_KEY"))
        }
    }
```

### 6. Hatchling README.md Not Found Error
**Problem**: `pip install -e .` fails with `OSError: Readme file does not exist: README.md`
```dockerfile
# ❌ Wrong - README.md not copied before pip install
COPY pyproject.toml ./
RUN pip install --no-cache-dir -e .

# ✅ Correct - Copy README.md with pyproject.toml
COPY pyproject.toml README.md ./
RUN pip install --no-cache-dir -e .
```
**Root Cause**: `pyproject.toml` has `readme = "README.md"` but hatchling can't find it during install.
**Files Affected**: `Dockerfile`, `Dockerfile.hf`

### 7. Multiple Dockerfiles Confusion
**Problem**: HuggingFace Spaces uses `Dockerfile` by default, not `Dockerfile.hf`
```bash
# You have TWO files:
Dockerfile       # Used by HF Spaces by default
Dockerfile.hf     # IGNORED by HF Spaces unless specified

# Solution: Keep BOTH files in sync or use one file
# Or specify in README.md frontmatter:
# sdk: docker
# dockerfile: Dockerfile.hf  # Optional override
```
**Lesson Learned**: When you have multiple Dockerfiles, HuggingFace uses `Dockerfile` by default. Either keep them synchronized or explicitly specify which one to use.

### 8. Docusaurus SSR Build Errors
**Problem**: `ReferenceError: window is not defined` or `localStorage is not defined` during build
```typescript
// ❌ Wrong - Runs during SSR build
function setupAPIConfig() {
  window.__API_BASE_URL__ = 'http://localhost:7860';
}
setupAPIConfig(); // Runs immediately at module load

// ✅ Correct - SSR guard
function setupAPIConfig() {
  window.__API_BASE_URL__ = 'http://localhost:7860';
}
if (typeof window !== 'undefined') {
  setupAPIConfig(); // Only runs in browser
}
```

**For AuthContext with localStorage:**
```typescript
// ❌ Wrong - getInitialState accesses localStorage during SSR
const getInitialState = (): AuthState => {
  const tokens = tokenManager.getTokens(); // Uses localStorage
  return { token: tokens.token, ... };
};

// ✅ Correct - SSR guard in init function
const getInitialState = (): AuthState => {
  // Return default state during SSR
  if (typeof window === 'undefined') {
    return {
      user: null,
      token: null,
      refreshToken: null,
      isLoading: false,
      isAuthenticated: false,
      error: null,
    };
  }

  const tokens = tokenManager.getTokens();
  return { token: tokens.token, ... };
};
```
**Files Affected**: `src/clientModules/apiConfig.ts`, `src/context/AuthContext.tsx`

### 9. HuggingFace Spaces Missing Configuration
**Problem**: "Missing configuration in README" error
```yaml
# ❌ Wrong - README.md missing YAML frontmatter
# My Backend

FastAPI backend...

# ✅ Correct - YAML frontmatter at TOP of README.md
---
title: AI Book Backend
emoji: 🤖
colorFrom: blue
colorTo: indigo
sdk: docker
sdk_version: "3.11"
app_file: main.py
pinned: false
license: mit
---

# AI Book Backend

FastAPI backend...
```
**Root Cause**: HuggingFace Spaces requires YAML configuration in README.md at the ROOT of the repository.
**Files Affected**: `backend/README.md`

### 10. Client Module SSR Execution
**Problem**: Client modules execute during SSR build in Docusaurus
```typescript
// ❌ Wrong - Immediately executes code that needs browser APIs
// src/clientModules/apiConfig.ts
const config = window.location.hostname; // FAILS during build

// ✅ Correct - Lazy execution with guard
// src/clientModules/apiConfig.ts
function setupAPIConfig() {
  if (typeof window !== 'undefined') {
    window.__API_BASE_URL__ = 'http://localhost:7860';
  }
}
// Only call if in browser
if (typeof window !== 'undefined') {
  setupAPIConfig();
}
export default {};
```
**Key Insight**: Docusaurus client modules are bundled server-side. Always check `typeof window !== 'undefined'` before accessing browser APIs.

### 11. Outdated Import Paths After Code Refactoring
**Problem**: Module import errors after code reorganization
```python
# ❌ Old import paths from refactored code
from database.config import get_db, SessionLocal, create_tables
from auth.auth import verify_token

# ✅ Fix: Update to new module structure
from src.core.database import get_async_db, SessionLocal, create_all_tables
from src.core.security import verify_token

# For sync operations in tests/migrations:
from src.core.database import get_sync_db
```
**Common Patterns**:
- `get_db` → `get_async_db` (async) or `get_sync_db` (sync)
- `Session` → `AsyncSession` (async type hints)
- `create_tables` → `create_all_tables`
- `database.config` → `src.core.database`

**Files Affected**: All files referencing old database modules after refactoring

### 12. Missing Configuration Attributes
**Problem**: `AttributeError: 'Settings' object has no attribute 'X'`
```python
# ❌ Settings class missing required attributes
class Settings(BaseSettings):
    database_url: str
    jwt_secret_key: str
    # Missing: openai_api_key, qdrant_url, etc.

# ✅ Fix: Add all required attributes with defaults
class Settings(BaseSettings):
    # Core
    database_url: str = "sqlite:///./database/auth.db"
    jwt_secret_key: str = "your-secret-key"

    # OpenAI (for RAG features)
    openai_api_key: Optional[str] = Field(default=None)
    openai_model: str = "gpt-4o-mini"
    openai_embedding_model: str = "text-embedding-3-small"

    # Qdrant (for vector search)
    qdrant_url: Optional[str] = Field(default=None)
    qdrant_api_key: Optional[str] = Field(default=None)

    # RAG settings
    chunk_size: int = 512
    chunk_overlap: int = 50
    batch_size: int = 32
    max_context_messages: int = 10
```
**Root Cause**: Settings class refactored but main.py still references old attributes.

**Files Affected**: `src/core/config.py`, `main.py`

### 13. Undefined Global Variables in Scripts
**Problem**: `NameError: name 'DATABASE_URL' is not defined` in init scripts
```python
# ❌ Using undefined global variable
print(f"Initializing database at: {DATABASE_URL}")

# ✅ Fix: Use Settings object
from src.core.config import settings
print(f"Initializing database at: {settings.database_url_sync}")
```
**Files Affected**: `init_database.py`, startup scripts

### 14. HuggingFace Spaces Docker Build Issues
**Problem**: Docker build fails with various errors on HuggingFace Spaces

| Error | Cause | Solution |
|-------|--------|----------|
| `OSError: Readme file does not exist: README.md` | pyproject.toml references README.md but Dockerfile doesn't copy it | `COPY pyproject.toml README.md ./` before `pip install -e .` |
| `ModuleNotFoundError: No module named 'X'` | Outdated import paths after refactoring | Update all imports to new module structure |
| `AttributeError: 'Settings' object has no attribute 'X'` | Settings class missing attributes | Add all required attributes to Settings class |
| `NameError: name 'VAR' is not defined` | Using undefined global variables | Use `from src.core.config import settings` and access via settings object |
| `Config file '.env' not found` | Missing .env file (warning only) | Ensure all required env vars set in HF Space secrets |
| `AttributeError: 'AsyncSession' object has no attribute 'query'` | Using sync query() with AsyncSession | Use `await db.execute(select(Model))` instead of `db.query(Model)` |
| `asyncpg.exceptions._base.InterfaceError: connection is closed` | Database connection pool giving stale connections | Add `pool_pre_ping=True` and reduce `pool_recycle` to 1800 for Neon |

### 15. Database Initialization in Async Context
**Problem**: Trying to use async functions in sync context during startup
```python
# ❌ Wrong: Calling async function without await
async def create_all_tables():
    await conn.run_sync(Base.metadata.create_all)

# In startup sync context:
create_all_tables()  # Doesn't actually create tables!

# ✅ Fix: Use sync engine for startup
from src.core.database import sync_engine, Base
Base.metadata.create_all(sync_engine)

# OR use async properly:
import asyncio
asyncio.create_task(create_all_tables())  # Fire and forget
```
**Files Affected**: `main.py` lifespan function, `init_database.py`

### 16. AsyncSession Query Method Error (Runtime)
**Problem**: `AttributeError: 'AsyncSession' object has no attribute 'query'`
```python
# ❌ Wrong: Using sync query() method with AsyncSession
@router.get("/users")
async def get_users(db: AsyncSession = Depends(get_async_db)):
    users = db.query(User).all()  # Error: AsyncSession has no 'query'
    return users

# ✅ Fix: Use select() with execute() for async
from sqlalchemy import select

@router.get("/users")
async def get_users(db: AsyncSession = Depends(get_async_db)):
    result = await db.execute(select(User))
    users = result.scalars().all()
    return users

# For single record:
result = await db.execute(select(User).filter(User.id == user_id))
user = result.scalar_one_or_none()

# For filtering:
result = await db.execute(
    select(User).filter(User.email == email)
)
user = result.scalar_one_or_none()
```
**Common Async Patterns:**
- `db.query(Model).filter(...).first()` → `result = await db.execute(select(Model).filter(...)); user = result.scalar_one_or_none()`
- `db.query(Model).all()` → `result = await db.execute(select(Model)); users = result.scalars().all()`
- `db.commit()` → `await db.commit()`
- `db.refresh(obj)` → `await db.refresh(obj)`
- `db.add(obj)` → `db.add(obj)` (no await needed)
- `db.delete(obj)` → `await db.delete(obj)` (if iterating) or use delete statement

**Files Affected**: All files using AsyncSession (routes, services, auth modules)

**Critical**: When converting from sync to async SQLAlchemy, ALL database operations must use the async pattern.

### 17. Database Connection Closed Error (Runtime)
**Problem**: `asyncpg.exceptions._base.InterfaceError: connection is closed`
```
Database session error: (sqlalchemy.dialects.postgresql.asyncpg.InterfaceError)
<class 'asyncpg.exceptions._base.InterfaceError'>: connection is closed
```
**Cause**: Database connection pool giving stale/closed connections, especially after idle periods.

**Fix**: Configure async engine with proper pool settings:
```python
# ❌ Wrong: Missing pool_pre_ping and incorrect pool settings
async_engine = create_async_engine(
    settings.database_url_async,
    pool_size=5,
    max_overflow=10,
    pool_recycle=3600,  # Too long for Neon's 5-min idle timeout
    # Missing pool_pre_ping
)

# ✅ Fix: Add pool_pre_ping and optimize settings for async
async_engine = create_async_engine(
    settings.database_url_async,
    echo=settings.debug,
    pool_size=3,  # Reduced from 5 for async
    max_overflow=5,  # Reduced from 10
    pool_timeout=30,
    pool_recycle=1800,  # 30 min (reduced from 3600 for Neon's idle timeout)
    pool_pre_ping=True,  # CRITICAL: Verify connections before use
    connect_args={
        "server_settings": {
            "application_name": "ai_book_backend",
            "timezone": "utc"
        },
        "command_timeout": 60,
        # Note: SSL configured via DATABASE_URL (sslmode=require)
    },
    pool_use_lifo=True,  # Use LIFO to reduce stale connections
    pool_drop_on_rollback=False,
)
```

**Common Issues:**
1. **Neon PostgreSQL idle timeout**: Free tier closes connections after 5 minutes of inactivity
2. **Missing pool_pre_ping**: Connections become stale but pool reuses them
3. **SSL misconfiguration**: Setting `ssl` directly in connect_args doesn't work with asyncpg
4. **Pool too large**: Async connections use more resources, keep pool smaller

**Files Affected**: `src/core/database.py`

**Critical for Neon PostgreSQL**: Reduce `pool_recycle` to 1800 (30 min) or less, and always use `pool_pre_ping=True`.

---

## HuggingFace Spaces Deployment: Complete Guide

### Critical Requirements

**1. README.md with YAML Frontmatter (REQUIRED)**
```yaml
---
title: AI Book Backend
emoji: 🤖
colorFrom: blue
colorTo: indigo
sdk: docker
sdk_version: "3.11"
app_file: main.py
pinned: false
license: mit
---
```
Must be at ROOT of repository with YAML at the TOP.

**2. Dockerfile Requirements**
```dockerfile
# MUST copy README.md before pip install
COPY pyproject.toml README.md ./
RUN pip install --no-cache-dir -e .

# Not just:
COPY pyproject.toml ./  # ❌ Will fail if pyproject.toml has readme field
```

**3. Environment Variables (Set in Space Settings)**
```
JWT_SECRET_KEY=your-super-secret-jwt-key-at-least-32-chars
DATABASE_URL=sqlite:///./database/auth.db
ALLOWED_ORIGINS=https://your-frontend.github.io,https://huggingface.co
```

**4. Import Path Consistency**
All Python imports must use the new module structure:
```python
# Old (broken):
from database.config import get_db
from auth.auth import verify_token

# New (working):
from src.core.database import get_async_db
from src.core.security import verify_token
```

**5. Database Session Types**
```python
# For async endpoints (most FastAPI routes):
from src.core.database import get_async_db
from sqlalchemy.ext.asyncio import AsyncSession

@router.get("/users")
async def get_users(db: AsyncSession = Depends(get_async_db)):
    result = await db.execute(select(User))
    return result.scalars().all()

# For sync operations (migrations, scripts):
from src.core.database import get_sync_db, sync_engine
from sqlalchemy.orm import Session

def run_migration():
    Base.metadata.create_all(sync_engine)
```

### Common Startup Sequence Failures

**Pattern 1: Import Errors**
```
File "/app/main.py", line 56, in <module>
    from routes import auth
File "/app/routes/auth.py", line 9, in <module>
    from database.config import get_db
ModuleNotFoundError: No module named 'database.config'
```
**Solution**: Update ALL import paths across the codebase.

**Pattern 2: Attribute Errors**
```
AttributeError: 'Settings' object has no attribute 'openai_api_key'
```
**Solution**: Add missing attributes to `src/core/config.py` Settings class.

**Pattern 3: Database Initialization Errors**
```
NameError: name 'DATABASE_URL' is not defined
```
**Solution**: Import settings and use `settings.database_url_sync`.

**Pattern 4: AsyncSession Query Errors (Runtime)**
```
AttributeError: 'AsyncSession' object has no attribute 'query'
```
**Solution**: Convert all database queries to async pattern using `select()`:
```python
# Replace db.query() with:
from sqlalchemy import select
result = await db.execute(select(Model).filter(...))
item = result.scalar_one_or_none()
```

**Pattern 5: Database Connection Closed (Runtime)**
```
asyncpg.exceptions._base.InterfaceError: connection is closed
```
**Solution**: Add `pool_pre_ping=True` to async engine and reduce `pool_recycle`:
```python
async_engine = create_async_engine(
    settings.database_url_async,
    pool_pre_ping=True,  # Verify connections before use
    pool_recycle=1800,   # 30 min (for Neon's idle timeout)
    pool_use_lifo=True,  # Use most recent connections first
)
```

### Production Deployment Checklist for HuggingFace Spaces

**Before Pushing:**
- [ ] README.md has YAML frontmatter at ROOT
- [ ] Dockerfile copies README.md before pip install
- [ ] All import paths updated to new structure
- [ ] Settings class has all required attributes
- [ ] Environment variables documented in `.env.hf-template`

**In HuggingFace Space Settings:**
- [ ] Set JWT_SECRET_KEY (required)
- [ ] Set DATABASE_URL (defaults to sqlite if not set)
- [ ] Set ALLOWED_ORIGINS (your frontend domain)
- [ ] Set OPENAI_API_KEY (if using RAG features)
- [ ] Set QDRANT_URL and QDRANT_API_KEY (if using vector search)

**After Deployment:**
- [ ] Check logs for startup errors
- [ ] Test `/health` endpoint
- [ ] Visit `/docs` for Swagger UI
- [ ] Test authentication endpoints
- [ ] Verify CORS with frontend requests

### Troubleshooting HuggingFace Spaces

**Issue**: "Config error" in Space UI
- **Fix**: Add YAML frontmatter to README.md

**Issue**: Build fails at pip install
- **Fix**: Ensure Dockerfile copies README.md with pyproject.toml

**Issue**: Module import errors
- **Fix**: Update all import paths from old structure to new `src.core.*` structure

**Issue**: AttributeError on startup
- **Fix**: Add missing configuration to Settings class

**Issue**: Database initialization fails
- **Fix**: Use sync operations in init scripts, ensure proper imports

## Deployment Checklist

### Pre-Deployment
- [ ] Verify branch names in workflows match actual branches
- [ ] Test Docker build locally: `docker build -t test .`
- [ ] Run container locally: `docker run -p 7860:7860 test`
- [ ] Check all environment variables are documented
- [ ] Validate API endpoints with health checks
- [ ] Test CORS configuration in browser dev tools

### Deployment
- [ ] Ensure secrets are configured in GitHub
- [ ] Monitor build logs for errors
- [ ] Verify deployment URL accessibility
- [ ] Test critical user flows
- [ ] Check error logs in production

### Post-Deployment
- [ ] Set up monitoring/alerting
- [ ] Document rollback procedure
- [ ] Update API documentation
- [ ] Notify stakeholders of deployment

## Platform-Specific Configurations

### HuggingFace Spaces
```yaml
# README.md frontmatter for HF Spaces
---
title: Your App Title
emoji: 🤖
colorFrom: blue
colorTo: green
sdk: docker
app_port: 7860
---
```

### GitHub Pages
```yaml
# docusaurus.config.ts for GitHub Pages
baseUrl: '/your-repo-name/',
organizationName: 'your-username',
projectName: 'your-repo',
deploymentBranch: 'gh-pages',
```

### Environment Variables
Create `.env.example`:
```env
# Required
OPENAI_API_KEY=your_key_here
QDRANT_URL=your_qdrant_url

# Optional
OPENAI_MODEL=gpt-4o-mini
NODE_ENV=production
```

## Troubleshooting Guide

### "HF_TOKEN not provided"
1. Check GitHub repository settings > Secrets
2. Verify secret name matches exactly: `HF_TOKEN`
3. Ensure workflow has permissions to access secrets

### Docker "Permission denied"
1. Install packages before creating non-root user
2. Use `--system` flag with uv/pip
3. Set proper file ownership: `chown -R user:user /app`

### CORS Errors
1. Add frontend domain to CORS origins
2. Check browser network tab for preflight requests
3. Verify API endpoint URLs are correct

### Application won't start
1. Check health endpoint: `curl /health`
2. Verify all environment variables
3. Check application logs for startup errors

## Scripts Directory

Include deployment helper scripts:
```bash
# scripts/deploy.sh
#!/bin/bash
set -e

echo "Starting deployment..."

# Build and test locally
docker build -t app .
docker run -d -p 7860:7860 --name test-app app
sleep 5
curl -f http://localhost:7860/health || exit 1
docker stop test-app

# Push to registry
echo "Deployment test passed!"
```

## Monitoring Setup

Always include basic monitoring:
```python
# Add to main.py
import structlog

logger = structlog.get_logger()

@app.middleware("http")
async def log_requests(request: Request, call_next):
    start_time = time.time()
    response = await call_next(request)
    process_time = time.time() - start_time

    logger.info(
        "request_processed",
        method=request.method,
        url=str(request.url),
        status_code=response.status_code,
        process_time=process_time
    )

    return response
```

## Security Considerations

1. **Never commit secrets**: Use environment variables
2. **Use HTTPS in production**: Configure SSL certificates
3. **Implement rate limiting**: Prevent abuse
4. **Validate inputs**: Sanitize all user inputs
5. **Regular updates**: Keep dependencies updated

## Rolling Back Deployments

```bash
# Git rollback
git revert <commit-hash>
git push origin master

# Or if using tags
git checkout previous-tag
git push -f origin master
```

Remember: The goal is not just to deploy, but to deploy reliably and maintainably. Test thoroughly, monitor continuously, and always have a rollback plan.