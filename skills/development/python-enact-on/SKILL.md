---
name: python
description: Python, FastAPI, Django, and Flask development patterns and best practices
license: MIT
compatibility: opencode
---

# Python Skill

Comprehensive patterns and best practices for Python backend development with FastAPI, Django, and Flask.

## What I Know

### Project Structure (FastAPI)

```
src/
├── app/
│   ├── __init__.py
│   ├── main.py              # Application entry point
│   ├── config.py            # Configuration settings
│   ├── dependencies.py      # Dependency injection
│   ├── api/
│   │   ├── __init__.py
│   │   ├── v1/
│   │   │   ├── __init__.py
│   │   │   ├── router.py    # API router aggregation
│   │   │   └── endpoints/
│   │   │       ├── users.py
│   │   │       └── items.py
│   ├── core/
│   │   ├── security.py      # Auth, JWT, passwords
│   │   └── exceptions.py    # Custom exceptions
│   ├── models/
│   │   ├── __init__.py
│   │   ├── user.py          # SQLAlchemy models
│   │   └── item.py
│   ├── schemas/
│   │   ├── __init__.py
│   │   ├── user.py          # Pydantic schemas
│   │   └── item.py
│   ├── services/
│   │   ├── __init__.py
│   │   └── user_service.py  # Business logic
│   └── repositories/
│       ├── __init__.py
│       └── user_repository.py
├── tests/
├── alembic/                  # Database migrations
├── pyproject.toml
└── requirements.txt
```

### FastAPI Patterns

**Application Setup**
```python
# app/main.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from contextlib import asynccontextmanager

from app.api.v1.router import api_router
from app.core.config import settings
from app.db.session import engine

@asynccontextmanager
async def lifespan(app: FastAPI):
    # Startup
    await create_db_tables()
    yield
    # Shutdown
    await engine.dispose()

app = FastAPI(
    title=settings.PROJECT_NAME,
    version="1.0.0",
    lifespan=lifespan,
)

app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.ALLOWED_ORIGINS,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

app.include_router(api_router, prefix="/api/v1")

@app.get("/health")
async def health_check():
    return {"status": "healthy"}
```

**Pydantic Schemas**
```python
# app/schemas/user.py
from pydantic import BaseModel, EmailStr, Field
from datetime import datetime
from typing import Optional

class UserBase(BaseModel):
    email: EmailStr
    name: str = Field(..., min_length=2, max_length=100)

class UserCreate(UserBase):
    password: str = Field(..., min_length=8)

class UserUpdate(BaseModel):
    email: Optional[EmailStr] = None
    name: Optional[str] = Field(None, min_length=2, max_length=100)

class UserResponse(UserBase):
    id: int
    created_at: datetime
    is_active: bool = True

    model_config = {"from_attributes": True}

class UserList(BaseModel):
    data: list[UserResponse]
    total: int
    page: int
    limit: int
```

**API Endpoints**
```python
# app/api/v1/endpoints/users.py
from fastapi import APIRouter, Depends, HTTPException, status, Query
from sqlalchemy.ext.asyncio import AsyncSession

from app.schemas.user import UserCreate, UserResponse, UserUpdate, UserList
from app.services.user_service import UserService
from app.dependencies import get_db, get_current_user
from app.models.user import User

router = APIRouter(prefix="/users", tags=["users"])

@router.get("", response_model=UserList)
async def list_users(
    page: int = Query(1, ge=1),
    limit: int = Query(20, ge=1, le=100),
    db: AsyncSession = Depends(get_db),
):
    """List all users with pagination."""
    service = UserService(db)
    users, total = await service.get_all(page=page, limit=limit)
    return UserList(data=users, total=total, page=page, limit=limit)

@router.get("/{user_id}", response_model=UserResponse)
async def get_user(
    user_id: int,
    db: AsyncSession = Depends(get_db),
):
    """Get a specific user by ID."""
    service = UserService(db)
    user = await service.get_by_id(user_id)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user

@router.post("", response_model=UserResponse, status_code=status.HTTP_201_CREATED)
async def create_user(
    user_data: UserCreate,
    db: AsyncSession = Depends(get_db),
):
    """Create a new user."""
    service = UserService(db)
    return await service.create(user_data)

@router.patch("/{user_id}", response_model=UserResponse)
async def update_user(
    user_id: int,
    user_data: UserUpdate,
    db: AsyncSession = Depends(get_db),
    current_user: User = Depends(get_current_user),
):
    """Update an existing user."""
    service = UserService(db)
    return await service.update(user_id, user_data)

@router.delete("/{user_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_user(
    user_id: int,
    db: AsyncSession = Depends(get_db),
    current_user: User = Depends(get_current_user),
):
    """Delete a user."""
    service = UserService(db)
    await service.delete(user_id)
```

