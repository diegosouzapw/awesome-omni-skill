---
name: codex-review
description: Two-pass adversarial review of design documents and implementation plans using OpenAI Codex CLI. Invokes Codex to review plans section-by-section (pass 1), then holistically (pass 2), feeding critique back for revision. Use when you have a design doc, architecture plan, or implementation plan that should be stress-tested before execution.
---

# Codex Adversarial Review

Use this skill to get an independent adversarial review of design documents and implementation plans by invoking OpenAI Codex CLI as a reviewer. The review happens in two passes:

1. **Pass 1 — Section Review**: Each major section is reviewed independently for local issues (correctness, completeness, feasibility, internal consistency)
2. **Pass 2 — Holistic Review**: The full document is reviewed for cross-cutting concerns (contradictions between sections, missing integration points, architectural gaps, unaddressed failure modes)

After each pass, Claude integrates the feedback and revises the plan before proceeding.

## Prerequisites

- Python 3.10+ (no extra packages required — stdlib only)
- `codex` CLI installed and authenticated (`npm i -g @openai/codex`)
- Authentication configured (ChatGPT auth or `OPENAI_API_KEY` / `CODEX_API_KEY` set)
- The plan/design document exists as a file (markdown preferred)

All scripts are Python for cross-platform compatibility (Windows, macOS, Linux).

## When to Use

- After drafting a design document or implementation plan
- Before executing a plan with Claude Code's superpowers/execute-plan
- When you want a second opinion on architecture decisions
- When a plan is complex enough that internal inconsistencies are likely

## Workflow

### Step 0: Preparation

Before invoking any review, ensure:
1. The plan document exists as a file. If the plan is only in conversation context, write it to a file first.
2. Identify the document path — all scripts expect this as input.
3. Check that `codex` is available: `which codex`

### Step 1: Section Extraction

Split the document into reviewable sections. Use the `scripts/extract-sections.sh` script:

```bash
python3 /path/to/codex-review/scripts/extract_sections.py <plan-file>
```

This creates a `.codex-review/sections/` directory with individual section files. Each file contains the section content plus minimal surrounding context (the document title and table of contents if present) so Codex can understand where the section fits.

### Step 2: Pass 1 — Section Reviews

For each section, invoke Codex with the section review prompt:

```bash
python3 /path/to/codex-review/scripts/review_section.py <section-file> [review-type]
```

Where `review-type` is one of:
- `architecture` (default) — focuses on structural soundness, component boundaries, data flow
- `implementation` — focuses on feasibility, ordering, dependencies, edge cases  
- `api` — focuses on interface contracts, backwards compatibility, error handling
- `data` — focuses on data models, migrations, consistency, performance

Each section review produces a structured feedback file in `.codex-review/feedback/pass1/`.

**Review the pass 1 feedback with the user.** Present a summary of findings per section, categorized by severity:
- 🔴 **Critical** — Blocking issues, correctness problems, missing requirements
- 🟡 **Warning** — Gaps, potential issues, unclear specifications
- 🔵 **Suggestion** — Improvements, alternatives worth considering

**Revise the plan** based on pass 1 feedback before proceeding to pass 2. This is important — pass 2 should review the *improved* plan, not the original.

### Step 3: Pass 2 — Holistic Review

After revisions, invoke the holistic review on the full (revised) document:

```bash
python3 /path/to/codex-review/scripts/review_holistic.py <plan-file> <pass1-feedback-dir>
```

This pass specifically looks for:
- Contradictions between sections
- Missing integration points or handoff gaps
- Unaddressed failure modes and error propagation
- Implicit assumptions that aren't documented
- Ordering and dependency issues across sections
- Security and operational concerns

The holistic review also receives the pass 1 feedback so it can verify that earlier issues were actually addressed.

Output goes to `.codex-review/feedback/pass2/holistic-review.md`.

### Step 4: Final Integration

Present the pass 2 findings to the user. Apply final revisions. The complete review trail is preserved in `.codex-review/` for reference.

## Configuration

The skill respects these environment variables:

| Variable | Default | Description |
|----------|---------|-------------|
| `CODEX_REVIEW_MODEL` | (codex default) | Override the Codex model for reviews |
| `CODEX_REVIEW_TIMEOUT` | `120` | Timeout in seconds per review invocation |
| `CODEX_REVIEW_VERBOSE` | `0` | Set to `1` to show Codex stderr output |

## Tips

- For very large documents (>3000 words), section review is essential — Codex gives better feedback on focused chunks
- The `implementation` review type is best for plans that will be fed to execute-plan
- You can re-run just pass 2 after manual edits without redoing pass 1
- If Codex flags something you disagree with, note it as `[ACKNOWLEDGED]` — the holistic pass will see this and won't re-flag it
- Keep the `.codex-review/` directory around — it's useful for understanding why decisions were made later

## Iterating on Subsections

If pass 1 reveals major issues in a specific section, use the iteration script to do focused multi-round review using Codex session resume:

```bash
python3 /path/to/codex-review/scripts/iterate_section.py <section-file> <revised-section-file> [review-type] [max-rounds]
```

**How it works:**

1. Round 1: Codex reviews the original section (creates a session)
2. Claude (or you) revises the section based on feedback
3. Round 2+: The script resumes the **same Codex session** via `codex exec resume --last`, passing the revised content. Codex evaluates whether its previous concerns were addressed, marks findings as RESOLVED or UNRESOLVED, and flags any new issues introduced by the revision.
4. Repeats until Codex responds with "✅ SECTION APPROVED" or `max-rounds` (default 3) is reached.

**Interactive mode (default):** The script pauses between rounds and waits for ENTER after editing the revised file. Press `q` to stop early.

**Non-interactive mode (`--no-interactive`):** Runs all rounds without pausing. Useful when Claude Code manages the edit-review loop externally.

**Single-round mode (`--round N`):** Runs only round N. This is the best option when Claude Code drives the loop — it can run round 1, read feedback, revise the file, then run round 2, etc.

Example Claude Code workflow:
```bash
# Round 1
python3 scripts/iterate_section.py sections/03-data-model.md revised.md data --round 1
# Claude reads feedback, revises revised.md
# Round 2
python3 scripts/iterate_section.py sections/03-data-model.md revised.md data --round 2
```

**Convergence detection:** The script tracks issue counts across rounds. If issues stop decreasing after 3+ rounds, it warns that manual review may be needed.

**Fallback:** If `codex exec resume --last` fails (e.g., session expired), the script falls back to a fresh `codex exec` with the revision prompt. This loses conversational context but still gets a review.

All iteration feedback is preserved in `.codex-review/feedback/iterations/<section-name>/` with a summary file.

### Recommended Workflow

For plans with mixed quality across sections:

1. Run pass 1 on all sections
2. Triage: identify sections with 🔴 critical findings
3. Use `iterate-section.sh` on critical sections until approved
4. Re-run pass 1 on remaining 🟡 warning sections if needed
5. Only then proceed to pass 2 holistic review

This avoids burning a holistic review on a document with known local problems.
