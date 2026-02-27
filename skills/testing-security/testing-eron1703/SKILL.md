---
name: testing
description: Access testing configuration and delegate to test-rig tool for test generation and execution
user-invocable: true
---

# Testing Skill

**Purpose:** Multi-agent testing infrastructure for monoliths and microservices. Integrates test-rig CLI tool with Claude-based development workflows for rapid test creation and validation.

---

## Table of Contents

1. [Quick Start](#quick-start)
2. [Command Reference](#command-reference)
3. [TDD Workflow Integration](#tdd-workflow-integration)
4. [Supervisor Methodology](#supervisor-methodology)
5. [Parallel Testing Patterns](#parallel-testing-patterns)
6. [Common Scenarios](#common-scenarios)
7. [Troubleshooting](#troubleshooting)
8. [When to Use test-rig vs Manual Testing](#when-to-use-test-rig-vs-manual-testing)

---

## Quick Start

**test-rig** is globally installed at `/opt/homebrew/bin/test-rig`. Results return via stdout/stderr through the Bash tool.

```bash
# Navigate to your project directory and run test-rig commands via Bash
test-rig run
```

### Basic Workflow

```bash
# 1. Initialize test infrastructure
test-rig setup

# 2. Generate tests for a component
test-rig generate user-service

# 3. Run tests
test-rig run

# 4. Check coverage
test-rig coverage
```

---

## Command Reference

### `test-rig setup`

Initialize test infrastructure for the current project.

**Behavior:**
- Auto-detects project type (Node.js/Python)
- Installs appropriate test frameworks (Vitest, Pytest, Playwright)
- Creates folder structure (tests/unit, tests/integration, tests/e2e, tests/specs)
- Generates configuration files (test-rig.config.yaml)
- Sets up testcontainers for databases (PostgreSQL, Redis, MongoDB, etc.)

**Usage:**
```bash
test-rig setup
```

**Flags:**
- `--framework` - Override framework detection (vitest, pytest, playwright)
- `--skip-containers` - Skip testcontainer setup

**Example:**
```bash
test-rig setup --framework vitest
```

---

### `test-rig generate <component>`

Generate tests for a specific component or service.

**Behavior:**
- Analyzes component code structure and dependencies
- Creates component spec file (YAML) at `tests/specs/<component>.spec.yaml`
- Generates unit test file(s) at `tests/unit/<component>/`
- Generates integration test file(s) at `tests/integration/<component>/`
- Creates test data factories and fixtures
- Identifies testable units and dependencies

**Usage:**
```bash
test-rig generate user-service
test-rig generate payment-processor
```

**Flags:**
- `--type` - Specify component type (service, controller, utility, model)
- `--unit-only` - Generate unit tests only (skip integration)
- `--integration-only` - Generate integration tests only (skip unit)

**Example:**
```bash
test-rig generate auth-middleware --type controller
```

**Current Status:** Infrastructure complete. Some TODOs in test generation - consult generated specs for coverage gaps.

---

### `test-rig run [type]`

Run tests with flexible filtering and parallel execution.

**Usage:**
```bash
test-rig run                     # Run all tests
test-rig run unit               # Unit tests only
test-rig run integration        # Integration tests only
test-rig run e2e                # End-to-end tests only
test-rig run --parallel         # Multi-agent parallel execution
test-rig run unit --parallel    # Run unit tests in parallel
```

**Flags:**
- `--parallel` - Enable multi-agent parallel execution
- `--agents <n>` - Number of parallel agents (default: 4, max: 12)
- `--coverage` - Generate coverage report
- `--watch` - Watch mode (re-run on file changes)
- `--timeout <ms>` - Test timeout in milliseconds
- `--bail` - Stop on first test failure
- `--verbose` - Verbose output

**Examples:**
```bash
test-rig run --parallel --agents 8
test-rig run integration --coverage
test-rig run unit --watch
test-rig run --bail --verbose
```

---

### `test-rig coverage`

Generate and display code coverage report.

**Usage:**
```bash
test-rig coverage              # Generate coverage report
test-rig coverage --threshold 80  # Enforce minimum threshold
test-rig coverage --html       # Generate HTML report
```

**Flags:**
- `--threshold <n>` - Enforce minimum coverage percentage
- `--html` - Generate HTML report (output to coverage/)
- `--lcov` - Generate LCOV report

**Example:**
```bash
test-rig coverage --threshold 80 --html
```

---

### `test-rig analyze`

Analyze codebase for testability and identify gaps.

**Usage:**
```bash
test-rig analyze
```

**Outputs:**
- Untested modules
- Complex functions needing tests
- Integration points
- Dependency graphs

---

### `test-rig doctor`

Check test setup health and verify environment.

**Usage:**
```bash
test-rig doctor
```

**Checks:**
- Test framework installed and working
- Test files structure
- Testcontainers availability
- Configuration validity
- Coverage thresholds

---

### `test-rig --help` / `test-rig --version`

Display help information or version number.

```bash
test-rig --help
test-rig --version
```

---

## TDD Workflow Integration

Test-rig is designed to support **Red-Green-Refactor** cycles in collaborative development.

### Red Phase: Write Failing Tests

Use `test-rig generate` to scaffold tests based on component analysis:

```bash
# 1. Generate tests for new component
test-rig generate payment-service

# 2. View generated spec
cat tests/specs/payment-service.spec.yaml

# 3. Review generated unit tests
cat tests/unit/payment-service/

# 4. Run tests (should fail initially)
test-rig run unit
```

**Claude's Role:** Generate tests using component analysis, focusing on expected behavior and contracts.

### Green Phase: Implement to Pass

Implement the component to pass generated tests:

```bash
# 1. Edit component source code
# 2. Run tests in watch mode
test-rig run unit --watch

# 3. Tests pass when implementation is complete
```

**Claude's Role:** Generate implementation code that satisfies test contracts.

### Refactor Phase: Improve Without Breaking Tests

```bash
# 1. Refactor code (improve structure, performance, readability)
# 2. Run tests continuously to ensure nothing breaks
test-rig run unit --watch

# 3. Commit only when tests pass
test-rig run && git commit
```

**Claude's Role:** Suggest refactoring opportunities while monitoring test status.

### Full TDD Cycle Example

```bash
# Start fresh component
test-rig generate user-repository

# Run tests (red - should fail)
test-rig run unit

# Implement the component
# (edit src/repositories/user.ts)

# Run tests (green - should pass)
test-rig run unit

# Refactor and optimize
# (edit src/repositories/user.ts)

# Verify tests still pass
test-rig run unit

# Move to integration testing
test-rig generate user-service  # Generate integration tests
test-rig run integration
```

---

## Supervisor Methodology

When coding agents work together under supervisor guidance, test-rig enables coordination and verification.

### Agent Responsibilities

**Agent 1 (Feature Dev):** Generate tests and implementation
```bash
test-rig generate new-feature
test-rig run unit --parallel
```

**Agent 2 (Integration Dev):** Verify cross-service contracts
```bash
test-rig run integration --parallel
```

**Supervisor (Verification):** Check overall health
```bash
test-rig doctor
test-rig coverage --threshold 80
```

### Supervisor Workflow

1. **Initialize:** Setup shared test infrastructure
   ```bash
   test-rig setup --framework vitest
   ```

2. **Delegate:** Direct agents to specific components
   ```bash
   # Agent 1
   test-rig generate auth-service

   # Agent 2
   test-rig generate payment-service

   # Agent 3
   test-rig generate notification-service
   ```

3. **Monitor:** Check status in parallel
   ```bash
   test-rig doctor
   test-rig run --parallel --agents 4
   ```

4. **Validate:** Ensure coverage and quality gates
   ```bash
   test-rig coverage --threshold 80 --html
   test-rig analyze
   ```

5. **Coordinate:** Verify integration points
   ```bash
   test-rig run integration --parallel
   ```

### Benefits for Multi-Agent Teams

- **Parallelization:** Each agent works on isolated components, test-rig runs them in parallel
- **Dependency Tracking:** Component specs define dependencies, preventing conflicts
- **Fast Feedback:** Results return via stdout/stderr immediately
- **Scalability:** Supervisor can coordinate 4-12 agents without bottlenecks

---

## Parallel Testing Patterns

test-rig spawns multiple agents to run tests in parallel, achieving 3-4x faster execution.

### When to Use `--parallel`

Use parallel execution when:
- Running on CI/CD pipelines (maximize resource utilization)
- Testing large codebases with many independent components
- You need fast feedback cycles
- Multiple team members work simultaneously

Avoid parallel execution when:
- Debugging specific test failures (use sequential for clarity)
- Tests have uncontrolled shared state
- Running on resource-constrained systems

### Basic Parallel Execution

```bash
# Default: 4 agents
test-rig run --parallel

# Custom agent count
test-rig run --parallel --agents 8

# Run specific test type in parallel
test-rig run unit --parallel
test-rig run integration --parallel --agents 6
```

### Component Specs and Parallel Strategy

test-rig uses **component specs** (YAML) to organize tests for parallel execution:

```yaml
# tests/specs/user-service.spec.yaml
component:
  name: user-service
  type: service

subcomponents:
  - name: repository
    test_file: tests/unit/user-service/repository.spec.ts
    dependencies: [database]

  - name: validator
    test_file: tests/unit/user-service/validator.spec.ts
    dependencies: []

  - name: service
    test_file: tests/unit/user-service/service.spec.ts
    dependencies: [repository, validator]
```

**Each agent:**
1. Picks a component from the queue
2. Starts its own testcontainers
3. Runs component tests
4. Reports results
5. Picks the next component

### Parallel Execution with Integration Tests

```bash
# Run integration tests in parallel with testcontainer isolation
test-rig run integration --parallel --agents 4
```

**Isolation Mechanism:**
- Each agent gets its own testcontainer instances
- PostgreSQL runs on different ports (5432, 5433, 5434, 5435)
- Redis instances isolated by database index
- No test conflicts or race conditions

### Monitoring Parallel Execution

```bash
# Verbose mode shows per-agent progress
test-rig run --parallel --verbose

# Expected output:
# [Agent 1] Running component: user-service
# [Agent 2] Running component: payment-service
# [Agent 3] Running component: notification-service
# [Agent 1] ✓ user-service (1.2s)
# [Agent 2] ✗ payment-service (2.1s) - 3 failures
```

### Parallel Workflow in CI/CD

```yaml
# GitHub Actions example
name: Tests
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
      - run: npm install -g @hcb-consulting/test-rig
      - run: test-rig run --parallel --agents 4
```

---

## Common Scenarios

### Scenario 1: Setting Up New Project Tests

You're starting a new Node.js project and need comprehensive test infrastructure.

**Step 1: Initialize**
```bash
cd ~/projects/my-new-project
test-rig setup
```

**Step 2: Verify Structure**
```bash
# Check generated structure
ls -la tests/
# Output:
# ├── specs/
# ├── unit/
# ├── integration/
# ├── e2e/
# ├── factories/
# ├── fixtures/
# ├── mocks/
# └── utils/
```

**Step 3: Review Configuration**
```bash
cat test-rig.config.yaml
```

**Step 4: Test Execution**
```bash
test-rig run
```

**Example test-rig.config.yaml:**
```yaml
framework: vitest
parallel_agents: 4
containers:
  - postgres:5432
  - redis:6379
coverage_threshold:
  unit: 80
  integration: 60
```

---

### Scenario 2: Adding Tests for New Component

You've created a new `EmailService` component and need comprehensive tests.

**Step 1: Generate Tests**
```bash
test-rig generate email-service
```

**Step 2: Review Generated Spec**
```bash
cat tests/specs/email-service.spec.yaml
```

Expected output shows component structure and dependencies.

**Step 3: Review Generated Tests**
```bash
ls tests/unit/email-service/
# Output:
# ├── email-service.spec.ts
# ├── smtp-client.spec.ts
# ├── template-engine.spec.ts
# └── ...
```

**Step 4: Run and Verify**
```bash
# Run unit tests for this component
test-rig run unit

# View coverage
test-rig coverage
```

**Step 5: Add Integration Tests**
If integration tests weren't generated, create manually:

```bash
cat > tests/integration/email-service/delivery.spec.ts << 'EOF'
import { describe, it, expect, beforeAll, afterAll } from 'vitest';
import { EmailService } from '../../../src/services/email-service';

describe('EmailService Integration', () => {
  let emailService: EmailService;

  beforeAll(() => {
    emailService = new EmailService({
      smtpHost: process.env.SMTP_HOST || 'localhost',
      smtpPort: 1025,
    });
  });

  it('should send email through SMTP', async () => {
    const result = await emailService.send({
      to: 'test@example.com',
      subject: 'Test',
      body: 'Test email',
    });
    expect(result.success).toBe(true);
  });
});
EOF

# Run integration tests
test-rig run integration
```

---

### Scenario 3: Running Tests Before Commits

Ensure code quality and test passage before committing.

**Setup Pre-commit Hook:**
```bash
cat > .git/hooks/pre-commit << 'EOF'
#!/bin/bash
set -e

echo "Running tests before commit..."
test-rig run --bail

if [ $? -eq 0 ]; then
  echo "✓ All tests passed"
else
  echo "✗ Tests failed - commit aborted"
  exit 1
fi
EOF

chmod +x .git/hooks/pre-commit
```

**Manual Pre-commit Workflow:**
```bash
# 1. Run all tests
test-rig run

# 2. If passing, check coverage
test-rig coverage

# 3. If coverage is acceptable, commit
git add .
git commit -m "Add feature X"
```

**Fast Pre-commit (Unit Tests Only):**
```bash
# For rapid feedback during development
test-rig run unit --bail

# Only run full suite before final commit
test-rig run
```

---

### Scenario 4: CI/CD Integration Patterns

#### GitHub Actions

```yaml
# .github/workflows/test.yml
name: Test Suite

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:14
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v3

      - name: Setup Node
        uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Install test-rig
        run: npm install -g @hcb-consulting/test-rig

      - name: Run tests in parallel
        run: test-rig run --parallel --agents 4

      - name: Generate coverage report
        run: test-rig coverage --html

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
```

#### GitLab CI

```yaml
# .gitlab-ci.yml
stages:
  - test

test:
  image: node:18
  services:
    - postgres:14
  before_script:
    - npm install -g @hcb-consulting/test-rig
  script:
    - test-rig run --parallel --agents 4
  coverage: '/coverage: \d+\.\d+%/'
```

#### Pre-deployment Checks

```bash
#!/bin/bash
# scripts/pre-deploy.sh

set -e

echo "Running pre-deployment tests..."

# 1. Run all tests
echo "Running test suite..."
test-rig run --parallel

# 2. Verify coverage thresholds
echo "Checking coverage..."
test-rig coverage --threshold 80

# 3. Analyze for issues
echo "Analyzing codebase..."
test-rig analyze

# 4. Check system health
echo "System health check..."
test-rig doctor

echo "✓ All pre-deployment checks passed"
```

---

## Troubleshooting

### Issue: "test-rig: command not found"

**Cause:** test-rig is not globally installed or not in PATH.

**Solution:**
```bash
# Check if installed
which test-rig

# Install globally
npm install -g @hcb-consulting/test-rig

# Or use full path
/opt/homebrew/bin/test-rig run
```

---

### Issue: "Port already in use" during parallel tests

**Cause:** Testcontainers are using ports that are already allocated.

**Solution:**
```bash
# Kill existing containers
docker ps -a | grep test-rig | awk '{print $1}' | xargs docker rm -f

# Run tests again
test-rig run --parallel
```

---

### Issue: Tests timeout during integration tests

**Cause:** Testcontainers taking too long to start or network latency.

**Solution:**
```bash
# Increase timeout
test-rig run integration --timeout 30000

# Or reduce parallel agents to free resources
test-rig run integration --parallel --agents 2
```

---

### Issue: Coverage threshold not met

**Cause:** Insufficient test coverage for the codebase.

**Solution:**
```bash
# 1. Analyze which code is untested
test-rig analyze

# 2. Generate tests for low-coverage components
test-rig generate low-coverage-component

# 3. Run and verify
test-rig run --parallel
test-rig coverage --html

# 4. Open HTML report
open coverage/index.html
```

---

### Issue: Component spec has TODOs or gaps

**Cause:** test-rig's test generation is incomplete for complex components.

**Solution:**
```bash
# 1. Review generated spec
cat tests/specs/component-name.spec.yaml

# 2. Manually add missing subcomponents
# Edit the spec to include missing testable units

# 3. Generate additional tests
# Manually create test files for uncovered areas

# 4. Run and verify
test-rig run
```

**Example Manual Spec Addition:**
```yaml
# tests/specs/user-service.spec.yaml
component:
  name: user-service
  type: service

subcomponents:
  - name: repository
    test_file: tests/unit/user-service/repository.spec.ts
    dependencies: [database]

  # TODO: Add validator subcomponent
  # - name: validator
  #   test_file: tests/unit/user-service/validator.spec.ts
  #   dependencies: []

  # Manually add missing subcomponent
  - name: auth-middleware
    test_file: tests/unit/user-service/auth-middleware.spec.ts
    dependencies: [jwt]
```

---

### Issue: Tests pass locally but fail in CI

**Cause:** Environment differences (database versions, missing services, timing).

**Solution:**
```bash
# 1. Check system health
test-rig doctor

# 2. Run with verbose output to see environment
test-rig run --verbose

# 3. Check CI environment matches local setup
# Verify test-rig.config.yaml container versions

# 4. Run tests with longer timeouts in CI
test-rig run --timeout 30000 --parallel --agents 4
```

---

## When to Use test-rig vs Manual Testing

### Use test-rig When:

- **Creating new components** - Generate comprehensive test suites quickly
- **Large codebases** - Parallel execution saves significant time
- **CI/CD pipelines** - Automated, reproducible testing
- **TDD workflows** - Generate tests first, implement second
- **Integration testing** - Testcontainers handle complex setups
- **Coverage tracking** - Automated coverage reports and thresholds
- **Multi-team development** - Coordinate testing across agents

### Use Manual Testing When:

- **Debugging specific failures** - Manual test run with debugger
- **Exploratory testing** - Discover edge cases interactively
- **UI/UX testing** - Manual interaction is essential
- **Rapid prototyping** - Quick feedback without full test suite
- **One-off verification** - Single test instead of full suite

### Hybrid Approach (Recommended)

```bash
# Development: Use test-rig for structure and automation
test-rig generate new-feature
test-rig run unit --watch

# Debugging: Drop into manual testing temporarily
# Run specific test file, use debugger
npm test -- tests/unit/problematic-component.spec.ts --inspect

# Pre-commit: Return to test-rig for comprehensive validation
test-rig run --bail

# CI/CD: test-rig for reproducible, parallel execution
test-rig run --parallel --agents 4
```

---

## Configuration

Create `test-rig.config.yaml` in your project root:

```yaml
framework: vitest           # vitest, pytest, or playwright
parallel_agents: 4
containers:
  - postgres:5432
  - redis:6379
  - mongodb:27017
coverage_threshold:
  unit: 80
  integration: 60
  overall: 75
timeouts:
  unit: 10000              # milliseconds
  integration: 30000
  e2e: 60000
```

---

## Project Mappings

Configured test infrastructure for known projects:

- **resolver:** vitest, 4 agents, postgres+arangodb+redis
- **commander:** vitest, 8 agents, microservices
- **flowmaster:** vitest+pytest, 12 agents, large microservices
- **dxg:** pytest, 4 agents, postgres+redis
- **engage:** vitest, 6 agents, MCP testing
- **sdx:** vitest, 4 agents, contract testing

---

## Full Documentation

Detailed documentation and examples available at:
- **README:** ~/projects/test-rig/README.md
- **GitHub:** https://github.com/HCB-Consulting-ME/test-rig
- **Issues:** https://github.com/HCB-Consulting-ME/test-rig/issues

---

**Note:** test-rig provides the infrastructure. Claude handles component analysis, test generation, and implementation logic through the Bash tool. Results come back via stdout/stderr immediately.
