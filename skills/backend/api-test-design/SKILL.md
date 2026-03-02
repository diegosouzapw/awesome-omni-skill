---
name: "API Test Design"
description: "Strategies for designing comprehensive API tests including contract testing, integration testing, and performance testing"
---

# API Test Design

You are an expert in API testing who helps design comprehensive test suites that ensure API reliability, correctness, and performance. You understand testing pyramids, contract testing, and the balance between different test types.

## Testing Philosophy

### The Testing Pyramid

```
        /\
       /E2E\          Few (10%)
      /------\
     /Integration\    Some (20%)
    /------------\
   /    Unit       \  Many (70%)
  /----------------\
```

**For APIs:**
```
        /\
       /Perf\         Few (5%)
      /------\
     / E2E API \      Some (15%)
    /----------\
   / Integration \    Some (30%)
  /--------------\
 / Unit + Contract \ Many (50%)
/------------------\
```

### Test Coverage Goals

**What to Test:**
- Happy paths (expected usage)
- Sad paths (error conditions)
- Edge cases (boundaries, empty, null)
- Security (auth, injection, validation)
- Performance (latency, throughput)
- Contract compliance (schema, types)

## Test Categories

### 1. Unit Tests

**Testing Business Logic:**
```python
# Function to test
def calculate_discount(price, user_tier):
    """Calculate discount based on user tier."""
    discounts = {
        'gold': 0.20,
        'silver': 0.10,
        'bronze': 0.05,
    }
    discount_rate = discounts.get(user_tier, 0)
    return price * discount_rate

# Unit tests
import pytest

def test_gold_tier_discount():
    assert calculate_discount(100, 'gold') == 20.0

def test_silver_tier_discount():
    assert calculate_discount(100, 'silver') == 10.0

def test_bronze_tier_discount():
    assert calculate_discount(100, 'bronze') == 5.0

def test_unknown_tier_no_discount():
    assert calculate_discount(100, 'platinum') == 0

def test_zero_price():
    assert calculate_discount(0, 'gold') == 0

def test_negative_price():
    # Should this be allowed? Test documents the behavior
    assert calculate_discount(-100, 'gold') == -20.0

@pytest.mark.parametrize("price,tier,expected", [
    (100, 'gold', 20),
    (100, 'silver', 10),
    (50, 'gold', 10),
    (0, 'gold', 0),
    (100, 'unknown', 0),
])
def test_discount_calculation(price, tier, expected):
    assert calculate_discount(price, tier) == expected
```

### 2. Integration Tests

**Testing API Endpoints:**
```python
import pytest
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

class TestUserAPI:
    def test_create_user_success(self):
        """Test successful user creation."""
        response = client.post("/api/users", json={
            "email": "test@example.com",
            "name": "Test User",
            "password": "secure_password123"
        })

        assert response.status_code == 201
        data = response.json()
        assert data["email"] == "test@example.com"
        assert data["name"] == "Test User"
        assert "password" not in data  # Password should not be returned
        assert "id" in data

    def test_create_user_duplicate_email(self):
        """Test duplicate email rejection."""
        # Create first user
        client.post("/api/users", json={
            "email": "duplicate@example.com",
            "name": "User 1",
            "password": "password123"
        })

        # Try to create duplicate
        response = client.post("/api/users", json={
            "email": "duplicate@example.com",
            "name": "User 2",
            "password": "password456"
        })

        assert response.status_code == 409
        assert "already exists" in response.json()["detail"].lower()

    def test_create_user_invalid_email(self):
        """Test email validation."""
        response = client.post("/api/users", json={
            "email": "invalid-email",
            "name": "Test User",
            "password": "password123"
        })

        assert response.status_code == 422
        errors = response.json()["detail"]
        assert any("email" in str(error).lower() for error in errors)

    def test_get_user_by_id(self):
        """Test retrieving user by ID."""
        # Create user
        create_response = client.post("/api/users", json={
            "email": "gettest@example.com",
            "name": "Get Test",
            "password": "password123"
        })
        user_id = create_response.json()["id"]

        # Get user
        response = client.get(f"/api/users/{user_id}")

        assert response.status_code == 200
        data = response.json()
        assert data["id"] == user_id
        assert data["email"] == "gettest@example.com"

    def test_get_user_not_found(self):
        """Test 404 for non-existent user."""
        response = client.get("/api/users/99999")

        assert response.status_code == 404
        assert "not found" in response.json()["detail"].lower()

    def test_update_user(self):
        """Test user update."""
        # Create user
        create_response = client.post("/api/users", json={
            "email": "update@example.com",
            "name": "Original Name",
            "password": "password123"
        })
        user_id = create_response.json()["id"]

        # Update user
        response = client.put(f"/api/users/{user_id}", json={
            "name": "Updated Name"
        })

        assert response.status_code == 200
        data = response.json()
        assert data["name"] == "Updated Name"
        assert data["email"] == "update@example.com"  # Unchanged

    def test_delete_user(self):
        """Test user deletion."""
        # Create user
        create_response = client.post("/api/users", json={
            "email": "delete@example.com",
            "name": "To Delete",
            "password": "password123"
        })
        user_id = create_response.json()["id"]

        # Delete user
        response = client.delete(f"/api/users/{user_id}")
        assert response.status_code == 204

        # Verify deleted
        get_response = client.get(f"/api/users/{user_id}")
        assert get_response.status_code == 404
```

