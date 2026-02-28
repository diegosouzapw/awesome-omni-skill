---
name: claude-command-authoring
description: Creates custom slash commands for Claude Code with proper syntax, frontmatter, arguments, bash execution, and file references. Use when building slash commands, creating custom Claude Code commands, setting up team workflows, or when users mention slash commands, command files, or .md command creation.
version: 1.0.0
---

# Claude Command Authoring

Create custom slash commands that extend Claude Code with frequently-used prompts and workflows.

## Overview

Slash commands are Markdown files that define reusable prompts. They support:
- **Arguments**: `$1`, `$2`, `$ARGUMENTS`
- **Bash execution**: `!` prefix for command output
- **File references**: `@` prefix to include file contents
- **Frontmatter**: Metadata for configuration
- **Namespacing**: Organize in subdirectories

## Quick Start

### Basic Command

```bash
# Create a simple command
mkdir -p .claude/commands
cat > .claude/commands/review.md << 'EOF'
---
description: Review code for best practices and potential issues
---

Review the following code for:
- Code quality and readability
- Potential bugs or edge cases
- Performance considerations
- Security concerns
EOF

# Use it
/review
```

### Command with Arguments

```markdown
---
description: Fix a specific issue by number
argument-hint: <issue-number>
---

Fix issue #$1 following our coding standards and best practices.
Review the issue details, implement a fix, add tests, and create a commit.
```

### Command with Bash Execution

```markdown
---
description: Create a git commit from staged changes
allowed-tools: Bash(git *)
---

## Context

Current branch: !`git branch --show-current`
Staged changes: !`git diff --staged`
Recent commits: !`git log --oneline -5`

## Task

Create a single commit with a clear message based on the staged changes above.
```

### Command with File References

```markdown
---
description: Compare two implementations
argument-hint: <file1> <file2>
---

Compare these two implementations and explain the differences:

**File 1**: @$1
**File 2**: @$2

Provide a detailed comparison focusing on architecture, performance, and maintainability.
```

## Command Scopes

### Project Commands (`.claude/commands/`)

- Shared with your team via git
- Show "(project)" in `/help`
- Team-specific workflows

### Personal Commands (`~/.claude/commands/`)

- Available across all your projects
- Show "(user)" in `/help`
- Individual preferences

### Plugin Commands (`plugin/commands/`)

- Bundled with plugins
- Show "(plugin-name)" in `/help`
- Distributed via marketplaces

## Frontmatter Options

| Field | Purpose | Example |
|-------|---------|---------|
| `description` | Brief description shown in `/help` | `"Deploy to staging environment"` |
| `argument-hint` | Expected arguments for autocomplete | `"<environment> [--skip-tests]"` |
| `allowed-tools` | Restrict tool usage | `"Bash(git *), Read, Write"` |
| `model` | Specific model to use | `"claude-3-5-haiku-20241022"` |
| `disable-model-invocation` | Prevent SlashCommand tool usage | `true` |

## Argument Handling

### All Arguments (`$ARGUMENTS`)

```markdown
Fix issues: $ARGUMENTS
```

Usage: `/fix-issues 123 456 789` → `$ARGUMENTS = "123 456 789"`

### Individual Arguments (`$1`, `$2`, `$3`, ...)

```markdown
Review PR #$1 with priority $2 and assign to $3
```

Usage: `/review-pr 456 high alice` → `$1="456"`, `$2="high"`, `$3="alice"`

### Combining Arguments with Files

```markdown
Compare @$1 with @$2 and summarize differences
```

Usage: `/compare src/old.ts src/new.ts`

## Bash Execution with `!`

Execute bash commands before the prompt runs:

```markdown
---
description: Show current project status
---

## Git Status
!`git status`

## Recent Activity
!`git log --oneline -10`

## Uncommitted Changes
!`git diff`

Based on the above, provide a summary of the current project state.
```

**Output truncation**: Commands producing >15,000 chars are truncated by default.
Set `SLASH_COMMAND_TOOL_CHAR_BUDGET` environment variable to adjust.

## File References with `@`

Include file contents in your command:

```markdown
---
description: Explain a specific file
argument-hint: <file-path>
---

# File Analysis

**File**: @$1

Provide a detailed explanation of:
1. Purpose and responsibility
2. Key functions and methods
3. Dependencies and imports
4. Potential improvements
```

## Namespacing

Organize commands in subdirectories:

```
.claude/commands/
├── frontend/
│   ├── component.md      → /component (project:frontend)
│   └── styling.md        → /styling (project:frontend)
└── backend/
    ├── migration.md      → /migration (project:backend)
    └── testing.md        → /testing (project:backend)
```

Access: `/component` or `/frontend/component`

## Tool Restrictions

Limit what tools Claude can use:

```markdown
---
description: Safe code review (read-only)
allowed-tools: Read, Grep, Glob
---

Review the codebase for issues without making any changes.
```

