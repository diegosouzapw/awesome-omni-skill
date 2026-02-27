---
name: maintain-docs
description: Periodic documentation maintenance audit. Finds orphaned docs, detects drift between .cursor/rules/ and docs/developer/, validates doc correctness against source code, tracks structural issues in a persistent backlog, and opens PRs to close highest-priority gaps per run. Use when the user asks to audit documentation, sync docs, or maintain the knowledge base.
---

# Documentation maintenance

A periodic skill that audits the documentation knowledge base, detects gaps and drift, validates correctness against the codebase, and opens PRs to incrementally close issues. Designed to be run repeatedly — each invocation checks previous PR outcomes to calibrate effort, uses a complexity budget to address the highest-priority findings, and tracks structural issues in a persistent backlog across runs.

For the architectural rationale behind this skill's design — including the materialized view concept, the distinction between code-coupled facts and human intent, and the prioritization philosophy — see [`intent.md`](intent.md).

## Hard constraints

These constraints are absolute and override any other instructions:

1. **NEVER modify source code files.** This skill may only create or edit markdown (`.md`, `.mdc`) and text files. No `.ts`, `.tsx`, `.js`, `.jsx`, `.json`, `.css`, or any other code files — ever. This applies to all sub-agents invoked by this skill.
2. **Before opening a PR, verify this constraint.** Run `git diff --name-only` and confirm every changed file ends in `.md` or `.mdc`. If any source file appears, abort and report the violation.
3. **Ignore `docs/design/`** entirely. Design documents are maintained by humans using the design-review skill. They are not documentation and are not in scope.
4. **Do not rewrite documentation from scratch.** This skill makes targeted, incremental edits. If a doc needs major restructuring, recommend it for separate work with a rationale — do not attempt it in this run.
5. **Run `npm run prettier` before committing.** The CI build chain requires formatted files.
6. **NEVER commit or push directly to main.** All changes must be on a feature branch submitted as a PR.

## Duplication governance principle

The documentation has two tiers with distinct roles:

- **`.cursor/rules/` files**: Compact, prescriptive agent constraints. What agents **must do** or **must not do** when working in a domain. These are loaded into agent context.
- **`docs/developer/` files**: Comprehensive reference documentation. How things work in detail, with examples and explanations. These are for deep dives.

When both tiers cover the same topic, the rule file should include a "For full reference, see `docs/developer/...`" link. Neither should duplicate the other's content verbatim. When drift is detected, the resolution is: check the codebase to determine which version is correct, update the incorrect version, then ensure the cross-reference link exists in both directions.

### Design rationale in docs

Some `docs/developer/` files contain design rationale — `## Key Design Decisions`, `### Why X?` subsections, or inline explanations of tradeoffs. This rationale is valuable context for agents making implementation decisions.

The maintain-docs skill treats design rationale as **opportunistic, not systematic**:

- When validating a doc for staleness, check whether existing rationale sections are still accurate against the code. Fix factual errors in rationale the same way you fix factual errors in any other section.
- When extracting rationale from code comments or design docs during a staleness fix, it is acceptable to add a brief `## Design notes` section summarizing what was found — but only when there is substantial, citable content to extract. Every claim must cite its source (e.g., `(from code comment in X)` or `(from design doc Y)`).
- **Never create stub sections with TODO placeholders.** A doc with no design rationale section is better than a doc with an empty template prompting humans to fill it in. Those stubs become permanent debt.
- **Never fabricate rationale.** If the code and design docs don't contain clear rationale signals, leave the doc as-is. The absence of documented rationale is information, not a gap to be filled with guesses.

## Maintenance backlog

The file `docs/_maintenance-backlog.md` is the skill's persistent memory across runs. It has three sections:

1. **Work items** — structural recommendations and deferred issues that cannot be resolved through incremental edits (e.g., "rule file is too long and should be split," "two docs should be merged").
2. **Validated docs** — a record of which docs were checked against their source code and found accurate, with the date of validation. This prevents the skill from re-checking docs that haven't meaningfully changed since last validation.
3. **Exclusions** — files that have been reviewed and confirmed as not needing an AGENTS.md entry. These are filtered out of orphan detection so the skill stops flagging them.

