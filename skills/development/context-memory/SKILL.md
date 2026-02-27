---
name: context-memory
description: Advanced context and memory management system for AI agents. Provides persistent memory across sessions through daily logs (memory/YYYY-MM-DD.md), long-term curated memory (MEMORY.md), conversation archives with semantic search, and automatic behavioral learning from user satisfaction tracking. Use when you need to: (1) Remember information across sessions, (2) Archive conversations before context loss, (3) Search past discussions, (4) Track and learn from user satisfaction patterns, (5) Maintain session continuity, (6) Implement proactive memory maintenance. Includes conversation-archiver.py and satisfaction-tracker.py scripts, session startup routines, and periodic reflection workflows.
---

# Context & Memory System

A comprehensive memory management system that gives AI agents persistent context and continuous learning capabilities.

## What This Skill Provides

1. **Persistent Memory Architecture** - Multi-tier memory system (daily logs, long-term memory, conversation archives)
2. **Conversation Archive & Search** - Save and semantically search past conversations
3. **Satisfaction Tracking** - Learn from user reactions and behavioral patterns
4. **Auto-Reflection** - Daily summaries and behavioral insights
5. **Session Startup Routines** - Load context at the beginning of each session
6. **Memory Maintenance Workflows** - Periodic review and consolidation

## Quick Start

### 1. Setup Directory Structure

```bash
mkdir -p memory/conversations memory/satisfaction-insights
```

### 2. Create Core Files

See `references/templates.md` for templates:
- `MEMORY.md` - Long-term curated memory
- `LEARNING.md` - Behavioral insights (auto-generated)
- `memory/YYYY-MM-DD.md` - Today's daily log
- `memory/heartbeat-state.json` - Periodic check state

### 3. Configure Workspace

```bash
# Optional: Set workspace path (defaults to current directory)
export OPENCLAW_WORKSPACE=/path/to/your/workspace
```

### 4. Session Startup Routine

At the start of each session, read these files in order:
1. `SOUL.md` (if exists) - Who you are
2. `USER.md` (if exists) - Who you're helping
3. `LEARNING.md` - Behavioral insights
4. `memory/YYYY-MM-DD.md` (today + yesterday)
5. `MEMORY.md` - **Only in main session** (not in group chats)

## Core Workflows

### Archive Conversations

Before context compaction or topic switches:

```bash
python3 scripts/conversation-archiver.py archive '<messages_json>' '<topic>' '<summary>'
```

Search archived conversations:

```bash
python3 scripts/conversation-archiver.py search "keyword"
python3 scripts/conversation-archiver.py get <conv_id>
```

### Track Satisfaction

Record user reactions:

```bash
python3 scripts/satisfaction-tracker.py record "positive" "context" "user message" "my response" "analysis"
```

Signals: `negative`, `positive`, `interested`

Generate daily insights:

```bash
python3 scripts/satisfaction-tracker.py daily-summary
python3 scripts/satisfaction-tracker.py update-learning
```

### Memory Maintenance

Periodically (every few days):
1. Read recent `memory/YYYY-MM-DD.md` files
2. Identify significant events/learnings
3. Update `MEMORY.md` with distilled wisdom
4. Remove outdated information

## Security Model

**MEMORY.md is private** - Only load in main session (direct chats with your human):
- ✅ Load in: One-on-one conversations, private sessions
- ❌ Don't load in: Group chats, shared contexts, public channels

This prevents leaking personal context to other users.

## Memory Philosophy

**Files > Brain** - Memory doesn't survive session restarts. Files do.

- Daily logs = raw notes
- MEMORY.md = curated wisdom
- No "mental notes" - write everything down immediately
- Archive before losing context
- Review and consolidate periodically

## Detailed Documentation

- **Memory Guidelines:** `references/memory-guidelines.md` - Complete workflow documentation
- **Templates:** `references/templates.md` - File templates and directory structure

## Script Reference

### conversation-archiver.py

Archive conversation blocks with topics and summaries:

```bash
# Archive a conversation
conversation-archiver.py archive '<messages_json>' [topic] [summary]

# Search conversations
conversation-archiver.py search <query> [topic]

# Retrieve full conversation
conversation-archiver.py get <conv_id>

# List topics
conversation-archiver.py topics
```

**Environment:**
- Workspace: `OPENCLAW_WORKSPACE` (default: current directory)
- Archive location: `memory/conversations/`

### satisfaction-tracker.py

Track satisfaction and generate behavioral insights:

```bash
# Record an incident
satisfaction-tracker.py record <signal> <context> <user_msg> <my_response> [analysis]

# Analyze patterns
satisfaction-tracker.py analyze [days]

# Generate daily summary
satisfaction-tracker.py daily-summary

# Update LEARNING.md
satisfaction-tracker.py update-learning
```

**Environment:**
- Workspace: `OPENCLAW_WORKSPACE` (default: current directory)
- Output: `memory/satisfaction-insights/`, `LEARNING.md`

## Integration with OpenClaw

### Semantic Search

Use built-in tools before answering questions about history:

```
1. memory_search - Search MEMORY.md + memory/*.md semantically
2. memory_get - Retrieve specific snippets by path/lines
```

### Cron Jobs

Schedule daily reflection (example):

```json
{
  "name": "Daily satisfaction reflection",
  "schedule": {"kind": "cron", "expr": "0 23 * * *", "tz": "UTC"},
  "payload": {
    "kind": "systemEvent",
    "text": "Run satisfaction-tracker.py daily-summary and update-learning"
  },
  "sessionTarget": "main",
  "enabled": true
}
```

### Heartbeats

Use heartbeat polls for:
- Memory maintenance (review and consolidate)
- Periodic checks (track in `memory/heartbeat-state.json`)
- Proactive context updates

## When to Archive

- **Before context compaction** - Save conversations before pruning
- **Topic switches** - When conversation shifts to new subject
- **User request** - "Remember this" or "save this conversation"
- **End of session** - Preserve important discussions

## Active Learning Loop

1. **Track** - Record satisfaction signals during interactions
2. **Analyze** - Daily summaries identify patterns
3. **Learn** - Update LEARNING.md with insights
4. **Apply** - Read LEARNING.md on startup, adjust behavior
5. **Repeat** - Continuous improvement cycle

## Tips for Success

- **Start simple** - Begin with MEMORY.md and daily logs only
- **Build habits** - Update daily logs as events happen, not at end of day
- **Review regularly** - Use heartbeats for periodic maintenance
- **Trust the system** - Write everything down, don't rely on memory
- **Archive proactively** - Before context loss, not after
- **Consolidate wisely** - Promote only significant items to MEMORY.md

---

**Note:** This skill provides the infrastructure. Customize templates and workflows to match your specific needs and preferences.