**Service Layer**
```python
# app/services/user_service.py
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select, func
from fastapi import HTTPException

from app.models.user import User
from app.schemas.user import UserCreate, UserUpdate
from app.core.security import get_password_hash

class UserService:
    def __init__(self, db: AsyncSession):
        self.db = db

    async def get_all(self, page: int = 1, limit: int = 20) -> tuple[list[User], int]:
        offset = (page - 1) * limit

        # Get total count
        count_query = select(func.count()).select_from(User)
        total = (await self.db.execute(count_query)).scalar()

        # Get paginated results
        query = select(User).offset(offset).limit(limit)
        result = await self.db.execute(query)
        users = result.scalars().all()

        return list(users), total

    async def get_by_id(self, user_id: int) -> User | None:
        query = select(User).where(User.id == user_id)
        result = await self.db.execute(query)
        return result.scalar_one_or_none()

    async def get_by_email(self, email: str) -> User | None:
        query = select(User).where(User.email == email)
        result = await self.db.execute(query)
        return result.scalar_one_or_none()

    async def create(self, data: UserCreate) -> User:
        # Check for existing email
        existing = await self.get_by_email(data.email)
        if existing:
            raise HTTPException(status_code=409, detail="Email already exists")

        user = User(
            email=data.email,
            name=data.name,
            hashed_password=get_password_hash(data.password),
        )
        self.db.add(user)
        await self.db.commit()
        await self.db.refresh(user)
        return user

    async def update(self, user_id: int, data: UserUpdate) -> User:
        user = await self.get_by_id(user_id)
        if not user:
            raise HTTPException(status_code=404, detail="User not found")

        update_data = data.model_dump(exclude_unset=True)
        for field, value in update_data.items():
            setattr(user, field, value)

        await self.db.commit()
        await self.db.refresh(user)
        return user

    async def delete(self, user_id: int) -> None:
        user = await self.get_by_id(user_id)
        if not user:
            raise HTTPException(status_code=404, detail="User not found")

        await self.db.delete(user)
        await self.db.commit()
```

**SQLAlchemy Models**
```python
# app/models/user.py
from sqlalchemy import Column, Integer, String, Boolean, DateTime
from sqlalchemy.sql import func
from app.db.base import Base

class User(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True, index=True)
    email = Column(String(255), unique=True, index=True, nullable=False)
    name = Column(String(100), nullable=False)
    hashed_password = Column(String(255), nullable=False)
    is_active = Column(Boolean, default=True)
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    updated_at = Column(DateTime(timezone=True), onupdate=func.now())
```

**Async Database Session**
```python
# app/db/session.py
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession, async_sessionmaker
from app.core.config import settings

engine = create_async_engine(
    settings.DATABASE_URL,
    echo=settings.DEBUG,
    pool_pre_ping=True,
    pool_size=5,
    max_overflow=10,
)

AsyncSessionLocal = async_sessionmaker(
    engine,
    class_=AsyncSession,
    expire_on_commit=False,
)

async def get_db():
    async with AsyncSessionLocal() as session:
        try:
            yield session
        finally:
            await session.close()
```

