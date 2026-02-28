---
name: bats-testing-patterns
description: Comprehensive guide for writing shell script tests using Bats (Bash Automated Testing System). Use when writing or improving tests for Bash/shell scripts, creating test fixtures, mocking commands, or setting up CI/CD for shell script testing. Includes patterns for assertions, setup/teardown, mocking, fixtures, and integration with GitHub Actions.
---

# Bats Testing Patterns

## Overview

Bats (Bash Automated Testing System) provides a TAP-compliant testing framework for shell scripts. This skill documents proven patterns for writing effective, maintainable shell script tests that catch bugs early and document expected behavior.

**Use this skill when:**

- Writing tests for Bash or shell scripts
- Creating test fixtures and mock data for shell testing
- Setting up test infrastructure for shell-based tools
- Debugging failing shell tests
- Integrating shell tests into CI/CD pipelines

## Core Testing Patterns

### Basic Test Structure

Every Bats test file is a shell script with a `.bats` extension:

```bash
#!/usr/bin/env bats

@test "Test description goes here" {
    # Test code
    [ condition ]
}
```

**Key Points:**

- Use descriptive test names that explain what is being verified
- Each `@test` block is an independent test
- Tests should be focused on one specific behavior
- Use the shebang `#!/usr/bin/env bats` at the top

### Exit Code Assertions

Test command success and failure explicitly:

```bash
#!/usr/bin/env bats

@test "Command succeeds as expected" {
    run echo "hello"
    [ "$status" -eq 0 ]
}

@test "Command fails as expected" {
    run false
    [ "$status" -ne 0 ]
}

@test "Command returns specific exit code" {
    run bash -c "exit 127"
    [ "$status" -eq 127 ]
}

@test "Can capture command result" {
    run echo "hello"
    [ $status -eq 0 ]
    [ "$output" = "hello" ]
}
```

**Best Practice:** Always use `run` to capture command output and exit status. The `run` command sets `$status`, `$output`, and `$lines` variables for assertions.

### Output Assertions

Verify command output matches expectations:

```bash
#!/usr/bin/env bats

@test "Output matches exact string" {
    result=$(echo "hello world")
    [ "$result" = "hello world" ]
}

@test "Output contains substring" {
    result=$(echo "hello world")
    [[ "$result" == *"world"* ]]
}

@test "Output matches regex pattern" {
    result=$(date +%Y)
    [[ "$result" =~ ^[0-9]{4}$ ]]
}

@test "Multi-line output comparison" {
    run printf "line1\nline2\nline3"
    [ "$output" = "line1
line2
line3" ]
}

@test "Using lines array for output" {
    run printf "line1\nline2\nline3"
    [ "${lines[0]}" = "line1" ]
    [ "${lines[1]}" = "line2" ]
    [ "${lines[2]}" = "line3" ]
    [ "${#lines[@]}" -eq 3 ]
}
```

**Tip:** Use the `$lines` array when testing multi-line output - it's cleaner than string comparison.

### File Assertions

Test file operations and attributes:

```bash
#!/usr/bin/env bats

setup() {
    TEST_DIR=$(mktemp -d)
    export TEST_DIR
}

teardown() {
    rm -rf "$TEST_DIR"
}

@test "File is created successfully" {
    [ ! -f "$TEST_DIR/output.txt" ]
    echo "content" > "$TEST_DIR/output.txt"
    [ -f "$TEST_DIR/output.txt" ]
}

@test "File contents match expected" {
    echo "expected content" > "$TEST_DIR/output.txt"
    [ "$(cat "$TEST_DIR/output.txt")" = "expected content" ]
}

@test "File is readable" {
    touch "$TEST_DIR/test.txt"
    [ -r "$TEST_DIR/test.txt" ]
}

@test "File has correct permissions (Linux)" {
    touch "$TEST_DIR/test.txt"
    chmod 644 "$TEST_DIR/test.txt"
    [ "$(stat -c %a "$TEST_DIR/test.txt")" = "644" ]
}

@test "File has correct permissions (macOS)" {
    touch "$TEST_DIR/test.txt"
    chmod 644 "$TEST_DIR/test.txt"
    [ "$(stat -f %OLp "$TEST_DIR/test.txt")" = "644" ]
}

@test "File size is correct" {
    echo -n "12345" > "$TEST_DIR/test.txt"
    [ "$(wc -c < "$TEST_DIR/test.txt")" -eq 5 ]
}

@test "Directory structure is created" {
    mkdir -p "$TEST_DIR/sub/nested/deep"
    [ -d "$TEST_DIR/sub/nested/deep" ]
}
```

