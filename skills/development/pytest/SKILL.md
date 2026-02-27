---
name: pytest
description: Python testing with pytest framework. This skill should be used when user asks to write tests, create test files, run tests, fix failing tests, add test coverage, set up fixtures, mock dependencies, or do any testing-related work for Python code. Triggers on requests like "write tests for this", "add unit tests", "test this function", "fix the failing test", "run pytest", "set up test fixtures", or "mock this dependency".
allowed-tools: Bash(pytest:*), Bash(python:*), Read, Glob, Grep
---

# Pytest Testing Skill

Write and run Python tests using pytest with Test-Driven Development (TDD).

## Before Implementation

Gather context to ensure successful test implementation:

| Source | Gather |
|--------|--------|
| **Codebase** | Existing test structure, conftest.py patterns, fixtures in use |
| **Conversation** | What code to test, expected behavior, edge cases |
| **Skill References** | Testing patterns from `references/` directory |
| **User Guidelines** | Project testing conventions, coverage requirements |

## Clarifications

### Required (ask if not clear)
1. **Test type?** Unit tests / Integration tests / End-to-end tests
2. **Framework being tested?** FastAPI / Django / CLI / Pure Python
3. **Async code?** Yes (need pytest-asyncio) / No

### Optional (ask if relevant)
4. **Coverage target?** 80% / 90% / 100% / No requirement
5. **Mocking needed?** External APIs / Database / File system

## Official Documentation

| Resource | URL | Use For |
|----------|-----|---------|
| Pytest Docs | https://docs.pytest.org | Official reference |
| pytest-asyncio | https://pytest-asyncio.readthedocs.io | Async testing |
| pytest-cov | https://pytest-cov.readthedocs.io | Coverage reports |
| pytest-mock | https://pytest-mock.readthedocs.io | Mocking utilities |
| HTTPX Testing | https://www.python-httpx.org/async/ | FastAPI async testing |

> **Version Note**: This skill follows pytest 7.x+ patterns. For older versions, check migration guides.

## TDD Workflow (Red-Green-Refactor)

**ALWAYS follow TDD when writing code:**

### The Cycle
```
🔴 RED    → Write a failing test first
🟢 GREEN  → Write minimal code to pass the test
🔄 REFACTOR → Clean up code, keep tests green
```

### TDD Rules
1. **Never write code without a failing test first**
2. **Write only enough code to make the test pass**
3. **Refactor only when tests are green**

### Quick Example
```python
# Step 1: 🔴 RED - Write failing test
def test_add():
    assert add(2, 3) == 5  # NameError: 'add' is not defined

# Step 2: 🟢 GREEN - Minimal implementation
def add(a, b):
    return a + b  # Test passes!

# Step 3: 🔄 REFACTOR - Improve if needed (tests stay green)
```

### TDD for FastAPI Endpoints
```python
# Step 1: 🔴 RED - Test first (endpoint doesn't exist)
def test_get_user(client):
    response = client.get("/users/1")
    assert response.status_code == 200
    assert response.json()["id"] == 1

# Step 2: 🟢 GREEN - Create endpoint
@app.get("/users/{user_id}")
def get_user(user_id: int):
    return {"id": user_id, "name": "Test User"}

# Step 3: 🔄 REFACTOR - Add proper DB lookup, error handling
```

See `references/tdd.md` for complete TDD workflow and patterns.

## Quick Start

### Write a Basic Test
```python
def test_function_name():
    result = function_under_test(input)
    assert result == expected
```

### Run Tests
```bash
uv run pytest                      # Run all tests
uv run pytest tests/test_file.py   # Run specific file
uv run pytest -k "test_name"       # Run matching tests
uv run pytest -x                   # Stop on first failure
uv run pytest --lf                 # Run last failed only
uv run pytest -v                   # Verbose output
```

## Test File Structure

Place tests in `tests/` directory mirroring source structure:
```
project/
├── src/
│   ├── api.py
│   └── utils.py
└── tests/
    ├── conftest.py      # Shared fixtures
    ├── test_api.py
    └── test_utils.py
```

Test files must start with `test_` or end with `_test.py`.

## Writing Tests

### Test Functions
```python
def test_addition():
    assert add(2, 3) == 5

def test_raises_error():
    with pytest.raises(ValueError):
        validate("")
```

### Test Classes
```python
class TestCalculator:
    def test_add(self):
        calc = Calculator()
        assert calc.add(2, 3) == 5

    def test_divide_by_zero(self):
        calc = Calculator()
        with pytest.raises(ZeroDivisionError):
            calc.divide(1, 0)
```

### Fixtures
```python
@pytest.fixture
def client():
    return TestClient(app)

def test_endpoint(client):
    response = client.get("/api/users")
    assert response.status_code == 200
```

### Parametrized Tests
```python
@pytest.mark.parametrize("input,expected", [
    ("hello", 5),
    ("", 0),
    ("world", 5),
])
def test_length(input, expected):
    assert len(input) == expected
```

