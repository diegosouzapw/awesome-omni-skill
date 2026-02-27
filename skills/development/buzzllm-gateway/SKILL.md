---
name: BuzzLLM Gateway
description: Use buzzllm CLI to perform web searches, execute Python code in Docker, search/analyze local codebases, or generate code change blocks. Trigger when user asks for web search, Python execution, code analysis, or code modifications via LLM.
---

## What is BuzzLLM

A CLI gateway that invokes LLMs with specialized tools:
- **websearch**: Search the web and scrape pages
- **pythonexec**: Execute Python code in a sandboxed Docker container
- **codesearch**: Search and read files in local codebases
- **hackhub**: Generate Search-Replace blocks for code modifications

## Command Structure

```bash
buzzllm "MODEL" "API_URL" "PROMPT" \
    --provider PROVIDER \
    --api-key-name ENV_VAR \
    --system-prompt TEMPLATE \
    [--think]  # for extended thinking
    [--brief]  # hide tool calls/results, show only final output
```

## Providers

| Provider | API URL | Notes |
|----------|---------|-------|
| `openai-chat` | `https://api.openai.com/v1/chat/completions` | Standard chat completions |
| `openai-responses` | `https://api.openai.com/v1/responses` | Reasoning models (o1, o3) |
| `anthropic` | `https://api.anthropic.com/v1/messages` | Claude models |
| `vertexai-anthropic` | `https://REGION-aiplatform.googleapis.com/v1/projects/PROJECT/locations/REGION/publishers/anthropic/models/MODEL:streamRawPredict` | Claude via GCP |

## System Prompt Templates

### With Tools
- `websearch` - Web search + page scraping (`search_web`, `scrape_webpage`)
- `codesearch` - Codebase analysis (`bash_find`, `bash_ripgrep`, `bash_read`)
- `pythonexec` - Python in Docker (`python_execute`)

### Without Tools
- `hackhub` - Search-Replace block generation
- `generate` - General generation
- `helpful` - Helpful assistant
- `replace` - Replacement template

## Usage Examples

### Web Search
```bash
buzzllm "gpt-4o-mini" \
    "https://api.openai.com/v1/chat/completions" \
    "What is the current price of Bitcoin?" \
    --provider openai-chat \
    --api-key-name OPENAI_API_KEY \
    --system-prompt websearch
```

### Python Execution
```bash
# Requires Docker container: cd python_runtime_docker && bash build_docker.sh build-python-exec
buzzllm "claude-sonnet-4-20250514" \
    "https://api.anthropic.com/v1/messages" \
    "Calculate the first 20 Fibonacci numbers using Python" \
    --provider anthropic \
    --api-key-name ANTHROPIC_API_KEY \
    --system-prompt pythonexec
```

### Code Search
```bash
buzzllm "gpt-4o-mini" \
    "https://api.openai.com/v1/chat/completions" \
    "How does error handling work in this codebase?" \
    --provider openai-chat \
    --api-key-name OPENAI_API_KEY \
    --system-prompt codesearch
```

### Code Modifications (HackHub)
```bash
buzzllm "claude-sonnet-4-20250514" \
    "https://api.anthropic.com/v1/messages" \
    "$(cat src/main.py)\nAdd logging to all functions" \
    --provider anthropic \
    --api-key-name ANTHROPIC_API_KEY \
    --system-prompt hackhub
```

### Extended Thinking (Claude/OpenAI reasoning)
```bash
buzzllm "claude-sonnet-4-20250514" \
    "https://api.anthropic.com/v1/messages" \
    "Solve this complex problem step by step" \
    --provider anthropic \
    --api-key-name ANTHROPIC_API_KEY \
    --think
```

## When to Use

1. **User asks for web search**: Use `--system-prompt websearch`
2. **User wants Python execution**: Use `--system-prompt pythonexec` (ensure Docker is running)
3. **User asks about codebase**: Use `--system-prompt codesearch`
4. **User wants code changes**: Use `--system-prompt hackhub`
5. **Complex reasoning needed**: Add `--think` flag

## Environment Variables Required

- `OPENAI_API_KEY` - For OpenAI models
- `ANTHROPIC_API_KEY` - For Anthropic models
- `BRAVE_SEARCH_AI_API_KEY` - Fallback for websearch if DuckDuckGo fails

## Output

Default: streams text to stdout
With `-S` flag: SSE format for programmatic consumption