**Platform Note:** File permission checking differs between Linux (`stat -c`) and macOS (`stat -f`). Test on your target platform or provide compatibility helpers.

## Setup and Teardown Patterns

### Basic Setup and Teardown

Execute code before and after each test:

```bash
#!/usr/bin/env bats

setup() {
    # Runs before EACH test
    TEST_DIR=$(mktemp -d)
    export TEST_DIR

    # Source the script under test
    source "${BATS_TEST_DIRNAME}/../bin/script.sh"
}

teardown() {
    # Runs after EACH test
    rm -rf "$TEST_DIR"
}

@test "Test using TEST_DIR" {
    touch "$TEST_DIR/file.txt"
    [ -f "$TEST_DIR/file.txt" ]
}

@test "Second test has clean TEST_DIR" {
    # TEST_DIR is recreated fresh for each test
    [ ! -f "$TEST_DIR/file.txt" ]
}
```

**Critical:** The `setup()` and `teardown()` functions run before and after EACH test, ensuring test isolation.

### Setup with Test Resources

Create fixtures and test data:

```bash
#!/usr/bin/env bats

setup() {
    # Create directory structure
    TEST_DIR=$(mktemp -d)
    mkdir -p "$TEST_DIR/data/input"
    mkdir -p "$TEST_DIR/data/output"

    # Create test fixtures
    echo "line1" > "$TEST_DIR/data/input/file1.txt"
    echo "line2" > "$TEST_DIR/data/input/file2.txt"
    echo "line3" > "$TEST_DIR/data/input/file3.txt"

    # Initialize environment variables
    export DATA_DIR="$TEST_DIR/data"
    export INPUT_DIR="$DATA_DIR/input"
    export OUTPUT_DIR="$DATA_DIR/output"
    
    # Source the script being tested
    source "${BATS_TEST_DIRNAME}/../scripts/process_files.sh"
}

teardown() {
    rm -rf "$TEST_DIR"
}

@test "Processes all input files" {
    process_files "$INPUT_DIR" "$OUTPUT_DIR"
    [ -f "$OUTPUT_DIR/file1.txt" ]
    [ -f "$OUTPUT_DIR/file2.txt" ]
    [ -f "$OUTPUT_DIR/file3.txt" ]
}

@test "Handles empty input directory" {
    rm -rf "$INPUT_DIR"/*
    process_files "$INPUT_DIR" "$OUTPUT_DIR"
    [ "$(ls -A "$OUTPUT_DIR")" = "" ]
}
```

### Global Setup/Teardown

Run expensive setup once for all tests:

```bash
#!/usr/bin/env bats

# Load shared test utilities
load test_helper

# setup_file runs ONCE before all tests in the file
setup_file() {
    export SHARED_RESOURCE=$(mktemp -d)
    export SHARED_DB="$SHARED_RESOURCE/test.db"
    
    # Expensive operation: initialize database
    echo "Creating test database..."
    sqlite3 "$SHARED_DB" < "${BATS_TEST_DIRNAME}/fixtures/schema.sql"
}

# teardown_file runs ONCE after all tests in the file
teardown_file() {
    rm -rf "$SHARED_RESOURCE"
}

# setup runs before each test (optional)
setup() {
    # Per-test setup if needed
    export TEST_ID=$(date +%s%N)
}

@test "First test uses shared resource" {
    [ -f "$SHARED_DB" ]
    sqlite3 "$SHARED_DB" "SELECT COUNT(*) FROM users;"
}

@test "Second test uses same shared resource" {
    [ -f "$SHARED_DB" ]
    # Database persists between tests
    sqlite3 "$SHARED_DB" "INSERT INTO users (name) VALUES ('test_$TEST_ID');"
}
```

