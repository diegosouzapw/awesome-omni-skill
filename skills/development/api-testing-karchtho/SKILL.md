---
name: api-testing
description: API testing strategies with Postman/Newman MCP integration, contract testing, unit and integration tests, and framework-specific examples. Use when setting up API testing infrastructure, integrating Postman MCP, writing test suites, or automating API testing in CI/CD pipelines.
version: 1.0.0
---

# API Testing

Master API testing strategies and automation to ensure your APIs are reliable, secure, and performant.

## When to Use This Skill

- Setting up API testing infrastructure
- Integrating Postman MCP for automation
- Writing unit and integration tests for endpoints
- Implementing contract testing and validation
- Setting up CI/CD pipeline testing
- Testing GraphQL queries and mutations
- Automating API test execution
- Building test suites with multiple frameworks

## Testing Pyramid for APIs

Structure your tests from bottom to top:

```
        E2E Tests (Postman collections, full flow)
       /                  \
      /                    \
   Integration Tests       API Tests
   (DB, caching, etc.)    (endpoint behavior)
    /                        \
   /                          \
Unit Tests (business logic, validation)
```

### 1. Unit Tests (Foundation)

Test individual functions and business logic in isolation:

```python
# pytest example
import pytest
from app.models import User

def test_user_password_hashing():
    user = User(email="test@example.com")
    user.set_password("mypassword")
    assert user.password != "mypassword"
    assert user.check_password("mypassword") is True
    assert user.check_password("wrongpassword") is False

def test_user_email_validation():
    with pytest.raises(ValueError):
        User(email="invalid-email")
```

### 2. Integration Tests

Test API endpoints with real dependencies (database, caching):

```python
# pytest example with fixtures
import pytest
from fastapi.testclient import TestClient
from app import app
from app.db import get_db

@pytest.fixture
def client(db_session):
    def override_get_db():
        return db_session
    app.dependency_overrides[get_db] = override_get_db
    return TestClient(app)

def test_create_user(client):
    response = client.post("/api/users", json={
        "email": "newuser@example.com",
        "name": "New User"
    })
    assert response.status_code == 201
    assert response.json()["email"] == "newuser@example.com"

def test_get_user(client, db_session):
    # Create test user
    from app.models import User
    user = User(email="test@example.com", name="Test")
    db_session.add(user)
    db_session.commit()

    response = client.get(f"/api/users/{user.id}")
    assert response.status_code == 200
    assert response.json()["name"] == "Test"
```

### 3. End-to-End Tests

Test full user workflows using Postman collections:

- Import from Postman collections
- Run with Newman CLI
- Execute in CI/CD pipelines
- Test realistic user scenarios

## REST API Testing with Pytest

### Basic Endpoint Testing

```python
import pytest
from fastapi.testclient import TestClient
from app import app

client = TestClient(app)

class TestUserAPI:
    def test_list_users(self):
        response = client.get("/api/users")
        assert response.status_code == 200
        assert isinstance(response.json(), list)

    def test_create_user_success(self):
        response = client.post("/api/users", json={
            "email": "newuser@example.com",
            "name": "New User",
            "password": "secure123"
        })
        assert response.status_code == 201
        data = response.json()
        assert data["email"] == "newuser@example.com"
        assert "password" not in data  # Never expose passwords

    def test_create_user_validation_error(self):
        response = client.post("/api/users", json={
            "email": "invalid-email",
            "name": "Test"
        })
        assert response.status_code == 422
        assert "email" in response.json()["detail"][0]["loc"]

    def test_get_nonexistent_user(self):
        response = client.get("/api/users/99999")
        assert response.status_code == 404

    def test_update_user(self):
        # Create user first
        create_resp = client.post("/api/users", json={
            "email": "user@example.com",
            "name": "Original"
        })
        user_id = create_resp.json()["id"]

        # Update user
        response = client.patch(f"/api/users/{user_id}", json={
            "name": "Updated"
        })
        assert response.status_code == 200
        assert response.json()["name"] == "Updated"

    def test_delete_user(self):
        # Create user
        create_resp = client.post("/api/users", json={
            "email": "deleteme@example.com",
            "name": "Delete Me"
        })
        user_id = create_resp.json()["id"]

        # Delete
        response = client.delete(f"/api/users/{user_id}")
        assert response.status_code == 204

        # Verify deletion
        get_resp = client.get(f"/api/users/{user_id}")
        assert get_resp.status_code == 404
```

