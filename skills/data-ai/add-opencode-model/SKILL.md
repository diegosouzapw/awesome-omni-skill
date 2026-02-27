---
name: add-opencode-model
description: Fetch OpenCode Zen model details and provide guidance for adding models to acai-ts provider configuration.
---

# Add OpenCode Zen Model

Fetches model details from both OpenCode and OpenRouter APIs to provide structured information for the agent to add the model to `source/models/opencode-zen-provider.ts`.

## Usage

Run this script from this file's directory.

```bash
./add-model.js google/gemini-3-flash-preview
```

## How It Works

1. **Confirms model exists** in OpenCode API (`https://opencode.ai/zen/v1/models`)
2. **Searches OpenRouter API** (`https://openrouter.ai/api/v1/models`) for matching model to get:
   - Context window length
   - Max output tokens
   - Pricing information
   - Supported features (reasoning/tool calling)
3. **Determines client type** based on model provider:
   - `openai` → uses `@ai-sdk/openai` (Responses API)
   - `google` → uses `@ai-sdk/google` (custom base URL)
   - `anthropic` → uses `@ai-sdk/anthropic`
   - `openai-compatible` → uses `createOpenAICompatible`
4. **Outputs JSON** with all details needed to add the model

## JSON Output Format

The script outputs a JSON object wrapped in markers:

```
--- MODEL_DETAILS_JSON_START ---
{...}
--- MODEL_DETAILS_JSON_END ---
```

### JSON Structure

```json
{
  "modelKey": "gemini-3-flash-preview",
  "opencodeId": "google/gemini-3-flash-preview",
  "openrouterId": "google/gemini-3-flash-preview",
  "providerFile": "./source/models/opencode-zen-provider.ts",
  "registryId": "opencode:gemini-3-flash-preview",
  "clientType": "google",
  "registryEntry": {
    "id": "opencode:gemini-3-flash-preview",
    "provider": "OpenCode",
    "contextWindow": 1048576,
    "maxOutputTokens": 65535,
    "defaultTemperature": 0.5,
    "promptFormat": "markdown",
    "supportsReasoning": true,
    "supportsToolCalling": true,
    "costPerInputToken": 0.0000005,
    "costPerOutputToken": 0.000003
  },
  "clientConfig": {
    "key": "gemini-3-flash-preview",
    "opencodeId": "google/gemini-3-flash-preview",
    "clientType": "google",
    "openrouterId": "google/gemini-3-flash-preview"
  },
  "modelInfo": {
    "opencodeId": "google/gemini-3-flash-preview",
    "opencodeName": "Gemini 3 Flash Preview",
    "openrouterId": "google/gemini-3-flash-preview",
    "openrouterName": "Google: Gemini 3 Flash Preview",
    "contextLength": 1048576,
    "maxCompletionTokens": 65535,
    "pricing": {...},
    "supportedParameters": [...]
  }
}
```

## Client Type Reference

| Client Type | Library | Base URL | Example Models |
|-------------|---------|----------|----------------|
| `openai` | `@ai-sdk/openai` | Responses API | `openai/gpt-4o` |
| `google` | `@ai-sdk/google` | `https://opencode.ai/zen/v1/models/<model-id>` | `google/gemini-3-flash-preview` |
| `anthropic` | `@ai-sdk/anthropic` | Messages API | `anthropic/claude-3-5-sonnet` |
| `openai-compatible` | `@ai-sdk/openai-compatible` | `https://opencode.ai/zen/v1` | All other models |

## How to Add Model to Provider File

### Step 1: Add client if needed

In `source/models/opencode-zen-provider.ts`, check if you need to add a new client based on `clientType`:

```typescript
// For OpenAI models (Responses API)
import { createOpenAI } from "@ai-sdk/openai";
const openaiClient = createOpenAI({
  apiKey: process.env["OPENCODE_ZEN_API_TOKEN"] ?? "",
  baseURL: "https://opencode.ai/zen/v1",
});

// For Google models
import { createGoogle } from "@ai-sdk/google";
const googleClient = createGoogle({
  apiKey: process.env["OPENCODE_ZEN_API_TOKEN"] ?? "",
  baseURL: "https://opencode.ai/zen/v1/models/<model-id>",
});
```

