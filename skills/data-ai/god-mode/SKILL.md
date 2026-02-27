---
name: god-mode
description: God's eye view of your dev repos. Multi-project tracking across GitHub/Azure DevOps. AI learns from your commits to upgrade your agents.md.
metadata: {"openclaw": {"requires": {"bins": ["gh", "sqlite3", "jq"]}}}
user-invocable: true
---

# god-mode Skill

> **Conversational project oversight and AI agent coaching for OpenClaw.**

## Overview

**god-mode** lets users ask about their projects in plain English. You run the commands and explain the results conversationally.

**What users can ask:**
- "Set up god-mode for my repos"
- "What's happening across my projects?"
- "Which repos need attention?"
- "Analyze my agents.md"
- "What should I work on today?"

**What you provide:**
- Multi-project status summaries (GitHub + Azure DevOps)
- Plain-English explanations of commit activity, PRs, issues
- LLM-powered agent instruction analysis
- Recommendations for improving their AGENTS.md

**Perfect for:**
- Developers juggling multiple repos who want conversational oversight
- Teams using AI coding assistants (Claude, Copilot, etc.)
- Anyone who wants data-driven improvements to their AGENTS.md

## Getting Started

### For OpenClaw Users (Conversational)

**Just tell me what you want!** I'll run the god-mode commands for you.

**Example conversations:**

> **You:** "Set up god-mode for my tada repository"
>
> **Me:** Sure! I'll set up god-mode and add your tada repo.
> *[runs `god setup` and `god projects add github:YourUsername/tada`]*
> Done! Your repo is configured. Want me to sync the data?

> **You:** "Yes, and show me what's happening"
>
> **Me:** *[runs `god sync` and `god status`]*
> 
> Here's your overview:
> - **tada**: 30 commits this week, last activity 3 days ago
> - 0 open PRs, 0 issues
>
> Your most active project! Commits are all features and bug fixes.

> **You:** "Analyze my agents.md for tada"
>
> **Me:** *[runs `god agents analyze github:YourUsername/tada`]*
>
> I've analyzed your AGENTS.md against 155 commits. Here's what I found:
>
> **GAPS:**
> - Testing practices (high impact) - 68 bug fixes but no testing guidance
> - Voice API debugging - 12 commits mention "voice" but no troubleshooting
>
> **STRENGTHS:**
> - TypeScript strictness is working - zero type-related fixes
>
> Want me to add testing guidance to your AGENTS.md?

**What you can ask me:**
- "Set up god-mode for [repo]"
- "Show me all my projects"
- "What's happening across my repos?"
- "Analyze my agents.md"
- "Which repos need attention?"
- "What should I work on today?"

I'll handle all the commands and explain the results in plain English.

### For CLI Users (Direct Commands)

If you prefer running commands yourself:

```bash
# Setup
god setup

# Add a project
god projects add github:username/repo

# Sync and view
god sync
god status
```

## Commands

### `god status [project]`
Show overview of all projects, or details for one:
```bash
god status              # All projects
god status myproject    # One project in detail
```

### `god sync [project] [--force]`
Fetch/update data from repositories:
```bash
god sync                # Incremental sync all
god sync myproject      # Just one project
god sync --force        # Full refresh (ignore cache)
```

### `god projects`
Manage configured projects:
```bash
god projects                        # List all
god projects add github:user/repo   # Add project
god projects remove myproject       # Remove project
```

### `god review [--month YYYY-MM]`
Generate monthly activity reviews:
```bash
god review                  # Last month's activity
god review --month 2026-01  # Specific month
god review --json           # JSON output
```

**What it shows:**
- Total commits across all projects
- Most active repositories
- Pull request activity (merged, active, closed)
- Detailed breakdown by project with date ranges
- Perfect for monthly retrospectives and planning

**Example output:**
```
Monthly Review: 2026-01
  📊 286 commits across 7 projects
  👥 10 unique contributors
  
Most Active Projects:
  tada - 155 commits
  ContentEngine - 63 commits
  brain - 27 commits
```

**Use cases:**
- Monthly team stand-ups
- Personal retrospectives ("What did I actually work on?")
- Quarterly planning ("Which projects got attention?")
- Automated monthly summaries via cron

### `god agents analyze <project>`
Analyze agents.md against commit history using LLM:
```bash
god agents analyze myproject
```

