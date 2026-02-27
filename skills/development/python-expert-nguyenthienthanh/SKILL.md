---
name: python-expert
description: "Python backend expert. PROACTIVELY use when working with Django, FastAPI, Flask, Python APIs. Triggers: python, django, fastapi, flask, async python"
autoInvoke: true
priority: high
triggers:
  - "python"
  - "django"
  - "fastapi"
  - "flask"
  - "pydantic"
  - "sqlalchemy"
allowed-tools: Read, Grep, Glob, Edit, Write
---

# Python Expert Skill

Expert-level Python backend patterns for Django, FastAPI, Flask, and async programming.

---

## Auto-Detection

This skill activates when:
- Working with Python backend projects
- Detected `django`, `fastapi`, or `flask` in requirements.txt/pyproject.toml
- Working with `*.py` files in API/backend context
- Using SQLAlchemy, Pydantic, or Django ORM

---

## 1. FastAPI Patterns

### Pydantic Models

```python
# ✅ GOOD - Type-safe validation
from pydantic import BaseModel, EmailStr, Field, field_validator

class UserCreate(BaseModel):
    email: EmailStr
    name: str = Field(..., min_length=2, max_length=100)
    password: str = Field(..., min_length=8)

    @field_validator('name')
    @classmethod
    def name_must_not_be_empty(cls, v: str) -> str:
        if not v.strip():
            raise ValueError('Name cannot be empty')
        return v.strip()

    model_config = {'str_strip_whitespace': True}
```

### Dependency Injection

```python
# ✅ GOOD - Dependencies for shared logic
from typing import Annotated
from fastapi import Depends, HTTPException

async def get_current_user(
    token: str = Depends(oauth2_scheme),
    db: AsyncSession = Depends(get_db)
) -> User:
    user = await verify_token(token, db)
    if user is None:
        raise HTTPException(status_code=401, detail="Invalid token")
    return user

CurrentUser = Annotated[User, Depends(get_current_user)]
DB = Annotated[AsyncSession, Depends(get_db)]

@app.get("/me")
async def get_me(user: CurrentUser, db: DB):
    return user
```

### Background Tasks

```python
# ✅ GOOD - Non-blocking operations
from fastapi import BackgroundTasks

@app.post("/users/")
async def create_user(
    user: UserCreate,
    background_tasks: BackgroundTasks,
    db: AsyncSession = Depends(get_db)
):
    db_user = await crud.create_user(db, user)
    background_tasks.add_task(send_welcome_email, user.email)
    return db_user
```

### Exception Handling

```python
# ✅ GOOD - Custom exceptions
from fastapi import HTTPException
from fastapi.responses import JSONResponse

class AppException(Exception):
    def __init__(self, code: str, message: str, status_code: int = 400):
        self.code = code
        self.message = message
        self.status_code = status_code

@app.exception_handler(AppException)
async def app_exception_handler(request, exc: AppException):
    return JSONResponse(
        status_code=exc.status_code,
        content={"code": exc.code, "message": exc.message}
    )
```

### Lifespan Events

```python
# ✅ GOOD - Startup/shutdown handling
from contextlib import asynccontextmanager

@asynccontextmanager
async def lifespan(app: FastAPI):
    # Startup
    await database.connect()
    yield
    # Shutdown
    await database.disconnect()

app = FastAPI(lifespan=lifespan)
```

---

## 2. Django Patterns

### Prevent N+1 Queries

```python
# ❌ BAD - N+1 queries
users = User.objects.all()
for user in users:
    print(user.profile.bio)  # 1 query per user!

# ✅ GOOD - select_related for FK
users = User.objects.select_related('profile').all()

# ✅ GOOD - prefetch_related for M2M/reverse FK
posts = Post.objects.prefetch_related('tags', 'comments').all()
```

### Atomic Updates

```python
from django.db.models import F

# ❌ BAD - Race condition
article = Article.objects.get(pk=1)
article.views += 1
article.save()

# ✅ GOOD - Atomic update
Article.objects.filter(pk=1).update(views=F('views') + 1)
```

### Complex Queries

```python
from django.db.models import Q

# ✅ GOOD - OR query
users = User.objects.filter(
    Q(is_staff=True) | Q(is_superuser=True)
)

# ✅ GOOD - Combined conditions
users = User.objects.filter(
    Q(email__endswith='@company.com') &
    (Q(is_active=True) | Q(is_staff=True))
)
```

### Efficient Checks

```python
# ❌ BAD - Loads all objects
if len(User.objects.filter(email=email)) > 0:
    ...

# ✅ GOOD - Efficient existence check
if User.objects.filter(email=email).exists():
    ...
```

### Bulk Operations

```python
# ❌ BAD - N queries
for user_data in users_data:
    User.objects.create(**user_data)

# ✅ GOOD - 1 query
User.objects.bulk_create([
    User(**data) for data in users_data
])
```

### Transactions

```python
from django.db import transaction

@transaction.atomic
def transfer_funds(from_account, to_account, amount):
    from_account.balance = F('balance') - amount
    from_account.save()
    to_account.balance = F('balance') + amount
    to_account.save()
```

---

## 3. SQLAlchemy 2.0 Patterns

### Modern Declarative Syntax

```python
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column, relationship
from sqlalchemy import ForeignKey

class Base(DeclarativeBase):
    pass

class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)
    email: Mapped[str] = mapped_column(unique=True, index=True)
    name: Mapped[str]
    is_active: Mapped[bool] = mapped_column(default=True)

    posts: Mapped[list["Post"]] = relationship(back_populates="author")
```

### Async Queries

```python
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select

async def get_user(db: AsyncSession, user_id: int) -> User | None:
    stmt = select(User).where(User.id == user_id)
    result = await db.execute(stmt)
    return result.scalar_one_or_none()
```

