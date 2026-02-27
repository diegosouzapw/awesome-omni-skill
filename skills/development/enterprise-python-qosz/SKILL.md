---
name: enterprise-python
description: Enterprise-ready Python development incorporating Kaizen (continuous improvement) and Monozukuri (meticulous craftsmanship) principles. Use this skill when building Python applications, APIs, CLI tools, data pipelines, automation scripts, or when the user requests clean, efficient, fast, simple, elegant, enterprise-grade, bulletproof, or production-ready Python code. This skill enforces modern Python 3.12+ best practices, type safety, testing patterns, security, and performance optimization.
---

# Enterprise Python Development

Build bulletproof, enterprise-ready Python applications that embody Kaizen (continuous improvement) and Monozukuri (meticulous craftsmanship) principles. This skill guides development of clean, efficient, performant Python code that is simple, elegant, and built to last.

## Core Philosophy

**Kaizen (改善)**: Continuous improvement through incremental refinement
**Monozukuri (ものづくり)**: The art of making things with meticulous attention to quality and craftsmanship

These principles translate to Python code that is:
- **Clean**: Self-documenting, readable, and maintainable
- **Efficient**: Optimized algorithms and minimal resource overhead
- **Fast**: Performance-first architecture with profiled bottlenecks
- **Simple**: Complexity only where justified
- **Elegant**: Beautiful solutions that feel inevitable

## Python Development Workflow

### 1. Understand Requirements Deeply

Before writing code:
- Clarify functionality goals and expected inputs/outputs
- Identify edge cases, error scenarios, and failure modes
- Understand performance requirements and constraints
- Consider data volumes and scalability needs
- Plan for testing, logging, and observability

### 2. Design Before Implementation

Plan the architecture:
- Choose appropriate patterns (functional, OOP, or hybrid)
- Design module hierarchy and data flow
- Plan for dependency injection and testability
- Consider async requirements (asyncio vs threading vs multiprocessing)
- Plan type safety with strict typing

### 3. Write Code with Craftsmanship

