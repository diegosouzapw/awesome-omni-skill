---
name: testing-strategies
description: Testing strategies, patterns, and best practices for production code
license: MIT
compatibility: opencode
---

# Testing Strategies Skill

## Overview

This skill provides comprehensive guidelines for implementing effective testing strategies across unit, integration, and end-to-end testing.

## Testing Pyramid

```
       /\
      /  \  E2E Tests (10%)
     /____\  Slow, expensive, critical paths
    /      \
   /________\  Integration Tests (30%)
  /          \  Component interactions, APIs
 /____________\
/              \
Unit Tests (60%)  Fast, cheap, comprehensive
```

## Unit Testing

### 1. Principles

```go
// FIRST Principles:
// F - Fast (milliseconds)
// I - Isolated (no dependencies)
// R - Repeatable (same result every time)
// S - Self-verifying (no manual checking)
// T - Timely (write with code)

// AAA Pattern:
// Arrange - Set up test data and mocks
// Act     - Execute the code under test
// Assert  - Verify the results

func TestUserService_CreateUser(t *testing.T) {
    // Arrange
    mockRepo := &mockUserRepository{}
    mockRepo.On("Create", mock.Anything, mock.Anything).Return(nil)
    
    service := NewUserService(mockRepo)
    ctx := context.Background()
    req := CreateUserRequest{
        Email: "user@example.com",
        Name:  "John Doe",
    }
    
    // Act
    user, err := service.CreateUser(ctx, req)
    
    // Assert
    assert.NoError(t, err)
    assert.NotNil(t, user)
    assert.Equal(t, "user@example.com", user.Email)
    mockRepo.AssertExpectations(t)
}
```

### 2. Table-Driven Tests

```go
func TestCalculateDiscount(t *testing.T) {
    tests := []struct {
        name           string
        price          float64
        discountCode   string
        expectedPrice  float64
        expectedError  bool
    }{
        {
            name:          "valid 10% discount",
            price:         100.0,
            discountCode:  "SAVE10",
            expectedPrice: 90.0,
            expectedError: false,
        },
        {
            name:          "valid 20% discount",
            price:         100.0,
            discountCode:  "SAVE20",
            expectedPrice: 80.0,
            expectedError: false,
        },
        {
            name:          "invalid discount code",
            price:         100.0,
            discountCode:  "INVALID",
            expectedPrice: 0.0,
            expectedError: true,
        },
        {
            name:          "negative price",
            price:         -50.0,
            discountCode:  "SAVE10",
            expectedPrice: 0.0,
            expectedError: true,
        },
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result, err := CalculateDiscount(tt.price, tt.discountCode)
            
            if tt.expectedError {
                assert.Error(t, err)
            } else {
                assert.NoError(t, err)
                assert.InDelta(t, tt.expectedPrice, result, 0.01)
            }
        })
    }
}
```

### 3. Mocking

```go
// Interface for dependency injection
type EmailService interface {
    Send(ctx context.Context, to string, subject string, body string) error
}

// Mock implementation
type mockEmailService struct {
    mock.Mock
}

func (m *mockEmailService) Send(ctx context.Context, to string, subject string, body string) error {
    args := m.Called(ctx, to, subject, body)
    return args.Error(0)
}

// Test with mock
func TestNotificationService_NotifyUser(t *testing.T) {
    mockEmail := &mockEmailService{}
    mockEmail.On("Send", 
        mock.Anything, 
        "user@example.com",
        "Welcome!",
        mock.Anything,
    ).Return(nil)
    
    service := NewNotificationService(mockEmail)
    err := service.NotifyUser(context.Background(), "user@example.com", "Welcome!")
    
    assert.NoError(t, err)
    mockEmail.AssertCalled(t, "Send", 
        mock.Anything, 
        "user@example.com",
        "Welcome!",
        mock.Anything,
    )
}
```

## Integration Testing

### 1. Database Integration

