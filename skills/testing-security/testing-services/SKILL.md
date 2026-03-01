---
name: testing-services
description: Writes unit tests for Python service classes using Arrange-Act-Assert pattern with proper mocking at boundaries. Tests behavior, not implementation. Mocks external systems only (API calls, file I/O, databases). Use when writing tests for services or fixing test coverage.
user-invocable: true
argument-hint: "[service-class-name]"
---

# Testing Services

## When to Use This Skill

Activate this skill when:
- Writing unit tests for service classes
- Need to improve test coverage for services
- Unclear what to mock vs what to test directly
- Writing integration tests that test service interactions

## Quick Start

### Running Tests Locally

```bash
# Run all unit tests
pytest tests/unit/ -v

# Run all integration tests
pytest tests/integration/ -v

# Run both unit and integration tests
pytest tests/unit/ tests/integration/ -v

# Run with coverage report
pytest tests/unit/ tests/integration/ --cov=src --cov-report=term-missing --cov-report=html

# Run tests for a specific module
pytest tests/unit/services/test_entity_service.py -v

# Run a specific test
pytest tests/unit/services/test_entity_service.py::TestEntityService::test_create_entity_success -v
```

### Understanding Test Results

```bash
# View coverage in terminal
pytest tests/ --cov=src --cov-report=term-missing

# View detailed HTML coverage report (opens in browser)
pytest tests/ --cov=src --cov-report=html
open htmlcov/index.html  # macOS
xdg-open htmlcov/index.html  # Linux
```

## Quick Reference

### AAA Pattern

```python
def test_service_operation():
    # Arrange: Set up dependencies and test data
    mock_dependency = Mock(spec=Dependency)
    mock_dependency.method.return_value = "expected"
    service = ServiceClass(mock_dependency)

    # Act: Execute the operation under test
    result = service.operation(param="value")

    # Assert: Verify behavior and outcomes
    assert result == "expected_outcome"
    mock_dependency.method.assert_called_once_with("value")
```

### Running Tests

```bash
pytest tests/ -v                              # Run all tests
pytest tests/unit/test_service.py -v          # Run specific file
pytest tests/ --cov=src --cov-report=term-missing  # With coverage
pytest tests/ -k "test_create" -v             # Pattern matching
```

### Coverage Guidelines

- Aim for 85%+ overall coverage
- Focus on testing behavior, not hitting targets
- Missing coverage acceptable for edge cases or trivial code

## Test Isolation and Independence

**Every test should be completely independent.** Tests must not:
- Rely on execution order
- Share mutable state
- Depend on previous test results
- Affect other tests when run in parallel

### Example

```python
# ✅ GOOD - Each test is self-contained
class TestTaskManagement:
    def test_find_first_task(self, tmp_path):
        """Should find the first unchecked task"""
        # Arrange - Create fresh test data for this test only
        spec_file = tmp_path / "spec.md"
        spec_file.write_text("""
        - [ ] Task 1
        - [ ] Task 2
        """)

        # Act
        result = find_next_available_task(str(spec_file))

        # Assert
        assert result == (1, "Task 1")

# ❌ BAD - Tests depend on shared state
shared_state = {"count": 0}

def test_increment():
    shared_state["count"] += 1
    assert shared_state["count"] == 1  # Breaks if tests run out of order
```

## Mock at Boundaries Rule

**The Golden Rule**: Mock external systems only, never internal logic.

### What to Mock ✅

**External systems and infrastructure**:
- HTTP API clients, database connections, repositories
- File system operations, subprocess calls
- External service clients (email, payment, etc.)

**Service dependencies**:
- Other services injected via constructor (enables isolation testing)

### What NOT to Mock ❌

**Internal logic**:
- Domain models and dataclasses (see **domain-modeling** skill for understanding what domain models are)
- Pure functions, helper methods, static methods
- Business logic calculations
- Exceptions and error classes

**Standard library basics**:
- datetime (use fixed times), string/list/dict operations

### Decision Guide

Ask: "Is this an external dependency or internal logic?"
- External dependency → Mock it
- Internal logic → Test it directly

## Service Test Templates

### Testing a Core Service

