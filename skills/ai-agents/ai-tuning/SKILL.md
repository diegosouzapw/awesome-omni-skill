---
name: ai-tuning
description: Optimize AI assistant configurations for maximum effectiveness. USE THIS SKILL when user says "improve CLAUDE.md", "better copilot instructions", "tune AI", "optimize prompts", "MCP configuration", or wants to enhance AI assistant behavior.
user-invocable: true
allowed-tools: Bash Read Write Edit Glob Grep
---

# AI Tuning Skill

## Purpose

Optimize AI assistant configurations for maximum effectiveness, including CLAUDE.md, copilot-instructions.md, and MCP server setup.

## Triggers

- "improve my CLAUDE.md"
- "better copilot instructions"
- "tune AI for this project"
- "add MCP servers"
- "optimize AI prompts"

## Usage

Provide your existing AI configuration files or describe your project, and this skill will generate optimized CLAUDE.md, copilot-instructions.md, and MCP configurations following best practices.

## Effective AI Instructions Principles

1. **Be Specific**: Vague → vague results
2. **Show Examples**: Code > descriptions
3. **State Constraints**: What NOT to do
4. **Organize Hierarchically**: General → specific
5. **Include Commands**: Quick reference

## CLAUDE.md Structure

```markdown
# CLAUDE.md

## Project Overview
[What, architecture, technologies]

## Project Structure
```
[directory tree]
```

## Build Commands
```bash
# Install
[command]

# Test
[command]

# Lint
[command]
```

## Code Style Requirements
[Formatter, linter, key rules with examples]

## Architecture Guidelines
[Patterns, layer rules]

## Important Patterns
[Code examples]
```

## copilot-instructions.md Structure

```markdown
# GitHub Copilot Instructions

## Project Context
[Brief description, tech stack]

## Code Generation Guidelines

### [Language] Patterns
[Conventions, type annotations, imports]

### Examples
```[language]
// GOOD
[example]

// AVOID
[counter-example]
```

## Common Patterns
[Reusable code]

## Commands
[Quick reference]
```

## MCP Configuration

```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@context7/mcp-server"]
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "${workspaceFolder}"]
    },
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    }
  }
}
```

## Optimization Techniques

### 1. Context Density

```markdown
# Instead of verbose:
"We use Python 3.12 as the language.
We use ruff for linting.
Testing is done with pytest."

# Write dense:
"Python 3.12 | ruff (lint+format) | pytest"
```

### 2. Example-Driven

```markdown
# Instead of:
"Use type annotations."

# Write:
"Type annotations required:
```python
def process(items: list[str]) -> dict[str, int]: ...
```"
```

### 3. Constraints Section

```markdown
## Constraints
- Max line length: 100 chars
- No star imports
- Error messages assigned to variables
- All public functions need docstrings
```

### 4. Command Quick Reference

```markdown
| Action | Command |
|--------|---------|
| Test | `uv run pytest` |
| Lint | `uv run ruff check .` |
| Format | `uv run ruff format .` |
```

## Validation

```bash
# Check AI files exist
[ -f "CLAUDE.md" ] && echo "✓ CLAUDE.md"
[ -f ".github/copilot-instructions.md" ] && echo "✓ Copilot"
[ -f ".vscode/mcp.json" ] && echo "✓ MCP"

# Check sections in CLAUDE.md
grep "^## " CLAUDE.md

# Check code examples
grep -c '```' CLAUDE.md
```

## Quality Metrics

| Metric | Target |
|--------|--------|
| Has examples | Yes (3+ code blocks) |
| Has commands | Yes |
| Organized | Yes (## headers) |
| Specific | No vague terms |
| Current | Tool versions updated |

## Pattern Library Development

Include reusable patterns:

```python
# Pattern: Error Handling
def fetch(id: str) -> User:
    """Fetch user by ID.

    Raises:
        UserNotFoundError: If not found.
    """
    result = db.query(User).filter_by(id=id).first()
    if result is None:
        raise UserNotFoundError(f"User {id} not found")
    return result
```

## AI File Audit

```bash
echo "=== AI Configuration Audit ==="

for f in CLAUDE.md .github/copilot-instructions.md .vscode/mcp.json; do
  if [ -f "$f" ]; then
    echo "✓ $f exists ($(wc -l < "$f") lines)"
  else
    echo "✗ $f MISSING"
  fi
done
```
