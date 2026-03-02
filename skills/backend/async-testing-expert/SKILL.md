---
name: Async Testing Expert
description: Comprehensive pytest skill for async Python testing with proper mocking, fixtures, and patterns from production-ready test suites. Use when writing or improving async tests for Python applications, especially FastAPI backends with database interactions.
---

# Async Testing Expert

Expert guidance for writing comprehensive async Python tests using pytest, based on production patterns from a 387-test FastAPI backend test suite.

## When to Use This Skill

Activate this skill when:
- Writing async tests for FastAPI applications
- Testing async database operations (PostgreSQL, MySQL, etc.)
- Setting up pytest fixtures for async applications
- Creating mock objects for database connections
- Testing services with dependency injection
- Writing DAO (Data Access Object) layer tests
- Testing async API endpoints

## Core Principles

### 1. Test Organization
```
tests/
├── conftest.py           # Shared fixtures (app, client, event_loop, faker)
├── fakes.py              # Reusable mock objects (FakeConnection, FakeRecord)
├── test_<module>_dao.py  # DAO layer tests
├── test_<module>_service.py  # Service layer tests
├── test_<module>_router.py   # API endpoint tests
└── test_<module>_dto.py      # DTO validation tests
```

### 2. Naming Conventions
- Test files: `test_<module>_<layer>.py`
- Test functions: `test_<what>_<scenario>` (e.g., `test_create_calls_execute`, `test_fetch_by_id_error_maps_to_500`)
- Be descriptive: readers should understand what's being tested without reading the code

### 3. Always Use Type Hints
```python
async def test_fetch_user_success(faker: Faker) -> None:
    user_id: int = faker.random_int(1, 100)
    conn: FakeConnection = FakeConnection()
    # ...
```

## Essential Fixtures (conftest.py)

### FastAPI Application Fixtures
```python
import asyncio
import pytest
from fastapi.testclient import TestClient
from httpx import AsyncClient, ASGITransport
from faker import Faker

@pytest.fixture(scope='session')
def app():
    """Create a FastAPI app instance for testing."""
    from src.config.factory import create_app
    return create_app()

@pytest.fixture(scope='session')
def client(app):
    """Provides a synchronous TestClient for FastAPI."""
    with TestClient(app) as c:
        yield c

@pytest.fixture
async def async_client(app):
    """Provides an asynchronous AsyncClient for FastAPI using ASGI transport."""
    transport = ASGITransport(app=app)
    async with AsyncClient(transport=transport, base_url='http://test') as ac:
        yield ac

@pytest.fixture
def event_loop():
    """Create a new event loop for each test."""
    loop = asyncio.new_event_loop()
    yield loop
    loop.close()

@pytest.fixture
def faker():
    """Provide a Faker instance configured for Brazilian Portuguese."""
    return Faker('pt_BR')  # Adjust locale as needed
```

## Mock Objects for Database Testing (fakes.py)

### FakeRecord - Simulate Query Results
```python
class FakeRecord:
    """Simulate a database record with a .result() method and optional rowcount."""
    def __init__(self, data, rowcount=None):
        self._data = data
        self.rowcount = rowcount if rowcount is not None else (
            data if isinstance(data, int) else 1
        )

    def result(self):
        return self._data
```

### FakeConnection - Full Database Mock
```python
class FakeConnection:
    """Simulate a psqlpy/asyncpg Connection with execute, fetch, fetch_val, and fetch_row."""
    def __init__(self):
        self.execute_return = None
        self.fetch_return = None
        self.fetch_row_return = None
        self.fetch_val_return = None
        self.execute_calls = []
        self.fetch_calls = []
        self.fetch_val_calls = []

    def transaction(self):
        return FakeTransactionContext(self)

    async def __aenter__(self):
        return self

    async def __aexit__(self, exc_type, exc, tb):
        return False

    async def execute(self, stmt, parameters=None):
        self.execute_calls.append((stmt, parameters))
        if isinstance(self.execute_return, Exception):
            raise self.execute_return

        # Support list of return values for multiple execute calls
        if (isinstance(self.execute_return, list) and
            len(self.execute_return) > 0 and
            all(isinstance(item, list) for item in self.execute_return)):
            return FakeRecord(self.execute_return.pop(0))

        return FakeRecord(self.execute_return)

    async def execute_many(self, stmt, parameters_list=None):
        """Simulate execute_many for bulk operations."""
        if parameters_list is None:
            parameters_list = []

        self.execute_calls.append((stmt, parameters_list))

        if isinstance(self.execute_return, Exception):
            raise self.execute_return

        total_rows = len(parameters_list) if parameters_list else 0
        return FakeRecord(data=total_rows, rowcount=total_rows)

    async def fetch(self, stmt, parameters=None):
        self.fetch_calls.append((stmt, parameters))
        return FakeRecord(self.fetch_return)

    async def fetch_val(self, stmt, parameters=None):
        self.fetch_val_calls.append((stmt, parameters))
        if isinstance(self.fetch_val_return, Exception):
            raise self.fetch_val_return
        return self.fetch_val_return

    async def fetch_row(self, stmt, parameters=None):
        """Simulate fetching a single row."""
        self.fetch_calls.append((stmt, parameters))

        if isinstance(self.fetch_row_return, Exception):
            raise self.fetch_row_return

        if self.fetch_row_return is not None:
            return self.fetch_row_return

        if isinstance(self.fetch_return, list) and len(self.fetch_return) > 0:
            return FakeRecord(self.fetch_return.pop(0))

        return FakeRecord(self.fetch_return)
```

