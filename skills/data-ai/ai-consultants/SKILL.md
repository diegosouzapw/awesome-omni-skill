---
name: ai-consultants
description: Consult Gemini CLI, Codex CLI, Mistral Vibe, Kilo CLI, Cursor, Claude, Amp, Kimi, Qwen, and Ollama as external experts for coding questions. Automatically excludes the invoking agent from the panel to avoid self-consultation. Use when you have doubts about implementations, want a second opinion, need to choose between different approaches, or when explicitly requested with phrases like "ask the consultants", "what do the other models think", "compare solutions".
---

# AI Consultants v2.9.1 - AI Expert Panel

Simultaneously consult multiple AIs as "consultants" for coding questions. Each consultant has a **configurable persona** that influences their response style.

## Quick Start

```
/ai-consultants:config-wizard       # Initial setup
/ai-consultants:consult "Your question here"
```

## What's New in v2.9

- **Kimi CLI Consultant**: New "The Eastern Sage" persona for holistic understanding (v2.9)
- **Amp CLI Consultant**: "The Systems Thinker" persona for system design (v2.8)
- **Qwen CLI Support**: CLI/API mode switching for Qwen3 (v2.7)
- **CLI/API Mode Switching**: Gemini, Codex, Claude, Mistral, Qwen3 can use CLI or API (v2.6)
- **Model Quality Tiers**: premium, standard, economy with `apply_model_tier()` (v2.5)
- **Budget Enforcement**: Configurable cost limits with `ENABLE_BUDGET_LIMIT` (v2.4)
- **Premium Model Defaults**: All consultants now use flagship models by default
- **14 Consultants**: Gemini, Codex, Mistral, Kilo, Cursor, Aider, Amp, Kimi, Claude, Qwen3, GLM, Grok, DeepSeek, Ollama

## Slash Commands

### Consultation Commands

| Command | Description |
|---------|-------------|
| `/ai-consultants:consult` | Main consultation - ask AI consultants a coding question |
| `/ai-consultants:ask-experts` | Quick query alias for consult |
| `/ai-consultants:debate` | Run consultation with multi-round debate |
| `/ai-consultants:help` | Show all commands and usage |

### Configuration Commands

| Command | Description |
|---------|-------------|
| `/ai-consultants:config-wizard` | Full interactive setup (CLI detection, API keys, personas) |
| `/ai-consultants:config-check` | Verify CLI agents are installed and authenticated |
| `/ai-consultants:config-status` | View current configuration |
| `/ai-consultants:config-preset` | Set default preset (minimal, balanced, high-stakes, local) |
| `/ai-consultants:config-strategy` | Set default synthesis strategy |
| `/ai-consultants:config-features` | Toggle features (Debate, Synthesis, Peer Review, etc.) |
| `/ai-consultants:config-personas` | Change consultant personas |
| `/ai-consultants:config-api` | Configure API-based consultants (Qwen3, GLM, Grok, DeepSeek) |

## Configuration Workflow

Set your preferences using slash commands:

```
/ai-consultants:config-preset       # Choose default preset
/ai-consultants:config-strategy     # Choose synthesis strategy
/ai-consultants:config-features     # Enable/disable features
/ai-consultants:config-status       # View current settings
```

## Consultants and Personas

| Consultant | CLI | Persona | Focus |
|------------|-----|---------|-------|
| **Google Gemini** | `gemini` | The Architect | Design patterns, scalability |
| **OpenAI Codex** | `codex` | The Pragmatist | Simplicity, proven solutions |
| **Mistral Vibe** | `vibe` | The Devil's Advocate | Edge cases, vulnerabilities |
| **Kilo Code** | `kilocode` | The Innovator | Creativity, unconventional |
| **Cursor** | `agent` | The Integrator | Full-stack perspective |
| **Aider** | `aider` | The Pair Programmer | Collaborative coding |
| **Amp** | `amp` | The Systems Thinker | System design, interactions |
| **Kimi** | `kimi` | The Eastern Sage | Holistic, balanced perspectives |
| **Claude** | `claude` | The Synthesizer | Big picture, synthesis |
| **Qwen** | `qwen` | The Analyst | Data-driven, metrics |
| **Ollama** | `ollama` | The Local Expert | Privacy-first, zero cost |

