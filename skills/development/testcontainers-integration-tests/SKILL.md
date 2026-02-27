---
name: testcontainers-integration-tests
description: Use when integration tests require real infrastructure (database, message queue, cache) or when mocking infrastructure is insufficient. Defines container lifecycle, test isolation, and performance optimization for Testcontainers-based testing.
metadata:
  type: Implementation
  priority: P1
---

# Testcontainers Integration Tests

## Overview

Use **real infrastructure in containers** for integration tests. Mocks don't catch SQL bugs
or database-specific issues. Testcontainers provides isolated, repeatable tests with
production-equivalent infrastructure.

**REQUIRED:** superpowers:test-driven-development, superpowers:verification-before-completion

## When to Use

- Integration tests need real database, queue, or cache
- Tests verify database-specific features (JSON, arrays, transactions)
- Mocking infrastructure is insufficient for test confidence
- Tests are flaky due to shared test database state
- Repository uses or should use Testcontainers

## Detection and Deference

Before creating new Testcontainers setup, check for existing patterns:

```bash
# Check for existing Testcontainers packages (.NET)
grep -r "Testcontainers" **/*.csproj 2>/dev/null

# Check for existing test fixtures
find . -name "*Fixture*.cs" -path "*/Tests/*" 2>/dev/null

# Check for existing container configurations
grep -r "PostgreSqlBuilder\|RedisBuilder\|RabbitMqBuilder" **/*.cs 2>/dev/null
```

**If existing setup found:**

- Use the existing fixture patterns and naming conventions
- Add new containers to existing collection definitions
- Follow established test isolation approach

**If no setup found:**

- Create shared fixtures using templates
- Document testing approach in `docs/testing-strategy.md`

## Decision Capture

Document your integration testing approach:

```markdown
<!-- docs/testing-strategy.md or docs/adr/NNNN-integration-testing.md -->

## Integration Testing Approach

**Decision:** Use Testcontainers for integration tests

**Containers Used:**

- PostgreSQL 15 - Primary database
- Redis 7 - Caching layer

**Rationale:**

- Real database catches SQL-specific issues
- Container isolation prevents test pollution
- CI/CD compatible with Docker support

**Lifecycle:**

- Shared fixtures per test collection (performance)
- Transaction rollback for test isolation
```

## Core Workflow

1. **Identify infrastructure** (database type, version, queue, cache)
2. **Select Testcontainers library** (Java, .NET, Python, Node)
3. **Configure container** with production-matching version
4. **Define lifecycle** (per-class or shared fixture)
5. **Implement isolation** (transactions, cleanup, fresh schema)
6. **Set up schema** (migrations or scripts)
7. **Write tests** against real infrastructure
8. **Optimize for CI** (reuse, parallel, image pre-pull)

See `references/language-setup.md` for language-specific configuration.
See `references/performance-optimization.md` for container reuse strategies.
See `references/ci-cd-integration.md` for CI/CD patterns.

## Quick Reference

| Container  | Use Case        | Image                     |
| ---------- | --------------- | ------------------------- |
| PostgreSQL | Relational DB   | postgres:15               |
| MySQL      | Relational DB   | mysql:8                   |
| MongoDB    | Document DB     | mongo:6                   |
| Redis      | Cache           | redis:7-alpine            |
| RabbitMQ   | Message Queue   | rabbitmq:3.12-management  |
| Kafka      | Event Streaming | confluentinc/cp-kafka:7.4 |

## Red Flags - STOP

- "Mocks are faster" / "H2 is close enough"
- "Shared test database is fine" / "CI doesn't support Docker"
- "Testcontainers is too slow" / "Just add better cleanup"

**All mean: Apply testcontainers-integration-tests with optimizations.**

See `references/performance-optimization.md` for rationalizations table.

## Reference Templates

Test fixtures and project templates for quick setup:

| Template                                                      | Purpose                                   |
| ------------------------------------------------------------- | ----------------------------------------- |
| [postgres-fixture.cs](templates/postgres-fixture.cs.template) | PostgreSQL shared fixture with xUnit      |
| [redis-fixture.cs](templates/redis-fixture.cs.template)       | Redis shared fixture with xUnit           |
| [test-project.csproj](templates/test-project.csproj.template) | Test project with Testcontainers packages |

