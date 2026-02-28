---
name: fastapi-project-generator
description: Generate complete FastAPI project skeleton with SQLAlchemy, MongoDB, JWT auth, Redis cache, Celery tasks, Docker, and CRUD generators. Use when user wants to create or scaffold a FastAPI project.
---

This skill guides the creation of a production-ready FastAPI project with all necessary components.

## Project Structure

Generate this standardized structure:

```
project/
├── app/
│   ├── api/                    # API modules
│   │   ├── router.py           # Central route registration
│   │   └── [module]/           # Feature module (e.g., user, form)
│   │       ├── __init__.py
│   │       ├── models.py       # SQLAlchemy models
│   │       ├── schemas.py      # Pydantic schemas
│   │       ├── router.py       # APIRouter definitions
│   │       ├── views.py        # View functions
│   │       └── tasks.py        # Celery tasks
│   ├── core/                   # Core components
│   │   ├── __init__.py
│   │   ├── exceptions.py       # Custom exception handlers
│   │   ├── middleware.py       # CORS, compression, rate limiting
│   │   └── oauth2.py           # JWT authentication
│   ├── extensions/             # Extensions layer
│   │   ├── __init__.py
│   │   ├── db.py               # SQLAlchemy sync/async engines
│   │   ├── cache.py            # Redis client
│   │   └── mongo.py            # MongoDB Motor client
│   ├── settings/               # Configuration management
│   │   ├── __init__.py         # Settings loader
│   │   ├── development.py      # Development config
│   │   ├── testing.py          # Testing config
│   │   └── production.py       # Production config
│   ├── utils/                  # Utilities
│   │   ├── __init__.py
│   │   ├── logger.py           # Logging setup
│   │   └── helpers.py          # Helper functions
├── common/                     # Shared modules
│   ├── response.py             # Response wrappers (response_ok/response_err)
│   ├── security.py             # Password hashing, JWT
│   ├── pagination.py           # Custom pagination
│   └── error.py                # Custom exceptions
├── tests/                      # Test suite
├── alembic/                    # Database migrations
│   ├── versions/
│   └── env.py
├── docker/                     # Docker configuration
│   ├── Dockerfile
│   └── docker-compose.yml
├── requirements.txt            # Python dependencies
├── main.py                     # Application entry point (root directory)
├── .env                        # Environment variables
├── .env.example                # Environment template
├── .gitignore
└── alembic.ini
```

## Core Dependencies

Use these versions as baseline:

```txt
# Core
fastapi>=0.115.0
uvicorn[standard]>=0.30.0
pydantic>=2.5.0
pydantic-settings>=2.1.0

# Database
sqlalchemy>=2.0.25
alembic>=1.13.0
asyncmy>=0.2.10  # Async MySQL driver
redis>=5.0.1
motor>=3.3.0     # MongoDB async

# Auth
PyJWT>=2.10.0
pwdlib[argon2]>=0.3.0

# Rate limiting
slowapi>=0.1.9

# Tasks
celery>=5.3.0
redis>=5.0.1

# Utilities
python-multipart>=0.0.6
python-dotenv>=1.0.0
loguru>=0.7.0
httpx>=0.26.0
limits>=5.6.0
```

## Configuration Management

Use multi-environment settings based on `ENV` variable:

```python
# app/settings/__init__.py
import os
from typing import Dict
from functools import lru_cache
from dotenv import load_dotenv
load_dotenv('.env')

from app.settings.development import DevelopmentSettings
from app.settings.testing import TestingSettings
from app.settings.production import ProductionSettings


@lru_cache()
def get_settings():
    env = os.getenv('ENV', None)
    env_config: Dict = {
        "development": DevelopmentSettings(),
        "testing": TestingSettings(),
        "production": ProductionSettings()
    }
    if env is None or env not in env_config:
        raise EnvironmentError("ENV is undefined!")
    return env_config[env]


settings = get_settings()
```