```go
func TestUserRepository_Create(t *testing.T) {
    // Setup test database
    ctx := context.Background()
    db := setupTestDB(t)
    defer teardownTestDB(t, db)
    
    repo := NewUserRepository(db)
    
    // Test
    user := &User{
        Email: "test@example.com",
        Name:  "Test User",
    }
    
    err := repo.Create(ctx, user)
    require.NoError(t, err)
    assert.NotEmpty(t, user.ID)
    
    // Verify in database
    var count int
    err = db.QueryRowContext(ctx, "SELECT COUNT(*) FROM users WHERE email = $1", user.Email).Scan(&count)
    require.NoError(t, err)
    assert.Equal(t, 1, count)
}

// Test database setup
type testDB struct {
    *sql.DB
    name string
}

func setupTestDB(t *testing.T) *testDB {
    // Create isolated test database
    dbName := fmt.Sprintf("test_%s_%d", t.Name(), time.Now().Unix())
    
    connStr := fmt.Sprintf("host=localhost user=postgres password=secret dbname=%s sslmode=disable", dbName)
    db, err := sql.Open("postgres", connStr)
    require.NoError(t, err)
    
    // Run migrations
    runMigrations(db)
    
    return &testDB{DB: db, name: dbName}
}

func teardownTestDB(t *testing.T, db *testDB) {
    db.Close()
    // Drop test database
}
```

### 2. HTTP API Testing

```typescript
// API integration test
import request from 'supertest';
import { app } from '../src/app';
import { setupTestDB, teardownTestDB } from './helpers/database';

describe('POST /api/users', () => {
    beforeAll(async () => {
        await setupTestDB();
    });
    
    afterAll(async () => {
        await teardownTestDB();
    });
    
    it('should create a new user', async () => {
        const response = await request(app)
            .post('/api/users')
            .send({
                email: 'test@example.com',
                name: 'Test User'
            })
            .expect(201);
        
        expect(response.body).toMatchObject({
            email: 'test@example.com',
            name: 'Test User'
        });
        expect(response.body.id).toBeDefined();
    });
    
    it('should return 400 for invalid email', async () => {
        const response = await request(app)
            .post('/api/users')
            .send({
                email: 'invalid-email',
                name: 'Test User'
            })
            .expect(400);
        
        expect(response.body.error).toBeDefined();
    });
    
    it('should return 409 for duplicate email', async () => {
        // Create user first
        await request(app)
            .post('/api/users')
            .send({
                email: 'duplicate@example.com',
                name: 'Test User'
            });
        
        // Try to create again
        await request(app)
            .post('/api/users')
            .send({
                email: 'duplicate@example.com',
                name: 'Test User'
            })
            .expect(409);
    });
});
```

### 3. External Service Testing

```go
// Use test containers for external services
func TestWithRedis(t *testing.T) {
    ctx := context.Background()
    
    // Start Redis container
    req := testcontainers.ContainerRequest{
        Image:        "redis:latest",
        ExposedPorts: []string{"6379/tcp"},
        WaitingFor:   wait.ForListeningPort("6379/tcp"),
    }
    
    redisC, err := testcontainers.GenericContainer(ctx, testcontainers.GenericContainerRequest{
        ContainerRequest: req,
        Started:          true,
    })
    require.NoError(t, err)
    defer redisC.Terminate(ctx)
    
    // Connect to Redis
    endpoint, err := redisC.Endpoint(ctx, "")
    require.NoError(t, err)
    
    client := redis.NewClient(&redis.Options{
        Addr: endpoint,
    })
    
    // Test cache operations
    cache := NewCache(client)
    err = cache.Set(ctx, "key", "value", time.Hour)
    require.NoError(t, err)
    
    val, err := cache.Get(ctx, "key")
    require.NoError(t, err)
    assert.Equal(t, "value", val)
}
```

## End-to-End Testing

### 1. E2E with Playwright

