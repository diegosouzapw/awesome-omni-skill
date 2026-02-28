---
name: fastapi-patterns
description: FastAPI patterns with Pydantic, async operations, and dependency injection
license: MIT
---

# FastAPI Patterns Skill

Modern FastAPI development patterns including Pydantic models, async operations, dependency injection, and production best practices.

## When to Use

- Building async Python APIs
- Implementing high-performance REST/GraphQL APIs
- Working with Pydantic for validation
- Structuring FastAPI projects

## Project Structure

```
src/
├── main.py                 # Application entry point
├── config.py               # Settings management
├── database.py             # Database connection
├── dependencies.py         # Shared dependencies
├── exceptions.py           # Custom exceptions
├── api/
│   ├── __init__.py
│   ├── router.py           # Main router aggregator
│   └── v1/
│       ├── __init__.py
│       ├── router.py
│       ├── users/
│       │   ├── __init__.py
│       │   ├── router.py
│       │   ├── schemas.py
│       │   ├── service.py
│       │   └── dependencies.py
│       └── posts/
│           └── ...
├── models/                 # SQLAlchemy models
│   ├── __init__.py
│   ├── base.py
│   ├── user.py
│   └── post.py
├── repositories/           # Data access layer
│   ├── __init__.py
│   ├── base.py
│   └── user.py
├── services/               # Business logic
│   ├── __init__.py
│   └── user.py
└── core/
    ├── __init__.py
    ├── security.py
    └── pagination.py
```

## Application Setup

```python
# src/main.py
from contextlib import asynccontextmanager
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

from src.config import settings
from src.database import engine, Base
from src.api.router import api_router
from src.exceptions import setup_exception_handlers


@asynccontextmanager
async def lifespan(app: FastAPI):
    # Startup
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    yield
    # Shutdown
    await engine.dispose()


def create_app() -> FastAPI:
    app = FastAPI(
        title=settings.PROJECT_NAME,
        version=settings.VERSION,
        openapi_url=f"{settings.API_PREFIX}/openapi.json",
        lifespan=lifespan,
    )

    # Middleware
    app.add_middleware(
        CORSMiddleware,
        allow_origins=settings.CORS_ORIGINS,
        allow_credentials=True,
        allow_methods=["*"],
        allow_headers=["*"],
    )

    # Exception handlers
    setup_exception_handlers(app)

    # Routes
    app.include_router(api_router, prefix=settings.API_PREFIX)

    return app


app = create_app()
```

## Configuration

```python
# src/config.py
from functools import lru_cache
from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):
    model_config = SettingsConfigDict(
        env_file=".env",
        env_file_encoding="utf-8",
        case_sensitive=True,
    )

    # Application
    PROJECT_NAME: str = "My API"
    VERSION: str = "1.0.0"
    DEBUG: bool = False
    API_PREFIX: str = "/api/v1"

    # Database
    DATABASE_URL: str
    DATABASE_POOL_SIZE: int = 5
    DATABASE_MAX_OVERFLOW: int = 10

    # Security
    SECRET_KEY: str
    ACCESS_TOKEN_EXPIRE_MINUTES: int = 60
    REFRESH_TOKEN_EXPIRE_DAYS: int = 7

    # CORS
    CORS_ORIGINS: list[str] = ["http://localhost:3000"]


@lru_cache
def get_settings() -> Settings:
    return Settings()


settings = get_settings()
```

## Pydantic Schemas

```python
# src/api/v1/users/schemas.py
from datetime import datetime
from uuid import UUID
from pydantic import BaseModel, EmailStr, Field, ConfigDict


# Base schemas
class UserBase(BaseModel):
    email: EmailStr
    name: str = Field(..., min_length=1, max_length=100)


# Create schemas
class UserCreate(UserBase):
    password: str = Field(..., min_length=8)


# Update schemas
class UserUpdate(BaseModel):
    email: EmailStr | None = None
    name: str | None = Field(None, min_length=1, max_length=100)


# Response schemas
class UserResponse(UserBase):
    model_config = ConfigDict(from_attributes=True)

    id: UUID
    is_active: bool
    created_at: datetime


class UserListResponse(BaseModel):
    items: list[UserResponse]
    total: int
    page: int
    page_size: int
    pages: int


# Internal schemas
class UserInDB(UserResponse):
    hashed_password: str
```

