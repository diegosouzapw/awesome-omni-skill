---
name: zeno
description: Evidence-first, read-only Zeno workflow for huge corpora via an external JSONL REPL server (list_files/read_file/peek/grep/extract_symbols). Minimize context bloat, prevent drift, and produce a cited architecture + audit report.
---

# Zeno -- Read-only, Evidence-first

## Core idea (1 paragraph)
The corpus (repo/docs/logs) is too large to paste into the model. Treat it as external environment state accessed through a JSONL REPL server. Retrieve only the minimum evidence needed (paths + small line ranges), optionally "recursing" by producing per-slice capsules, then consolidate into a report.

## How this skill is triggered (and when to use it)
Zeno activates when the user explicitly asks for Zeno or names a mode. Common triggers:
- "Use Zeno codebase-archaeology to trace <symbol> across the repo."
- "Run Zeno security-audit on src/ and build evidence chains."
- "Use Zeno architecture-mapping to document entrypoints and lifecycle."
- "Zeno pr-review these changed files: ... with downstream impact."
- "Zeno skill-generation on this repo and draft SKILL.md."
- "Zeno deep-research on this project question: ..."

Full use cases and mode details are defined in `references/modes.md`.

## Non-negotiables (hard guardrails)
- READ-ONLY: never modify files. Never propose "just change X" unless user explicitly requests edits later.
- No evidence, no claim: every non-trivial statement must cite evidence as path:Lx-Ly or a grep hit.
- Small bites: prefer peek + targeted read_file slices over whole-file reads.
- Be deterministic: when unsure, retrieve more evidence; do not guess.

## Tools available (only these ops)
- list_files(args: {glob|regex, max, include_hidden, exclude_dirs, exclude_globs, max_files})
- peek(args: {path, head_lines, tail_lines})
- read_file(args: {path, start_line, end_line, max_lines})
- grep(args: {pattern, paths?, max_hits, context, include_hidden, exclude_dirs, exclude_globs, max_files, max_bytes})
- extract_symbols(args: {path, max_symbols})
- stat(args: {path?, paths?})

## Budgets (to prevent tail-cost blowups)
You MUST enforce these caps unless user explicitly requests deeper coverage:
- Retrieval ops: <= 30 total per answer, <= 12 per section
- read_file: <= 400 lines per call, <= 2000 lines total returned across the whole answer
- grep: <= 200 hits, always narrow with paths or a more specific pattern if truncated
- Recursion (capsules): <= 12 file capsules, depth <= 2 (capsule -> sub-capsule), then consolidate
- If you hit any budget: STOP, summarize, and output Next Retrieval Plan (max 10 ops)

## Default excludes (reduce noise)
Unless user requests otherwise, pass these as excludes when listing/grepping:
- exclude_dirs: [".git",".hg",".svn",".venv","venv","node_modules","dist","build",".next",".turbo",".cache",".pytest_cache"]
- exclude_globs: ["**/*.min.*","**/*.map","**/generated/**","**/vendor/**"]

---

# Mode profiles (choose one when asked)
Zeno supports six explicit modes. Pick the closest mode when the user asks for one of these outcomes. If unsure, ask a brief clarifying question and default to architecture-mapping.

Mode quick selector:
- codebase-archaeology: trace a symbol across a monorepo with file:line citations.
- security-audit: pattern-based vulnerability triage with evidence chains.
- architecture-mapping: lifecycle and boundary mapping without losing context.
- pr-review: full-context review for large diffs and ripple effects.
- skill-generation: draft SKILL.md from a new tool's codebase with citations.
- deep-research: evidence-first research report without hallucination.

Mode selection workflow:
1) Confirm the mode and required inputs (symbol name, changed_files, or research question).
2) Announce budgets and scope explicitly.
3) Use the default retrieval plan for the chosen mode (see references/modes.md).
4) Produce the standard Output schema plus mode-specific add-ons below.

If the user asks for multiple modes, run them sequentially and keep evidence ledgers separate per mode.

---

# Operating procedure (the playbook)

## Phase 0 -- Frame the task (always do this)
1) Restate the user's goal in 1-2 lines.
2) Declare budgets you'll enforce.
3) Declare what "done" looks like using the Output Schema below.

## Phase 1 -- Repo map (fast scan)
Goal: build a working mental index without reading everything.
1) list_files (glob) for key file types (language + configs).
2) Identify likely entrypoints + orchestration files:
   - build manifests (Package.swift, package.json, pyproject.toml, Cargo.toml, etc.)
   - app bootstrap (main.*, App*, AppDelegate/SceneDelegate, CLI entry)
   - routing (router, routes, navigation)
   - dependency injection / wiring (container, module, registry)
3) peek the top candidates to confirm relevance.

## Phase 2 -- Entrypoints and lifecycle (evidence-backed)
Goal: answer "what runs first" and "how requests/events flow."
1) grep for entry symbols ("main", "App", "init", "run", "serve", "router", "register", "bootstrap").
2) read_file only the line ranges that define:
   - startup sequence
   - primary event loop / request loop
   - global configuration loading
