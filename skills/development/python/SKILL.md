---
name: "python"
description: 'Write production-ready Python code following modern best practices. Use when building Python applications, adding type hints, writing async code, implementing error handling, testing with pytest, or structuring Python project layouts.'
metadata:
  author: "AgentX"
  version: "1.0.0"
  created: "2025-01-15"
  updated: "2025-01-15"
compatibility:
  languages: ["python"]
  frameworks: ["flask", "django", "fastapi"]
  platforms: ["windows", "linux", "macos"]
---

# Python Development

> **Purpose**: Production-ready Python development standards for building secure, performant, maintainable applications.  
> **Audience**: Engineers building Python applications, APIs, data pipelines, or AI/ML systems.  
> **Standard**: Follows [github/awesome-copilot](https://github.com/github/awesome-copilot) Python development patterns.

---

## When to Use This Skill

- Building Python applications or APIs
- Adding type hints to Python code
- Writing async/await patterns in Python
- Testing with pytest
- Structuring Python project layouts

## Prerequisites

- Python 3.11+ installed
- pip or poetry package manager
- pytest for testing

## Quick Reference

| Need | Solution | Pattern |
|------|----------|---------|
| **Type hints** | Use everywhere | `def get_user(id: int) -> Optional[User]:` |
| **Async code** | Use `async`/`await` | `async def fetch_data() -> str:` |
| **Error handling** | Specific exceptions | `try-except ValueError` |
| **Testing** | pytest | `def test_user_creation():` |
| **Logging** | Standard library | `logger.info("User %s created", user_id)` |
| **Docstrings** | Google style | `"""Gets user by ID.\n\nArgs:\n    id: User identifier` |

---

## Python Version

**Current**: Python 3.11+  
**Minimum**: Python 3.9+

### Modern Python Features (Use These)

```python
# Type hints (PEP 484) - Use everywhere
from typing import Optional, List, Dict, Any
from dataclasses import dataclass

def get_user(user_id: int) -> Optional[dict[str, Any]]:
    """Get user by ID."""
    return users.get(user_id)

# Dataclasses for data structures
@dataclass
class User:
    id: int
    name: str
    email: str
    is_active: bool = True

# f-strings for formatting
name = "Alice"
age = 30
message = f"User {name} is {age} years old"

# Walrus operator (:=) in Python 3.8+
if (user := get_user(123)) is not None:
    print(f"Found user: {user['name']}")

# Pattern matching (Python 3.10+)
def process_response(status: int) -> str:
    match status:
        case 200:
            return "Success"
        case 404:
            return "Not found"
        case 500:
            return "Server error"
        case _:
            return "Unknown status"
```

---

## Type Hints

**Always use type hints** for function parameters, return values, and class attributes.

```python
from typing import Optional, List, Dict, Any, Union, TypeVar, Generic

# Basic types
def calculate_total(price: float, quantity: int) -> float:
    return price * quantity

# Optional types
def find_user(user_id: int) -> Optional[User]:
    """Returns None if user not found."""
    return db.query(User).filter_by(id=user_id).first()

# Collections
def get_active_users() -> List[User]:
    return [u for u in users if u.is_active]

def get_user_map() -> Dict[int, User]:
    return {u.id: u for u in users}

# Union types
def process_data(data: Union[str, bytes]) -> str:
    if isinstance(data, bytes):
        return data.decode('utf-8')
    return data

# Generic types
T = TypeVar('T')

def first_or_none(items: List[T]) -> Optional[T]:
    """Get first item or None if list is empty."""
    return items[0] if items else None

# Type aliases for complex types
UserId = int
UserData = Dict[str, Any]

def create_user(user_id: UserId, data: UserData) -> User:
    return User(id=user_id, **data)
```

---

## Best Practices Summary

### Code Style (PEP 8)

```python
# ✅ GOOD: Follow PEP 8
def calculate_total(items: List[Item]) -> float:
    """Calculate total price of items."""
    return sum(item.price * item.quantity for item in items)

# Variable naming
user_count = 10  # snake_case for variables
MAX_RETRIES = 3  # UPPER_CASE for constants
UserService    # PascalCase for classes

# ✅ GOOD: List comprehensions
active_users = [u for u in users if u.is_active]

# ❌ BAD: Mutable default arguments
def add_item(item, items=[]):  # Don't do this!
    items.append(item)
    return items

# ✅ GOOD: Use None as default
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items
```

### Performance

```python
# ✅ GOOD: Use generators for large datasets
def process_large_file(filename: str):
    """Process large file line by line."""
    with open(filename) as f:
        for line in f:  # Generator - memory efficient
            yield process_line(line)

# ✅ GOOD: Use collections.defaultdict
from collections import defaultdict

user_groups = defaultdict(list)
for user in users:
    user_groups[user.group].append(user)

# ✅ GOOD: Use set for membership testing
valid_ids = {1, 2, 3, 4, 5}
if user_id in valid_ids:  # O(1) lookup
    process_user(user_id)
```

---

## Common Pitfalls

| Issue | Problem | Solution |
|-------|---------|----------|
| **Mutable defaults** | `def func(items=[]):` | Use `items=None` then `if items is None: items = []` |
| **Missing type hints** | No type information | Add types everywhere |
| **Broad exceptions** | `except Exception:` | Catch specific exceptions |
| **No docstrings** | Undocumented code | Add Google-style docstrings |
| **String concatenation** | `s = s + "text"` in loop | Use `"".join(list)` or f-strings |
| **Not using context managers** | Manual file.close() | Use `with open(...) as f:` |

---

## Project Structure

```
my_project/
├── src/
│   ├── my_project/
│   │   ├── __init__.py
│   │   ├── models/
│   │   │   ├── __init__.py
│   │   │   └── user.py
│   │   ├── services/
│   │   │   ├── __init__.py
│   │   │   └── user_service.py
│   │   ├── repositories/
│   │   │   ├── __init__.py
│   │   │   └── user_repository.py
│   │   └── utils/
│   │       ├── __init__.py
│   │       └── helpers.py
├── tests/
│   ├── __init__.py
│   ├── test_models.py
│   ├── test_services.py
│   └── test_repositories.py
├── requirements.txt
├── pyproject.toml
├── README.md
└── .gitignore
```

---

## Resources

- **Official Docs**: [docs.python.org](https://docs.python.org)
- **PEP 8**: [pep8.org](https://pep8.org)
- **Type Hints**: [PEP 484](https://peps.python.org/pep-0484/)
- **pytest**: [pytest.org](https://pytest.org)
- **Async**: [docs.python.org/asyncio](https://docs.python.org/3/library/asyncio.html)
- **Awesome Copilot**: [github.com/github/awesome-copilot](https://github.com/github/awesome-copilot)

---

**See Also**: [Skills.md](../../../../Skills.md) • [AGENTS.md](../../../../AGENTS.md)

**Last Updated**: January 27, 2026


## Scripts

| Script | Purpose | Usage |
|--------|---------|-------|
| [`scaffold-project.py`](scripts/scaffold-project.py) | Generate Python project with pyproject.toml, ruff, mypy, pre-commit | `python scripts/scaffold-project.py --name myapp [--fastapi]` |

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Type hint errors with mypy | Install type stubs, use type: ignore sparingly |
| Async event loop already running | Use asyncio.run() at top level only, use await inside async functions |
| pytest not finding tests | Name test files test_*.py and functions test_*, check pytest.ini paths |

## References

- [Async Errors Context](references/async-errors-context.md)
- [Docs Testing Logging](references/docs-testing-logging.md)
- [Dataclasses Patterns](references/dataclasses-patterns.md)