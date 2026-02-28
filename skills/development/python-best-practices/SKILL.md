---
name: python-best-practices
description: Python development best practices, patterns, and conventions. Use when writing Python code, reviewing .py files, discussing pytest, asyncio, type hints, pydantic, dataclasses, or Python project structure. Triggers on mentions of Python, pytest, mypy, ruff, black, FastAPI, Django, Flask.
---

# Python Best Practices Skill

This skill provides guidance on Python development best practices, patterns, and conventions.

## Code Style

### Naming Conventions
- **Variables/Functions**: `snake_case`
- **Classes**: `PascalCase`
- **Constants**: `UPPER_SNAKE_CASE`
- **Private**: `_single_leading_underscore`
- **"Dunder"**: `__double_underscore__` (reserved for Python)

### Formatting
- 4 spaces for indentation (never tabs)
- Max line length: 88-120 characters (project dependent)
- Use Black or Ruff for auto-formatting
- Blank lines: 2 between top-level definitions, 1 within classes

## Type Hints

```python
# Function signatures
def process_data(items: list[str], limit: int = 10) -> dict[str, int]:
    ...

# Optional values
def find_user(user_id: int) -> User | None:
    ...

# Complex types
from typing import TypeVar, Generic, Protocol

T = TypeVar('T')

class Repository(Protocol[T]):
    def get(self, id: int) -> T | None: ...
    def save(self, entity: T) -> None: ...
```

## Error Handling

```python
# DO: Specific exceptions
try:
    user = get_user(user_id)
except UserNotFoundError:
    logger.warning(f"User {user_id} not found")
    return None

# DON'T: Bare except
try:
    user = get_user(user_id)
except:  # Bad - catches everything including KeyboardInterrupt
    pass

# Custom exceptions
class DomainError(Exception):
    """Base class for domain exceptions."""
    pass

class UserNotFoundError(DomainError):
    def __init__(self, user_id: int):
        self.user_id = user_id
        super().__init__(f"User {user_id} not found")
```

## Data Classes and Models

```python
from dataclasses import dataclass, field
from datetime import datetime

@dataclass
class User:
    id: int
    name: str
    email: str
    created_at: datetime = field(default_factory=datetime.now)
    tags: list[str] = field(default_factory=list)

    def __post_init__(self):
        self.email = self.email.lower()

# For validation, use Pydantic
from pydantic import BaseModel, EmailStr, Field

class UserCreate(BaseModel):
    name: str = Field(min_length=1, max_length=100)
    email: EmailStr

    model_config = {"str_strip_whitespace": True}
```

## Async Patterns

```python
import asyncio
from typing import AsyncIterator

# Async context manager
class AsyncDatabaseConnection:
    async def __aenter__(self) -> "AsyncDatabaseConnection":
        await self.connect()
        return self

    async def __aexit__(self, exc_type, exc_val, exc_tb):
        await self.close()

# Async generator
async def stream_results(query: str) -> AsyncIterator[dict]:
    async with get_connection() as conn:
        async for row in conn.execute(query):
            yield dict(row)

# Concurrent execution
async def fetch_all(urls: list[str]) -> list[Response]:
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_one(session, url) for url in urls]
        return await asyncio.gather(*tasks)
```

## Testing

```python
import pytest
from unittest.mock import Mock, patch, AsyncMock

# Fixtures
@pytest.fixture
def sample_user():
    return User(id=1, name="Test", email="test@example.com")

@pytest.fixture
def mock_db():
    with patch("myapp.database.get_connection") as mock:
        yield mock

# Parametrized tests
@pytest.mark.parametrize("input,expected", [
    ("hello", "HELLO"),
    ("World", "WORLD"),
    ("", ""),
])
def test_uppercase(input: str, expected: str):
    assert uppercase(input) == expected

# Async tests
@pytest.mark.asyncio
async def test_async_fetch():
    result = await fetch_data("http://example.com")
    assert result.status == 200

# Exception testing
def test_invalid_input_raises():
    with pytest.raises(ValueError, match="must be positive"):
        process_value(-1)
```

## Project Structure

```
myproject/
├── src/
│   └── myproject/
│       ├── __init__.py
│       ├── core/           # Business logic
│       ├── api/            # API layer
│       ├── models/         # Data models
│       └── utils/          # Utilities
├── tests/
│   ├── conftest.py         # Shared fixtures
│   ├── unit/
│   └── integration/
├── pyproject.toml          # Project config
├── README.md
└── .env.example
```

## Dependencies

```toml
# pyproject.toml
[project]
name = "myproject"
version = "0.1.0"
requires-python = ">=3.11"
dependencies = [
    "pydantic>=2.0",
    "httpx>=0.24",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.0",
    "pytest-asyncio>=0.21",
    "ruff>=0.1",
    "mypy>=1.0",
]

[tool.ruff]
line-length = 88
target-version = "py311"

[tool.mypy]
strict = true
```

## Common Anti-Patterns to Avoid

1. **Mutable default arguments**
   ```python
   # Bad
   def add_item(item, items=[]):
       items.append(item)
       return items

   # Good
   def add_item(item, items=None):
       if items is None:
           items = []
       items.append(item)
       return items
   ```

2. **Using `type()` for type checking**
   ```python
   # Bad
   if type(x) == list:

   # Good
   if isinstance(x, list):
   ```

3. **Catching too broadly**
   ```python
   # Bad
   except Exception:

   # Good
   except (ValueError, TypeError):
   ```

4. **String concatenation in loops**
   ```python
   # Bad
   result = ""
   for item in items:
       result += str(item)

   # Good
   result = "".join(str(item) for item in items)
   ```