### 3. Contract Testing

**Schema Validation:**
```python
from jsonschema import validate, ValidationError

# Define expected schema
USER_SCHEMA = {
    "type": "object",
    "properties": {
        "id": {"type": "integer"},
        "email": {"type": "string", "format": "email"},
        "name": {"type": "string", "minLength": 1},
        "created_at": {"type": "string", "format": "date-time"},
        "is_active": {"type": "boolean"}
    },
    "required": ["id", "email", "name", "created_at"],
    "additionalProperties": False
}

def test_user_response_schema():
    """Test that user response matches schema."""
    response = client.get("/api/users/1")
    assert response.status_code == 200

    data = response.json()

    # Validate against schema
    try:
        validate(instance=data, schema=USER_SCHEMA)
    except ValidationError as e:
        pytest.fail(f"Response doesn't match schema: {e.message}")

def test_list_users_schema():
    """Test that list response matches schema."""
    LIST_SCHEMA = {
        "type": "object",
        "properties": {
            "items": {
                "type": "array",
                "items": USER_SCHEMA
            },
            "total": {"type": "integer", "minimum": 0},
            "page": {"type": "integer", "minimum": 1},
            "page_size": {"type": "integer", "minimum": 1}
        },
        "required": ["items", "total"]
    }

    response = client.get("/api/users")
    assert response.status_code == 200

    validate(instance=response.json(), schema=LIST_SCHEMA)
```

**Pact Contract Testing:**
```python
from pact import Consumer, Provider

# Consumer side (API client)
pact = Consumer('UserServiceClient').has_pact_with(Provider('UserService'))

def test_get_user_contract():
    """Define contract for getting user."""
    expected = {
        'id': 1,
        'email': 'test@example.com',
        'name': 'Test User'
    }

    (pact
     .given('user 1 exists')
     .upon_receiving('a request for user 1')
     .with_request('GET', '/api/users/1')
     .will_respond_with(200, body=expected))

    with pact:
        client = UserServiceClient('http://localhost:1234')
        user = client.get_user(1)
        assert user['id'] == 1
        assert user['email'] == 'test@example.com'

# Provider side (API implementation) verifies it meets the contract
```

### 4. Authentication and Authorization Testing

**Auth Flow Testing:**
```python
class TestAuthentication:
    def test_login_success(self):
        """Test successful login."""
        response = client.post("/api/auth/login", json={
            "email": "user@example.com",
            "password": "correct_password"
        })

        assert response.status_code == 200
        data = response.json()
        assert "access_token" in data
        assert "refresh_token" in data
        assert data["token_type"] == "bearer"

    def test_login_invalid_credentials(self):
        """Test login with wrong password."""
        response = client.post("/api/auth/login", json={
            "email": "user@example.com",
            "password": "wrong_password"
        })

        assert response.status_code == 401
        assert "invalid credentials" in response.json()["detail"].lower()

    def test_access_protected_endpoint_without_token(self):
        """Test accessing protected endpoint without auth."""
        response = client.get("/api/protected/resource")

        assert response.status_code == 401
        assert "authorization" in response.json()["detail"].lower()

    def test_access_protected_endpoint_with_token(self):
        """Test accessing protected endpoint with valid token."""
        # Login to get token
        login_response = client.post("/api/auth/login", json={
            "email": "user@example.com",
            "password": "password123"
        })
        token = login_response.json()["access_token"]

        # Access protected endpoint
        response = client.get(
            "/api/protected/resource",
            headers={"Authorization": f"Bearer {token}"}
        )

        assert response.status_code == 200

    def test_expired_token(self):
        """Test that expired tokens are rejected."""
        expired_token = "expired.jwt.token"

        response = client.get(
            "/api/protected/resource",
            headers={"Authorization": f"Bearer {expired_token}"}
        )

        assert response.status_code == 401
        assert "expired" in response.json()["detail"].lower()

    def test_insufficient_permissions(self):
        """Test accessing admin endpoint as regular user."""
        # Login as regular user
        login_response = client.post("/api/auth/login", json={
            "email": "regular_user@example.com",
            "password": "password123"
        })
        token = login_response.json()["access_token"]

        # Try to access admin endpoint
        response = client.get(
            "/api/admin/users",
            headers={"Authorization": f"Bearer {token}"}
        )

        assert response.status_code == 403
        assert "permission" in response.json()["detail"].lower()
```

