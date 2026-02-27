---
name: clawapi
description: Switch AI models and manage API keys for OpenClaw with a native macOS app. Supports 16 providers including OpenAI, Anthropic, xAI, Google, Groq, Ollama, LM Studio, and more.
homepage: https://github.com/Gogo6969/clawapi
user-invocable: true
metadata: {"openclaw":{"emoji":"🔑","requires":{"bins":["curl"],"config":["skills.entries.clawapi"]},"install":[{"kind":"script","command":"curl -fsSL https://raw.githubusercontent.com/Gogo6969/clawapi/main/install.sh | bash"}]}}
---

# ClawAPI — Model Switcher & Key Vault for OpenClaw

ClawAPI is a native macOS app that lets you switch AI models and manage API keys for OpenClaw.

## What It Does

- **One-click model switching** — Pick any model from any provider and apply it instantly
- **Touch ID** — Biometric authentication for adding and deleting API keys
- **16 providers** — OpenAI, Anthropic, xAI, Google, Groq, Mistral, OpenRouter, Cerebras, Kimi, MiniMax, Z.AI, OpenCode Zen, Vercel AI, HuggingFace, Ollama, LM Studio
- **Config safety** — JSON validation before writing, automatic `.bak` backups

## Installation

### Option 1: Download from GitHub Releases (recommended)

Download the latest signed and notarized `.zip` from [GitHub Releases](https://github.com/Gogo6969/clawapi/releases), unzip, and move `ClawAPI.app` to `/Applications`.

### Option 2: Install script

```bash
curl -fsSL https://raw.githubusercontent.com/Gogo6969/clawapi/main/install.sh | bash
```

The install script downloads the same signed `.zip` from GitHub Releases, verifies the SHA-256 checksum, unzips it, and moves the app to `/Applications`. You can [review the script source](https://github.com/Gogo6969/clawapi/blob/main/install.sh) before running it.

Requires macOS 14+. The app is signed with an Apple Developer ID and notarized by Apple.

## How It Works

1. **Add a provider** — Click `+`, pick a provider, paste your API key
2. **Pick a model** — Use the dropdown to choose a sub-model (GPT-4.1, Claude Sonnet 4, Grok 4, etc.)
3. **Done** — ClawAPI syncs everything to OpenClaw automatically

## Where API Keys Are Stored

API keys are managed in **two places** by design:

1. **macOS Keychain (master copy)** — The key you enter is stored in the macOS Keychain, protected by hardware encryption and Touch ID. This is the authoritative copy.
2. **`auth-profiles.json` (sync copy for OpenClaw)** — OpenClaw reads API keys from its own `auth-profiles.json` config file. ClawAPI writes a copy of the key there so OpenClaw can use it. This file lives in `~/Library/Application Support/OpenClaw/`.

The active model selection is written to `openclaw.json`. No proxy, no middleware — OpenClaw talks directly to provider APIs.

## Security & Privacy

- API keys are stored in the **macOS Keychain** with hardware encryption; a sync copy is written to OpenClaw's `auth-profiles.json` so that OpenClaw can read them
- **Touch ID** authentication for adding/deleting keys (password fallback on Macs without Touch ID)
- The app is **signed with Apple Developer ID** and **notarized by Apple**
- Hardened runtime enabled
- **No data leaves your machine** — ClawAPI only reads/writes local OpenClaw config files
- No telemetry, no analytics, no phone-home

## External Endpoints

| Endpoint | Purpose | Data Sent |
|----------|---------|-----------|
| `raw.githubusercontent.com` | Check for app updates | None (reads `update.json`) |
| `localhost:11434` | Discover Ollama models | None (reads local API) |
| `localhost:1234` | Discover LM Studio models | None (reads local API) |

No other network requests are made by ClawAPI.

## Links

- **GitHub:** [github.com/Gogo6969/clawapi](https://github.com/Gogo6969/clawapi)
- **Wiki:** [github.com/Gogo6969/clawapi/wiki](https://github.com/Gogo6969/clawapi/wiki)
- **User Guide:** [docs/USER_GUIDE.md](https://github.com/Gogo6969/clawapi/blob/main/docs/USER_GUIDE.md)
