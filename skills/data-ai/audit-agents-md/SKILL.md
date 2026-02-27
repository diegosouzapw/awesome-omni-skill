---
name: audit-agents-md
description: Audit AGENTS.md files for token efficiency, completeness, scope hygiene, and actionability. Also considers skills and Cursor rules for redundancy. Use when the user wants to review, optimize, or restructure project agent instructions.
---

# Audit AGENTS.md

Analyze AGENTS.md files across the repo and produce a structured audit with concrete recommendations.

## Context

AGENTS.md is injected into **every agent session** within its directory tree. Every line competes for context window space alongside conversation history, skills, and tool output. A directive must earn its tokens by changing agent behavior in a way the agent wouldn't do by default.

## Audit Workflow

1. **Discover scope**:
   - If the user specifies a file, audit only that file.
   - Otherwise, find all AGENTS.md files in the repo (`**/AGENTS.md`).
2. **Gather related config**: read all `.cursor/rules/*.mdc` files (note which are `alwaysApply`) and inventory available skills from the session context.
3. **Read** every AGENTS.md in scope.
4. **Inventory** each directive across all files — one line per directive noting its intent and which file it lives in.
5. **Score** each directive against the dimensions below.
6. **Cross-file analysis**: evaluate redistribution, linkage, and overlap between AGENTS.md files (see dimensions 3 and 8).
7. **Produce** the audit report (see Output Format).
8. **Propose** concrete revisions or a redistribution plan if the user wants one.

## Audit Dimensions

Work through each dimension. For every directive, ask the guiding question. Flag anything that fails.

### 1. Token ROI

> "Does this directive change what the agent would actually do?"

- If the agent already follows the behavior by default (e.g. standard pytest usage, common git conventions, obvious language idioms), the directive wastes tokens.
- If the directive restates what Cursor's system prompt already enforces, it adds noise.
- **Action**: recommend removal or, if borderline, mark as low-value with reasoning.

### 2. Actionability

> "Could an agent act on this directive unambiguously?"

- Vague guidance ("write clean code", "follow best practices") is not actionable.
- Good directives are specific and testable: "use `ruff check` before committing", "never use `git add .`".
- **Action**: recommend sharpening or removing.

### 3. Scope Placement

> "Does this directive apply to the entire repo, or only a subtree?"

- Root AGENTS.md should contain only repo-wide behavioral directives.
- Subtree-specific rules belong in the narrowest applicable directory's AGENTS.md.
- **Action**: recommend moving to the narrowest applicable directory.

When multiple AGENTS.md files exist, also check:
- **Upward leakage**: directives in root that only matter for a subtree.
- **Downward duplication**: subtree AGENTS.md restating what root already covers.
- **Missing linkage**: if a subtree AGENTS.md overrides or narrows a root directive, it should reference the root directive it refines.

### 4. Completeness

> "What high-value directives are missing?"

Common gaps to check for (only flag if relevant to the repo):
- **Verification matrix** — a mapping from change area to the smallest targeted verification command to run before committing. Without this, the agent either skips verification or runs overly broad checks. This belongs in root AGENTS.md. Should include a directive: "Always run the smallest targeted verification scope for the area you changed before committing." Example shape:

  ```
  ## Verification Matrix
  | Area changed         | Run to verify                                      |
  | -------------------- | -------------------------------------------------- |
  | C# unit-level logic  | `dotnet test src/App.Tests/App.Tests.csproj`       |
  | Python services      | `python -m pytest utils/tests/ -v`                 |
  | SQL migrations       | `sqlfluff lint migrations/`                        |
  ```
- **Error handling conventions** (e.g. custom exception hierarchy)
- **Naming conventions** the agent can't infer from code alone
- **Architecture boundaries** (e.g. "services never import from handlers")
- **Secret/env handling** (e.g. "never hardcode credentials, use `.env`")
- **PR/review standards** if not covered by a skill
- **Key non-obvious project patterns** the agent would get wrong without guidance

Do NOT recommend adding things the agent already knows from reading code or that are covered by existing skills.

### 5. Verbosity

> "Can the same intent be expressed in fewer tokens without losing precision?"

- Replace paragraphs with single directive sentences where possible.
- Collapse examples into one representative example, not three.
- Remove hedging language ("you might want to consider...").
- **Action**: propose tighter wording.

### 6. Meta-Description Value

> "Does explaining how the AI system works actually improve agent behavior?"

Descriptions of how rules/skills/system prompts interact (e.g. "rules take precedence over model defaults") are generally low-value because:
- The agent already knows its own architecture.
- These descriptions consume tokens without changing behavior.
- **Action**: recommend removal unless a concrete behavioral issue motivated the description.

