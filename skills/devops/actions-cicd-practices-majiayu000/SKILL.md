---
name: actions-cicd-practices
description: GitHub Actions and CI/CD best practices for automated testing, building, and deployment.
---

# GitHub Actions CI/CD practices

## Purpose

Guide for GitHub Actions and CI/CD workflows covering testing, building, caching, and deployment automation.

## When to use

This skill activates when:

- Creating GitHub Actions workflows
- Setting up CI/CD pipelines
- Configuring automated testing
- Optimizing workflow performance
- Managing secrets and environments

## Core structure

### Basic workflow

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.14'

      - name: Install dependencies
        run: pip install -e ".[dev]"

      - name: Run tests
        run: pytest
```

## Python workflows

### With uv

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install uv
        uses: astral-sh/setup-uv@v4
        with:
          enable-cache: true

      - name: Set up Python
        run: uv python install 3.14

      - name: Install dependencies
        run: uv sync

      - name: Run tests
        run: uv run pytest
```

### Matrix testing

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
        python-version: ['3.14']

    steps:
      - uses: actions/checkout@v4

      - name: Install uv
        uses: astral-sh/setup-uv@v4

      - name: Set up Python ${{ matrix.python-version }}
        run: uv python install ${{ matrix.python-version }}

      - name: Install dependencies
        run: uv sync

      - name: Run tests
        run: uv run pytest
```

## Caching

### uv cache

```yaml
- name: Install uv
  uses: astral-sh/setup-uv@v4
  with:
    enable-cache: true
    cache-dependency-glob: "uv.lock"
```

### Manual caching

```yaml
- name: Cache dependencies
  uses: actions/cache@v4
  with:
    path: ~/.cache/pip
    key: ${{ runner.os }}-pip-${{ hashFiles('**/requirements.txt') }}
    restore-keys: |
      ${{ runner.os }}-pip-
```

## Linting and type checking

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install uv
        uses: astral-sh/setup-uv@v4
        with:
          enable-cache: true

      - name: Set up Python
        run: uv python install 3.14

      - name: Install dependencies
        run: uv sync

      - name: Lint with ruff
        run: uv run ruff check .

      - name: Type check with basedpyright
        run: uv run basedpyright
```

## Code coverage

```yaml
- name: Run tests with coverage
  run: uv run pytest --cov=src --cov-report=xml

- name: Upload coverage
  uses: codecov/codecov-action@v4
  with:
    files: ./coverage.xml
    fail_ci_if_error: true
```

## Workflow optimization

### Concurrency

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

### Path filters

```yaml
on:
  push:
    paths:
      - 'src/**'
      - 'tests/**'
      - 'pyproject.toml'
      - '.github/workflows/ci.yml'
```

### Job dependencies

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps: ...

  test:
    needs: lint
    runs-on: ubuntu-latest
    steps: ...

  build:
    needs: [lint, test]
    runs-on: ubuntu-latest
    steps: ...
```

## Secrets and environments

### Using secrets

```yaml
- name: Deploy
  env:
    API_KEY: ${{ secrets.API_KEY }}
  run: ./deploy.sh
```

### Environment protection

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Deploy to production
        run: ./deploy.sh
```

## Release workflow

```yaml
name: Release

on:
  release:
    types: [published]

jobs:
  publish:
    runs-on: ubuntu-latest
    permissions:
      id-token: write  # For PyPI trusted publishing

    steps:
      - uses: actions/checkout@v4

      - name: Install uv
        uses: astral-sh/setup-uv@v4

      - name: Build package
        run: uv build

      - name: Publish to PyPI
        uses: pypa/gh-action-pypi-publish@release/v1
```

## Reusable workflows

### Define reusable workflow

```yaml
# .github/workflows/test-reusable.yml
name: Reusable Test

on:
  workflow_call:
    inputs:
      python-version:
        required: true
        type: string

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up Python ${{ inputs.python-version }}
        uses: actions/setup-python@v5
        with:
          python-version: ${{ inputs.python-version }}
```

### Use reusable workflow

```yaml
jobs:
  test-3-11:
    uses: ./.github/workflows/test-reusable.yml
    with:
      python-version: '3.14'
```

## Security

### Minimal permissions

```yaml
permissions:
  contents: read
  pull-requests: write
```

### Pin action versions

```yaml
# Good: Pinned to specific version
- uses: actions/checkout@v4

# Better: Pinned to commit SHA
- uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11
```

## Checklist

- [ ] Workflow triggers appropriate
- [ ] Matrix covers required platforms/versions
- [ ] Caching configured for performance
- [ ] Secrets not exposed in logs
- [ ] Permissions minimized
- [ ] Action versions pinned
- [ ] Concurrency configured
- [ ] Path filters for efficiency

---

**Additional resources:**

- [GitHub Actions documentation](https://docs.github.com/en/actions)
- [Workflow syntax](https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions)
