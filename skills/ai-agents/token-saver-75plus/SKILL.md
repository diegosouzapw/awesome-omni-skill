---
name: token-saver-75plus
description: Always-on token optimization + model routing protocol. Auto-classifies requests (T1-T4), routes execution to the cheapest capable model via sessions_spawn, and applies maximum output compression. Target: 75%+ token savings.
---

# Token Saver 75+ with Model Routing

## Core Principle
**Understand fully, execute cheaply.** The orchestrator must fully understand the task before routing. Never sacrifice comprehension for speed.

## Request Classifier (silent, every message)

| Tier | Pattern | Orchestrator | Executor |
|---|---|---|---|
| T1 | yes/no, status, trivial facts, quick lookups | Handle alone | — |
| T2 | summaries, how-to, lists, bulk processing, formatting | Handle alone OR spawn Groq | Groq (FREE) |
| T3 | debugging, multi-step, code generation, structured analysis | Orchestrate + spawn | Codex for code, Groq for bulk |
| T4 | strategy, complex decisions, multi-agent coordination, creative | **Spawn Opus** | Opus orchestrates, spawns Codex/Groq from within |

## Model Routing Table

| Model | Use For | Cost | Spawn with |
|---|---|---|---|
| `groq/llama-3.1-8b-instant` | Summarization, formatting, classification, bulk transforms — NO thinking | FREE | `model: "groq/llama-3.1-8b-instant"` |
| `openai/gpt-5.3-codex` | ALL code generation, code review, refactoring | $$$ | `model: "openai/gpt-5.3-codex"` |
| `openai/gpt-5.2` | Structured analysis, data extraction, JSON transforms | $$$ | `model: "openai/gpt-5.2"` |
| `anthropic/claude-opus-4-6` | Strategy, complex orchestration, failure recovery (T4 only) | $$$$ | `model: "anthropic/claude-opus-4-6"` |

## Routing via sessions_spawn

### When to spawn (MANDATORY)
- **Code generation of any kind** → spawn Codex
- **Bulk text processing (>3 items)** → spawn Groq
- **Complex multi-step tasks** → spawn Opus (T4)
- **Simple formatting/rewriting** → spawn Groq

### When NOT to spawn
- T1 questions (yes/no, time, status) — handle directly
- Single tool calls (calendar, web search) — handle directly
- Short responses that need no processing — handle directly

### Spawn patterns

**Groq (free bulk work):**
```
sessions_spawn(
  task: "<clear instruction with all context included>",
  model: "groq/llama-3.1-8b-instant"
)
```

**Codex (all code):**
```
sessions_spawn(
  task: "Write <language> code that <detailed spec>. Include comments. Output the complete file.",
  model: "openai/gpt-5.3-codex"
)
```

**Opus (T4 strategy):**
```
sessions_spawn(
  task: "<full context + goal>. You have full tool access. Use sessions_spawn with Codex for code and Groq for bulk subtasks.",
  model: "anthropic/claude-opus-4-6"
)
```

### Critical spawn rules
1. **Include ALL context in the task string** — spawned agents have no conversation history
2. **Be specific** — vague tasks waste tokens on clarification
3. **One task per spawn** — don't bundle unrelated work
4. **For code: always use Codex** — never write code yourself

## Output Compression (applies to ALL tiers, ALL models)

### Templates
- **STATUS:** OK/WARN/FAIL one-liner
- **CHOICE:** A vs B → Recommend: X (1 line why)
- **CAUSE→FIX→VERIFY:** 3 bullets max
- **RESULT:** data/output directly, no wrap-up

### Rules
- No filler. No restating the question. Lead with the answer.
- Bullets/tables/code > prose.
- Do not narrate routine tool calls.
- If user asks for depth ("why", "explain", "go deep") → allow more tokens for that turn only.

### Budget by tier
| Tier | Max output |
|---|---|
| T1 | 1-3 lines |
| T2 | 5-15 bullets |
| T3 | Structured sections, <400 words |
| T4 | Longer allowed, still dense |

## Tool Gating (before ANY tool call)
1. Already known? → No tool.
2. Batchable? → Parallelize.
3. Can a spawned Groq handle it? → Spawn instead of doing it yourself.
4. Cheapest path? → memory_search > partial read > full read > web.
5. Needed? → Do not fetch "just in case."

## Failure Protocol
- If Groq spawn fails → retry with GPT-5.2
- If Codex spawn fails → retry with GPT-5.2
- If orchestrator can't handle T3 → spawn Opus (escalate to T4)
- **Never retry same model.** Escalate.

## Measurement (when asked or during testing)
Append: `[~X tokens | Tier: Tn | Route: model(s) used]`
