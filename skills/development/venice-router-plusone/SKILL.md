---
name: venice-router
version: 1.5.0
description: Supreme model router for Venice.ai — the privacy-first, uncensored AI platform. Automatically classifies query complexity and routes to the cheapest adequate model. Supports web search, uncensored mode, private-only mode (zero data retention), conversation-aware routing, cost budgets, function calling, thinking/reasoning mode, and 35+ Venice.ai text models. Use when the user wants to chat via Venice.ai, send prompts through Venice, or needs smart model selection to minimize API costs while keeping data private from Big Tech.
homepage: https://venice.ai
source: https://github.com/PlusOne/venice.ai-router-openclaw
user-invocable: true
metadata:
  openclaw:
    emoji: "🦞🚀"
    homepage: https://github.com/PlusOne/venice.ai-router-openclaw
    os: ["linux", "macos"]
    requires:
      bins: ["python3"]
      env: ["VENICE_API_KEY"]
    primaryEnv: VENICE_API_KEY
    optionalEnv:
      - VENICE_DEFAULT_TIER
      - VENICE_MAX_TIER
      - VENICE_TEMPERATURE
      - VENICE_MAX_TOKENS
      - VENICE_STREAM
      - VENICE_UNCENSORED
      - VENICE_PRIVATE_ONLY
      - VENICE_WEB_SEARCH
      - VENICE_THINKING
      - VENICE_DAILY_BUDGET
      - VENICE_SESSION_BUDGET
    notes: "Python 3.8+ (stdlib only, no pip dependencies). All scripts bundled under scripts/. Source: https://github.com/PlusOne/venice.ai-router-openclaw"
    cliHelp: "python3 venice-router.py --help\nusage: venice-router.py [-h] [--prompt PROMPT] [--tier {cheap,budget,budget-medium,mid,high,premium}] [--model MODEL] [--classify CLASSIFY] [--list-models] [--stream] [--temperature TEMPERATURE] [--max-tokens MAX_TOKENS] [--system SYSTEM] [--prefer-anon] [--uncensored] [--private-only] [--web-search] [--character CHARACTER] [--json] [--thinking] [--conversation CONVERSATION] [--tools TOOLS] [--tool-choice TOOL_CHOICE] [--budget-status] [--session-id SESSION_ID]\nExamples: --prompt \"What is 2+2?\" | --tier mid --prompt \"Explain recursion\" | --stream --prompt \"Write a haiku\" | --web-search --prompt \"Latest AI news\" | --uncensored --prompt \"Creative fiction\" | --private-only --prompt \"Sensitive data\" | --thinking --prompt \"Prove the halting problem\" | --conversation history.json --prompt \"continue\" | --tools tools.json --prompt \"Get weather\" | --budget-status | --classify \"Design a microservices architecture\" | --list-models"
    install:
      - id: python-brew
        kind: brew
        formula: python
        bins: ["python3"]
        label: "Install Python (brew)"
---

# Venice.ai Supreme Router