```typescript
// e2e/user-journey.spec.ts
import { test, expect } from '@playwright/test';

test.describe('User Registration Flow', () => {
    test.beforeEach(async ({ page }) => {
        // Setup: Clean database, seed data if needed
        await page.goto('/register');
    });
    
    test('user can register successfully', async ({ page }) => {
        // Fill registration form
        await page.fill('[name="email"]', 'newuser@example.com');
        await page.fill('[name="password"]', 'SecurePass123!');
        await page.fill('[name="confirmPassword"]', 'SecurePass123!');
        
        // Submit form
        await page.click('button[type="submit"]');
        
        // Verify redirect to dashboard
        await expect(page).toHaveURL('/dashboard');
        await expect(page.locator('h1')).toContainText('Welcome');
        
        // Verify user is logged in
        await expect(page.locator('[data-testid="user-menu"]')).toBeVisible();
    });
    
    test('shows error for existing email', async ({ page }) => {
        await page.fill('[name="email"]', 'existing@example.com');
        await page.fill('[name="password"]', 'SecurePass123!');
        await page.click('button[type="submit"]');
        
        await expect(page.locator('[data-testid="error-message"]'))
            .toContainText('Email already exists');
    });
    
    test('validates password requirements', async ({ page }) => {
        await page.fill('[name="email"]', 'test@example.com');
        await page.fill('[name="password"]', 'weak');
        await page.click('button[type="submit"]');
        
        await expect(page.locator('[data-testid="password-error"]'))
            .toContainText('Password must be at least 8 characters');
    });
});
```

### 2. Critical Path Testing

```typescript
// Test only critical user journeys
const criticalPaths = [
    {
        name: 'Checkout Flow',
        steps: [
            'Add item to cart',
            'Proceed to checkout',
            'Fill shipping info',
            'Enter payment',
            'Confirm order'
        ]
    },
    {
        name: 'User Authentication',
        steps: [
            'Register new account',
            'Login',
            'Access protected resource',
            'Logout'
        ]
    }
];

// Run these in CI pipeline
```

## Test Organization

### 1. File Structure

```
project/
├── src/
│   └── user/
│       ├── service.go
│       ├── service_test.go         # Unit tests
│       └── service_integration_test.go  # Integration tests (build tag)
├── tests/
│   ├── integration/
│   │   ├── api/
│   │   │   └── user_api_test.go
│   │   └── database/
│   │       └── user_repo_test.go
│   └── e2e/
│       └── user-journey.spec.ts
└── testdata/
    └── fixtures/
        └── users.json
```

### 2. Test Tags

```go
// Unit tests (default)
// go test ./...

// Integration tests with build tag
//go:build integration
// +build integration

package user_test

func TestUserRepositoryIntegration(t *testing.T) {
    // Requires database
}

// Run: go test -tags=integration ./...

// E2E tests
//go:build e2e
// +build e2e

func TestUserJourney(t *testing.T) {
    // Full system test
}

// Run: go test -tags=e2e ./...
```

## Coverage and Metrics

### 1. Coverage Targets

```bash
# Overall coverage > 80%
go test -cover ./...

# Critical code coverage > 90%
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out -o coverage.html

# Coverage by package
go test -coverprofile=coverage.out ./...
grep -v "_test.go" coverage.out | awk -F: '{print $1}' | sort | uniq -c | sort -rn
```

### 2. Mutation Testing

```bash
# Go
# Install: go install github.com/zimmski/go-mutesting@latest
go-mutesting ./...

# JavaScript
# Install: npm install -g stryker-cli
stryker run
```

## Best Practices

### DO:
- ✅ Test behavior, not implementation
- ✅ Use table-driven tests for multiple cases
- ✅ Keep tests independent and isolated
- ✅ Use meaningful test names
- ✅ Test edge cases and error conditions
- ✅ Use test fixtures and factories
- ✅ Mock external dependencies
- ✅ Run tests in CI/CD pipeline

### DON'T:
- ❌ Test private methods (test public API)
- ❌ Share state between tests
- ❌ Use sleep() in tests (use async/await)
- ❌ Ignore flaky tests (fix them)
- ❌ Test everything (focus on critical paths)
- ❌ Write tests that depend on order
- ❌ Use production data in tests

## When to Use

Use this skill when:
- Writing unit tests
- Setting up integration tests
- Planning test strategy
- Reviewing test coverage
- Debugging test failures
- Setting up CI/CD testing
