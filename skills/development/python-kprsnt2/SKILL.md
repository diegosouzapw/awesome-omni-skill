---
name: python
description: Best practices for Python development including PEP 8, modern Python 3.10+ features, and package management.
globs: ["**/*.py", "**/Pipfile", "**/pyproject.toml", "**/setup.py", "**/requirements.txt"]
priority: 100
tags: ["language"]
---

# Python Best Practices

## Code Style
- Follow PEP 8 style guide
- Use snake_case for functions/variables
- Use PascalCase for classes
- Maximum line length: 88 (Black) or 79 (PEP 8)
- Use meaningful variable names

## Modern Python (3.10+)
- Use f-strings for formatting: f"Hello {name}"
- Use pathlib for file paths instead of os.path
- Use dataclasses or Pydantic for data models
- Type hints for function signatures
- Use walrus operator := for assignment expressions
- Use match-case for pattern matching
- Use | for union types instead of Union

## Error Handling
- Be specific with exception types
- Use context managers (with) for resources
- Log exceptions with traceback
- Never use bare except:
- Use finally for cleanup

## Package Management
- Always use virtual environments (venv, conda, poetry)
- Pin dependency versions in requirements.txt
- Use pyproject.toml for modern projects
- Separate dev dependencies from production

## Testing
- Use pytest over unittest
- Use fixtures for test setup
- Use parametrize for multiple test cases
- Aim for high test coverage
