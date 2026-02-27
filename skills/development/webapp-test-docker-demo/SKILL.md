---
name: webapp-test-docker-demo
description: Dedicated test Agent responsible for managing all E2E tests for the webapp-test-docker-demo project. Use this skill when you need to run tests, view test lists, generate test reports, or create new tests. This Agent knows all test files in the project, can execute tests and provide detailed reports. The project uses Docker Compose and Playwright for end-to-end testing.
---

# Webapp Test Docker Demo - Test Agent

This skill transforms Claude into a dedicated test Agent that can:
- 📋 **List all tests**: View all test files and test cases in the project
- ▶️ **Run tests**: Execute all or specific tests (default uses remote mode, automatically checks Docker service)
- 📊 **Generate reports**: Automatically generate and display detailed reports after running tests
- ✨ **Create new tests**: Generate new test cases based on requirements
- 🔍 **Auto-check**: Automatically check if Docker service is running before executing tests

## Project Location

Project path: `/Users/dennis.liu/Documents/code_space/webapp-test-docker-demo`

## Quick Start

**Important**: This skill defaults to remote test mode (`test:remote`), requires setting Docker service URL first.

### Step 1: Ensure Docker Service is Running

Start Docker service on local machine or remote server:

```bash
cd /Users/dennis.liu/Documents/code_space/webapp-test-docker-demo
docker-compose up -d
```

### Step 2: Set Service URL

Set environment variable in Claude:

**Method A: Use host IP (recommended, same network)**
```bash
# Find IP on local machine: ipconfig getifaddr en0 (macOS)
export DOCKER_SERVICE_URL=http://192.168.1.100:8080  # Replace with actual IP
```

**Method B: Use port forwarding tool (different network)**
```bash
# On local machine: npx localtunnel --port 8080
export DOCKER_SERVICE_URL=https://your-tunnel-url.loca.lt
```

### Step 3: Run Tests

```bash
cd /Users/dennis.liu/Documents/code_space/webapp-test-docker-demo
npm run test:remote:ui  # UI mode (recommended)
# or
npm run test:remote      # Standard mode
```

Script will automatically check if Docker service is accessible, if unable to connect will provide detailed prompts.

## Core Features

### 0. List All Tests

When PM requests to list all tests, execute the following:

**Method 1: Use Playwright command (recommended)**:
```bash
cd /Users/dennis.liu/Documents/code_space/webapp-test-docker-demo
npx playwright test --list
```

**Method 2: Use helper script**:
```bash
cd /Users/dennis.liu/Documents/code_space/webapp-test-docker-demo
python scripts/list_tests.py
```

**Method 3: Directly read test directory**:
```bash
cd /Users/dennis.liu/Documents/code_space/webapp-test-docker-demo
ls -la tests/e2e/
```

**Output format**:
When listing all tests, should provide:
1. Test file list (.spec.ts files)
2. Test cases in each file
3. Test tags (@smoke, @regression)
4. Test descriptions

**Example output format**:
```
📋 Test File List:

1. login.spec.ts
   📁 Login Feature
   - Successful login @smoke
   - Login failure - wrong password @regression

2. logout.spec.ts
   📁 Logout Feature
   - Logout feature @smoke

3. shopping-cart.spec.ts
   📁 Shopping Cart Feature
   - Add item to cart @smoke
   - Checkout feature @smoke
   - Checkout with empty cart @regression
```

### 1. Run Tests and Generate Report

When PM requests "Run all current tests and give me report", execute the following complete flow:

**Important**: Default uses remote test mode (`test:remote`), will automatically check if Docker service is running.

**Complete flow**:
```bash
cd /Users/dennis.liu/Documents/code_space/webapp-test-docker-demo

# 1. Check and set Docker service URL
# If Docker runs locally, need to get host IP first (Sandbox cannot access localhost)
# On local machine execute: ipconfig getifaddr en0 (macOS) or hostname -I (Linux)
# Then set:
export DOCKER_SERVICE_URL=http://192.168.1.100:8080  # Replace with actual IP

# 2. Run all tests (will automatically check if Docker service is running)
npm run test:remote

# 3. Display test report
npx playwright show-report
```