**Configuration with Pydantic Settings**
```python
# app/core/config.py
from pydantic_settings import BaseSettings
from functools import lru_cache

class Settings(BaseSettings):
    PROJECT_NAME: str = "My API"
    DEBUG: bool = False
    DATABASE_URL: str
    SECRET_KEY: str
    ACCESS_TOKEN_EXPIRE_MINUTES: int = 30
    ALLOWED_ORIGINS: list[str] = ["http://localhost:3000"]

    model_config = {"env_file": ".env", "case_sensitive": True}

@lru_cache()
def get_settings() -> Settings:
    return Settings()

settings = get_settings()
```

**Authentication & Security**
```python
# app/core/security.py
from datetime import datetime, timedelta
from typing import Any
from jose import jwt, JWTError
from passlib.context import CryptContext
from fastapi import HTTPException, status, Depends
from fastapi.security import OAuth2PasswordBearer

from app.core.config import settings

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/api/v1/auth/login")

def get_password_hash(password: str) -> str:
    return pwd_context.hash(password)

def verify_password(plain_password: str, hashed_password: str) -> bool:
    return pwd_context.verify(plain_password, hashed_password)

def create_access_token(data: dict[str, Any], expires_delta: timedelta | None = None) -> str:
    to_encode = data.copy()
    expire = datetime.utcnow() + (expires_delta or timedelta(minutes=15))
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, settings.SECRET_KEY, algorithm="HS256")

async def get_current_user(token: str = Depends(oauth2_scheme)):
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = jwt.decode(token, settings.SECRET_KEY, algorithms=["HS256"])
        user_id: str = payload.get("sub")
        if user_id is None:
            raise credentials_exception
    except JWTError:
        raise credentials_exception

    # Get user from database
    # user = await user_service.get_by_id(int(user_id))
    # if user is None:
    #     raise credentials_exception
    # return user
```

**Exception Handlers**
```python
# app/core/exceptions.py
from fastapi import Request, HTTPException
from fastapi.responses import JSONResponse
from pydantic import ValidationError

async def http_exception_handler(request: Request, exc: HTTPException):
    return JSONResponse(
        status_code=exc.status_code,
        content={
            "success": False,
            "error": {
                "code": exc.status_code,
                "message": exc.detail,
            },
        },
    )

async def validation_exception_handler(request: Request, exc: ValidationError):
    return JSONResponse(
        status_code=422,
        content={
            "success": False,
            "error": {
                "code": 422,
                "message": "Validation error",
                "details": exc.errors(),
            },
        },
    )
```

### Django Patterns

**Models**
```python
# models.py
from django.db import models
from django.contrib.auth.models import AbstractUser

class User(AbstractUser):
    email = models.EmailField(unique=True)
    bio = models.TextField(blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    USERNAME_FIELD = 'email'
    REQUIRED_FIELDS = ['username']

    class Meta:
        ordering = ['-created_at']

class Post(models.Model):
    title = models.CharField(max_length=200)
    content = models.TextField()
    author = models.ForeignKey(User, on_delete=models.CASCADE, related_name='posts')
    published = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.title
```

**Views (Class-Based)**
```python
# views.py
from rest_framework import viewsets, status
from rest_framework.decorators import action
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticated

from .models import Post
from .serializers import PostSerializer
from .permissions import IsAuthorOrReadOnly

class PostViewSet(viewsets.ModelViewSet):
    queryset = Post.objects.select_related('author')
    serializer_class = PostSerializer
    permission_classes = [IsAuthenticated, IsAuthorOrReadOnly]

    def perform_create(self, serializer):
        serializer.save(author=self.request.user)

    @action(detail=True, methods=['post'])
    def publish(self, request, pk=None):
        post = self.get_object()
        post.published = True
        post.save()
        return Response({'status': 'published'})
```

