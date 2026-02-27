---
name: copilot-cli
description: Enable autonomous code development and collaboration using GitHub Copilot CLI. Use this skill when you need to write code, fix bugs, implement features, refactor code, create pull requests, or perform any development tasks autonomously with AI-powered assistance. Supports interactive and non-interactive modes, session management, automated PR workflows, and comprehensive code analysis. Ideal for automated development workflows, CI/CD integration, code reviews, and autonomous feature implementation.
---

# GitHub Copilot CLI Integration

## Overview

Enable agent to autonomously use GitHub Copilot CLI for comprehensive code development, debugging, refactoring, and automated pull request workflows. This skill provides full access to Copilot CLI's capabilities including interactive sessions, non-interactive automation, permission management, model selection, and collaborative development features.

## When to Use This Skill

Use GitHub Copilot CLI when you need to:
- Autonomously write, debug, or refactor code
- Implement features or fix bugs with AI assistance
- Create automated pull request workflows
- Execute complex development tasks with minimal user intervention
- Resume previous coding sessions
- Work with specific AI models for different tasks
- Integrate AI-powered development into CI/CD pipelines
- Share development sessions as documentation

## Core Capabilities

### 1. Interactive Development Sessions

Start interactive mode for ongoing development conversations:

```bash
# Basic interactive mode
copilot

# Start with specific prompt
copilot -i "Fix authentication bug in user service"

# Resume most recent session
copilot --continue

# Resume specific session with picker
copilot --resume

# Interactive with specific model
copilot --model claude-sonnet-4.5
```

**Use when:** Complex tasks requiring multiple iterations, exploratory development, or ongoing conversations.

### 2. Non-Interactive Automation

Execute single prompts and exit, perfect for automation:

```bash
# Basic non-interactive execution (requires --allow-all-tools or tool permissions)
copilot -p "Fix the bug in main.js" --allow-all-tools

# Quick permission grant with --allow-all or --yolo
copilot -p "Implement user authentication" --allow-all

# Silent mode for scripting (outputs only agent response)
copilot -p "Generate API documentation" --allow-all -s

# Share results as markdown
copilot -p "Refactor database layer" --allow-all --share ./result.md

# Share to GitHub gist
copilot -p "Create test suite" --allow-all --share-gist
```

**Use when:** CI/CD pipelines, automated tasks, scripting, one-off operations.

### 3. Automated Pull Request Workflow

Complete workflow for autonomous PR creation (see [references/pr-workflow.md](references/pr-workflow.md) for detailed examples):

```bash
# Step 1: Implement feature in non-interactive mode
copilot -p "Implement user profile feature with tests" \
  --allow-all \
  --share ./implementation-notes.md

# Step 2: Create PR (manual git commands or use MCP GitHub tools)
# The implementation-notes.md can be used as PR description

# Step 3: Request Copilot review
copilot -p "Review the changes in current PR and suggest improvements" \
  --allow-all
```

**When implementing autonomous PR workflows:**
1. Use `--allow-all-tools` or `--allow-all` for full automation
2. Save session documentation with `--share` for PR descriptions
3. Combine with GitHub CLI or MCP tools for PR creation
4. Use `--resume` to continue work on the same task

### 4. Permission Management

Control what Copilot CLI can access and modify:

```bash
# Allow all permissions (fastest for automation)
copilot --allow-all  # or --yolo

# Granular tool permissions
copilot --allow-tool write --allow-tool 'shell(git:*)'
copilot --deny-tool 'shell(git push)'

# Path permissions
copilot --add-dir ~/workspace --add-dir /tmp
copilot --allow-all-paths  # Allow any path

# URL permissions
copilot --allow-url github.com --allow-url api.example.com
copilot --allow-all-urls

# Mixed permissions
copilot --allow-all-tools --deny-tool 'shell(rm *)' --add-dir ~/safe-workspace
```

**Security best practices:**
- Use `--allow-all` only in trusted, isolated environments
- Grant minimal permissions for production automation
- Use `--deny-tool` to exclude dangerous operations
- Restrict paths with `--add-dir` instead of `--allow-all-paths`

### 5. Model Selection

Choose the appropriate AI model for different tasks:

```bash
# Use Claude Sonnet 4.5 (default, best for most coding tasks)
copilot --model claude-sonnet-4.5

# Use Claude Opus 4.5 for complex architecture decisions
copilot --model claude-opus-4.5

# Use GPT-5 for specialized tasks
copilot --model gpt-5

# Use Haiku for faster, simpler operations
copilot --model claude-haiku-4.5
```