**Rules for the backlog file**:

- If the file does not exist, create it on the first run with the template below.
- Read it at the start of Phase 0 to incorporate carry-forward items, validated timestamps, and exclusions.
- Write to it at the end of Phase 3 — add new structural recommendations, remove resolved items, update validated timestamps, and add new exclusions.
- Each work item has a date, a brief description, and a rationale. Keep entries concise.
- Each validated entry has a doc path and a date. Update the date when a doc is re-validated.
- Each exclusion has a doc path and a brief reason why it does not need indexing.
- If a backlog work item has been present for 3+ runs without progress and is not blocked by external factors, consider closing it with a brief note explaining why it was dropped.

**Backlog file template**:

```markdown
# Documentation maintenance backlog

Persistent tracker for the maintain-docs skill's persistent state across runs.

## Work items

<!-- Structural issues requiring dedicated effort. Format: date, description, rationale. Remove when resolved. -->

## Validated docs

<!-- Docs checked against source and found accurate. Format: date, doc path. Update date on re-validation. -->

## Exclusions

<!-- Files confirmed as not needing an AGENTS.md entry. Format: path, reason. -->
```

## Workflow

### Phase 0: Feedback check

Goal: Determine whether previous skill runs produced value, and calibrate this run accordingly.

1. Read `docs/_maintenance-backlog.md` if it exists. Load work items, validated doc timestamps, and exclusions into memory for use in later phases.
2. Check the outcome of recent PRs from this skill:
   ```
   gh pr list --state all --label documentation --search "skill:maintain-docs" --limit 5 --json number,state,mergedAt,closedAt,title
   ```
3. Classify each recent PR:
   - **Merged**: The skill's work was accepted. Normal operation.
   - **Closed without merge**: The skill's work was rejected. This is a signal to reduce scope or change approach.
   - **Open for >7 days**: The PR is creating review burden. Likely too large or too marginal to prioritize.
4. Apply calibration rules:
   - If the **last 2+ PRs were closed without merge**, report the pattern to the user and **exit without making changes**. The skill's output is not aligned with team needs and requires human guidance.
   - If the **last PR is still open after 7+ days**, reduce the complexity budget for this run to **4 points** (from 7). Smaller PRs are easier to review.
   - If recent PRs were merged promptly, proceed normally.

### Phase 1: Lightweight audit

Goal: Build a prioritized list of documentation issues without reading every file in full.

#### Step 1: Parse the discovery graph

1. Read `AGENTS.md`
2. Extract all file references from:
   - The "On-demand context" table
   - Inline links and references throughout the file
   - The "PR reviews" section
   - Any other tables or lists that reference documentation files
3. This produces the **indexed set** — files reachable from the agent entry point

#### Step 2: Discover all documentation files

1. Glob `docs/**/*.md` excluding `docs/design/**`. Note: `docs/sources/` contains plugin documentation published to Grafana.com for external human readers. Index and maintain these like any other docs, but be aware their audience is end users, not agents — they are high-level and functionality-oriented.
2. Glob `.cursor/rules/*`
3. Glob `.cursor/skills/**/SKILL.md`
4. This produces the **full set**

#### Step 3: Classify findings

For each file in the full set, assign one status:

- **Indexed**: File is in the indexed set. Candidate for staleness check.
- **Orphaned**: File exists but has no path from AGENTS.md. Candidate for indexing.
- **Missing**: Referenced in AGENTS.md but file does not exist. Broken link, candidate for cleanup.

For each **orphaned** file:

- Check the **exclusions list** from the backlog. If the file is listed there, skip it — it has been reviewed and confirmed as not needing indexing.
- For files not excluded, read only the first ~30 lines to classify its task domain and estimate its value. Do NOT read full file contents in this phase — context budget matters.

For each **indexed** file:

- Check whether a corresponding file exists in the other tier (`.cursor/rules/` ↔ `docs/developer/`). If a pair exists, flag it as a drift check candidate.
- **Staleness check**: Infer which source directories the doc describes from its content path or AGENTS.md glob triggers (e.g., `docs/developer/engines/context-engine.md` describes `src/context-engine/`). Then apply a two-stage filter:
  1. **Validated recently?** Check the backlog's "Validated docs" section. If this doc was validated within the last 30 days, skip the staleness check entirely — it was recently confirmed accurate.
  2. **Structural changes?** For docs not recently validated, check the source directory for _structural_ changes since the doc's last modification: new or deleted files (`git diff --name-status --diff-filter=ADR` between the doc's last commit and HEAD), or renamed exports. Cosmetic changes (formatting, typo fixes, test-only changes) are not meaningful staleness signals. Use `git log --stat` or `git diff --stat` to gauge change magnitude. Only flag the doc as a staleness candidate if structural changes are detected.

#### Step 4: Score findings

| Priority   | Criteria                                                                                                                               |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| **HIGH**   | Operational doc with no discovery path that constrains a common agent task (releases, feature flags, interactive authoring, CLI tools) |
| **HIGH**   | `.cursor/rules/` ↔ `docs/developer/` pair with suspected drift (both cover the same topic)                                             |
| **HIGH**   | Missing referenced file — broken link in AGENTS.md                                                                                     |
| **HIGH**   | Indexed doc with structural source changes (new/deleted/renamed files) in a high-traffic domain                                        |
| **HIGH**   | Backlog work item older than 3 runs that has not been addressed                                                                        |
| **MEDIUM** | Indexed doc with structural source changes in a lower-traffic domain                                                                   |
| **MEDIUM** | Engine or subsystem doc that helps agents working on specific code areas (orphaned)                                                    |
| **MEDIUM** | Recent backlog work item (carried forward from last 1-2 runs)                                                                          |
| **LOW**    | Supplementary docs (known issues, scale testing) that agents rarely need (orphaned)                                                    |
| **LOW**    | README files for stable, rarely-changed areas (orphaned)                                                                               |

#### Steady-state behavior

If the audit finds no orphaned docs (after applying exclusions), no staleness candidates (after checking validated timestamps and filtering for structural changes), no drift pairs, and the backlog work items section is empty — **do nothing**. Report that everything is clean and exit without creating a branch or PR.

If the only actionable items are in the backlog (no new findings from the filesystem audit), use the complexity budget to burn down backlog work items. The backlog is not just a record — it is a work queue. Structural issues that were too large for previous runs should be attempted when they are the highest-priority remaining work.

### Branch setup (before making any changes)

Before making any edits, prepare the working branch:

1. Run `git checkout main && git pull origin main` to ensure you have the latest main.
2. Create and switch to a new branch: `git checkout -b docs/maintain-docs-YYYY-MM-DD-<2 unique chars>`

All Phase 2 edits happen on this branch. Never edit files while on main.

### Phase 2: Scoped fixes

Select findings from the top of the scored list, using a **complexity budget** rather than a flat item count:

| Fix type  | Cost     | Examples                                                                                                      |
| --------- | -------- | ------------------------------------------------------------------------------------------------------------- |
| **Light** | 1 point  | Add an index entry to AGENTS.md, add a cross-reference link, fix a stale file path, add an exclusion entry    |
| **Heavy** | 3 points | Drift correction between rule/doc pair, new rule file creation, staleness validation with factual corrections |

**Budget per run: 7 points** (reduced to 4 if Phase 0 detects review burden). This allows up to 7 light fixes, or 2 heavy fixes + 1 light fix, or similar combinations. When related docs share a domain (e.g., all files under `docs/developer/engines/`), group them as a single finding.

For each selected finding, delegate to a sub-agent with a tightly scoped task description.

**CRITICAL**: Every sub-agent invocation must include this instruction: "You may ONLY modify `.md` and `.mdc` files. No source code changes are permitted under any circumstances."

#### Fix type: Orphaned doc needs indexing

Sub-agent task:

1. Read the orphaned doc in full
2. **Decide whether indexing is appropriate.** Not every doc belongs in AGENTS.md. Component READMEs, local utility docs, and end-user-facing docs in `docs/sources/` may serve their purpose without agent indexing. If the doc does not constrain agent behavior or provide context agents need for implementation tasks, add it to the backlog's **Exclusions** section with a brief reason and move on. This is a **light fix** (1 point).
3. If indexing is appropriate, read any related `.cursor/rules/` file if one exists for the same domain
4. Validate key claims against the codebase:
   - Do referenced file paths still exist?
   - Do mentioned npm scripts exist in `package.json`?
   - Do described APIs, functions, or components exist in the source?
5. Fix factual errors found in the doc (stale file paths, renamed scripts, changed API names)
6. Propose the AGENTS.md table entry: file path, "when to load" description, and glob trigger if appropriate
7. If a `.cursor/rules/` counterpart exists for the same domain, add cross-reference links in both directions

#### Fix type: Rule/doc drift

Sub-agent task:

1. Read both the `.cursor/rules/` file and its `docs/developer/` counterpart
2. Identify claims that differ between them
3. For each difference, check the codebase to determine which version is correct
4. Update the incorrect version to match the code
5. Ensure the rule file contains a "For full reference, see `docs/developer/...`" pointer
6. Keep the rule file compact and prescriptive; keep the doc comprehensive and explanatory

#### Fix type: Staleness validation

For an indexed doc flagged as stale (structural source changes detected):

Sub-agent task:

1. Read the flagged doc in full
2. Identify the source directories it describes (from path conventions or AGENTS.md glob triggers)
3. Focus on the structural changes detected in Phase 1 (new/deleted/renamed files) and compare against claims in the doc:
   - File paths and directory structures mentioned
   - Function, class, and component names referenced
   - npm scripts and CLI commands documented
   - Configuration values, constants, and default behaviors described
   - Architecture descriptions and data flow claims
4. Fix all factual errors found (stale paths, renamed symbols, changed defaults, outdated descriptions)
5. If the doc also contains design rationale sections, verify those claims are still accurate against the code. Fix factual errors. If substantial rationale exists in code comments or design docs that isn't captured, it is acceptable to add a brief `## Design notes` section — but only with citable, real content (see "Design rationale in docs" above).
6. If corrections are extensive enough that the doc's overall structure no longer holds, do not attempt a rewrite — add it to the maintenance backlog as a work item instead
7. **Record validation**: After fixing or confirming the doc is accurate, update the backlog's "Validated docs" section with today's date and the doc path. This prevents the same doc from being re-checked next run unless new structural changes occur.

This is a **heavy fix** (3 points). Prioritize staleness validation for docs that constrain common agent tasks (feature flags, interactive authoring, engines) over docs for rarely-touched areas.

#### Fix type: New rule file needed

For a complex domain that has a `docs/developer/` reference but no `.cursor/rules/` constraint file:

1. Read the reference doc and the relevant source code to verify accuracy
2. Draft a compact `.cursor/rules/` file with prescriptive constraints only — not a copy of the reference doc
3. Include a "For full reference, see `docs/developer/...`" pointer
4. Add appropriate frontmatter (`alwaysApply: false`, `description`, and glob triggers if applicable)
5. Propose an AGENTS.md table entry for the new rule

#### Fix type: Structural recommendation only

If an issue is too large for incremental editing (e.g., a doc needs a complete rewrite, or multiple docs should be merged):

1. Do NOT attempt the restructuring
2. Add the item to `docs/_maintenance-backlog.md` with the current date and a brief rationale
3. Include this in the PR description under "Recommendations for separate work"

### Phase 2.5: Verification

After all sub-agents have completed their work, review the combined diff before proceeding to PR creation.

1. Run `git diff` and read the full output.
2. For each changed file, verify:
   - The change makes a factual claim → spot-check that claim against the code. If a sub-agent changed "function `foo`" to "function `bar`", confirm `bar` actually exists.
   - The change doesn't introduce contradictions with other parts of the same doc or with other changed files.
   - The change doesn't remove content that was correct (sub-agents may over-correct).
3. If a sub-agent's change looks wrong or dubious, **revert that file** (`git checkout -- <file>`) rather than trying to fix it. A skipped fix is better than an incorrect one.
4. If all sub-agent changes are reverted, exit cleanly without creating a PR. Report what was attempted and why it was reverted.