### Quick Setup

```bash
# Copy test project template
cp templates/test-project.csproj.template tests/YourProject.Tests/YourProject.Tests.csproj

# Copy fixture templates
cp templates/postgres-fixture.cs.template tests/YourProject.Tests/Fixtures/PostgresFixture.cs
cp templates/redis-fixture.cs.template tests/YourProject.Tests/Fixtures/RedisFixture.cs

# Restore packages
dotnet restore tests/YourProject.Tests/
```

### Container Packages

Install the appropriate Testcontainers package for your infrastructure:

```bash
# PostgreSQL
dotnet add package Testcontainers.PostgreSql

# Redis
dotnet add package Testcontainers.Redis

# RabbitMQ
dotnet add package Testcontainers.RabbitMq

# MongoDB
dotnet add package Testcontainers.MongoDb
```

## Sample CI Run Logs

### Successful Test Run

```text
Starting test execution...
[Testcontainers] Connected to Docker:
  Host: unix:///var/run/docker.sock
  Server Version: 24.0.7
  API Version: 1.43

[Testcontainers] Pulling image: postgres:15
[Testcontainers] Pulling image: redis:7-alpine
[Testcontainers] Creating container for image: postgres:15
[Testcontainers] Container 8a3f2b... created (postgres:15)
[Testcontainers] Starting container 8a3f2b...
[Testcontainers] Waiting for container 8a3f2b to be ready...
[Testcontainers] Container 8a3f2b started and ready (took 2.3s)
[Testcontainers] Creating container for image: redis:7-alpine
[Testcontainers] Container 4c7e91... created (redis:7-alpine)
[Testcontainers] Starting container 4c7e91...
[Testcontainers] Container 4c7e91 started and ready (took 0.8s)

Test run for MyApp.SystemTest.dll (.NETCoreApp,Version=v8.0)
Microsoft (R) Test Execution Command Line Tool Version 17.8.0

Starting test execution, please wait...
A total of 1 test files matched the specified pattern.

Passed!  - Failed:     0, Passed:    47, Skipped:     0, Total:    47, Duration: 12.4 s

[Testcontainers] Stopping container 8a3f2b...
[Testcontainers] Container 8a3f2b stopped
[Testcontainers] Stopping container 4c7e91...
[Testcontainers] Container 4c7e91 stopped
```

### Failed Test Run (Connection Issue)

```text
Starting test execution...
[Testcontainers] Connected to Docker:
  Host: unix:///var/run/docker.sock

[Testcontainers] Creating container for image: postgres:15
[Testcontainers] Container 2d8a4f... created (postgres:15)
[Testcontainers] Starting container 2d8a4f...
[Testcontainers] Waiting for container 2d8a4f to be ready...
[Testcontainers] Container 2d8a4f started and ready (took 2.1s)

Test run for MyApp.SystemTest.dll (.NETCoreApp,Version=v8.0)

Starting test execution, please wait...

[xUnit.net 00:00:03.45]     MyApp.SystemTest.OrderTests.CreateOrder_WithValidData_ReturnsCreated [FAIL]
  Failed MyApp.SystemTest.OrderTests.CreateOrder_WithValidData_ReturnsCreated [1.2s]
  Error Message:
   Npgsql.NpgsqlException : Failed to connect to 127.0.0.1:32789
   ---> System.Net.Sockets.SocketException : Connection refused
  Stack Trace:
     at Npgsql.Internal.NpgsqlConnector.Connect(...)
     at MyApp.SystemTest.OrderTests.CreateOrder_WithValidData_ReturnsCreated() in OrderTests.cs:line 45

Failed!  - Failed:     1, Passed:    46, Skipped:     0, Total:    47, Duration: 8.7 s

[Testcontainers] Stopping container 2d8a4f...
```

### CI Pipeline Output (GitHub Actions)