**Use Case:** Global setup/teardown is perfect for expensive operations like database initialization, server startup, or large file downloads that can be shared across tests.

## Mocking and Stubbing Patterns

### Function Mocking

Override functions for testing:

```bash
#!/usr/bin/env bats

# Mock external command
curl() {
    echo '{"status": "success", "data": "mocked"}'
    return 0
}

@test "Function uses mocked curl" {
    export -f curl
    
    # Source script that calls curl
    source "${BATS_TEST_DIRNAME}/../scripts/api_client.sh"
    
    result=$(fetch_data "https://api.example.com/data")
    [[ "$result" == *"mocked"* ]]
}

@test "Mock can simulate failure" {
    curl() {
        echo "Connection refused"
        return 1
    }
    export -f curl
    
    source "${BATS_TEST_DIRNAME}/../scripts/api_client.sh"
    run fetch_data "https://api.example.com/data"
    [ "$status" -ne 0 ]
}
```

### Command Stubbing with PATH Manipulation

Create stub commands that override system commands:

```bash
#!/usr/bin/env bats

setup() {
    # Create stub directory
    STUBS_DIR="$BATS_TEST_TMPDIR/stubs"
    mkdir -p "$STUBS_DIR"

    # Prepend to PATH so stubs are found first
    export PATH="$STUBS_DIR:$PATH"
}

teardown() {
    rm -rf "$STUBS_DIR"
}

create_stub() {
    local cmd="$1"
    local output="$2"
    local exit_code="${3:-0}"

    cat > "$STUBS_DIR/$cmd" <<EOF
#!/bin/bash
echo "$output"
exit $exit_code
EOF
    chmod +x "$STUBS_DIR/$cmd"
}

@test "Function works with stubbed curl" {
    create_stub curl '{"status": "ok"}' 0
    
    source "${BATS_TEST_DIRNAME}/../scripts/api_client.sh"
    run fetch_api_status
    [ "$status" -eq 0 ]
    [[ "$output" == *"ok"* ]]
}

@test "Function handles stubbed curl failure" {
    create_stub curl "curl: (6) Could not resolve host" 6
    
    source "${BATS_TEST_DIRNAME}/../scripts/api_client.sh"
    run fetch_api_status
    [ "$status" -ne 0 ]
}

@test "Can stub multiple commands" {
    create_stub git "commit abc123" 0
    create_stub docker "Container started" 0
    
    # Test code that uses both git and docker
    run git status
    [[ "$output" == *"abc123"* ]]
    
    run docker ps
    [[ "$output" == *"started"* ]]
}
```

**Powerful Pattern:** PATH manipulation allows stubbing any command without modifying the code under test.

### Environment Variable Stubbing

Test different configurations:

```bash
#!/usr/bin/env bats

@test "Function uses environment override" {
    export LOG_LEVEL="DEBUG"
    export API_ENDPOINT="https://staging.example.com"
    
    source "${BATS_TEST_DIRNAME}/../scripts/config.sh"
    run get_config_value "log_level"
    [[ "$output" == *"DEBUG"* ]]
}

@test "Function uses defaults when vars unset" {
    unset LOG_LEVEL
    unset API_ENDPOINT
    
    source "${BATS_TEST_DIRNAME}/../scripts/config.sh"
    run get_config_value "log_level"
    [[ "$output" == *"INFO"* ]]  # Default value
}

@test "Function handles missing config file" {
    export CONFIG_FILE="/nonexistent/config.yaml"
    
    source "${BATS_TEST_DIRNAME}/../scripts/config.sh"
    run load_config
    [ "$status" -ne 0 ]
    [[ "$output" == *"not found"* ]]
}
```

## Fixture Management

### Using Fixture Files

Organize test data in a dedicated directory:

```bash
#!/usr/bin/env bats

# Directory structure:
# tests/
# ├── fixtures/
# │   ├── input.json
# │   ├── expected_output.json
# │   └── schema.sql
# └── script.bats

setup() {
    FIXTURES_DIR="${BATS_TEST_DIRNAME}/fixtures"
    WORK_DIR=$(mktemp -d)
    export WORK_DIR
}

teardown() {
    rm -rf "$WORK_DIR"
}

@test "Process fixture file produces expected output" {
    # Copy fixture to working directory
    cp "$FIXTURES_DIR/input.json" "$WORK_DIR/input.json"

    # Run processing function
    source "${BATS_TEST_DIRNAME}/../scripts/processor.sh"
    run process_json "$WORK_DIR/input.json" "$WORK_DIR/output.json"
    [ "$status" -eq 0 ]

    # Compare output with expected fixture
    diff "$WORK_DIR/output.json" "$FIXTURES_DIR/expected_output.json"
}

@test "Handles malformed input file" {
    echo "invalid json" > "$WORK_DIR/bad.json"
    
    source "${BATS_TEST_DIRNAME}/../scripts/processor.sh"
    run process_json "$WORK_DIR/bad.json" "$WORK_DIR/output.json"
    [ "$status" -ne 0 ]
    [[ "$output" == *"invalid"* ]]
}
```

**Organization Tip:** Keep fixtures in a `fixtures/` directory alongside your test files for easy maintenance.

### Dynamic Fixture Generation

Generate test data programmatically:

```bash
#!/usr/bin/env bats

generate_csv_fixture() {
    local rows="$1"
    local file="$2"

    echo "id,name,email" > "$file"
    for i in $(seq 1 "$rows"); do
        echo "$i,User$i,user$i@example.com" >> "$file"
    done
}

generate_log_fixture() {
    local lines="$1"
    local file="$2"

    for i in $(seq 1 "$lines"); do
        echo "[$(date -Iseconds)] INFO: Log entry $i" >> "$file"
    done
}

@test "Handles large CSV file" {
    generate_csv_fixture 10000 "$BATS_TEST_TMPDIR/large.csv"
    
    source "${BATS_TEST_DIRNAME}/../scripts/csv_parser.sh"
    run parse_csv "$BATS_TEST_TMPDIR/large.csv"
    [ "$status" -eq 0 ]
    [ "$(wc -l < "$BATS_TEST_TMPDIR/large.csv")" -eq 10001 ]  # Header + 10000 rows
}

@test "Handles log file with 1000 entries" {
    generate_log_fixture 1000 "$BATS_TEST_TMPDIR/app.log"
    
    source "${BATS_TEST_DIRNAME}/../scripts/log_analyzer.sh"
    run analyze_logs "$BATS_TEST_TMPDIR/app.log"
    [ "$status" -eq 0 ]
}
```

**Benefit:** Dynamic fixtures make tests more flexible and can test edge cases like performance with large datasets.

## Advanced Testing Patterns

### Testing Error Conditions

Ensure proper error handling:

```bash
#!/usr/bin/env bats

@test "Fails gracefully with missing required file" {
    source "${BATS_TEST_DIRNAME}/../scripts/processor.sh"
    run process_file "/nonexistent/file.txt"
    [ "$status" -ne 0 ]
    [[ "$output" == *"not found"* || "$output" == *"No such file"* ]]
}

@test "Fails with helpful message for invalid input" {
    source "${BATS_TEST_DIRNAME}/../scripts/processor.sh"
    run process_file ""
    [ "$status" -ne 0 ]
    [[ "$output" == *"Usage:"* || "$output" == *"required"* ]]
}

@test "Handles permission denied gracefully" {
    touch "$BATS_TEST_TMPDIR/readonly.txt"
    chmod 000 "$BATS_TEST_TMPDIR/readonly.txt"
    
    source "${BATS_TEST_DIRNAME}/../scripts/processor.sh"
    run process_file "$BATS_TEST_TMPDIR/readonly.txt"
    [ "$status" -ne 0 ]
    [[ "$output" == *"Permission denied"* || "$output" == *"cannot read"* ]]
    
    # Cleanup: restore permissions
    chmod 644 "$BATS_TEST_TMPDIR/readonly.txt"
}

@test "Provides usage help with invalid option" {
    source "${BATS_TEST_DIRNAME}/../scripts/processor.sh"
    run process_file --invalid-option
    [ "$status" -ne 0 ]
    [[ "$output" == *"Usage:"* ]]
}

@test "Validates input format" {
    echo "not a number" > "$BATS_TEST_TMPDIR/invalid.txt"
    
    source "${BATS_TEST_DIRNAME}/../scripts/numeric_processor.sh"
    run process_numbers "$BATS_TEST_TMPDIR/invalid.txt"
    [ "$status" -ne 0 ]
    [[ "$output" == *"invalid"* || "$output" == *"number"* ]]
}
```