Smart, cost-optimized model routing for [Venice.ai](https://venice.ai) — the AI platform for people who don't want Big Tech watching over their shoulder.

Unlike OpenAI, Anthropic, and Google — where every prompt is logged, analyzed, and potentially used to train future models — Venice offers **true privacy** with zero data retention on private models. Your conversations stay yours. Venice is also **uncensored**: no content filters, no refusals, no "I can't help with that."

## Setup

1. Get a Venice.ai API key from [venice.ai/settings/api](https://venice.ai/settings/api)
2. Set the environment variable:

```bash
export VENICE_API_KEY="your-key-here"
```

Or configure in `~/.openclaw/openclaw.json`:

```json
{
  "skills": {
    "entries": {
      "venice-router": {
        "enabled": true,
        "apiKey": "YOUR_VENICE_API_KEY"
      }
    }
  }
}
```

## Usage

### Route a prompt (auto-selects model)

```bash
python3 {baseDir}/scripts/venice-router.py --prompt "What is 2+2?"
```

### Force a specific tier

```bash
python3 {baseDir}/scripts/venice-router.py --tier cheap --prompt "Tell me a joke"
python3 {baseDir}/scripts/venice-router.py --tier budget-medium --prompt "Write a Python function"
python3 {baseDir}/scripts/venice-router.py --tier mid --prompt "Explain quantum computing"
python3 {baseDir}/scripts/venice-router.py --tier premium --prompt "Write a distributed systems architecture"
```

### Stream output

```bash
python3 {baseDir}/scripts/venice-router.py --stream --prompt "Write a poem about lobsters"
```

### Web search (LLM searches the web and cites sources)

```bash
python3 {baseDir}/scripts/venice-router.py --web-search --prompt "Latest news on AI regulation"
```

### Uncensored mode (prefer models with no content filters)

```bash
python3 {baseDir}/scripts/venice-router.py --uncensored --prompt "Write edgy creative fiction"
```

### Private-only mode (zero data retention, no Big Tech proxying)

```bash
python3 {baseDir}/scripts/venice-router.py --private-only --prompt "Analyze this confidential contract"
```

### Conversation-aware routing (multi-turn context)

```bash
# Save conversation history as JSON, then route follow-ups with context
python3 {baseDir}/scripts/venice-router.py --conversation history.json --prompt "Can you add tests too?"
```

The router analyzes conversation history to keep context: trivial follow-ups ("thanks") go cheap, while follow-ups in complex code discussions stay at the right tier.

### Function calling (tool use)

```bash
# Define tools in a JSON file (OpenAI tools format)
python3 {baseDir}/scripts/venice-router.py --tools tools.json --prompt "What's the weather in NYC?"
python3 {baseDir}/scripts/venice-router.py --tools tools.json --tool-choice auto --prompt "Search for latest AI news"
```

Tool definitions use the standard OpenAI format. The router auto-bumps to `mid` tier minimum for function calling since it requires capable models.

### Cost budget tracking

```bash
# Show current spending
python3 {baseDir}/scripts/venice-router.py --budget-status

# Track per-session costs
python3 {baseDir}/scripts/venice-router.py --session-id my-project --prompt "help me code"
```

Set `VENICE_DAILY_BUDGET` and/or `VENICE_SESSION_BUDGET` to enforce spending limits. The router auto-downgrades tiers as you approach budget limits.

### Classify only (no API call)

```bash
python3 {baseDir}/scripts/venice-router.py --classify "Explain the Riemann hypothesis"
```

### List available models and tiers

```bash
python3 {baseDir}/scripts/venice-router.py --list-models
```

### Override model directly

```bash
python3 {baseDir}/scripts/venice-router.py --model deepseek-v3.2 --prompt "Hello"
```

## Tiers

| Tier | Models | Cost (input/output per 1M tokens) | Best For |
|------|--------|-----------------------------------|----------|
| **cheap** | Venice Small (qwen3-4b), GLM 4.7 Flash, GPT OSS 120B, Llama 3.2 3B | $0.05–$0.15 / $0.15–$0.60 | Simple Q&A, greetings, math, lookups |
| **budget** | Qwen 3 235B, Venice Uncensored, GLM 4.7 Flash Heretic | $0.14–$0.20 / $0.75–$0.90 | Moderate questions, summaries, translations |
| **budget-medium** | Grok Code Fast, DeepSeek V3.2, MiniMax M2.1 | $0.25–$0.40 / $1.00–$1.87 | Moderate-to-complex tasks, code snippets, structured output |
| **mid** | DeepSeek V3.2, MiniMax M2.1/M2.5, Qwen3 Thinking 235B, Venice Medium, Llama 3.3 70B | $0.25–$0.70 / $1.00–$3.50 | Code generation, analysis, longer writing, reasoning |
| **high** | GLM 5, Kimi K2 Thinking, Kimi K2.5, Grok 4.1 Fast, Hermes 3 405B, Gemini 3 Flash | $0.50–$1.10 / $1.25–$3.75 | Complex reasoning, multi-step tasks, code review |
| **premium** | GPT-5.2, GPT-5.2 Codex, Gemini 3 Pro, Gemini 3.1 Pro (1M ctx), Claude Opus/Sonnet 4.5/4.6 | $2.19–$6.00 / $15.00–$30.00 | Expert-level analysis, architecture, research papers |

## Routing Strategy

The router classifies each prompt using keyword + heuristic analysis:

1. **Length** — longer prompts suggest more complex tasks
2. **Keywords** — domain-specific terms (e.g., "architecture", "optimize", "prove") signal complexity
3. **Code markers** — presence of code blocks, function names, or technical syntax
4. **Instruction depth** — multi-step instructions, comparisons, or "explain in detail" bump the tier
5. **Conversational simplicity** — greetings, yes/no, small talk stay on the cheapest tier
6. **Conversation history** — when `--conversation` is provided, analyzes full chat context: code in history boosts tier, trivial follow-ups ("thanks") downgrade, tool calls in history signal complexity
7. **Function calling** — `--tools` auto-bumps to at least `mid` tier (capable models required)
8. **Thinking/reasoning mode** — `--thinking` prefers chain-of-thought reasoning models (Qwen3 Thinking, Kimi K2) and bumps to at least `mid` tier
9. **Budget constraints** — progressive tier downgrade as spending approaches daily/session limits (95% → cheap, 80% → budget, 60% → mid, 40% → high)

The classifier errs on the side of cheaper models — it only escalates when there's strong signal for complexity.

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `VENICE_API_KEY` | Venice.ai API key (required) | — |
| `VENICE_DEFAULT_TIER` | Minimum floor tier — auto-classification never goes below this. Valid: `cheap`, `budget`, `budget-medium`, `mid`, `high`, `premium` | `budget` |
| `VENICE_MAX_TIER` | Maximum tier to ever use (cost cap) | `premium` |
| `VENICE_TEMPERATURE` | Default temperature | `0.7` |
| `VENICE_MAX_TOKENS` | Default max tokens | `4096` |
| `VENICE_STREAM` | Enable streaming by default | `false` |
| `VENICE_UNCENSORED` | Always prefer uncensored models | `false` |
| `VENICE_PRIVATE_ONLY` | Only use private models (zero data retention) | `false` |
| `VENICE_WEB_SEARCH` | Enable web search by default ($10/1K calls) | `false` |
| `VENICE_THINKING` | Always prefer thinking/reasoning models | `false` |
| `VENICE_DAILY_BUDGET` | Max daily spend in USD (0 = unlimited) | `0` |
| `VENICE_SESSION_BUDGET` | Max per-session spend in USD (0 = unlimited) | `0` |

## Why Venice.ai?

- **🔒 Private inference** — Models marked "Private" have zero data retention. Your data never trains anyone's model.
- **🔓 Uncensored** — No guardrails blocking legitimate use cases. No refusals, no filters.
- **🔌 OpenAI-compatible** — Same API format, just change the base URL. Drop-in replacement.
- **📦 30+ models** — From tiny efficient models ($0.05/M) to Claude Opus 4.6 and GPT-5.2.
- **🌐 Built-in web search** — LLMs can search the web and cite sources in a single API call.

## Tips

- Use `--classify` to preview which tier a prompt would hit before spending tokens
- Set `VENICE_MAX_TIER=mid` to cap costs and never hit premium models
- Use `--uncensored` for creative, security research, or other content mainstream AI won't touch
- Use `--private-only` when processing sensitive/confidential data — zero retention guaranteed
- Use `--web-search` when you need up-to-date information with cited sources
- Use `--conversation` with a JSON message history for smarter multi-turn routing
- Use `--tools` to enable function calling — the router auto-bumps to capable models
- Set `VENICE_DAILY_BUDGET=1.00` to cap daily spend at $1 — the router auto-downgrades tiers as you approach the limit
- Use `--budget-status` to see a detailed breakdown of your spending by tier
- Use `--thinking` for math proofs, logic puzzles, and multi-step reasoning — routes to Qwen3 Thinking or Kimi K2 models
- The router prefers **private** (self-hosted) Venice models over anonymized ones when available at the same tier
- When `--uncensored` is active, the router auto-bumps to the nearest tier with uncensored models
- Combine with OpenClaw WebChat for a seamless chat experience routed through Venice.ai
