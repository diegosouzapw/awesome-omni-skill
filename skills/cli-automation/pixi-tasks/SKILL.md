---
name: pixi-tasks
description: Complex pixi task workflows and orchestration. Use when building task dependency chains, configuring caching with inputs/outputs, creating parameterized tasks, or setting up CI pipelines—e.g., "pixi task depends-on", "task caching for build automation", "multi-environment test matrices".
---

# Pixi Tasks

## Purpose

Complex task patterns: dependency chains, caching with inputs/outputs, cross-environment workflows.

## Task Basics

```toml
[tasks]
# Simple command
test = "pytest -v"

# With working directory
build = { cmd = "make", cwd = "src" }

# Multiple steps
setup = "mkdir -p build && cmake -B build"
```

## Task Dependencies

**YOU MUST use `depends-on` for execution order.** Tasks without explicit dependencies execute in undefined order and will fail in CI pipelines. No exceptions.

```toml
[tasks]
configure = "cmake -B build"
build = { cmd = "cmake --build build", depends-on = ["configure"] }
test = { cmd = "ctest", depends-on = ["build"] }
start = { cmd = "./build/app", depends-on = ["build"] }
```

```bash
pixi run start  # Runs: configure → build → start
```

### Diamond Pattern

Every time you have multiple upstream dependencies, use this pattern. Tasks with multiple `depends-on` entries wait for ALL dependencies to complete before executing.

```toml
[tasks]
fetch-data = "curl -o data.csv ..."
fetch-model = "curl -o model.pkl ..."
train = { cmd = "python train.py", depends-on = ["fetch-data", "fetch-model"] }
evaluate = { cmd = "python eval.py", depends-on = ["train"] }
```

Execution: fetch-data and fetch-model complete before train; order is dependency-driven.

## Task Caching

Cache results based on inputs/outputs:

```toml
[tasks]
# Cache by input file
run = { cmd = "python main.py", inputs = ["main.py"] }

# Cache by inputs and outputs
build = {
  cmd = "cargo build --release",
  inputs = ["src/**/*.rs", "Cargo.toml"],
  outputs = ["target/release/myapp"]
}

# Download caching (by output existence)
download = {
  cmd = "curl -o data.csv https://example.com/data.csv",
  outputs = ["data.csv"]
}
```

**Cache invalidates immediately when ANY of these conditions are met:**

- Input file changes (hash mismatch)
- Output file missing
- Command string changes
- Environment packages change

**Caches without proper inputs/outputs are always stale.** Define both or accept that your tasks will re-run unnecessarily.

## Platform-Specific Tasks

Define tasks that behave differently per platform using `[target.<platform>.tasks]`. Pixi automatically selects the correct implementation based on the current platform.

```toml
# Target-agnostic tasks that depend on platform-specific setup
task = { cmd = "echo 'Running main task'", depends-on = ["setup"] }

[target.linux-64.tasks]
setup = { cmd = "echo 'Linux setup'", description = "Setup for Linux" }

[target.osx-arm64.tasks]
setup = { cmd = "echo 'macOS setup'", description = "Setup for macOS" }
```

```bash
pixi run task  # Runs the correct 'setup' for your platform, then 'task'
```

### Platform-Specific Dependencies + Tasks

Combine with `[target.<platform>.dependencies]` to install packages only on specific platforms:

```toml
# Install stow only on Linux (via conda-forge)
[target.linux-64.dependencies]
stow = "*"

# Target-agnostic tasks
task = { cmd = "stow --version", depends-on = ["check-stow"] }

# Platform-specific setup tasks (same name, different implementations)
[target.linux-64.tasks]
check-stow = { cmd = "which stow", description = "Verify stow is available (pixi-installed)" }

[target.osx-arm64.tasks]
check-stow = { cmd = "which stow || (echo 'Install with: brew install stow' && exit 1)", description = "Verify stow is available (brew-installed)" }
```

**Key insight**: Tasks with the same name in different `[target.*.tasks]` sections are automatically resolved based on the current platform. Target-agnostic tasks can depend on these platform-specific tasks by name.

## Cross-Environment Tasks

Run tasks across multiple environments. **Always test in all target environments before committing.** Environment-specific failures in CI are expensive and avoidable.

```toml
[feature.py311.dependencies]
python = "3.11.*"

[feature.py312.dependencies]
python = "3.12.*"

[environments]
py311 = ["py311"]
py312 = ["py312"]

[tasks]
test = "pytest"

# Test in all Python versions
test-all = {
  depends-on = [
    { task = "test", environment = "py311" },
    { task = "test", environment = "py312" },
  ]
}
```

### Build/Test Separation

```toml
[feature.build.dependencies]
rust = ">=1.70"

[feature.build.tasks]
compile = "cargo build --release"

[feature.test.dependencies]
pytest = "*"

[feature.test.tasks]
test-python = "pytest"

[environments]
build = ["build"]
test = ["test"]

[tasks]
ci = {
  depends-on = [
    { task = "compile", environment = "build" },
    { task = "test-python", environment = "test" },
  ]
}
```

## Parameterized Tasks

Use MiniJinja templates:

```toml
[tasks]
process = {
  cmd = "python process.py inputs/{{ filename }}.txt",
  args = ["filename"],
  inputs = ["inputs/{{ filename }}.txt"],
  outputs = ["outputs/{{ filename }}.out"]
}
```

```bash
pixi run process data1
```

## CI/CD Patterns

### Full Pipeline

```toml
[tasks]
lint = "ruff check ."
format = "ruff format --check ."
typecheck = "mypy src/"

unit-test = { cmd = "pytest tests/unit", depends-on = ["lint", "format"] }
int-test = { cmd = "pytest tests/integration", depends-on = ["unit-test"] }

build = { cmd = "python -m build", depends-on = ["int-test", "typecheck"] }

ci = { depends-on = ["build"] }
```

### GitHub Actions

```yaml
- uses: prefix-dev/setup-pixi@v0.9.2
  with:
    cache: true
- run: pixi run ci
```

## References

- **[references/caching.md](references/caching.md)** - Load when tuning cache behavior or using inputs/outputs/globs.
- **[references/dependencies.md](references/dependencies.md)** - Load for complex dependency graphs and CI workflows.
- **[references/cross-environment.md](references/cross-environment.md)** - Load for multi-version testing and environment matrices.

## Do / Don't

**YOU MUST:**

- Verify task schema and behavior against Context7 or official pixi docs before implementing complex workflows
- Keep task graphs simple; move complex patterns to references

**NEVER:**

- Assume parallel execution without explicit documentation confirming it
- Use undocumented task keys in production workflows
- Skip environment testing before committing dependency changes