```text
Run dotnet test tests/MyApp.SystemTest/
  dotnet test tests/MyApp.SystemTest/ --logger "console;verbosity=detailed" --collect:"XPlat Code Coverage"

info: Testcontainers[0]
      Docker container 8a3f2b is starting (image: postgres:15)
info: Testcontainers[0]
      Docker container 8a3f2b is ready (postgres:15)
info: Testcontainers[0]
      Docker container 4c7e91 is starting (image: redis:7-alpine)
info: Testcontainers[0]
      Docker container 4c7e91 is ready (redis:7-alpine)

Test Run Successful.
Total tests: 47
     Passed: 47
 Total time: 15.234 Seconds

Code Coverage Results:
  Generating report 'coverage.cobertura.xml'
  Line coverage: 78.5%
  Branch coverage: 72.3%
```

## Container Lifecycle Checklist

### Pre-Test Setup

- [ ] Docker daemon running and accessible
- [ ] Required images available (or network access to pull)
- [ ] Sufficient disk space for container volumes
- [ ] Port range available (Testcontainers uses random ports)
- [ ] Test database migration scripts ready

### Per-Test Class Lifecycle

```text
[Collection Start]
  │
  ├─► Create fixture (IAsyncLifetime.InitializeAsync)
  │     ├─► Pull image (if not cached)
  │     ├─► Create container
  │     ├─► Start container
  │     ├─► Wait for ready (health check)
  │     └─► Run migrations/seed data
  │
  ├─► Run tests (parallel within collection)
  │     ├─► Test 1: Begin transaction → Execute → Rollback
  │     ├─► Test 2: Begin transaction → Execute → Rollback
  │     └─► Test N: Begin transaction → Execute → Rollback
  │
  └─► Dispose fixture (IAsyncLifetime.DisposeAsync)
        ├─► Stop container
        └─► Remove container
[Collection End]
```

### Test Isolation Checklist

- [ ] Each test runs in its own transaction (rolled back after)
- [ ] No shared mutable state between tests
- [ ] Test data created with unique identifiers
- [ ] Parallel tests don't conflict on resources
- [ ] Container state reset between test collections

### CI Environment Checklist

- [ ] Docker-in-Docker or Docker socket mounted
- [ ] Sufficient memory for containers (min 2GB recommended)
- [ ] Container cleanup on job failure (use `finally` or `always`)
- [ ] Image caching configured for faster runs
- [ ] Timeout configured for container startup

### Troubleshooting Checklist

| Symptom               | Check                  | Fix                                   |
| --------------------- | ---------------------- | ------------------------------------- |
| Container won't start | Docker daemon running? | `docker ps` to verify                 |
| Connection refused    | Using mapped port?     | Use `container.GetMappedPublicPort()` |
| Tests timeout         | Container ready check? | Add explicit wait strategy            |
| Flaky tests           | Transaction isolation? | Ensure rollback after each test       |
| Slow CI               | Image caching?         | Pre-pull images in CI setup           |
| Port conflicts        | Random ports?          | Let Testcontainers assign ports       |

### Container Health Verification

```csharp
// PostgreSQL ready check
public async Task WaitForPostgresReady()
{
    var connectionString = _postgres.GetConnectionString();
    await using var connection = new NpgsqlConnection(connectionString);

    var retries = 10;
    while (retries > 0)
    {
        try
        {
            await connection.OpenAsync();
            await using var cmd = new NpgsqlCommand("SELECT 1", connection);
            await cmd.ExecuteScalarAsync();
            return; // Ready
        }
        catch
        {
            retries--;
            await Task.Delay(500);
        }
    }
    throw new Exception("PostgreSQL container failed to become ready");
}
```

### Evidence Template

```markdown
## Integration Test Evidence

**Date**: YYYY-MM-DD
**Commit**: abc1234
**CI Run**: [link to CI job]

### Container Configuration

| Container  | Image          | Startup Time | Port  |
| ---------- | -------------- | ------------ | ----- |
| PostgreSQL | postgres:15    | 2.3s         | 32789 |
| Redis      | redis:7-alpine | 0.8s         | 32790 |

### Test Results

| Suite      | Total | Passed | Failed | Duration |
| ---------- | ----- | ------ | ------ | -------- |
| SystemTest | 47    | 47     | 0      | 12.4s    |

### Isolation Verification

- [x] Transaction rollback confirmed
- [x] No cross-test data leakage
- [x] Parallel execution stable

### Container Cleanup

- [x] All containers stopped
- [x] All containers removed
- [x] No orphaned volumes
```