**Best Practice:** Test error paths thoroughly - they're often overlooked but critical for user experience.

### Testing with External Dependencies

Handle optional dependencies gracefully:

```bash
#!/usr/bin/env bats

setup() {
    # Check for required tools
    if ! command -v jq &>/dev/null; then
        skip "jq is not installed - required for JSON tests"
    fi

    source "${BATS_TEST_DIRNAME}/../scripts/json_processor.sh"
}

@test "JSON parsing works with jq" {
    run parse_json '{"name": "test", "value": 42}'
    [ "$status" -eq 0 ]
    [[ "$output" == *"test"* ]]
}

@test "Can process complex nested JSON" {
    skip_if_missing jq
    
    json='{"users": [{"name": "Alice", "age": 30}, {"name": "Bob", "age": 25}]}'
    run extract_user_names "$json"
    [ "$status" -eq 0 ]
    [[ "$output" == *"Alice"* ]]
    [[ "$output" == *"Bob"* ]]
}

skip_if_missing() {
    local tool="$1"
    if ! command -v "$tool" &>/dev/null; then
        skip "$tool is not installed"
    fi
}
```

**Graceful Degradation:** Use `skip` to mark tests that can't run in the current environment instead of failing them.

### Testing Shell Compatibility

Ensure scripts work across different shells:

```bash
#!/usr/bin/env bats

@test "Script works in bash" {
    bash "${BATS_TEST_DIRNAME}/../scripts/portable.sh" --version
}

@test "Script works in sh (POSIX mode)" {
    sh "${BATS_TEST_DIRNAME}/../scripts/portable.sh" --version
}

@test "Script works in dash" {
    if ! command -v dash &>/dev/null; then
        skip "dash not installed"
    fi
    dash "${BATS_TEST_DIRNAME}/../scripts/portable.sh" --version
}

@test "Script uses POSIX-compliant syntax" {
    # Check for bash-specific features
    ! grep -q 'function ' "${BATS_TEST_DIRNAME}/../scripts/portable.sh"
    ! grep -q '\[\[' "${BATS_TEST_DIRNAME}/../scripts/portable.sh"
    ! grep -q '=~' "${BATS_TEST_DIRNAME}/../scripts/portable.sh"
}
```

**Portability Tip:** If your script should work on different systems, test with multiple shells to catch compatibility issues.

### Parallel Test Execution

Test concurrent operations:

```bash
#!/usr/bin/env bats

@test "Multiple operations can run concurrently" {
    source "${BATS_TEST_DIRNAME}/../scripts/parallel_processor.sh"
    
    # Start multiple background processes
    for i in {1..5}; do
        process_item "$i" "$BATS_TEST_TMPDIR/output_$i.txt" &
    done
    
    # Wait for all to complete
    wait
    
    # Verify all outputs
    for i in {1..5}; do
        [ -f "$BATS_TEST_TMPDIR/output_$i.txt" ]
    done
}

@test "Concurrent file operations don't conflict" {
    source "${BATS_TEST_DIRNAME}/../scripts/file_handler.sh"
    
    # Create multiple files concurrently
    for i in {1..10}; do
        (
            echo "Content $i" > "$BATS_TEST_TMPDIR/file_$i.txt"
        ) &
    done
    wait
    
    # Verify no data corruption
    [ "$(ls "$BATS_TEST_TMPDIR"/file_*.txt | wc -l)" -eq 10 ]
}
```