**API-only consultants**: GLM (The Methodologist), Grok (The Provocateur), DeepSeek (The Code Specialist)

**CLI/API Mode**: Gemini, Codex, Claude, Mistral, and Qwen can switch between CLI and API mode via `*_USE_API` environment variables.

**Self-Exclusion**: The invoking agent is automatically excluded from the panel. When invoked from Claude Code, Claude is excluded; when invoked from Codex CLI, Codex is excluded, etc.

## Requirements

- **At least 2 consultant CLIs** installed and authenticated
- **jq** for JSON processing

### Quick Install

```bash
curl -fsSL https://raw.githubusercontent.com/matteoscurati/ai-consultants/main/scripts/install.sh | bash
~/.claude/skills/ai-consultants/scripts/doctor.sh --fix
```

### CLI Installation

```bash
npm install -g @google/gemini-cli      # Gemini
npm install -g @openai/codex           # Codex
pip install mistral-vibe               # Mistral
npm install -g @kilocode/cli           # Kilo
npm install -g @qwen-code/qwen-code@latest  # Qwen
curl -fsSL https://ampcode.com/install.sh | bash  # Amp
pip install kimi-cli && kimi login     # Kimi
brew install jq                        # Required

# For local inference (optional)
curl -fsSL https://ollama.com/install.sh | sh
ollama pull llama3.2
```

## Configuration Presets

| Preset | Consultants | Use Case |
|--------|-------------|----------|
| `minimal` | 2 (Gemini + Codex) | Quick questions |
| `balanced` | 4 (+Mistral +Kilo) | Standard use |
| `thorough` | 5 (+Cursor) | Comprehensive |
| `high-stakes` | All + debate | Critical decisions |
| `local` | Ollama only | Full privacy |
| `security` | Security-focused | +Debate |
| `cost-capped` | Budget-friendly | Low cost |

## Synthesis Strategies

| Strategy | Description |
|----------|-------------|
| `majority` | Most common answer wins (default) |
| `risk_averse` | Weight conservative responses |
| `security_first` | Prioritize security |
| `cost_capped` | Prefer cheaper solutions |
| `compare_only` | No recommendation |

## Usage Examples

### Basic Consultation

```
/ai-consultants:consult "How to optimize this SQL query?"
```

### With File Context

```
/ai-consultants:consult "Review this authentication flow" src/auth.ts
```

### With Debate

```
/ai-consultants:debate "Microservices or monolith for our new service?"
```

### Bash Usage

```bash
cd ~/.claude/skills/ai-consultants

# With preset
./scripts/consult_all.sh --preset balanced "Best approach for caching?"

# With strategy
./scripts/consult_all.sh --strategy risk_averse "Security question"

# With local model
./scripts/consult_all.sh --preset local "Private question"
```

## Workflow

```
Query -> Classify -> Parallel Queries -> Voting -> Synthesis -> Report
                          |                |           |
                     Gemini (8)      Consensus    Recommendation
                     Codex (7)       Analysis     Comparison
                     Mistral (6)                  Risk Assessment
```

With debate:
```
Round 1 -> Cross-Critique -> Round 2 -> Final Synthesis
```

## Usage Triggers

### Automatic

- Doubts about implementation approach
- Validating complex solutions
- Exploring architectural alternatives

### Explicit

- "Ask the consultants..."
- "What do the other models think?"
- "Compare solutions"
- "I want a second opinion"

## Features

