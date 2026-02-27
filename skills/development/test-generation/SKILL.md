---
name: test-generation
description: Master skill for intelligent test case generation. Identifies technology stack and delegates to specialized testing sub-skills for creating high-quality integration and API tests.
---

# Test Generation Framework

## Instructions
You are the entry point for test case generation and quality assurance. Your goal is to **Identify** the technology stack, **Prioritize** testable components, **Generate** comprehensive tests, and **Document** the testing strategy.

## Core Test Generation Process

### Phase 1: Discovery & Analysis
1. **Technology Stack Detection**: Identify languages, frameworks, and testing tools
2. **Component Mapping**: Enumerate all testable endpoints (APIs, services, workflows)
3. **Dependency Review**: Check existing test frameworks and libraries
4. **Coverage Analysis**: Assess current test coverage if tests exist

### Phase 2: Test Scope Prioritization

#### A. Components to ALWAYS Test
- **API Endpoints**: All HTTP routes (GET, POST, PUT, PATCH, DELETE)
- **Service Layer Functions**: Business logic orchestration and data processing
- **Main Execution Functions**: Functions that call multiple sub-functions
- **Database Operations**: CRUD operations, queries, transactions
- **Authentication & Authorization**: Login flows, token validation, permissions
- **Integration Points**: External APIs, message queues, webhooks
- **Critical Workflows**: Payment processing, order fulfillment, user registration

#### B. Components to AVOID Testing
- **Simple Utility Functions**: String formatters, basic validators (unless complex)
- **Framework Generated Code**: ORM models, auto-generated getters/setters
- **Third-Party Libraries**: Already tested by maintainers
- **Pure Configuration**: JSON/YAML without logic

### Phase 3: Test Type Selection

#### A. Integration Tests (Primary Focus - 70%)
**Concept**: Test how components work together in realistic scenarios.
**Action**:
1. **API Integration**: Test endpoints with database interactions
2. **Service Integration**: Test services with real/mocked external dependencies
3. **Database Integration**: Use test containers or in-memory databases
4. **End-to-End Workflows**: Multi-step business processes

#### B. Unit Tests (Secondary Focus - 20%)
**Concept**: Test isolated, complex business logic only.
**Action**:
1. **Complex Algorithms**: Functions with intricate logic and edge cases
2. **Business Rules**: State machines, validation engines
3. **Calculations**: Financial calculations, data transformations

#### C. Contract Tests (10%)
**Concept**: Ensure API stability and compatibility.
**Action**:
1. **Schema Validation**: Request/response structure verification
2. **API Versioning**: Backward compatibility checks
3. **Integration Contracts**: External API mock validation

### Phase 4: Test Generation

#### Test Structure Standards
All generated tests must follow the **AAA (Arrange-Act-Assert)** pattern:

```
describe('Component/Function Name', () => {
  it('should [expected behavior] when [condition]', () => {
    // Arrange: Set up test data and dependencies
    const testData = createTestData();
    const mockDependency = mockService();
    
    // Act: Execute the function under test
    const result = functionUnderTest(testData);
    
    // Assert: Verify expected outcomes
    expect(result).toMatchExpectedValue();
    expect(mockDependency).toHaveBeenCalledWith(expectedArgs);
  });
});
```

#### Coverage Requirements
For each testable component, generate tests that cover:
1. **Happy Path**: Normal, expected flow
2. **Edge Cases**: Boundary values, empty inputs, null handling
3. **Error Scenarios**: Invalid inputs, network failures, timeout handling
4. **Security Cases**: Authentication failures, authorization violations, injection attempts

#### Test Data Management
1. **Use Factory Functions**: Generate realistic test data programmatically
2. **Create Fixtures**: Define reusable test datasets
3. **Avoid Hardcoding**: Use constants and data generators
4. **Realistic Data**: Mirror production data patterns

### Phase 5: Documentation Generation

#### Test Report Structure
Generate comprehensive test documentation in `test-reports/` folder:

```
test-reports/
├── index.md                    # Test coverage summary
├── test-plan.md                # Testing strategy and scope
├── integration-tests/
│   ├── api-tests.md           # API endpoint test docs
│   ├── service-tests.md       # Service layer test docs
│   └── workflow-tests.md      # E2E workflow test docs
├── coverage/
│   ├── coverage-summary.md    # Code coverage metrics
│   └── gaps-analysis.md       # Coverage gaps
└── execution/
    ├── setup-guide.md         # How to run tests
    └── ci-integration.md      # CI/CD pipeline guide
```