### FakeTransaction - Transaction Context Mock
```python
class FakeTransaction:
    """Simulate a database transaction context."""
    def __init__(self, connection):
        self.connection = connection

    async def execute(self, stmt, parameters=None):
        return await self.connection.execute(stmt, parameters)

    async def execute_many(self, stmt, parameters_list=None, parameters=None):
        """Simulate execute_many - delegate to connection's execute_many if available."""
        params = parameters if parameters is not None else parameters_list
        if hasattr(self.connection, 'execute_many'):
            return await self.connection.execute_many(stmt, params)

        # Fallback: simulate by calling execute for each parameter set
        if params is None:
            params = []

        results = []
        for param_set in params:
            result = await self.connection.execute(stmt, param_set)
            results.append(result)

        if results:
            total_rowcount = sum(getattr(r, 'rowcount', 0) for r in results)
            return FakeRecord(data=total_rowcount, rowcount=total_rowcount)
        else:
            return FakeRecord(data=0, rowcount=0)

    async def fetch(self, stmt, parameters=None):
        return await self.connection.fetch(stmt, parameters)

    async def fetch_row(self, stmt, parameters=None):
        return await self.connection.fetch_row(stmt, parameters)

    async def fetch_val(self, stmt, parameters=None):
        return await self.connection.fetch_val(stmt, parameters)

class FakeTransactionContext:
    """Simulate the transaction context manager returned by conn.transaction()."""
    def __init__(self, connection):
        self.connection = connection
        self.transaction = FakeTransaction(connection)

    async def __aenter__(self):
        return self.transaction

    async def __aexit__(self, exc_type, exc, tb):
        return False
```

## Testing Patterns

### Pattern 1: DAO Layer Tests (Direct Method Testing)

**Use `__wrapped__` to bypass connection decorators:**

```python
@pytest.mark.asyncio
async def test_create_calls_execute(faker):
    """Test that create method calls execute with correct SQL and parameters."""
    # Arrange: Prepare test data
    create_dto = UserDTO.Create(
        name=faker.name(),
        email=faker.email(),
        cpf=faker.ssn()
    )
    conn = FakeConnection()

    # Act: Call DAO method directly with __wrapped__
    await UserDAO.create.__wrapped__(conn, create_dto)

    # Assert: Verify execute was called with correct SQL
    assert len(conn.execute_calls) == 1
    stmt, params = conn.execute_calls[0]
    assert 'INSERT INTO users' in stmt
    assert isinstance(params, list)
    assert len(params) == len(create_dto.model_dump())
```

### Pattern 2: Testing Exception Handling

```python
@pytest.mark.asyncio
async def test_fetch_by_id_error_maps_to_500():
    """Test that database errors are properly mapped to DAOException."""
    conn = FakeConnection()

    async def broken_fetch_row(stmt, parameters=None):
        raise RustPSQLDriverPyBaseError('db fail')

    conn.fetch_row = broken_fetch_row

    with pytest.raises(DAOException) as exc:
        await UserDAO.fetch_by_id.__wrapped__(conn, 1)

    err = exc.value
    assert err.status_code == 500
    assert 'Erro ao buscar' in err.detail
```

### Pattern 3: Service Layer Tests with Dependency Injection

**Create dummy dependencies for isolated testing:**

```python
class DummyUserAdapter:
    """Mock adapter for testing service layer."""
    def __init__(self, users):
        self.users = users
        self.called = False

    async def get_users_by_permission(self, _permission_id, _auth_header, _permission_scope):
        self.called = True
        return self.users

class DummyUserDAO:
    """Mock DAO for testing service layer."""
    def __init__(self):
        self.fetch_called = False
        self.create_called = False

    async def fetch_all(self):
        self.fetch_called = True
        return [UserDTO.Read(id=1, name='Test User', email='test@example.com')]

    async def create(self, dto):
        self.create_called = (dto,)

@pytest.mark.asyncio
async def test_service_coordinates_dao_and_adapter():
    """Test that service properly coordinates between DAO and adapter."""
    adapter = DummyUserAdapter([])
    dao = DummyUserDAO()
    service = UserService(user_adapter=adapter, user_dao=dao)

    result = await service.get_all_users()

    assert dao.fetch_called
    assert isinstance(result[0], UserDTO.Read)
```

