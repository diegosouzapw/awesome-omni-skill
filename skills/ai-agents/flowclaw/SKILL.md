---
name: flowclaw
description: "Unkillable Agent Infrastructure. Unified usage monitoring and intelligent load balancing for Anthropic, Google, OpenAI, and Ollama. Never let your agent get stuck."
metadata:
  openclaw:
    emoji: "🦞"
    os:
      - darwin
      - linux
    requires:
      bins:
        - curl
        - python3
---

# FlowClaw — LLM Usage Monitor & Load Balancer

> *Track usage. Balance routing. Never waste a credit.*

Unified dashboard + auto-routing for all your LLM subscriptions. Uses **Earliest Deadline First** scheduling — accounts resetting soonest are prioritized so unused credits aren't wasted.

## Supported Providers

| Provider | Auth | Data Source |
|----------|------|------------|
| **Anthropic Claude Max** | OAuth (unlimited accounts) | `api.anthropic.com/api/oauth/usage` |
| **Google Gemini CLI** | OAuth via OpenClaw | `cloudcode-pa.googleapis.com` |
| **Google Antigravity** | codexbar | codexbar usage API |
| **OpenAI Codex** | OAuth via OpenClaw | `chatgpt.com/backend-api/wham/usage` |
| **GitHub Copilot** | OAuth via OpenClaw | `api.github.com/copilot_internal/user` |
| **Ollama** | Local (auto-detected) | `localhost:11434/api/tags` |

## Commands

```bash
# 📊 Usage Monitoring
flowclaw status [--fresh] [--json]     # Full provider dashboard
flowclaw monitor [--json] [--cached]   # Clean usage report (no scoring)

# 🧠 Load Balancing
flowclaw score [--json]                # Scored ranking of all accounts
flowclaw optimize [--dry-run]          # Reorder OpenClaw model priority
flowclaw auto                          # Optimize silently (for cron jobs)

# 🛠 Utilities
flowclaw test                          # Run scoring engine unit tests
flowclaw history [N]                   # Routing decision history
```

## Setup

### Anthropic (Claude Max) — unlimited accounts
```bash
claude login
bash {baseDir}/scripts/save-account.sh
# Repeat for each account
```

### Google Gemini CLI
```bash
openclaw models auth login --provider google-gemini-cli
```

### Google Antigravity
```bash
openclaw models auth login --provider google-antigravity
brew install --cask steipete/tap/codexbar   # Required for usage metrics
```

### OpenAI Codex
```bash
openclaw models auth login --provider openai-codex
```

### GitHub Copilot
```bash
openclaw models auth login-github-copilot
```

### Ollama
```bash
brew install ollama && ollama pull qwen3:235b
# Auto-detected — no config needed
```

### Cron Automation
```bash
# Optimize routing every 30 minutes
*/30 * * * * bash ~/clawd/skills/flowclaw/scripts/flowclaw.sh auto
```

## Scoring Algorithm

```
score = urgency(0.30) + availability(0.25) + proximity(0.15) + weekly_headroom(0.20) + tier(0.10)
```

| Factor | Formula | Measures |
|--------|---------|----------|
| Urgency | `remaining / hours_to_reset` | Credits wasting per hour |
| Availability | `√(remaining)` | Dampened capacity |
| Proximity | `1 - (hours / window)` | Deadline pressure |
| Weekly headroom | `(100 - weekly%) / 100` | 7-day capacity reserve |
| Tier bonus | Free=+0.8, Sub=0, Local=-0.3 | Cost preference |

**Perishable inventory:** Both 5h session and 7d weekly windows expire. When ≤6h to weekly reset, penalty is removed — burn remaining credits. **Family-aware:** Only swaps within same capability class (Opus↔Opus, not Opus↔Gemini).
