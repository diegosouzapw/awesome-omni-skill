---
name: StopTimizer
description: Precise token counter for GPT, Claude, and Gemini models (source of truth from software kernel)
category: optimization
tags: [tokens, precision, llm]
tools:
  - ./stoptimizer.ts
---

# StopTimizer

Precise token counting using official tokenizers as source of truth from software kernel.

## Purpose

Provides accurate token counts for LLM models (GPT, Claude, Gemini) without approximation. Does not perform validation or judgment—returns raw numbers only (Unix philosophy).

## Usage

### Count All Models

```bash
deno run --allow-net <url>/stoptimizer.ts "hello world"
# Output: 2 2 2 2 2 2 2 (space-separated: gpt-5 gpt-5.2 gpt-5-mini gpt-4o gpt-4 gpt-3.5 claude)
```

### JSON Output

```bash
deno run --allow-net <url>/stoptimizer.ts --json "test"
# Output: {"gpt-5":1,"gpt-5.2":1,"gpt-5-mini":1,"gpt-4o":1,"gpt-4":1,"gpt-3.5":1,"claude-3.5-sonnet":1}
```

### Single Model

```bash
deno run --allow-net <url>/stoptimizer.ts --model gpt-4o "text"
# Output: 1
```

### stdin Support (Large Files)

For files too large for command-line arguments, use stdin:

```bash
# Using dash (Unix tradition)
cat KERNEL.md | deno run --allow-net <url>/stoptimizer.ts -

# Using --stdin flag (modern clarity)
deno run --allow-net <url>/stoptimizer.ts --stdin < large-file.txt

# Combined with other options
cat prompt.txt | deno run --allow-net <url>/stoptimizer.ts - --json
cat README.md | deno run --allow-net <url>/stoptimizer.ts --stdin --model gpt-4o
```

## Supported Models

### GPT Models (OpenAI)
- **gpt-5** - GPT-5 (o200k_base encoding, 100% precise)
- **gpt-5.2** - GPT-5.2 (o200k_base encoding, 100% precise)
- **gpt-5-mini** - GPT-5 mini (o200k_base encoding, 100% precise)
- **gpt-4o** - GPT-4o (o200k_base encoding, 100% precise)
- **gpt-4** - GPT-4 (cl100k_base encoding, 100% precise)
- **gpt-3.5** - GPT-3.5 Turbo (cl100k_base encoding, 100% precise)

### Other Models
- **claude-3.5-sonnet** - Claude 3.5 Sonnet (100% precise via Anthropic tokenizer)
- **gemini-2.0-flash** - Gemini 2.0 Flash (research in progress, use Vertex AI API)

**Encoding Note**: GPT-5 family and GPT-4o use `o200k_base` (~2.3 MB vocabulary), while GPT-4 and GPT-3.5 use `cl100k_base` (~1.1 MB vocabulary). Token counts will differ between encodings for the same text.

## STOP Protocol Validation

Tool provides counts only. Compose validation logic in shell scripts:

```bash
#!/bin/bash
# Example: STOP validation with stdin (for large prompts)

OLD=$(cat prompt-v1.txt | deno run --allow-net <url>/stoptimizer.ts - --model gpt-4o)
NEW=$(cat prompt-v2.txt | deno run --allow-net <url>/stoptimizer.ts - --model gpt-4o)

if [ "$OLD" -eq "$NEW" ]; then
  CHARS_OLD=$(wc -c < prompt-v1.txt)
  CHARS_NEW=$(wc -c < prompt-v2.txt)
  GAIN=$((CHARS_NEW - CHARS_OLD))
  echo "✅ STOP COMPLIANT: Both $OLD tokens, gained $GAIN chars"
  exit 0
else
  echo "❌ STOP VIOLATION: $OLD vs $NEW tokens"
  exit 1
fi
```

### CI/CD Integration

```yaml
# .github/workflows/stop-validation.yml
name: STOP Protocol Validation
on: [pull_request]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: denoland/setup-deno@v1
      
      - name: Validate prompt changes
        run: |
          TOOL_URL="https://raw.githubusercontent.com/ShipFail/promptware/main/os/skills/stoptimizer/stoptimizer.ts"
          
          # Check each optimization
          deno run --allow-net $TOOL_URL --model gpt-4o "args" > /tmp/old
          deno run --allow-net $TOOL_URL --model gpt-4o "arguments" > /tmp/new
          [ "$(cat /tmp/old)" -eq "$(cat /tmp/new)" ] || exit 1
```

## Performance

- **First run**: ~2-3 MB download (vocabularies cached in `~/.cache/deno/`)
- **Subsequent runs**: Instant (cached modules)
- **Memory**: ~10-15 MB runtime
- **Speed**: ~100-200 tokens/sec per model

## Exit Codes

- `0` - Success (token count returned)
- `2` - Error (network failure, invalid model, missing argument)

## Limitations

- Requires network access on first run for vocabulary download
- Gemini tokenizer requires additional research (SentencePiece WASM integration)
- Text-only (no multimodal token counting)
- No streaming support (processes entire text at once)

## References

- [RFC 0022: STOP Protocol](../../rfcs/0022-semantic-token-optimization-protocol.md)
- [RFC 0012: Skill Specification](../../rfcs/0012-sys-skill-spec.md)
- [js-tiktoken](https://github.com/dqbd/tiktoken)
- [OpenAI Tokenizer](https://platform.openai.com/tokenizer)

## Examples

### Basic Token Counting

```bash
$ deno run --allow-net stoptimizer.ts "The quick brown fox"
5 5 5 5
```

### Comparing Abbreviations vs Full Words

```bash
$ deno run --allow-net stoptimizer.ts --json "params"
{"gpt-4o":1,"gpt-4":1,"gpt-3.5":1,"claude-3.5-sonnet":1}

$ deno run --allow-net stoptimizer.ts --json "parameters"
{"gpt-4o":1,"gpt-4":1,"gpt-3.5":1,"claude-3.5-sonnet":1}

# Both tokenize to 1 token - STOP compliant!
```

### Programmatic Use

```typescript
import { Tiktoken, getGPTVocab } from "./deps.ts";

const vocab = getGPTVocab("cl100k_base");
const tokenizer = new Tiktoken(
  vocab.bpe_ranks,
  vocab.special_tokens,
  vocab.pat_str
);

const tokens = tokenizer.encode("hello world");
console.log(`Token count: ${tokens.length}`);
tokenizer.free();
```