## Dependency Injection

```python
# src/dependencies.py
from typing import Annotated, AsyncGenerator
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer
from sqlalchemy.ext.asyncio import AsyncSession
from jose import JWTError, jwt

from src.config import settings
from src.database import async_session_maker
from src.models.user import User
from src.repositories.user import UserRepository


oauth2_scheme = OAuth2PasswordBearer(tokenUrl=f"{settings.API_PREFIX}/auth/login")


async def get_db() -> AsyncGenerator[AsyncSession, None]:
    async with async_session_maker() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise


async def get_current_user(
    token: Annotated[str, Depends(oauth2_scheme)],
    db: Annotated[AsyncSession, Depends(get_db)],
) -> User:
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

    repo = UserRepository(db)
    user = await repo.get_by_id(user_id)
    if user is None:
        raise credentials_exception

    return user


async def get_current_active_user(
    current_user: Annotated[User, Depends(get_current_user)],
) -> User:
    if not current_user.is_active:
        raise HTTPException(status_code=400, detail="Inactive user")
    return current_user


# Type aliases for cleaner signatures
DBSession = Annotated[AsyncSession, Depends(get_db)]
CurrentUser = Annotated[User, Depends(get_current_active_user)]
```

## Repository Pattern

```python
# src/repositories/base.py
from typing import Generic, TypeVar, Type
from uuid import UUID
from sqlalchemy import select, func
from sqlalchemy.ext.asyncio import AsyncSession

from src.models.base import Base


ModelType = TypeVar("ModelType", bound=Base)


class BaseRepository(Generic[ModelType]):
    def __init__(self, db: AsyncSession, model: Type[ModelType]):
        self.db = db
        self.model = model

    async def get_by_id(self, id: UUID) -> ModelType | None:
        result = await self.db.execute(
            select(self.model).where(self.model.id == id)
        )
        return result.scalar_one_or_none()

    async def get_all(
        self,
        skip: int = 0,
        limit: int = 100,
    ) -> list[ModelType]:
        result = await self.db.execute(
            select(self.model).offset(skip).limit(limit)
        )
        return list(result.scalars().all())

    async def count(self) -> int:
        result = await self.db.execute(
            select(func.count()).select_from(self.model)
        )
        return result.scalar_one()

    async def create(self, obj: ModelType) -> ModelType:
        self.db.add(obj)
        await self.db.flush()
        await self.db.refresh(obj)
        return obj

    async def update(self, obj: ModelType) -> ModelType:
        await self.db.flush()
        await self.db.refresh(obj)
        return obj

    async def delete(self, obj: ModelType) -> None:
        await self.db.delete(obj)
        await self.db.flush()
```

```python
# src/repositories/user.py
from sqlalchemy import select
from sqlalchemy.ext.asyncio import AsyncSession

from src.models.user import User
from src.repositories.base import BaseRepository


class UserRepository(BaseRepository[User]):
    def __init__(self, db: AsyncSession):
        super().__init__(db, User)

    async def get_by_email(self, email: str) -> User | None:
        result = await self.db.execute(
            select(User).where(User.email == email)
        )
        return result.scalar_one_or_none()

    async def get_active_users(
        self,
        skip: int = 0,
        limit: int = 100,
    ) -> list[User]:
        result = await self.db.execute(
            select(User)
            .where(User.is_active == True)
            .offset(skip)
            .limit(limit)
        )
        return list(result.scalars().all())
```

## Service Layer

