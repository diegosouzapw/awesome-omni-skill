---
name: aissist-cli
description: Use the aissist CLI tool for personal goal tracking, todo management, daily history logging, context-specific notes, guided reflections, and AI-powered semantic recall. Activate when users mention goals, tasks, todos, progress tracking, journaling, work history, personal assistant, meal planning, fitness tracking, or want to search their past activities and reflections.
allowed-tools: Bash(aissist:*)
---

# Aissist CLI - Personal AI Assistant

Aissist is a local-first CLI personal assistant that helps users track goals, manage todos, log daily activities, organize context-specific information, and search their data using AI-powered semantic recall.

## When to Use This Skill

Activate this skill when users:
- Want to **set or track goals** ("I want to learn Rust", "track my fitness goals")
- Need **todo/task management** ("add this to my todo list", "what's on my plate today")
- Want to **log their work or activities** ("record what I did today", "log this accomplishment")
- Organize **context-specific information** (work notes, meal plans, fitness logs, project details)
- Need **guided reflection** ("reflect on my week", "journal about today")
- Want to **search past activities** ("what did I work on last month", "find when I learned about X")
- Ask for **action proposals** based on their goals and history

## Quick Start

### Initialization
Before using aissist, storage must be initialized:

```bash
# Local storage (project-specific in ./.aissist/)
aissist init

# Global storage (user-wide in ~/.aissist/)
aissist init --global
```

**When to use which:**
- **Local**: Project-specific goals and todos (e.g., software project tasks)
- **Global**: Personal life goals, general todos, daily reflections

**Interactive Onboarding:**
After successful initialization, aissist will guide you through:
1. Creating your first goal (optional, can be skipped)
2. Adding a todo linked to that goal (optional, can be skipped)

This interactive flow helps you get started quickly. You can press Ctrl+C at any time to skip prompts.

### Hierarchical Configuration (Monorepo Support)

Aissist supports reading data from parent directories while keeping writes isolated to the local directory. Perfect for monorepos and nested projects.

**Initialization with Hierarchy:**
```bash
cd ~/monorepo/packages/api
aissist init

# Aissist detects parent .aissist directories and prompts:
# ✓ Detected .aissist directories in parent paths:
#   • ~/monorepo/.aissist (2 levels up)
#   • ~/.aissist (global)
#
# ? Would you like to include these directories for reading? (Y/n)
```

**Runtime Configuration:**
```bash
# Enable hierarchy after initialization
aissist config hierarchy enable

# Check current status
aissist config hierarchy status

# Disable (sandbox mode)
aissist config hierarchy disable

# View read hierarchy
aissist path --hierarchy
```

**Behavior:**
- **Read**: Access goals, history, and context from parent directories
- **Write**: All changes saved to local directory only (no pollution of parent configs)
- **Precedence**: Local data takes priority when codenames/identifiers conflict

## Core Commands

### Goal Management

```bash
# Add a new goal (AI generates a memorable codename)
aissist goal add "Learn TypeScript fundamentals"

# Add goal with deadline
aissist goal add "Complete project proposal" --deadline "next Friday"

# List all active goals interactively
aissist goal list

# List all active goals in plain text
aissist goal list --plain

# Filter goals by deadline
aissist goal list --deadline 2025-12-01
aissist goal list --deadline "next week"

# Complete a goal
aissist goal complete <codename>

# Set/update deadline
aissist goal deadline <codename> "2024-12-31"
```

**Goal Features:**
- Auto-generated kebab-case codenames (e.g., "learn-typescript-fundamentals")
- Deadline support with natural language parsing
- Interactive management with checkbox UI
- Priority tracking (high/normal/low)
- Post-creation prompt to add linked todos (helps build goal-driven workflows)

### Todo Management

```bash
# Add a todo
aissist todo add "Review PR #123"

# Add todo with priority
aissist todo add "Fix critical bug" --priority high

# Add todo linked to a goal
aissist todo add "Read TypeScript docs" --goal learn-typescript-fundamentals

# List todos interactively
aissist todo list

# Mark todo as done (logs to history automatically)
aissist todo done "Review PR"

# Interactive management (full CRUD)
aissist todo manage
```

**Todo Features:**
- Priority levels (high/normal/low)
- Link to goals for tracking progress
- Interactive checkbox UI
- Automatic history logging when completed

### History Logging

```bash
# Log an entry
aissist history log "Completed code review for authentication feature"

# Log with goal linking
aissist history log "Finished TypeScript tutorial chapter 3" --goal learn-typescript-fundamentals

# Import from GitHub
aissist history log --from github

# Show all history entries
aissist history show

# Show history since specific date
aissist history show --date 2025-01-01
aissist history show --date "last week"
aissist history show --date "last month"
```

**History Features:**
- Daily markdown files (YYYY-MM-DD.md)
- Goal linking for progress tracking
- GitHub import (requires gh CLI)
- Timestamped entries

### Context Management

Organize information by context (work, diet, fitness, projects, etc.):

