---
name: agents-md
description: 'Refactor or create AGENTS.md using progressive disclosure: keep root
  minimal, split detailed instructions into linked docs, and flag contradictions/redundancy.
  Use when the user asks to create, update, or refactor AGENTS.md.'
knowledge_graph_profile: references/task-profile.json
---

# Agents Md

## Remember
Use judgment, adapt to context, and stay within explicit safety and scope constraints.

## Compliance
- Check against GOLD Industry Standards guide in ~/.codex/instructions/standards.md
- Use ~/.codex/AGENTS.md as the global index that links to current standards docs.
- If this skill is used from `~/.codex/`, write reference links only under `~/.codex/instructions/` (and not to `docs/` for global protocols).
- Visual reference: [agents-md workflow map](assets/agents-md.png)

## Philosophy

Prefer concise, verifiable instructions over comprehensive prose. Every command and path must be real and sourced from the repo. Treat AGENTS.md as an operator checklist: short, direct, and actionable. Use progressive disclosure: keep root minimal, link out for details.

Guiding principles:
- Optimize for reader success in under 2 minutes.
- Favor deterministic steps over narrative.
- Keep scope tight; expand only when the repo requires it.
- Default to canonical implementations for unreleased/greenfield projects; do not add backwards-compatibility layers unless explicitly required.

Mandatory snippet (include verbatim in guidance):
1. Explore project first, then invoke skill.
2. IMPORTANT: Prefer retrieval-led reasoning over pre-training-led reasoning for any React, Tauri, Apps-SDK-ui, Tailwind, Vite, Storybook + Chat Widget tasks.
3. Add a Table of Contents for docs.

## Core Engineering Principles

1. **Clarity over cleverness** — Prefer straightforward, maintainable instructions over elegant or complex phrasing.
2. **Explicit over implicit** — State assumptions, decisions, and side effects directly.
3. **Composition over inheritance** — Prefer small, reusable instruction units over monolithic guidance blocks.
4. **Fail fast, fail loud** — Surface validation failures at the point they occur and stop safely.
5. **Delete before adding** — Remove redundant, outdated, or duplicate instructions before writing new ones.
6. **Verify, don’t assume** — Confirm commands and paths from source files before presenting them.
7. **Stop before you are asked** — Do not expand scope beyond the request without explicit consent.

## Token Efficiency

- Prefer short, one-shot edits; avoid unnecessary tool calls or repeated edits for the same intent.
- Do not re-read files that were just written when the result is known and unchanged.
- Do not rerun validation commands unless behavior or assumptions changed.
- Summarize decisions, not large chunks of raw file contents.
- Batch related changes into one cohesive update.
- Avoid confirmation loops like “I will continue...” unless branching is required.
- Ask once for missing scope questions, then act.

## Scope and triggers

- The user asks to create or update AGENTS.md.
- The user asks to refactor AGENTS.md for progressive disclosure or split instructions into multiple files.
- The repo needs a short contributor guide for agents or humans.
- The user requests “Repository Guidelines” content under 400 words.
- The repo already has `AGENTS.md`, `CLAUDE.md`, and `GEMINI.md`, and the user asks to keep shared instruction guidance synchronized across them.

## Response format (required)
- Always include all three sections in every response:
  - `## When to use` explaining the trigger or noting "in scope".
  - `## Outputs` describing delivered artifacts.
  - `## Inputs` listing missing info or noting "none".
- Use the exact heading text and casing shown above.
- For out-of-scope requests, start with `## When to use` and still include `## Outputs` and `## Inputs` below.
- Do not omit `## When to use` under any circumstance.
- For out-of-scope requests, do not write any text before `## When to use`.

## Cognitive Support / Plain-Language
- Optimize for low cognitive load (TBI support): one task at a time, explicit steps.
- Use plain language first; define jargon in parentheses.
- Keep steps short and checklist-driven where possible.
- Externalize state: decisions, assumptions, and the next step.
- Provide ELI5 explanations for non-trivial logic.
- Ask one question at a time; prefer multiple-choice when possible.