### Pattern 4: Monkeypatching for Connection Mocking

```python
@pytest.mark.asyncio
async def test_assign_with_dal_connection(monkeypatch, faker):
    """Test method that uses DAL connection wrapper."""
    from src.domain.dal import DAL

    conn = FakeConnection()

    # Monkeypatch connection acquisition
    async def fake_get_connection(cls):
        return conn

    monkeypatch.setattr(DAL, '_DAL__get_connection', classmethod(fake_get_connection))

    # Stub other dependencies
    async def fake_verify_scope(id_, scope_type):
        return None

    monkeypatch.setattr(UserDAO, '_verify_scope', fake_verify_scope)

    # Prepare test data
    dto = UserDTO.Assign(user_id=1, role_id=2)

    # Call the actual DAO method (not __wrapped__)
    await UserDAO.assign(10, dto)

    # Verify execution
    assert len(conn.execute_calls) > 0
```

### Pattern 5: Testing Batch Operations

```python
@pytest.mark.asyncio
async def test_sync_calls_execute_many(faker):
    """Test that bulk sync uses execute_many for efficiency."""
    items = [
        UserDTO.Create(name=faker.name(), email=faker.email())
        for _ in range(3)
    ]

    conn = FakeConnection()
    executed = []

    async def fake_execute_many(stmt, parameters=None, **kwargs):
        params = parameters if parameters is not None else kwargs.get('parameters_list')
        executed.append((stmt, params))

    # Patch transaction's execute_many
    original_transaction = conn.transaction

    async def patched_transaction():
        t = await original_transaction().__aenter__()
        t.execute_many = fake_execute_many
        return t

    class PatchedTransactionContext:
        async def __aenter__(self):
            return await patched_transaction()

        async def __aexit__(self, exc_type, exc, tb):
            return False

    conn.transaction = lambda: PatchedTransactionContext()

    await UserDAO.sync.__wrapped__(conn, items)

    # Verify batch execution
    assert len(executed) == 1
    stmt, params = executed[0]
    assert 'INSERT INTO users' in stmt
    assert len(params[0]) == len(items)
```

### Pattern 6: FastAPI Endpoint Testing

```python
@pytest.mark.asyncio
async def test_get_users_endpoint(async_client, monkeypatch):
    """Test GET /users endpoint returns proper response."""
    # Mock the service layer
    async def mock_get_users():
        return [UserDTO.Read(id=1, name='Test', email='test@example.com')]

    monkeypatch.setattr('src.api.path.users.UserService.get_all', mock_get_users)

    # Make request
    response = await async_client.get('/users')

    # Assert response
    assert response.status_code == 200
    data = response.json()
    assert len(data) == 1
    assert data[0]['name'] == 'Test'
```

### Pattern 7: Testing with Multiple Return Values

```python
@pytest.mark.asyncio
async def test_multiple_queries_with_different_results(faker):
    """Test method that makes multiple queries with different expected results."""
    conn = FakeConnection()

    # Set up multiple return values (will be popped in order)
    conn.execute_return = [
        [{'id': 1, 'status': 'pending'}],  # First query
        [{'id': 2, 'status': 'approved'}]  # Second query
    ]

    # First call gets first result
    result1 = await UserDAO.some_method.__wrapped__(conn, 1)
    assert result1[0]['status'] == 'pending'

    # Second call gets second result
    result2 = await UserDAO.some_method.__wrapped__(conn, 2)
    assert result2[0]['status'] == 'approved'
```

### Pattern 8: Parametrized Tests for Multiple Scenarios

```python
@pytest.mark.asyncio
@pytest.mark.parametrize('status,expected_count', [
    ('pending', 5),
    ('approved', 3),
    ('rejected', 2),
])
async def test_count_by_status(status, expected_count):
    """Test counting users by different status values."""
    conn = FakeConnection()
    conn.fetch_val_return = expected_count

    result = await UserDAO.count_by_status.__wrapped__(conn, status)

    assert result == expected_count
    assert len(conn.fetch_val_calls) == 1
```

## Best Practices Checklist

### Before Writing Tests
- [ ] Identify the layer being tested (DAO/Service/Router/DTO)
- [ ] Determine required fixtures (app, client, faker, etc.)
- [ ] Plan mock objects needed (FakeConnection, dummy services, etc.)
- [ ] Understand the happy path and error scenarios

