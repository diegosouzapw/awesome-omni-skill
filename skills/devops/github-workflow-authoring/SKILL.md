---
name: github-workflow-authoring
description: This skill should be used when creating or improving GitHub Actions CI/CD workflows for Breenix kernel development. Use for authoring new test workflows, optimizing existing CI pipelines, adding new test types, fixing workflow configuration issues, or adapting workflows for new kernel features.
---

# GitHub Workflow Authoring for Breenix

Create and improve GitHub Actions workflows for Breenix OS kernel development and testing.

## Purpose

This skill provides patterns, templates, and best practices for authoring GitHub Actions workflows specifically for Breenix kernel development. It addresses the unique challenges of OS kernel CI/CD: QEMU virtualization, custom Rust targets, userspace binary building, timeout management, and kernel-specific test patterns.

## When to Use This Skill

Use this skill when:

- **Creating new test workflows**: Adding CI for new kernel features or test suites
- **Optimizing CI performance**: Reducing build times, improving caching, tuning timeouts
- **Fixing CI failures**: Workflow configuration issues, missing dependencies, wrong environment
- **Adapting workflows**: Modifying workflows for new kernel capabilities or test requirements
- **Debugging CI issues**: Understanding why workflows fail, reproducing issues locally
- **Adding test coverage**: Expanding CI to cover more kernel subsystems or scenarios

## Key Breenix CI Patterns

### Rust Toolchain Requirements

Breenix requires specific Rust configuration:

```yaml
- name: Install Rust
  uses: actions-rs/toolchain@v1
  with:
    toolchain: nightly-2025-06-24      # Pinned for consistency
    override: true
    target: x86_64-unknown-none        # Custom kernel target
    components: rust-src, llvm-tools-preview
```

**Critical**: The Rust nightly version is pinned to avoid unexpected breakage from compiler changes.

### System Dependencies

All kernel tests require QEMU and supporting tools:

```yaml
- name: Install system dependencies
  run: |
    sudo apt-get update
    sudo apt-get install -y \
      qemu-system-x86 \
      qemu-utils \
      ovmf \
      mtools \
      dosfstools \
      xorriso \
      nasm \
      lld
```

### Userspace Binary Building

**CRITICAL**: Before running kernel tests that execute userspace code, userspace binaries must be built:

```yaml
- name: Build userspace tests
  run: |
    export PATH="$PATH:$(rustc --print sysroot)/lib/rustlib/x86_64-unknown-linux-gnu/bin"
    cd userspace/programs
    ./build.sh
```

Forgetting this step causes kernel tests to fail mysteriously!

### Using xtask for Tests

Breenix uses the `xtask` pattern for complex test workflows:

```yaml
- name: Run Ring-3 smoke test
  run: cargo run -p xtask -- ring3-smoke
```

This handles:
- Building the kernel with correct features
- Starting QEMU with appropriate flags
- Monitoring serial output for success signals
- Timeout management (30s local, 60s CI)
- Cleanup and artifact collection

## Timeout Strategies

Different workflows need different timeouts:

```yaml
jobs:
  quick-test:
    timeout-minutes: 20    # Build + simple smoke test

  full-integration:
    timeout-minutes: 45    # Complete test suite with shared QEMU

  code-quality:
    timeout-minutes: 15    # Clippy and static analysis
```

**Rule of thumb**: CI environments are 2-3x slower than local development machines. Budget accordingly.

## Caching for Performance

Proper caching reduces build times from ~10 minutes to ~2 minutes:

```yaml
- name: Cache cargo
  uses: actions/cache@v4
  with:
    path: |
      ~/.cargo/registry
      ~/.cargo/git
      target
    key: cargo-${{ runner.os }}-${{ hashFiles('**/Cargo.lock') }}
```

Cache invalidates when `Cargo.lock` changes, ensuring fresh builds for dependency updates.

## Log Artifact Upload

Always upload logs, especially on failure:

```yaml
- name: Upload logs
  if: always()        # Run even if previous steps failed
  uses: actions/upload-artifact@v4
  with:
    name: breenix-logs
    path: |
      logs/*.log
      target/xtask_ring3_smoke_output.txt
      target/xtask_ring3_enosys_output.txt
    if-no-files-found: ignore
    retention-days: 7
```