3) Capture evidence excerpts for the lifecycle narrative.

## Phase 3 -- Wiring map (dependencies + boundaries)
Goal: map interactions without pretending you read the whole repo.
1) For each core module/file:
   - extract_symbols
   - grep for imports / references to/from it
   - read_file only the minimal blocks that show how it is called and what it calls
2) Produce a wiring edge list: A -> B (why) with evidence.

## Phase 4 -- Cross-cutting audits (security / concurrency / dead code)
Goal: detect "unpriced risk" areas with evidence.
Use focused greps and short reads:
- Security: auth checks, token parsing, crypto usage, secrets, unsafe eval, injection risks
- Concurrency: shared mutable state, locks, async boundaries, cancellation, reentrancy
- Dead/orphan: unused symbols, duplicate modules, old feature flags, unused routes
- Layering: cycles, "UI calls DB directly", config scattered, global singletons

If you cannot prove something, move it to Open Questions.

## Phase 5 -- Consolidate and deliver
Stop retrieving when:
- you have entrypoints + lifecycle + top-level module map
- additional retrieval mostly repeats patterns
- budgets are close to limit
Then produce the report with strict schema + claim-evidence ledger.

---

# Output schema (STRICT)
## 1) Executive Summary
- What this system is (1-2 lines)
- Primary entrypoints
- 3-8 key architectural facts (each with evidence IDs)

## 2) System Map (Lifecycle Narrative)
- Startup sequence (ordered steps)
- Main runtime loop / routing
- Configuration and environment assumptions
(Each step must cite evidence.)

## 3) File Capsules (table)
Columns:
- File
- Role / responsibility
- Key symbols
- Calls into / calls out
- Risks / smells
- Evidence

## 4) Wiring Map (edges)
List A -> B edges with short explanation + evidence.

## 5) Risks and Smells (prioritized)
For each:
- Severity (High/Med/Low)
- What could go wrong
- Where seen (evidence)
- Suggested next evidence to confirm or refute