### Parametrized Testing

```python
import pytest

@pytest.mark.parametrize("status_code,expected_status", [
    (200, 200),
    (201, 201),
    (400, 400),
    (404, 404),
])
def test_status_codes(status_code, expected_status):
    response = client.get(f"/api/test?status={status_code}")
    assert response.status_code == expected_status
```

## GraphQL Testing with Jest

### Basic Query Testing

```javascript
import { gql } from '@apollo/client';
import { ApolloClient } from '@apollo/client';

const client = new ApolloClient({
  uri: 'http://localhost:4000/graphql'
});

describe('GraphQL User Queries', () => {
  it('should fetch user by ID', async () => {
    const query = gql`
      query GetUser($id: ID!) {
        user(id: $id) {
          id
          name
          email
        }
      }
    `;

    const result = await client.query({
      query,
      variables: { id: '1' }
    });

    expect(result.data.user).toBeDefined();
    expect(result.data.user.email).toBe('user@example.com');
  });

  it('should list users with pagination', async () => {
    const query = gql`
      query ListUsers($first: Int!) {
        users(first: $first) {
          edges {
            node {
              id
              name
            }
          }
          pageInfo {
            hasNextPage
          }
        }
      }
    `;

    const result = await client.query({
      query,
      variables: { first: 10 }
    });

    expect(result.data.users.edges).toHaveLength(10);
    expect(result.data.users.pageInfo.hasNextPage).toBeDefined();
  });
});

describe('GraphQL Mutations', () => {
  it('should create user with validation', async () => {
    const mutation = gql`
      mutation CreateUser($input: CreateUserInput!) {
        createUser(input: $input) {
          user {
            id
            email
          }
          errors {
            field
            message
          }
          success
        }
      }
    `;

    const result = await client.mutate({
      mutation,
      variables: {
        input: { email: 'newuser@example.com', name: 'New' }
      }
    });

    expect(result.data.createUser.success).toBe(true);
    expect(result.data.createUser.user.id).toBeDefined();
    expect(result.data.createUser.errors).toHaveLength(0);
  });

  it('should return errors for invalid input', async () => {
    const mutation = gql`
      mutation CreateUser($input: CreateUserInput!) {
        createUser(input: $input) {
          user {
            id
          }
          errors {
            field
            message
          }
          success
        }
      }
    `;

    const result = await client.mutate({
      mutation,
      variables: {
        input: { email: 'invalid-email', name: '' }
      }
    });

    expect(result.data.createUser.success).toBe(false);
    expect(result.data.createUser.errors.length).toBeGreaterThan(0);
  });
});
```

## Postman MCP Integration

### Installation

The Postman MCP server enables running Postman collections through Claude Code:

**Method 1: Via Smithery (Recommended)**
```bash
npx -y @smithery/cli install mcp-postman --client claude
```

**Method 2: Manual Installation**
```bash
git clone https://github.com/shannonlal/mcp-postman.git
cd mcp-postman
pnpm install
pnpm build
```

### Configuration

Add to `~/.config/claude/config.json`:
```json
{
  "mcpServers": {
    "postman": {
      "command": "node",
      "args": ["/path/to/mcp-postman/build/index.js"],
      "env": {
        "POSTMAN_API_KEY": "your-api-key-here"
      }
    }
  }
}
```

### Using Postman MCP

Once configured, interact with Postman from Claude Code:

```
User: "Run the user-api collection from my Postman workspace"

Claude can now:
- List available collections
- Execute collections with specific environments
- Parse test results
- Run in CI/CD pipelines
- Generate test reports
```

## Newman Automation

Newman CLI runs Postman collections headlessly for CI/CD:

### Installation
```bash
npm install -g newman
npm install -g newman-reporter-html
```

### Running Collections