This enables post-mortem analysis of failed runs.

## Workflow Patterns

### Pattern 1: Smoke Test (Fast Feedback)

Runs on every push to provide quick feedback:

```yaml
name: Ring-3 Smoke Test

on:
  push:
    branches: [ "**" ]
  pull_request:

jobs:
  ring3-smoke:
    runs-on: ubuntu-latest
    timeout-minutes: 20

    steps:
      - uses: actions/checkout@v4
      - name: Install Rust
        # ... (see above)
      - name: Cache cargo
        # ... (see above)
      - name: Install system dependencies
        # ... (see above)
      - name: Build userspace tests
        # ... (see above)
      - name: Run smoke test
        run: cargo run -p xtask -- ring3-smoke
      - name: Upload logs
        if: always()
        # ... (see above)
```

**Purpose**: Verify basic kernel functionality (boot, userspace execution) quickly.

### Pattern 2: Code Quality (Static Analysis)

Runs on kernel code changes:

```yaml
name: Code Quality

on:
  push:
    paths:
      - 'kernel/**'
      - '.github/workflows/code-quality.yml'

jobs:
  clippy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install Rust
        with:
          components: clippy, rust-src
      - name: Run Clippy
        run: |
          cd kernel
          cargo clippy --target x86_64-unknown-none \
            -- -Dclippy::debug_assert_with_mut_call
```

**Purpose**: Catch code quality issues before they reach main branch.

### Pattern 3: Manual Trigger (Expensive Tests)

For tests that take too long to run on every commit:

```yaml
name: Full Integration Tests

on:
  workflow_dispatch:   # Only run manually

jobs:
  integration:
    timeout-minutes: 45
    # ... full test suite
```

**Purpose**: Comprehensive testing before releases or major merges.

## Reference Files

The skill includes reference material in the `references/` directory:

- **`breenix-ci-patterns.md`**: Comprehensive CI patterns, timeout strategies, caching, success signals

To reference these during workflow authoring:
```bash
cat github-workflow-authoring/references/breenix-ci-patterns.md
```

## Workflow Creation Process

When creating a new workflow:

1. **Identify the test type**: Smoke test, integration test, static analysis, etc.
2. **Determine trigger**: Every push, PR only, manual, or path-specific
3. **Set appropriate timeout**: Based on test complexity and CI overhead
4. **Copy template from existing workflow**: Start with `ring3-smoke.yml` or `code-quality.yml`
5. **Customize steps**: Add specific build steps, test commands, or checks
6. **Add caching**: Use cargo cache unless testing cache-sensitive issues
7. **Configure artifact upload**: Always include logs for debugging
8. **Test locally first**: Use the same commands that will run in CI
9. **Add to PR**: Update .github/workflows/ and create PR for review
10. **Monitor first runs**: Watch for environment-specific issues

## Common Workflow Issues and Fixes

### Issue: "rustup: command not found"
**Cause**: Rust toolchain not installed or wrong action version
**Fix**: Use `actions-rs/toolchain@v1` or `dtolnay/rust-toolchain`

### Issue: "error: target 'x86_64-unknown-none' may not be installed"
**Cause**: Missing custom target
**Fix**: Add `target: x86_64-unknown-none` to toolchain setup

### Issue: "error: could not compile `bootloader`"
**Cause**: Missing `rust-src` component
**Fix**: Add `rust-src` to components list

### Issue: "qemu-system-x86_64: command not found"
**Cause**: QEMU not installed in CI environment
**Fix**: Add qemu-system-x86 to apt-get install list

### Issue: Test times out but works locally
**Cause**: CI environment slower, or test hung
**Fix**: Increase timeout-minutes or investigate kernel hang

### Issue: Cache seems corrupted
**Cause**: Cache key collision or partial build artifacts
**Fix**: Change cache key (add version suffix) or clear cache

### Issue: Userspace test fails with "file not found"
**Cause**: Userspace binaries not built before kernel test
**Fix**: Add userspace build step before kernel test runs

## Advanced Patterns

### Matrix Builds (Future)

Test multiple configurations:

```yaml
strategy:
  matrix:
    mode: [uefi, bios]
    features: [testing, production]

steps:
  - run: cargo run --bin qemu-${{ matrix.mode }} --features ${{ matrix.features }}
```

### Conditional Steps

