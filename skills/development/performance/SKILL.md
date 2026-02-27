---
name: performance
description: Apply performance optimization best practices. Use when profiling slow code, optimizing memory usage, implementing caching, handling large datasets, or improving database queries.
user-invocable: false
---

# Performance Optimization

**Profile before optimizing.** Never guess where bottlenecks are.

```bash
uv run python -m cProfile -s cumtime script.py

uv add --dev memray
uv run memray run script.py
uv run memray flamegraph memray-output.bin
```

## Caching Strategies

### In-Memory Caching

```python
from functools import lru_cache, cache

@lru_cache(maxsize=128)
def expensive_calculation(n: int) -> int:
    return sum(i**2 for i in range(n))

@cache
def fibonacci(n: int) -> int:
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)
```

### TTL Cache

```python
from cachetools.func import ttl_cache

@ttl_cache(maxsize=100, ttl=600)
def fetch_user_data(user_id: int) -> dict:
    return api.get_user(user_id)
```

## Memory Optimization

### Generators for Large Data

```python
from collections.abc import Iterator

# Bad - loads entire file into memory
def read_all_lines(filepath: str) -> list[str]:
    with open(filepath) as f:
        return f.readlines()

# Good - yields one line at a time
def read_lines(filepath: str) -> Iterator[str]:
    with open(filepath) as f:
        for line in f:
            yield line.strip()
```

### __slots__ and Data Classes

```python
from dataclasses import dataclass

@dataclass(slots=True)
class User:
    id: int
    name: str
    email: str
```

## Bulk Operations

```python
# Bad - N individual inserts
for user in users:
    db.execute("INSERT INTO users (name) VALUES (?)", (user.name,))

# Good - single bulk insert
db.executemany(
    "INSERT INTO users (name) VALUES (?)",
    [(u.name,) for u in users]
)
```

## Async/Concurrent Processing

```python
import asyncio
import httpx

async def fetch_all(urls: list[str]) -> list[dict]:
    async with httpx.AsyncClient() as client:
        tasks = [client.get(url) for url in urls]
        responses = await asyncio.gather(*tasks)
        return [r.json() for r in responses]

# With concurrency limit
async def fetch_with_limit(urls: list[str], limit: int = 10) -> list[dict]:
    semaphore = asyncio.Semaphore(limit)

    async def fetch_one(url: str) -> dict:
        async with semaphore:
            async with httpx.AsyncClient() as client:
                response = await client.get(url)
                return response.json()

    return await asyncio.gather(*[fetch_one(url) for url in urls])
```

## Database Optimization

### Avoid N+1 Queries

```python
# Bad - N+1 queries
users = session.query(User).all()
for user in users:
    print(user.posts)  # Each access triggers a query

# Good - eager loading
users = session.query(User).options(joinedload(User.posts)).all()
```

### Cursor Pagination

```python
def get_users_cursor(
    session: Session,
    after_id: int | None = None,
    limit: int = 20,
) -> list[User]:
    stmt = select(User).order_by(User.id).limit(limit)
    if after_id:
        stmt = stmt.where(User.id > after_id)
    return list(session.scalars(stmt))
```

## Lazy Loading

```python
from functools import cached_property

class DataProcessor:
    def __init__(self, config_path: str) -> None:
        self._config_path = config_path

    @cached_property
    def config(self) -> dict:
        return load_config(self._config_path)
```

## Quick Reference

| Technique | Use Case |
|-----------|----------|
| `@lru_cache` | Pure functions, repeated calls |
| `@ttl_cache` | Data that expires |
| Generators | Large files, streaming data |
| `__slots__` | Many instances of same class |
| Bulk operations | Database inserts/updates |
| `asyncio.gather` | Concurrent I/O operations |
| Eager loading | Avoid N+1 database queries |
| Cursor pagination | Large result sets |

## See Also

- **conventions** — general coding conventions
