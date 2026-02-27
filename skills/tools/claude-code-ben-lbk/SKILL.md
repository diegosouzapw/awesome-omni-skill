---
name: claude-code
description: Run Anthropic Claude Code CLI for coding tasks, code reviews, bug fixes, and feature development via background process with full programmatic control.
metadata:
  {
    "openclaw": { "emoji": "🧑‍💻", "requires": { "bins": ["claude"] } },
  }
---

# Claude Code Skill

Use Anthropic's Claude Code CLI via OpenClaw for intelligent coding assistance. Claude Code can build features, fix bugs, review code, and automate tedious development tasks.

## Installation

Install Claude Code first:

```bash
# macOS / Linux / WSL
curl -fsSL https://claude.ai/install.sh | bash

# Windows PowerShell
irm https://claude.ai/install.ps1 | iex

# Homebrew
brew install --cask claude-code

# WinGet
winget install Anthropic.ClaudeCode
```

**Prerequisites:**
- Claude subscription (Pro, Max, Teams, or Enterprise) OR Claude Console account
- On first run, you'll be prompted to log in

---

## ⚠️ Critical: Use PTY Mode

Claude Code is an interactive terminal application. **Always use `pty: true`**:

```bash
# ✅ Correct - with PTY
bash pty: true command: "claude 'Build a REST API'"

# ❌ Wrong - will break or hang
bash command: "claude 'Build a REST API'"
```

---

## Quick Start Patterns

### One-Shot Task

For quick tasks in any directory:

```bash
bash pty: true workdir: ~/Projects/myapp command: "claude 'Add input validation to the login form'"
```

### Background Long-Running Task

```bash
# Start in background
bash pty: true workdir: ~/Projects/myapp background: true command: "claude 'Refactor the authentication module'"

# Monitor with sessionId
process action: log sessionId: <session-id>

# Check if still running
process action: poll sessionId: <session-id>

# Send input if Claude asks a question
process action: submit sessionId: <session-id> data: "yes"

# Kill if needed
process action: kill sessionId: <session-id>
```

---

## Spec-Driven Development with Claude Code

Claude Code integrates with **Spec Kit** (GitHub) and **OpenSpec** (Fission AI) for structured, specification-driven development.

### Quick Comparison

| Framework | Style | Best For |
|-----------|-------|----------|
| **OpenSpec** | Light, iterative | Personal projects, brownfield, quick iteration |
| **Spec Kit** | Structured, thorough | Enterprise, greenfield, strict process |

---

## OpenSpec

**Philosophy:** Fluid, iterative, easy, built for brownfield projects.

### Installation

```bash
npm install -g @fission-ai/openspec@latest
```

### Initialize Project

```bash
# In your project directory
cd ~/Projects/myapp
openspec init --tools claude
```

### OpenSpec Workflow

```bash
# 1. Start a new change
bash pty: true workdir: ~/Projects/myapp command: "claude '/opsx:new add-dark-mode'"

# 2. Fast-forward - create all planning docs at once
bash pty: true workdir: ~/Projects/myapp command: "claude '/opsx:ff'"

# 3. Or step-by-step
bash pty: true workdir: ~/Projects/myapp command: "claude '/opsx:continue'"  # creates proposal
bash pty: true workdir: ~/Projects/myapp command: "claude '/opsx:continue'"  # creates specs
bash pty: true workdir: ~/Projects/myapp command: "claude '/opsx:continue'"  # creates design
bash pty: true workdir: ~/Projects/myapp command: "claude '/opsx:continue'"  # creates tasks

# 4. Apply - implement the tasks
bash pty: true workdir: ~/Projects/myapp background: true command: "claude '/opsx:apply'"

# 5. Verify - check implementation matches spec
bash pty: true workdir: ~/Projects/myapp command: "claude '/opsx:verify'"

# 6. Archive - finalize and merge specs
bash pty: true workdir: ~/Projects/myapp command: "claude '/opsx:archive'"
```

### OpenSpec Key Commands

| Command | Purpose |
|---------|---------|
| `/opsx:explore` | Think through ideas before committing |
| `/opsx:new <name>` | Start a new change |
| `/opsx:continue` | Create next artifact step by step |
| `/opsx:ff` | Fast-forward: create all docs at once |
| `/opsx:apply` | Implement tasks |
| `/opsx:verify` | Validate implementation |
| `/opsx:sync` | Merge delta specs into main specs |
| `/opsx:archive` | Archive completed change |
| `/opsx:bulk-archive` | Archive multiple changes |
| `/opsx:onboard` | Interactive tutorial |

### OpenSpec Project Structure

