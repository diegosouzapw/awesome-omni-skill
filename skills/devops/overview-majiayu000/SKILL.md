---
name: overview
description: Overview of Claude Code capabilities, features, and use cases. Use when user asks about what Claude Code is, what it can do, or general capabilities.
---

# Claude Code Overview

Claude Code is Anthropic's agentic CLI tool designed for developers working in terminal environments.

## Core Capabilities

### Code Development

**Build features from descriptions:**
Turn plain English requirements into functional code with planning and validation.

**Examples:**
- "Implement user authentication with JWT"
- "Add a REST API endpoint for user profiles"
- "Create a dashboard component with charts"

### Debugging & Problem Solving

**Analyze codebases to identify and fix issues:**
Work from error messages or bug descriptions provided by users.

**Examples:**
- "Fix the TypeError in dashboard.tsx"
- "Debug why the API returns 500 errors"
- "Resolve the memory leak in the worker process"

### Codebase Navigation

**Ask questions about project structure:**
Claude maintains awareness of entire codebases while accessing current web information and external data sources via MCP integrations.

**Examples:**
- "Where is user authentication handled?"
- "What does this project do?"
- "How do the payment flows work?"
- "Show me all the API endpoints"

### Task Automation

**Handle routine development work:**
- Resolve lint issues
- Manage merge conflicts
- Generate release notes
- Run test suites and fix failures
- Update dependencies
- Refactor code for consistency

## Why Developers Prefer It

### Native Integration

**Works directly within developer workflows:**
Unlike separate chat interfaces, Claude Code operates directly in the terminal and integrates with existing tools.

**Terminal-native features:**
- Interactive REPL mode
- Command history
- Vim mode support
- File path autocomplete
- Background task execution

### Proactive Execution

**Independent action:**
The tool can edit files, execute commands, and create commits independently.

**Extensibility through MCP:**
Access design documents, project management systems, and custom tooling through Model Context Protocol integrations.

### Unix Philosophy

**Composable, scriptable design:**
- Pipe input and output
- Chain with other commands
- Integrate into CI/CD pipelines
- Use in automation scripts
- Parse JSON output programmatically

**Examples:**
```bash
# Pipe git diff for review
git diff | claude -p "review these changes"

# Parse JSON output
claude -p "task" --output-format json | jq -r '.result'

# Use in scripts
if claude -p "run tests" --output-format json | jq -e '.subtype == "success"'; then
  echo "Tests passed"
fi
```

### Enterprise Ready

**Multiple deployment options:**
- Claude API (direct)
- AWS Bedrock
- Google Cloud Vertex AI

**Built-in features:**
- Security controls
- Privacy protections
- Compliance support
- Usage monitoring
- Cost tracking
- Team collaboration

## Key Features

### Intelligent Context Management

**Progressive disclosure:**
- Only loads relevant information as needed
- Manages large codebases efficiently
- Uses prompt caching for cost optimization

**CLAUDE.md memory files:**
- Project-level instructions
- Team-shared guidelines
- Personal preferences
- Hierarchical configuration

### Customization & Extension

**Slash commands:**
Create custom commands for frequent tasks.

**Subagents:**
Specialized AI assistants for specific domains.

**Skills:**
Modular capabilities that extend Claude's expertise.

**Hooks:**
Automated scripts triggered by events.

**MCP servers:**
Connect to external tools and data sources.

**Plugins:**
Shareable packages combining all of the above.

### Developer Experience

**Interactive features:**
- Real-time diff viewing
- File attachment with @-mentions
- Conversation history
- Session resumption
- Checkpoint system for easy rollback

**IDE integration:**
- VS Code extension (beta)
- JetBrains plugin
- Native diff viewers
- Selection context sharing

### Safety & Control

**Permission management:**
- Granular tool access control
- File-level restrictions
- Command approval workflows
- Sandbox mode for isolation

**Checkpointing:**
- Automatic state tracking
- Easy rollback to previous states
- Conversation and code rewinding

**Git integration:**
- Safe commit practices
- Pull request generation
- Adherence to git workflows
- Pre-commit hook support

## Common Use Cases

### Feature Development
```
"Implement a new search feature with autocomplete"
"Add pagination to the user list"
"Create an admin dashboard"
```

### Bug Fixing
```
"Fix the authentication redirect issue"
"Resolve the race condition in the worker"
"Debug why emails aren't sending"
```

### Code Quality
```
"Review this code for security issues"
"Refactor for better performance"
"Add error handling throughout"
"Improve test coverage"
```

### DevOps & Automation
```
"Set up CI/CD pipeline"
"Create Docker configuration"
"Write deployment scripts"
"Generate infrastructure as code"
```

### Documentation
```
"Add JSDoc comments to all functions"
"Generate API documentation"
"Create README with examples"
"Write migration guide"
```

### Learning & Exploration
```
"Explain how this authentication works"
"What design patterns are used here?"
"Show me examples of using this API"
"How can I optimize this query?"
```

## Getting Started

### Installation

**Requires:** Node.js 18+

**NPM:**
```bash
npm install -g @anthropic-ai/claude-code
```

**Native installer:**
```bash
# macOS/Linux
curl -fsSL https://claude.ai/install.sh | bash

# Homebrew
brew install --cask claude-code
```

### Authentication

Start an interactive session and log in:
```bash
claude
/login
```

**Options:**
- **Claude.ai** (subscription-based, recommended)
- **Claude Console** (API access with prepaid credits)

### First Steps

1. Navigate to your project
2. Run `claude`
3. Ask about your codebase
4. Make some changes
5. Create a commit

**Approximate time:** 30 seconds from installation to first task

## Resources

- **Documentation:** https://docs.claude.com/en/docs/claude-code
- **GitHub:** https://github.com/anthropics/claude-code
- **Support:** https://github.com/anthropics/claude-code/issues

## Comparison: Claude Code vs Other Tools

### vs GitHub Copilot
- **Claude Code:** Full agentic workflows, terminal-native, multi-file operations
- **Copilot:** Autocomplete, inline suggestions, IDE-focused

### vs ChatGPT
- **Claude Code:** Direct code execution, file operations, git integration
- **ChatGPT:** Conversational interface, manual copy-paste

### vs Traditional IDEs
- **Claude Code:** Natural language interface, autonomous execution
- **IDEs:** Manual coding, refactoring tools, debugging

### vs CI/CD Tools
- **Claude Code:** Intelligent automation, context-aware
- **CI/CD:** Script-based, deterministic workflows

**Best approach:** Use Claude Code alongside these tools for comprehensive development support.

## Enterprise Features

### Team Collaboration
- Shared plugins via marketplaces
- Project-level configuration
- Team memory files (CLAUDE.md)
- Standardized workflows

### Security & Compliance
- Permission controls
- File access restrictions
- Audit logging via OpenTelemetry
- Sandbox execution mode

### Cost Management
- Usage monitoring
- Cost tracking by user/team
- Budget alerts
- ROI measurement tools

### Deployment Options
- Direct API access
- AWS Bedrock integration
- Google Vertex AI integration
- On-premises support (coming soon)

## What's Next?

- Explore **quickstart guide** for hands-on tutorial
- Learn about **slash commands** for custom workflows
- Set up **subagents** for specialized tasks
- Configure **hooks** for automation
- Install **plugins** from the marketplace
- Join the **community** on GitHub