```bash
# Basic execution
newman run collection.json

# With environment
newman run collection.json -e environment.json

# With variables
newman run collection.json --global-var "base_url=https://api.example.com"

# Multiple reporters
newman run collection.json \
  --reporters cli,json,html \
  --reporter-json-export results.json \
  --reporter-html-export report.html
```

### CI/CD Integration Example

**GitHub Actions:**
```yaml
name: API Tests
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
      - run: npm install -g newman
      - run: |
          newman run postman/collection.json \
            -e postman/environment.json \
            --reporters cli,json \
            --reporter-json-export results.json
```

## Contract Testing

Validate APIs match their specifications:

### OpenAPI Validation

```python
# Using openapi-spec-validator
from openapi_spec_validator import validate_spec
import json

with open('openapi.json') as f:
    spec = json.load(f)

try:
    validate_spec(spec)
    print("OpenAPI spec is valid")
except Exception as e:
    print(f"Spec validation error: {e}")
```

### Pact Consumer/Provider Testing

```javascript
// Consumer test with Pact
import { Pact } from '@pact-foundation/pact';

describe('User Consumer', () => {
  const provider = new Pact({
    consumer: 'UserApp',
    provider: 'UserAPI'
  });

  it('should fetch user', () => {
    return provider
      .addInteraction({
        state: 'user exists',
        uponReceiving: 'a request for user 123',
        withRequest: { method: 'GET', path: '/api/users/123' },
        willRespondWith: {
          status: 200,
          body: { id: 123, name: 'John', email: 'john@example.com' }
        }
      })
      .then(() => fetch('http://localhost:8080/api/users/123'))
      .then(response => response.json())
      .then(data => expect(data.email).toBe('john@example.com'));
  });
});
```

## Authentication Testing

```python
def test_bearer_token_required(client):
    response = client.get("/api/users")
    assert response.status_code == 401

def test_valid_bearer_token(client):
    headers = {"Authorization": "Bearer valid-token-123"}
    response = client.get("/api/users", headers=headers)
    assert response.status_code == 200

def test_invalid_bearer_token(client):
    headers = {"Authorization": "Bearer invalid-token"}
    response = client.get("/api/users", headers=headers)
    assert response.status_code == 401

def test_api_key_validation(client):
    headers = {"X-API-Key": "valid-key"}
    response = client.get("/api/users", headers=headers)
    assert response.status_code == 200

    headers = {"X-API-Key": "invalid-key"}
    response = client.get("/api/users", headers=headers)
    assert response.status_code == 401
```

## Performance Testing

```python
# Load testing with locust
from locust import HttpUser, task, constant

class APIUser(HttpUser):
    wait_time = constant(1)

    @task
    def list_users(self):
        self.client.get("/api/users")

    @task
    def get_user(self):
        self.client.get("/api/users/1")
```

Run with: `locust -f locustfile.py -u 100 -r 10`

## Best Practices

1. **Test at Multiple Levels**: Unit → Integration → E2E
2. **Use Test Fixtures**: Reusable setup/teardown for consistent tests
3. **Parametrize Tests**: Test multiple scenarios efficiently
4. **Mock External APIs**: Isolate your tests from dependencies
5. **Automate in CI/CD**: Run tests on every commit
6. **Document Test Cases**: Clear test names and docstrings
7. **Test Error Cases**: Not just happy paths
8. **Use Postman MCP**: Automate Postman collection execution
9. **Monitor Test Coverage**: Aim for >80% coverage
10. **Test Performance**: Regular load and performance tests

## Common Pitfalls to Avoid

- Testing implementation details instead of behavior
- Flaky tests that fail intermittently
- Not testing error responses
- Ignoring authentication/authorization tests
- Missing edge cases and boundary conditions
- Slow test suites (optimize or split)
- Testing external APIs without mocking
- No test data management strategy

## Cross-Skill References

- **rest-api-design skill** - For REST endpoint patterns to test
- **graphql-api-design skill** - For GraphQL query patterns to test
- **api-architecture skill** - For security and monitoring strategies

## Additional Resources

For detailed guidance, see:
- `references/postman-mcp-setup.md` - Complete Postman MCP setup
- `references/testing-strategies.md` - Testing patterns and organization
- `references/contract-testing.md` - OpenAPI and Pact testing