```
myapp/
├── openspec/
│   ├── specs/              # Main specs (source of truth)
│   │   └── ui/
│   │       └── spec.md
│   └── changes/            # Changes (one folder per feature)
│       └── add-dark-mode/
│           ├── proposal.md
│           ├── specs/      # Delta specs (ADDED/MODIFIED/REMOVED)
│           ├── design.md
│           └── tasks.md
└── .claude/commands/opsx/  # Claude slash commands
```

---

## Spec Kit (GitHub)

**Philosophy:** Structured, phase-gated, comprehensive, ideal for enterprise.

### Installation

```bash
# Install uv first: https://docs.astral.sh/uv/
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git
```

### Initialize Project

```bash
# Create new project
specify init my-project --ai claude

# Or initialize in existing project
cd ~/Projects/myapp
specify init . --ai claude
```

### Spec Kit Workflow

```bash
# 1. Establish constitution (project principles)
bash pty: true workdir: ~/Projects/myapp command: "claude '/speckit.constitution Create principles for code quality, testing, and performance'"

# 2. Create specification
bash pty: true workdir: ~/Projects/myapp command: "claude '/speckit.specify Build a photo album app with drag-and-drop organization'"

# 3. Clarify (optional but recommended)
bash pty: true workdir: ~/Projects/myapp command: "claude '/speckit.clarify'"

# 4. Create technical plan
bash pty: true workdir: ~/Projects/myapp command: "claude '/speckit.plan Use React, Vite, and localStorage. No backend.'"

# 5. Generate tasks
bash pty: true workdir: ~/Projects/myapp command: "claude '/speckit.tasks'"

# 6. Implement
bash pty: true workdir: ~/Projects/myapp background: true command: "claude '/speckit.implement'"
```

### Spec Kit Key Commands

| Command | Purpose |
|---------|---------|
| `/speckit.constitution` | Create project principles |
| `/speckit.specify` | Define what to build |
| `/speckit.clarify` | Clarify ambiguous requirements |
| `/speckit.plan` | Create technical implementation plan |
| `/speckit.tasks` | Generate actionable task list |
| `/speckit.implement` | Execute implementation |
| `/speckit.analyze` | Cross-artifact consistency check |
| `/speckit.checklist` | Generate quality checklists |

### Spec Kit Project Structure

```
myapp/
├── .specify/
│   ├── memory/
│   │   └── constitution.md     # Project principles
│   ├── specs/
│   │   └── 001-photo-albums/   # Feature specs
│   │       ├── spec.md
│   │       ├── plan.md
│   │       ├── tasks.md
│   │       └── research.md
│   ├── scripts/                # Helper scripts
│   └── templates/              # Document templates
└── .claude/commands/           # Claude slash commands
    ├── speckit.constitution.md
    ├── speckit.specify.md
    └── ...
```

---

## Choosing Between OpenSpec and Spec Kit

### Use OpenSpec when:
- You want to get started quickly ⭐
- Working on existing (brownfield) projects
- Prefer iterative, fluid workflow
- Personal projects or small teams
- Need delta specs to track changes

### Use Spec Kit when:
- Need strict, structured process
- Enterprise environment with compliance requirements
- Large teams needing governance
- Starting new (greenfield) projects
- Want comprehensive documentation

### Using Both:
- **OpenSpec** for day-to-day iteration and quick features
- **Spec Kit** for major architectural decisions and large features
- Both can coexist in the same project

---

## CLI Reference

### OpenSpec CLI

```bash
openspec init [--tools claude,cursor,...]    # Initialize project
openspec list                                 # List active changes
openspec show <change-name>                   # View change details
openspec validate <change-name>               # Validate spec format
openspec update                               # Update agent instructions
openspec view                                 # Interactive dashboard
```

### Spec Kit CLI

```bash
specify init <project> --ai claude            # Create new project
specify init . --ai claude                    # Init in current directory
specify init --here --ai claude               # Alternative syntax
specify check                                 # Check installed tools
specify init . --force --ai claude            # Force overwrite
```

---

## See Also

- [Claude Code Docs](https://code.claude.com/docs/)
- [Claude Code on the Web](https://claude.ai/code)
- [MCP Documentation](https://code.claude.com/docs/mcp)
- [GitHub Actions Integration](https://code.claude.com/docs/continuous-integration/github-actions)
- [Slack Integration](https://code.claude.com/docs/slack)
- [OpenSpec Docs](https://github.com/Fission-AI/OpenSpec)
- [Spec Kit Docs](https://github.com/github/spec-kit)

---

**Remember:** Claude Code is powerful - give it clear instructions and trust it to do the work, but always review the results before committing! 🚀