### During Test Writing
- [ ] Use descriptive test names: `test_<action>_<scenario>`
- [ ] Follow Arrange-Act-Assert pattern with clear sections
- [ ] Add docstrings explaining what the test validates
- [ ] Use type hints for all variables
- [ ] Mock at the right level (connection for DAO, service for router)
- [ ] Verify both success and failure paths
- [ ] Check SQL statements, not just return values
- [ ] Validate parameter counts and types

### After Writing Tests
- [ ] Run tests: `pytest tests/test_your_module.py -v`
- [ ] Check coverage: `pytest --cov=src/domain/dao/your_module tests/test_your_module.py`
- [ ] Verify all code paths are tested
- [ ] Remove commented code and print statements
- [ ] Ensure tests are isolated (no shared state)
- [ ] Run tests multiple times to verify consistency

## Common Pitfalls to Avoid

1. **Forgetting @pytest.mark.asyncio**: All async tests need this decorator
2. **Not using __wrapped__**: When testing DAO methods directly, bypass decorators
3. **Sharing state between tests**: Each test should be independent
4. **Over-mocking**: Mock at boundaries, not internal implementation details
5. **Ignoring SQL validation**: Always verify the actual SQL being executed
6. **Not testing exceptions**: Error paths are critical for robustness
7. **Missing type hints**: Makes tests harder to understand and maintain
8. **Vague test names**: Name should describe what and when

## Performance Tips

- Use `scope='session'` for expensive fixtures (app creation)
- Use `scope='function'` (default) for mutable fixtures
- Mock database connections rather than hitting real databases
- Group related tests in same file for better context
- Use `pytest -x` to stop on first failure during development
- Run specific test files during development: `pytest tests/test_dao.py`

## Integration with CI/CD

```bash
# Run all tests with coverage
pytest --cov=src --cov-report=html --cov-report=term

# Run only unit tests (fast)
pytest tests/ -m "not integration"

# Run with verbose output
pytest -v --tb=short

# Run specific test file
pytest tests/test_user_dao.py -v

# Run tests matching pattern
pytest -k "test_create" -v
```

## Example: Complete Test File

```python
"""Tests for UserDAO database access layer."""
from datetime import datetime
import pytest
from src.domain.dal.dao.user import UserDAO
from src.domain.dal.dao.exception import DAOException
from src.domain.dto.user import UserDTO
from tests.fakes import FakeConnection, FakeRecord


@pytest.mark.asyncio
async def test_create_inserts_user(faker):
    """Test that create method inserts user with correct parameters."""
    create_dto = UserDTO.Create(
        name=faker.name(),
        email=faker.email(),
        cpf=faker.ssn()
    )
    conn = FakeConnection()

    await UserDAO.create.__wrapped__(conn, create_dto)

    assert len(conn.execute_calls) == 1
    stmt, params = conn.execute_calls[0]
    assert 'INSERT INTO users' in stmt
    assert params[0] == create_dto.name


@pytest.mark.asyncio
async def test_fetch_by_id_returns_user(faker):
    """Test that fetch_by_id returns properly formatted UserDTO."""
    fake_row = {
        'id': faker.random_int(1, 100),
        'name': faker.name(),
        'email': faker.email(),
        'created_at': faker.date_time()
    }
    conn = FakeConnection()
    conn.fetch_row_return = FakeRecord(fake_row)

    result = await UserDAO.fetch_by_id.__wrapped__(conn, fake_row['id'])

    assert result.id == fake_row['id']
    assert result.name == fake_row['name']
    assert isinstance(result, UserDTO.Read)


@pytest.mark.asyncio
async def test_fetch_by_id_raises_on_db_error():
    """Test that database errors are properly handled and mapped."""
    conn = FakeConnection()

    async def broken_fetch_row(stmt, parameters=None):
        raise Exception('Connection lost')

    conn.fetch_row = broken_fetch_row

    with pytest.raises(DAOException) as exc:
        await UserDAO.fetch_by_id.__wrapped__(conn, 1)

    assert exc.value.status_code == 500
```

## Quick Reference Commands

```bash
# Run single test
pytest tests/test_user_dao.py::test_create_inserts_user -v

# Run all tests in file
pytest tests/test_user_dao.py -v

# Run with coverage for specific module
pytest --cov=src/domain/dao/user tests/test_user_dao.py

# Stop on first failure
pytest -x tests/

# Show local variables on failure
pytest --showlocals tests/

# Run last failed tests
pytest --lf tests/
```

## Summary

This skill provides production-proven patterns for async Python testing:

1. **Proper fixture setup** for FastAPI apps and async clients
2. **Comprehensive mocking** with FakeConnection and related classes
3. **Layer-specific testing** patterns (DAO, Service, Router)
4. **Exception handling** and error path testing
5. **Monkeypatching** for dependency injection
6. **Batch operation** testing patterns
7. **Best practices** for maintainable, robust tests

When in doubt, follow the "Arrange-Act-Assert" pattern and always verify both the happy path and error scenarios.
