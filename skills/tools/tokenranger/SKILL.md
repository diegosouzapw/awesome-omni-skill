---
name: tokenranger
version: 1.0.0
description: Install, configure, and operate the TokenRanger OpenClaw plugin. Use when you want to reduce cloud LLM token costs by 50-80% via local Ollama context compression, or when diagnosing TokenRanger sidecar issues.
metadata:
  {
    "openclaw":
      {
        "emoji": "🗜️",
        "category": "performance",
        "requires": { "bins": ["openclaw"] },
        "links": {
          "plugin": "https://github.com/peterjohannmedina/openclaw-plugin-tokenranger",
          "npm": "https://www.npmjs.com/package/openclaw-plugin-tokenranger"
        }
      }
  }
---

# TokenRanger

**TokenRanger** compresses session context through a local Ollama SLM before sending to cloud LLMs — reducing input token costs by **50–80%** per turn with graceful fallthrough if anything goes wrong.

- **Plugin repo:** https://github.com/peterjohannmedina/openclaw-plugin-tokenranger
- **npm:** `openclaw-plugin-tokenranger`
- **Maintained by:** [@peterjohannmedina](https://github.com/peterjohannmedina)

---

## When to Load This Skill

- User asks to install, configure, or troubleshoot TokenRanger
- User wants to reduce token costs or enable context compression
- User runs `/tokenranger` commands and needs help interpreting output
- User wants to switch compression strategy (GPU/CPU/off)
- User asks about upgrading or uninstalling TokenRanger

---

## How It Works

```
User message → OpenClaw gateway
  → before_agent_start hook
  → Turn 1: skip (full fidelity)
  → Turn 2+: send history to localhost:8100/compress
  → FastAPI sidecar runs LangChain LCEL chain via Ollama
  → Compressed summary prepended to context
  → Cloud LLM receives compressed context instead of full history
```

Inference strategy is auto-selected by GPU availability:

| Strategy | Trigger | Model | Approach |
|---|---|---|---|
| `full` | GPU available | `mistral:7b` | Deep semantic summarization |
| `light` | CPU only | `phi3.5:3b` | Extractive bullet points |
| `passthrough` | Ollama unreachable | — | Truncate to last 20 lines |

---

## Install

### Step 1 — Install the plugin

```bash
openclaw plugins install openclaw-plugin-tokenranger
```

To pin an exact version:

```bash
openclaw plugins install openclaw-plugin-tokenranger@1.0.0 --pin
```

### Step 2 — First-time setup

```bash
openclaw tokenranger setup
```

This pulls Ollama models, creates the Python venv, installs FastAPI/LangChain deps,
and registers the sidecar as a system service (systemd on Linux, launchd on macOS).

### Step 3 — Restart gateway

```bash
openclaw gateway restart
```

### Step 4 — Verify

```bash
openclaw tokenranger
```

Should show current settings and sidecar status (reachable / unreachable).

---

## Configuration

Set config values with:

```bash
openclaw config set plugins.entries.tokenranger.config.<key> <value>
openclaw gateway restart
```

| Key | Default | Description |
|---|---|---|
| `serviceUrl` | `http://127.0.0.1:8100` | TokenRanger sidecar URL |
| `timeoutMs` | `10000` | Max wait before fallthrough |
| `minPromptLength` | `500` | Min chars before compressing |
| `ollamaUrl` | `http://127.0.0.1:11434` | Ollama API URL |
| `preferredModel` | `mistral:7b` | Model for GPU strategy |
| `compressionStrategy` | `auto` | `auto` / `full` / `light` / `passthrough` |
| `inferenceMode` | `auto` | `auto` / `cpu` / `gpu` / `remote` |

**Force CPU-only mode:**

```bash
openclaw config set plugins.entries.tokenranger.config.compressionStrategy light
openclaw config set plugins.entries.tokenranger.config.inferenceMode cpu
openclaw gateway restart
```

---

## Commands

| Command | Description |
|---|---|
| `/tokenranger` | Show current settings and sidecar health |
| `/tokenranger mode gpu` | Force GPU (full) compression |
| `/tokenranger mode cpu` | Force CPU (light) compression |
| `/tokenranger mode off` | Disable compression (passthrough) |
| `/tokenranger model` | List available Ollama models |
| `/tokenranger toggle` | Enable / disable the plugin |

---

## Upgrading

```bash
# Check for updates (dry run)
openclaw plugins update tokenranger --dry-run

# Apply update
openclaw plugins update tokenranger
openclaw tokenranger setup   # re-runs setup if sidecar deps changed
openclaw gateway restart
```

To pin a specific version:

```bash
openclaw plugins install openclaw-plugin-tokenranger@2026.3.1 --pin
openclaw tokenranger setup
openclaw gateway restart
```

List all published versions:

```bash
npm view openclaw-plugin-tokenranger versions --json
```

---

## Uninstalling

```bash
openclaw plugins uninstall tokenranger
openclaw gateway restart
```

Remove the sidecar service manually:

```bash
# Linux
systemctl --user stop tokenranger && systemctl --user disable tokenranger
rm ~/.config/systemd/user/tokenranger.service

# macOS
launchctl unload ~/Library/LaunchAgents/com.peterjohannmedina.tokenranger.plist
rm ~/Library/LaunchAgents/com.peterjohannmedina.tokenranger.plist
```

---

## Troubleshooting

**Sidecar unreachable after setup:**

```bash
# Linux
systemctl --user status tokenranger
journalctl --user -u tokenranger -n 50

# macOS
launchctl list | grep tokenranger
cat ~/Library/Logs/tokenranger.log

# Manual start (any platform)
~/.openclaw/extensions/tokenranger/service/start.sh
```

**Ollama not found:**

```bash
curl http://127.0.0.1:11434/api/tags
# If not running:
ollama serve
```

**Compression not reducing tokens:**
- Check `minPromptLength` — default 500 chars; short conversations are skipped by design
- Run `/tokenranger` to confirm strategy is not `passthrough`
- Check sidecar logs for errors

**Graceful degradation:** TokenRanger never blocks a message. Any failure → silent fallthrough to uncompressed cloud LLM call.

---

## Performance Reference

5-turn Discord benchmark (GPU, `mistral:7b-instruct`):

| Turn | Input tokens | Compressed | Reduction |
|---|---|---|---|
| 2 | 732 | 125 | 82.9% |
| 3 | 1,180 | 150 | 87.3% |
| 4 | 1,685 | 212 | 87.4% |
| 5 | 2,028 | 277 | 86.3% |

**Cumulative: 5,866 → 885 tokens (84.9% reduction)**