```python
# src/services/user.py
from uuid import UUID
from fastapi import HTTPException, status

from src.models.user import User
from src.repositories.user import UserRepository
from src.api.v1.users.schemas import UserCreate, UserUpdate
from src.core.security import get_password_hash, verify_password


class UserService:
    def __init__(self, repository: UserRepository):
        self.repository = repository

    async def get_user(self, user_id: UUID) -> User:
        user = await self.repository.get_by_id(user_id)
        if not user:
            raise HTTPException(
                status_code=status.HTTP_404_NOT_FOUND,
                detail="User not found",
            )
        return user

    async def get_users(
        self,
        skip: int = 0,
        limit: int = 100,
    ) -> tuple[list[User], int]:
        users = await self.repository.get_all(skip=skip, limit=limit)
        total = await self.repository.count()
        return users, total

    async def create_user(self, data: UserCreate) -> User:
        # Check if email exists
        existing = await self.repository.get_by_email(data.email)
        if existing:
            raise HTTPException(
                status_code=status.HTTP_400_BAD_REQUEST,
                detail="Email already registered",
            )

        # Create user
        user = User(
            email=data.email,
            name=data.name,
            hashed_password=get_password_hash(data.password),
        )
        return await self.repository.create(user)

    async def update_user(self, user_id: UUID, data: UserUpdate) -> User:
        user = await self.get_user(user_id)

        if data.email is not None:
            existing = await self.repository.get_by_email(data.email)
            if existing and existing.id != user_id:
                raise HTTPException(
                    status_code=status.HTTP_400_BAD_REQUEST,
                    detail="Email already in use",
                )
            user.email = data.email

        if data.name is not None:
            user.name = data.name

        return await self.repository.update(user)

    async def delete_user(self, user_id: UUID) -> None:
        user = await self.get_user(user_id)
        await self.repository.delete(user)

    async def authenticate(self, email: str, password: str) -> User | None:
        user = await self.repository.get_by_email(email)
        if not user:
            return None
        if not verify_password(password, user.hashed_password):
            return None
        return user
```

## Router Implementation

```python
# src/api/v1/users/router.py
from uuid import UUID
from fastapi import APIRouter, Query, status

from src.dependencies import DBSession, CurrentUser
from src.repositories.user import UserRepository
from src.services.user import UserService
from .schemas import UserCreate, UserUpdate, UserResponse, UserListResponse


router = APIRouter(prefix="/users", tags=["users"])


def get_user_service(db: DBSession) -> UserService:
    return UserService(UserRepository(db))


@router.get("", response_model=UserListResponse)
async def list_users(
    db: DBSession,
    page: int = Query(1, ge=1),
    page_size: int = Query(20, ge=1, le=100),
):
    service = get_user_service(db)
    skip = (page - 1) * page_size
    users, total = await service.get_users(skip=skip, limit=page_size)

    return UserListResponse(
        items=users,
        total=total,
        page=page,
        page_size=page_size,
        pages=(total + page_size - 1) // page_size,
    )


@router.get("/me", response_model=UserResponse)
async def get_current_user_profile(current_user: CurrentUser):
    return current_user


@router.get("/{user_id}", response_model=UserResponse)
async def get_user(user_id: UUID, db: DBSession):
    service = get_user_service(db)
    return await service.get_user(user_id)


@router.post("", response_model=UserResponse, status_code=status.HTTP_201_CREATED)
async def create_user(data: UserCreate, db: DBSession):
    service = get_user_service(db)
    return await service.create_user(data)


@router.patch("/{user_id}", response_model=UserResponse)
async def update_user(
    user_id: UUID,
    data: UserUpdate,
    db: DBSession,
    current_user: CurrentUser,
):
    if current_user.id != user_id:
        raise HTTPException(status_code=403, detail="Cannot update other users")

    service = get_user_service(db)
    return await service.update_user(user_id, data)


@router.delete("/{user_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_user(
    user_id: UUID,
    db: DBSession,
    current_user: CurrentUser,
):
    if current_user.id != user_id:
        raise HTTPException(status_code=403, detail="Cannot delete other users")

    service = get_user_service(db)
    await service.delete_user(user_id)
```