## Test Organization with Helpers

### test_helper.sh Pattern

Create reusable test utilities:

```bash
#!/usr/bin/env bash
# File: tests/test_helper.bash

# Source script under test
export SCRIPT_DIR="${BATS_TEST_DIRNAME%/*}/scripts"
export BIN_DIR="${BATS_TEST_DIRNAME%/*}/bin"

# Common assertion helpers
assert_file_exists() {
    local file="$1"
    if [ ! -f "$file" ]; then
        echo "Expected file to exist: $file" >&2
        return 1
    fi
}

assert_file_contains() {
    local file="$1"
    local pattern="$2"
    
    if [ ! -f "$file" ]; then
        echo "File does not exist: $file" >&2
        return 1
    fi

    if ! grep -q "$pattern" "$file"; then
        echo "File does not contain pattern: $pattern" >&2
        echo "File contents:" >&2
        cat "$file" >&2
        return 1
    fi
}

assert_output_contains() {
    local pattern="$1"
    if [[ ! "$output" == *"$pattern"* ]]; then
        echo "Output does not contain: $pattern" >&2
        echo "Actual output:" >&2
        echo "$output" >&2
        return 1
    fi
}

# Setup helpers
create_test_workspace() {
    export TEST_WORKSPACE=$(mktemp -d)
    mkdir -p "$TEST_WORKSPACE/input"
    mkdir -p "$TEST_WORKSPACE/output"
    mkdir -p "$TEST_WORKSPACE/temp"
}

cleanup_test_workspace() {
    if [ -n "$TEST_WORKSPACE" ] && [ -d "$TEST_WORKSPACE" ]; then
        rm -rf "$TEST_WORKSPACE"
    fi
}

# Stub helpers
create_command_stub() {
    local cmd="$1"
    local output="$2"
    local exit_code="${3:-0}"
    local stub_dir="${STUBS_DIR:-$BATS_TEST_TMPDIR/stubs}"
    
    mkdir -p "$stub_dir"
    
    cat > "$stub_dir/$cmd" <<EOF
#!/bin/bash
echo "$output"
exit $exit_code
EOF
    chmod +x "$stub_dir/$cmd"
    export PATH="$stub_dir:$PATH"
}
```

**Usage in tests:**

```bash
#!/usr/bin/env bats

load test_helper

setup() {
    create_test_workspace
}

teardown() {
    cleanup_test_workspace
}

@test "Uses test helper for assertions" {
    echo "content" > "$TEST_WORKSPACE/output/result.txt"
    
    assert_file_exists "$TEST_WORKSPACE/output/result.txt"
    assert_file_contains "$TEST_WORKSPACE/output/result.txt" "content"
}

@test "Uses helper to create stubs" {
    create_command_stub "git" "commit abc123" 0
    
    run git status
    assert_output_contains "abc123"
}
```

**Benefits:**

- Reduces code duplication across test files
- Provides consistent error messages
- Makes tests more readable and maintainable

## CI/CD Integration

### GitHub Actions Workflow

```yaml
name: Shell Script Tests

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        shell: [bash, dash, sh]

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Install Bats
        run: |
          npm install --global bats

      - name: Run tests with ${{ matrix.shell }}
        run: |
          export TEST_SHELL=${{ matrix.shell }}
          bats tests/*.bats

      - name: Run tests with TAP output
        if: always()
        run: |
          bats tests/*.bats --tap | tee test_output.tap

      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: test-results-${{ matrix.shell }}
          path: test_output.tap
```

**Advanced workflow with parallel execution:**