```bash
# Log context-specific information
aissist context log work "Sprint planning notes: focus on auth module"
aissist context log diet "Meal prep: chicken, rice, vegetables"
aissist context log fitness "Workout: 5k run in 28 minutes"

# Show context entries
aissist context show work --from "this week"

# List all contexts
aissist context list

# Bulk import files
aissist context ingest work ./meeting-notes/
```

### Reflection

Guided reflection with AI-powered prompts:

```bash
# Start interactive reflection
aissist reflect

# Reflect on specific timeframe
aissist reflect --from "this week"
```

### AI-Powered Features

```bash
# Semantic search across all data
aissist recall "what did I learn about TypeScript?"
aissist recall "show my progress on fitness goals"

# Generate action proposals
aissist propose now         # Get 1 immediate action (1-2 hours)
aissist propose             # Daily proposals
aissist propose "this week" # Weekly proposals
```

### Claude Code Plugin Commands

When working within Claude Code, use these slash commands for quick access:

#### `/aissist:log` - AI-Enhanced Quick Logging

Log history entries with AI enhancement and multimodal support. Claude automatically rephrases your input into well-structured entries and links to relevant goals.

```
/aissist:log Fixed auth bug, took 3 hours
```
→ Claude enhances: "Resolved authentication bug in login flow (3 hours)"

**With images/screenshots:**
```
/aissist:log [attach screenshot] Updated dashboard design
```
→ Claude analyzes image: "Updated dashboard: redesigned metrics cards with improved data visualization..."

**Features:**
- AI rephrases rough notes into polished entries
- Preserves exact metrics and time estimates
- Analyzes attached images using vision capabilities
- Automatically links to relevant goals based on content
- Handles multi-line descriptions

**When to use:**
- Quick logging without leaving Claude Code
- Have screenshots or images to log
- Want AI to structure your rough notes
- Need automatic goal linking

#### `/aissist:log-github` - Import GitHub Activity

Import GitHub commits and pull requests as history entries with semantic summarization.

```
/aissist:log-github today
/aissist:log-github "this week"
/aissist:log-github "last month"
```

**Features:**
- Fetches commits and PRs from specified timeframe
- Semantically groups related changes
- Auto-links to relevant goals
- Requires GitHub CLI (`gh`) authentication

**When to use:**
- Bulk import of recent development work
- End of day/week to catch all GitHub activity
- Want automated activity summarization

#### `/aissist:recall` - Semantic Search

Search your history, goals, and reflections using natural language queries.

```
/aissist:recall "what did I learn about TypeScript last month?"
/aissist:recall "show my fitness progress"
```

#### `/aissist:report` - Generate Reports

Create accomplishment reports from your history for various purposes.

```
/aissist:report "this week" --purpose standup
/aissist:report "last month" --purpose performance-review
```

**Command Comparison:**

| Command | Purpose | Input | Output |
|---------|---------|-------|--------|
| `/aissist:log` | Quick history logging | Text + images | AI-enhanced entry |
| `/aissist:log-github` | GitHub import | Timeframe | Summarized commits/PRs |
| `/aissist:recall` | Search history | Natural language query | Relevant entries |
| `/aissist:report` | Generate reports | Timeframe + purpose | Formatted report |

## Command Construction Guidelines

1. **Always check if storage is initialized** - If user is new to aissist, suggest `aissist init` first
2. **Use goal codenames for linking** - When linking to goals, use the kebab-case codename
3. **Natural language deadlines** - Aissist parses "next Friday", "end of month", "2024-12-31"
4. **Interactive when appropriate** - Suggest `list` or `manage` commands for better UX over plain text
5. **Link related data** - When logging history or todos, link to relevant goals using `--goal`

## Progressive Disclosure

For detailed information, reference these supporting files:
- **command-reference.md** - Complete command documentation with all options
- **workflow-examples.md** - Multi-step workflows and usage patterns
- **storage-model.md** - Storage concepts and file organization

## Common Patterns

**Morning Routine:**
```bash
aissist todo list  # Review today's tasks
```

**Evening Routine:**
```bash
aissist history log "Completed X, Y, and Z today"
aissist todo done "task description"  # Marks done and logs to history
```

**Goal Progress:**
```bash
aissist goal list  # Review goals
aissist history show --from "this week"  # See recent progress
aissist recall "progress on <goal>"  # AI-powered progress review
```

**Weekly Planning:**

```bash
aissist reflect --from "this week"  # Reflect on the week
aissist propose  # Get AI-generated action proposals
```


## Error Handling

If commands fail:
- Check if storage is initialized: `aissist path` shows current storage location
- Verify goal codenames with: `aissist goal list --plain`
- For recall/propose, ensure Claude API key is configured: `claude login`

## Tips for Effective Use

1. **Consistent logging** - Encourage users to log daily for better recall
2. **Goal linking** - Always link todos and history to goals for progress tracking
3. **Use contexts** - Separate work, personal, fitness, etc. for better organization
4. **Weekly reflection** - Suggest regular reflection for self-awareness
5. **Leverage recall** - Use semantic search instead of manually browsing files