Claude can only use specified tools without asking permission.

## Best Practices

### 1. Clear Descriptions

```markdown
---
# ❌ Too vague
description: Deploy stuff

# ✅ Specific and helpful
description: Deploy to staging with health checks and Slack notification
---
```

### 2. Helpful Argument Hints

```markdown
---
# ❌ Not helpful
argument-hint: args

# ✅ Clear expectations
argument-hint: <environment> [--skip-tests] [--no-notify]
---
```

### 3. Context Before Instructions

```markdown
---
description: Create feature branch from issue
---

## Context
Current branch: !`git branch --show-current`
Issue: $1

## Task
1. Fetch latest changes
2. Create feature branch: feature/$1
3. Push to remote with tracking
```

### 4. Use Tool Restrictions

```markdown
---
description: Security audit (read-only)
allowed-tools: Read, Grep, Glob, Bash(find:*)
---
```

### 5. Document Complex Commands

```markdown
---
description: Full deployment pipeline with validation
---

# Deployment Pipeline

This command runs the complete deployment process:

1. **Pre-flight**: Runs test suite
2. **Build**: Creates Docker image
3. **Deploy**: Updates Kubernetes
4. **Validate**: Health checks
5. **Notify**: Posts to Slack

## Usage
- `/deploy staging` - Deploy to staging
- `/deploy production` - Deploy to production
- `/deploy staging --skip-tests` - Skip test suite

## Requirements
- Docker installed and running
- kubectl configured for target environment
- SLACK_WEBHOOK in .env file

---

[Command implementation here...]
```

## Testing Commands

```bash
# 1. Register command
# Create the .md file in .claude/commands/

# 2. Verify registration
/help
# Should see your command listed

# 3. Test invocation
/your-command arg1 arg2

# 4. Check bash execution (if using !)
# Enable transcript mode: Ctrl-R
# Look for bash command output

# 5. Test with different arguments
/your-command different args

# 6. Validate tool restrictions (if using allowed-tools)
# Try operations outside allowed list
# Should ask permission or be blocked
```

## Common Patterns

### Code Review

```markdown
---
description: Review code changes with security focus
allowed-tools: Read, Grep, Glob, Bash(git *)
---

Review changes: !`git diff $1`

Focus on:
1. Security vulnerabilities
2. Input validation
3. Authentication/authorization
4. Data sanitization
5. Error handling
```

### Issue Handling

```markdown
---
description: Create feature branch and link issue
argument-hint: <issue-number>
---

Issue #$1 details: !`gh issue view $1`

1. Create feature branch: `feature/$1`
2. Update issue with branch link
3. Provide implementation plan
```

### Testing Workflow

```markdown
---
description: Run tests and analyze failures
allowed-tools: Bash(bun test:*, npm test:*)
---

Test results: !`bun test`

Analyze any failures and suggest fixes.
```

### Documentation Generation

```markdown
---
description: Generate API docs from code
argument-hint: <file-or-directory>
---

Code to document: @$1

Generate comprehensive API documentation including:
- Function signatures with types
- Parameter descriptions
- Return value documentation
- Usage examples
- Edge cases and error handling
```

## Troubleshooting

### Command Not Found

- Verify file location: `.claude/commands/your-command.md`
- Check filename: no spaces, lowercase, `.md` extension
- Restart Claude Code if needed

### Arguments Not Working

- Use `$1`, `$2`, not `{1}`, `{2}`
- Use `$ARGUMENTS` for all arguments
- Quote arguments if they contain spaces

### Bash Commands Failing

- Use `!` prefix before backticks: `!\`command\``
- Verify command works in terminal first
- Check command output length (<15k chars)

### File References Not Working

- Use `@` prefix: `@path/to/file.ts`
- Verify file paths are relative to project root
- Combine with arguments: `@$1`

### Tool Restrictions Not Applied

- Verify frontmatter syntax (YAML format)
- Check tool names match exactly: `Read`, not `read`
- Use wildcards for bash: `Bash(git *)`

## Advanced Patterns

See [REFERENCE.md](REFERENCE.md) for:
- SlashCommand tool configuration
- Complex argument parsing
- Multi-step command workflows
- Error handling patterns
- Integration with hooks

See [EXAMPLES.md](EXAMPLES.md) for:
- Real-world command implementations
- Team workflow examples
- Integration patterns
- Common use cases

## Quick Reference

```bash
# Scaffold new command
./scripts/scaffold-command.sh my-command "Description here"

# Validate command
./scripts/validate-command.sh .claude/commands/my-command.md

# Test command
/my-command test args
```

## Related Skills

- **claude-hook-authoring**: Add automation to commands with hooks
- **claude-plugin-authoring**: Bundle commands into distributable plugins
- **claude-config-management**: Configure command behavior globally