**Simplicity First**
- Prefer standard library over external dependencies
- Use composition over inheritance
- Avoid premature abstraction
- Keep functions focused and composable
- YAGNI (You Aren't Gonna Need It)

**Clean Code Standards**
- Meaningful, descriptive names (functions, classes, variables)
- Functions should do one thing well
- Keep functions under 50 lines, classes under 300 lines
- Extract complex logic into helper functions
- Use early returns to reduce nesting
- Comments explain WHY, not WHAT

**Type Hints Best Practices**
- Use strict typing (`# type: ignore` only when truly necessary)
- Leverage `typing` module: `Optional`, `Union`, `TypeVar`, `Generic`
- Use `Protocol` for structural subtyping
- Define `TypedDict` for dictionary shapes
- Use Pydantic or dataclasses for data validation

**Error Handling**
- Use specific exception types, not bare `except:`
- Create custom exceptions for domain errors
- Provide actionable error messages with context
- Log errors with proper severity and context
- Handle edge cases gracefully
- Fail fast with clear diagnostics

**Security Mindset**
- Sanitize user inputs (prevent injection attacks)
- Use parameterized queries for databases
- Validate all external data with Pydantic
- Use environment variables for secrets (never hardcode)
- Implement proper authentication/authorization
- Audit dependencies for vulnerabilities

### 4. Optimize for Performance

**Algorithm Optimization**
- Choose appropriate data structures (dict > list for lookups)
- Use generators for memory efficiency
- Profile before optimizing (`cProfile`, `line_profiler`)
- Use `functools.lru_cache` for expensive computations
- Consider `__slots__` for memory-heavy classes

**Async and Concurrency**
- Use `asyncio` for I/O-bound operations
- Use `multiprocessing` for CPU-bound operations
- Use `concurrent.futures` for simple parallelism
- Avoid blocking calls in async code
- Use connection pooling for databases/HTTP

**Import Optimization**
- Lazy import heavy modules when appropriate
- Avoid circular imports through proper structure
- Use `__all__` to control public API

### 5. Ensure Robustness

**Comprehensive Testing**
- Unit tests for functions/methods (pytest)
- Integration tests for module interactions
- Property-based tests for edge cases (hypothesis)
- Fixtures for test data management
- Mock external dependencies
- Aim for >80% coverage on critical paths

**Logging and Observability**
- Use `logging` module with proper levels
- Structured logging (JSON) for production
- Add correlation IDs for request tracing
- Instrument with metrics (timing, counts)
- Set up alerts for errors and anomalies

**Documentation**
- Docstrings for public functions/classes (Google style)
- Type hints serve as inline documentation
- README with setup, usage, and examples
- Architecture Decision Records for significant choices

### 6. Refine Through Kaizen

Continuously improve:
- Review and refactor regularly
- Eliminate duplicate code (DRY)
- Improve function/class design
- Update dependencies regularly
- Address technical debt incrementally
- Monitor performance metrics

## Code Quality Standards

### Project Structure

```
project-name/
├── src/
│   └── project_name/
│       ├── __init__.py
│       ├── main.py              # Entry point
│       ├── config.py            # Configuration
│       ├── models/              # Data models
│       │   ├── __init__.py
│       │   └── user.py
│       ├── services/            # Business logic
│       │   ├── __init__.py
│       │   └── user_service.py
│       ├── repositories/        # Data access
│       │   ├── __init__.py
│       │   └── user_repo.py
│       ├── api/                 # API layer
│       │   ├── __init__.py
│       │   └── routes.py
│       └── utils/               # Helpers
│           ├── __init__.py
│           └── validators.py
├── tests/
│   ├── __init__.py
│   ├── conftest.py             # Pytest fixtures
│   ├── unit/
│   └── integration/
├── pyproject.toml              # Project config (PEP 518)
├── README.md
└── .env.example
```

### Naming Conventions

**Modules/Packages**: snake_case (`user_service.py`, `data_processing`)
**Classes**: PascalCase (`UserService`, `DataProcessor`, `HTTPClient`)
**Functions/Methods**: snake_case (`fetch_user`, `calculate_total`, `process_data`)
**Constants**: UPPER_SNAKE_CASE (`API_URL`, `MAX_RETRIES`, `DEFAULT_TIMEOUT`)
**Private**: Leading underscore (`_internal_method`, `_helper_function`)
**Type Variables**: Single uppercase or PascalCase (`T`, `KeyType`, `ValueType`)

### Code Patterns

```python
# GOOD: Type hints and dataclasses
from dataclasses import dataclass
from typing import Optional

@dataclass(frozen=True, slots=True)
class User:
    id: str
    email: str
    name: str
    is_active: bool = True

# GOOD: Pydantic for validation
from pydantic import BaseModel, EmailStr, Field

class CreateUserRequest(BaseModel):
    email: EmailStr
    name: str = Field(min_length=1, max_length=100)
    age: int = Field(ge=0, le=150)

# GOOD: Context managers for resources
from contextlib import contextmanager
from typing import Generator

@contextmanager
def database_transaction(db: Database) -> Generator[Session, None, None]:
    session = db.create_session()
    try:
        yield session
        session.commit()
    except Exception:
        session.rollback()
        raise
    finally:
        session.close()

# GOOD: Async with proper error handling
async def fetch_user(user_id: str) -> User | None:
    try:
        async with httpx.AsyncClient() as client:
            response = await client.get(f"{API_URL}/users/{user_id}")
            response.raise_for_status()
            return User(**response.json())
    except httpx.HTTPStatusError as e:
        if e.response.status_code == 404:
            return None
        raise UserFetchError(f"Failed to fetch user: {e}") from e

# GOOD: Dependency injection
class UserService:
    def __init__(self, repo: UserRepository, cache: Cache) -> None:
        self._repo = repo
        self._cache = cache

    async def get_user(self, user_id: str) -> User | None:
        if cached := await self._cache.get(f"user:{user_id}"):
            return User(**cached)
        user = await self._repo.find_by_id(user_id)
        if user:
            await self._cache.set(f"user:{user_id}", user.model_dump())
        return user
```

## Advanced Patterns

For complex scenarios, consult:
- **references/async-patterns.md**: Asyncio patterns and concurrency
- **references/testing-patterns.md**: Testing strategies and fixtures
- **references/api-patterns.md**: FastAPI/Flask best practices
- **references/data-patterns.md**: Data processing and pipelines

## Modern Python Best Practices

### Type Hints (Python 3.12+)
```python
# Use built-in generics (no typing import needed)
def process_items(items: list[str]) -> dict[str, int]:
    return {item: len(item) for item in items}

# Use | for unions
def fetch_data(id: str) -> User | None:
    ...

# Use TypedDict for structured dicts
from typing import TypedDict

class Config(TypedDict):
    host: str
    port: int
    debug: bool
```

### Structural Pattern Matching
```python
def handle_response(response: dict) -> str:
    match response:
        case {"status": "success", "data": data}:
            return f"Success: {data}"
        case {"status": "error", "message": msg}:
            return f"Error: {msg}"
        case {"status": status}:
            return f"Unknown status: {status}"
        case _:
            return "Invalid response"
```

### Modern Configuration
```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    database_url: str
    api_key: str
    debug: bool = False
    max_connections: int = 10

    class Config:
        env_file = ".env"

settings = Settings()
```

## Review Checklist

Before finalizing code:

- [ ] Type hints on all public functions/methods
- [ ] No `# type: ignore` without justification
- [ ] Custom exceptions for domain errors
- [ ] Proper logging (not print statements)
- [ ] Input validation on all external data
- [ ] Unit tests for critical functions
- [ ] Docstrings on public API
- [ ] No hardcoded secrets or credentials
- [ ] Resources properly closed (context managers)
- [ ] Async code doesn't block event loop
- [ ] Dependencies pinned in pyproject.toml
- [ ] No bare `except:` clauses
- [ ] Error messages are actionable
- [ ] Performance-critical code profiled

## When to Use This Skill

Apply this skill whenever:
- Building Python applications or libraries
- Creating APIs (FastAPI, Flask, Django)
- Writing CLI tools (Click, Typer)
- Building data pipelines
- Writing automation scripts
- Implementing microservices
- User requests enterprise-grade Python code
- User mentions Python, scripting, or backend development

This skill transforms good Python code into exceptional Python applications—fast, secure, maintainable, and delightful to work with.
