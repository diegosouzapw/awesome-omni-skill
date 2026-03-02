---
name: sv-dev
description: Development workflow for Security Verifiers. Use when asked to run tests, lint code, format files, set up the development environment, or perform CI checks on the codebase.
metadata:
  author: security-verifiers
  version: "1.0"
---

# Security Verifiers Development

Testing, linting, formatting, and development workflow for the Security Verifiers codebase.

## Quick Start

```bash
# Full setup (creates venv, installs all deps)
make setup
source .venv/bin/activate

# Run all checks
make check  # lint + format + test
```

## Testing

### Run All Tests

```bash
make test
```

### Test Specific Environment

```bash
make test-env E=network-logs
make test-env E=config-verification
make test-env E=code-vulnerability
make test-env E=phishing-detection
make test-env E=redteam-attack
make test-env E=redteam-defense
```

### Environment Shortcuts

```bash
make e1  # test network-logs
make e2  # test config-verification
make e3  # test code-vulnerability
make e4  # test phishing-detection
make e5  # test redteam-attack
make e6  # test redteam-defense
```

### Test Utils Package

```bash
make test-utils
```

### Test with Coverage

```bash
make test-cov
```

### Run Single Test

```bash
uv run pytest environments/sv-env-network-logs/sv_env_network_logs_test.py::TestNetworkLogParser::test_extracts_label_and_confidence -q
```

## Linting and Formatting

### Check Linting (no changes)

```bash
make lint
```

### Fix Linting Issues

```bash
make lint-fix
```

### Format Code

```bash
make format
```

### Quick Fix (lint + format)

```bash
make quick-fix
```

## Pre-commit Hooks

```bash
# Install and run hooks
make pre-commit
```

This runs:
- Ruff linting
- Ruff formatting
- Trailing whitespace removal
- End-of-file fixes

## CI Checks

Run the same checks as CI:

```bash
make ci
```

This runs:
- `ruff check . --exit-non-zero-on-fix`
- `pytest -q --tb=short`

## Environment Setup

### Full Setup

```bash
make setup
```

Creates `.venv`, installs all environments and dev tools.

### Manual Steps

```bash
make venv           # Create virtual environment
make install        # Install all environments
make install-dev    # Install dev tools (pytest, ruff, etc.)
```

### Install Security Tools (Ubuntu)

For E2 config-verification, install pinned tool versions:

```bash
make install-linux  # Installs kube-linter, opa, semgrep
make check-tools    # Verify versions match ci/versions.txt
```

## Project Structure

```
security-verifiers/
├── environments/          # Environment packages
│   ├── sv-env-network-logs/
│   ├── sv-env-config-verification/
│   └── ...
├── sv_shared/             # Shared utilities (security-verifiers-utils)
├── scripts/               # Build, eval, data scripts
├── outputs/               # Evaluation outputs
└── skills/                # Agent skills
```

## Common Workflows

### Before Committing

```bash
make check  # or make quick-check
```

### After Changing Environment Code

```bash
make test-env E=network-logs  # test specific env
make lint                      # check linting
```

### After Changing Shared Code

```bash
make test-utils
make test  # run all tests to check for regressions
```

## Cleaning Up

```bash
make clean           # Build artifacts and caches
make clean-outputs   # Eval outputs (preserves logs)
make clean-logs      # Log files only
make clean-all       # Everything including venv
```

## File Watch (requires entr)

```bash
# Install: brew install entr (macOS) or apt install entr (Ubuntu)
make watch
```

Automatically runs tests when Python files change.

## Environment Info

```bash
make info       # Show environment status
make list-envs  # List environment names
```

## Troubleshooting

**venv issues**: `make clean-all && make setup`
**Import errors**: Ensure `source .venv/bin/activate`
**Tool version mismatch**: `make check-tools` then `make install-linux`
**Pre-commit fails**: `make lint-fix && make format`
