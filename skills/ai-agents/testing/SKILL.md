---
name: testing
description: Lingua's testing tools for transformation coverage and end-to-end SDK validation.
---

# Lingua Testing

Two complementary testing approaches for validating cross-provider transformations.

## Overview

| Tool | Purpose | Uses API? | When to Use |
|------|---------|-----------|-------------|
| `coverage-report` | Field-level roundtrip coverage | No | Finding missing/changed fields |
| `lingua-capture` + `test:transforms` | Real SDK validation | Capture only | Validating against real APIs |

## Coverage Report (Rust)

Tests transformations roundtrip without calling APIs. Detects lost, added, or changed fields.

### Quick Iteration (Compact Mode)

```bash
# After code changes - fast validation
cargo run --bin coverage-report -- -f compact

# Filter to specific providers
cargo run --bin coverage-report -- -f compact -p anthropic,responses

# Filter to specific test case
cargo run --bin coverage-report -- -f compact -t "reasoning*"
```

### Full Investigation (Markdown Mode)

```bash
# Initial investigation with full error details
cargo run --bin coverage-report

# Save to bug file for planning
cargo run --bin coverage-report -- -p anthropic > .claude/anthropic_bugs.md
```

### Reading Compact Output

```
Stats: 669/1704 (39.3%) [512+157lim] 1035fail

[P1] L:usage.prompt_cache_tokens (123)
  ant→ggl: cacheParam (response)...(+44)
```

- `L:` = Lost fields, `A:` = Added, `C:` = Changed
- `(123)` = affected test cases
- Abbreviations: `oai`=ChatCompletions, `ant`=Anthropic, `rsp`=Responses, `ggl`=Google, `bed`=Bedrock

## SDK Transforms Testing (TypeScript)

Validates against real provider SDKs with schema validation at every step.

### Transformation Pairs

- `chat-completions ↔ anthropic`
- `responses ↔ anthropic`

### Capture Responses

Calls real SDKs (requires API keys):

```bash
cd payloads

pnpm lingua-capture              # Capture missing only
pnpm lingua-capture --force      # Recapture all
pnpm lingua-capture toolCall     # Filter by name
```

Output: `transforms/{source}_to_{target}/{caseName}.json`

### Run Tests

Uses captured responses (no API calls):

```bash
cd payloads

pnpm test:transforms             # Run all
pnpm test:transforms:update      # Update snapshots
pnpm test:transforms:watch       # Watch mode
```

### What It Validates

1. **Request transform** → validates output against target schema
2. **Load captured response** → validates against target response schema
3. **Response transform back** → validates against source schema
4. **Snapshots** → detects unintended output changes

### Known Incompatibilities

Some transformations are impossible. Document in `transform_errors.json`:

```json
{
  "responses_to_anthropic": {
    "reasoningRequestTruncated": "max_tokens (100) < budget_tokens (1024 min)"
  }
}
```

These tests pass silently.

## Workflow

### Adding a New Test Case

1. Add case to `payloads/cases/`:
   ```typescript
   newFeature: {
     "chat-completions": { model: "...", messages: [...] },
     anthropic: { model: "...", max_tokens: 1024, messages: [...] },
   }
   ```

2. Check coverage:
   ```bash
   cargo run --bin coverage-report -- -f compact -t newFeature
   ```

3. Capture SDK responses:
   ```bash
   cd payloads && pnpm lingua-capture newFeature
   ```

4. Run transform tests:
   ```bash
   pnpm test:transforms
   ```

### Fixing a Transformation Bug

1. Investigate with full coverage report:
   ```bash
   cargo run --bin coverage-report -- -p anthropic,responses > .claude/fix_bugs.md
   ```

2. Iterate with compact mode:
   ```bash
   cargo run --bin coverage-report -- -f compact -p anthropic,responses
   ```

3. Verify SDK compatibility:
   ```bash
   cd payloads && pnpm test:transforms
   ```

4. If snapshots changed intentionally:
   ```bash
   pnpm test:transforms:update
   ```