Skip steps based on conditions:

```yaml
- name: Upload logs
  if: failure()   # Only on failure
  # or
  if: always()    # Always run
  # or
  if: success()   # Only on success
```

### Environment-Specific Behavior

```yaml
env:
  CI: true
  RUST_BACKTRACE: full
  BREENIX_TIMEOUT: 60    # Used by xtask
```

## Best Practices

1. **Pin Rust version**: Avoid unexpected breakage from nightly changes
2. **Cache aggressively**: Cargo builds are slow, caching saves 5-8 minutes
3. **Fail fast**: Set reasonable timeouts to avoid wasting CI minutes
4. **Upload artifacts**: Always capture logs for post-mortem analysis
5. **Test locally first**: Run the exact commands that CI will run
6. **Use xtask**: Complex test logic belongs in xtask, not YAML
7. **Monitor CI time**: If workflows exceed 20-30 minutes, consider splitting
8. **Document workflows**: Add comments explaining non-obvious steps

## Integration with Breenix Development

When adding new kernel features that require CI testing:

1. **Identify test requirements**: What needs to be verified?
2. **Create or extend xtask**: Add new test command (e.g., `ring3-fork-test`)
3. **Add workflow**: Either extend existing or create new workflow file
4. **Add success signal**: Add kernel log marker for test completion
5. **Update workflow docs**: Document the new test in CLAUDE.md or README

## Example: Adding a New Test Workflow

Let's say you want to add CI for testing the fork() syscall:

```yaml
name: Fork System Call Test

on:
  push:
    paths:
      - 'kernel/src/process/**'
      - 'kernel/src/syscall/**'
      - 'userspace/programs/fork_test*'

jobs:
  fork-test:
    runs-on: ubuntu-latest
    timeout-minutes: 25

    steps:
      - uses: actions/checkout@v4

      - name: Install Rust
        uses: actions-rs/toolchain@v1
        with:
          toolchain: nightly-2025-06-24
          override: true
          target: x86_64-unknown-none
          components: rust-src, llvm-tools-preview

      - name: Cache cargo
        uses: actions/cache@v4
        with:
          path: |
            ~/.cargo/registry
            ~/.cargo/git
            target
          key: cargo-${{ runner.os }}-fork-${{ hashFiles('**/Cargo.lock') }}

      - name: Install dependencies
        run: |
          sudo apt-get update
          sudo apt-get install -y qemu-system-x86 ovmf nasm

      - name: Build userspace tests
        run: |
          export PATH="$PATH:$(rustc --print sysroot)/lib/rustlib/x86_64-unknown-linux-gnu/bin"
          cd userspace/programs
          ./build.sh

      - name: Run fork test
        run: cargo run -p xtask -- fork-test

      - name: Upload logs
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: fork-test-logs
          path: |
            logs/*.log
            target/xtask_fork_test_output.txt
          retention-days: 7
```

Then update `xtask/src/main.rs`:

```rust
#[derive(StructOpt)]
enum Cmd {
    Ring3Smoke,
    Ring3Enosys,
    ForkTest,  // New test
}

fn main() -> Result<()> {
    match Cmd::from_args() {
        Cmd::Ring3Smoke => ring3_smoke(),
        Cmd::Ring3Enosys => ring3_enosys(),
        Cmd::ForkTest => fork_test(),
    }
}

fn fork_test() -> Result<()> {
    // Similar to ring3_smoke but looks for fork-specific signals
    // ...
}
```

## Troubleshooting CI Failures

When a workflow fails:

1. **Download artifacts**: Get the log files from the Actions run
2. **Search for errors**: Look for panic, double fault, timeout messages
3. **Compare with local**: Run the exact same command locally
4. **Check environment**: Verify Rust version, QEMU version, dependencies
5. **Reproduce in clean environment**: Use Docker or fresh VM if needed
6. **Use ci-failure-analysis skill**: Systematic analysis of CI failures

## Summary

GitHub workflow authoring for Breenix requires understanding:
- Rust nightly toolchain with custom targets
- QEMU-based kernel testing patterns
- xtask for test orchestration
- Timeout management for CI environments
- Caching strategies for performance
- Log artifact collection for debugging

Always reference the existing workflows as templates, test locally before committing, and leverage the xtask pattern for complex test logic.
