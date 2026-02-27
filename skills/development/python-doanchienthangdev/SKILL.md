---
name: python
description: Python development with type hints, async patterns, testing, and modern best practices
category: languages
triggers:
  - python
  - py
  - pip
  - pydantic
  - fastapi
  - django
---

# Python

Modern **Python development** following industry best practices. This skill covers type hints, async programming, data validation, testing, and production-ready patterns used by top engineering teams.

## Purpose

Write clean, maintainable Python code:

- Use type hints for better code quality
- Implement async patterns for I/O operations
- Validate data with Pydantic
- Structure projects properly
- Write comprehensive tests
- Follow PEP standards

## Features

### 1. Type Hints and Annotations

```python
from typing import Optional, List, Dict, Union, Callable, TypeVar, Generic
from dataclasses import dataclass
from datetime import datetime

# Basic type hints
def greet(name: str) -> str:
    return f"Hello, {name}"

def process_items(items: List[str], limit: int = 10) -> Dict[str, int]:
    return {item: len(item) for item in items[:limit]}

# Optional and Union
def find_user(user_id: str) -> Optional[User]:
    return db.users.get(user_id)

def parse_input(value: Union[str, int]) -> str:
    return str(value)

# Callable types
Handler = Callable[[str, int], bool]

def register_handler(name: str, handler: Handler) -> None:
    handlers[name] = handler

# Generic types
T = TypeVar('T')

class Repository(Generic[T]):
    def __init__(self, model: type[T]) -> None:
        self.model = model

    def find_by_id(self, id: str) -> Optional[T]:
        ...

    def find_all(self) -> List[T]:
        ...

    def create(self, data: Dict) -> T:
        ...

# Type aliases
UserId = str
UserMap = Dict[UserId, User]
EventHandler = Callable[[Event], None]
```

### 2. Dataclasses and Pydantic

```python
from dataclasses import dataclass, field
from typing import Optional, List
from datetime import datetime
from pydantic import BaseModel, EmailStr, Field, validator
from enum import Enum

# Dataclasses for simple data structures
@dataclass
class User:
    id: str
    email: str
    name: str
    created_at: datetime = field(default_factory=datetime.now)
    roles: List[str] = field(default_factory=list)

@dataclass(frozen=True)
class Point:
    x: float
    y: float

    def distance_to(self, other: 'Point') -> float:
        return ((self.x - other.x) ** 2 + (self.y - other.y) ** 2) ** 0.5

# Pydantic for validation
class UserRole(str, Enum):
    ADMIN = "admin"
    USER = "user"
    GUEST = "guest"

class UserCreate(BaseModel):
    email: EmailStr
    password: str = Field(..., min_length=8)
    name: str = Field(..., min_length=2, max_length=100)
    role: UserRole = UserRole.USER

    @validator('password')
    def password_strength(cls, v: str) -> str:
        if not any(c.isupper() for c in v):
            raise ValueError('Password must contain uppercase letter')
        if not any(c.isdigit() for c in v):
            raise ValueError('Password must contain digit')
        return v

    class Config:
        str_strip_whitespace = True

class UserResponse(BaseModel):
    id: str
    email: str
    name: str
    role: UserRole
    created_at: datetime

    class Config:
        from_attributes = True  # For ORM compatibility

class PaginatedResponse(BaseModel, Generic[T]):
    data: List[T]
    total: int
    page: int
    limit: int
    has_more: bool
```

### 3. Async Programming

