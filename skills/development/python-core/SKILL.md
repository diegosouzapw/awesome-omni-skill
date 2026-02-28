---
name: python-core
description: Create, write, build, debug, test, refactor, and optimize Python 3.10+ applications across all domains (data science, backend APIs, scripting, automation). Manage dependencies with uv (preferred), poetry, or pip. Implement type hints (typing module, generics, protocols), write tests with pytest (fixtures, parametrize, mocking), work with pandas DataFrames (creation, selection, groupby, merge), use dataclasses and decorators, handle errors with logging integration, and follow TDD workflows (Red-Green-Refactor). Configure virtual environments, pyproject.toml, and static analysis tools (mypy, pyright). Use when implementing Python features, fixing bugs, writing tests, managing packages, analyzing data, or building Python projects.
license: MIT
metadata:
  version: 1.0.0
  audience: developers
  workflow: python-development
---

## Quick Start

**Prerequisites:**
- Python 3.10+ installed (`python --version`)
- Dependency manager: uv (recommended), poetry, or pip+venv
- IDE with Python language server (VS Code, PyCharm)

**Tools Used:** Read, Write, Edit, Bash (for uv/poetry/pip commands), LSP diagnostics

**Dependency Management Decision Tree:**
```
New project? → Use uv: uv init, uv add, uv sync
Existing poetry? → Use poetry: poetry install, poetry add
Legacy/simple? → Use pip+venv: python -m venv, pip install
```

**Basic Usage:**
1. Set up environment (see decision tree)
2. Write code with type hints
3. Write tests first (TDD)
4. Run tests: `pytest`
5. Verify types: `mypy .`

## What I Do

- Create Python 3.10+ applications (data science, backend, scripting, automation)
- Manage dependencies with uv, poetry, or pip
- Implement type hints (typing module, generics, protocols)
- Write tests with pytest (fixtures, parametrize, mocking)
- Work with pandas DataFrames (selection, groupby, merge)
- Use Python patterns (decorators, comprehensions, generators, dataclasses)
- Handle errors with logging integration
- Follow TDD workflow (Red-Green-Refactor)

## When to Use Me

Use this skill when you:
- Create, refactor, or debug Python 3.10+ code
- Set up projects with dependency management (uv, poetry, pip)
- Implement type hints or fix type errors
- Write pytest tests (fixtures, parametrize, mocking)
- Work with pandas DataFrames
- Use Python patterns (decorators, generators, dataclasses)
- Handle errors with logging
- Follow TDD workflows

## Python Version Features

| Version | Features |
|---------|----------|
| **3.10** | Pattern matching, union types (`\|`), `TypeAlias` |
| **3.11** | Exception groups (`except*`), `Self` type |
| **3.12** | Type parameters (`def func[T]`), f-string improvements |

## Quick Reference Tables

### Built-in Functions
| Function | Purpose |
|----------|---------|
| `map(func, iterable)` | Apply function to each item |
| `filter(func, iterable)` | Keep items where condition is True |
| `zip(*iterables)` | Combine iterables element-wise |
| `enumerate(iterable)` | Add index to iterable |

### String Operations
| Operation | Example |
|-----------|---------|
| f-strings | `f"Hello {name}"` |
| `.join()` | `", ".join(['a', 'b'])` |
| `.split()` | `"a,b".split(",")` |

### Collection Methods
| Type | Common Methods |
|------|----------------|
| **list** | `.append()`, `.extend()`, `.pop()`, `.sort()` |
| **dict** | `.get()`, `.keys()`, `.values()`, `.items()` |
| **set** | `.add()`, `.remove()`, `.union()` |

## Type Hints Basics

```python
from typing import Optional

def greet(name: str) -> str:
    return f"Hello {name}"

def process_items(items: list[int]) -> dict[str, int]:
    return {"count": len(items), "sum": sum(items)}

def find_user(user_id: int) -> Optional[str]:
    return users.get(user_id)

def parse_value(val: str | int) -> int:  # Python 3.10+
    return int(val)
```

