---
name: python-engineer
description: Expert Python engineering for code review, quality improvement, and debugging. Use when reviewing Python code for best practices, refactoring for cleaner code, debugging errors, improving type safety with type hints, writing tests with pytest, or developing Web APIs with FastAPI. Focuses on Pythonic patterns, SOLID principles, and production-ready code quality.
---

# Python Engineer

Expert guidance for Python code review, quality improvement, and debugging with focus on Web API development.

## Core Workflow

1. **Understand** - Analyze current code structure and identify issues
2. **Diagnose** - Apply relevant best practices from references
3. **Improve** - Implement changes with proper typing and tests
4. **Validate** - Verify improvements through testing

## Code Review

When reviewing Python code, check:

1. **Type Safety** - All functions have type hints, mypy passes
2. **Testing** - pytest coverage for critical paths
3. **Clean Code** - Pythonic patterns, no code smells
4. **Error Handling** - Proper exception handling
5. **Documentation** - Docstrings for public APIs

For detailed checklist: [references/code-review.md](references/code-review.md)

## Debugging

Debug workflow:
1. Reproduce the issue with minimal test case
2. Identify error type and stack trace
3. Apply targeted debugging technique
4. Verify fix with test

For debugging techniques: [references/debugging.md](references/debugging.md)

## Quality Improvement

### Type Hints

Add comprehensive type annotations:
```python
from typing import TypeVar, Generic
from collections.abc import Callable, Sequence

T = TypeVar("T")

def process_items(
    items: Sequence[T],
    transformer: Callable[[T], T],
) -> list[T]:
    return [transformer(item) for item in items]
```

For complete guide: [references/type-hints.md](references/type-hints.md)

### Testing with pytest

Write meaningful tests:
```python
import pytest

class TestUserService:
    def test_create_user_with_valid_email_returns_user(
        self, user_service: UserService
    ) -> None:
        result = user_service.create("test@example.com")
        assert result.email == "test@example.com"
        assert result.id is not None

    def test_create_user_with_invalid_email_raises_validation_error(
        self, user_service: UserService
    ) -> None:
        with pytest.raises(ValidationError, match="Invalid email"):
            user_service.create("invalid-email")
```

For testing patterns: [references/testing.md](references/testing.md)

### Clean Code

Prefer:
- Composition over inheritance
- Small, focused functions (< 20 lines)
- Descriptive names over comments
- Early returns to reduce nesting
- Dataclasses/Pydantic over raw dicts

For patterns: [references/clean-code.md](references/clean-code.md)

## Web API Development

FastAPI best practices:
```python
from fastapi import FastAPI, HTTPException, status
from pydantic import BaseModel

class UserCreate(BaseModel):
    email: str
    name: str

@app.post("/users", status_code=status.HTTP_201_CREATED)
async def create_user(user: UserCreate) -> User:
    if await user_exists(user.email):
        raise HTTPException(
            status_code=status.HTTP_409_CONFLICT,
            detail="User already exists"
        )
    return await create_user_in_db(user)
```

For FastAPI patterns: [references/fastapi.md](references/fastapi.md)

## References

- [references/code-review.md](references/code-review.md) - Code review checklist
- [references/clean-code.md](references/clean-code.md) - Pythonic patterns
- [references/testing.md](references/testing.md) - pytest best practices
- [references/type-hints.md](references/type-hints.md) - Type annotation guide
- [references/fastapi.md](references/fastapi.md) - FastAPI development
- [references/debugging.md](references/debugging.md) - Debugging techniques
