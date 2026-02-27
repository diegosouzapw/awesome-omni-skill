---
name: agent-tools
short: Claude Code tool permissions reference
description: Reference for configuring tool permissions when launching Claude Code agents. Use when setting up --allowedTools flags, restricting file access, or configuring agent permissions.
when: User wants to configure agent permissions, restrict tool access, set up --allowedTools flags, or understand Claude Code CLI tool restrictions
---

# Claude Code Tools Reference

Configure tool permissions when launching parallel Claude Code agents.

## Available Tools

| Tool | Description | Use Case |
|------|-------------|----------|
| `Read` | Read files | Always needed for context |
| `Write` | Create new files | Creating new code files |
| `Edit` | Modify existing files | Updating existing code |
| `Bash` | Execute shell commands | Running tests, builds, git |
| `Glob` | Find files by pattern | File discovery |
| `Grep` | Search file contents | Code search |
| `WebFetch` | Fetch web content | Documentation lookup |
| `WebSearch` | Search the web | Research |
| `TodoWrite` | Manage task lists | Progress tracking |
| `Task` | Launch sub-agents | Delegation |
| `NotebookEdit` | Edit Jupyter notebooks | Data science |
| `mcp__<server>` | MCP server tools | External integrations |

## CLI Syntax

Each tool is a separate quoted argument:

```bash
claude --allowedTools "Tool1" "Tool2" "Tool3(...)" --print "prompt"
```

Example with multiple tools:

```bash
claude --allowedTools "Read" "Edit" "Bash(pytest:*)" --print "implement feature"
```

## Path-Specific Restrictions

Restrict file operations to specific directories using gitignore-style patterns.

### Path Pattern Syntax

| Pattern | Meaning | Example |
|---------|---------|---------|
| `//path` | Absolute filesystem path | `Edit(//Users/alice/src/**)` |
| `~/path` | Home directory relative | `Read(~/.zshrc)` |
| `/path` | Relative to settings file | `Edit(/src/**/*.ts)` |
| `path` | Relative to current directory | `Read(src/**)` |

### Examples

```bash
# Allow editing only in src/ directory
claude --allowedTools "Edit(/src/**)" --print "..."

# Allow editing TypeScript files only
claude --allowedTools "Edit(/src/**/*.ts)" --print "..."

# Multiple path restrictions
claude --allowedTools "Read" "Edit(/apps/users/**)" "Edit(/tests/**)" --print "..."

# Absolute path restriction
claude --allowedTools "Edit(//tmp/scratch.txt)" --print "..."
```

## Bash Command Restrictions

Restrict which shell commands can be executed using prefix matching.

### Syntax

```bash
Bash(command:*)
```

The `:*` wildcard only works at the **END** of patterns (prefix matching).

### Pattern Examples

| Pattern | Matches | Does NOT Match |
|---------|---------|----------------|
| `Bash(pytest:*)` | `pytest`, `pytest apps/` | `python -m pytest` |
| `Bash(npm run test:*)` | `npm run test`, `npm run test:unit` | `npm run build` |
| `Bash(git log:*)` | `git log --oneline` | `git commit` |
| `Bash(git status:*)` | `git status` | `git push` |
| `Bash(mypy:*)` | `mypy apps/` | `python -m mypy` |
| `Bash(ruff:*)` | `ruff check .` | `python -m ruff` |

### Example

```bash
claude --allowedTools "Bash(pytest:*)" "Bash(mypy:*)" "Bash(ruff:*)" "Read" --print "run tests"
```

### Security Note

Claude Code prevents bypass via shell operators (`&&`, `;`, `||`). Be aware:
- Different invocations may bypass patterns (`python -m pytest` vs `pytest`)
- For URL restrictions, prefer `WebFetch(domain:...)` over `Bash(curl:*)`

## WebFetch Domain Restrictions

Restrict web fetches to specific domains:

```bash
claude --allowedTools "WebFetch(domain:github.com)" "WebFetch(domain:docs.python.org)" --print "..."
```

## MCP Tool Restrictions

### Allow All Tools from a Server

```bash
claude --allowedTools "mcp__puppeteer" --print "..."
```