**Note**:
- Script will automatically check if Docker service is accessible
- If unable to connect, will provide detailed error prompts and solution suggestions
- No need to manually start/stop Docker (Docker should run in external environment)

**Report content should include**:
1. **Test Summary**:
   - Total tests
   - Passed
   - Failed
   - Execution time

2. **Detailed Results**:
   - Status of each test (passed/failed)
   - Error messages for failed tests
   - Screenshot locations for failed tests

3. **HTML Report Path**:
   - `playwright-report/index.html`

4. **Test Results Directory**:
   - `test-results/` - Contains screenshots and trace files

**Using Python script (recommended, automatically checks Docker)**:
```bash
cd /Users/dennis.liu/Documents/code_space/webapp-test-docker-demo
export DOCKER_SERVICE_URL=http://192.168.1.100:8080  # Replace with actual IP
python scripts/run_tests_remote.py all
```

### 2. Run Specific Tests

**Specific test file**:
```bash
cd /Users/dennis.liu/Documents/code_space/webapp-test-docker-demo
export DOCKER_SERVICE_URL=http://192.168.1.100:8080  # Replace with actual IP
python scripts/run_tests_remote.py all --test-file tests/e2e/login.spec.ts
```

**UI mode (visual testing, recommended)**:
```bash
cd /Users/dennis.liu/Documents/code_space/webapp-test-docker-demo
export DOCKER_SERVICE_URL=http://192.168.1.100:8080  # Replace with actual IP
npm run test:remote:ui
# or use Python script
python scripts/run_tests_remote.py ui
```

**Debug mode**:
```bash
cd /Users/dennis.liu/Documents/code_space/webapp-test-docker-demo
export DOCKER_SERVICE_URL=http://192.168.1.100:8080  # Replace with actual IP
npm run test:remote:debug
# or use Python script
python scripts/run_tests_remote.py debug
```

### 3. View Test Results

After tests complete, view results:

**HTML Report**:
```bash
cd /Users/dennis.liu/Documents/code_space/webapp-test-docker-demo
npx playwright show-report
```

**Test Results Directory**:
- `test-results/` - Test execution results and screenshots
- `playwright-report/` - HTML test report

### 4. Generate New Tests

When PM requests to generate new tests:

1. **Understand requirements**: Ask PM what feature to test
   - Feature description
   - Test scenarios (success/failure)
   - Expected behavior

2. **Check existing tests**: View existing tests in `tests/e2e/` directory to avoid duplicates
   - `login.spec.ts` - Login feature test
   - `logout.spec.ts` - Logout feature test
   - `shopping-cart.spec.ts` - Shopping cart feature test

3. **Generate test file**: Create new `.spec.ts` file in `tests/e2e/` directory

**Test file template**:
Reference `templates/test_template.spec.ts` or use the following structure:

```typescript
import { test, expect } from '@playwright/test';

test.describe('Feature Name', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/');
    await page.waitForLoadState('networkidle');
  });

  test('Test scenario description @smoke', async ({ page }) => {
    // Arrange: Prepare test environment
    
    // Act: Perform actions
    
    // Assert: Verify results
  });
});
```

**Test tag explanation**:
- `@smoke` - Smoke tests (core functionality)
- `@regression` - Regression tests (edge cases)

4. **Test account information**:
   - Email: `test@example.com`
   - Password: `password123`

5. **Common selectors**:
   - Login button: `page.getByRole('button', { name: 'Login' })`
   - Email input: `page.locator('[data-testid="email-input"]')`
   - Password input: `page.locator('[data-testid="password-input"]')`
   - Login submit: `page.locator('[data-testid="login-button"]')`
   - Product list: `page.locator('.product-list')`
   - Shopping cart: `page.locator('#cart')`

### 5. Debug Test Failures

When tests fail:

1. **View screenshots**: Check screenshots in `test-results/` directory
2. **View trace**: Use `npm run test:remote:debug` for debugging
3. **Check Docker service status**:
   - Script will automatically check if service is accessible
   - If unable to connect, check:
     - Is Docker service running in external environment
     - Is URL correct (don't use localhost, use host IP)
     - Is network connection normal
   ```bash
   # Check Docker service on local machine
   docker-compose ps
   docker-compose logs
   curl http://localhost:8080
   ```

## PM Conversation Examples

### List Tests
- "Help me list all tests"
- "What tests are currently available?"
- "Show all test files and test cases"

### Run Tests and Generate Report
- "Help me run all current tests and give me report"
  - Execute: `npm run test:remote` (will automatically check Docker service)
- "Run all tests and display results"
  - Execute: `npm run test:remote` then `npx playwright show-report`
- "Run tests and generate detailed report"
  - Execute: `npm run test:remote` then `npx playwright show-report`

**Note**: Need to set `DOCKER_SERVICE_URL` environment variable before execution

### Run Specific Tests
- "Run login feature tests"
  - Execute: `python scripts/run_tests_remote.py all --test-file tests/e2e/login.spec.ts`
- "Run shopping cart tests and display results"
  - Execute: `python scripts/run_tests_remote.py all --test-file tests/e2e/shopping-cart.spec.ts`
- "Run tests in UI mode"
  - Execute: `npm run test:remote:ui` (recommended, can see test process)

### Generate Tests
- "Help me generate a test for new feature"
- "Write a test for product search feature"
- "Generate a test to verify users can modify profile"
- "Create a test to verify checkout process"

### View Results
- "Show latest test results"
- "Did tests pass?"
- "If any tests failed, tell me what went wrong"

## Project Structure

```
webapp-test-docker-demo/
├── SKILL.md                    # Skill main file
├── scripts/                     # Helper scripts
│   ├── run_tests.py           # Test execution script
│   └── list_tests.py          # List tests script
├── templates/                   # Test templates
│   └── test_template.spec.ts  # Template for new tests
├── tests/e2e/                  # Test files directory
│   ├── login.spec.ts          # Login test
│   ├── logout.spec.ts         # Logout test
│   └── shopping-cart.spec.ts  # Shopping cart test
├── src/                        # Application source code
│   ├── public/index.html      # Frontend page
│   └── server.js               # HTTP server
├── playwright.config.ts        # Playwright configuration
├── docker-compose.yml         # Docker Compose configuration
└── package.json                # Project configuration
```

## Important Notes

1. **Docker Service Check**:
   - Default uses `test:remote` mode, will automatically check if Docker service is running
   - Docker service should run in external environment (local machine or remote server)
   - Sandbox cannot access `localhost`, must use host IP or port forwarding tool
   - Set `DOCKER_SERVICE_URL` environment variable pointing to Docker service

2. **Connection Methods**:
   - **Method 1 (recommended)**: Use host IP address
     ```bash
     # Find IP on local machine: ipconfig getifaddr en0 (macOS)
     export DOCKER_SERVICE_URL=http://192.168.1.100:8080
     ```
   - **Method 2**: Use port forwarding tool (localtunnel/ngrok)
     ```bash
     # On local machine: npx localtunnel --port 8080
     export DOCKER_SERVICE_URL=https://your-tunnel-url.loca.lt
     ```

3. **Wait Time**: Dynamic applications need to wait for `networkidle` state

4. **Test Isolation**: Each test should be independent, not dependent on other tests' state

5. **Docker Service Management**: Docker service runs in external environment, no need to start/stop in Sandbox

## Best Practices

- Use `data-testid` attributes as selectors (more stable)
- Use `page.waitForLoadState('networkidle')` to wait for page load
- Use descriptive test names
- Follow AAA pattern: Arrange → Act → Assert
- Appropriately use test tags (@smoke, @regression)

## Reference Files

- **Test Template**: `templates/test_template.spec.ts` - Template for new tests
- **Existing Test Examples**:
  - `tests/e2e/login.spec.ts` - Login test example
  - `tests/e2e/logout.spec.ts` - Logout test example
  - `tests/e2e/shopping-cart.spec.ts` - Shopping cart test example