```python
from unittest.mock import Mock
import pytest
from src.services.entity_service import EntityService
from src.domain.entity import Entity
from src.domain.exceptions import ValidationError


class TestEntityService:
    """Tests for EntityService."""

    def test_create_entity_success(self):
        # Arrange: Mock external dependencies
        mock_repository = Mock()
        mock_validator = Mock()
        mock_validator.is_valid.return_value = True
        service = EntityService(mock_repository, mock_validator)

        # Act
        result = service.create_entity(name="Test Entity", value=42)

        # Assert: Verify behavior
        assert result.name == "Test Entity"
        assert result.value == 42
        mock_validator.is_valid.assert_called_once()
        mock_repository.save.assert_called_once()

    def test_create_entity_validation_failure(self):
        # Arrange
        mock_repository = Mock()
        mock_validator = Mock()
        mock_validator.is_valid.return_value = False
        service = EntityService(mock_repository, mock_validator)

        # Act & Assert
        with pytest.raises(ValidationError, match="Invalid entity data"):
            service.create_entity(name="Bad", value=-1)

        mock_repository.save.assert_not_called()

    def test_find_by_criteria_filters_correctly(self):
        # Arrange: Use real Entity objects (don't mock domain models!)
        mock_repository = Mock()
        entities = [
            Entity(name="Low", value=10),
            Entity(name="High", value=100),
        ]
        mock_repository.list_all.return_value = entities
        service = EntityService(mock_repository, Mock())

        # Act
        result = service.find_by_criteria(min_value=50)

        # Assert
        assert len(result) == 1
        assert result[0].name == "High"
```

### Testing a Composite Service

```python
from unittest.mock import Mock
import pytest
from src.services.workflow_service import WorkflowService
from src.domain.exceptions import WorkflowError, ValidationError


class TestWorkflowService:
    """Tests for WorkflowService."""

    def test_execute_workflow_success(self):
        # Arrange: Mock all Core service dependencies
        mock_entity_service = Mock()
        mock_notification_service = Mock()
        mock_audit_service = Mock()

        mock_entity = Mock(id="entity-123")
        mock_entity_service.create_entity.return_value = mock_entity

        service = WorkflowService(
            mock_entity_service,
            mock_notification_service,
            mock_audit_service
        )

        # Act
        result = service.execute_workflow(name="Test", value=100, user_id="user-456")

        # Assert: Verify orchestration
        assert result.success is True
        assert result.entity_id == "entity-123"
        mock_entity_service.create_entity.assert_called_once_with("Test", 100)
        mock_notification_service.notify_entity_created.assert_called_once()
        mock_audit_service.log_success.assert_called_once()

    def test_execute_workflow_handles_failure(self):
        # Arrange
        mock_entity_service = Mock()
        mock_entity_service.create_entity.side_effect = ValidationError("Invalid")
        mock_notification_service = Mock()
        mock_audit_service = Mock()

        service = WorkflowService(
            mock_entity_service,
            mock_notification_service,
            mock_audit_service
        )

        # Act & Assert
        with pytest.raises(WorkflowError, match="Entity creation failed"):
            service.execute_workflow(name="Bad", value=-1, user_id="user-456")

        mock_audit_service.log_failure.assert_called_once()
        mock_notification_service.notify_entity_created.assert_not_called()

    def test_calculate_priority_static_method(self):
        # Static methods are pure - test directly, no mocking needed
        result = WorkflowService.calculate_priority(value=200, user_tier="premium")
        assert result == 4

        result = WorkflowService.calculate_priority(value=150, user_tier="standard")
        assert result == 1
```

## Layer-Based Testing Strategy

Python applications often follow a **layered architecture**, and we test each layer differently:

```
┌─────────────────────────────────────────┐
│          CLI Layer                      │  ← Test command orchestration
│  (commands: prepare, finalize, etc.)    │     Mock everything below
├─────────────────────────────────────────┤
│       Service Layer                     │  ← Test business logic
│  (services: task mgmt, reviewer mgmt)   │     Mock infrastructure
├─────────────────────────────────────────┤
│     Infrastructure Layer                │  ← Test external integrations
│  (git, github, filesystem)              │     Mock external systems
├─────────────────────────────────────────┤
│         Domain Layer                    │  ← Test directly
│  (models, config, exceptions)           │     Minimal/no mocking
└─────────────────────────────────────────┘
```