```python
# app/settings/development.py
from pydantic import Field
from typing import List


class DevelopmentSettings:
    # App
    PROJECT_NAME: str = "FastAPI Project"
    VERSION: str = "1.0.0"
    ENVIRONMENT: str = "development"

    # Database
    SQLALCHEMY_DATABASE_ASYNC_URL: str = "mysql+asyncmy://user:pass@localhost:3306/db"
    SQLALCHEMY_DATABASE_SYNC_URL: str = "mysql+pymysql://user:pass@localhost:3306/db"
    SQLALCHEMY_POOL_SIZE: int = 5
    SQLALCHEMY_POOL_RECYCLE: int = 3600
    SQLALCHEMY_POOL_PRE_PING: bool = True
    SQLALCHEMY_ECHO: bool = False

    # Redis
    REDIS_URL: str = "redis://localhost:6379/0"

    # JWT
    JWT_SECRET_KEY: str = "your-secret-key"
    ALGORITHM: str = "HS256"
    ACCESS_TOKEN_EXPIRE_MINUTES: int = 30
    REFRESH_TOKEN_EXPIRE_DAYS: int = 7

    # MongoDB
    MONGODB_URL: str = "mongodb://localhost:27017"
    MONGODB_DB: str = "app"

    # Celery
    CELERY_BROKER_URL: str = "redis://localhost:6379/1"
    CELERY_RESULT_URL: str = "redis://localhost:6379/2"

    # CORS
    BACKEND_CORS_ORIGINS: List[str] = ["http://localhost:3000"]

    # Swagger
    SWAGGER_DOCS_URL: str = "/docs"
    SWAGGER_REDOC_URL: str = "/redoc"
    OPENAPI_URL: str = "/openapi.json"
```

## Response Wrapper

Use unified response format:

```python
# common/response.py
from typing import Union, List, Dict, Tuple, Optional
from starlette import status
from pydantic.types import JsonValue
from fastapi.responses import JSONResponse


class ErrCode(object):
    """Error codes"""
    # Common
    SYSTEM_ERROR = (1000, 'System error')
    QUERY_NOT_EXISTS = (1001, "Data not exists")
    QUERY_HAS_EXISTS = (1002, "Data already exists")
    COMMON_INTERNAL_ERR = (1004, 'Server error')
    COMMON_PERMISSION_ERR = (1005, 'No access permission')
    REQUEST_PARAMS_ERROR = (1006, "Request params error")
    DB_INTEGRITY_ERROR = (1007, 'Data conflict')
    DB_CONNECTION_ERROR = (1008, 'DB connection failed')
    TOO_MANY_REQUEST = (1009, "Too many requests")

    # Auth
    USER_NOT_EXISTS = (2000, 'User not exists')
    USER_HAS_EXISTS = (2001, 'User already exists')
    USER_NOT_ACTIVE = (2002, 'User not active')
    UNAME_OR_PWD_ERROR = (2003, "Username or password error")
    TOKEN_INVALID_ERROR = (2004, 'Token error')
    TOKEN_EXPIRED_ERROR = (2005, 'Token expired')


def response_ok(
    data: Optional[JsonValue] = None,
    msg: str = 'success',
    status_code = status.HTTP_200_OK,
    **kwargs
) -> JSONResponse:
    """Success response"""
    ret = {'errcode': 0, 'errmsg': msg}
    if data is not None:
        ret['data'] = data
    ret.update({k: v for k, v in kwargs.items() if k not in ret})
    return JSONResponse(ret, status_code=status_code)


def response_err(
    errcode: Tuple[int, str],
    errmsg: Optional[str] = None,
    detail: Union[List, Dict, None] = None,
    status_code = status.HTTP_200_OK
) -> JSONResponse:
    """Error response"""
    ret = {"errcode": errcode[0], "errmsg": errcode[1]}
    if errmsg is not None:
        ret['errmsg'] = errmsg
    if detail is not None:
        ret['detail'] = detail
    return JSONResponse(ret, status_code=status_code)
```

## Custom Pagination

```python
# common/pagination.py
from typing import Any, Optional
from fastapi import Query
from pydantic import BaseModel


class PageNumberPagination(BaseModel):
    """Page number based pagination"""
    page: int = Query(default=1, ge=1, description="Page number")
    page_size: int = Query(default=10, ge=1, le=100, description="Page size")
    total: int = 0
    data: list = []

    class Config:
        arbitrary_types_allowed = True
```

## Database Setup

```python
# app/extensions/db.py
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy.ext.asyncio import create_async_engine, async_sessionmaker
from app.settings import settings

# Sync engine
engine = create_engine(
    url=settings.SQLALCHEMY_DATABASE_SYNC_URL,
    pool_size=settings.SQLALCHEMY_POOL_SIZE,
    pool_recycle=settings.SQLALCHEMY_POOL_RECYCLE,
    pool_pre_ping=settings.SQLALCHEMY_POOL_PRE_PING,
    echo=settings.SQLALCHEMY_ECHO
)
session = sessionmaker(
    engine,
    autocommit=False,
    autoflush=False,
    expire_on_commit=False
)

# Async engine
async_engine = create_async_engine(
    url=settings.SQLALCHEMY_DATABASE_ASYNC_URL,
    pool_size=settings.SQLALCHEMY_POOL_SIZE,
    pool_recycle=settings.SQLALCHEMY_POOL_RECYCLE,
    pool_pre_ping=settings.SQLALCHEMY_POOL_PRE_PING,
    echo=settings.SQLALCHEMY_ECHO
)

async_session = async_sessionmaker(
    bind=async_engine,
    expire_on_commit=False,
    autocommit=False,
    autoflush=False
)
```

