---
name: add-provider
description: Add a new AI provider or model for recipe generation. Use when adding support for a new LLM provider (Anthropic, Google, etc.) or adding models to an existing provider.
---

# Adding AI Providers

## Quick Start

To add a new provider:

1. Create provider directory: `app/lib/providers/{provider-name}/`
2. Create `prompts.ts` with model-specific prompts
3. Create `index.ts` implementing the `RecipeProvider` interface
4. Register in `app/lib/providers/index.ts`
5. Add model display names to `app/recipes/page.tsx`
6. Install SDK: `bun add @{provider-name}/sdk`

> **Warning**: API parameters can change between model versions! For example, OpenAI's `gpt-4o-mini` uses `max_tokens` while `gpt-5-mini` uses `max_completion_tokens`. Always check the provider's docs for each specific model.

## Architecture Overview

```
app/lib/providers/
├── index.ts              # Provider registry + factory
├── types.ts              # Shared provider interfaces
├── base.ts               # Base provider class with shared logic
├── openai/
│   ├── index.ts          # OpenAI provider implementation
│   └── prompts.ts        # OpenAI-specific prompts
└── {your-provider}/
    ├── index.ts          # Your provider implementation
    └── prompts.ts        # Your provider's prompts
```

## Step-by-Step Implementation

### 1. Create Provider Directory

```bash
mkdir -p app/lib/providers/{provider-name}
```

### 2. Create Prompts (`prompts.ts`)

Define the system prompt and user prompt template optimized for your model:

```typescript
export const SYSTEM_PROMPT = `You are an expert Colorist...`

export const USER_PROMPT_TEMPLATE = (analysisJson: string) =>
  `Pre-processing data:\n${analysisJson}\n\nAnalyze this image and create a Ricoh GR III recipe.`
```

See `app/lib/providers/openai/prompts.ts` for the full prompt structure.

### 3. Create Provider (`index.ts`)

```typescript
import { BaseProvider, transformToRecipe } from '../base'
import { registerProvider } from '../index'
import type { GenerateRecipeResponse, ModelConfig, ProviderConfig } from '../types'
import type { ImageAnalysis } from '../../types'
import { SYSTEM_PROMPT, USER_PROMPT_TEMPLATE } from './prompts'

class YourProvider extends BaseProvider {
  readonly config: ProviderConfig = {
    id: 'your-provider',
    name: 'Your Provider',
    models: [
      {
        id: 'model-id-here',
        name: 'Model Display Name',
        systemPrompt: SYSTEM_PROMPT,
        userPromptTemplate: USER_PROMPT_TEMPLATE,
        maxTokens: 8000,
      },
    ],
    defaultModel: 'model-id-here',
  }

  async generateRecipe(
    imageBase64: string,
    mimeType: string,
    analysis: ImageAnalysis,
    modelId?: string
  ): Promise<GenerateRecipeResponse> {
    const model = this.getModelOrDefault(modelId)
    const analysisJson = JSON.stringify(analysis, null, 2)

    // Call your provider's API here
    // Parse the response
    // Use transformToRecipe() from base.ts to convert to LLMRecipe

    const recipe = transformToRecipe(parsed)

    return {
      recipe,
      reasoning: parsed.reasoning || '',
      model: model.id,
      provider: this.config.id,
    }
  }
}

const provider = new YourProvider()

export function register(): void {
  registerProvider(provider)
}

export { provider }
```

### 4. Register Provider

In `app/lib/providers/index.ts`, add the import at the bottom:

```typescript
import('./your-provider').then((m) => m.register()).catch(console.error)
```

### 5. Add Model Display Names

In `app/recipes/page.tsx`, add your models to `MODEL_DISPLAY_NAMES`:

```typescript
const MODEL_DISPLAY_NAMES: Record<string, string> = {
  // ... existing models
  'your-model-id': 'Display Name',
}
```

### 6. Set Environment Variables

Add required API keys to `.env.local`:

```
YOUR_PROVIDER_API_KEY=sk-xxx
```

Optionally set the default model:

```
RECIPE_MODEL=your-model-id
```

## Key Interfaces

### ProviderConfig

```typescript
interface ProviderConfig {
  id: string           // Unique ID: 'openai', 'anthropic'
  name: string         // Display name: 'OpenAI', 'Anthropic'
  models: ModelConfig[]
  defaultModel: string // Default model ID
}
```

### ModelConfig

```typescript
interface ModelConfig {
  id: string                              // Model ID from provider
  name: string                            // Human-readable name
  systemPrompt: string                    // System prompt for this model
  userPromptTemplate: (json: string) => string  // User prompt template
  maxTokens: number                       // Max completion tokens
}
```

### GenerateRecipeResponse

```typescript
interface GenerateRecipeResponse {
  recipe: LLMRecipe    // Transformed recipe
  reasoning: string    // AI's reasoning
  model: string        // Model ID used
  provider: string     // Provider ID used
}
```

## Shared Utilities (from `base.ts`)

- `transformToRecipe(parsed)` - Convert raw API response to LLMRecipe
- `transformWhiteBalance(wb)` - Convert white balance format
- `transformCorrection(value)` - Normalize correction enum values
- `clamp(value, min, max)` - Clamp numbers to valid ranges

## Recipe Output Schema

The model must output JSON matching this schema:

| Field | Type | Range/Values |
|-------|------|--------------|
| recipe_name | string | Evocative name |
| image_control_mode | enum | Standard, Vivid, Monotone, etc. |
| saturation | integer | -4 to 4 |
| hue | integer | -4 to 4 |
| high_low_key | integer | -4 to 4 |
| contrast | integer | -4 to 4 |
| contrast_highlight | integer | -4 to 4 |
| contrast_shadow | integer | -4 to 4 |
| sharpness | integer | -4 to 4 |
| shading | integer | -4 to 4 |
| clarity | integer | -4 to 4 |
| grain_effect | integer | 0 to 3 |
| white_balance | object | mode, color_temperature_k, compensation_a, compensation_g |
| highlight_correction | enum | Auto, On, Off |
| shadow_correction | enum | Auto, Low, Medium, High, Off |
| peripheral_illumination_correction | boolean | |
| high_iso_noise_reduction | enum | Auto, Low, Medium, High, Off, Custom |
| reasoning | string | 2-4 sentences |

See [REFERENCE.md](REFERENCE.md) for full implementation details.
See [examples/anthropic.md](examples/anthropic.md) for a complete example.