**Key insight:** The **lower** the layer, the **less** you mock. Domain models are pure logic and need minimal mocking. CLI commands orchestrate everything and need maximum mocking.

### Domain Layer (99% coverage target)

**Testing approach:**
- ✅ **Direct testing** - Minimal mocking
- ✅ **Test business rules** - Validation logic, data transformations
- ✅ **Test edge cases** - Invalid inputs, boundary conditions

**What to mock:** Almost nothing. Domain models are pure logic.

**Example:**
```python
class TestMarkdownFormatter:
    """Test suite for MarkdownFormatter functionality"""

    def test_bold_formatting_for_github(self):
        """Should format text as bold using GitHub markdown syntax"""
        # Arrange - No mocking needed
        formatter = MarkdownFormatter(for_slack=False)

        # Act
        result = formatter.bold("important")

        # Assert
        assert result == "**important**"
```

### Infrastructure Layer (97% coverage target)

**Testing approach:**
- ✅ **Mock external systems** - subprocess, HTTP, file I/O
- ✅ **Test success and error paths** - What happens on success? On failure?
- ✅ **Verify system calls** - Check correct commands are executed

**What to mock:** Everything external to Python (subprocess, network, filesystem)

### Service Layer (95% coverage target)

**Testing approach:**
- ✅ **Mock infrastructure** - Mock git, GitHub API, filesystem
- ✅ **Test business logic** - Algorithms, data processing, validation
- ✅ **Test service instantiation** - Verify services are created with correct dependencies
- ✅ **Test edge cases** - No reviewers available, all tasks complete, etc.

**What to mock:** Infrastructure layer (subprocess, GitHub API, file I/O) and service dependencies

### CLI Integration Tests (98% coverage target)

**Testing approach:**
- ✅ **Mock service layer** - Mock service classes and their methods
- ✅ **Mock infrastructure** - Mock git, GitHub API, filesystem operations
- ✅ **Test command orchestration** - Verify correct sequence of service calls
- ✅ **Test service instantiation** - Verify services are created with correct dependencies
- ✅ **Test output** - Verify CLI outputs are formatted correctly

**What to mock:** Service Layer classes and Infrastructure dependencies

## Key Testing Patterns

### Pattern: Use Real Domain Models

```python
def test_process_order():
    mock_repository = Mock()
    service = OrderService(mock_repository)

    # Use real Order domain model, not a mock
    result = service.process_order(customer_id="c123", items=["item1"], total=99.99)

    assert isinstance(result, Order)
    assert result.customer_id == "c123"
    assert result.total == 99.99
```

### Pattern: Test Exception Handling

```python
def test_find_user_not_found():
    mock_repository = Mock()
    mock_repository.get_by_id.return_value = None
    service = UserService(mock_repository)

    with pytest.raises(UserNotFoundError, match="User u999 not found"):
        service.find_user("u999")
```

### Pattern: Verify Side Effects

```python
def test_send_notification():
    mock_email_client = Mock()
    service = NotificationService(mock_email_client)

    service.send_notification(user_email="user@example.com", message="Hello!")

    mock_email_client.send.assert_called_once_with(
        to="user@example.com",
        subject="Notification",
        body="Hello!"
    )
```

### Pattern: Parametrized Tests

```python
import pytest

@pytest.mark.parametrize("value,tier,expected", [
    (100, "premium", 2),
    (100, "standard", 1),
    (200, "premium", 4),
])
def test_calculate_priority(value, tier, expected):
    result = WorkflowService.calculate_priority(value, tier)
    assert result == expected
```

## Mock vs MagicMock vs Real Objects

### Use `Mock(spec=ClassName)`

For most scenarios - provides autocomplete and catches attribute errors.

```python
mock_repository = Mock(spec=UserRepository)
mock_repository.find_by_id.return_value = User(id="123")
```

### Use `MagicMock`

When you need magic methods (`__len__`, `__iter__`, `__getitem__`).

```python
from unittest.mock import MagicMock

mock_collection = MagicMock()
mock_collection.__len__.return_value = 5
```

### Use Real Objects

For domain models, value objects, exceptions.

```python
user = User(id="123", name="Alice")  # Real domain model
order = Order(items=[Item("book")])  # Real value objects

mock_service.process.side_effect = ValidationError("Bad data")  # Real exception
```