**What it does:**
1. Fetches your AGENTS.md from the repository
2. Analyzes commit patterns (types, pain points, frequently changed files)
3. Calls an LLM (Claude/GPT) to find gaps and suggest improvements
4. Displays recommendations interactively
5. Optionally applies changes to your AGENTS.md

**LLM Configuration:**

god-mode automatically detects and uses the best available LLM:

1. **OpenClaw (default when running as skill)** - Uses your OpenClaw agent
2. **Anthropic** - Set `ANTHROPIC_API_KEY="sk-ant-..."`
3. **OpenAI** - Set `OPENAI_API_KEY="sk-..."`  
4. **OpenRouter** - Set `OPENROUTER_API_KEY="sk-or-..."`
5. **Manual** - Outputs prompt if no LLM available

**When running in OpenClaw:**
- The analysis prompt is displayed to your OpenClaw agent
- You (or your agent) provides the JSON analysis directly in the conversation
- Much simpler than managing separate API keys!

**OpenClaw Workflow:**

When you run `god agents analyze` in OpenClaw:

1. **Analysis starts:**
   ```
   🔭 Analyzing github:InfantLab/tada
   ✅ Found AGENTS.md (remote)
   ✅ 155 commits analyzed
   🤖 Using OpenClaw's LLM
   ```

2. **I (OpenClaw agent) receive the analysis prompt** showing:
   - Your complete AGENTS.md content
   - Commit pattern summary (45 features, 68 bug fixes, etc.)
   - Most changed files/directories
   - Pain points and commit samples

3. **I analyze and provide JSON response:**
   ```json
   {
     "gaps": [
       {
         "area": "Testing",
         "observation": "68 bug fixes but no testing guidance in AGENTS.md",
         "impact": "high",
         "suggestion": "Add testing section with coverage targets"
       }
     ],
     "strengths": [...],
     "recommendations": [...]
   }
   ```

4. **god-mode displays results** and offers to apply changes to your AGENTS.md

5. **You choose** which recommendations to accept, and god-mode updates the file

**Standalone Workflow (outside OpenClaw):**

If you set `ANTHROPIC_API_KEY` or `OPENAI_API_KEY`, god-mode calls the API directly:
```bash
export ANTHROPIC_API_KEY="sk-ant-..."
god agents analyze myproject  # Fully automated
```

### `god agents generate <project>` (Coming Soon)
Bootstrap agents.md for a new project by analyzing repo structure.

### `god logs [options]`
View activity logs:
```bash
god logs                # Last 50 lines
god logs -n 100         # Last 100 lines
god logs -f             # Follow log output
god logs --path         # Show log file location
god logs --clear        # Clear all logs
```

All god-mode activity is logged to `~/.god-mode/logs/activity.log` with timestamps for transparency and debugging.

## Configuration

Config file: `~/.config/god-mode/config.yaml`

```yaml
projects:
  - id: github:user/repo
    name: My Project      # Display name
    priority: high        # high/medium/low
    tags: [work, api]
    local: ~/code/myrepo  # Local clone path

sync:
  initialDays: 90         # First sync lookback
  commitsCacheMinutes: 60

analysis:
  agentFiles:             # Files to search for
    - agents.md
    - AGENTS.md
    - CLAUDE.md
    - .github/copilot-instructions.md
```

## Data Storage

All data stored locally in `~/.god-mode/`:
- `cache.db` - SQLite database (commits, PRs, issues, analyses)
- `contexts/` - Saved workspace contexts (v0.2)

## Authentication

god-mode uses your existing CLI authentication:

| Provider | CLI | Setup |
|----------|-----|-------|
| GitHub | `gh` | `gh auth login` |
| Azure | `az` | `az login` |
| GitLab | `glab` | `glab auth login` |

**No tokens stored by god-mode.** We delegate to CLIs you already trust.

## Requirements

- `gh` - GitHub CLI (for GitHub repos)
- `sqlite3` - Database
- `jq` - JSON processing

## Examples

### Morning Check-In
```bash
god status
# See all projects at a glance
# Notice any stale PRs or quiet projects
```

### Before Switching Projects
```bash
god status myproject
# See recent activity, open PRs, issues
# Remember where you left off
```

### Improving Your AI Assistant
```bash
god agents analyze myproject
# Get suggestions based on your actual commit patterns
# Apply recommendations to your agents.md
```

### Weekly Review
```bash
god status
# Review activity across all projects
# Identify projects needing attention
```

## Agent Guide (For OpenClaw Agents)

**When the user asks about their projects, here's what to do:**