### Allow Specific Tool Only

```bash
claude --allowedTools "mcp__puppeteer__puppeteer_navigate" --print "..."
```

**Note:** MCP permissions do NOT support wildcards (`*`).

## Recommended Configurations

### By Task Type

| Task Type | Recommended `--allowedTools` |
|-----------|------------------------------|
| **Implementation** | `"Read" "Write" "Edit(/apps/myapp/**)" "Bash(pytest:*)" "Bash(mypy:*)" "Glob" "Grep"` |
| **Code Review** | `"Read" "Glob" "Grep"` (read-only) |
| **Testing Only** | `"Read" "Bash(pytest:*)" "Bash(npm test:*)"` |
| **Documentation** | `"Read" "Write(/docs/**)" "Edit(/docs/**)" "WebFetch"` |
| **Full Access** | `--dangerously-skip-permissions` |

### For Parallel Development

When using git worktrees for isolation, `--dangerously-skip-permissions` is safe:
- Each agent runs in an isolated worktree
- Agents can only affect files in their workspace
- Main branch remains protected until explicit merge

```bash
# Safe in isolated worktree
claude --dangerously-skip-permissions --print "$(cat prompts/task-001.txt)"
```

### For Granular Control

When agents share a workspace, use path-scoped permissions:

```bash
claude \
  --allowedTools \
    "Read" \
    "Write(/apps/users/**)" \
    "Edit(/apps/users/**)" \
    "Bash(pytest apps/users/:*)" \
    "Bash(mypy apps/users/:*)" \
    "Glob" \
    "Grep" \
  --print "$(cat prompts/task-001.txt)"
```

## Complete Examples

### Django App Implementation Agent

```bash
claude \
  --allowedTools \
    "Read" \
    "Write(/apps/orders/**)" \
    "Edit(/apps/orders/**)" \
    "Bash(pytest apps/orders/:*)" \
    "Bash(mypy apps/orders/:*)" \
    "Bash(ruff check apps/orders/:*)" \
    "Glob" \
    "Grep" \
  --print "Implement order management per task-004 spec"
```

### React Component Agent

```bash
claude \
  --allowedTools \
    "Read" \
    "Write(/src/components/Dashboard/**)" \
    "Edit(/src/components/Dashboard/**)" \
    "Bash(npm run test:*)" \
    "Bash(npm run lint:*)" \
    "Glob" \
    "Grep" \
  --print "Implement Dashboard components per task-003 spec"
```

### Read-Only Analysis Agent

```bash
claude \
  --allowedTools \
    "Read" \
    "Glob" \
    "Grep" \
    "WebFetch(domain:docs.python.org)" \
  --print "Analyze codebase and suggest improvements"
```

## Quick Reference

| Restriction Type | Syntax |
|-----------------|--------|
| Allow tool everywhere | `"Edit"` |
| Restrict to directory | `"Edit(/src/**)"` |
| Restrict to file type | `"Edit(/src/**/*.ts)"` |
| Restrict bash command | `"Bash(pytest:*)"` |
| Restrict web domain | `"WebFetch(domain:github.com)"` |
| Allow MCP server | `"mcp__puppeteer"` |
| Allow specific MCP tool | `"mcp__puppeteer__puppeteer_navigate"` |
| Skip all permissions | `--dangerously-skip-permissions` |

## Common Patterns

### Task-Scoped Permissions

Match permissions to task boundaries:

```bash
# Task owns apps/users/
--allowedTools "Edit(/apps/users/**)" "Write(/apps/users/**)"

# Task owns apps/orders/
--allowedTools "Edit(/apps/orders/**)" "Write(/apps/orders/**)"
```

### Test Commands Only

```bash
--allowedTools "Read" "Bash(pytest:*)" "Bash(npm test:*)" "Bash(go test:*)"
```

### Documentation Writer

```bash
--allowedTools "Read" "Write(/docs/**)" "Edit(/docs/**)" "WebFetch" "WebSearch"
```

### Infrastructure Agent

```bash
--allowedTools "Read" "Edit(/terraform/**)" "Edit(/docker-compose.yml)" "Bash(terraform:*)" "Bash(docker:*)"
```