### Response template (minimum)

```md
## When to use
- in scope

## Outputs
- ...

## Inputs
- ...
```

### Failure-mode template (out of scope)

```md
## When to use
- This skill applies when the user asks to create or refactor AGENTS.md using progressive disclosure.

## Outputs
- None (out of scope).

## Inputs
- None (out of scope).
```

Use the failure-mode template verbatim for out-of-scope requests.

## Required inputs

- Target repo root path.
- Existing AGENTS.md content (if present).
- Verified commands and paths from the repo (README, docs, config files).
- Package-manager signals from repo facts (`package.json#packageManager`, lockfiles, and existing command style in README/CI/docs).
- Any adjacent instruction files that may conflict (global or per-directory).
- Whether Jamie's agent-first scaffold standard is requested (`~/.codex/instructions/agent-first-scaffold-spec.md`).
- Compatibility posture (default: canonical-only for unreleased/greenfield repos; replace only when explicitly requested).


## Local Memory usage

- Follow `instructions/local-memory.md` for memory read/write workflow, tagging, and safety rules.
- Rule: search memory before writing new memory; store durable facts only.
- Never store secrets in memory.

## local-memory-mcp policy

- Use `local-memory-mcp` for durable context and cross-run continuity.
- Mandatory workflow:
  - `bootstrap(mode="minimal", include_questions=true, session_id="repo:<name>:task:<id>")`
  - `search(query="...", session_id="repo:<name>:task:<id>")`
- Record durable facts only with `observe(...)` (level `observation|learning`) and stable tags.
- Do not store secrets, tokens, keys, or PII.

## Deliverables

- A minimal root `AGENTS.md` that links to separate instruction files.
- One file per instruction category under the repo's canonical instruction root (e.g., `docs/agents/...` or `instructions/agents/...`).
- A suggested `docs/` folder structure.
- A table of contents for docs that are created or updated.
- A contradictions list with a question for each conflict.
- A “flag for deletion” list (redundant, vague, overly obvious).
- A detected package-manager command map (`install`, `run`, optional `exec`) derived from repo evidence and reused across generated docs.
- When requested: idempotent scaffold blocks for `AGENTS.md`, `.agent/PLANS.md`, and `README.md`.
- Output contract schema_version: 1

## Constraints
- Redact secrets/PII by default.
- Do not invent commands, scripts, or paths.
- Redact secrets and sensitive data by default.
- Use ASCII only unless the repo already uses non-ASCII.
- Do not add dependencies or tools.
- Do not add legacy shims, adapter layers, dual-write paths, or backwards-compatibility promises unless the user explicitly requires compatibility.
- Do not hardcode npm/pnpm/yarn/bun command examples without repo evidence.

## Workflow

1) Discover repo facts
- Read README and `docs/` for real commands and structure.
- Inspect config files (for example `pyproject.toml`, package scripts).
- Determine canonical instruction root before generating files:
  - If target path is `~/.codex` or a Codex config repo clone (for example `/Users/<user>/dev/config/codex`), prefer `instructions/` and do not create `docs/` for agent guidance.
  - Else, follow existing repo convention (`docs/agents` vs `instructions/agents`) and avoid introducing a second instruction tree.