### 5. Error Handling Tests

**Comprehensive Error Coverage:**
```python
class TestErrorHandling:
    def test_malformed_json(self):
        """Test handling of malformed JSON."""
        response = client.post(
            "/api/users",
            data="not valid json",
            headers={"Content-Type": "application/json"}
        )

        assert response.status_code == 400
        assert "json" in response.json()["detail"].lower()

    def test_missing_required_field(self):
        """Test validation error for missing field."""
        response = client.post("/api/users", json={
            "email": "test@example.com"
            # Missing required 'name' field
        })

        assert response.status_code == 422
        errors = response.json()["detail"]
        assert any(
            error["loc"][-1] == "name" and error["type"] == "missing"
            for error in errors
        )

    def test_type_validation(self):
        """Test type validation errors."""
        response = client.post("/api/users", json={
            "email": "test@example.com",
            "name": "Test",
            "age": "not a number"  # Should be integer
        })

        assert response.status_code == 422

    def test_rate_limit_exceeded(self):
        """Test rate limiting."""
        # Make many requests
        for _ in range(100):
            response = client.get("/api/users")
            if response.status_code == 429:
                break
        else:
            pytest.fail("Rate limit not enforced")

        assert response.status_code == 429
        assert "retry-after" in response.headers

    def test_request_too_large(self):
        """Test request size limit."""
        large_data = {"name": "x" * 1_000_000}  # 1MB of data

        response = client.post("/api/users", json=large_data)

        assert response.status_code == 413  # Payload Too Large

    def test_unsupported_media_type(self):
        """Test content-type validation."""
        response = client.post(
            "/api/users",
            data="plain text data",
            headers={"Content-Type": "text/plain"}
        )

        assert response.status_code == 415  # Unsupported Media Type
```

### 6. Pagination and Filtering Tests

**Test Query Parameters:**
```python
class TestPaginationAndFiltering:
    def test_default_pagination(self):
        """Test default pagination parameters."""
        response = client.get("/api/users")

        assert response.status_code == 200
        data = response.json()
        assert "items" in data
        assert "total" in data
        assert "page" in data
        assert "page_size" in data
        assert data["page"] == 1
        assert data["page_size"] == 20  # Default

    def test_custom_page_size(self):
        """Test custom page size."""
        response = client.get("/api/users?page_size=5")

        data = response.json()
        assert len(data["items"]) <= 5
        assert data["page_size"] == 5

    def test_page_navigation(self):
        """Test navigating through pages."""
        # Get first page
        page1 = client.get("/api/users?page=1&page_size=10").json()

        # Get second page
        page2 = client.get("/api/users?page=2&page_size=10").json()

        # Should be different results
        assert page1["items"] != page2["items"]

    def test_filtering(self):
        """Test filtering by fields."""
        response = client.get("/api/users?status=active&role=admin")

        data = response.json()
        for user in data["items"]:
            assert user["status"] == "active"
            assert user["role"] == "admin"

    def test_sorting(self):
        """Test sorting results."""
        response = client.get("/api/users?sort=created_at&order=desc")

        data = response.json()
        dates = [item["created_at"] for item in data["items"]]

        # Verify descending order
        assert dates == sorted(dates, reverse=True)

    def test_search(self):
        """Test search functionality."""
        response = client.get("/api/users?search=john")

        data = response.json()
        for user in data["items"]:
            # Name or email should contain search term
            assert (
                "john" in user["name"].lower() or
                "john" in user["email"].lower()
            )
```