### Eager Loading

```python
from sqlalchemy.orm import selectinload, joinedload

# ✅ GOOD - Prevent N+1
stmt = select(User).options(
    selectinload(User.posts),  # Separate query
    joinedload(User.profile)   # JOIN
)
```

### Pagination

```python
from sqlalchemy import func

async def get_users_paginated(
    db: AsyncSession,
    page: int = 1,
    per_page: int = 20
) -> tuple[list[User], int]:
    # Count total
    count_stmt = select(func.count()).select_from(User)
    total = await db.scalar(count_stmt)

    # Get page
    stmt = select(User).offset((page - 1) * per_page).limit(per_page)
    result = await db.execute(stmt)
    users = result.scalars().all()

    return users, total
```

---

## 4. Async Best Practices

### Parallel Operations

```python
import asyncio

# ✅ GOOD - Concurrent execution
async def get_dashboard_data(user_id: int) -> dict:
    user, posts, notifications = await asyncio.gather(
        get_user(user_id),
        get_user_posts(user_id),
        get_notifications(user_id)
    )
    return {"user": user, "posts": posts, "notifications": notifications}
```

### Timeout Handling

```python
# ✅ GOOD - Proper timeout
async def fetch_with_timeout(url: str, timeout: float = 5.0):
    async with asyncio.timeout(timeout):
        async with httpx.AsyncClient() as client:
            return await client.get(url)
```

### Rate Limiting

```python
# ✅ GOOD - Semaphore for concurrency control
semaphore = asyncio.Semaphore(10)  # Max 10 concurrent

async def fetch_limited(url: str):
    async with semaphore:
        return await fetch(url)
```

### Task Groups (Python 3.11+)

```python
# ✅ GOOD - Structured concurrency
async def process_all(items: list[Item]):
    async with asyncio.TaskGroup() as tg:
        for item in items:
            tg.create_task(process_item(item))
    # All tasks completed or exception raised
```

---

## 5. Type Hints Best Practices

```python
# ✅ GOOD - Modern type hints (Python 3.10+)
def get_users(active: bool | None = None) -> list[User]:
    ...

# ✅ GOOD - TypeVar for generics
from typing import TypeVar

T = TypeVar('T', bound=BaseModel)

async def get_or_404(db: AsyncSession, model: type[T], id: int) -> T:
    obj = await db.get(model, id)
    if obj is None:
        raise HTTPException(status_code=404)
    return obj

# ✅ GOOD - Protocol for structural typing
from typing import Protocol

class Repository(Protocol):
    async def get(self, id: int) -> Model | None: ...
    async def create(self, data: dict) -> Model: ...
    async def delete(self, id: int) -> bool: ...
```

---

## 6. Testing Patterns

### Pytest Fixtures

```python
import pytest
from httpx import AsyncClient

@pytest.fixture
async def client(app):
    async with AsyncClient(app=app, base_url="http://test") as client:
        yield client

@pytest.fixture
async def db_session():
    async with async_session() as session:
        yield session
        await session.rollback()
```

### Parametrized Tests

```python
@pytest.mark.parametrize("email,valid", [
    ("test@example.com", True),
    ("invalid", False),
    ("", False),
    ("a@b.c", True),
])
def test_validate_email(email: str, valid: bool):
    result = validate_email(email)
    assert result == valid
```

### Factory Pattern

```python
from factory import Factory, Faker

class UserFactory(Factory):
    class Meta:
        model = User

    email = Faker('email')
    name = Faker('name')
    is_active = True

# Usage
user = UserFactory()
inactive_user = UserFactory(is_active=False)
```

### Mocking

```python
from unittest.mock import AsyncMock, patch

@pytest.mark.asyncio
async def test_create_user_sends_email():
    with patch('app.services.send_email', new_callable=AsyncMock) as mock:
        user = await create_user(UserCreate(email="test@example.com"))
        mock.assert_called_once()
```

---

## 7. Error Handling

### Custom Exception Hierarchy

```python
class AppError(Exception):
    """Base application error"""
    def __init__(self, message: str, code: str = "UNKNOWN"):
        self.message = message
        self.code = code
        super().__init__(message)

class NotFoundError(AppError):
    def __init__(self, resource: str, id: int):
        super().__init__(f"{resource} with id {id} not found", "NOT_FOUND")

class ValidationError(AppError):
    def __init__(self, field: str, message: str):
        super().__init__(message, f"VALIDATION_{field.upper()}")
```

### Result Pattern

```python
from dataclasses import dataclass
from typing import Generic, TypeVar

T = TypeVar('T')
E = TypeVar('E')

@dataclass
class Ok(Generic[T]):
    value: T

@dataclass
class Err(Generic[E]):
    error: E

Result = Ok[T] | Err[E]

async def get_user(user_id: int) -> Result[User, str]:
    user = await db.get(User, user_id)
    if user is None:
        return Err("User not found")
    return Ok(user)

# Usage
match await get_user(123):
    case Ok(user):
        print(f"Found: {user.name}")
    case Err(error):
        print(f"Error: {error}")
```

---

## Quick Reference

```toon
checklist[12]{pattern,best_practice}:
  FastAPI,Pydantic models + Depends injection
  Django ORM,select_related/prefetch_related
  SQLAlchemy,Mapped types + selectinload
  Async,asyncio.gather for parallel
  Typing,Modern 3.10+ syntax X | None
  Testing,pytest fixtures + parametrize
  Validation,Pydantic or Django Forms
  Errors,Custom exception hierarchy
  Transactions,atomic or db.transaction
  Queries,F() objects for atomic updates
  Bulk,bulk_create over loops
  Cache,Redis or Django cache
```

---

**Version:** 1.3.0