### Phase 3: PR creation

#### Update the maintenance backlog

1. Add any new work items discovered in this run to the "Work items" section with today's date.
2. Remove any work items that were resolved by this PR's changes.
3. Update the "Validated docs" section — add or update entries for docs that were validated in this run (even if no corrections were needed).
4. Add any new exclusions identified during orphan processing.
5. If the file does not exist yet, create it using the template from the "Maintenance backlog" section above.
6. Review the work items list: if any item has been present for 3+ consecutive runs (compare dates) and hasn't been attempted, either attempt it in this run (if budget allows) or add a note explaining why it's blocked.

#### Safety checks

1. Run `git diff --name-only` and verify **every** changed file ends in `.md` or `.mdc`. If any other file type appears, **stop immediately** and report the problem. Do not proceed.
2. Run `npm run prettier` to format all markdown files per CI rules.
3. Run `git diff --name-only` again after prettier. Prettier should only touch markdown files. If it modified anything else, abort and report.

#### Commit and PR

You should already be on the `docs/maintain-docs-*` branch created before Phase 2.

1. Verify you are NOT on main: run `git branch --show-current` and confirm it starts with `docs/maintain-docs-`. If you are on main, **stop immediately**.
2. Stage and commit all changes.
3. Push the branch and open a PR with the label `documentation` and using the template below.

#### PR conventions

- **Title prefix**: Always start the PR title with `skill:maintain-docs` so humans can identify skill-generated PRs. Example: `skill:maintain-docs index orphaned operational docs in AGENTS.md`
- **Label**: Always add the `documentation` label to the PR (e.g., `gh pr create --label documentation ...`)

#### PR template

Use this structure for the PR body:

```
## Summary

Documentation maintenance run — [DATE].

### Changes made

- [Brief description of each change]

### Full audit findings

| Priority | Finding | Status |
| -------- | ------- | ------ |
| HIGH | [description] | Fixed in this PR |
| HIGH | [description] | Deferred |
| MEDIUM | [description] | Deferred |
| LOW | [description] | Deferred |

### Recommendations for separate work

- [Any structural recommendations that need a dedicated effort]

### Validation checklist

- [ ] All changed files are `.md` or `.mdc` (verified via `git diff --name-only`)
- [ ] Phase 2.5 verification passed (sub-agent output reviewed)
- [ ] `npm run prettier` passed
- [ ] No source code files were modified
```

## Context window management

This skill is designed to stay within context limits:

- **Phase 0** reads the backlog file and runs one `gh` command — minimal context
- **Phase 1** reads only AGENTS.md, file listings, and first ~30 lines of orphaned docs (after exclusion filtering)
- **Phase 2** delegates deep reads to sub-agents, each scoped to one doc plus its related code
- **Phase 2.5** reads the combined `git diff` for verification — proportional to the number of fixes attempted
- **Phase 3** is mechanical (backlog update, git operations, prettier, PR creation)
- Each run is bounded by a 7-point complexity budget (or 4 points under review burden), not a flat item count

## Expected invocation patterns

- **Post-feature work**: Run after a large feature lands to catch docs that fell behind. This is the highest-value trigger.
- **Periodic maintenance**: Run on a schedule (weekly or biweekly) to catch gradual drift.
- **On demand**: User asks "audit the docs" or "check if documentation is up to date."

### Adaptive frequency

The skill self-regulates through several mechanisms:

- **Phase 0 feedback check** halts the skill entirely if recent PRs were rejected, preventing wasted effort.
- **Validated doc timestamps** prevent re-checking docs that were recently confirmed accurate, so repeated runs on an unchanged codebase converge to no-ops.
- **Exclusion list** permanently silences false-positive orphan detections.
- **Structural change filtering** in staleness detection ignores cosmetic commits, reducing noise.
- **Steady-state exit** skips PR creation when no actionable findings exist.

If the skill produces no-op runs for 3+ consecutive invocations, consider reducing frequency to biweekly or trigger-based only. The skill works best when run in response to actual codebase changes, not on a fixed calendar.
