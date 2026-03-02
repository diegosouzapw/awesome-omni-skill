---
metadata.clawdbot:
  name: aura-openclaw
  description: Compile documents into knowledge bases and manage persistent AI agent memory with Aura Core
  version: 0.1.1
  author: Auralith Inc.
  homepage: https://github.com/Auralith-Inc/aura-openclaw
requires:
  env: []
files: ["scripts/*"]
---

# Aura for OpenClaw

Persistent memory and instant knowledge retrieval for your OpenClaw agent. Compile 60+ file formats into queryable `.aura` archives and give your agent a 3-tier memory that survives sessions, compaction, and restarts.

## Setup

```bash
pip install auralith-aura
```

For full document support (PDFs, DOCX, etc.):
```bash
pip install 'aura-core[all]'
```

## What It Looks Like

### Learn an entire codebase

```
You: Learn my project in ~/repos/api-service
Agent: 🔥 Compiling ~/repos/api-service...
       ✅ Compiled 847 files → api-service.aura
       Indexed: Python (312), TypeScript (201), YAML (44), Markdown (73), JSON (217)

You: How does the auth middleware handle token refresh?
Agent: Based on src/middleware/auth.ts (lines 45-89), the auth middleware
       intercepts expired JWTs by checking the `exp` claim. When expired,
       it calls `refreshTokenService.rotate()` which issues a new access
       token and sets it via the `X-Refreshed-Token` response header.
       The original request is then replayed with the new token.
```

### Compile legal docs with PII masking

```
You: Learn everything in ~/legal but mask any personal info
Agent: 🔥 Compiling with PII masking enabled...
       ✅ Compiled → legal.aura
       Masked: 23 emails, 8 phone numbers, 4 SSNs

You: What are the termination conditions in the Series A agreement?
Agent: Per series_a_term_sheet.pdf, Section 7.2 — the agreement may be
       terminated by either party with 30 days written notice if:
       (a) material breach remains uncured after 15 business days,
       (b) insolvency or bankruptcy filing, or
       (c) failure to meet the minimum revenue threshold in Schedule B...
```

### Remember facts across sessions

```
You: Remember that our production database is on us-east-1, the staging
     API key rotates every 30 days, and I prefer verbose error logs.

Agent: ✅ Written to /fact:
       • Production DB region: us-east-1
       • Staging API key rotation: 30-day cycle
       • Preference: verbose error logging

--- next day, new session ---

You: Set up the deployment config for staging
Agent: I remember your staging API key rotates every 30 days — I'll add
       a reminder comment in the config. Also setting error logging to
       verbose per your preference. Targeting us-east-1 since that's
       where your production DB lives...
```

### Track decisions across a long project

```
You: We decided to use PostgreSQL over MongoDB for the user service,
     and the team agreed on GraphQL for the public API.

Agent: ✅ Written to /episodic:
       • Architecture decision: PostgreSQL for user service (over MongoDB)
       • API decision: GraphQL for public-facing API

--- two weeks later ---

You: Wait, should we use MongoDB for the new analytics service?
Agent: Based on your earlier decision log — you chose PostgreSQL over
       MongoDB for the user service citing ACID compliance needs. The
       analytics service has different requirements (write-heavy, schema
       flexibility), so MongoDB could make sense here. Want me to draft
       a comparison?
```

## Commands

### Compile a Knowledge Base

When the user asks you to "learn", "compile", "index", or "remember" files from a directory:

```bash
python {SKILL_DIR}/scripts/compile.py <input_directory> <output_file>
```

Options:
```bash
# Mask PII before compilation
python {SKILL_DIR}/scripts/compile.py ./data knowledge.aura --pii-mask

# Filter low-quality content
python {SKILL_DIR}/scripts/compile.py ./data knowledge.aura --min-quality 0.3
```

### Query the Knowledge Base

```bash
python {SKILL_DIR}/scripts/query.py knowledge.aura "search query here"
```

### Agent Memory

Write to memory tiers:
```bash
python {SKILL_DIR}/scripts/memory.py write pad "scratch note"
python {SKILL_DIR}/scripts/memory.py write fact "verified information"
python {SKILL_DIR}/scripts/memory.py write episodic "session event"
```

Search and manage memory:
```bash
python {SKILL_DIR}/scripts/memory.py query "search query"
python {SKILL_DIR}/scripts/memory.py list
python {SKILL_DIR}/scripts/memory.py usage
python {SKILL_DIR}/scripts/memory.py prune --before 2026-01-01
python {SKILL_DIR}/scripts/memory.py end-session
```

## Memory Tiers

| Tier | What It Stores | Lifecycle |
|------|---------------|-----------|
| **`/pad`** | Working notes, scratch space, in-progress thinking | Transient — cleared between sessions |
| **`/episodic`** | Session transcripts, decisions, conversation history | Auto-archived — retained for reference |
| **`/fact`** | Verified facts, user preferences, learned rules | Persistent — survives indefinitely |

## Supported File Types

Documents: PDF, DOCX, DOC, RTF, ODT, EPUB, TXT, HTML, PPTX, EML
Data: CSV, TSV, XLSX, XLS, Parquet, JSON, JSONL, YAML, TOML
Code: Python, JavaScript, TypeScript, Rust, Go, Java, C/C++, and 20+ more
Markup: Markdown (.md), reStructuredText, LaTeX

## External Endpoints

| URL | Data Sent |
|-----|-----------|
| None | None |

This skill makes **zero network requests**. All processing is local.

## Security & Privacy

- **No data leaves your machine.** All compilation and memory operations run locally.
- The `.aura` format uses `safetensors` (no pickle) — no arbitrary code execution risk.
- Memory files are stored locally at `~/.aura/memory/`.
- No environment variables or API keys are required.
- No telemetry, analytics, or usage reporting.

## Model Invocation Note

This skill is autonomously invoked by the agent as part of its normal operation. The agent decides when to compile documents and manage memory based on user requests. You can disable autonomous invocation in your OpenClaw settings.

## Trust Statement

By using this skill, **no data is sent to any external service**. All processing happens on your local machine. Only install this skill if you trust [Auralith Inc.](https://auralith.org) and have reviewed the source code at [GitHub](https://github.com/Auralith-Inc/aura-openclaw).

## Notes

- Memory uses a Two-Speed WAL: instant writes (~0.001s), background compilation to durable shards.
- For emphasis weighting and training features, see [OMNI Platform](https://omni.auralith.org).