```python
import asyncio
from typing import List, Dict, Any
import aiohttp
import asyncpg

# Basic async functions
async def fetch_data(url: str) -> Dict[str, Any]:
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.json()

# Concurrent requests
async def fetch_all(urls: List[str]) -> List[Dict[str, Any]]:
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_url(session, url) for url in urls]
        return await asyncio.gather(*tasks)

async def fetch_url(session: aiohttp.ClientSession, url: str) -> Dict[str, Any]:
    async with session.get(url) as response:
        return await response.json()

# Database operations
class Database:
    def __init__(self, dsn: str):
        self.dsn = dsn
        self.pool: Optional[asyncpg.Pool] = None

    async def connect(self) -> None:
        self.pool = await asyncpg.create_pool(
            self.dsn,
            min_size=5,
            max_size=20,
        )

    async def disconnect(self) -> None:
        if self.pool:
            await self.pool.close()

    async def fetch_user(self, user_id: str) -> Optional[Dict]:
        async with self.pool.acquire() as conn:
            row = await conn.fetchrow(
                "SELECT * FROM users WHERE id = $1",
                user_id
            )
            return dict(row) if row else None

    async def create_user(self, email: str, name: str) -> Dict:
        async with self.pool.acquire() as conn:
            row = await conn.fetchrow(
                """
                INSERT INTO users (email, name)
                VALUES ($1, $2)
                RETURNING *
                """,
                email, name
            )
            return dict(row)

# Context managers
class AsyncResource:
    async def __aenter__(self) -> 'AsyncResource':
        await self.setup()
        return self

    async def __aexit__(self, exc_type, exc_val, exc_tb) -> None:
        await self.cleanup()

# Rate limiting
class RateLimiter:
    def __init__(self, rate: int, per: float):
        self.rate = rate
        self.per = per
        self.semaphore = asyncio.Semaphore(rate)

    async def acquire(self) -> None:
        await self.semaphore.acquire()
        asyncio.create_task(self._release())

    async def _release(self) -> None:
        await asyncio.sleep(self.per)
        self.semaphore.release()
```

### 4. Error Handling

```python
from typing import TypeVar, Generic
from dataclasses import dataclass

# Custom exceptions
class AppError(Exception):
    def __init__(self, message: str, code: str, status_code: int = 500):
        self.message = message
        self.code = code
        self.status_code = status_code
        super().__init__(message)

class NotFoundError(AppError):
    def __init__(self, resource: str, id: str):
        super().__init__(
            f"{resource} with id {id} not found",
            "NOT_FOUND",
            404
        )

class ValidationError(AppError):
    def __init__(self, field: str, message: str):
        super().__init__(
            f"Validation error on {field}: {message}",
            "VALIDATION_ERROR",
            400
        )

# Result type pattern
T = TypeVar('T')
E = TypeVar('E', bound=Exception)

@dataclass
class Ok(Generic[T]):
    value: T

    def is_ok(self) -> bool:
        return True

    def is_err(self) -> bool:
        return False

@dataclass
class Err(Generic[E]):
    error: E

    def is_ok(self) -> bool:
        return False

    def is_err(self) -> bool:
        return True

Result = Ok[T] | Err[E]

def divide(a: float, b: float) -> Result[float, ValueError]:
    if b == 0:
        return Err(ValueError("Division by zero"))
    return Ok(a / b)

# Usage
result = divide(10, 2)
if result.is_ok():
    print(f"Result: {result.value}")
else:
    print(f"Error: {result.error}")
```

### 5. Testing with Pytest

```python
import pytest
from unittest.mock import Mock, patch, AsyncMock
from datetime import datetime

# Basic tests
def test_create_user():
    user = create_user("test@example.com", "Test User")
    assert user.email == "test@example.com"
    assert user.name == "Test User"

# Parametrized tests
@pytest.mark.parametrize("email,expected", [
    ("valid@example.com", True),
    ("invalid-email", False),
    ("", False),
])
def test_validate_email(email: str, expected: bool):
    assert validate_email(email) == expected

# Fixtures
@pytest.fixture
def user():
    return User(
        id="test-id",
        email="test@example.com",
        name="Test User"
    )

@pytest.fixture
def db():
    db = Database(":memory:")
    db.connect()
    yield db
    db.disconnect()

def test_user_creation(user: User):
    assert user.id == "test-id"

# Async tests
@pytest.mark.asyncio
async def test_fetch_user():
    db = AsyncMock()
    db.fetch_user.return_value = {"id": "1", "name": "Test"}

    result = await db.fetch_user("1")
    assert result["name"] == "Test"

# Mocking
def test_external_api_call():
    with patch('module.requests.get') as mock_get:
        mock_get.return_value.json.return_value = {"data": "value"}

        result = fetch_external_data()

        assert result == {"data": "value"}
        mock_get.assert_called_once()

# Exception testing
def test_not_found_raises():
    with pytest.raises(NotFoundError) as exc_info:
        find_user("nonexistent")

    assert "not found" in str(exc_info.value)

# Test class organization
class TestUserService:
    @pytest.fixture(autouse=True)
    def setup(self):
        self.service = UserService()
        self.mock_repo = Mock()
        self.service.repo = self.mock_repo

    def test_create_user_success(self):
        self.mock_repo.create.return_value = User(id="1", email="test@example.com")

        user = self.service.create_user("test@example.com", "password")

        assert user.email == "test@example.com"

    def test_create_user_duplicate_email(self):
        self.mock_repo.find_by_email.return_value = User(id="1", email="test@example.com")

        with pytest.raises(ValidationError):
            self.service.create_user("test@example.com", "password")
```

