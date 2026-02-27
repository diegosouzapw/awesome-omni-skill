---
name: toon-format
description: When working with JSON data in LLM prompts (especially large arrays or tabular data), consider the token-efficient TOON (Token-Oriented Object Notation) format which reduces tokens by 30-70% while maintaining lossless JSON representation and structural validation. Use for reading/writing .toon files, converting JSON↔TOON, or optimizing structured data for LLM consumption with guardrails like [N] counts and {field} headers.
---

# TOON Format Skill

TOON (Token-Oriented Object Notation) is a compact, human-readable, schema-aware encoding of JSON designed to be easy for LLMs to *read* and (with good prompting) to *generate*. It is especially effective for **uniform arrays of objects** (tabular data), where keys are declared once and rows carry values.

## Quick decision: should I use TOON?
Use TOON when:
- You are passing **large structured context** (especially arrays of objects) to an LLM and want fewer tokens.
- You want **structure guardrails** (`[N]` counts + `{fields}` headers) to reduce model drift and enable validation.
- You need a format that stays close to CSV’s readability but is a **lossless representation of JSON**.

Prefer JSON (or JSONL/CSV) when:
- You need interoperability with systems that expect standard JSON.
- Data is extremely irregular (ragged objects) and the TOON representation won’t be more compact or readable.
- The task is primarily “machine ↔ machine” with no human/LLM in the loop.

For more detail: see [reference/when-to-use.md](reference/when-to-use.md).

## Core patterns you must follow

### 1) Recognize TOON in the wild
TOON commonly looks like:

- Scalars/objects as `key: value` / indented blocks
- Arrays with lengths: `users[3]: ...`
- Tabular arrays with field headers: `users[3]{id,name,role}:`
- Rows are delimiter-separated (comma by default; tab often best)

### 2) Converting JSON ↔ TOON (recommended paths)

#### A) CLI (fastest, good for pipelines)
Use the official CLI for round-tripping and stats.

**Encode JSON → TOON**
```bash
npx @toon-format/cli input.json -o output.toon
```

**Decode TOON → JSON**
```bash
npx @toon-format/cli input.toon -o output.json
```

**Token stats + maximum efficiency**
```bash
npx @toon-format/cli input.json --delimiter "\t" --stats -o output.toon
```

**Optional structure compression**
- Key folding (encode): `--keyFolding safe`
- Path expansion (decode): `--expandPaths safe`

For full CLI options and examples: see [reference/cli.md](reference/cli.md).

#### B) TypeScript/JavaScript library
Use `@toon-format/toon` for in-app encode/decode. See [reference/library.md](reference/library.md).

### 3) Prompting patterns for LLMs (critical)
When sending TOON as input:
- **Show, don’t explain.**
- Wrap data in a fenced code block labeled `toon` (or `yaml` if needed):
  - ```toon
  - (TOON data)
  - ```

When asking for TOON output:
- Provide the **expected header template** (e.g., `users[N]{id,name,role}:`)
- Explicit rules:
  - 2-space indent
  - `[N]` must match row count
  - no extra commentary; output only the code block (if you need strict parsing)

For proven prompt patterns and pitfalls: see [reference/llm-prompting.md](reference/llm-prompting.md).

### 4) Always validate model-generated TOON
If the model outputs TOON, you MUST validate it (strict mode by default in official tooling). If validation fails:
- Ask for re-output with the same header template
- Or fall back to JSON output

## Minimal working examples

### Example: filter rows (LLM output expected as TOON)
**Input**
```toon
users[3]{id,name,role,lastLogin}:
  1,Alice,admin,2025-01-15T10:30:00Z
  2,Bob,user,2025-01-14T15:22:00Z
  3,Charlie,user,2025-01-13T09:45:00Z
```

**Instruction**
Return only users with role "user" as TOON. Use the same header format. Set `[N]` correctly. Output only the code block.

---

## Utilities included in this Skill
- `scripts/toon_convert.py`: A convenience wrapper that:
  - detects TOON code fences in Markdown and extracts them
  - calls the official CLI via `toon` (if installed) or `npx @toon-format/cli`
  - supports `--encode/--decode`, `--delimiter`, `--stats`, and folding/expansion flags

See: [scripts/README.md](scripts/README.md)