**See [references/type-hints.md](references/type-hints.md) for Generics and Protocols.**

## TDD Workflow (Red-Green-Refactor)

1. **Red**: Write failing test
2. **Green**: Write minimal code to pass
3. **Refactor**: Improve while keeping tests green

```python
# Red: Test fails (function doesn't exist)
def test_total():
    assert calculate_total([10, 20, 30]) == 60

# Green: Make it pass
def calculate_total(items):
    return sum(items)

# Refactor: Add types and edge cases
def calculate_total(items: list[int]) -> int:
    return sum(items) if items else 0
```

**See [references/pytest.md](references/pytest.md) for fixtures and mocking.**

## Error Handling with Logging

```python
import logging

logger = logging.getLogger(__name__)

# Try/Except/Finally
try:
    result = risky_operation()
except ValueError as e:
    logger.error(f"Invalid value: {e}")
    raise
finally:
    cleanup_resources()

# Custom Exceptions
class DataValidationError(Exception):
    pass

def validate_data(data: dict) -> None:
    if "required_field" not in data:
        raise DataValidationError("required_field missing")

# Context Manager for cleanup
from contextlib import contextmanager

@contextmanager
def db_connection(url: str):
    conn = connect(url)
    try:
        yield conn
    finally:
        conn.close()
```

## Examples

### Example 1: Dataclass with Type Hints
```python
from dataclasses import dataclass
from typing import List, Optional

@dataclass
class User:
    id: int
    name: str
    email: str
    tags: List[str] = None
    
    def __post_init__(self):
        if self.tags is None:
            self.tags = []

# Usage
user = User(id=1, name="Alice", email="alice@example.com")
user.tags.append("admin")
```

### Example 2: Decorator Pattern
```python
import functools
import time
import logging

logger = logging.getLogger(__name__)

def timing_decorator(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        elapsed = time.time() - start
        logger.info(f"{func.__name__} took {elapsed:.2f}s")
        return result
    return wrapper

@timing_decorator
def slow_function():
    time.sleep(1)
    return "done"
```

### Example 3: List Comprehension with Filtering
```python
# Filter and transform in one line
numbers = [1, 2, 3, 4, 5, 6]
even_squares = [x**2 for x in numbers if x % 2 == 0]
# Result: [4, 16, 36]

# Dict comprehension
users = [("alice", 25), ("bob", 30)]
user_dict = {name: age for name, age in users}
# Result: {"alice": 25, "bob": 30}
```

### Example 4: Generator for Memory Efficiency
```python
def read_large_file(file_path: str):
    with open(file_path, 'r') as f:
        for line in f:
            yield line.strip()

for line in read_large_file("huge.txt"):
    process(line)
```

**See [references/patterns.md](references/patterns.md) for more patterns.**

## Common Errors

| Error | Solution |
|-------|----------|
| `ModuleNotFoundError` | Run `uv add <package>` or `pip install <package>` |
| `TypeError` | Check types, add type hints |
| `KeyError` | Use `.get('key', default)` instead of `['key']` |
| `IndentationError` | Use 4 spaces (PEP 8) |
| `AttributeError: 'NoneType'` | Check for None: `if obj is not None:` |

## Related Skills

- **pytest-testing**: Fixtures, parametrize, mocking
- **pandas-data-analysis**: Advanced DataFrame operations
- **fastapi-backend**: REST APIs with FastAPI
- **django-web**: Web applications with Django
- **github-actions**: CI/CD for Python

## References

- [references/patterns.md](references/patterns.md) - Context managers, decorators, comprehensions, generators, dataclasses, async/await
- [references/type-hints.md](references/type-hints.md) - Advanced typing (Generics, Protocols, TypeVar, type guards)
- [references/pytest.md](references/pytest.md) - Fixtures, parametrize, mocking, assertions
- [references/pandas.md](references/pandas.md) - DataFrame operations, groupby, merge, performance
- [references/dependency-management.md](references/dependency-management.md) - uv, poetry, pip workflows and pyproject.toml