## Fixtures for Reusable Setup

```python
import pytest
from unittest.mock import Mock

@pytest.fixture
def mock_repository():
    return Mock(spec=UserRepository)

@pytest.fixture
def user_service(mock_repository):
    return UserService(mock_repository)

def test_create_user(user_service, mock_repository):
    mock_repository.save.return_value = User(id="123")
    result = user_service.create_user(name="Alice")
    assert result.id == "123"
```

## Integration Tests

Integration tests verify services work together - only mock infrastructure.

```python
def test_workflow_integration():
    # Mock infrastructure only
    mock_database = Mock()
    mock_email_client = Mock()

    # Use real services
    entity_service = EntityService(mock_database)
    notification_service = NotificationService(mock_email_client)
    workflow_service = WorkflowService(entity_service, notification_service)

    # Test real interaction
    result = workflow_service.execute_workflow(name="Test", value=100, user_id="u123")

    assert result.success is True
    mock_database.save.assert_called()
    mock_email_client.send.assert_called()
```

## One Concept Per Test

**Each test should verify ONE specific behavior.** If a test has multiple unrelated assertions, split it into separate tests.

**Why?** When a test fails, you should immediately know what broke. Multi-concept tests make debugging harder.

**Exception for E2E Tests:** End-to-end tests are expensive and slow to run (often taking minutes). For E2E tests, it's acceptable and encouraged to verify multiple related aspects of a workflow in a single test to minimize execution time. For example, an E2E test that triggers a workflow can verify:
- The workflow completes successfully
- A PR was created
- The PR has correct labels
- The PR has an AI summary
- The PR has cost information

This is a pragmatic trade-off: E2E tests prioritize execution speed over granular isolation, while unit/integration tests maintain strict one-concept-per-test discipline.

### Example

```python
# ❌ WRONG: Tests multiple unrelated things
def test_user_service():
    service = UserService(mock_repo)
    user = service.create_user("Alice")
    assert user.name == "Alice"
    found = service.find_user(user.id)
    service.delete_user(user.id)  # Multiple unrelated behaviors

# ✅ RIGHT: Separate focused tests
def test_create_user_sets_name():
    service = UserService(mock_repo)
    user = service.create_user("Alice")
    assert user.name == "Alice"

def test_find_user_returns_existing():
    service = UserService(mock_repo)
    mock_repo.get_by_id.return_value = User(id="123")
    found = service.find_user("123")
    assert found.id == "123"
```

## Coverage Guidelines

### Current Coverage Status

**Target:** 85%+ overall coverage with meaningful tests that provide confidence in refactoring and catch real regressions.

### What to Test

**High Priority (90%+ coverage):**
- Business logic (task management, data processing algorithms)
- Command orchestration (CLI commands, workflow coordination)
- Configuration validation
- Error handling paths

**Medium Priority (70%+ coverage):**
- Utilities and formatters
- API integrations (GitHub, external services)
- File operations and artifact handling

**Low Priority (may skip):**
- Simple getters/setters
- Third-party library wrappers
- CLI argument parsing (tested via E2E)
- Entry points (`__main__.py` - tested via E2E)

### Viewing Coverage Reports

```bash
# Generate HTML coverage report
pytest tests/ --cov=src --cov-report=html

# Open in browser
open htmlcov/index.html  # macOS
xdg-open htmlcov/index.html  # Linux

# View in terminal with missing lines
pytest tests/ --cov=src --cov-report=term-missing
```

Coverage reports show which lines are tested and which are not, helping identify gaps in test coverage.

## Common Anti-Patterns

### ❌ Mocking Internal Logic

```python
# WRONG
def test_bad():
    service = UserService(mock_repo)
    with patch.object(UserService, 'validate_email', return_value=True):
        service.create_user(email="test@example.com")  # ❌ Not testing real behavior

# RIGHT
def test_good():
    service = UserService(mock_repo)
    service.create_user(email="test@example.com")  # ✓ Tests real validation
```

### ❌ Mocking Domain Models

```python
# WRONG
mock_user = Mock(spec=User)  # ❌

# RIGHT
user = User(id="123", name="Alice")  # ✓
```

### ❌ Testing Implementation Details