## Exception Handling

```python
# src/exceptions.py
from fastapi import FastAPI, Request, status
from fastapi.responses import JSONResponse
from pydantic import ValidationError


class AppException(Exception):
    def __init__(
        self,
        status_code: int,
        detail: str,
        code: str | None = None,
    ):
        self.status_code = status_code
        self.detail = detail
        self.code = code or "ERROR"


class NotFoundError(AppException):
    def __init__(self, resource: str):
        super().__init__(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"{resource} not found",
            code="NOT_FOUND",
        )


class ConflictError(AppException):
    def __init__(self, detail: str):
        super().__init__(
            status_code=status.HTTP_409_CONFLICT,
            detail=detail,
            code="CONFLICT",
        )


def setup_exception_handlers(app: FastAPI) -> None:
    @app.exception_handler(AppException)
    async def app_exception_handler(request: Request, exc: AppException):
        return JSONResponse(
            status_code=exc.status_code,
            content={
                "status": "error",
                "code": exc.code,
                "detail": exc.detail,
            },
        )

    @app.exception_handler(ValidationError)
    async def validation_exception_handler(request: Request, exc: ValidationError):
        return JSONResponse(
            status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
            content={
                "status": "error",
                "code": "VALIDATION_ERROR",
                "detail": exc.errors(),
            },
        )
```

## Background Tasks

```python
from fastapi import BackgroundTasks

async def send_welcome_email(email: str, name: str):
    # Email sending logic
    pass


@router.post("", response_model=UserResponse)
async def create_user(
    data: UserCreate,
    db: DBSession,
    background_tasks: BackgroundTasks,
):
    service = get_user_service(db)
    user = await service.create_user(data)

    # Queue background task
    background_tasks.add_task(send_welcome_email, user.email, user.name)

    return user
```

## Testing

```python
# tests/test_users.py
import pytest
from httpx import AsyncClient, ASGITransport
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker

from src.main import app
from src.database import Base
from src.dependencies import get_db


# Test database
TEST_DATABASE_URL = "sqlite+aiosqlite:///./test.db"
engine = create_async_engine(TEST_DATABASE_URL, echo=True)
TestSessionLocal = sessionmaker(engine, class_=AsyncSession, expire_on_commit=False)


@pytest.fixture
async def db_session():
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)

    async with TestSessionLocal() as session:
        yield session

    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.drop_all)


@pytest.fixture
async def client(db_session):
    async def override_get_db():
        yield db_session

    app.dependency_overrides[get_db] = override_get_db

    async with AsyncClient(
        transport=ASGITransport(app=app),
        base_url="http://test"
    ) as client:
        yield client

    app.dependency_overrides.clear()


@pytest.mark.asyncio
async def test_create_user(client: AsyncClient):
    response = await client.post(
        "/api/v1/users",
        json={
            "email": "test@example.com",
            "name": "Test User",
            "password": "password123",
        },
    )

    assert response.status_code == 201
    data = response.json()
    assert data["email"] == "test@example.com"
    assert "id" in data


@pytest.mark.asyncio
async def test_get_user_not_found(client: AsyncClient):
    response = await client.get(
        "/api/v1/users/00000000-0000-0000-0000-000000000000"
    )

    assert response.status_code == 404
```

## Best Practices

1. **Use Pydantic for all validation** - leverage automatic validation
2. **Dependency injection everywhere** - makes testing easy
3. **Async all the way** - don't mix sync and async
4. **Repository pattern** - abstract database access
5. **Service layer** - keep business logic out of routes
6. **Type hints everywhere** - better IDE support and docs
7. **Use `Annotated`** - cleaner dependency signatures
8. **Background tasks** - for non-blocking operations
9. **Proper error handling** - custom exceptions with codes
10. **Settings with Pydantic** - validated configuration
