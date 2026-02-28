---
name: amnesia
description: Set up persistent memory and identity for AI agents. Use when the user wants to give an agent persistent identity, manage agent memory files, set up IDENTITY.md/SOUL.md/MEMORY.md, create memory backup systems, or implement file-based agent continuity. Triggers on "agent memory," "persistent identity," "IDENTITY.md," "MEMORY.md," "SOUL.md," "agent backup," "remember across sessions," or "agent continuity."
---

# Amnesia

Memory management toolkit for AI agents. Because forgetting is not an option.

## Quick Start

1. Copy templates to workspace:
```bash
cp assets/templates/*.md /path/to/agent/workspace/
```

2. Fill in each template with agent-specific information

3. Set up automated backups:
```bash
cp scripts/backup.sh /path/to/agent/scripts/
chmod +x /path/to/agent/scripts/backup.sh
```

## Core Files

| File | Purpose |
|------|---------|
| `IDENTITY.md` | Who the agent is — name, birthday, capabilities |
| `SOUL.md` | How the agent behaves — values, style, boundaries |
| `MEMORY.md` | Curated long-term memories |
| `USER.md` | Information about the primary user |
| `TOOLS.md` | Environment-specific notes |
| `memory/*.md` | Daily episodic logs |

## Setting Up Identity

### IDENTITY.md

Define static facts about the agent:

```markdown
# IDENTITY.md - Who Am I?

- **Name:** [Agent name]
- **Creature:** [What the agent is]
- **Vibe:** [One-line personality]
- **Emoji:** [Representative emoji]
- **Birthday:** [Creation date]
```

### SOUL.md

Define behavioral guidelines:

```markdown
# SOUL.md - Who You Are

## Core Truths
[Fundamental principles]

## Boundaries
[What the agent won't do]

## Vibe
[Communication style]
```

## Memory Architecture

### Long-term Memory (MEMORY.md)

Curated, important information:
- Key learnings with dates
- Important facts
- People and relationships
- Project status
- Preferences discovered
- Mistakes to avoid

### Daily Logs (memory/YYYY-MM-DD.md)

Raw episodic memory:
- Session starts
- Events and timestamps
- Conversations
- Learnings
- Tomorrow's tasks

**Maintenance cycle:** Periodically review daily logs → extract insights → update MEMORY.md

## Backup System

### backup.sh

Backs up workspace to git:

```bash
./scripts/backup.sh [workspace_dir] [backup_repo_dir]
```

Run on a cron schedule (e.g., hourly):
```
0 * * * * /path/to/scripts/backup.sh
```

### restore.sh

Restores from backup after a wipe:

```bash
./scripts/restore.sh [backup_repo_dir] [workspace_dir]
```

### new-day.sh

Initializes today's memory log:

```bash
./scripts/new-day.sh [workspace_dir]
```

## Session Startup Checklist

When an agent starts a new session:

1. Read `IDENTITY.md` — establish who you are
2. Read `SOUL.md` — establish how you behave
3. Read `USER.md` — know who you're helping
4. Read `MEMORY.md` — recall long-term context
5. Read `memory/YYYY-MM-DD.md` (today + yesterday) — recent context
6. Check `TOOLS.md` — environment specifics

## Best Practices

- **Save immediately** — Don't rely on context; write to files
- **Date your entries** — When was this learned?
- **Curate aggressively** — Not everything goes in MEMORY.md
- **Back up frequently** — Hourly is good, more often is better
- **Version control everything** — Git is your friend

## Resources

- Templates: `assets/templates/`
- Scripts: `scripts/`
- GitHub: https://github.com/mulletmcnasty/amnesia