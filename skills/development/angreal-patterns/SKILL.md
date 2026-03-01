---
name: angreal-patterns
description: This skill should be used when the user asks to "test angreal tasks", "mock angreal", "document tasks", "angreal best practices", "error handling in tasks", "subprocess patterns", "dry run mode", "verbose mode", or needs guidance on testing patterns, development workflows, documentation strategies, or common implementation patterns for angreal tasks.
version: 2.8.0
---

# Angreal Patterns

Common patterns for testing, documenting, and developing angreal tasks.

## Testing Patterns

### Unit Testing Task Functions

```python
# tests/test_build.py
import sys
sys.path.insert(0, ".angreal")
from task_build import build

def test_build_debug_mode():
    result = build(release=False)
    assert result == 0

def test_build_release_mode():
    result = build(release=True)
    assert result == 0
```

### Mocking External Commands

```python
from unittest.mock import patch, MagicMock

def test_run_tests_success():
    with patch("subprocess.run") as mock_run:
        mock_run.return_value = MagicMock(returncode=0)
        from task_test import run_tests
        result = run_tests()
        assert result == 0
```

### Mocking angreal.get_root()

```python
# tests/conftest.py
import pytest
from pathlib import Path

@pytest.fixture
def temp_project(tmp_path):
    """Create temporary project structure."""
    angreal_dir = tmp_path / ".angreal"
    angreal_dir.mkdir()
    (tmp_path / "src").mkdir()
    return tmp_path

@pytest.fixture
def mock_root(temp_project, monkeypatch):
    """Mock get_root() to return .angreal/ directory."""
    import angreal
    angreal_dir = temp_project / ".angreal"
    monkeypatch.setattr(angreal, "get_root", lambda: angreal_dir)
    return temp_project  # Return project root for assertions
```

### Testing Output

```python
def test_task_output(capsys):
    from task_status import status
    status()
    captured = capsys.readouterr()
    assert "Project Status" in captured.out
```

## Development Patterns

### Verbose Mode

```python
import angreal

@angreal.command(name="build", about="Build project")
@angreal.argument(name="verbose", short="v", long="verbose",
                  is_flag=True, takes_value=False)
def build(verbose=False):
    if verbose:
        print("Starting build...")

    do_build()

    if verbose:
        print("Build complete!")
```

### Quiet Mode

```python
@angreal.command(name="check", about="Run checks")
@angreal.argument(name="quiet", short="q", long="quiet",
                  is_flag=True, takes_value=False)
def check(quiet=False):
    issues = run_checks()

    if not issues:
        if not quiet:
            print("All checks passed!")
        return 0

    if not quiet:
        for issue in issues:
            print(f"  - {issue}")
    return 1
```

### Dry Run Mode

```python
import angreal
import shutil
import os

@angreal.command(name="clean", about="Clean build artifacts")
@angreal.argument(name="dry_run", short="n", long="dry-run",
                  is_flag=True, takes_value=False)
def clean(dry_run=False):
    project_root = angreal.get_root().parent
    targets = ["dist/", "build/", ".cache/"]

    for target in targets:
        path = project_root / target
        if path.exists():
            if dry_run:
                print(f"Would remove: {path}")
            else:
                print(f"Removing: {path}")
                shutil.rmtree(path)

    if dry_run:
        print("\nDry run - no changes made.")
```

### Progress Indicators

```python
@angreal.command(name="test", about="Run tests")
def test():
    tests = discover_tests()
    for i, test in enumerate(tests, 1):
        print(f"[{i}/{len(tests)}] Running {test}...")
        run_test(test)
    print("All tests complete!")
```

## Subprocess Patterns

### Running Commands in Project Root

```python
import subprocess
import angreal

def run_in_project(cmd, **kwargs):
    """Run command in project root."""
    project_root = angreal.get_root().parent
    defaults = {
        "cwd": project_root,
        "capture_output": True,
        "text": True
    }
    defaults.update(kwargs)
    return subprocess.run(cmd, **defaults)
```

### Streaming Output