```python
# WRONG
def test_bad():
    service.create_user("Alice")
    assert service._validate_name.called  # ❌ Internal implementation

# RIGHT
def test_good():
    user = service.create_user("Alice")
    assert user.name == "Alice"  # ✓ Observable behavior
```

### ❌ Not Using spec= Parameter

```python
# WRONG
mock_repo = Mock()
mock_repo.typo_method.return_value = "data"  # No error caught

# RIGHT
mock_repo = Mock(spec=UserRepository)
# mock_repo.typo_method raises AttributeError ✓
```

## Coverage Guidance

**Skip testing**:
- Trivial getters/setters
- Simple `__repr__` or `__str__`
- Framework boilerplate
- One-line defensive code

**Focus coverage on**:
- Business logic in services
- Complex conditionals and loops
- Error handling for important cases
- Integration points between services

## Test Organization

Tests should follow consistent organization conventions for maintainability. See the **python-code-style** skill for code organization conventions that also apply to test files.

### Directory Structure

Tests mirror the `src/` directory structure:

```
tests/
├── conftest.py                           # Shared fixtures
├── unit/
│   ├── domain/                           # Domain layer tests
│   │   ├── test_config.py
│   │   ├── test_models.py
│   │   └── test_exceptions.py
│   ├── infrastructure/                   # Infrastructure layer tests
│   │   ├── git/
│   │   │   └── test_operations.py
│   │   ├── github/
│   │   │   ├── test_operations.py
│   │   │   └── test_actions.py
│   │   └── filesystem/
│   │       └── test_operations.py
│   └── services/                         # Service layer tests
│       ├── formatters/
│       │   └── test_table_formatter.py
│       ├── test_entity_service.py
│       ├── test_workflow_service.py
│       └── test_statistics_service.py
├── integration/                          # Integration tests
│   └── cli/                              # CLI command integration tests
│       └── commands/
│           ├── test_prepare.py
│           ├── test_finalize.py
│           └── test_statistics.py
├── e2e/                                  # End-to-end tests
└── builders/                             # Test helpers/factories
```

### Naming Conventions

**Class Names**: `TestModuleName` or `TestFunctionName`
- ✅ `TestEntityService`
- ✅ `TestCheckUserCapacity`
- ❌ `ServiceTests` (wrong suffix)
- ❌ `TestServices` (too vague)

**Test Method Names**: `test_<what>_<when>_<condition>`
- ✅ `test_find_user_returns_none_when_all_at_capacity`
- ✅ `test_create_branch_raises_error_when_branch_exists`
- ✅ `test_parse_data_extracts_id_and_name`
- ❌ `test_user` (not descriptive)
- ❌ `test_find_user_works` (vague)

**Fixture Names**: `<resource>_<state>` or `mock_<service>`
- ✅ `user_config`, `sample_spec_file`, `mock_github_api`
- ❌ `data`, `setup`, `fixture1`

**File Names**: `test_<module_name>.py`
- ✅ `test_entity_service.py`
- ✅ `test_operations.py`
- ❌ `entity_tests.py`

## CI/CD Integration

### Automated Testing

Tests run automatically on:
- Every push to main branch
- Every pull request
- Via GitHub Actions workflow

### Test Workflow Example

```yaml
# .github/workflows/test.yml
name: Run Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install pytest pytest-cov
      - name: Run tests with coverage
        run: |
          pytest tests/unit/ tests/integration/ \
            --cov=src \
            --cov-report=term-missing \
            --cov-report=html \
            --cov-fail-under=70
      - name: Upload coverage report
        uses: actions/upload-artifact@v3
        with:
          name: coverage-report
          path: htmlcov/
```

### PR Requirements

- All tests must pass
- Coverage must meet minimum threshold (typically 70%)
- New features require tests
- Bug fixes should include regression tests

## Related Skills

- **creating-services**: For understanding service constructor patterns that you'll be testing
- **domain-modeling**: For understanding domain models and why not to mock them
- **python-code-style**: For code organization conventions that apply to test files
- **identifying-layer-placement**: For understanding which layer your code belongs to and how to test it

## Further Reading

- [Martin Fowler on Testing](https://martinfowler.com/testing/)
- [pytest Documentation](https://docs.pytest.org/)
- [unittest.mock Documentation](https://docs.python.org/3/library/unittest.mock.html)
