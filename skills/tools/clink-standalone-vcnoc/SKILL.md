---
name: clink-standalone
description: "Standalone CLI bridge - launch external AI CLIs (gemini, codex, claude) directly without MCP server. Use when you need to delegate tasks to specialized CLI tools with their own context windows. Supports role-based prompts and file references."
---

# Clink Standalone - CLI Bridge Skill (No MCP Required)

## Overview

This skill provides a **standalone** interface to launch external AI CLI tools (gemini, codex, claude) **without requiring an MCP server**. It runs as a local Python script that directly executes CLI commands.

**Key Benefits:**
- **No MCP Server Needed**: Runs standalone as a Python script
- **Isolated Context**: Fresh context window for each CLI
- **Full CLI Capabilities**: Web search, file tools, native features
- **Role-based Prompts**: Pre-configured personas (default, planner, codereviewer)

## Prerequisites

Before using this skill, install the CLIs you want to use:

```bash
# Gemini CLI (Google)
npm install -g @google/gemini-cli
gemini auth login

# Codex CLI (Sourcegraph)
# Visit https://docs.sourcegraph.com/codex

# Claude Code (Anthropic)
# Visit https://www.anthropic.com/claude-code
```

## Installation

1. **Copy the skill to your Claude skills directory:**

```bash
cp -r vc/clink-standalone ~/.claude/skills/clink-standalone
```

2. **Install Python dependencies:**

```bash
pip install pydantic
```

## Usage

### Basic Usage

```bash
# Run from the skill directory
cd ~/.claude/skills/clink-standalone
python bin/clink.py <cli_name> "<prompt>"
```

### Examples

```bash
# Ask Gemini a question
python bin/clink.py gemini "Explain async/await in Python"

# Use Codex for code review
python bin/clink.py codex "Review this code" --files src/auth.py

# Use planner role
python bin/clink.py gemini "Plan a microservices migration" --role planner

# Output as JSON
python bin/clink.py gemini "What is Rust?" --json

# List available CLIs
python bin/clink.py --list-clients

# List roles for a CLI
python bin/clink.py --list-roles gemini
```

### In Claude Code

When using this skill in Claude Code, Claude will execute the clink script:

```
User: "Use gemini to explain Rust ownership"

Claude will run:
python bin/clink.py gemini "Explain Rust ownership system"
```

## Available CLIs and Roles

| CLI | Install | Strengths | Roles |
|-----|---------|-----------|-------|
| **gemini** | `npm install -g @google/gemini-cli` | 1M context, web search | default, planner, codereviewer |
| **codex** | Sourcegraph Codex | Code analysis, review | default, planner, codereviewer |
| **claude** | Claude Code | General purpose | default, planner, codereviewer |

## Role Definitions

| Role | Purpose | Best For |
|------|---------|----------|
| `default` | General tasks | Questions, summaries, quick answers |
| `planner` | Strategic planning | Multi-phase plans, architecture, migrations |
| `codereviewer` | Code analysis | Security review, quality checks, bug hunting |

## Command Reference

```
python bin/clink.py <cli_name> <prompt> [OPTIONS]

Options:
  --role, -r       Role to use (default: default)
  --files, -f      File paths to reference
  --images, -i     Image paths to include
  --config-dir     Custom config directory
  --json           Output as JSON
  --list-clients   List available CLIs
  --list-roles     List roles for a CLI
```

## Directory Structure

```
clink-standalone/
├── bin/
│   └── clink.py           # Main CLI script
├── clink_core/
│   ├── __init__.py
│   ├── models.py          # Pydantic models
│   ├── registry.py        # Config loader
│   └── runner.py          # CLI execution
├── config/
│   ├── gemini.json        # Gemini CLI config
│   ├── codex.json         # Codex CLI config
│   └── claude.json        # Claude CLI config
├── systemprompts/
│   ├── gemini/
│   ├── codex/
│   └── claude/
└── SKILL.md               # This file
```

## Configuration

CLI configurations are in `config/*.json`:

```json
{
  "name": "gemini",
  "command": "gemini",
  "additional_args": ["--telemetry", "false", "--yolo", "-o", "json"],
  "timeout_seconds": 300,
  "roles": {
    "default": {"prompt_path": "systemprompts/gemini/default.txt"},
    "planner": {"prompt_path": "systemprompts/gemini/planner.txt"},
    "codereviewer": {"prompt_path": "systemprompts/gemini/codereviewer.txt"}
  }
}
```

Customize by editing these files.

## System Prompts

Role-specific prompts are in `systemprompts/<cli>/<role>.txt`. Edit these to customize behavior.

## Error Handling

### CLI Not Found
```
Error: Executable 'gemini' not found in PATH
```
**Solution**: Install the CLI first (see Prerequisites)

### Timeout
```
Error: CLI 'gemini' timed out after 300 seconds
```
**Solution**: Increase `timeout_seconds` in config or break into smaller tasks

### Invalid Output
```
Output was 75000 characters, exceeding limit
```
**Solution**: Narrow your prompt or request a summary

## Best Practices

1. **Choose the Right CLI**
   - Large context → gemini
   - Code tasks → codex
   - General tasks → claude

2. **Use Appropriate Roles**
   - Strategic work → planner
   - Code review → codereviewer
   - Everything else → default

3. **File References**
   - Pass file paths via `--files`, CLI reads what it needs
   - More efficient than embedding full content

4. **Break Down Large Tasks**
   - If timeout occurs, split into smaller subtasks

## Python API

You can also use clink as a Python module:

```python
from clink_core import get_registry, run_cli

# Get registry
registry = get_registry()

# Get CLI and role
client = registry.get_client("gemini")
role = client.get_role("default")

# Run
result = run_cli(
    client=client,
    role=role,
    prompt="Explain async/await in Python",
    files=["/path/to/file.py"],
)

print(result.content)
print(result.metadata)
```

## License

This is a standalone extraction of the clink functionality from zen-mcp-server.
