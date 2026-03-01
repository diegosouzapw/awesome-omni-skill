---
name: python-testing
description: Use when implementing new Python code (follow TDD), designing test suites, reviewing test coverage, setting up pytest infrastructure, writing fixtures, mocking dependencies, or performing parametrized testing
---

# Python Testing Patterns

Comprehensive testing strategies for Python applications using pytest, TDD methodology, and best practices.

## When to Use

- Writing new Python code (follow TDD: red, green, refactor)
- Designing test suites for Python projects
- Reviewing Python test coverage
- Setting up testing infrastructure
- Need guidance on fixtures, mocking, or parametrization

## TDD Cycle

Always follow the Red → Green → Refactor cycle:

1. **RED**: Write a failing test for the desired behavior
2. **GREEN**: Write minimal code to make the test pass
3. **REFACTOR**: Improve code while keeping tests green

## Quick Start

```python
import pytest

# Basic test
def test_addition():
    assert 2 + 2 == 4

# Test with fixture
@pytest.fixture
def sample_data():
    return {"name": "Alice", "age": 30}

def test_sample_data(sample_data):
    assert sample_data["name"] == "Alice"

# Parametrized test
@pytest.mark.parametrize("input,expected", [
    ("hello", "HELLO"),
    ("world", "WORLD"),
])
def test_uppercase(input, expected):
    assert input.upper() == expected

# Exception testing
def test_divide_by_zero():
    with pytest.raises(ZeroDivisionError):
        _ = 1 / 0
```

## Core Topics

### Fixtures
Setup and teardown for tests, sharing data, and scopes (function, module, session).
**Details:** `references/fixtures.md`

### Parametrization
Running tests with multiple inputs using `@pytest.mark.parametrize`.
**Details:** `references/parametrization.md`

### Mocking and Patching
Using `unittest.mock` to mock external dependencies and API calls.
**Details:** `references/mocking.md`

### Markers and Test Selection
Custom markers (`@pytest.mark.slow`) and selective test execution.
**Details:** `references/markers.md`

### Async Testing
Testing async functions with `pytest-asyncio`.
**Details:** `references/async_testing.md`

### Test Organization
Directory structure, conftest.py, and grouping related tests.
**Details:** `references/test_organization.md`

## Quick Reference

| Pattern | Usage |
|---------|-------|
| `pytest.raises()` | Test expected exceptions |
| `@pytest.fixture()` | Create reusable test fixtures |
| `@pytest.mark.parametrize()` | Run tests with multiple inputs |
| `@pytest.mark.slow` | Mark slow tests |
| `pytest -m "not slow"` | Skip slow tests |
| `@patch()` | Mock functions and classes |
| `tmp_path` fixture | Automatic temp directory |
| `pytest --cov` | Generate coverage report |

## Running Tests

```bash
# Run all tests
pytest

# Run with coverage
pytest --cov=mypackage --cov-report=html

# Run only fast tests
pytest -m "not slow"

# Run with verbose output
pytest -v

# Run until first failure
pytest -x

# Run last failed tests
pytest --lf
```

## Best Practices

**DO:**
- Follow TDD (write tests before code)
- Test one thing per test
- Use descriptive test names
- Use fixtures to eliminate duplication
- Mock external dependencies
- Aim for 80%+ coverage
- Keep tests fast

**DON'T:**
- Test implementation details
- Use complex conditionals in tests
- Share state between tests
- Catch exceptions in tests (use `pytest.raises`)
- Write tests that are too brittle

**Details:** `references/best_practices.md`

## Common Patterns

### API Endpoint Testing
```python
@pytest.fixture
def client():
    app = create_app(testing=True)
    return app.test_client()

def test_get_user(client):
    response = client.get("/api/users/1")
    assert response.status_code == 200
```

### Database Testing
```python
@pytest.fixture
def db_session():
    session = Session(bind=engine)
    session.begin_nested()
    yield session
    session.rollback()
    session.close()
```

**More patterns:** `references/common_patterns.md`