```yaml
name: Comprehensive Shell Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ${{ matrix.os }}
    
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest]
        bats-version: ['1.11.0']
      fail-fast: false

    steps:
      - uses: actions/checkout@v4

      - name: Setup Bats
        uses: bats-core/bats-action@2.0.0

      - name: Run unit tests
        run: bats tests/unit/*.bats --timing

      - name: Run integration tests
        run: bats tests/integration/*.bats --timing

      - name: Run tests in parallel
        run: bats tests/*.bats --jobs 4 --timing

      - name: Generate coverage report
        if: matrix.os == 'ubuntu-latest'
        run: |
          # Optional: Use kcov or similar for coverage
          echo "Coverage reporting setup here"
```

### Makefile Integration

```makefile
.PHONY: test test-verbose test-tap test-unit test-integration test-parallel clean

# Default test target
test:
    @echo "Running all tests..."
    bats tests/*.bats

# Verbose output for debugging
test-verbose:
    @echo "Running tests with verbose output..."
    bats tests/*.bats --verbose

# TAP format output
test-tap:
    @echo "Running tests with TAP output..."
    bats tests/*.bats --tap

# Run only unit tests
test-unit:
    @echo "Running unit tests..."
    bats tests/unit/*.bats

# Run only integration tests
test-integration:
    @echo "Running integration tests..."
    bats tests/integration/*.bats

# Run tests in parallel for speed
test-parallel:
    @echo "Running tests in parallel..."
    bats tests/*.bats --jobs 4

# Watch mode for TDD workflow
test-watch:
    @echo "Watching for changes..."
    while true; do \
        make test; \
        inotifywait -qre close_write tests/ scripts/; \
    done

# Clean up test artifacts
clean:
    @echo "Cleaning up test artifacts..."
    rm -rf tests/tmp/
    rm -f test_output.tap
    rm -f coverage/

# Run linting on shell scripts
lint:
    @echo "Linting shell scripts..."
    shellcheck scripts/*.sh
    shellcheck tests/*.bats

# Full validation: lint + test
validate: lint test
    @echo "✓ All checks passed"
```

**Usage:**

```bash
# Run all tests
make test

# Run with verbose output for debugging
make test-verbose

# Run tests in parallel for CI
make test-parallel

# Run only integration tests
make test-integration

# Full validation before commit
make validate
```

## Best Practices Summary

### Test Quality

1. **One assertion per test** - Tests should verify a single behavior
2. **Descriptive test names** - Use clear, complete sentences
3. **Test independence** - Tests should not depend on execution order
4. **Clean up resources** - Always remove temporary files in teardown
5. **Test both paths** - Verify success AND failure scenarios

### Test Organization

1. **Group related tests** - Use separate files for unit vs integration tests
2. **Use fixtures** - Store test data in dedicated `fixtures/` directory
3. **Create helpers** - Extract common patterns into `test_helper.bash`
4. **Document complex setups** - Explain unusual test patterns with comments

### Performance

1. **Use global setup** - Run expensive operations once with `setup_file`
2. **Run in parallel** - Use `bats --jobs N` for faster test execution
3. **Mock external calls** - Stub network requests and slow commands
4. **Keep tests fast** - Each test should complete in milliseconds

### Maintainability

1. **Follow conventions** - Consistent naming and structure
2. **Version control fixtures** - Check in test data files
3. **Update tests with code** - Keep tests in sync with implementation
4. **Review test failures** - Investigate and fix flaky tests immediately

### CI/CD Integration

1. **Run tests automatically** - On every push and pull request
2. **Test multiple environments** - Different OS and shell versions
3. **Generate reports** - Use TAP output for test dashboards
4. **Fail fast** - Stop CI on test failures

## Common Pitfalls and Solutions

### Pitfall: Tests Pass but Code is Broken

**Problem:** Tests don't actually verify the behavior

**Solution:** Always watch tests fail first (TDD approach)

```bash
# Write test first
@test "Function returns correct value" {
    run my_function "input"
    [ "$output" = "expected" ]
}

# Run and verify it FAILS (function doesn't exist yet)
# Then implement the function
# Run again and verify it PASSES
```

### Pitfall: Tests Depend on System State

**Problem:** Tests pass on developer machine but fail in CI

**Solution:** Isolate tests with proper setup/teardown

