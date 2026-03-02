---
name: clawsaver
version: 1.4.0
description: "Behavior-change skill that trains your agent to batch related asks into fewer responses. No credentials required. Pure instruction-based — no scripts, no network calls."
metadata:
  {"openclaw":{"emoji":"💸","os":["darwin","linux","win32"]}}
---

# ClawSaver v1.1

> 💡 **Did you know?** Every message re-sends your full workspace context — SOUL.md, MEMORY.md, AGENTS.md, conversation history — before it even gets to your question. Ask 3 related things in 3 turns and you pay to load that context 3 times. ClawSaver fixes that.

ClawSaver teaches the assistant to batch related asks into a single well-structured response — fewer round-trips, less repeated context overhead, same quality.

---

## Commands

When a user runs a `/batch` command, respond as follows:

- **`/batch`** — Review the last 5 turns. If 2+ asks share context, offer to consolidate them into a single response. List what you'd combine and ask for confirmation.
- **`/batch status`** — Report how many asks have been batched this session vs. kept separate. Estimate tokens/requests saved.
- **`/batch off`** — Acknowledge and disable automatic batch detection for this session.
- **`/batch on`** — Re-enable batch detection (default state when skill is loaded).

---

## When to Use

Load this skill when:
- The user is asking multiple related questions in a session
- You notice follow-up questions on the same topic across recent turns
- You want to reduce repeated context load overhead
- The user is on a quota-limited plan and burning requests on related asks

Do **not** batch when:
- Task A's output is required input for Task B (sequential dependency)
- The user explicitly says "answer each one separately"
- Topics are genuinely unrelated (3+ distinct domains)
- The user needs incremental output to review as it arrives

---

## Core Behavior

Apply these rules before every response when ClawSaver is active:

### Batch Decision Rules

| Signal | Action |
|--------|--------|
| Same topic, 2+ questions | Batch into one response |
| Follow-up within 3 turns | Offer to consolidate |
| Same data fetched twice | Cache and reuse |
| Clarifying Q from context | Anticipate and answer |
| Unrelated tasks | Keep separate |

### Trigger Phrases to Watch For

- "Also..." / "And one more thing..." / "While you're at it..."
- "Can you also check..." / "What about..."
- Multiple `?` in a single message
- "Before you answer, also tell me..."

---

## Response Structure (When Batching)

```
╭──────────────────────────────────────────────╮
│  💸 ClawSaver — Batching 3 asks → 1 response │
╰──────────────────────────────────────────────╯

**[Q1 / Task 1]**
Answer here.

**[Q2 / Task 2]**
Answer here.

**[Q3 / Task 3]**
Answer here.

---
💸 Batched 3 asks → 1 response · Est. savings: ~2 API calls · ~800 tokens
```

The savings footer is optional — include when it adds value, skip in high-frequency task flows.

---

## Cost Impact Reference

**💰 Token-based** (OpenRouter, Anthropic, OpenAI):

| Session | Before | After | Saved |
|---------|--------|-------|-------|
| Code review (3-part) | ~9,000 tok | ~4,500 tok | ~50% |
| Config questions (5) | ~15,000 tok | ~6,000 tok | ~60% |
| Research & summary | ~12,000 tok | ~5,000 tok | ~58% |
| Unrelated tasks | — | — | 0% |

**🎫 Call-based / quota** (GitHub Copilot, enterprise):

| Session | Before | After | Saved |
|---------|--------|-------|-------|
| Code review (3-part) | 3 req | 1 req | 67% |
| Status + next steps | 2 req | 1 req | 50% |
| Config questions (5) | 5 req | 2 req | 60% |
| Research & summary | 3 req | 1 req | 67% |
| Unrelated tasks | 2 req | 2 req | 0% |

---

## Safety

- ✅ **Never merges sequential tasks** — if result A feeds task B, they stay separate
- ✅ **Never compresses for brevity** — batched responses are structured, not squeezed
- ✅ **Never assumes context** — if combining requires guessing, it asks instead
- ✅ **Explicit opt-out** — "answer each one separately" and ClawSaver defers immediately

---

## Version History

- **1.4.0** — Removed analyze.py; ClawSaver is now pure behavior-change skill with zero credentials/scripts
- **1.3.3** — Declared `OPENROUTER_MANAGEMENT_KEY` as optional env; addressed ClawHub security scanner
- **1.3.1** — Honest scope language: behavior-change skill, not active interceptor
- **1.3.0** — Dual registration: `skills.entries` + `agents.list`; full install docs
- **1.2.0** — Proper openclaw.json install instructions; dogfooded on own instance
- **1.1.0** — Added `/batch` commands, dashboard preview, cost hook, ✅ safety format
- **1.0.0** — Initial release
