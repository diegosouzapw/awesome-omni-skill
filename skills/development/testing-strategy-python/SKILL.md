---
name: testing-strategy-python
description: Python/FastAPI/Django testing conventions. pytest, fixtures, httpx, TestClient, factory_boy. Use when writing or reviewing Python tests.
user-invocable: false
---

Python testing conventions for FastAPI, Django, and Flask.

## Frameworks
- **Unit/Integration tests**: pytest (always — never unittest style)
- **API tests**: `httpx.AsyncClient` (FastAPI) or `TestClient`
- **Mocking**: `pytest-mock` (wrapper around `unittest.mock`)
- **Fixtures**: pytest fixtures (not setUp/tearDown)
- **Test data**: `factory_boy` or plain fixture functions

## Unit Tests

```python
import pytest
from unittest.mock import AsyncMock
from app.services.order_service import OrderService

@pytest.fixture
def mock_repo():
    repo = AsyncMock()
    repo.find_by_id.return_value = {"id": 1, "status": "PENDING"}
    repo.save.return_value = {"id": 1, "status": "PENDING"}
    return repo

@pytest.fixture
def service(mock_repo):
    return OrderService(repository=mock_repo)

async def test_create_order_sets_pending_status(service, mock_repo):
    result = await service.create_order(product_id=1, quantity=2)

    assert result["status"] == "PENDING"
    mock_repo.save.assert_called_once()

async def test_find_order_raises_when_not_found(service, mock_repo):
    mock_repo.find_by_id.return_value = None

    with pytest.raises(OrderNotFoundError):
        await service.find_by_id(999)
```

### Rules
- Use `pytest.fixture` for all setup — never class-based `setUp`/`tearDown`
- Use `AsyncMock` for async functions, `MagicMock` for sync
- Fixtures compose: small fixtures combine into larger ones
- Use `conftest.py` for shared fixtures across test modules

## FastAPI API Tests

```python
import pytest
from httpx import AsyncClient, ASGITransport
from app.main import app

@pytest.fixture
async def client():
    transport = ASGITransport(app=app)
    async with AsyncClient(transport=transport, base_url="http://test") as client:
        yield client

async def test_get_orders_returns_list(client):
    response = await client.get("/api/orders", headers={"Authorization": "Bearer test-token"})

    assert response.status_code == 200
    assert isinstance(response.json(), list)

async def test_create_order_validates_input(client):
    response = await client.post("/api/orders", json={"quantity": -1})

    assert response.status_code == 422
    assert "quantity" in response.json()["detail"][0]["loc"]

async def test_get_order_returns_404_when_not_found(client):
    response = await client.get("/api/orders/999")

    assert response.status_code == 404
```

### Rules
- Use `httpx.AsyncClient` with `ASGITransport` for async FastAPI
- Use `TestClient` from Starlette for sync tests (simpler)
- Override dependencies with `app.dependency_overrides[get_db] = mock_db`
- Test status codes, response body, validation errors

## Django Tests

```python
import pytest
from django.test import Client
from app.models import Order

@pytest.fixture
def api_client():
    return Client()

@pytest.mark.django_db
def test_list_orders(api_client):
    Order.objects.create(status="PENDING")

    response = api_client.get("/api/orders/")

    assert response.status_code == 200
    assert len(response.json()) == 1

@pytest.mark.django_db
def test_create_order(api_client):
    response = api_client.post("/api/orders/", data={"product_id": 1}, content_type="application/json")

    assert response.status_code == 201
    assert Order.objects.count() == 1
```

### Rules
- Mark DB tests with `@pytest.mark.django_db`
- Use `pytest-django` plugin
- Use `baker.make(Order)` from `model_bakery` for test data
- Transaction rollback is automatic per test

## Fixtures Pattern (conftest.py)

```python
# tests/conftest.py
import pytest
from app.database import get_test_db

@pytest.fixture(scope="session")
def db_engine():
    """Create test database once per session."""
    engine = create_engine("sqlite:///test.db")
    Base.metadata.create_all(engine)
    yield engine
    Base.metadata.drop_all(engine)

@pytest.fixture
def db_session(db_engine):
    """Fresh DB session per test with rollback."""
    connection = db_engine.connect()
    transaction = connection.begin()
    session = Session(bind=connection)
    yield session
    session.close()
    transaction.rollback()
    connection.close()

@pytest.fixture
def sample_order(db_session):
    order = Order(status="PENDING", product_id=1)
    db_session.add(order)
    db_session.commit()
    return order
```

## File Structure
```
tests/
├── conftest.py                 Shared fixtures
├── unit/
│   ├── test_order_service.py
│   └── test_payment_service.py
├── api/
│   ├── test_orders_api.py
│   └── test_auth_api.py
└── integration/
    └── test_order_workflow.py
```

## Naming
- File: `test_module_name.py` (pytest auto-discovers `test_` prefix)
- Function: `test_should_behavior_when_condition` or `test_behavior_description`
- No classes needed — plain functions with fixtures