### Redis Cache

```python
# app/extensions/cache.py
import redis.asyncio as aioredis
from app.settings import settings


async def get_redis() -> aioredis.Redis:
    """Dependency for Redis connection"""
    return aioredis.from_url(settings.REDIS_URL, decode_responses=True)


class RedisCache:
    """Redis cache utility"""

    def __init__(self):
        self.redis: aioredis.Redis = None

    async def connect(self):
        self.redis = await get_redis()

    async def close(self):
        if self.redis:
            await self.redis.close()

    async def get(self, key: str) -> str:
        return await self.redis.get(key)

    async def set(self, key: str, value: str, expire: int = 3600):
        await self.redis.set(key, value, ex=expire)

    async def delete(self, key: str):
        await self.redis.delete(key)
```

### MongoDB

```python
# app/extensions/mongo.py
from motor.motor_asyncio import AsyncIOMotorClient, AsyncIOMotorDatabase
from app.settings import settings


class MongoDB:
    client: AsyncIOMotorClient = None
    db: AsyncIOMotorDatabase = None


mongodb = MongoDB()


async def get_mongo() -> AsyncIOMotorDatabase:
    """Dependency for MongoDB connection"""
    return mongodb.db


async def connect_mongo():
    """Connect to MongoDB on startup"""
    mongodb.client = AsyncIOMotorClient(settings.MONGODB_URL)
    mongodb.db = mongodb.client[settings.MONGODB_DB]


async def close_mongo():
    """Close MongoDB connection on shutdown"""
    if mongodb.client:
        mongodb.client.close()
```

```python
# app/extensions/__init__.py
from app.extensions.db import session, async_session
from app.extensions.cache import get_redis, aioredis
from app.extensions.mongo import get_mongo


async def get_db() -> AsyncSession:
    async with async_session() as db:
        try:
            yield db
            await db.commit()
        except Exception:
            await db.rollback()
            raise
        finally:
            await db.close()
```

## Security & Auth

### Password Hashing (pwdlib)

```python
# common/security.py
from pwdlib import PasswordHash

password_hash = PasswordHash.recommended()


def set_password(password: str) -> str:
    """Hash password"""
    return password_hash.hash(password)


def verify_password(password: str, password_hash_value: str) -> bool:
    """Verify password"""
    return password_hash.verify(password, password_hash_value)
```

### JWT Token

```python
# common/security.py (continued)
from datetime import datetime, timedelta
import jwt
from jwt import PyJWTError
from typing import Union, Any, Optional
from app.settings import settings


def create_access_token(data: dict, expires_delta: Optional[timedelta] = None) -> str:
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=settings.ACCESS_TOKEN_EXPIRE_MINUTES)
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, settings.JWT_SECRET_KEY, algorithm=settings.ALGORITHM)


def create_refresh_token(data: dict, expires_delta: Optional[timedelta] = None) -> str:
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(days=settings.REFRESH_TOKEN_EXPIRE_DAYS)
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, settings.JWT_REFRESH_SECRET_KEY, algorithm=settings.ALGORITHM)
```

### OAuth2 with Scopes

```python
# app/core/oauth2.py
from fastapi import Security, Depends
from fastapi.security import OAuth2PasswordBearer
from jose import JWTError, jwt
from common.response import ErrCode, response_err
from common.security import create_access_token


oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/api/v1/auth/login")


async def get_current_active_user(current_user = Security(get_current_user, scopes=['user'])):
    return current_user


async def get_current_admin_user(current_user = Security(get_current_user, scopes=['admin'])):
    return current_user
```

## Exception Handlers

