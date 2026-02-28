---
name: add-model
description: Add a new AI model to a provider (anthropic, openai, groq, cerebras). Use when a new model is announced and needs to be supported in aico.
argument-hint: "[provider:model-id]"
---

# Add Model Skill

Add a new AI model to the aico codebase.

## Arguments

`$ARGUMENTS` should be in the format `provider:model-id` (e.g. `anthropic:claude-opus-4-6`).
If only a model ID is given, infer the provider from the model name prefix:
- `claude-*` → `anthropic`
- `gpt-*`, `o1*`, `o3*` → `openai`
- `llama*`, `mixtral*` → `groq`
- Other → ask the user

## Prerequisite: Gather Model Information

Before writing any code, gather the following from the user or from provided reference URLs:

1. **Model ID** (e.g. `claude-opus-4-6`)
2. **Description** - what the model is best at
3. **Pricing** - input/output per MTok
4. **Context window** and **max output tokens**
5. **Notable capabilities** (e.g. extended thinking, vision, tool calling)
6. **Predecessor model to deprecate** (optional)

If the user provides a reference URL, fetch it to extract this information.

### Provider Reference URLs

When the user does not provide a reference URL, consult the official documentation for the relevant provider:

| Provider  | Models Documentation                                              | Pricing Documentation             |
|-----------|-------------------------------------------------------------------|-----------------------------------|
| Anthropic | https://platform.claude.com/docs/en/about-claude/models/overview  | https://claude.com/pricing#api    |
| OpenAI    | https://platform.openai.com/docs/models                           | https://openai.com/api/pricing/   |
| Groq      | https://console.groq.com/docs/models                              | https://groq.com/pricing/         |
| Cerebras  | https://inference-docs.cerebras.ai/models/overview                | https://www.cerebras.ai/pricing   |

Fetch the relevant page(s) to extract:
- The exact model ID string used in API calls
- Context window size and max output tokens
- Input/output pricing per million tokens
- Release date and predecessor model (if applicable)
- Special capabilities or limitations

## Step-by-step Procedure

### 1. Create the model Go file

Create a new file at `internal/providers/<provider>/<model_file_name>.go`.

The file name convention uses underscores for separators:
- `claude-opus-4-6` → `claude_opus_4_6.go`
- `gpt-4o` → `gpt4o.go`
- `llama-3.3-70b-versatile` → `llama3_3_70b.go`

Use the existing model files in the same provider as a template. Each provider has a different pattern:

#### Anthropic models (`internal/providers/anthropic/`)

- Uses `github.com/anthropics/anthropic-sdk-go` SDK
- Struct has `client *anthropic.Client` and `opts []anthropicopt.RequestOption`
- Constructor takes `*anthropic.Client`
- Uses `buildRequestBody()` helper and `m.client.Messages.New()` / `m.client.Messages.NewStreaming()`
- Must define a `ModelName` constant: `const ModelNameXxx = "model-id"`

Template reference: any existing `claude_*.go` file in the same directory.

#### OpenAI models (`internal/providers/openai/`)

- Uses the in-house `openai.APIClient`
- Constructor takes `apiKey string`
- Uses `BuildChatRequest()` and `m.client.DoPost()` / `m.client.DoStream()`

Template reference: `gpt4o.go`

#### Groq / Cerebras models (OpenAI-compatible)

- Uses `openai.APIClient` from the openai package
- Constructor takes `apiKey string`
- Delegates to `openai.GenerateContent()` / `openai.GenerateContentStream()` with the provider's `Endpoint`

Template reference: any model file in `internal/providers/groq/` or `internal/providers/cerebras/`

### 2. Register the model in the provider file

Edit the provider's main file (`internal/providers/<provider>/<provider>.go` or `anthropic.go`):

1. **`AvailableModels()`** - Add the new model struct to the returned slice
2. **`selectModel()`** - Add a `case` for the model ID
3. **`NewGenerativeModel()`** - Add a `case` for the model ID with the constructor

Place the new model at an appropriate position (typically at the top for the latest/best model).

### 3. Update the doc comment

Update the documentation comment block at the top of the provider file to include the new model's description and pricing.

### 4. Deprecate predecessor (if requested)

If the user requests deprecating an older model:

1. Prefix its `Description()` return with `[Deprecated] <Model Name> - superseded by <New Model>.`
2. Move its entry in the doc comment to the bottom with `(Deprecated)` suffix
3. Keep all code functional — do NOT remove the model

### 5. Build and test

Run:
```
go build ./...
go test ./...
```

### 6. Verify with CLI

Run and show the user the output of:
```
go run ./cmd/aico models
go run ./cmd/aico models describe <new-model-id>
```

If a predecessor was deprecated, also show:
```
go run ./cmd/aico models describe <deprecated-model-id>
```

### 7. Commit

Follow the Conventional Commits format defined in CLAUDE.md:
```
feat(api): add <Model Name> support
```

If a predecessor was deprecated, mention it:
```
feat(api): add <Model Name> support and deprecate <Old Model>
```
