---
name: use-skills-npm-package
description: "CLI tool for discovering, installing, and managing reusable agent skills across multiple coding agents. Enables efficient skill discovery from repositories, local sources, and community repositories. Essential resource for discovering new Flutter/Dart skills."
---

# Using the Skills NPM Package

The `skills` npm package is a powerful CLI tool published by Vercel Labs that manages agent skills - reusable instruction sets that extend coding agent capabilities across multiple platforms.

> **💡 COMPANION SKILL:** The official [find-skills](https://github.com/vercel-labs/skills/tree/main/skills/find-skills) skill (from vercel-labs/skills package) provides AI-assisted skill discovery and evaluation. Always load it alongside this skill for enhanced discovery capabilities.

---

## What is the Skills Package?

The `skills` CLI is a unified interface for:
- **Discovering** skills from GitHub repositories and other sources
- **Installing** skills to multiple coding agents simultaneously
- **Managing** skill installations with updates, removal, and listing
- **Creating** new skills following the Agent Skills specification
- **Searching** skill repositories by keyword or interactively

**Official Documentation:** https://www.npmjs.com/package/skills

---

## Installation & Setup

### Install the CLI

```bash
npm install -g skills
# or use npx without installation
npx skills [command]
```

### Verify Installation

```bash
skills --version
npx skills --help
```

---

## Key Features & Capabilities

### ⭐ Enhanced Discovery with find-skills

**Use the official [find-skills](https://github.com/vercel-labs/skills/tree/main/skills/find-skills) skill** for intelligent skill discovery:
- AI-powered skill evaluation and recommendations
- Contextual skill search based on your needs
- Quality assessment and compatibility checking
- Integration guidance and best practices

Load this skill when you need to find skills for specific use cases.

### 1. Discover Skills

**Interactive Search** - Browse available skills visually:
```bash
npx skills find
```

**Keyword Search** - Find skills by specific criteria:
```bash
npx skills find typescript
npx skills find flutter
npx skills find supabase
```

### 2. Install Skills from Repositories

**From GitHub (shorthand):**
```bash
npx skills add vercel-labs/agent-skills
```

**From GitHub (full URL):**
```bash
npx skills add https://github.com/vercel-labs/agent-skills
```

**Direct skill path:**
```bash
npx skills add https://github.com/vercel-labs/agent-skills/tree/main/skills/web-design-guidelines
```

**From GitLab or other git sources:**
```bash
npx skills add https://gitlab.com/org/repo
npx skills add git@github.com:owner/repo.git
```

**From local path:**
```bash
npx skills add ./my-local-skills
```

### 3. Installation Options

**List available skills without installing:**
```bash
npx skills add vercel-labs/agent-skills --list
```

**Install specific skills:**
```bash
npx skills add owner/repo --skill frontend-design --skill skill-creator
```

**Install to specific agents** (e.g., GitHub Copilot, Claude Code):
```bash
npx skills add vercel-labs/agent-skills -a github-copilot -a claude-code
```

**Install all skills:**
```bash
npx skills add vercel-labs/agent-skills --all
```

**Global installation** (available across projects):
```bash
npx skills add vercel-labs/agent-skills -g
```

**Non-interactive (CI/CD friendly):**
```bash
npx skills add vercel-labs/agent-skills --skill frontend-design -y
```

### 4. Manage Installed Skills

**List all installed skills:**
```bash
npx skills list
# or
npx skills ls
```

**Filter by agent:**
```bash
npx skills ls -a claude-code -a cursor
```

**Check for updates:**
```bash
npx skills check
```

**Update skills to latest versions:**
```bash
npx skills update
```

**Remove skills:**
```bash
npx skills remove my-skill
npx skills remove --all  # Remove all
```

### 5. Create New Skills

**Create new skill template:**
```bash
npx skills init my-skill
```

**Skill structure (SKILL.md template):**
```markdown
---
name: my-skill
description: What this skill does and when to use it
---

# My Skill

Introduction and context...

## When to Use

Describe scenarios where this skill should be used.

## Steps

1. First, do this
2. Then, do that
```

---

## Supported Agents

The `skills` CLI works with 40+ coding agents including:

- **GitHub Copilot** - Copilot in VS Code and other editors
- **Claude Code** - Anthropic's Claude Code platform
- **Cursor** - Advanced code editor with AI
- **OpenCode** - Open-source agent platform
- **Cline** - Another popular coding agent
- **Windsurf** - Code editor with AI capabilities
- **Continue** - Open-source code completion
- And 30+ more agents (Amp, Kimi CLI, Replit, Codex, etc.)

Each agent stores skills in its own directory:
- `~/.claude/skills/` (Claude Code - global)
- `.claude/skills/` (Claude Code - project)
- `.cursor/skills/` (Cursor - project)
- `~/.cursor/skills/` (Cursor - global)
- Similar patterns for other agents

---

## Skill Discovery Workflow for Flutter

### Step 1: Search for Available Skills

```bash
# Search for Flutter-related skills
npx skills find flutter

# Search for state management
npx skills find "state management"

# Search for specific integrations
npx skills find supabase
```

### Step 2: List Before Installing

```bash
# See what's available in a repository
npx skills add vercel-labs/agent-skills --list

# Filter to specific skills
npx skills add some-org/flutter-skills --list
```

### Step 3: Preview Skill Content

When considering a skill:
1. View the source repository
2. Read the SKILL.md file directly on GitHub
3. Assess relevance and quality
4. Check last update date

### Step 4: Install Selected Skills

```bash
# Install specific Flutter-related skills
npx skills add my-flutter-repo/skills --skill "flutter-testing" --skill "performance-optimization"

# Install to project scope (committed with code)
npx skills add my-flutter-repo/skills --skill "flutter-testing"

# Install globally (available everywhere)
npx skills add my-flutter-repo/skills -g
```

### Step 5: Verify Installation

```bash
npx skills ls -a github-copilot
```

---

## Finding Flutter & Supabase Skills

### Official Skill Repositories

**Vercel Labs Agent Skills:**
- Repository: https://github.com/vercel-labs/agent-skills
- Command: `npx skills add vercel-labs/agent-skills`
- Contains frontend, backend, and general development skills

**Skills Directory:**
- Browse: https://skills.sh/
- Search and filter by topic, agent, and category
- Community-contributed and curated skills

### Searching for Specific Technologies

**Search patterns:**
```bash
npx skills find flutter
npx skills find supabase
npx skills find "firebase integration"
npx skills find "state management"
npx skills find testing
```

**GitHub search for skill repositories:**
- Search GitHub for "agent-skills" + technology
- Look for repositories with multiple SKILL.md files
- Check `.github/skills/`, `skills/`, or `.claude/skills/` directories

---

## Important: Always Check Current Documentation

**⚠️ This skill description is based on `skills` v1.4.1**

The npm package updates frequently with new features and agent support. When using this skill:

1. **Always check the latest documentation:**
   - https://www.npmjs.com/package/skills
   - GitHub repo: https://github.com/vercel-labs/skills

2. **Verify agent compatibility:**
   - New agents are added regularly
   - Agent paths may change
   - Check latest supported agents list

3. **Check for new search features:**
   - The CLI evolves with new commands
   - Search capabilities expand over time
   - New installation options may be available

4. **Review breaking changes:**
   - Before major version updates
   - Especially if using in CI/CD pipelines
   - Check release notes on npm

---

## Best Practices

### ✅ DOs

- **Use `--list` first** to preview available skills before installing
- **Search multiple sources** - Not all skills are in one repository
- **Keep skills updated** - Run `npx skills update` regularly
- **Organize by scope** - Use project scope for team-specific skills, global for personal tools
- **Version control** - Commit symlink information or skill references
- **Document reasoning** - Note why a skill was installed and what it does

### ❌ DON'Ts

- **Don't install all skills indiscriminately** - Clutter reduces clarity
- **Don't skip the --list preview** - Avoid installing irrelevant skills
- **Don't ignore skill sources** - Verify repository credibility
- **Don't forget to read SKILL.md** - Understand what you're installing
- **Don't use outdated repositories** - Watch for abandoned projects

---

## Troubleshooting

### "No skills found"

```bash
# Ensure you're searching valid repositories
npx skills add owner/repo --list

# Check for typos in repository name
# Verify repository contains SKILL.md files
```

### Skill not loading in agent

- Verify installation path: `npx skills ls`
- Check agent's skill directory exists
- Ensure SKILL.md has proper YAML frontmatter
- Try reinstalling: `npx skills add source --skill name -y`

### Permission errors

```bash
# For global installation
npx skills add repo -g

# Or use --copy instead of symlink
npx skills add repo --copy
```

### Conflicting skills

- Skills can have same name across agents
- Use `-a` flag to target specific agents
- Remove first if upgrading: `npx skills remove old-skill`

---

## Integration with Flutter Development

### Recommended Workflow

1. **When starting new project or feature:**
   ```bash
   npx skills find flutter
   npx skills add flutter-community/skills --list
   ```

2. **When exploring new patterns:**
   ```bash
   npx skills find "state management"
   npx skills find "testing"
   ```

3. **When integrating external services:**
   ```bash
   npx skills find supabase
   npx skills find firebase
   npx skills find stripe
   ```

4. **Keep skills current:**
   ```bash
   npx skills check
   npx skills update
   ```

---

## Commands Quick Reference

| Command | Purpose |
|---------|---------|
| `npx skills add <repo>` | Install skills from repository |
| `npx skills find [query]` | Search for skills |
| `npx skills list` | List installed skills |
| `npx skills check` | Check for updates |
| `npx skills update` | Update all skills |
| `npx skills remove <skill>` | Remove a skill |
| `npx skills init [name]` | Create new skill template |

---

## Resources

- **Official find-skills Skill:** https://github.com/vercel-labs/skills/tree/main/skills/find-skills
- **npm Package:** https://www.npmjs.com/package/skills
- **GitHub Repository:** https://github.com/vercel-labs/skills
- **Skills Directory:** https://skills.sh/
- **Agent Skills Spec:** https://agentskills.io/
- **Vercel Blog:** https://vercel.com/blog

