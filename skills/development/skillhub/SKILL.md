---
name: skillhub
description: Cross-platform Agent Skills manager. Use when user wants to sync, install, list, or manage skills across Claude Code, Codex CLI, and Gemini CLI.
---

# SkillHub

A meta-skill for managing Agent Skills across multiple AI coding platforms.

## Commands

- `/skillhub sync` - Sync all skills to detected platforms
- `/skillhub install @scope/name` - Install a specific skill
- `/skillhub list` - List available skills
- `/skillhub status` - Show sync status
- `/skillhub config` - View/edit configuration

## Usage

When the user invokes any skillhub command, execute the corresponding Python script:

```bash
python3 ~/.claude/skills/skillhub/scripts/skillhub.py <command> [args]
```

## Examples

User: "/skillhub sync"
Action: Run `python3 ~/.claude/skills/skillhub/scripts/skillhub.py sync`

User: "/skillhub install @anthropic/tdd"
Action: Run `python3 ~/.claude/skills/skillhub/scripts/skillhub.py install @anthropic/tdd`

User: "/skillhub list"
Action: Run `python3 ~/.claude/skills/skillhub/scripts/skillhub.py list`
