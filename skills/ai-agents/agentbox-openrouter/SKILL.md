---
name: agentbox-openrouter
description: "Set up OpenRouter as your LLM provider. Guides through account creation, API key setup, config, and making it the default model. Use when a user wants to use OpenRouter models like Claude Sonnet 4.5."
metadata: {"openclaw": {"always": true}}
user-invocable: true
---

# OpenRouter Setup for AgentBox

This skill guides users through configuring OpenRouter as their LLM provider on this AgentBox instance. OpenRouter gives access to models from Anthropic, OpenAI, Google, Meta, and others through a single API key.

## When to use this skill

Invoke this when the user:
- Wants to use a model through OpenRouter (e.g., "I want to use Claude Sonnet 4.5")
- Asks about configuring a different LLM provider
- Mentions OpenRouter
- Wants access to models not available through the default blockrun provider

## Setup flow

### Step 1: Check for OpenRouter account

Ask the user if they have an OpenRouter account. If not, guide them:

> To use OpenRouter, you'll need an account and API key:
> 1. Go to https://openrouter.ai and sign up (Google/GitHub login works)
> 2. Go to https://openrouter.ai/keys
> 3. Click "Create Key"
> 4. Copy the key (starts with `sk-or-`)
>
> Let me know when you have your API key.

### Step 2: Get the API key

Ask the user to provide their API key. It should start with `sk-or-`.

### Step 3: Ask which model they want

If the user already specified a model, use that. Otherwise, recommend:

> **Recommended: Claude Sonnet 4.5** (`openrouter/anthropic/claude-sonnet-4-5`) - best balance of capability and cost for most tasks.
>
> Other popular options:
> - `openrouter/anthropic/claude-opus-4-6` - most capable, higher cost
> - `openrouter/openai/gpt-4o` - OpenAI's flagship
> - `openrouter/google/gemini-2.5-pro` - Google's best
>
> Which model would you like as your default?

### Step 4: Configure OpenClaw

Read the current config, modify it, and write it back:

```bash
# Read current config
cat ~/.openclaw/openclaw.json
```

Use `jq` to update the config. The two fields to set:

1. **`env.OPENROUTER_API_KEY`** - the API key
2. **`agents.defaults.model.primary`** - the default model

```bash
jq --arg key "sk-or-USER_KEY_HERE" \
   --arg model "openrouter/anthropic/claude-sonnet-4-5" \
   '.env.OPENROUTER_API_KEY = $key | .agents.defaults.model.primary = $model' \
   ~/.openclaw/openclaw.json > /tmp/openclaw-update.json \
   && mv /tmp/openclaw-update.json ~/.openclaw/openclaw.json
```

**IMPORTANT**: Always read the full config first, then modify. Never write a partial config file.

### Step 5: Restart the gateway

```bash
openclaw gateway restart
```

Wait a few seconds, then verify:

```bash
openclaw status
```

### Step 6: Confirm

Tell the user the setup is complete and their default model is now set to the chosen OpenRouter model. Suggest they send a test message to verify everything works.

## Model reference format

OpenRouter models use the format `openrouter/<provider>/<model>`:
- `openrouter/anthropic/claude-sonnet-4-5`
- `openrouter/anthropic/claude-opus-4-6`
- `openrouter/anthropic/claude-haiku-3-5`
- `openrouter/openai/gpt-4o`
- `openrouter/openai/o1`
- `openrouter/google/gemini-2.5-pro`
- `openrouter/meta-llama/llama-3.3-70b-instruct`

Full model list at https://openrouter.ai/models

## Switching models later

To change the default model without re-entering the API key:

```bash
jq --arg model "openrouter/anthropic/claude-opus-4-6" \
   '.agents.defaults.model.primary = $model' \
   ~/.openclaw/openclaw.json > /tmp/openclaw-update.json \
   && mv /tmp/openclaw-update.json ~/.openclaw/openclaw.json
openclaw gateway restart
```

## Troubleshooting

- **"Invalid API key"**: Verify the key starts with `sk-or-` and has credit on https://openrouter.ai/credits
- **Model not responding**: Check if the model is available on https://openrouter.ai/models - some models have downtime
- **Config broken after edit**: The issue is usually malformed JSON. Read the file with `cat ~/.openclaw/openclaw.json | jq .` to check syntax
- **Changes not taking effect**: Must run `openclaw gateway restart` after any config change