### 7. Performance Testing

**Load and Stress Tests:**
```python
import time
import concurrent.futures
import statistics

def test_response_time():
    """Test that API responds within acceptable time."""
    start = time.time()
    response = client.get("/api/users")
    duration = time.time() - start

    assert response.status_code == 200
    assert duration < 1.0  # Should respond within 1 second

def test_concurrent_requests():
    """Test handling concurrent requests."""
    def make_request():
        return client.get("/api/users")

    # Make 10 concurrent requests
    with concurrent.futures.ThreadPoolExecutor(max_workers=10) as executor:
        futures = [executor.submit(make_request) for _ in range(10)]
        results = [f.result() for f in concurrent.futures.as_completed(futures)]

    # All should succeed
    assert all(r.status_code == 200 for r in results)

def test_throughput():
    """Test API throughput."""
    num_requests = 100
    start = time.time()

    for _ in range(num_requests):
        response = client.get("/api/users")
        assert response.status_code == 200

    duration = time.time() - start
    requests_per_second = num_requests / duration

    print(f"Throughput: {requests_per_second:.2f} requests/second")
    assert requests_per_second > 10  # At least 10 req/s

def test_response_time_under_load():
    """Test response times under load."""
    response_times = []

    def make_timed_request():
        start = time.time()
        response = client.get("/api/users")
        duration = time.time() - start
        response_times.append(duration)
        return response

    # Make 100 concurrent requests
    with concurrent.futures.ThreadPoolExecutor(max_workers=20) as executor:
        futures = [executor.submit(make_timed_request) for _ in range(100)]
        results = [f.result() for f in concurrent.futures.as_completed(futures)]

    # Calculate statistics
    avg_time = statistics.mean(response_times)
    p95_time = statistics.quantiles(response_times, n=20)[18]  # 95th percentile
    p99_time = statistics.quantiles(response_times, n=100)[98]  # 99th percentile

    print(f"Average response time: {avg_time:.3f}s")
    print(f"P95 response time: {p95_time:.3f}s")
    print(f"P99 response time: {p99_time:.3f}s")

    assert avg_time < 0.5  # Average under 500ms
    assert p95_time < 1.0  # 95% under 1 second
    assert p99_time < 2.0  # 99% under 2 seconds
```

**Using Locust for Load Testing:**
```python
from locust import HttpUser, task, between

class APIUser(HttpUser):
    wait_time = between(1, 3)  # Wait 1-3 seconds between requests

    def on_start(self):
        """Login before starting tasks."""
        response = self.client.post("/api/auth/login", json={
            "email": "loadtest@example.com",
            "password": "password123"
        })
        self.token = response.json()["access_token"]

    @task(3)  # Weight: 3x more likely than other tasks
    def list_users(self):
        """Test listing users."""
        self.client.get(
            "/api/users",
            headers={"Authorization": f"Bearer {self.token}"}
        )

    @task(2)
    def get_user(self):
        """Test getting specific user."""
        self.client.get(
            f"/api/users/{random.randint(1, 100)}",
            headers={"Authorization": f"Bearer {self.token}"}
        )

    @task(1)
    def create_user(self):
        """Test creating user."""
        self.client.post(
            "/api/users",
            headers={"Authorization": f"Bearer {self.token}"},
            json={
                "email": f"user{random.randint(1, 10000)}@example.com",
                "name": "Load Test User",
                "password": "password123"
            }
        )

# Run: locust -f locustfile.py --host=http://localhost:8000
```

## Test Organization

### Test Structure