### 6. Project Structure

```
project/
├── src/
│   └── myapp/
│       ├── __init__.py
│       ├── main.py
│       ├── config.py
│       ├── models/
│       │   ├── __init__.py
│       │   └── user.py
│       ├── services/
│       │   ├── __init__.py
│       │   └── user_service.py
│       ├── repositories/
│       │   ├── __init__.py
│       │   └── user_repository.py
│       └── api/
│           ├── __init__.py
│           ├── routes.py
│           └── dependencies.py
├── tests/
│   ├── __init__.py
│   ├── conftest.py
│   ├── unit/
│   │   └── test_user_service.py
│   └── integration/
│       └── test_api.py
├── pyproject.toml
├── requirements.txt
└── README.md
```

### 7. Configuration Management

```python
from pydantic_settings import BaseSettings
from functools import lru_cache
from typing import Optional

class Settings(BaseSettings):
    app_name: str = "MyApp"
    debug: bool = False

    database_url: str
    redis_url: Optional[str] = None

    secret_key: str
    jwt_algorithm: str = "HS256"
    jwt_expire_minutes: int = 30

    cors_origins: list[str] = ["http://localhost:3000"]

    class Config:
        env_file = ".env"
        env_file_encoding = "utf-8"

@lru_cache
def get_settings() -> Settings:
    return Settings()

# Usage
settings = get_settings()
print(settings.database_url)
```

## Use Cases

### FastAPI Application
```python
from fastapi import FastAPI, Depends, HTTPException
from sqlalchemy.orm import Session

app = FastAPI()

@app.get("/users/{user_id}", response_model=UserResponse)
async def get_user(user_id: str, db: Session = Depends(get_db)):
    user = await db.users.find_by_id(user_id)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user

@app.post("/users", response_model=UserResponse, status_code=201)
async def create_user(data: UserCreate, db: Session = Depends(get_db)):
    existing = await db.users.find_by_email(data.email)
    if existing:
        raise HTTPException(status_code=400, detail="Email already exists")
    return await db.users.create(data.dict())
```

### CLI Application
```python
import click

@click.group()
def cli():
    """My CLI application."""
    pass

@cli.command()
@click.option('--name', prompt='Your name', help='User name')
@click.option('--count', default=1, help='Number of greetings')
def hello(name: str, count: int):
    """Greet the user."""
    for _ in range(count):
        click.echo(f'Hello, {name}!')

if __name__ == '__main__':
    cli()
```

## Best Practices

### Do's
- Use type hints everywhere
- Use dataclasses or Pydantic for data
- Use async for I/O operations
- Follow PEP 8 and PEP 257
- Use virtual environments
- Write comprehensive docstrings
- Use pytest for testing

### Don'ts
- Don't use mutable default arguments
- Don't catch bare exceptions
- Don't ignore type errors
- Don't use global state
- Don't skip error handling
- Don't write untestable code

## References

- [Python Documentation](https://docs.python.org/3/)
- [PEP 8 Style Guide](https://peps.python.org/pep-0008/)
- [Pydantic Documentation](https://docs.pydantic.dev/)
- [Pytest Documentation](https://docs.pytest.org/)
