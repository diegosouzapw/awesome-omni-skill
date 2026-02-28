---
name: windsurf-cascade
version: 1.0.0
description: A comprehensive skill for using the Windsurf IDE and its Cascade AI agent for various software engineering tasks (updated for 2026 features, includes Skills, Workflows, Memories, MCP, and multi-agent sessions).
author: Lucas Carrijo
---

# Windsurf Cascade Agent Skill

This skill provides a comprehensive guide and set of workflows for utilizing the Windsurf IDE and its Cascade AI agent, including all features from Wave 13 (January 2026).

## Installation

### Download

Download Windsurf Editor from [windsurf.com](https://windsurf.com) for your platform:
- **macOS**: `.dmg` installer (Intel and Apple Silicon)
- **Windows**: `.exe` installer
- **Linux**: `.deb` package (Debian/Ubuntu) or `.tar.gz` tarball

### Post-Installation Setup

**Add to PATH (optional but recommended):**

During onboarding, you can optionally install `windsurf` in PATH to launch from the command line:

```bash
# macOS - create symlink manually if needed
sudo ln -sF /Applications/Windsurf.app/Contents/Resources/app/bin/windsurf /usr/local/bin/windsurf

# Open a project from terminal
windsurf /path/to/project
```

**Import existing configuration:**

Windsurf supports importing settings, extensions, and keybindings from VS Code or Cursor during onboarding. You can also do this later via the Command Palette:

```
Cmd+Shift+P → "Import VS Code Settings"
Cmd+Shift+P → "Import Cursor Settings"
```

### WSL Setup (Windows)

Windsurf supports WSL (Beta). Connect to your WSL instance:
1. Click the bottom-left remote connection button
2. Select "Connect to WSL" or use Command Palette: `Remote-WSL: Connect to WSL`

For CLI access from WSL terminal, create a helper script:

```bash
#!/bin/bash
CURRENT_PATH=$(readlink -f "$1")
windsurf --folder-uri "vscode-remote://wsl+Ubuntu$CURRENT_PATH"
```

### Authentication

Sign up or log in to your Windsurf (formerly Codeium) account during onboarding or via the profile menu.

## Cascade — The AI Agent

Cascade is Windsurf's agentic AI assistant. It understands your entire codebase, tracks your real-time actions (edits, terminal, clipboard), and can autonomously create files, edit code across multiple files, run terminal commands, and maintain project memory.

### Cascade Modes

Cascade comes in two primary modes:

- **Write Mode** (`Cmd+L` / `Ctrl+L`): Full write access — creates files, edits code, runs terminal commands, and makes modifications to your codebase.
- **Chat Mode** (`Cmd+Shift+L` / `Ctrl+Shift+L` to toggle): Read-only — answers questions about your codebase and general coding principles without making changes.

### Model Selection

Switch between models from the dropdown below the Cascade input box. Available models include:

- **SWE-1.5** (Windsurf's own near-frontier model, free for all users)
- **Claude Opus 4.6**, Claude Sonnet 4.5
- **GPT-5.2**, GPT-5.2-Codex, GPT-5.1, GPT-5.1-Codex
- **Gemini 3 Flash**, Gemini 3 Pro
- **Falcon Alpha** (stealth model, speed-optimized)
- **BYOK** (Bring Your Own Key) for custom models

Each prompt consumes credits depending on the model selected.

### Tool Calling

Cascade has a variety of built-in tools:

- **Search**: Semantic code search across your repository
- **Analyze**: Deep analysis of code structure and relationships
- **Web Search**: Search the web for documentation and references
- **MCP**: Call external tools via Model Context Protocol
- **Terminal**: Execute shell commands directly

Cascade can make up to **25 tool calls per prompt**. If the trajectory stops, type `continue` and Cascade will resume.

### Context Selection with @ Mentions

Reference specific files, functions, or context in your prompts:

```
@filename.ts
@src/components/
@function:calculateTotal
```

You can also:
- Drag and drop files from the File Explorer into Cascade
- Send problems from the Problems panel to Cascade
- Highlight errors and click "Explain and Fix"
- @-mention previous conversations for cross-session context

### Voice Input

Use voice input to interact with Cascade via speech-to-text transcription.

### Checkpoints and Reverts

Cascade creates named checkpoints. You can revert changes by hovering over a prompt and clicking the revert arrow, or via the table of contents. **Reverts are currently irreversible.**

## Keyboard Shortcuts

| Action | macOS | Windows/Linux |
|---|---|---|
| Open Cascade (Write) | `Cmd+L` | `Ctrl+L` |
| Toggle Write/Chat | `Cmd+Shift+L` | `Ctrl+Shift+L` |
| Command Palette | `Cmd+Shift+P` | `Ctrl+Shift+P` |
| Inline AI (terminal) | `Cmd+I` | `Ctrl+I` |
| Accept focused diff hunk | `Option+Enter` | `Alt+Enter` |
| Reject focused diff hunk | `Option+Shift+Backspace` | `Alt+Shift+Backspace` |
| Fast Context (first msg) | `Cmd+Enter` | `Ctrl+Enter` |

## Skills

Skills let you bundle instructions, templates, checklists, and supporting files into folders that Cascade can invoke for complex, multi-step tasks.

### Creating a Skill

1. Click the Customizations icon in Cascade's top-right slider menu
2. Navigate to the Skills panel
3. Click `+ Workspace` (project-specific) or `+ Global`
4. Name the skill (lowercase letters, numbers, hyphens only)

### Skill Structure

Each skill is a folder with a `SKILL.md` file and optional supporting files:

```
.windsurf/skills/deploy-to-production/
├── SKILL.md
├── deployment-checklist.md
├── rollback-procedure.md
└── config-template.yaml
```

### SKILL.md Format

```markdown
---
name: deploy-to-production
description: Guides the deployment process to production with safety checks
---

## Pre-deployment Checklist
1. Run all tests
2. Check for pending migrations
3. Verify environment variables
...
```

The `name` field is used for display and @-mentions. The `description` helps Cascade decide when to automatically invoke the skill.

### Invoking Skills

- **Automatic**: Cascade uses progressive disclosure to invoke skills when they're relevant to your task
- **Manual**: @-mention the skill name in your prompt

For the full Skills specification, visit [agentskills.io](https://agentskills.io).

## Workflows

Workflows define a series of steps to guide Cascade through repetitive tasks. They are saved as markdown files and invoked via slash commands.

### Creating a Workflow

1. Click Customizations icon → Workflows panel → `+ Workflow`
2. Or ask Cascade to generate a Workflow for you

### Workflow Storage

Workflows are saved in `.windsurf/workflows/` directories. Windsurf discovers them from:
- Current workspace and sub-directories
- Parent directories up to the git root (for git repos)
- Multiple workspace support with deduplication

Workflow files are limited to **12,000 characters** each.

### Invoking Workflows

```
/workflow-name
```

Workflows can call other workflows:

```markdown
## Steps
1. Call /lint-and-format
2. Call /run-tests
3. Deploy to staging
```

### Example Workflow — PR Review

```markdown
---
name: pr-review
description: Review PR comments and address them
---

## Steps
1. Check out the PR branch: `gh pr checkout [id]`
2. Get comments on PR:
   ```bash
   gh api --paginate repos/[owner]/[repo]/pulls/[id]/comments | jq '.[] | {user: .user.login, body, path, line}'
   ```
3. For EACH comment, address the feedback and commit the fix
4. Push changes and reply to each comment
```

## Memories & Rules

Memories persist context across Cascade conversations. Rules guide Cascade behavior.

### Memories

- **Auto-generated**: Cascade creates memories when it encounters useful context. Does NOT consume credits.
- **User-created**: Type `create memory ...` in Cascade to manually save context.
- Auto-generated memories are workspace-specific.

**Managing Memories:**
- Windsurf Settings → Settings tab → Manage next to "Cascade-Generated Memories"
- Or: three dots in Cascade → Manage Memories
- Toggle auto-generation: Settings → "Auto-Generate Memories"

### Rules

Rules are manually defined instructions for Cascade.

**Rule Levels:**
- `global_rules.md` — applies across all workspaces
- `.windsurf/rules/` — workspace-level directory with rules tied to globs or descriptions
- System-level rules (Enterprise) — deployed via MDM policies

**Activation Modes:**
- **Always**: Rule is always active
- **Glob**: Applied to files matching a pattern (e.g., `*.js`, `src/**/*.ts`)
- **Manual / Description-based**: Activated by natural language match

**Rules Best Practices:**
- Keep rules simple, concise, and specific
- Use bullet points and markdown formatting
- Avoid generic rules like "write good code"
- Use XML tags to group similar rules

**Example Rule:**

```markdown
# Coding Guidelines
- My project's programming language is Python
- Use early returns when possible
- Always add documentation when creating new functions and classes
- Use pytest for testing
- Follow PEP 8 style guide
```

## Terminal Integration

### Inline AI Terminal

Press `Cmd+I` / `Ctrl+I` in the terminal to access an inline chat box that generates CLI commands from natural language.

### Cascade Terminal Execution

Cascade can run terminal commands directly. Configure auto-execution levels in Windsurf Settings:

1. **Manual**: Always ask for permission (default)
2. **Semi-auto**: Auto-run safe commands
3. **Turbo Mode**: Auto-execute all commands without confirmation
4. **Custom**: Use Allow/Deny lists for specific commands

### Dedicated Terminal (Wave 13)

Windsurf uses a dedicated zsh shell for Cascade command execution, providing improved reliability. It uses your `.zshrc` environment variables and is fully interactive.

### ⚠️ Using with AI Agents / Automation

When running Windsurf from automated environments (AI agents, scripts, orchestrators), the IDE requires a GUI context. For headless automation scenarios, consider:

1. **Using Windsurf's Workflows**: Define multi-step tasks as Workflows that Cascade executes
2. **MCP Integration**: Connect external automation tools via MCP servers
3. **Cascade Hooks**: Execute custom shell commands at key points in Cascade's workflow

## MCP Integration

Windsurf supports Model Context Protocol (MCP) for connecting external tools and services.

### Configuration

Configure MCP servers in `mcp_config.json`:

```json
{
  "mcpServers": {
    "github": {
      "command": "uvx",
      "args": ["github-mcp"],
      "env": {
        "GITHUB_TOKEN": "your_token_here"
      }
    }
  }
}
```

Access via: Windsurf Settings → Cascade → Manage MCPs → View raw config

### MCP Features

- **MCP Marketplace**: Browse curated servers in Windsurf Settings for one-click setup
- **@ Mentions**: Trigger MCP tools by @-mentioning in Cascade
- **Enable/Disable**: Toggle MCP servers from the Cascade header
- **Transports**: Supports stdio, Streamable HTTP, and SSE
- **Enterprise**: Team admins can whitelist/blacklist MCP servers

Each MCP tool call costs one prompt credit.

## Cascade Hooks

Execute custom shell commands at key points during Cascade's workflow:

- **On model response**: For logging, auditing, security controls
- **Pre/Post hooks**: For validation and governance (Enterprise)

## Simultaneous Cascades

Run multiple Cascade sessions in parallel:

- Open side-by-side Cascade panes
- Use Git worktrees to work on different branches without conflicts
- Start work in a new Cascade while another is executing

## Fast Context

Press `Cmd+Enter` / `Ctrl+Enter` on the first message to enable Fast Context. Uses SWE-grep models for up to 20x faster code retrieval from large codebases.

## Ignoring Files

Add files to `.codeiumignore` at the root of your workspace (same syntax as `.gitignore`). For global ignore rules across all repositories, place `.codeiumignore` in `~/.codeium/`.

## Workflows (Common Use Cases)

### Code Review

```
Review the changes in the current branch against main.
Focus on security and performance.
```

### Refactoring

```
Refactor src/utils.ts to reduce complexity and improve type safety.
```

### Debugging

```
Analyze the following error log and suggest a fix: [paste error]
```

Or: Highlight error → "Explain and Fix"

### Git Integration

```
Generate a commit message for the staged changes adhering to conventional commits.
```

### Deployment

Create a Workflow in `.windsurf/workflows/deploy.md`:

```markdown
---
name: deploy
description: Deploy to production with safety checks
---

## Steps
1. Run all tests: `npm test`
2. Build the project: `npm run build`
3. Run linter: `npm run lint`
4. If all pass, deploy: `npm run deploy:production`
5. Verify deployment health checks
```

Invoke with `/deploy` in Cascade.

### Live Preview

Windsurf has built-in live preview for web apps. Ask Cascade to start a dev server and preview will appear inside the editor. Click any element to let Cascade reshape it.

### App Deploy

Deploy your app with one click via Cascade tool calls (beta Netlify deployment support).

## Pricing

| Plan | Price | Credits/month | Best For |
|---|---|---|---|
| Free | $0 | 25 | Students, hobbyists |
| Pro | $15/mo | 500 | Individual developers |
| Teams | $30/user/mo | Custom | Development teams |
| Enterprise | $60/user/mo | Custom | Large organizations |

## Key Differences from Cursor

| Feature | Windsurf | Cursor |
|---|---|---|
| AI Agent | Cascade (agentic, flow-aware) | Cursor Agent (CLI-based) |
| Rules | `.windsurf/rules/` + `global_rules.md` | `.cursor/rules` + `CLAUDE.md` |
| Workflows | `.windsurf/workflows/` (slash commands) | N/A (manual) |
| Memories | Auto-generated + user-created | Codebase indexing + Project Rules |
| Skills | `.windsurf/skills/` (bundled folders) | N/A |
| Terminal | Dedicated zsh shell + Turbo Mode | Standard terminal |
| Live Preview | Built-in | Extension-based |
| MCP | Native with marketplace | Native |