```bash
setup() {
    # Create isolated environment
    export HOME="$BATS_TEST_TMPDIR/home"
    export XDG_CONFIG_HOME="$HOME/.config"
    mkdir -p "$XDG_CONFIG_HOME"
    
    # Set up clean PATH
    export PATH="/usr/local/bin:/usr/bin:/bin"
}
```

### Pitfall: Flaky Tests Due to Timing

**Problem:** Tests occasionally fail due to race conditions

**Solution:** Use proper synchronization

```bash
@test "Waits for background process" {
    my_background_task &
    local pid=$!
    
    # Wait with timeout
    for i in {1..30}; do
        if [ -f "$BATS_TEST_TMPDIR/done.txt" ]; then
            break
        fi
        sleep 0.1
    done
    
    wait $pid
    [ -f "$BATS_TEST_TMPDIR/done.txt" ]
}
```

### Pitfall: Hard to Debug Failures

**Problem:** Test fails but output doesn't show why

**Solution:** Add diagnostic output

```bash
@test "Processes file correctly" {
    run process_file "$input"
    
    # Add diagnostic output on failure
    if [ "$status" -ne 0 ]; then
        echo "Command failed with status: $status" >&2
        echo "Output:" >&2
        echo "$output" >&2
        echo "Input file contents:" >&2
        cat "$input" >&2
    fi
    
    [ "$status" -eq 0 ]
}
```

## Additional Resources

### Official Documentation

- **Bats Core**: <https://github.com/bats-core/bats-core>
- **Bats Docs**: <https://bats-core.readthedocs.io/>
- **TAP Protocol**: <https://testanything.org/>

### Bats Libraries

- **bats-support**: <https://github.com/bats-core/bats-support> - Additional assertions
- **bats-assert**: <https://github.com/bats-core/bats-assert> - Helpful assertion functions
- **bats-file**: <https://github.com/bats-core/bats-file> - File system assertions

### Testing Methodology

- **Test-Driven Development**: <https://en.wikipedia.org/wiki/Test-driven_development>
- **Testing Best Practices**: Write tests that document behavior, not implementation

### Shell Testing Tools

- **ShellCheck**: <https://www.shellcheck.net/> - Static analysis for shell scripts
- **shfmt**: <https://github.com/mvdan/sh> - Shell script formatter
- **bashate**: <https://github.com/openstack/bashate> - Bash style checker

## Quick Reference

### Common Bats Variables

- `$status` - Exit code of last `run` command
- `$output` - Combined stdout/stderr of last `run` command
- `$lines` - Array of output lines from last `run` command
- `${lines[0]}` - First line of output
- `${#lines[@]}` - Number of output lines
- `$BATS_TEST_DIRNAME` - Directory containing the test file
- `$BATS_TEST_FILENAME` - Filename of the test file
- `$BATS_TEST_NAME` - Name of the current test
- `$BATS_TEST_TMPDIR` - Temporary directory for the current test

### Common Assertions

```bash
# Exit codes
[ "$status" -eq 0 ]      # Success
[ "$status" -ne 0 ]      # Failure
[ "$status" -eq 127 ]    # Specific code

# String comparison
[ "$output" = "exact" ]                # Exact match
[[ "$output" == *"substring"* ]]       # Contains
[[ "$output" =~ ^pattern$ ]]           # Regex match

# File tests
[ -f "$file" ]           # File exists
[ -d "$dir" ]            # Directory exists
[ -r "$file" ]           # Readable
[ -w "$file" ]           # Writable
[ -x "$file" ]           # Executable
[ -s "$file" ]           # Not empty

# Numeric comparison
[ "$count" -eq 5 ]       # Equal
[ "$count" -gt 0 ]       # Greater than
[ "$count" -lt 10 ]      # Less than
```

### Useful Patterns

```bash
# Run command and capture output
run command arg1 arg2

# Skip test conditionally
skip "Reason for skipping"
skip_if_missing "jq"

# Load helper functions
load test_helper

# Create temp directory
TEST_DIR=$(mktemp -d)

# Check command exists
command -v tool &>/dev/null

# Stub a command
PATH="/path/to/stubs:$PATH"
```
