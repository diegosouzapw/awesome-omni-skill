---
name: 'instruction-creator'
description: 'Guides creation of .instructions.md files that enforce coding standards, conventions, and project rules. Use when asked to "create an instruction", "add coding standards", "write convention rules", or "make a .instructions.md file". Produces a single self-contained .instructions.md file with frontmatter, prose intro, domain-named XML groups, and NEVER/ALWAYS enforcement rules.'
---

Follow these steps to create a self-contained instruction file by studying an exemplar and adapting its patterns for a new concern domain. The governing principle is ambient-constraint authoring — study existing patterns, then encode them as passive rules that shape behavior without directing action. Begin with `<step_1_understand>` to capture intent and scope.


<use_cases>

- Add TypeScript type-safety conventions for a project
- Create SQL query and schema standards for database work
- Write error handling rules for Python services
- Add API security conventions scoped to route files
- Create React component conventions for frontend development

</use_cases>


<workflow>

Execute steps sequentially. Each step builds on the previous — intent determines scope, the exemplar informs structure, the spec governs content, generation produces the file, and validation confirms delivery readiness.


<step_1_understand>

Capture the user's intent and determine instruction scope.

- Determine the **concern domain** — what topic the instruction covers (type safety, naming, error handling, security, testing, etc.)
- Determine the **target file scope** — what files or directories the rules apply to (`**/*.ts`, `src/api/**`, `**/*.py`, etc.)
- Determine the **discovery mode** — file-triggered, on-demand, both, or manual. Default to "both" unless the user specifies otherwise
- Determine the **output location** — where to save the file. Default: `.github/instructions/`
- If the concept describes a multi-step process or task workflow, redirect to skill-creator — instructions are ambient constraints, not procedures
- If the concept describes a persona or role, redirect to agent-creator — instructions are rules, not identities

**Output of this step:** concern domain, target file scope, discovery mode, output path, any domain-specific context the user provided.

</step_1_understand>


<step_2_study>

Study the gold-reference exemplar to understand structural patterns.

- Load [exemplar.md](assets/exemplar.md) — study the embedded instruction alongside its structural annotations
- Observe how the exemplar handles: frontmatter field choices, prose intro with governing principle, XML group naming (domain-specific terms, not generic labels), rule style (NEVER/ALWAYS binary enforcement), scoping decisions (`applyTo` glob granularity)
- Determine how many XML groups the new instruction needs — typically 2-4 groups, one concern per group
- Note what is absent: no `<workflow>`, no `<step_N_verb>` tags, no identity prose, no subfolders, no markdown headings — the output is a single flat file of passive rules

</step_2_study>


<step_3_plan>

Apply the structural specification to plan each section of the new instruction.

Load [instruction-spec.md](references/instruction-spec.md) and plan:

- **Frontmatter** — Plan fields based on discovery mode. File-triggered or both: include `applyTo` glob matching the target file scope. On-demand or both: include `description` following the "{verb} {scope} {concern}" formula. Omit `name` unless disambiguation is required. All string values single-quoted.

- **Prose intro** — 1-2 imperative sentences before any XML tag. State the directive and the governing principle for the concern domain. No hedging, no conditionals.

- **XML groups** — Plan 2-4 domain-named tags, each covering exactly one concern. Name tags using domain vocabulary the target developer would recognize (e.g., `<type_safety>`, `<error_boundaries>`, `<query_style>`). Never use generic names like `<rules>` or `<guidelines>`.

- **Rules** — Plan 3-7 NEVER/ALWAYS rules per group. Each rule states one prohibition or mandate. Include inline code examples when the correct form is non-obvious. No hedge words.

- **Confirm output shape** — The output is a SINGLE flat `.instructions.md` file. No `<workflow>`, no `<step_N_verb>`, no identity prose, no subfolder structure.

</step_3_plan>


<step_4_generate>

Write the complete instruction file as a single flat `.instructions.md` file.

- **Frontmatter** — YAML between `---` fences. Include only the fields needed for the chosen discovery mode. All string values single-quoted.

- **Prose intro** — 1-2 imperative sentences stating the directive and governing principle. Imperative voice, no hedging.

- **XML groups** — Domain-named tags. Each group opens with an optional explanatory sentence stating the group's purpose, followed by 3-7 NEVER/ALWAYS bullet rules. One concern per group. Rules ordered from most common scenario to least common.

- **File naming** — `{concern}.instructions.md` (e.g., `typescript-conventions.instructions.md`, `error-handling.instructions.md`). Name reflects the concern, not the target files.

- **Output** — ONE file. Not a folder, not multiple files, not a skill structure.

</step_4_generate>


<step_5_validate>

Quality-check the generated instruction against all validation gates.

Load [quality-gates.md](references/quality-gates.md) and run each tier:

- **P1 — Blocking** (fix before delivery):
  - File ends in `.instructions.md`
  - YAML string values use single quotes
  - No platform-reserved tags in body
  - No secrets, drive letters, or absolute paths
  - Single flat file — no subfolders
  - 150 lines or fewer

- **P2 — Quality** (fix before finalizing):
  - Discovery mode appropriate for the concern
  - XML groups use domain-specific names, not generic labels
  - All rules use NEVER/ALWAYS — no hedge words
  - `description` follows "{verb} {scope} {concern}" formula if present
  - `applyTo` scope is neither too broad nor too narrow
  - Prose intro present with imperative voice
  - One concern per group, 3-7 rules per group

- **P3 — Polish** (flag as suggestions):
  - Active voice throughout
  - Concise — no filler words
  - Code examples enhance complex rules

- **Banned patterns scan** — Check for: identity prose, workflow/step tags, stance words (should, prefer), hedge language (try to, when possible), generic group names, prompt variables, drive letters, temporal language

Fix all P1 and P2 issues. Report P3 suggestions to the user. If P1 fixes require structural changes, return to `<step_4_generate>`.

</step_5_validate>


</workflow>


<error_handling>

Recovery actions for common failure modes. Apply the matching recovery when an issue surfaces during any step.

- If the concept is **too broad** (multiple unrelated concerns in one instruction) — split into separate instruction files, one per concern domain
- If the concept is actually a **multi-step workflow or procedure** — redirect to skill-creator — instructions are ambient constraints, not task orchestration
- If the concept is a **persona or role description** — redirect to agent-creator — instructions are rules, not identities
- If all rules use **"should" or "prefer"** instead of NEVER/ALWAYS — rewrite with binary enforcement — hedged rules produce lower LLM compliance
- If **P1 validation failures** are found in step 5 — return to `<step_4_generate>` and fix specific violations — do not regenerate the entire file unless structural issues require it

</error_handling>


<resources>

Reference files loaded on demand during workflow steps. All paths are relative to the skill folder.

**References:**

- [instruction-spec.md](references/instruction-spec.md) — Instruction structural specification: frontmatter fields, discovery modes, body structure, scoping strategies, and design boundaries. Loaded in step 3.
- [quality-gates.md](references/quality-gates.md) — Validation tiers (P1/P2/P3), banned patterns, platform-reserved tags, and instruction anti-patterns. Loaded in step 5.

**Assets:**

- [exemplar.md](assets/exemplar.md) — Annotated gold-reference instruction (sql-conventions) with structural observations. Loaded in step 2.

</resources>