| Feature | Description | Toggle |
|---------|-------------|--------|
| **Personas** | Each consultant has a role that shapes responses | `ENABLE_PERSONA` |
| **Synthesis** | Auto-combine responses into recommendation | `ENABLE_SYNTHESIS` |
| **Debate** | Consultants critique each other's answers | `ENABLE_DEBATE` |
| **Peer Review** | Consultants anonymously rank each other | `ENABLE_PEER_REVIEW` |
| **Smart Routing** | Auto-select best consultants per question type | `ENABLE_SMART_ROUTING` |
| **Cost Tracking** | Track API usage costs | `ENABLE_COST_TRACKING` |
| **Panic Mode** | Auto-add rigor when uncertainty detected | `ENABLE_PANIC_MODE` |

## Configuration

```bash
# Defaults (v2.9)
DEFAULT_PRESET=balanced      # Preset when --preset not given
DEFAULT_STRATEGY=majority    # Strategy when --strategy not given

# Core features
ENABLE_DEBATE=true           # Multi-agent debate
ENABLE_SYNTHESIS=true        # Automatic synthesis
ENABLE_PEER_REVIEW=false     # Anonymous peer review
ENABLE_PANIC_MODE=auto       # Auto-rigor for uncertainty

# CLI/API Mode Switching (v2.6+)
GEMINI_USE_API=false         # Use Google AI API instead of CLI
CODEX_USE_API=false          # Use OpenAI API instead of CLI
CLAUDE_USE_API=false         # Use Anthropic API instead of CLI
MISTRAL_USE_API=false        # Use Mistral API instead of CLI
QWEN3_USE_API=true           # Use DashScope API (default) or CLI

# New consultants (v2.7-2.9)
ENABLE_AMP=false             # Amp CLI - The Systems Thinker
AMP_MODEL=amp
ENABLE_KIMI=false            # Kimi CLI - The Eastern Sage
KIMI_MODEL=kimi-code/kimi-for-coding
ENABLE_QWEN3=false           # Qwen CLI/API - The Analyst
QWEN3_MODEL=qwen3-max

# Ollama (local models)
ENABLE_OLLAMA=true
OLLAMA_MODEL=qwen2.5-coder:32b

# Budget management (v2.4)
ENABLE_BUDGET_LIMIT=false
MAX_SESSION_COST=1.00
BUDGET_ACTION=warn           # warn or stop
```

## Output

```
/tmp/ai_consultations/TIMESTAMP/
├── gemini.json        # Individual responses
├── codex.json
├── voting.json        # Consensus
├── synthesis.json     # Recommendation
└── report.md          # Human-readable
```

## Doctor Command

Diagnose and fix issues:

```bash
./scripts/doctor.sh              # Full check
./scripts/doctor.sh --fix        # Auto-fix
./scripts/doctor.sh --json       # JSON output
```

## Interpreting Results

| Scenario | Recommendation |
|----------|----------------|
| High confidence + High consensus | Proceed confidently |
| Low confidence OR Low consensus | Consider more options |
| Mistral disagrees | Investigate risks |
| Panic mode triggered | Add debate rounds |

## Best Practices

### Security

- **Never** include credentials in queries
- Use `--preset local` for sensitive code

### Effective Queries

- Be specific about the question
- Include constraints (performance, etc.)
- Use debate for controversial decisions

## Troubleshooting

| Issue | Solution |
|-------|----------|
| "Unknown skill" | Run install script or check `~/.claude/commands/` |
| "Exit code 1" | Run `/ai-consultants:config-check` to diagnose |
| No consultants | Run `/ai-consultants:config-wizard` |
| API errors | Check `/ai-consultants:config-status` |
| CLI not found | Run `./scripts/doctor.sh --fix` |

## Extended Documentation

- [Setup Guide](docs/SETUP.md) - Installation, authentication, Claude Code setup
- [Cost Rates](docs/COST_RATES.md) - Model pricing
- [Smart Routing](docs/SMART_ROUTING.md) - Category routing
- [JSON Schema](docs/JSON_SCHEMA.md) - Output format

## Known Limitations

- Minimum 2 consultants required
- Smart Routing off by default
- Synthesis requires Claude CLI (fallback available)
- Estimated costs (heuristic token counting)
