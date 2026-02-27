---
name: exa-answer
description: Generate answers to questions with structured output using AI search and synthesis. Use when you need factual answers with citations from web sources, or when you want to extract specific structured information in response to a query.
---

# Exa Answer

Token-efficient strategies for generating answers with structured output using exa-ai.

**Use `--help` to see available commands and verify usage before running:**
```bash
exa-ai <command> --help
```

## Critical Requirements

**MUST follow these rules when using exa-ai answer:**

### Shared Requirements

This skill inherits requirements from [Common Requirements](../../../docs/common-requirements.md):
- Schema design patterns → All schema operations
- Output format selection → All output operations

### MUST NOT Rules

1. **Avoid --text flag**: Use `--text` only when you need full source text; otherwise rely on default behavior for better token efficiency

## Cost Optimization

### Pricing
- **Per answer**: $0.005

**Cost strategy:**
- Use `answer` for questions with moderate complexity that need AI synthesis
- For simple lookups, use `search` instead (same cost but gives you URLs for verification)
- Consider whether you need a synthesized answer or just search results

## Token Optimization

**Apply these strategies:**

- **Use toon format**: `--output-format toon` for 40% fewer tokens than JSON (use when reading output directly)
- **Use JSON + jq**: Extract only needed fields with jq (use when piping/processing output)
- **Use schemas**: Structure answers with `--output-schema` for consistent, parseable output
- **Custom system prompts**: Use `--system-prompt` to guide answer style and format

**IMPORTANT**: Choose one approach, don't mix them:
- **Approach 1: toon only** - Compact YAML-like output for direct reading
- **Approach 2: JSON + jq** - Extract specific fields programmatically
- **Approach 3: Schemas + jq** - Get structured data, always use JSON output (default) and pipe to jq

Examples:
```bash
# ❌ High token usage
exa-ai answer "What is Claude?"

# ✅ Approach 1: toon format for direct reading (40% reduction)
exa-ai answer "What is Claude?" --output-format toon

# ✅ Approach 2: JSON + jq for field extraction (80% reduction)
exa-ai answer "What is Claude?" \
  --output-schema '{"type":"object","properties":{"product":{"type":"string"}}}' | jq -r '.answer.product'

# ❌ Don't mix toon with jq (toon is YAML-like, not JSON)
exa-ai answer "What is Claude?" --output-format toon | jq -r '.answer'
```

## Quick Start

### Basic Answer
```bash
exa-ai answer "What is Anthropic's main product?" --output-format toon
```

### Structured Output
```bash
exa-ai answer "What is Claude?" \
  --output-schema '{"type":"object","properties":{"product_name":{"type":"string"},"company":{"type":"string"},"description":{"type":"string"}}}'
```

### Array Output for Lists
```bash
exa-ai answer "What are the top 5 programming languages in 2024?" \
  --output-schema '{"type":"object","properties":{"languages":{"type":"array","items":{"type":"string"}}}}' | jq -r '.answer.languages | map("- " + .) | join("\n")'
```

### Custom System Prompt
```bash
exa-ai answer "Explain quantum computing" \
  --system-prompt "Respond in simple terms suitable for a high school student"
```

## Detailed Reference

For complete options, examples, and schema design tips, consult [REFERENCE.md](REFERENCE.md).

### Shared Requirements

<shared-requirements>

## Schema Design

### MUST: Use object wrapper for schemas

**Applies to**: answer, search, find-similar, get-contents

When using schema parameters (`--output-schema` or `--summary-schema`), always wrap properties in an object:

```json
{"type":"object","properties":{"field_name":{"type":"string"}}}
```

**DO NOT** use bare properties without the object wrapper:
```json
{"properties":{"field_name":{"type":"string"}}}  // ❌ Missing "type":"object"
```

**Why**: The Exa API requires a valid JSON Schema with an object type at the root level. Omitting this causes validation errors.

**Examples**:
```bash
# ✅ CORRECT - object wrapper included
exa-ai search "AI news" \
  --summary-schema '{"type":"object","properties":{"headline":{"type":"string"}}}'

# ❌ WRONG - missing object wrapper
exa-ai search "AI news" \
  --summary-schema '{"properties":{"headline":{"type":"string"}}}'
```