### "Set up god-mode" / "Track my [repo]"
1. Run `god setup` (checks dependencies)
2. Run `god projects add github:username/repo`
3. Run `god sync` to fetch data
4. Summarize what you found

### "Show me my projects" / "What's happening?"
1. Run `god status`
2. Translate output to conversational summary:
   - "Your most active repo is X with Y commits this week"
   - "Z repo hasn't had activity in N days"
   - "You have M open PRs across all projects"

### "Analyze my agents.md"
1. Run `god agents analyze github:username/repo`
2. You'll receive the analysis prompt with:
   - Their complete AGENTS.md
   - Commit pattern analysis (155 commits, 68 bug fixes, etc.)
   - Pain points and frequently changed files
3. **Provide JSON analysis** in this format:
   ```json
   {
     "gaps": [
       {"area": "Testing", "observation": "68 bug fixes but no test guidance", "impact": "high", "suggestion": "Add testing section"}
     ],
     "strengths": [
       {"area": "TypeScript", "observation": "Zero type errors in 155 commits"}
     ],
     "recommendations": [
       {"priority": 1, "section": "## Testing", "content": "- Write unit tests for new code\n- Run tests before commits"}
     ],
     "summary": "Strong TypeScript practices, needs testing guidance"
   }
   ```
4. Summarize the analysis conversationally
5. Offer to apply recommendations

### "Which repos need attention?"
1. Run `god status`
2. Look for:
   - Stale PRs (>3 days old)
   - No activity in >5 days
   - Open issues piling up
3. Suggest what to focus on

### Automated Workflows

**Daily Briefing (Heartbeat):**
```markdown
# HEARTBEAT.md
- Run `god status` and summarize:
  - Projects with stale PRs (>3 days)
  - Projects with no activity (>5 days)
  - Open PRs needing review
```

**Monthly Review (Cron - 1st of month):**
```yaml
schedule:
  kind: cron
  expr: "0 9 1 * *"  # 9am on 1st of each month
  tz: "America/New_York"
payload:
  kind: agentTurn
  message: |
    Run god review for last month and summarize:
    - Which projects were most active?
    - Any projects that went quiet?
    - Major accomplishments from commit messages
    - Recommendations for next month
sessionTarget: isolated
```

**Weekly Analysis (Cron):**
```yaml
schedule: "0 9 * * 1"  # Monday 9am
task: |
  Run `god agents analyze` on high-priority projects.
  If gaps found, notify with suggestions.
```

## Common Questions

### How do I use god-mode?
**In OpenClaw:** Just ask me! "Show me my projects", "Analyze my agents.md", etc. I'll run the commands and explain the results.

**CLI:** Run commands directly: `god status`, `god sync`, `god agents analyze`

### Do I need to set up anything first?
**First time:** Tell me to "set up god-mode for [your repo]" and I'll handle it. Or run `god setup` and `god projects add github:your/repo` yourself.

**Authentication:** Make sure `gh auth login` is done (GitHub CLI authentication).

### Do I need an API key?
**No!** When you ask me to analyze your agents.md, I receive the analysis prompt and provide the JSON response directly. No separate API key needed.

**Standalone:** If using god-mode outside OpenClaw, you can set `ANTHROPIC_API_KEY` or `OPENAI_API_KEY` for automated analysis.

### How often does it sync data?
**When you ask:** I run `god sync` when you ask about your projects. First sync fetches 90 days of history. Subsequent syncs are incremental (only new data).

**Manual:** You can run `god sync` anytime, or `god sync --force` for a full refresh.

### What data gets stored?
**Locally only:** Commits, PRs, issues, and analysis results in `~/.god-mode/cache.db`. Activity logs in `~/.god-mode/logs/activity.log`. Nothing sent to external servers (except when calling LLM APIs if configured).

### Does it work with private repos?
**Yes!** Uses your `gh` CLI authentication, so it has access to whatever your GitHub account can access.

## Troubleshooting

### "gh: command not found"
Install GitHub CLI: https://cli.github.com/

### "Not logged in to GitHub"
Run: `gh auth login`

### "No projects configured"
Add a project: `god projects add github:user/repo`

### Stale data
Force refresh: `god sync --force`

### Agent analysis returns empty {}
This is normal in OpenClaw mode - the prompt is displayed for the OpenClaw agent to analyze. The agent provides the JSON response in conversation, not as return value.

---

*OpenClaw Community Skill*  
*License: MIT*  
*Repository: https://github.com/InfantLab/god-mode-skill*