### 7. Duplication with Skills and Cursor Rules

> "Is this directive already covered by a skill or a Cursor rule?"

**Skills**:
- Cross-reference against skills listed in the session's available_skills.
- If a skill already enforces a behavior, the AGENTS.md directive is redundant.
- Brief *references* to skills (e.g. "follow fcommit philosophy") are fine — they act as triggers. Full re-explanations are not.

**Cursor rules** (`.cursor/rules/*.mdc`):
- Rules marked `alwaysApply` are injected into every session, just like root AGENTS.md — duplication between these two is pure waste.
- Rules with glob or manual triggers overlap only conditionally; flag duplication but note the trigger scope.

- **Action**: recommend removing duplicate content, keeping only a skill trigger or rule reference if needed.

### 8. Content Placement

> "Is this directive in the right home: AGENTS.md, a skill, or a Cursor rule?"

Each has a distinct purpose:

- **AGENTS.md (root)**: repo-wide behavioral directives that should apply to every session touching this codebase. Persistent, always-on context.
- **AGENTS.md (subfolder)**: directives scoped to that subtree only. Use when a rule applies broadly within a folder but not the whole repo.
- **Skill**: reusable workflows that are invoked on demand, potentially across multiple repos or subfolders. Good for multi-step procedures, templates, and conditional logic that doesn't need to be in every session.
- **Cursor rule**: editor-level or file-pattern-scoped guidance. Best for formatting, language-specific conventions, or patterns tied to globs (e.g. `*.test.ts`).

Flag directives that would work better in a different home. Common misplacements:
- A multi-step workflow in AGENTS.md that should be a skill (avoids token cost when not needed).
- A file-pattern convention in AGENTS.md that should be a glob-triggered Cursor rule.
- A skill that re-explains repo context already in AGENTS.md.

## Output Format

Structure the audit as sections per dimension (A–H), with numbered findings within each. Skip dimensions with no findings.

When auditing multiple files, group findings by file, then by dimension within each file. Add a cross-file section at the end.

```
## Audit: <repo name or path>

### Files analyzed
- `AGENTS.md` (root) — N directives
- `tests/AGENTS.md` — N directives
- `.cursor/rules/` — N rules (M alwaysApply)
- Skills — N active

### A. Token ROI

A1. `AGENTS.md:L12` "<directive paraphrased>" — <why it's low-value>. **Remove** / **Keep (borderline)**.

### B. Actionability

B1. `AGENTS.md:L25` "<directive paraphrased>" — too vague to act on. **Rewrite**: "<proposed tighter wording>".

### C. Scope Placement

C1. `AGENTS.md:L8` "<directive paraphrased>" — only applies to `tests/`. **Move** to `tests/AGENTS.md`.

### D. Completeness

D1. Missing: <what's absent>. **Add** to `<target file>`: "<proposed directive>".

### E. Verbosity

E1. `AGENTS.md:L30` "<directive paraphrased>" — <what's wordy>. **Rewrite**: "<tighter version>".

### F. Meta-Description Value

F1. `AGENTS.md:L2` "<directive paraphrased>" — explains system internals without behavioral impact. **Remove**.

### G. Duplication with Skills / Rules

G1. `AGENTS.md:L18` "<directive paraphrased>" — fully covered by skill `fcommit`. **Remove** / **Reduce to trigger**.
G2. `AGENTS.md:L40` "<directive paraphrased>" — duplicates alwaysApply rule `some-rule.mdc`. **Remove from one**.

### H. Content Placement

H1. `AGENTS.md:L35` "<directive paraphrased>" — multi-step workflow better suited as a **skill**.
H2. `tests/AGENTS.md:L5` "<directive paraphrased>" — file-pattern convention better suited as a **glob-triggered Cursor rule**.

### Cross-File Summary

Redistribution plan (if applicable):
- **Create** `tests/AGENTS.md` ← C1, C3
- **Move** H1 → new skill `skill-name`
- **Remove** G1, G2 (covered elsewhere)
- **Linkage**: `tests/AGENTS.md` should reference root directive on <topic> that it narrows.

### Revised AGENTS.md

(Only if user requests) Full rewrite with recommendations applied.
```

## Guardrails

- Do not invent problems. If the AGENTS.md is already tight, say so.
- Preserve the author's intent — reword, don't redefine.
- When uncertain whether the agent "already knows" something, err on the side of keeping the directive but flagging it as "verify: may be redundant".
- Never recommend adding boilerplate the agent doesn't need.