```python
@angreal.command(name="test", about="Run tests")
def test():
    project_root = angreal.get_root().parent

    process = subprocess.Popen(
        ["pytest", "-v"],
        cwd=project_root,
        stdout=subprocess.PIPE,
        stderr=subprocess.STDOUT,
        text=True
    )

    for line in process.stdout:
        print(line, end="")

    return process.wait()
```

### Handling Failures

```python
def run_or_fail(cmd, error_msg):
    """Run command, exit on failure."""
    result = subprocess.run(cmd, capture_output=True, text=True)

    if result.returncode != 0:
        print(f"Error: {error_msg}")
        if result.stderr:
            print(result.stderr)
        return 1

    return 0
```

## Error Handling Patterns

### Fail Fast

```python
@angreal.command(name="deploy", about="Deploy application")
def deploy():
    # Check all prerequisites first
    if not check_credentials():
        print("Error: Missing credentials")
        return 1

    if not check_build_exists():
        print("Error: No build. Run 'angreal build' first.")
        return 1

    # Only proceed if everything ready
    do_deploy()
    return 0
```

### Informative Error Messages

```python
def validate_env(env):
    valid = ["development", "staging", "production"]

    if env not in valid:
        print(f"Error: Invalid environment '{env}'")
        print(f"Valid options: {', '.join(valid)}")
        return False

    return True
```

### Environment Variable Checking

```python
import os

@angreal.command(name="deploy", about="Deploy")
def deploy():
    required = ["AWS_ACCESS_KEY_ID", "AWS_SECRET_ACCESS_KEY"]
    missing = [v for v in required if not os.environ.get(v)]

    if missing:
        print("Error: Missing environment variables:")
        for var in missing:
            print(f"  - {var}")
        return 1

    do_deploy()
    return 0
```

## Documentation Patterns

### Documentation Layers

| Layer | Audience | Location |
|-------|----------|----------|
| `about` | CLI users | `--help` output |
| `long_about` | CLI users | Detailed help |
| `help` (args) | CLI users | Argument help |
| `ToolDescription` | AI agents | `angreal tree --long` |
| Docstrings | Developers | Source code |

### Consistent Structure

```python
@angreal.command(
    name="deploy",
    about="Deploy to environment",  # Short for listings
    long_about="""
    Deploy the application to a specified environment.

    Handles: building, uploading, migrations, health checks.

    Supported environments: development, staging, production
    """,  # Detailed for --help
    tool=angreal.ToolDescription("""
        Deploy application to environment.

        ## When to use
        - After tests pass
        - When release is approved

        ## Examples
        ```
        angreal deploy --env staging
        ```
        """, risk_level="destructive")  # For AI agents
)
def deploy():
    """Deploy application. (For developers reading code)"""
    pass
```

## Anti-Patterns to Avoid

### Don't Hardcode Paths

```python
# Bad
config = open("/Users/me/project/config.yaml")

# Good
project_root = angreal.get_root().parent
config = open(project_root / "config.yaml")
```

### Don't Ignore Return Codes

```python
# Bad
subprocess.run(["npm", "install"])
subprocess.run(["npm", "test"])  # Runs even if install failed

# Good
result = subprocess.run(["npm", "install"])
if result.returncode != 0:
    return 1
subprocess.run(["npm", "test"])
```

### Don't Swallow Exceptions

```python
# Bad
try:
    do_something()
except:
    pass

# Good
try:
    do_something()
except SpecificError as e:
    print(f"Error: {e}")
    return 1
```

### Don't Require Interactive Input

```python
# Bad - breaks CI/automation
name = input("Enter name: ")

# Good - use arguments
@angreal.argument(name="name", long="name", required=True)
def cmd(name):
    pass
```

## Composing Tasks

### Calling Other Task Functions

```python
from task_test import test_all
from task_lint import lint_check

@angreal.command(name="ci", about="Run CI pipeline")
def ci():
    print("Running linter...")
    if lint_check() != 0:
        return 1

    print("Running tests...")
    if test_all() != 0:
        return 1

    print("CI passed!")
    return 0
```

## pytest.ini Configuration

```ini
[pytest]
testpaths = tests
pythonpath = .angreal

markers =
    unit: Unit tests
    integration: Integration tests
```
