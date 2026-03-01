---
name: fix-bug
description: Implement bug fixes for Rust code. Use when improvements, issues, GitHub Issue links, or error messages are provided. Triggers on keywords like "fix", "bug", "issue", "error", "broken".
argument-hint: "[issue-number or description]"
---

# Bug Fix Workflow

A systematic workflow for implementing bug fixes.

## Phase 1: Understand the Bug

### Issue/Error Analysis
1. **GitHub Issue**: Fetch details from issue number
   - Check reproduction steps, expected vs actual behavior
   - Review labels, milestones, linked PRs

2. **Error Message**: Parse the stack trace
   - Identify error type (panic, Result::Err, compile error)
   - Locate occurrence point (file:line)

3. **Description Only**: Gather additional information
   - Confirm reproduction steps
   - Check environment (OS, Rust version, DB type)

## Phase 2: Explore the Code

### Find Related Code
```
Exploration order:
1. File where error occurs
2. Callers (upstream)
3. Related test files
4. Related code in the same module
```

### Project Structure
- `src/cli/`: CLI command handling
- `src/core/`: Domain models (schema, migration)
- `src/db/`: Database adapters, SQL generation

## Phase 3: Diagnose Root Cause

### Root Cause Analysis
1. **Form hypothesis**: "This bug might be caused by X"
2. **Verify hypothesis**: Read code, confirm with tests
3. **Identify cause**: Create minimal reproduction case

### Debugging Techniques
```bash
# Run with stack trace
RUST_BACKTRACE=1 cargo run -- <command>

# Run specific test
cargo test <test_name> -- --nocapture

# Inspect variable values (temporary)
dbg!(variable);
```

## Phase 4: Create Fix Branch

### Branch Creation (Before Making Changes)
```bash
# Create and switch to fix branch
git checkout -b fix/<issue-number>-<short-description>

# Example
git checkout -b fix/10-sqlite-nested-transaction
```

### Branch Naming Convention
- `fix/<issue-number>-<short-description>` - For issue-based fixes
- `fix/<short-description>` - For fixes without issue number

## Phase 5: Implement the Fix

### Coding Conventions
- **Error handling**: No `unwrap()`/`expect()`, propagate with `?`
- **Ownership**: Prefer borrowing over unnecessary `.clone()`
- **Naming**: snake_case (functions), PascalCase (types)

### Fix Scope
- Minimal changes to solve the problem
- Avoid unrelated refactoring
- Consider API breaking changes carefully

### Fix Patterns

**Panic Fix**:
```rust
// Before: Potential panic
let value = map.get(key).unwrap();

// After: Proper error handling
let value = map.get(key)
    .ok_or_else(|| anyhow::anyhow!("Key not found: {}", key))?;
```

**Logic Error Fix**:
```rust
// Before: Missing boundary condition
if items.len() > 0 { ... }

// After: Handle empty case
if !items.is_empty() { ... }
```

## Phase 6: Write Tests

### Add Regression Tests
1. **Write a test that reproduces the bug first** (Red)
2. **Confirm the test fails**
3. **Implement the fix** (Green)
4. **Confirm the test passes**

### Test Location
- Unit tests: `#[cfg(test)]` module in the same file
- Integration tests: `src/cli/tests/` directory

### Test Naming Convention
```rust
#[test]
fn test_<feature>_<condition>_<expected_result>() {
    // e.g., test_parse_array_type_with_mixed_case_preserves_casing
}
```

## Phase 7: Verify

### Local Verification
```bash
# Format
cargo fmt

# Lint
cargo clippy -- -D warnings

# Run all tests
cargo test

# Run specific test file
cargo test --test <test_file_name>
```

### Pre-Commit Checklist
- [ ] Bug is fixed and verified
- [ ] Regression test is added
- [ ] All existing tests pass
- [ ] No errors from `cargo fmt` / `cargo clippy`
- [ ] Commit message is appropriate

## Commit Message Format

```
fix(<scope>): <summary>

<body - optional>

Fixes #<issue-number>

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
```

**Scope examples**: cli, core, db, schema, migration, sql, test

## Phase 8: Create Pull Request

### Push Branch and Create PR via MCP

After verification passes, create a pull request using GitHub MCP tools:

1. **Push the fix branch**
   ```bash
   git push -u origin <branch-name>
   ```

2. **Create PR using MCP** (`mcp__github__create_pull_request`)

### PR Content

Read the PR template from `.github/pull_request_template.md` and fill in the sections accordingly:
- **Summary**: Brief description of the fix
- **Related Issue**: `Fixes #<issue-number>`
- **Type of Change**: Check "Bug fix"
- **Test Plan**: Check items based on what was tested
- **Checklist**: Check completed items

### MCP Tool Parameters
```
owner: Lazialize
repo: stratum
title: fix(<scope>): <summary>
head: <fix-branch-name>
base: main
body: <PR content from template>
```

## GitHub Issue Integration

When issue number is provided:
1. Fetch issue details with `gh issue view <number>`
2. Include `Fixes #<number>` in commit message after fix
3. Link to issue when creating PR

## Escape Hatches

Stop and confirm with user in these cases:
- Breaking API changes required
- Multiple fix approaches possible
- Cannot identify root cause
- Security-related issues