## Technology-Specific Testing Patterns

### Node.js / JavaScript Testing
**Focus Areas**: API testing, async operations, service integration
**Key Frameworks**: Jest, Vitest, Mocha + Chai, Supertest
**Patterns**:
- API endpoint testing with Supertest
- Service layer mocking with Jest
- Database testing with mongodb-memory-server
- Async/await test patterns
*Refer to [node_testing.md](node_testing.md) for detailed patterns.*

### Python Testing
**Focus Areas**: API testing, fixtures, parametrized tests
**Key Frameworks**: Pytest, unittest, FastAPI TestClient
**Patterns**:
- FastAPI/Django/Flask API testing
- Pytest fixtures for setup/teardown
- Parametrized tests for similar scenarios
- SQLAlchemy integration testing
*Refer to [python_testing.md](python_testing.md) for detailed patterns.*

### Java / Kotlin Testing
**Focus Areas**: Spring Boot testing, REST API testing, JPA integration
**Key Frameworks**: JUnit 5, Mockito, Spring Boot Test, RestAssured
**Patterns**:
- @SpringBootTest for integration tests
- MockMvc for API testing
- TestContainers for database testing
- JPA repository testing
*Refer to [java_testing.md](java_testing.md) for detailed patterns.*

### .NET / C# Testing
**Focus Areas**: ASP.NET Core testing, Entity Framework, API integration
**Key Frameworks**: xUnit, NUnit, MSTest, FluentAssertions, Moq
**Patterns**:
- WebApplicationFactory for API tests
- In-memory databases for EF Core
- Mocking with Moq
- Integration test slices
*Refer to [dotnet_testing.md](dotnet_testing.md) for detailed patterns.*

### Go Testing
**Focus Areas**: HTTP handlers, table-driven tests, interface testing
**Key Frameworks**: testing package, testify, httptest
**Patterns**:
- Table-driven test patterns
- httptest for HTTP handler testing
- Interface mocking
- Subtests for organized test cases
*Refer to [go_testing.md](go_testing.md) for detailed patterns.*

### PHP Testing
**Focus Areas**: Laravel/Symfony testing, database factories, feature tests
**Key Frameworks**: PHPUnit, Pest, Laravel Dusk
**Patterns**:
- Laravel HTTP tests
- Database factories and seeders
- Feature vs unit tests
- Browser testing with Dusk
*Refer to [php_testing.md](php_testing.md) for detailed patterns.*

### Rust Testing
**Focus Areas**: Unit tests, integration tests, async testing
**Key Frameworks**: Built-in test framework, mockall, tokio-test
**Patterns**:
- Module-level unit tests
- Integration test directory structure
- Async test patterns with tokio
- Mock traits with mockall
*Refer to [rust_testing.md](rust_testing.md) for detailed patterns.*

### React / Frontend Testing
**Focus Areas**: Component testing, user interaction, integration tests
**Key Frameworks**: Vitest, Jest, React Testing Library, Cypress
**Patterns**:
- Component rendering tests
- User event simulation
- API mocking with MSW
- E2E tests with Cypress
*Refer to [react_testing.md](react_testing.md) for detailed patterns.*

### Vue.js Testing
**Focus Areas**: Component testing, Vuex, composables testing
**Key Frameworks**: Vitest, Vue Test Utils, Cypress
**Patterns**:
- Component mounting and testing
- Vuex store testing
- Composables testing
- Router testing
*Refer to [vue_testing.md](vue_testing.md) for detailed patterns.*

### Next.js Testing
**Focus Areas**: API routes, page components, SSR/SSG testing
**Key Frameworks**: Jest, Vitest, Playwright, Cypress
**Patterns**:
- API route testing
- Page component testing
- Server-side rendering tests
- E2E testing with Playwright
*Refer to [next_testing.md](next_testing.md) for detailed patterns.*

### NestJS Testing
**Focus Areas**: Controller testing, service testing, E2E testing
**Key Frameworks**: Jest, Supertest, @nestjs/testing
**Patterns**:
- Unit tests with TestingModule
- E2E tests with Supertest
- Mocking providers
- Database integration tests
*Refer to [nest_testing.md](nest_testing.md) for detailed patterns.*