```python
# app/core/exceptions.py
from fastapi import FastAPI, Request
from fastapi.exceptions import RequestValidationError
from sqlalchemy.exc import IntegrityError, OperationalError
from common.response import ErrCode, response_err


def register_exceptions(app: FastAPI):
    @app.exception_handler(Exception)
    async def internal_err_handler(request: Request, exc: Exception):
        return response_err(ErrCode.COMMON_INTERNAL_ERR, detail=str(exc))

    @app.exception_handler(RequestValidationError)
    async def validation_error_handler(request: Request, exc: RequestValidationError):
        return response_err(ErrCode.REQUEST_PARAMS_ERROR, detail=exc.errors())

    @app.exception_handler(IntegrityError)
    async def integrity_error_handler(request: Request, exc: IntegrityError):
        return response_err(ErrCode.DB_INTEGRITY_ERROR, detail=str(exc.detail))

    @app.exception_handler(OperationalError)
    async def db_connection_error(request: Request, exc: OperationalError):
        return response_err(ErrCode.DB_CONNECTION_ERROR, detail=str(exc))
```

## API Module Template

```python
# app/api/user/views.py
from fastapi import APIRouter, Query, Body, Depends, Security, BackgroundTasks
from sqlalchemy import select
from typing import Optional

from common.response import ErrCode, response_ok, response_err
from common.pagation import PageNumberPagination
from common.security import set_password
from app.extensions import get_db, AsyncSession, get_redis, aioredis
from app.core.oauth2 import get_current_active_user
from app.api.user.models import User
from app.api.user.schemas import UserCreateSchema
from app.api.user.tasks import generate_user_avatar_task


router = APIRouter()


@router.post('/create', summary='Create user')
async def create_user(
    background_tasks: BackgroundTasks,
    args: UserCreateSchema,
    db: AsyncSession = Depends(get_db),
    redis: aioredis.Redis = Depends(get_redis),
    current_user: User = Security(get_current_active_user, scopes=['admin'])
):
    # Check if user exists
    existing = await db.scalar(select(User).where(User.username == args.username))
    if existing:
        return response_err(ErrCode.UNAME_HAS_OCCUPIED)

    user = User(
        username=args.username,
        password_hash=set_password(args.password),
        name=args.name or args.username,
        is_active=True,
        role=args.role
    )
    db.add(user)
    await db.commit()
    background_tasks.add_task(generate_user_avatar_task, user.id)
    return response_ok(data=user.to_dict(include=['id']))


@router.get('/list', summary='User list')
async def user_list(
    username: str = Query(default=None, description='Username'),
    paginate: PageNumberPagination = Depends(),
    current_user: User = Security(get_current_active_user, scopes=['admin'])
):
    query_filter = []
    if username:
        query_filter.append(User.username.ilike(f"%{username}%"))

    result_data = await paginate(User, query_filter)
    return response_ok(**result_data)


@router.get('/info', summary='User info')
async def user_info(
    current_user: User = Security(get_current_active_user, scopes=['user'])
):
    return response_ok(data=current_user.to_dict())
```

## Main Application Entry

```python
# main.py (project root)
from contextlib import asynccontextmanager
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

from app.settings import settings
from app.api.router import register_router
from app.core.exceptions import register_exceptions
from app.core.middleware import register_middleware


@asynccontextmanager
async def lifespan(app: FastAPI):
    register_router(app)
    yield


app = FastAPI(
    title=settings.PROJECT_NAME,
    version=settings.VERSION,
    lifespan=lifespan,
    docs_url=settings.SWAGGER_DOCS_URL,
    redoc_url=settings.SWAGGER_REDOC_URL,
    openapi_url=settings.OPENAPI_URL,
)

register_middleware(app)
register_exceptions(app)
```

## Docker Configuration

```dockerfile
# docker/Dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

```yaml
# docker/docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8000:8000"
    environment:
      - ENV=development
      - SQLALCHEMY_DATABASE_ASYNC_URL=mysql+asyncmy://user:pass@db:3306/app
      - REDIS_URL=redis://redis:6379/0
    depends_on:
      - db
      - redis
    volumes:
      - .:/app

  db:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: app
    command: --default-authentication-plugin=mysql_native_password

  redis:
    image: redis:7-alpine

volumes:
  mysql_data:
  redis_data:
```

## Implementation Workflow

When user requests a FastAPI project:

1. **Gather Requirements**: Project name, database, auth requirements
2. **Generate Structure**: Create directory structure
3. **Create Config**: Settings with multi-environment support
4. **Setup Response**: Response wrappers and error codes
5. **Setup Auth**: JWT with pwdlib password hashing
6. **Add Database**: SQLAlchemy with sync/async support
7. **Add Cache**: Redis integration
8. **Add Tasks**: Celery configuration
9. **Add Docker**: Containerization files
10. **Write README**: Documentation

Generate production-grade code with proper error handling, logging, and unified response format.
