---
name: llm
description: Universal LLM Router — route prompts to any model across all providers
platform: claude-code
invoke: /llm
---

# /llm — Universal LLM Router

Route prompts to any LLM model across all providers. CLI-first for Codex/Kimi/Claude (zero cost), Google API for Gemini, OpenRouter for everything else. Auto-discovers new models.

## When to Use

- You need to call a specific LLM model (not the current Claude Code session)
- You want to compare outputs across different models
- A task benefits from a specific model's strengths (e.g., GPT-5.3-Codex for code generation, Gemini for grounded search)
- You need to route through a specific provider (e.g., force OpenRouter for a model that usually goes through CLI)

## When NOT to Use

- The current Claude Code session can handle the task directly (don't call yourself via CLI)
- You need Aristotle formal verification — use `/prove` instead
- You need multi-model debate — use `/debate` instead
- You need parallel Codex swarm — use `/CodexCode` instead

## Quick Start

```bash
# Call a model
python3 ~/.claude/skills/llm/scripts/llm_route.py --model opus --prompt "Say hello in 3 words"

# With system prompt
python3 ~/.claude/skills/llm/scripts/llm_route.py --model gpt-5.3-codex --prompt "Write fizzbuzz" --system "You are a Python expert"

# From files
python3 ~/.claude/skills/llm/scripts/llm_route.py --model gemini-3-pro --prompt-file prompt.txt --system-file system.txt

# Force a different provider
python3 ~/.claude/skills/llm/scripts/llm_route.py --model opus --prompt "Hello" --route openrouter

# JSON output with metadata
python3 ~/.claude/skills/llm/scripts/llm_route.py --model opus --prompt "Hello" --json

# Custom parameters
python3 ~/.claude/skills/llm/scripts/llm_route.py --model opus --prompt "Hello" --temperature 0.3 --max-tokens 8192 --timeout 300

# Pipe prompt via stdin
echo "Explain quantum computing" | python3 ~/.claude/skills/llm/scripts/llm_route.py --model opus

# List models
python3 ~/.claude/skills/llm/scripts/llm_route.py --list-models
python3 ~/.claude/skills/llm/scripts/llm_route.py --list-models --all
python3 ~/.claude/skills/llm/scripts/llm_route.py --list-models --tier 2

# List providers
python3 ~/.claude/skills/llm/scripts/llm_route.py --list-providers
```

## Routing Hierarchy

Models are routed to providers in this priority order:

| Model prefix | Provider | Cost | Auth |
|---|---|---|---|
| Anthropic (opus, sonnet, haiku) | Claude CLI | Subscription (free) | `claude login` |
| OpenAI (gpt-5.2, gpt-5.3-codex) | Codex CLI | Subscription (free) | `codex login` |
| Moonshot (kimi-2.5) | Kimi CLI | Subscription (free) | `kimi login` |
| Google (gemini-3-pro, gemini-3-flash) | Google GenAI API | Free tier | `GOOGLE_API_KEY` |
| Everything else | OpenRouter | Per-token | `OPENROUTER_API_KEY` |

Use `--route <provider>` to override the default route for any model. For example, `--route openrouter` forces a CLI model through OpenRouter instead.

> **Important: CLI tools must be installed on the machine running Claude Code.** The CLI-first routing for GPT models (Codex CLI) and Kimi models (Kimi CLI) requires those CLIs to be installed and authenticated locally. If you're running Claude Code on a remote server, CI runner, or any machine without these CLIs, those routes will silently fail. Use `--route openrouter` to force API-based routing instead, or install the CLIs:
> ```bash
> npm install -g @openai/codex && codex login     # GPT models
> npm install -g kimi-cli && kimi login            # Kimi models
> ```
> Claude CLI is always available since you're already running inside Claude Code.

## Model Tiers

| Tier | Description | Auto-update | Default visibility |
|---|---|---|---|
| 1 | Manually curated (11 models) | Never overwritten | Always shown |
| 2 | Auto-discovered notable (major provider, context >= 32k) | Added automatically | Shown by default |
| 3 | Auto-discovered everything else | Added automatically | Hidden (use `--all`) |

### Tier 1 Models (Curated)

| Name | Provider | Description |
|---|---|---|
| opus | Claude CLI | Claude Opus 4.6 — most capable |
| sonnet | Claude CLI | Claude Sonnet 4.5 — fast + capable |
| haiku | Claude CLI | Claude Haiku 4.5 — fastest |
| gpt-5.3-codex | Codex CLI | GPT-5.3 — best for code, reasoning_effort=xhigh |
| gpt-5.2 | Codex CLI | GPT-5.2 — strong general purpose |
| gemini-3-pro | Google API | Gemini 3 Pro — thinkingLevel=HIGH |
| gemini-3-flash | Google API | Gemini 3 Flash — fast + grounded |
| kimi-2.5 | Kimi CLI | Kimi 2.5 — --thinking flag |
| glm-5 | OpenRouter | GLM-5 — ZhipuAI, built-in thinking |
| minimax-m2.5 | OpenRouter | MiniMax M2.5 — built-in thinking |
| aristotle | Aristotle | Formal theorem prover (use /prove) |

## Auto-Discovery

Discover new models from OpenRouter and Google APIs:

```bash
# Dry run — show what's new
python3 ~/.claude/skills/llm/scripts/discover_models.py

# Apply — update the registry
python3 ~/.claude/skills/llm/scripts/discover_models.py --apply

# Query only one source
python3 ~/.claude/skills/llm/scripts/discover_models.py --source openrouter
python3 ~/.claude/skills/llm/scripts/discover_models.py --source google
```

Discovery never overwrites tier 1 models. New models from major providers with context >= 32k become tier 2; everything else is tier 3.

## Prompting Overrides

Per-model temperature and system prompt wrapping is applied automatically from `settings/prompting-overrides.json`:

- **Claude** (opus/sonnet/haiku): XML format preferred, no temperature override
- **GPT** (gpt-5.2/gpt-5.3-codex): CTCO framework preamble, temperature not overridden (reasoning model)
- **Gemini** (gemini-3-pro/gemini-3-flash): Forced temperature=1.0, concise preamble
- **Kimi** (kimi-2.5): temperature=1.0
- **MiniMax** (minimax-m2.5): temperature=1.0

## Credential Resolution

API keys are resolved in this order:
1. Environment variable (e.g., `GOOGLE_API_KEY`)
2. Debate agent's shared keys at `~/.claude/skills/convolutional-debate-agent/api-keys/provider-keys.env`
3. Error with setup instructions

CLI tools (claude, codex, kimi) use their own stored logins — no API keys needed.

## File Structure

```
~/.claude/skills/llm/
├── SKILL.md                              # This file
├── scripts/
│   ├── llm_route.py                      # Core router (provider calls + CLI)
│   ├── discover_models.py                # Auto-discovery from OpenRouter/Google
│   └── fetch_benchmarks.py              # Fetch benchmarks from public leaderboards
├── settings/
│   ├── model-registry.json               # All known models + routes + tiers
│   ├── routing-rules.json                # Regex patterns for auto-routing new models
│   ├── prompting-overrides.json          # Per-model temperature + system preambles
│   └── benchmark-quality.json            # BetterBench quality metadata per benchmark
├── benchmarks/
│   ├── rankings.csv                      # Unified rankings — THE file other skills read
│   └── _meta.json                        # Fetch timestamps and source status
└── references/
    ├── provider-setup.md                 # Auth setup per provider
    └── betterbench-notes.md              # BetterBench paper findings + methodology
```

## Benchmarks

Fetch and cache LLM benchmark rankings from public leaderboards. The unified CSV at `benchmarks/rankings.csv` is the canonical source other skills should read for model comparisons.

### Data Sources

| Source | What it provides | Update frequency | Auth needed |
|---|---|---|---|
| **Chatbot Arena** (LMArena) | Elo rankings from human preference voting | Monthly (arena-catalog JSON, Dec 2025) | None |
| **Epoch AI** | GPQA Diamond, MATH, SWE-bench, coding, LiveBench scores | Daily CSV updates (has 2026 data) | None |
| **OpenRouter** | Pricing ($/1M tokens), context length | Real-time | None |
| **Artificial Analysis** | Intelligence Index (0-100), speed (TPS/TTFT), eval scores | Continuous | `ARTIFICIAL_ANALYSIS_API_KEY` |

**Evaluated but skipped**: LiveBench (already in Epoch AI data), LM Council (aggregator of our sources), LLM Stats/ZeroEval (aggregator of our sources).

**Quality methodology**: Informed by [BetterBench](https://arxiv.org/abs/2411.12990) (NeurIPS 2024 Spotlight). Each model row gets a `benchmark_quality` tier (high/medium/low) based on which high-quality data sources contributed scores. See `references/betterbench-notes.md` for details.

### Fetch Commands

```bash
# Fetch all sources, update rankings.csv
python3 ~/.claude/skills/llm/scripts/fetch_benchmarks.py

# Fetch only one source
python3 ~/.claude/skills/llm/scripts/fetch_benchmarks.py --source arena
python3 ~/.claude/skills/llm/scripts/fetch_benchmarks.py --source epoch
python3 ~/.claude/skills/llm/scripts/fetch_benchmarks.py --source openrouter
python3 ~/.claude/skills/llm/scripts/fetch_benchmarks.py --source aa

# View local rankings (no network calls)
python3 ~/.claude/skills/llm/scripts/fetch_benchmarks.py --list
python3 ~/.claude/skills/llm/scripts/fetch_benchmarks.py --list --top 50

# Look up a specific model
python3 ~/.claude/skills/llm/scripts/fetch_benchmarks.py --model opus
python3 ~/.claude/skills/llm/scripts/fetch_benchmarks.py --model gemini-3-pro --json
```

### CSV Schema (`benchmarks/rankings.csv`)

| Column | Description | Source |
|---|---|---|
| `model` | Display name from best available source | All |
| `provider` | Organization (anthropic, openai, google, etc.) | All |
| `arena_elo` | Chatbot Arena Elo score (human preference) | Arena |
| `gpqa` | GPQA Diamond score (%) — PhD-level science reasoning | Epoch AI, AA |
| `mmlu` | MMLU score (%) — knowledge + reasoning | Epoch AI, AA |
| `coding` | Best coding score (Aider polyglot, %) | Epoch AI, AA |
| `math` | MATH Level 5 score (%) | Epoch AI, AA |
| `swe_bench` | SWE-bench Verified resolve rate (%) | Epoch AI |
| `aa_index` | Artificial Analysis Intelligence Index (0-100 composite) | AA |
| `quality_index` | LiveBench global average | Epoch AI |
| `speed_tps` | Tokens per second (median) | AA |
| `speed_ttft` | Time to first token in seconds | AA |
| `context` | Context window (tokens) | OpenRouter, AA |
| `price_in` | Input price ($/1M tokens) | OpenRouter |
| `price_out` | Output price ($/1M tokens) | OpenRouter |
| `benchmark_quality` | BetterBench-informed quality tier: high/medium/low | Computed |
| `registry_name` | Matching name in model-registry.json (empty if none) | Computed |
| `sources` | Comma-separated list of data sources | Computed |

### Usage from Other Skills

Other skills can read the local CSV for model selection decisions:

```python
import csv
from pathlib import Path

rankings_path = Path.home() / ".claude/skills/llm/benchmarks/rankings.csv"
with open(rankings_path) as f:
    for row in csv.DictReader(f):
        if row["registry_name"] == "opus":
            print(f"Opus GPQA: {row['gpqa']}%, Arena Elo: {row['arena_elo']}")
```

Or via CLI for quick lookups:
```bash
# Get opus benchmarks as JSON
python3 ~/.claude/skills/llm/scripts/fetch_benchmarks.py --model opus --json
```

## Error Handling

| Error | Cause | Fix |
|---|---|---|
| "CLI not found on PATH" | CLI tool not installed | Install it or use `--route openrouter` |
| "API key not found" | Missing credential | Set env var or add to provider-keys.env |
| "Unknown model" | Model not in registry | Run `discover_models.py --apply` or check spelling |
| "No route" | Model has no configured route | Add route in model-registry.json |
| "API error 429" | Rate limited | Wait and retry, or switch provider |
| "Timed out" | Slow response | Increase `--timeout` (default 120s, CLI default 300s) |

## Usage from Claude Code (Programmatic)

When invoked as `/llm` from Claude Code, the skill works as a reference for how to call models. Claude Code should:

1. Read this SKILL.md for routing decisions
2. Call `llm_route.py` via Bash tool for external model calls
3. Use `--json` flag when parsing the response programmatically
4. Use `--route` to force a specific provider when needed

Example from Claude Code:
```bash
# Get a response from GPT-5.3 and parse it
response=$(python3 ~/.claude/skills/llm/scripts/llm_route.py --model gpt-5.3-codex --prompt "Your prompt" --json)
```

## Setup

1. **Install CLIs** (for zero-cost routing):
   ```bash
   npm install -g @anthropic-ai/claude-code   # claude login
   npm install -g @openai/codex               # codex login
   npm install -g kimi-cli                     # kimi login
   ```

2. **Set API keys** (for Google/OpenRouter):
   ```bash
   export GOOGLE_API_KEY="your-key"
   export OPENROUTER_API_KEY="your-key"
   ```

3. **Verify**:
   ```bash
   python3 ~/.claude/skills/llm/scripts/llm_route.py --list-models
   python3 ~/.claude/skills/llm/scripts/llm_route.py --model sonnet --prompt "Hello"
   ```

See `references/provider-setup.md` for detailed per-provider instructions.