```python
# tests/
# ├── unit/
# │   ├── test_models.py
# │   ├── test_services.py
# │   └── test_utils.py
# ├── integration/
# │   ├── test_user_api.py
# │   ├── test_order_api.py
# │   └── test_auth_api.py
# ├── contract/
# │   ├── test_schemas.py
# │   └── pacts/
# ├── e2e/
# │   └── test_user_workflows.py
# ├── performance/
# │   ├── test_load.py
# │   └── locustfile.py
# └── conftest.py  # Shared fixtures

# conftest.py - Shared test fixtures
import pytest
from fastapi.testclient import TestClient
from app.main import app
from app.database import Base, engine, SessionLocal

@pytest.fixture(scope="function")
def db_session():
    """Create a clean database session for each test."""
    Base.metadata.create_all(bind=engine)
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
        Base.metadata.drop_all(bind=engine)

@pytest.fixture(scope="function")
def client(db_session):
    """Create test client with clean database."""
    def override_get_db():
        yield db_session

    app.dependency_overrides[get_db] = override_get_db
    yield TestClient(app)
    app.dependency_overrides.clear()

@pytest.fixture
def auth_headers(client):
    """Get authentication headers for tests."""
    response = client.post("/api/auth/login", json={
        "email": "test@example.com",
        "password": "password123"
    })
    token = response.json()["access_token"]
    return {"Authorization": f"Bearer {token}"}
```

## Test Data Management

### Factories and Fixtures

```python
import factory
from factory.faker import Faker
from app.models import User, Order

class UserFactory(factory.Factory):
    class Meta:
        model = User

    email = Faker('email')
    name = Faker('name')
    is_active = True

class OrderFactory(factory.Factory):
    class Meta:
        model = Order

    user = factory.SubFactory(UserFactory)
    total = Faker('pydecimal', left_digits=4, right_digits=2, positive=True)
    status = 'pending'

# Usage in tests
def test_create_order():
    user = UserFactory.create()
    order = OrderFactory.create(user=user, total=99.99)

    assert order.user.email == user.email
    assert order.total == 99.99
```

### Test Database Strategies

**Strategy 1: Transaction Rollback**
```python
import pytest
from sqlalchemy.orm import Session

@pytest.fixture
def db_session():
    """Rollback all changes after test."""
    connection = engine.connect()
    transaction = connection.begin()
    session = Session(bind=connection)

    yield session

    session.close()
    transaction.rollback()
    connection.close()
```

**Strategy 2: Separate Test Database**
```python
# Test configuration
TEST_DATABASE_URL = "postgresql://user:pass@localhost/test_db"

@pytest.fixture(scope="session")
def test_engine():
    """Create test database engine."""
    engine = create_engine(TEST_DATABASE_URL)
    Base.metadata.create_all(engine)
    yield engine
    Base.metadata.drop_all(engine)
```

**Strategy 3: In-Memory SQLite**
```python
@pytest.fixture
def db_session():
    """Use in-memory SQLite for fast tests."""
    engine = create_engine("sqlite:///:memory:")
    Base.metadata.create_all(engine)
    SessionLocal = sessionmaker(bind=engine)
    session = SessionLocal()

    yield session

    session.close()
```

## Best Practices

### 1. Test Independence

```python
# BAD: Tests depend on each other
def test_create_user():
    global user_id
    response = client.post("/api/users", json={"email": "test@example.com"})
    user_id = response.json()["id"]

def test_get_user():
    # Depends on test_create_user running first!
    response = client.get(f"/api/users/{user_id}")

# GOOD: Each test is independent
def test_create_user():
    response = client.post("/api/users", json={"email": "test@example.com"})
    assert response.status_code == 201

def test_get_user():
    # Create user within this test
    create_response = client.post("/api/users", json={"email": "test@example.com"})
    user_id = create_response.json()["id"]

    response = client.get(f"/api/users/{user_id}")
    assert response.status_code == 200
```

### 2. Clear Test Names

```python
# BAD: Unclear names
def test_user_1():
    ...

def test_user_2():
    ...

# GOOD: Descriptive names
def test_create_user_with_valid_data_returns_201():
    ...

def test_create_user_with_duplicate_email_returns_409():
    ...

def test_get_user_with_invalid_id_returns_404():
    ...
```

### 3. AAA Pattern (Arrange-Act-Assert)

```python
def test_update_user_email():
    # Arrange: Set up test data
    user = UserFactory.create(email="old@example.com")
    new_email = "new@example.com"

    # Act: Perform the action
    response = client.put(f"/api/users/{user.id}", json={
        "email": new_email
    })

    # Assert: Verify the results
    assert response.status_code == 200
    assert response.json()["email"] == new_email
```

## Related Skills

- **REST API Design**: Designing testable APIs
- **HTTP Protocol**: Understanding status codes and headers
- **Test Automation**: CI/CD integration
- **Performance Testing**: Load and stress testing
- **Security Testing**: Authentication and authorization testing
- **Mock Services**: Creating test doubles
- **OpenAPI/Swagger**: API documentation and contract testing
