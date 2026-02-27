---
name: code-reviewer
description: Reviews code changes in pull requests for the Morphir Moonbit monorepo. Checks for code quality, best practices, security issues, and adherence to project conventions. Use when reviewing PRs or code changes before merging.
license: Apache-2.0
compatibility:
  - claude
  - copilot
  - codex
  - cursor
  - windsurf
  - google-antigravity
---

# Code Reviewer Skill

## Purpose

This skill provides comprehensive code review capabilities for the Morphir Moonbit monorepo. It helps ensure code quality, consistency, and adherence to project standards before changes are merged.

## When to Use

- Reviewing pull requests
- Before merging code changes
- During code quality checks
- When validating adherence to project conventions

## Review Areas

### 1. Moonbit Code Quality

- **Formatting**: Ensure code follows Moonbit formatting standards (use `moon fmt`)
- **Naming Conventions**: Check that functions, types, and variables follow consistent naming
- **Code Structure**: Verify proper use of modules and package organization
- **Type Safety**: Ensure proper type annotations and type-safe code
- **Comments**: Check for appropriate documentation and comments

### 2. Package Structure

- **Dependencies**: Verify correct dependency declarations in `moon.mod.json`
- **Package Config**: Ensure `moon.pkg.json` is properly configured
- **Module Organization**: Check that code is in the correct package
- **Imports**: Verify proper import statements in `moon.pkg.json`

### 3. Testing

- **Test Coverage**: Ensure new code has corresponding tests
- **Test Files**: Verify test files follow naming convention (`*_test.mbt`)
- **Test Quality**: Check that tests are meaningful and cover edge cases
- **Test Execution**: Verify tests pass with `mise run test`

### 4. Build Configuration

- **Build Targets**: Ensure code works for both WASI and browser (wasm-gc) targets
- **Build Success**: Verify builds complete successfully
- **No Build Warnings**: Check for and address any build warnings

### 5. Documentation

- **README Updates**: Check if README needs updates for new features
- **API Documentation**: Ensure public APIs are documented
- **Code Comments**: Verify complex logic has explanatory comments
- **Examples**: Check if new features include usage examples

### 6. Security

- **Dependencies**: Check for known vulnerabilities in dependencies
- **Input Validation**: Ensure proper input validation
- **Error Handling**: Verify appropriate error handling
- **Secrets**: Ensure no secrets or sensitive data in code

### 7. Mise Tasks

- **Task Updates**: Check if new functionality requires new mise tasks
- **Task Documentation**: Ensure tasks are documented in `docs/MISE_TASKS.md`
- **Cross-Platform**: Verify both bash and PowerShell versions exist

### 8. CI/CD

- **Workflow Updates**: Check if workflows need updates for new features
- **No Inline Scripts**: Ensure workflows only use mise tasks
- **Parallel Execution**: Verify CI jobs run efficiently in parallel

## Review Process

### Step 1: Understand the Changes

```bash
# View the changes
git diff main...HEAD

# Check which files changed
git diff --name-only main...HEAD
```

### Step 2: Run Local Checks

```bash
# Format code
mise run format

# Run linting
mise run lint

# Run tests
mise run test

# Validate configuration
mise run validate

# Run all checks
mise run check
```

### Step 3: Review Code Quality

- Read through the code changes
- Check for logic errors or potential bugs
- Verify adherence to Moonbit best practices
- Look for opportunities to improve code quality

### Step 4: Check Package Structure

```bash
# Verify package configurations exist
ls -la pkgs/*/moon.mod.json
ls -la pkgs/*/moon.pkg.json

# Validate package structure
mise run validate:packages
```

### Step 5: Verify Tests

```bash
# Run tests for specific package
cd pkgs/<package-name>
moon test

# Run all tests
cd ../..
mise run test
```

### Step 6: Build for All Targets

```bash
# Build for WASI
mise run build:wasi

# Build for browser
mise run build:browser

# Or build all
mise run build
```

### Step 7: Check Documentation

