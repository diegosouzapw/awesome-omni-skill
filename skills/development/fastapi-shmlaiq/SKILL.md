---
name: fastapi
description: Comprehensive FastAPI development guide from beginner to production. This skill should be used when building REST APIs, web backends, microservices, or any Python web application with FastAPI. Covers project setup, routing, Pydantic models, async database operations, authentication (JWT/OAuth2), dependency injection, background tasks, WebSockets, testing, and production deployment. Triggers on tasks like "create a FastAPI app", "build an API", "add authentication", "connect database", "deploy FastAPI", or any FastAPI-related development.
---

# FastAPI Development Guide

Build modern, fast Python APIs from hello world to production-ready applications.

## Before Implementation

Gather context to ensure successful implementation:

| Source | Gather |
|--------|--------|
| **Codebase** | Existing project structure, database setup, auth patterns, dependencies |
| **Conversation** | User's specific API requirements, preferred database, deployment target |
| **Skill References** | Relevant patterns from `references/` directory |
| **User Guidelines** | Project-specific conventions, naming standards |

## Clarifications

### Required (ask if not clear)
1. **Project type?** Simple API / Production application / Microservice
2. **Database?** PostgreSQL / SQLite / None / Other
3. **Authentication needed?** JWT / API Key / OAuth2 / None

### Optional (ask if relevant)
4. **Deployment target?** Docker / Cloud (Railway/Render/Fly.io) / Local only
5. **Testing approach?** TDD / Post-implementation / None

## Official Documentation

| Resource | URL | Use For |
|----------|-----|---------|
| FastAPI Docs | https://fastapi.tiangolo.com | Official reference, tutorials |
| Pydantic Docs | https://docs.pydantic.dev | Model validation, settings |
| SQLAlchemy Docs | https://docs.sqlalchemy.org | Database ORM patterns |
| Uvicorn | https://www.uvicorn.org | ASGI server configuration |
| Alembic | https://alembic.sqlalchemy.org | Database migrations |

> **Version Note**: This skill follows FastAPI 0.100+ and Pydantic v2 patterns. For older versions, check official migration guides.

## Quick Start

```bash
uv init my-api && cd my-api
uv add "fastapi[standard]"
```

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def root():
    return {"message": "Hello World"}
```

```bash
uv run fastapi dev main.py
```

Access at `http://127.0.0.1:8000` | Docs at `/docs` | ReDoc at `/redoc`

## Project Structure

**Simple projects:**
```
project/
├── main.py
├── requirements.txt
└── .env
```

**Production projects:**
```
project/
├── app/
│   ├── __init__.py
│   ├── main.py           # FastAPI app instance
│   ├── config.py         # Settings/configuration
│   ├── dependencies.py   # Shared dependencies
│   ├── models/           # SQLAlchemy models
│   ├── schemas/          # Pydantic schemas
│   ├── routers/          # API route modules
│   ├── services/         # Business logic
│   └── utils/            # Helper functions
├── tests/
├── alembic/              # DB migrations
├── requirements.txt
├── .env
└── Dockerfile
```

## Core Concepts

### Path Operations
```python
@app.get("/items/{item_id}")      # Read
@app.post("/items/")               # Create
@app.put("/items/{item_id}")       # Update (full)
@app.patch("/items/{item_id}")     # Update (partial)
@app.delete("/items/{item_id}")    # Delete
```

### Parameters
```python
from fastapi import FastAPI, Query, Path, Body

@app.get("/items/{item_id}")
async def read_item(
    item_id: int = Path(..., ge=1),              # Path param, required, >= 1
    q: str | None = Query(None, max_length=50),  # Query param, optional
    skip: int = Query(0, ge=0),                  # Query param with default
):
    return {"item_id": item_id, "q": q, "skip": skip}
```

### Request/Response Models
```python
from pydantic import BaseModel, Field, EmailStr

class UserCreate(BaseModel):
    email: EmailStr
    password: str = Field(..., min_length=8)
    name: str

class UserResponse(BaseModel):
    id: int
    email: EmailStr
    name: str

    model_config = {"from_attributes": True}

@app.post("/users/", response_model=UserResponse)
async def create_user(user: UserCreate):
    # Create user in DB...
    return user
```

### Dependency Injection
```python
from fastapi import Depends

async def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

async def get_current_user(token: str = Depends(oauth2_scheme)):
    user = decode_token(token)
    if not user:
        raise HTTPException(401, "Invalid token")
    return user

@app.get("/users/me")
async def read_users_me(user: User = Depends(get_current_user)):
    return user
```