**Serializers**
```python
# serializers.py
from rest_framework import serializers
from .models import User, Post

class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['id', 'email', 'username', 'bio', 'created_at']
        read_only_fields = ['id', 'created_at']

class PostSerializer(serializers.ModelSerializer):
    author = UserSerializer(read_only=True)

    class Meta:
        model = Post
        fields = ['id', 'title', 'content', 'author', 'published', 'created_at']
        read_only_fields = ['id', 'author', 'created_at']
```

### Testing

**Pytest with FastAPI**
```python
# tests/test_users.py
import pytest
from httpx import AsyncClient
from app.main import app
from app.db.session import get_db
from app.models.user import User

@pytest.fixture
async def client():
    async with AsyncClient(app=app, base_url="http://test") as ac:
        yield ac

@pytest.fixture
async def test_user(db_session):
    user = User(email="test@example.com", name="Test User", hashed_password="hashed")
    db_session.add(user)
    await db_session.commit()
    await db_session.refresh(user)
    return user

class TestUserEndpoints:
    @pytest.mark.asyncio
    async def test_create_user(self, client: AsyncClient):
        response = await client.post(
            "/api/v1/users",
            json={"email": "new@example.com", "name": "New User", "password": "password123"}
        )
        assert response.status_code == 201
        data = response.json()
        assert data["email"] == "new@example.com"

    @pytest.mark.asyncio
    async def test_get_user_not_found(self, client: AsyncClient):
        response = await client.get("/api/v1/users/999")
        assert response.status_code == 404

    @pytest.mark.asyncio
    async def test_list_users(self, client: AsyncClient, test_user):
        response = await client.get("/api/v1/users")
        assert response.status_code == 200
        data = response.json()
        assert data["total"] >= 1
```

### Type Hints Best Practices

```python
from typing import TypeVar, Generic, Sequence
from collections.abc import Callable, Awaitable

# Generic type for repository pattern
T = TypeVar("T")

class Repository(Generic[T]):
    async def get(self, id: int) -> T | None: ...
    async def get_all(self) -> Sequence[T]: ...
    async def create(self, data: dict) -> T: ...

# Callable types
Handler = Callable[[dict], Awaitable[dict]]

# Union types (Python 3.10+)
def process(value: int | str | None) -> str:
    if value is None:
        return "none"
    return str(value)
```

### Common Patterns

**Dependency Injection**
```python
from functools import lru_cache

class EmailService:
    def send(self, to: str, subject: str, body: str) -> None:
        pass

@lru_cache()
def get_email_service() -> EmailService:
    return EmailService()

# Usage in endpoint
@router.post("/notify")
async def notify(
    email_service: EmailService = Depends(get_email_service)
):
    email_service.send(...)
```

**Background Tasks**
```python
from fastapi import BackgroundTasks

def send_notification(email: str, message: str):
    # Send email in background
    pass

@router.post("/users")
async def create_user(
    user_data: UserCreate,
    background_tasks: BackgroundTasks,
):
    user = await service.create(user_data)
    background_tasks.add_task(send_notification, user.email, "Welcome!")
    return user
```

### Common Pitfalls

1. **Not using async properly** - Use `async def` and `await` consistently
2. **N+1 queries** - Use `select_related()` or eager loading
3. **Missing type hints** - Always add type annotations
4. **Hardcoding config** - Use environment variables and settings
5. **Not validating input** - Use Pydantic for all input validation
6. **Ignoring errors** - Always handle exceptions properly
7. **Sync in async context** - Don't block the event loop

### Best Practices

1. **Use Pydantic for validation** - All input/output through schemas
2. **Async all the way** - Use async database drivers
3. **Type everything** - Full type hints for better IDE support
4. **Repository pattern** - Separate data access from business logic
5. **Dependency injection** - Use FastAPI's `Depends()` system
6. **Environment configuration** - pydantic-settings for config
7. **Alembic for migrations** - Never modify database manually
8. **pytest for testing** - Use fixtures and async tests
9. **Structured logging** - Use structlog or loguru
10. **API versioning** - Version your APIs from the start

---

*Part of SuperAI GitHub - Centralized OpenCode Configuration*