### Step 2: Add to `opencodeZenModels` object

Find the `opencodeZenModels` const object and add the new model entry **in alphabetical order by key**, using the appropriate client:

```typescript
const opencodeZenModels = {
  "glm-4-7": completionsClient("glm-4.7-free"),
  "minimax-m2-1": messagesClient("minimax-m2.1-free"),
  // For OpenAI-compatible models:
  "new-model": completionsClient("provider/new-model"),
  // For Anthropic models:
  "claude-model": messagesClient("anthropic/claude-model"),
  // For Google models:
  "gemini-flash": googleClient("google/gemini-3-flash-preview"),
} as const;
```

### Step 3: Add to `opencodeZenModelRegistry` object

Find the `opencodeZenModelRegistry` object and add the registry entry **in alphabetical order by `registryId`**:

```typescript
export const opencodeZenModelRegistry: {
  [K in ModelName]: ModelMetadata<ModelName>;
} = {
  "opencode:glm-4-7": {
    id: "opencode:glm-4-7",
    provider: "opencode",
    // ... existing entry ...
  },
  // ... existing entries ...
  "opencode:new-model": {
    id: "opencode:new-model",
    provider: "opencode",
    contextWindow: 1048576,
    maxOutputTokens: 65535,
    defaultTemperature: 0.5,
    promptFormat: "markdown",
    supportsReasoning: true,
    supportsToolCalling: true,
    costPerInputToken: 0.0000005,
    costPerOutputToken: 0.000003,
  },
  // ...
};
```

### Step 4: Validate changes

Run the project's validation commands:

```bash
npm run typecheck
npm run lint
npm run format
```

## Field Mapping Reference

| JSON Field | Registry Field | Notes |
|------------|----------------|-------|
| `registryEntry.id` | `id` | Full registry ID |
| `registryEntry.provider` | `provider` | Always "opencode" |
| `registryEntry.contextWindow` | `contextWindow` | From OpenRouter API |
| `registryEntry.maxOutputTokens` | `maxOutputTokens` | From OpenRouter API |
| `registryEntry.defaultTemperature` | `defaultTemperature` | Inferred from model ID |
| `registryEntry.promptFormat` | `promptFormat` | Inferred from model family |
| `registryEntry.supportsReasoning` | `supportsReasoning` | From OpenRouter API |
| `registryEntry.supportsToolCalling` | `supportsToolCalling` | From OpenRouter API |
| `registryEntry.costPerInputToken` | `costPerInputToken` | From OpenRouter API |
| `registryEntry.costPerOutputToken` | `costPerOutputToken` | From OpenRouter API |

## Matching Strategy

The script uses multiple strategies to find a matching OpenRouter model:

1. **Direct key match**: Match by model key (e.g., `glm-4.7` matches `glm-4.7`)
2. **Provider + name match**: Match by provider prefix and model name (e.g., `google/gemini-3-flash` matches `google/gemini-3-flash-preview`)
3. **Partial name match**: Match by partial model name (fallback)

If no match is found, defaults are used for pricing and capability fields.

## Default Value Inferences

| Field | Logic |
|-------|-------|
| `defaultTemperature` | Returns `-1` if model ID contains "codex", "coder", or "code"; otherwise `0.5` |
| `promptFormat` | Maps model family to format: gemini/claude/qwen/mistral/moonshotai → "markdown", gpt/openai → "xml", deepseek → "bracket", defaults to "markdown" |
| `maxOutputTokens` | Falls back to `context_length` if not available from OpenRouter |
| `costPerInputToken` | Defaults to `0` if no OpenRouter match |
| `costPerOutputToken` | Defaults to `0` if no OpenRouter match |

## Error Handling

- Validates model ID exists in OpenCode API
- Shows first 10 available OpenCode models if ID not found
- Warns if no matching OpenRouter model found
- Provides clear error messages with suggestions

## Examples

```bash
# Add Google Gemini Flash
./add-model.js google/gemini-3-flash-preview

# Add OpenAI GPT model
./add-model.js openai/gpt-4o

# Add Anthropic Claude model
./add-model.js anthropic/claude-3-5-sonnet

# Add other provider model
./add-model.js deepseek/deepseek-v3
```