## 6) Evidence Ledger (required)
List evidence items only (no claims in this section). Each evidence entry must include the location and a one-line rationale for why it supports a claim.
Evidence items must be formatted:
- E1 = path/to/file.ext:L10-L44
- E1_why = One-line reason this excerpt supports a claim.
- E2 = grep(pattern=\"AuthMiddleware\", hit=src/security.py:L123)
- E2_why = One-line reason this grep hit supports a claim.
Optional: group evidence by file when it improves readability.

## 7) Claim Ledger (required)
Map claims to evidence IDs:
- C1: <claim> -> [E1, E3]
- C2: <claim> -> [E2]

## 8) Open Questions + Next Retrieval Plan
- Open questions (only those lacking proof)
- Next retrieval plan (max 10 ops, each fully specified)

---

## Mode-specific add-ons (required)
When a mode is active, include these extra sections in the report. These do not replace the standard Output schema.

### codebase-archaeology add-ons
- Definition capsule: file, signature, and minimal excerpt with evidence IDs.
- Usage table: path, line, short context, evidence IDs.
- Coverage statement: files scanned, grep hits, read lines.

### security-audit add-ons
- Risk table: severity, pattern, file:line, evidence IDs.
- Evidence chains: Source -> Validation -> Sink (if visible), each step cited.
- Remediation questions: what to verify next, with retrieval plan items.

### architecture-mapping add-ons
- Component map: components, responsibilities, and boundaries with evidence.
- Lifecycle narrative: ordered steps with evidence IDs.
- Cross-cutting concerns: auth, storage, telemetry, config.

### pr-review add-ons
- Change summary: what changed and where, with evidence IDs.
- Impact map: changed file -> dependents with evidence.
- Test coverage notes: existing tests or gaps with evidence.

### skill-generation add-ons
- Draft SKILL.md (inline) with citations for key claims.
- Allowed tools policy and guardrails based on evidence.
- Open questions where docs are missing.

### deep-research add-ons
- Claim ledger with confidence and evidence IDs.
- Evidence snapshot list (files and sections read).
- Open questions and next retrieval plan.

---

## Mandatory end-of-turn blocks (Pattern A)
At the end of every response while this skill is active, append the three machine-parseable blocks below. These are consumed by `notify_persist.py` on `agent-turn-complete` to persist state, evidence, and claims.

Hard rules:
- The blocks must be valid JSON/JSONL with no commentary inside.
- The blocks must appear in the exact order below.
- Do not include trailing commas.

Required format:

```
===ZENO_STATE_UPDATE_JSON===
{ "thread_id":"T1", "turn_id":"U4", "mode":"read-only", "budgets":{"retrieval_ops_max":30,"retrieval_ops_used":12,"read_lines_max":2000,"read_lines_used":740,"grep_hits_max":200,"grep_hits_used":55}, "high_level_summary":"Mapped entrypoints and routing", "open_questions":["Where is auth middleware registered?"], "next_retrieval_plan":[{"op":"grep","args":{"pattern":"Auth","paths":["**/*.py"],"max_hits":50},"purpose":"Locate auth wiring"}] }
===/ZENO_STATE_UPDATE_JSON===

===ZENO_EVIDENCE_LEDGER_JSONL===
{"evidence_id":"E1","kind":"read","path":"src/app.py","lines":[10,44],"why":"Defines the FastAPI app construction","hash":"","timestamp":"2026-01-04T18:11:30Z"}
{"evidence_id":"E2","kind":"grep","pattern":"AuthMiddleware","hit":"src/security.py:L123","why":"Shows auth middleware registration","hash":"","timestamp":"2026-01-04T18:11:30Z"}
===/ZENO_EVIDENCE_LEDGER_JSONL===

===ZENO_CLAIM_LEDGER_JSONL===
{"claim_id":"C1","claim":"App starts in app.py with FastAPI","evidence":["E1"],"confidence":"high","timestamp":"2026-01-04T18:11:30Z"}
{"claim_id":"C2","claim":"Auth middleware registers in security.py","evidence":["E2"],"confidence":"med","timestamp":"2026-01-04T18:11:30Z"}
===/ZENO_CLAIM_LEDGER_JSONL===
```

If you are missing any field, leave it as an empty string or empty list rather than omitting the field.

---

## Pattern A persistence (notify checkpoints)
The notify hook writes state and ledgers after every turn. This is the authoritative checkpoint when telemetry is unavailable.
Follow the block format exactly so `notify_persist.py` can extract and persist data to `.codex/zeno/` under `state/`, `evidence/`, and `claims/`.

---

## Pattern B telemetry (OTEL)
If OpenTelemetry is enabled, tool-level events are exported to an OTEL collector for audit and replay.
Default privacy: `otel.log_user_prompt = false` unless the user explicitly asks to enable prompt logging.

---

# NON-NEGOTIABLE: Read the full manual
Before using Zeno in a real task, you MUST read the full manual and follow it. This SKILL file is intentionally dense; the references are the source of truth.

Read these in order:
- `references/modes.md` (mode specs and required add-ons)
- `references/operator_instructions_and_review.md` (operator-grade rules)
- `references/recipes.md` (copy/paste retrieval plans)
- `references/protocol.md` (JSONL REPL protocol)
- `references/security.md` and `references/otel.md`

---

# Anti-drift checklist (run before finalizing)
- Did I exceed budgets? If yes, did I stop and provide next plan?
- Does every non-trivial claim map to evidence?
- Did I accidentally assume behavior not supported by retrieved lines?
- Is the report complete per schema, even if partial coverage is noted?

---

# Improvements and extensions (do not skip if asked to extend the skill)

## Evidence registry (mechanical)
Maintain an evidence registry while working:
- E1, E2, E3, E4 each is a path + line range or a grep hit.
Then output a Claim Ledger (separate from the Evidence Ledger):
- C1 -> [E1, E4]
- C2 -> [E2]
If you cannot attach an E#, the statement must become an Open Question.

## Stop rules (prevent infinite exploring)
Stop retrieval and write the report when:
- you have entrypoints + lifecycle + top module map
- additional retrieval repeats the same patterns
- you are within 20 percent of any budget
Always include Open Questions + Next Retrieval Plan.

## Failure-mode playbooks
- grep truncated -> narrow by paths and specificity
- big file -> peek then targeted read_file slices
- uncertainty -> Open Questions + next 3-10 retrieval ops

## Coverage report (required when sampling)
Include explicit coverage numbers:
- files sampled
- greps executed
- lines read total
- list_files filters applied

## Subcall templates (for recursion)
File capsule:
- Purpose (1-2 sentences)
- Key types/functions
- Imports and dependencies
- Internal dependencies (callers/callees)
- Risks: security, concurrency, error handling
- TODO/dead code signals
- 3-8 evidence excerpts (line ranges)

Wiring map:
- Lifecycle order
- Runtime path narrative
- Dependency edges (A -> B)
- Cyclic risks

## Determinism and safety
- Never read outside --root.
- Sort paths and hits.
- Respect max_files and max_bytes to avoid huge scans.
- Prefer literal grep patterns unless regex is required.

## References
- references/protocol.md
- references/operator_instructions_and_review.md
- references/recipes.md
- references/data_model.md
- references/security.md
- references/otel.md
- references/troubleshooting.md

## Acknowledgments
This skill is inspired by and references:
- Zhang et al., "Recursive Language Models" (arXiv:2512.24601v1): https://arxiv.org/abs/2512.24601v1
- Alex Zhang's reference implementation: https://github.com/alexzhang13/rlm
- Original announcement thread: https://x.com/a1zhang/status/2007566581409144852?s=46

Thank you to Alex Zhang and collaborators for the Zeno concept and open resources that informed this work.
