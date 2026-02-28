---
name: skill-cli
description: Testing and using skill-cli for progressive disclosure across platforms (CI/CD, Cursor, Windsurf, hot-swapping)
---

# skill-cli Skill

Quick reference for testing and using skill-cli - the progressive disclosure tool for AI coding assistants.

## When to Use This Skill

Load when:
- Testing skill-cli functionality
- Verifying progressive disclosure works
- Setting up skill-cli in new environments
- Debugging skill-cli issues
- Learning how to use skill-cli

## Quick Reference

### What is skill-cli?

**A CLI tool that brings Claude Code's Skills system to any environment**

Key features:
- 🚀 Works in GitHub Actions, Cursor, Windsurf, Codex
- 🔄 Hot-swap skills without restarting IDE
- 📦 70-90% token savings via progressive disclosure
- 🌍 Zero dependencies (pure Node.js)
- 🔍 Search across all skills

### Basic Commands

```bash
./skill-cli.js list                    # List available skills
./skill-cli.js show <skill-name>       # Quick reference
./skill-cli.js files <skill-name>      # Available detail files
./skill-cli.js detail <skill> <file>   # Load detail file
./skill-cli.js search <query>          # Search skills
```

### Testing Checklist

Run these commands to verify skill-cli works:

```bash
# 1. List skills (should show this skill + example-skill)
./skill-cli.js list

# 2. Show this skill's quick reference
./skill-cli.js show skill-cli

# 3. Check for detail files
./skill-cli.js files skill-cli

# 4. Load testing guide
./skill-cli.js detail skill-cli TESTING

# 5. Search functionality
./skill-cli.js search "progressive disclosure"
```

### Environment Variable Override

```bash
# Test with different skills directory
SKILLS_DIR=/path/to/skills ./skill-cli.js list

# Test with Claude Code skills
SKILLS_DIR=~/.claude/skills ./skill-cli.js list
```

## Progressive Detail Loading

For deeper information, use Read tool to load:

- **TESTING.md** - Complete testing procedures and verification steps
- **TROUBLESHOOTING.md** - Common issues and solutions

## Common Use Cases

**GitHub Actions:**
```yaml
- run: |
    git clone https://github.com/raw391/skill-cli.git
    export SKILLS_DIR=./.ai-skills
    ./skill-cli/skill-cli.js show my-skill
```

**Cursor/Windsurf:**
```bash
# Add to project, point AI to it
SKILLS_DIR=.ai-skills ./skill-cli.js show my-skill
```

**Hot-swap development:**
```bash
# Test new skill without restarting IDE
SKILLS_DIR=/tmp/test-skills ./skill-cli.js show new-skill
```

## Expected Output

When working correctly:
- `list` shows skills with descriptions
- `show` displays formatted skill content
- `files` lists available detail files
- `detail` loads full documentation
- `search` finds and groups matches
- Exit codes: 0 for success, 1 for errors

## Philosophy

**Progressive disclosure saves tokens.** Load minimal context first (quick reference), add details only when needed. This mirrors how humans read documentation and respects token budgets.