- Detect package manager in this precedence: `package.json#packageManager` -> lockfiles (`pnpm-lock.yaml`, `yarn.lock`, `bun.lockb`/`bun.lock`, `package-lock.json`, `npm-shrinkwrap.json`) -> existing command style in README/CI/docs.
- If package-manager signals conflict or are missing, state "not observed" and ask which command style should be used before emitting manager-specific commands.
- Build one package-manager command map from detected evidence and apply it consistently in generated AGENTS/CLAUDE/GEMINI updates.
- If commit conventions are not visible, state “not observed.”
- Read global instructions from `~/.codex/AGENTS.md`.
- Also check `~/.codex/instructions/` for applicable global standards and guidance.
- Then read project instructions from repo root down to the working directory and treat them as canonical.
- Note: Codex `AGENTS.md` does not support `@` imports; Claude `CLAUDE.md` and `~/.claude/rules/*.md` do.
- Canonical hierarchy rule: when both `AGENTS.md` and `CLAUDE.md`/`GEMINI.md` exist, treat `AGENTS.md` as canonical source of truth for repository-wide, cross-tool instructions. Keep CLAUDE/GEMINI focused on agent/CLI-specific usage and memory conventions.
- If a repo already has `AGENTS.md`, `CLAUDE.md`, and `GEMINI.md`, update shared sections in `CLAUDE.md` and `GEMINI.md` to reference the canonical `AGENTS.md` guidance rather than duplicate it.
- If existing `AGENTS.md`, `.agent/PLANS.md`, `README.md`, or required `docs/` directories already exist, merge instead of overwrite.
- Apply **idempotent updates**: preserve existing content, insert only missing sections/links, and dedupe existing lines.
- Create directories/files only if absent; do not recreate duplicates.

2) Find contradictions
- Identify conflicting instructions and ask which one should win.
- Do not resolve conflicts without user confirmation.

2.1) Set compatibility posture
- Default to canonical-only guidance for unreleased/greenfield projects.
- Only include backwards-compatibility instructions when explicitly requested or when the repo shows clear released-version compatibility commitments.

3) Identify the essentials (root AGENTS.md)
- One-sentence project description.
- Repo-native package manager command style (install/run/exec, as observed).
- Non-standard build/typecheck commands.
- Anything truly relevant to every single task.

4) Add inserts (global references)
- If a canonical global protocol exists (for example `~/.codex/instructions/rvcp-common.md`), add a short "References" or "Imports" section at the top of the root `AGENTS.md` that points to it.
- Never duplicate the full protocol content in repo files; link only.
- If `CODEX_HOME` is set, prefer `$CODEX_HOME/...` for global references; otherwise use `~/.codex/...`.
- Only insert references that exist on disk; if not found, state "not observed" and do not invent paths.
- In `~/.codex/` context, do not write global protocol references to `docs/` paths. Always use `~/.codex/instructions/...` for protocol links.
- If the repo uses a different global protocol, add the same style of reference block.
- When scaffold mode is requested, include references to the scaffold spec and governance docs listed above.
  - Example (root `AGENTS.md` block):
    ```md
    ## References (informational)
    - Global protocol: ~/.codex/instructions/rvcp-common.md
    - Security and standards baseline: ~/.codex/instructions/standards.md
    ```

5) Group the rest
- Organize remaining instructions into logical categories (TypeScript, testing, deployment, accessibility, etc.).
- Keep each category file focused and scoped.

6) Create the file structure
- Output a minimal root `AGENTS.md` with Markdown links to category files under the canonical instruction root selected in step 1.
- Output each category file with its relevant instructions.
- Provide a suggested folder structure rooted at the selected instruction root.

7) Flag for deletion
- Identify redundant, vague, or overly obvious instructions.

8) Validate content
- Confirm commands exist and are runnable.
- Confirm naming conventions match the codebase.
- Ensure no secrets or private endpoints appear.
- For scaffold mode: verify marker blocks are present and not duplicated.
- For scaffold mode: run `python3 ~/.codex/scripts/plan-graph-lint.py <repo>/.agent/PLANS.md`.
- For scaffold mode: run link-integrity checks with `rg -n` for required global references.

## Required sections (root AGENTS.md)

- One-sentence project description
- Tooling essentials (repo-native package-manager commands, including install/run equivalents)
- Non-standard build/typecheck commands
- Code quality standards (inject when requested or supported by repo evidence)
- Planning guidance (inject plan-review rules for complex implementation work)
- Shell script conventions (inject when shell scripts/wrappers are present)
- References or imports (global protocol pointers; no duplication)
- Global instructions discovery order (brief, link to full doc)
- Links to category files
  - For Codex home/config repos: prefer `instructions/agents/...` links over `docs/agents/...`.

