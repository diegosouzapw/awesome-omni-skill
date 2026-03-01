---
name: measure-ai-proficiency
description: Assess and improve repository AI coding proficiency and context engineering maturity. Use when users ask about: (1) AI readiness or AI maturity assessment, (2) context engineering quality or improvement, (3) CLAUDE.md, .cursorrules, or copilot-instructions files, (4) measuring how well a repo is prepared for AI coding assistants, (5) recommendations for improving AI collaboration, (6) what context files to add, or (7) comparing their repo to AI proficiency best practices.
---

# Measure AI Proficiency

Assess repository context engineering maturity and provide actionable recommendations for improving AI collaboration.

This skill works with Claude Code, GitHub Copilot, Cursor, and OpenAI Codex (via the [Agent Skills](https://agentskills.io/) open standard).

## Prerequisites

Install the measure-ai-proficiency tool:

```bash
pip install measure-ai-proficiency
```

## Workflow

### 1. Choose Your Scanning Method

#### Option A: Scan GitHub Directly (No Cloning Required!)

Scan GitHub repositories without cloning them:

```bash
# Scan a single GitHub repository
measure-ai-proficiency --github-repo owner/repo

# Scan entire GitHub organization
measure-ai-proficiency --github-org your-org-name

# Limit number of repos scanned
measure-ai-proficiency --github-org your-org-name --limit 50

# Output to file
measure-ai-proficiency --github-org your-org --format json --output report.json
```

**Requirements:** [GitHub CLI (gh)](https://cli.github.com/) authenticated with `gh auth login`

**How it works:**
- Uses GitHub API to fetch repository file tree
- Downloads only AI proficiency files (CLAUDE.md, .cursorrules, skills, etc.)
- Scans in temporary directories
- Cleans up automatically
- Much faster than cloning!

#### Option B: Discover Then Clone (Traditional Method)

For organizations wanting more control, first discover which repos have AI context artifacts:

```bash
# Find active repos (commits in last 90 days) with AI context files
./scripts/find-org-repos.sh your-org-name

# JSON output for automation
./scripts/find-org-repos.sh your-org-name --json > repos.json
```

**What you get:**
- Total repos in organization
- Active repos (with recent commits)
- Repos with AI context artifacts (CLAUDE.md, AGENTS.md, .cursorrules, etc.)
- Percentage baseline for your org
- List of repos to scan

**Requirements:** [GitHub CLI (gh)](https://cli.github.com/) and [jq](https://stedolan.github.io/jq/)

Then clone and scan the identified repos.

#### Option C: Scan Local Repositories

```bash
# Scan current directory
measure-ai-proficiency

# Scan specific repository
measure-ai-proficiency /path/to/repo

# Scan multiple repositories
measure-ai-proficiency /path/to/repo1 /path/to/repo2

# Scan all repos in directory (cloned org)
measure-ai-proficiency --org /path/to/org-repos
```

### 2. Run Assessment

Most common commands:

```bash
# Local scan
measure-ai-proficiency

# GitHub scan (recommended for orgs)
measure-ai-proficiency --github-org your-org-name
```

### 3. Interpret Results

**Maturity Levels (aligned with Steve Yegge's 8-stage model):**

| Level | Name | Yegge Stage | Indicators |
|-------|------|-------------|------------|
| 1 | Zero AI | Stage 1 | No AI-specific files (baseline) |
| 2 | Basic Instructions | Stage 2 | CLAUDE.md, .cursorrules exist |
| 3 | Comprehensive Context | Stage 3 | Architecture, conventions documented |
| 4 | Skills & Automation | Stage 4 | Hooks, commands, memory files, skills |
| 5 | Multi-Agent Ready | Stage 5 | Specialized agents, MCP configs |
| 6 | Fleet Infrastructure | Stage 6 | Beads, shared context, workflows |
| 7 | Agent Fleet | Stage 7 | Governance, scheduling, 10+ agents |
| 8 | Custom Orchestration | Stage 8 | Gas Town, meta-automation, frontier |

**Score interpretation:** File count matters more than percentage. The tool includes hundreds of patterns for comprehensive detection.

### Understanding Quality Scoring

Each AI instruction file is scored 0-10 based on quality indicators:

| Symbol | Indicator | What It Means | Points |
|--------|-----------|---------------|--------|
| § | Sections | Markdown headers (`##`) | 0-2 |
| ⌘ | Paths | File/dir paths (`/src/`) | 0-2 |
| $ | Commands | CLI in backticks | 0-2 |
| ! | Constraints | never/avoid/don't | 0-2 |
| ↻N | Commits | Git history (N commits) | 0-2 |

**Commit scoring:** Files with 5+ commits get full points (indicates active maintenance). 3-4 commits = 1pt.

### Cross-Reference Detection

The tool detects links between your AI instruction files:

- **Markdown links:** `[architecture](ARCHITECTURE.md)`
- **File mentions:** `"CONVENTIONS.md"` or `` `TESTING.md` ``
- **Relative paths:** `./docs/API.md`
- **Directory refs:** `skills/`, `.claude/commands/`

Resolution tracking shows if referenced files exist (helps find broken links).

**Bonus points:** Up to +10 points from cross-references (5 pts) + quality (5 pts).

### Content Validation

The tool validates that your documentation references real files:

- **Missing references:** Files mentioned in docs that don't exist
- **Stale references:** References to deleted files (detected via git history)
- **Template markers:** Uncustomized content (TODO, PLACEHOLDER, etc.)

**Validation penalty:** Up to -4 points for validation issues.

**Skip false positives:** If your docs contain example file names (meta-tools, templates), configure `skip_validation_patterns` in `.ai-proficiency.yaml`:

```yaml
skip_validation_patterns:
  - "COMPLIANCE.md"    # Example mentioned in docs
  - ".mcp.json"        # Best practice not yet adopted
  - "examples/*"       # All files under examples/
```

### 4. Provide Recommendations

After assessment, offer to create missing high-priority files:

**Level 2 gaps:** Create CLAUDE.md, .cursorrules, or .github/copilot-instructions.md

**Level 3 gaps:** Create ARCHITECTURE.md, CONVENTIONS.md, or TESTING.md

**Level 4 gaps:**
- Create skills directories: `.claude/skills/`, `.github/skills/`, or `.cursor/skills/`
- Add `.claude/commands/` for slash commands
- Create MEMORY.md or LEARNINGS.md
- Consider SOUL.md or IDENTITY.md (ClawdBot pattern) for agent personality
- **Boris Cherny's key insight:** Add verification loops (tests, linters) - this 2-3x quality

**Level 5 gaps:**
- Create specialized agents in `.github/agents/` or `.claude/agents/`
- Set up `.mcp.json` at root (Boris Cherny pattern) for team-shared tool configs
- Add agents/HANDOFFS.md for agent coordination

**Level 6 gaps:** Beads memory system, shared context, workflow pipelines

**Level 7 gaps:**
- Add GOVERNANCE.md for agent permissions and policies
- Set up convoys/ or molecules/ (Gas Town work decomposition)
- Consider swarm/, wisps/, polecats/ for advanced agent patterns

**Level 8 gaps:**
- Build custom orchestration in orchestration/
- Consider .gastown/ for Kubernetes-like agent management
- Add protocols: MAIL_PROTOCOL.md, FEDERATION.md, ESCALATION.md, watchdog/

### 5. Create Missing Files

When creating context files, include:

**CLAUDE.md structure:**
- Project overview (what it does, who it's for)
- Directory structure and key files
- Important conventions and patterns
- Common tasks and how to perform them
- Things to avoid

**ARCHITECTURE.md structure:**
- System overview and purpose
- Key components and responsibilities
- Data flow between components
- Important design decisions

**CONVENTIONS.md structure:**
- Naming conventions
- Code organization patterns
- Error handling approach
- Testing conventions

## Quick Reference

Common triggers for this skill:
- "Assess my AI proficiency"
- "How mature is my context engineering?"
- "What context files should I add?"
- "Help me improve for AI coding"
- "Check my CLAUDE.md setup"
- "Am I ready for AI-assisted development?"

## Customization

Use the **customize-measurement** skill for guided configuration:
```
"Customize measurement for my repo"
```

Or see the manual guide:
https://github.com/pskoett/measuring-ai-proficiency/blob/main/CUSTOMIZATION.md
