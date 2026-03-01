---
name: models-dev
description: Query AI model specifications, pricing, and capabilities from models.dev database. Use when users ask about AI model parameters (context window, token limits, cost per token), model comparisons, provider information, or need to look up specific model IDs for AI SDK integration. Triggers on queries like "What's the context window for GPT-4o?", "Compare Claude vs GPT", "How much does Gemini Pro cost?", "List OpenAI models", or "What models support tool calling?".
---

# Models.dev - AI Model Information Query

Query comprehensive AI model specifications from the models.dev open-source database.

## Usage

Run the script at `scripts/fetch_models.py`:

```bash
# List all providers
python scripts/fetch_models.py --providers

# Search models by name or ID
python scripts/fetch_models.py --search <query>

# Get specific model info
python scripts/fetch_models.py <model_id>

# List models from a provider
python scripts/fetch_models.py --provider <provider_name>

# Compare two models
python scripts/fetch_models.py --compare <model_id_1> <model_id_2>
```

## Examples

```bash
# Search for Claude models
python scripts/fetch_models.py --search claude

# Get GPT-4o details
python scripts/fetch_models.py gpt-4o

# List Anthropic models
python scripts/fetch_models.py --provider anthropic

# Compare Claude Sonnet 3.7 vs GPT-4o
python scripts/fetch_models.py --compare claude-3-7-sonnet-20250219 gpt-4o
```

## Available Data

Each model includes (when available):
- **Model ID**: Identifier for AI SDK integration
- **Provider**: Company/platform offering the model
- **SDK Package**: npm package for AI SDK
- **Capabilities**: Reasoning, Tool Calling, Structured Output, Attachments, Temperature
- **Modalities**: Input/output types (text, image, audio, video, pdf)
- **Limits**: Context window, max input/output tokens
- **Cost**: Per-million token pricing (input, output, cache, reasoning)
- **Dates**: Knowledge cutoff, release date, last updated
- **Status**: alpha, beta, deprecated (if applicable)
- **Open Weights**: Whether model weights are publicly available

## API Source

Data fetched from `https://models.dev/api.json` - a community-maintained open-source database.