### Mocking
```python
from unittest.mock import patch, Mock

@patch("module.external_api")
def test_with_mock(mock_api):
    mock_api.return_value = {"status": "ok"}
    result = fetch_data()
    assert result["status"] == "ok"
    mock_api.assert_called_once()
```

### FastAPI Async Testing (httpx)
```python
import pytest
from httpx import ASGITransport, AsyncClient
from app.main import app

@pytest.mark.anyio
async def test_async_endpoint():
    async with AsyncClient(
        transport=ASGITransport(app=app),
        base_url="http://test"
    ) as ac:
        response = await ac.get("/")
    assert response.status_code == 200
```

See `references/fastapi_testing.md` for dependency overrides, auth testing, WebSockets.

## Common Markers

```python
@pytest.mark.skip(reason="Not implemented")
@pytest.mark.skipif(sys.version < "3.10", reason="Requires 3.10+")
@pytest.mark.xfail(reason="Known bug")
@pytest.mark.asyncio  # For async tests (requires pytest-asyncio)
@pytest.mark.slow     # Custom marker for slow tests
```

## Coverage

```bash
uv run pytest --cov=src --cov-report=term-missing
uv run pytest --cov=src --cov-report=html
```

## Resources

### Scripts
- `scripts/run_tests.py` - Run pytest with useful defaults
- `scripts/generate_tests.py` - Generate test file stubs from source
- `scripts/check_coverage.py` - Check coverage thresholds and report files below target

### References
- `references/tdd.md` - **TDD workflow, Red-Green-Refactor cycle, TDD patterns for APIs**
- `references/patterns.md` - Fixture patterns, mocking, async testing, database testing
- `references/conftest_templates.md` - Ready-to-use conftest.py templates for web apps, FastAPI, Django, CLI testing
- `references/fastapi_testing.md` - FastAPI-specific testing: dependency overrides, async testing with httpx, auth, WebSockets, background tasks
- `references/plugins.md` - Popular pytest plugins with usage examples
- `references/troubleshooting.md` - Common errors and how to fix them
- `references/ci_cd.md` - GitHub Actions and GitLab CI integration

## Popular Plugins

| Plugin | Purpose | Install |
|--------|---------|---------|
| pytest-cov | Code coverage | `uv add pytest-cov` |
| pytest-asyncio | Async test support | `uv add pytest-asyncio` |
| pytest-anyio | Alternative async support | `uv add pytest-anyio anyio` |
| pytest-mock | Easier mocking | `uv add pytest-mock` |
| pytest-xdist | Parallel testing | `uv add pytest-xdist` |
| pytest-httpx | Mock httpx requests | `uv add pytest-httpx` |
| pytest-env | Environment variables | `uv add pytest-env` |
| pytest-timeout | Test timeouts | `uv add pytest-timeout` |

See `references/plugins.md` for detailed usage.

## CI/CD Quick Start

### GitHub Actions
```yaml
- name: Install uv
  uses: astral-sh/setup-uv@v4
- name: Run tests
  run: uv run pytest --cov=src --cov-report=xml
```

### GitLab CI
```yaml
test:
  script:
    - pip install uv
    - uv run pytest --cov=src
```

See `references/ci_cd.md` for complete examples.

## Troubleshooting

| Error | Solution |
|-------|----------|
| `ModuleNotFoundError` | Add `__init__.py` or fix PYTHONPATH |
| `fixture not found` | Check conftest.py location |
| `async test not running` | Add `@pytest.mark.asyncio` |
| `collected 0 items` | Check test file/function naming |

See `references/troubleshooting.md` for detailed solutions.

## Common Mistakes

| Mistake | Why It's Wrong | Fix |
|---------|----------------|-----|
| Testing implementation, not behavior | Brittle tests that break on refactor | Test inputs → outputs |
| Missing `@pytest.mark.asyncio` | Async test silently passes | Add marker for async tests |
| Hardcoded test data | Tests fail in different environments | Use fixtures and factories |
| Not using `conftest.py` | Duplicate fixtures across files | Centralize shared fixtures |
| Ignoring test isolation | Tests affect each other | Use fresh fixtures per test |
| Mocking too much | Tests don't catch real bugs | Mock only external dependencies |

## Before Delivery Checklist

### Test Quality
- [ ] Tests follow TDD (written before implementation)
- [ ] Each test tests one thing (single assertion focus)
- [ ] Tests are independent (can run in any order)
- [ ] Edge cases covered (empty, null, boundaries)

### Coverage
- [ ] Coverage meets project requirements
- [ ] Critical paths have 100% coverage
- [ ] Run: `uv run pytest --cov --cov-report=term-missing`

### Organization
- [ ] Tests mirror source structure
- [ ] Shared fixtures in `conftest.py`
- [ ] Descriptive test names (`test_<action>_<scenario>_<expected>`)

### CI Ready
- [ ] All tests pass: `uv run pytest`
- [ ] No hardcoded paths or credentials
- [ ] Async tests properly marked