### React Native Testing
**Focus Areas**: Component testing, navigation, native module mocking
**Key Frameworks**: Jest, React Native Testing Library, Detox
**Patterns**:
- Component testing
- Navigation testing
- Native module mocking
- E2E with Detox
*Refer to [react_native_testing.md](react_native_testing.md) for detailed patterns.*

## Test Quality Standards

### Readability
- ✅ Clear, descriptive test names: `should reject payment when card is expired`
- ✅ Self-documenting: Test describes the behavior being verified
- ✅ Minimal logic: Tests should be straightforward, not complex
- ✅ Follow AAA pattern: Arrange, Act, Assert

### Isolation
- ✅ Independent tests: Each test can run alone or in any order
- ✅ Clean state: Reset databases, clear caches between tests
- ✅ No shared state: Avoid global variables or test interdependencies
- ✅ Parallel execution: Tests can run concurrently

### Performance
- ✅ Fast execution: Integration tests should complete in seconds
- ✅ Parallel ready: Design tests to run concurrently
- ✅ Optimize setup: Use beforeAll/beforeEach wisely
- ✅ Clean teardown: Proper cleanup after tests

### Maintainability
- ✅ DRY principle: Extract common setup into helpers
- ✅ Consistent patterns: Use the same approach across codebase
- ✅ Version controlled: Treat test code like production code
- ✅ Well documented: Comment complex test scenarios

## Mocking Strategy

### When to Mock
**MOCK** external dependencies:
- External APIs and third-party services
- Email/SMS services
- Payment gateways
- File systems (in unit tests)
- Time/date functions

**DON'T MOCK** for integration tests:
- Databases (use test containers instead)
- Internal services (test real interactions)
- Simple utilities
- Framework functions

### Mocking Patterns
```javascript
// Node.js - Jest mocking
jest.mock('external-service');
const mockService = require('external-service');
mockService.someMethod.mockResolvedValue({ data: 'test' });

// Python - Pytest fixtures
@pytest.fixture
def mock_external_api(mocker):
    return mocker.patch('app.services.external_api.call')

// Java - Mockito
@Mock
private ExternalService externalService;
when(externalService.getData()).thenReturn(testData);

// .NET - Moq
var mockService = new Mock<IExternalService>();
mockService.Setup(x => x.GetData()).Returns(testData);
```

## Output Format

### Generated Test File Template
```language
// tests/integration/api/users.test.{ext}

// Imports for test framework and utilities
import { test framework imports };
import { application imports };
import { test helpers and fixtures };

// Describe test suite
describe('API: User Endpoints', () => {
  // Setup/teardown hooks
  beforeAll(async () => {
    // Initialize test database, server, etc.
  });
  
  afterAll(async () => {
    // Cleanup resources
  });
  
  beforeEach(async () => {
    // Reset state before each test
  });
  
  // Test cases following AAA pattern
  describe('POST /api/users', () => {
    it('should create user with valid data', async () => {
      // Arrange
      const userData = createValidUserData();
      
      // Act
      const response = await makeRequest('POST', '/api/users', userData);
      
      // Assert
      expect(response.status).toBe(201);
      expect(response.body).toMatchObject({
        id: expect.any(String),
        email: userData.email
      });
    });
    
    it('should reject invalid email format', async () => {
      // Arrange
      const invalidData = { ...validData, email: 'invalid' };
      
      // Act
      const response = await makeRequest('POST', '/api/users', invalidData);
      
      // Assert
      expect(response.status).toBe(400);
      expect(response.body.error).toContain('email');
    });
  });
});
```

## Best Practices

### Test Organization
- Group related tests using describe/context blocks
- Mirror source code directory structure
- Use clear, descriptive file names
- Separate unit tests from integration tests

### Test Data
- Use factory functions for test data creation
- Create reusable fixtures
- Use realistic data that mimics production
- Clean up test data after tests

### Assertions
- Use specific assertions (toBe, toEqual, toContain)
- Assert on behavior, not implementation
- Include negative assertions where appropriate
- Check both success and error cases

### Error Handling
- Test error scenarios explicitly
- Verify error messages and status codes
- Test exception handling
- Check error recovery mechanisms

## References
For detailed, language-specific testing patterns and examples, refer to the individual skill files in this directory.