### Error Handling
```python
from fastapi import HTTPException
from fastapi.responses import JSONResponse

# Simple exception
raise HTTPException(status_code=404, detail="Item not found")

# Custom exception handler
class ItemNotFound(Exception):
    def __init__(self, item_id: int):
        self.item_id = item_id

@app.exception_handler(ItemNotFound)
async def item_not_found_handler(request, exc):
    return JSONResponse(
        status_code=404,
        content={"message": f"Item {exc.item_id} not found"}
    )
```

## Workflow Selection

**Starting a new project?** → See [references/basics.md](references/basics.md)

**Need data validation/schemas?** → See [references/models-validation.md](references/models-validation.md)

**Connecting a database?** → See [references/database.md](references/database.md)

**Adding authentication?** → See [references/authentication.md](references/authentication.md)

**Need WebSockets, background tasks, or middleware?** → See [references/advanced.md](references/advanced.md)

**Writing tests?** → See [references/testing.md](references/testing.md) *(includes TDD workflow)*

**Deploying to production?** → See [references/deployment.md](references/deployment.md)

### Searching Large References

For specific patterns in reference files, use grep:

```bash
# Authentication patterns
grep -n "JWT\|OAuth2\|token\|password" references/authentication.md

# Database patterns
grep -n "async\|session\|CRUD\|relationship" references/database.md

# Testing patterns
grep -n "fixture\|mock\|async_client" references/testing.md

# Deployment patterns
grep -n "Docker\|gunicorn\|nginx" references/deployment.md
```

## Common Patterns

### Router Organization
```python
# app/routers/users.py
from fastapi import APIRouter

router = APIRouter(prefix="/users", tags=["users"])

@router.get("/")
async def list_users():
    return []

@router.get("/{user_id}")
async def get_user(user_id: int):
    return {"id": user_id}

# app/main.py
from fastapi import FastAPI
from app.routers import users, items

app = FastAPI()
app.include_router(users.router)
app.include_router(items.router)
```

### Background Tasks
```python
from fastapi import BackgroundTasks

def send_email(email: str, message: str):
    # Send email logic...
    pass

@app.post("/send-notification/")
async def send_notification(
    email: str,
    background_tasks: BackgroundTasks
):
    background_tasks.add_task(send_email, email, "Welcome!")
    return {"message": "Notification sent in background"}
```

### File Upload
```python
from fastapi import UploadFile, File

@app.post("/upload/")
async def upload_file(file: UploadFile = File(...)):
    contents = await file.read()
    return {
        "filename": file.filename,
        "size": len(contents),
        "content_type": file.content_type
    }
```

### CORS
```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

## Essential Dependencies

```bash
# Core
uv add "fastapi[standard]"

# Database
uv add sqlalchemy asyncpg alembic  # PostgreSQL async
uv add sqlalchemy aiosqlite        # SQLite async

# Authentication
uv add "python-jose[cryptography]" "passlib[bcrypt]"

# Validation
uv add "pydantic[email]"

# Testing
uv add pytest pytest-asyncio httpx --dev
```

## Common Mistakes

| Mistake | Why It's Wrong | Fix |
|---------|----------------|-----|
| `allow_origins=["*"]` in production | Security risk, allows any origin | Use specific domains |
| Hardcoding `SECRET_KEY` | Exposed in version control | Use environment variables |
| Missing `response_model` | May leak sensitive fields (password) | Always filter responses |
| Forgetting `await` in async DB | Coroutine never executes | Add `await` to async calls |
| No input validation | SQL injection, invalid data | Use Pydantic models |
| Sync operations in async routes | Blocks event loop | Use async libraries |

## Quick Reference

| Need | Solution |
|------|----------|
| Run dev server | `uv run fastapi dev main.py` |
| Run prod server | `uv run fastapi run main.py` or `uv run uvicorn app.main:app` |
| Path parameter | `@app.get("/items/{id}")` |
| Query parameter | `def func(q: str = None)` |
| Request body | `def func(item: ItemModel)` |
| Require auth | `Depends(get_current_user)` |
| Database session | `Depends(get_db)` |
| Return specific model | `response_model=UserResponse` |
| Background work | `BackgroundTasks` |
| WebSocket | `@app.websocket("/ws")` |

## Before Delivery Checklist

### Code Quality
- [ ] All endpoints have appropriate HTTP status codes (201 for create, 204 for delete)
- [ ] Sensitive data filtered via `response_model` (no passwords in responses)
- [ ] Input validation using Pydantic models
- [ ] Error handling with proper `HTTPException` messages

### Security
- [ ] SECRET_KEY from environment variable
- [ ] CORS configured with specific origins (not `*`)
- [ ] Authentication on protected routes
- [ ] No hardcoded credentials

### Testing
- [ ] Tests written for all endpoints
- [ ] Error cases covered (404, 401, 422)
- [ ] Tests pass: `uv run pytest`

### Documentation
- [ ] OpenAPI docs accessible at `/docs`
- [ ] Endpoints have descriptive tags and summaries
