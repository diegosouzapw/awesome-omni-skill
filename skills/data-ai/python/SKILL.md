---
name: python
description: Python programming with type hints, async/await, decorators, and package management. Use for .py files and data science.
---

# Python

Modern Python development with type hints, async/await, and best practices.

## When to Use

- Working with `.py` files
- Building APIs with FastAPI/Django
- Data analysis with pandas/numpy
- Scripting and automation

## Quick Start

```python
from typing import Optional, List

def process_items(
    items: list[str],
    transform: Optional[callable] = None,
) -> list[str]:
    if transform:
        return [transform(item) for item in items]
    return items
```

## Core Concepts

### Type Hints

```python
from typing import TypeVar, Generic
from collections.abc import Callable, Iterator

def process_items(
    items: list[str],
    transform: Callable[[str], str] | None = None,
) -> list[str]:
    if transform:
        return [transform(item) for item in items]
    return items

# Use TypeVar for generics
T = TypeVar('T')

def first(items: list[T]) -> T | None:
    return items[0] if items else None
```

### Dataclasses & Pydantic

```python
from dataclasses import dataclass, field
from pydantic import BaseModel, Field

# Dataclass for simple data containers
@dataclass
class User:
    name: str
    email: str
    tags: list[str] = field(default_factory=list)

# Pydantic for validation
class UserCreate(BaseModel):
    name: str = Field(..., min_length=1, max_length=100)
    email: str = Field(..., pattern=r'^[\w\.-]+@[\w\.-]+\.\w+$')
```

## Common Patterns

### Async Patterns

```python
import asyncio

async def fetch_all(urls: list[str]) -> list[Response]:
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_one(session, url) for url in urls]
        return await asyncio.gather(*tasks)

# Use TaskGroup for structured concurrency (3.11+)
async def process_batch(items: list[Item]) -> list[Result]:
    results = []
    async with asyncio.TaskGroup() as tg:
        for item in items:
            tg.create_task(process_item(item, results))
    return results
```

### Context Managers

```python
from contextlib import contextmanager, asynccontextmanager

@contextmanager
def managed_resource() -> Iterator[Resource]:
    resource = Resource()
    try:
        yield resource
    finally:
        resource.cleanup()
```

## Best Practices

**Do**:

- Use type hints for function signatures
- Use `pyproject.toml` for project configuration
- Use virtual environments (`venv`, `poetry`)
- Use generators for large datasets

**Don't**:

- Use mutable default arguments (`def f(x=[]`)
- Use `import *` (pollutes namespace)
- Catch bare `except:` (catch specific exceptions)
- Use `assert` for input validation

## Troubleshooting

| Error                        | Cause                  | Solution                           |
| ---------------------------- | ---------------------- | ---------------------------------- |
| `ModuleNotFoundError`        | Package not installed  | Run `pip install package`          |
| `IndentationError`           | Mixed tabs/spaces      | Use consistent 4-space indentation |
| `TypeError: unhashable type` | Using list as dict key | Use tuple instead                  |

## References

- [Python Official Docs](https://docs.python.org/3/)
- [Real Python](https://realpython.com/)