- Review if documentation needs updates
- Verify public APIs are documented
- Check if CHANGELOG should be updated
- Ensure README reflects new features

## Common Issues to Look For

### Moonbit-Specific

- Missing type annotations
- Inconsistent formatting (not using `moon fmt`)
- Improper module imports
- Missing test files for new functionality
- Hard-coded values that should be configurable

### Project-Specific

- Missing both bash and PowerShell versions of mise tasks
- Inline scripts in GitHub workflows (should use mise tasks)
- Breaking changes to existing APIs
- Missing dependencies in `moon.mod.json`
- Incorrect import statements in `moon.pkg.json`

### General Best Practices

- Code duplication
- Overly complex functions (consider breaking down)
- Poor error messages
- Missing edge case handling
- Performance issues

## Review Checklist

Use this checklist when reviewing code:

- [ ] Code is properly formatted (`mise run format`)
- [ ] All linting passes (`mise run lint`)
- [ ] Tests exist for new functionality
- [ ] All tests pass (`mise run test`)
- [ ] Builds succeed for both targets (`mise run build`)
- [ ] Package structure is valid (`mise run validate:packages`)
- [ ] Documentation is updated if needed
- [ ] No security vulnerabilities introduced
- [ ] No secrets or sensitive data in code
- [ ] Mise tasks updated if needed (bash + PowerShell)
- [ ] CI workflows use mise tasks only (no inline scripts)
- [ ] Breaking changes are documented
- [ ] Code follows project conventions
- [ ] Dependencies are properly declared

## Providing Feedback

When providing review feedback:

1. **Be Specific**: Point to exact lines or files
2. **Be Constructive**: Suggest improvements, don't just criticize
3. **Prioritize**: Distinguish between must-fix and nice-to-have
4. **Explain Why**: Help developers understand the reasoning
5. **Acknowledge Good Work**: Point out well-written code too

### Severity Levels

- **Critical**: Blocks merge - security issues, broken functionality
- **Major**: Should fix before merge - code quality, missing tests
- **Minor**: Nice to have - style improvements, documentation
- **Suggestion**: Optional - potential improvements

## Example Review Comments

### Good Examples

```
Critical: Missing input validation on line 45. This could lead to a runtime 
error if the user provides an empty string. Please add validation.

Major: The function `processData` at line 120 is missing test coverage. 
Please add tests covering both success and error cases.

Minor: Consider extracting lines 80-95 into a separate function for better 
readability and reusability.

Suggestion: Nice use of pattern matching here! You might also consider 
using the Option type instead of nullable for clearer intent.
```

### What to Avoid

```
❌ "This code is bad."
✅ "This function could be simplified by using pattern matching."

❌ "You forgot tests."
✅ "Please add tests for this new function, particularly for the edge case 
   when the input list is empty."

❌ "Wrong."
✅ "The dependency declaration should use version 0.1.0 to match the other 
   packages in the monorepo."
```

## Tools and Commands

### View Changes

```bash
git diff main...HEAD                    # View all changes
git diff --name-only main...HEAD        # List changed files
git diff main...HEAD -- path/to/file    # View changes in specific file
```

### Run Checks Locally

```bash
mise run check          # Run all checks
mise run lint           # Linting only
mise run test           # Tests only
mise run build          # Build only
mise run validate       # Validation only
```

### Package-Specific Checks

```bash
cd pkgs/morphir-core
moon fmt --check        # Check formatting
moon test               # Run tests
moon build --target wasm        # Build for WASI
moon build --target wasm-gc     # Build for browser
```

## Resources

- [Moonbit Documentation](https://www.moonbitlang.com/docs/)
- [Project Architecture](docs/ARCHITECTURE.md)
- [Mise Tasks Reference](docs/MISE_TASKS.md)
- [Contributing Guide](CONTRIBUTING_DEV.md)

## Notes

- Always run `mise run check` before approving a PR
- Tests must pass in CI before merge
- All workflows must use mise tasks (no inline scripts)
- Each package must have both `moon.mod.json` and `moon.pkg.json`
- Mise tasks must have both bash and PowerShell versions