**Model selection guidance:**
- **claude-sonnet-4.5**: Default, excellent for most development tasks
- **claude-opus-4.5**: Complex reasoning, architecture design, large refactorings
- **claude-haiku-4.5**: Fast responses, simple fixes, documentation
- **gpt-5 variants**: Alternative for specific use cases or preferences

### 6. Session Management

Resume and manage development sessions:

```bash
# Resume most recent session
copilot --continue

# Resume with session picker (interactive selection)
copilot --resume

# Resume specific session by ID
copilot --resume abc123

# Resume with auto-approval
copilot --continue --allow-all-tools
```

**Use cases:**
- Continue interrupted work
- Review previous implementations
- Iterate on existing solutions
- Maintain context across sessions

## Workflow Patterns

### Pattern 1: Autonomous Feature Implementation

For complete feature development with automated PR:

1. **Implement:** Execute non-interactive prompt with full permissions
2. **Document:** Save session notes for PR description
3. **Create PR:** Use GitHub CLI or MCP tools
4. **Review:** Optional Copilot-assisted code review

See [references/pr-workflow.md](references/pr-workflow.md) for complete example.

### Pattern 2: Bug Fix Automation

For automated bug fixes in CI/CD:

1. Detect issue (tests, monitoring, issue tracker)
2. Execute: `copilot -p "Fix <issue>" --allow-all -s`
3. Capture output for logging/reporting
4. Auto-commit and create PR if changes made

### Pattern 3: Interactive Exploration

For complex problems requiring iteration:

1. Start: `copilot -i "Investigate performance bottleneck"`
2. Iteratively provide feedback and direction
3. Resume if interrupted: `copilot --continue`
4. Share results: Use in-session `/share` command

### Pattern 4: Code Review Assistant

For reviewing existing code or PRs:

```bash
# Review specific file
copilot -p "Review src/auth.ts for security issues" --allow-all

# Review PR changes
copilot -p "Review changes in current branch and suggest improvements" --allow-all

# Generate review comments
copilot -p "Analyze test coverage in the current PR" --allow-all -s > review.md
```

## Advanced Features

### Custom Instructions

Use repository-specific instructions via AGENTS.md:

```bash
# Uses .copilot/AGENTS.md or AGENTS.md in current directory
copilot

# Disable custom instructions
copilot --no-custom-instructions
```

### MCP Server Integration

Extend capabilities with Model Context Protocol servers:

```bash
# Enable all GitHub MCP tools
copilot --enable-all-github-mcp-tools

# Add specific GitHub MCP tools
copilot --add-github-mcp-tool create_pull_request --add-github-mcp-tool merge_pull_request

# Add custom MCP server configuration
copilot --additional-mcp-config @~/my-mcp-config.json

# Disable built-in MCP servers
copilot --disable-builtin-mcps
```

### Logging and Debugging

Control logging for troubleshooting:

```bash
# Enable debug logging
copilot --log-level debug

# Custom log directory
copilot --log-dir ~/copilot-logs

# Disable logs
copilot --log-level none
```

## Command Reference

For detailed documentation on all commands and options, see:
- [references/commands.md](references/commands.md) - Complete command reference
- [references/options.md](references/options.md) - All CLI options and flags
- [references/pr-workflow.md](references/pr-workflow.md) - Automated PR workflow examples
- [references/examples.md](references/examples.md) - Common usage patterns

## Best Practices

1. **Automation:** Use `--allow-all` for trusted environments, granular permissions for production
2. **Documentation:** Always use `--share` for non-interactive tasks to create audit trails
3. **Session Management:** Use `--continue` to maintain context across interruptions
4. **Model Selection:** Match model to task complexity (Haiku for simple, Opus for complex)
5. **Error Handling:** Capture exit codes and output in scripts for proper error handling
6. **Security:** Never grant `--allow-all-paths` in untrusted environments
7. **PR Workflows:** Combine with GitHub CLI/MCP tools for complete automation

## Quick Reference

```bash
# Most common patterns
copilot                                    # Interactive mode
copilot -i "task"                          # Interactive with prompt
copilot -p "task" --allow-all              # Non-interactive automation
copilot -p "task" --allow-all -s           # Silent (scripting)
copilot --continue                         # Resume last session
copilot -p "task" --allow-all --share ./out.md  # Save documentation
```
