---
name: add-openrouter-model
description: Fetch OpenRouter model details and provide guidance for adding models to acai-ts provider configuration.
---

# Add OpenRouter Model

Fetches model details from OpenRouter API and provides structured information for the agent to add the model to `source/models/openrouter-provider.ts`.

## Usage

Run this script from this file's directory.

```bash
./add-model.js google/gemini-3-flash-preview
```

## How It Works

1. **Fetches model data** from `https://openrouter.ai/api/v1/models`
2. **Locates the exact model** by the provided OpenRouter ID
3. **Extracts and computes** key information:
   - Context window length (`context_length`)
   - Max output tokens (`top_provider.max_completion_tokens`)
   - Pricing (`pricing.prompt`, `pricing.completion`)
   - Supported features (checks `supported_parameters` for reasoning/tool calling)
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
  "openrouterId": "google/gemini-3-flash-preview",
  "providerFile": "./source/models/openrouter-provider.ts",
  "registryId": "openrouter:gemini-3-flash-preview",
  "registryEntry": {
    "id": "openrouter:gemini-3-flash-preview",
    "provider": "openrouter",
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
    "openrouterId": "google/gemini-3-flash-preview"
  },
  "modelInfo": {
    "id": "google/gemini-3-flash-preview",
    "name": "Gemini 3.0 Flash Preview",
    "contextLength": 1048576,
    "maxCompletionTokens": 65535,
    "pricing": {...},
    "supportedParameters": [...]
  }
}
```

## How to Add Model to Provider File

### Step 1: Add to `openrouterModels` object

In `source/models/openrouter-provider.ts`, find the `openrouterModels` const object (starts with `const openrouterModels = {`). Add the new model entry **in alphabetical order by key**:

```typescript
const openrouterModels = {
  "claude-3-5-sonnet": openRouterClient("anthropic/claude-3.5-sonnet"),
  // ... existing models ...
  "gemini-3-flash-preview": openRouterClient("google/gemini-3-flash-preview"),  // ADD THIS
  "sonnet-4.5": openRouterClient("anthropic/claude-sonnet-4.5"),
  // ...
} as const;
```

### Step 2: Add to `openrouterModelRegistry` object

Find the `openrouterModelRegistry` object and add the registry entry **in alphabetical order by `registryId`**:

```typescript
export const openrouterModelRegistry: {
  [K in ModelName]: ModelMetadata<ModelName>;
} = {
  "openrouter:claude-3-5-sonnet": {
    id: "openrouter:claude-3-5-sonnet",
    // ... existing entry ...
  },
  // ... existing entries ...
  "openrouter:gemini-3-flash-preview": {
    id: "openrouter:gemini-3-flash-preview",
    provider: "openrouter",
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

### Step 3: Validate changes

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
| `registryEntry.provider` | `provider` | Always "openrouter" |
| `registryEntry.contextWindow` | `contextWindow` | Direct mapping from API |
| `registryEntry.maxOutputTokens` | `maxOutputTokens` | From `top_provider.max_completion_tokens` |
| `registryEntry.defaultTemperature` | `defaultTemperature` | Inferred from model ID (-1 for code models) |
| `registryEntry.promptFormat` | `promptFormat` | Inferred: gemini/claude/qwen/mistral → "markdown", gpt/openai → "xml", deepseek → "bracket" |
| `registryEntry.supportsReasoning` | `supportsReasoning` | Checks `supported_parameters` for `include_reasoning` or `reasoning` |
| `registryEntry.supportsToolCalling` | `supportsToolCalling` | Checks `supported_parameters` for `tools` |
| `registryEntry.costPerInputToken` | `costPerInputToken` | From `pricing.prompt` |
| `registryEntry.costPerOutputToken` | `costPerOutputToken` | From `pricing.completion` |

## Default Value Inferences

| Field | Logic |
|-------|-------|
| `defaultTemperature` | Returns `-1` if model ID contains "codex", "coder", or "code"; otherwise `0.5` |
| `promptFormat` | Maps model family to format: gemini/claude/qwen/mistral/moonshotai → "markdown", gpt/openai → "xml", deepseek → "bracket", defaults to "markdown" |
| `maxOutputTokens` | Falls back to `context_length` if `max_completion_tokens` is not available |

## Error Handling

- Validates model ID exists in OpenRouter API
- Shows first 10 available models if ID not found
- Provides clear error messages with suggestions

## Examples

```bash
# Add Gemini 3 Flash Preview
./add-model.js google/gemini-3-flash-preview

# Add Claude 3.5 Sonnet
./add-model.js anthropic/claude-3-5-sonnet

# Add DeepSeek V3
./add-model.js deepseek/deepseek-v3.2
```