## Flaky Test Artifact Capture (injectable block, conditional)

Use this block when the repo has automated tests (Node, Python, Rust, Playwright, Vitest, Jest, or pytest evidence).

Decision rule:
1) If the user asks for flaky-test detection/artifacts/history, always inject this block.
2) Else, inject when repo facts show test commands/config (`test` scripts, `pytest`, `vitest`, `playwright`, `jest`, `Cargo test`, or `tests/` directory).
3) If no test evidence exists, do not inject.

When injected, include these concrete requirements:
- Required script path: `scripts/test-with-artifacts.sh`
- Required modes: `all`, `unit`, `integration`, `e2e`
- Required output root: `artifacts/test`
- Required stable outputs:
  - `artifacts/test/summary-*.json`
  - `artifacts/test/test-output-*.log`
  - `artifacts/test/junit-*.xml` (when runner supports it)
  - `artifacts/test/*-results.json` (when runner supports it)
  - `artifacts/test/artifact-manifest.json`
- If `package.json` exists, wire scripts:
  - `test:artifacts`, `test:artifacts:unit`, `test:artifacts:integration`, `test:artifacts:e2e`
- Use the detected package-manager command map for script invocations; do not mix manager variants in one generated block.

Insert this exact section in generated AGENTS.md for test repos:

```md
## Flaky Test Artifact Capture
- Run `bash scripts/test-with-artifacts.sh all` (or the detected repo-native command for `test:artifacts`) to emit machine-readable flaky evidence under `artifacts/test`.
- Optional targeted modes:
  - `bash scripts/test-with-artifacts.sh unit`
  - `bash scripts/test-with-artifacts.sh integration`
  - `bash scripts/test-with-artifacts.sh e2e`
- Commit/retain stable artifact paths for local automation ingestion:
  - `artifacts/test/summary-*.json`
  - `artifacts/test/test-output-*.log`
  - `artifacts/test/junit-*.xml` (when supported by test runner)
  - `artifacts/test/*-results.json` (when supported by test runner)
  - `artifacts/test/artifact-manifest.json`
- Keep artifact filenames stable (no timestamps in filenames) so recurring flake scans can compare runs.
```

## Code Quality Standards (injectable block, conditional)

Decision rule:
1) Inject when the user explicitly asks for code-quality or testing standards.
2) Else, inject when repo evidence shows automated tests, linting, or typechecking workflows.
3) If no evidence exists, do not inject.

Insert this section when injected:

```md
## Code Quality Standards
- Run full test suite before committing: `npm test` or the detected repo-native equivalent.
- Fix TypeScript errors and lint issues before marking tasks complete.
- Ensure test isolation - tests should not depend on execution order.
```

## Plan Review Guidelines (injectable block, conditional)

Decision rule:
1) Inject when the user asks for planning guardrails.
2) Else, inject when the task includes complex features, refactors, or architecture changes.
3) Skip for trivial edits unless explicitly requested.

Insert this section when injected:

```md
## Planning
### Plan Review Guidelines
- Before implementing complex features, create a minimal v1 scope.
- Avoid over-engineering - prefer simple solutions over comprehensive ones.
- Scale back ambition if initial plan feels too large.
```

## Shell Script Conventions (injectable block, conditional)

Decision rule:
1) Inject when the user asks for shell/script quality guidance.
2) Else, inject when repo evidence includes shell scripts, wrapper scripts, or script-based automation.
3) If no shell-script evidence exists, do not inject.

Insert this section when injected:

```md
## Shell Script Conventions
- Always validate wrapper scripts with shellcheck before considering complete.
- Test script syntax with `bash -n script.sh` to catch errors early.
- Handle edge cases for function conflicts and environment variable loading.
```

## Frontend Website Rules (injectable block, conditional)

Use this only when there is evidence the repo/task is frontend website work.

Decision rule (to let the skill "figure it out"):
1) If the user explicitly requests UI/frontend implementation, visual parity, screenshot, component polish, or reference-based frontend behavior, treat as frontend.
2) Else, infer from repo facts:
   - `package.json` includes React/Next/Vite/Tauri/Vue/Svelte frameworks or UI tooling.
   - `index.html`, `vite.config.*`, `next.config.*`, or `src/main.*` exists near `src/`.
   - A `brand/` folder is relevant and used for style assets.
3) If no evidence, do not inject this block; output a short note in `## Outputs` asking for clarification.

If frontend evidence is present, inject the block into the generated `AGENTS.md` (or category docs) so behavior is concrete and repeatable:

### AGENTS.md — Frontend Website Rules

- **Always Do First**: invoke `$ui-ux-creative-coding` and `$interface-craft` before writing any frontend code.
- If a reference image is provided:
  - Match layout, spacing, typography, and color exactly.
  - Use placeholders (`https://placehold.co/`) only when content is missing.
  - Do not improve or add to the design beyond the reference.
- If no reference image: design from scratch with the guardrails in this section.
- **Local server required**: Always serve from `http://localhost:2000` using the project’s `node serve.mjs`.
  - Never use `file:///`.
  - Start `node serve.mjs` in background before screenshots.
  - If already running, reuse that instance.
- Replace Puppeteer-specific assumptions with **agent-browser**:
  - Use `agent-browser` for navigation and screenshot capture.
  - Keep workflow tool-first: `agent-browser open http://localhost:2000` then capture screenshots.
- Pair with `$agentation` (or `agentation` MCP invocation) for execution orchestration:
  - session setup, screenshot loop control, and comparison pass tracking.
- **Screenshot naming convention**:
  - If the screenshot is a full page: `screenshot-page-<name>-<pass>.png`
  - If the screenshot is a component: `screenshot-component-<type>-<state>-<pass>.png`
    - examples: `screenshot-component-card-default-1.png`, `screenshot-component-button-hover-2.png`
  - Never overwrite; increment pass number when rerunning comparisons.
- Compare against the reference after each pass and continue at least 2 rounds until no visible mismatch.
- Output should use inline styles in a single `index.html` and Tailwind via `https://cdn.tailwindcss.com`.
- Tailwind classes:
  - Do not use default indigo/blue primaries.
  - Do not use `transition-all`.
  - For clickables, define hover, focus-visible, and active states.
- If `brand/` assets exist, prefer them over placeholders.
- Use existing brand variables; do not invent colors, spacing tokens, or typography scales.

### Runtime checks for screenshot workflows

- Run at least 2 screenshot rounds for visual parity.
- Use consistent comparison criteria: spacing/padding, typography scale, color values, alignment, border radii, shadows, sizing.
- For component screenshots, include the component type in filename (`card`, `button`, `modal`, `form`, etc.) to keep review context explicit.

## Agent-first scaffold integration (Jamie standard)

Apply this when the user asks for agent-first rollout/scaffold across repos (especially under `~/dev`).

Required global references (verify they exist before insertion):
- `~/.codex/instructions/openai-agent-workflow-playbook.md`
- `~/.codex/instructions/README.checklist.md`
- `~/.codex/instructions/validator-contracts.md`
- `~/.codex/instructions/strict-toggle-governance.md`
- `~/.codex/instructions/agent-first-scaffold-spec.md`

Use idempotent marker blocks:
- `AGENTS.md`: `<!-- AGENT-FIRST-SCAFFOLD:START --> ... <!-- AGENT-FIRST-SCAFFOLD:END -->`
- `.agent/PLANS.md`: `<!-- AGENT-FIRST-PLANS:START --> ... <!-- AGENT-FIRST-PLANS:END -->`
- `README.md`: `<!-- AGENT-FIRST-WORKFLOW:START --> ... <!-- AGENT-FIRST-WORKFLOW:END -->`

`.agent/PLANS.md` contract requirements:
- `tasks[]`, each task has `id`, `title`, `depends_on`
- `id` format `^T[1-9][0-9]*$`
- IDs unique within plan
- `depends_on` references in-plan IDs only
- no self-dependency; DAG required; single connected component
- validation command: `python3 ~/.codex/scripts/plan-graph-lint.py <plan-file>`

Canonical verification command:
- `bash ~/.codex/scripts/verify-work.sh`

Rollout policy:
- Link to 3-gate warn->block model in `~/.codex/instructions/agent-first-scaffold-spec.md`.

## Variation

- Vary examples and commands to match the target repo’s stack (Python vs Node).
- Use repo-specific paths and filenames; avoid repeating generic defaults across repos.

## Empowerment

- Offer two to three clear next-step options after drafting (accept, revise, or add missing info).
- Call out unknowns explicitly and ask for confirmation before finalizing.
- Encourage the user to prioritize sections when the scope is broad.
- Empower the user to choose between a minimal or detailed guideline set.
- Empower the user with explicit **choice and control** over scope, depth, and inserts before expanding.
- Explicitly empower the user to defer optional inserts until core guidance is approved.
- Ask whether to proceed with inserts when the global protocol is detected but optional.
- Provide a one-sentence rationale for each recommended insert or deletion.

## Validation

- Fail fast: stop at the first failed validation gate, fix it, then re-run.
- Run `~/.venvs/pyyaml/bin/python scripts/quick_validate.py <skill>` if available.
- Run `~/.venvs/pyyaml/bin/python scripts/skill_gate.py <skill>` and fix any missing sections.
- If needed, consult `references/contract.yaml` and `references/evals.yaml`.
- If validation scripts or paths are missing, state "not run (tooling not available)" and continue.

## Anti-patterns

- Generic boilerplate that ignores repo specifics.
- Fabricated commands or paths.
- Omitting contradictions or failing to ask which instruction wins.
- Burying risks or assumptions in long prose.
- Using vague headings like “Misc” or “Notes.”
- Presenting unverified commands as facts.
- Mixing unrelated policies into the same section.
- Adding global protocol content directly into repo `AGENTS.md` instead of linking.
- Stating paths that do not exist under the current `$CODEX_HOME` or repo.
- Treating imports as supported in Codex `AGENTS.md` (they are not).
- Hiding conflicts in linked docs instead of calling them out in the root file.
- Expanding root `AGENTS.md` beyond 400 words without explicit user approval.
- Creating a new `docs/agents` tree in Codex home/config repos where `instructions/` is canonical.
- Skipping project exploration before applying or invoking the skill.
- Adding a Table of Contents that does not match actual document headings.
- Never proceed with contradictory instructions without asking which one wins.
- Do not introduce new sections without confirming they are required for every task.
- Avoid “one‑size‑fits‑all” templates that erase repo‑specific commands.
- In scaffold mode, writing non-idempotent edits without marker blocks.
- Omitting `~/.codex/instructions/agent-first-scaffold-spec.md` when Jamie standard is requested.
- Overwriting existing instruction files/directories instead of performing scoped, deduplicated inserts.
- Adding backwards-compatibility requirements by default in unreleased/greenfield projects.
- Generating extra legacy-preservation code paths without an explicit compatibility requirement.
- Mixing npm/pnpm/yarn/bun command examples in one output block or defaulting to npm without repository evidence.

## Example prompts that should trigger this skill

- "Draft an AGENTS.md for this repo."
- "Create a Repository Guidelines AGENTS.md under 400 words."
- "Standardize our AGENTS.md using actual repo commands."
## Procedure
1) Clarify scope and inputs.
2) Execute the core workflow.
3) Summarize outputs and next steps.