---

## Output Format Selection

### MUST NOT: Mix toon format with jq

**Applies to**: answer, context, search, find-similar, get-contents

`toon` format produces YAML-like output, not JSON. DO NOT pipe toon output to jq for parsing:

```bash
# ❌ WRONG - toon is not JSON
exa-ai search "query" --output-format toon | jq -r '.results'

# ✅ CORRECT - use JSON (default) with jq
exa-ai search "query" | jq -r '.results[].title'

# ✅ CORRECT - use toon for direct reading only
exa-ai search "query" --output-format toon
```

**Why**: jq expects valid JSON input. toon format is designed for human readability and produces YAML-like output that jq cannot parse.

### SHOULD: Choose one output approach

**Applies to**: answer, context, search, find-similar, get-contents

Pick one strategy and stick with it throughout your workflow:

1. **Approach 1: toon only** - Compact YAML-like output for direct reading
   - Use when: Reading output directly, no further processing needed
   - Token savings: ~40% reduction vs JSON
   - Example: `exa-ai search "query" --output-format toon`

2. **Approach 2: JSON + jq** - Extract specific fields programmatically
   - Use when: Need to extract specific fields or pipe to other commands
   - Token savings: ~80-90% reduction (extracts only needed fields)
   - Example: `exa-ai search "query" | jq -r '.results[].title'`

3. **Approach 3: Schemas + jq** - Structured data extraction with validation
   - Use when: Need consistent structured output across multiple queries
   - Token savings: ~85% reduction + consistent schema
   - Example: `exa-ai search "query" --summary-schema '{...}' | jq -r '.results[].summary | fromjson'`

**Why**: Mixing approaches increases complexity and token usage. Choosing one approach optimizes for your use case.

---

## Shell Command Best Practices

### MUST: Run commands directly, parse separately

**Applies to**: monitor, search (websets), research, and all skills using complex commands

When using the Bash tool with complex shell syntax, run commands directly and parse output in separate steps:

```bash
# ❌ WRONG - nested command substitution
webset_id=$(exa-ai webset-create --search '{"query":"..."}' | jq -r '.webset_id')

# ✅ CORRECT - run directly, then parse
exa-ai webset-create --search '{"query":"..."}'
# Then in a follow-up command:
webset_id=$(cat output.json | jq -r '.webset_id')
```

**Why**: Complex nested `$(...)` command substitutions can fail unpredictably in shell environments. Running commands directly and parsing separately improves reliability and makes debugging easier.

### MUST NOT: Use nested command substitutions

**Applies to**: All skills when using complex multi-step operations

Avoid nesting multiple levels of command substitution:

```bash
# ❌ WRONG - deeply nested
result=$(exa-ai search "$(cat query.txt | tr '\n' ' ')" --num-results $(cat config.json | jq -r '.count'))

# ✅ CORRECT - sequential steps
query=$(cat query.txt | tr '\n' ' ')
count=$(cat config.json | jq -r '.count')
exa-ai search "$query" --num-results $count
```

**Why**: Nested command substitutions are fragile and hard to debug when they fail. Sequential steps make each operation explicit and easier to troubleshoot.

### SHOULD: Break complex commands into sequential steps

**Applies to**: All skills when working with multi-step workflows

For readability and reliability, break complex operations into clear sequential steps:

```bash
# ❌ Less maintainable - everything in one line
exa-ai webset-create --search '{"query":"startups","count":1}' | jq -r '.webset_id' | xargs -I {} exa-ai webset-search-create {} --query "AI" --behavior override

# ✅ More maintainable - clear steps
exa-ai webset-create --search '{"query":"startups","count":1}'
webset_id=$(jq -r '.webset_id' < output.json)
exa-ai webset-search-create $webset_id --query "AI" --behavior override
```

**Why**: Sequential steps are easier to understand, debug, and modify. Each step can be verified independently.

</shared-requirements>
