---
name: learn
description: After completing a task, capture learnings by updating AI tool configuration. Creates or updates rules, agents, and context files based on what was discovered — in the native format for each tool (Claude Code, VS Code Copilot, Cursor, AGENTS.md). Use after any task where something surprised you, went wrong, or produced a reusable insight.
---

# Learn — Update AI Tool Configuration From Experience

## Purpose

After completing a task, turn what you learned into native artifacts for all configured AI tools so the next task benefits automatically. The output is always a rule, agent update, or context file update — never a log file or template.

## When to Use

### Manual Trigger
- After any task where something surprised you or went wrong
- After discovering a pattern worth preserving
- After stepping on a landmine that wasn't documented
- After working in a module that had no CLAUDE.md context
- Skip for routine tasks where nothing new was learned

### Automatic Trigger (run without being asked)
Run `/learn` automatically when any of these signals occur during a session:

| Signal | What happened | What to capture |
|--------|--------------|-----------------|
| **User corrects your approach** | "No, don't do it that way — use X" | **Pattern** — the correct approach |
| **Same fix requested twice** | User asks to fix the same issue/area again | **Landmine** — the fragile area and root cause |
| **Your change breaks something** | Tests fail, build breaks, behavior regresses | **Landmine** — what broke and why |
| **User rejects generated code** | "That's wrong", "revert that", user undoes your change | **Pattern or Landmine** — what was wrong, what's correct |
| **Undocumented convention discovered** | Code follows a pattern not in any rule | **Pattern** — document it |
| **Undocumented trap hit** | Something looked safe but caused problems | **Landmine** — document the trap |

**Auto-trigger workflow:**
1. Detect the signal during normal work
2. Finish the immediate fix or correction first
3. Then run `/learn` — start from Step 1 below
4. The signal itself provides the learning — you don't need the user to describe it

## Content Depth

Generated rules must carry enough depth to be genuinely useful. Use two content structures from the project's knowledge base:

- **Pattern structure** (`skills/_references/patterns/_TEMPLATE.md`) — for conventions and reusable approaches. Follow the full template: When to Use, Problem It Solves, Core Approach with step-by-step code examples, Best Practices, Common Mistakes with wrong/fix code, Variations, Related patterns and landmines.
- **Landmine structure** (`skills/_references/landmines/_TEMPLATE.md`) — for danger zones and known traps. Follow the full template: Severity, Symptoms, Root Cause, The Trap, Safe Approach (Don't/Do with code), Validation, Prevention, Related patterns and landmines.

**Line limits:**
- **Context files** (CLAUDE.md, AGENTS.md, copilot-instructions.md): MUST stay under **100 lines** (max 125 if critical context would be lost). Split into multiple scoped files rather than exceeding.
- **Pattern and landmine rules**: Should be as detailed as the templates require — typically **50-150 lines**. Depth and working code examples matter more than brevity.

## Tool Target Preferences

Before generating any output, check for `.north-starr.json` in the project root:

```json
{
  "version": 1,
  "targets": ["claude", "copilot", "cursor"]
}
```

- If the file exists, only generate artifacts for the listed targets
- If the file is missing, generate for all tools (backward compatible)
- "All formats" throughout this skill means **enabled target formats only**
- `AGENTS.md` is always updated regardless of preferences (it's universal)

## Workflow

### Step 1: Identify What Was Learned

Ask these questions about the completed task:

1. **Surprises** — Did anything behave differently than expected?
2. **Friction** — Where did you slow down, get stuck, or make a mistake?
3. **Patterns** — Did you discover a reusable approach that should be followed again?
4. **Danger zones** — Did you find an area that's more fragile or complex than it appeared?
5. **Vocabulary** — Did any terms turn out to mean something non-obvious?
6. **Rule gaps** — Did an existing rule fail to fire, or was a rule missing?

If the answer to all of these is "nothing" — skip the rest. Not every task produces learnings.

### Step 2: Decide What to Create or Update

Map each learning to the right artifact:

| What you learned | Artifact to create/update |
|---|---|
| A convention that should always be followed | **Pattern rule** — create in enabled tool formats (`.claude/rules/`, `.github/instructions/`, `.cursor/rules/`) using pattern structure |
| A danger zone or known trap | **Landmine rule** — create in enabled tool formats using landmine structure, plus module-level `CLAUDE.md` Caution section (if `claude` enabled) |
| A reusable pattern for how things are done | **Pattern rule** — create in enabled tool formats using pattern structure |
| Architecture understanding deepened | Root context — update `CLAUDE.md` (if `claude` enabled), `AGENTS.md`, `.github/copilot-instructions.md` (if `copilot` enabled) |
| A new term was clarified | Root context — update Vocabulary section in enabled context files |
| The explorer agent needs more context | Agent files — update `.claude/agents/` (if `claude` enabled) and/or `.github/agents/` (if `copilot` enabled) |
| A recurring task type was identified | Suggest creating a new skill |

### Step 2.5: Detect Conflicts With Existing Configuration

Before generating artifacts, check whether any new learning **contradicts, replaces, or narrows** something already documented. This step prevents silently overwriting rules that still apply to existing code.

**Actions:**

1. For each learning identified in Step 1, read the existing rules, CLAUDE.md files, and context files that cover the same area
2. Classify the relationship between the new learning and existing content:

| Relationship | Example | Action |
|---|---|---|
| **Additive** — no existing content covers this | New rule for a previously undocumented convention | Auto-create. No prompt needed. |
| **Contradicts** — existing content says the opposite | Old rule says "use library A", new learning says "use library B" | **Prompt user** before changing anything. |
| **Narrows** — existing content is too broad now | Rule applies to all modules, but only some modules should follow it now | **Prompt user** with scope adjustment proposal. |
| **Deepens** — existing content is correct but incomplete | Existing CLAUDE.md has Architecture section, new insight adds nuance | Auto-update. Append or refine, don't replace. |

3. For each **Contradicts** or **Narrows** conflict, present the user with the conflict and ask which resolution applies:

**Resolution options:**

| Resolution | When to use | What to do |
|---|---|---|
| **Replace** | Old pattern is fully gone, no code uses it anymore | Delete old content, write new content. Add `<!-- [SUPERSEDES] old-rule-name — reason -->` comment to the new artifact for traceability. |
| **Deprecate** | Old code still exists but new code should follow the new pattern | Keep old rule, add `[DEPRECATED — migrate to X]` marker. Create new rule for the new pattern. |
| **Scope-split** | Both patterns coexist — some modules use old, some use new | Narrow old rule's path glob to only the modules still using it. Create new rule scoped to the modules using the new pattern. |
| **Keep existing** | The new learning was wrong or situational | Do not update. Optionally note the exception in the relevant CLAUDE.md. |

**Do not silently replace existing rules.** Old code may still depend on existing guidance. Prompting ensures no working guidance is lost.

#### Tag Conventions

**`[DEPRECATED — migrate to X]`** — Added to rules, CLAUDE.md sections, or context content that still applies to existing code but should not be used for new code.

Format in rules:
```markdown
---
paths: ["src/legacy/**"]
---

[DEPRECATED — migrate to new-pattern. See `.claude/rules/new-pattern.md`]

Use old-pattern for files in this directory.
```

Format in CLAUDE.md sections:
```markdown
## Error Handling

[DEPRECATED — migrate to centralized error handler. See `Architecture` section.]

This module uses inline try/catch with local error formatting...
```

**`<!-- [SUPERSEDES] old-name — reason -->`** — Added as a comment to the new artifact that replaced an old one, for traceability.

```markdown
---
paths: ["src/**"]
---

<!-- [SUPERSEDES] old-error-handling — team adopted centralized error handler -->

Use the shared error handler for all error responses.
```

**Tag lifecycle:**
- `[DEPRECATED]` tags are reviewed whenever `/learn` runs — if the old code has been fully migrated, the deprecated rule should be deleted
- `[SUPERSEDES]` comments are permanent traceability — they stay in the file

### Step 2.7: Handle Existing Patterns and Landmines

Before creating a new pattern or landmine, check whether one already exists for the same area or concern. This prevents duplicates and keeps rules consolidated.

**Actions:**

1. Search existing rules in all enabled tool directories (`.claude/rules/`, `.github/instructions/`, `.cursor/rules/`) for files covering the same topic, module, or concern
2. Also check module-level `CLAUDE.md` files for related caution sections or pattern notes
3. Classify what you found:

| Finding | Action |
|---------|--------|
| **No existing file** — this is a new concern | Create a new pattern or landmine rule (proceed to Step 3) |
| **Existing file covers the same concern** — new learning adds depth | **Update the existing file** — append new examples, symptoms, or steps. Do not create a duplicate. |
| **Existing file covers the same concern** — new learning contradicts it | **Prompt the user** with the conflict (same as Step 2.5 resolution options: Replace, Deprecate, Scope-split, Keep existing) |
| **Existing file is related but different** — overlapping but distinct concern | Create a new file and add cross-references in the Related section of both files |

**When updating an existing file:**
- Read the full existing content first
- Preserve the existing structure (pattern template or landmine template)
- Add new information in the appropriate section (new symptoms to Symptoms, new code examples to Safe Approach, new mistakes to Common Mistakes, etc.)
- Update the Changelog at the bottom with a new version entry
- Update `Last Updated` date
- **Always prompt the user** before updating an existing file — show what will change and ask for confirmation:

```
I found an existing [pattern/landmine] that covers this area:
  → [file path]

The [signal: user correction / repeated fix / etc.] revealed new information:
  → [what was learned]

I'd like to update the existing file by:
  → [specific changes: add new symptom, add code example, update Safe Approach, etc.]

Should I:
1. Update the existing file (recommended)
2. Create a separate new rule instead
3. Skip — this isn't worth capturing
```

### Step 3: Generate the Artifacts

For each learning, create or update the appropriate file using the resolution determined in Steps 2.5 and 2.7:

#### New Rule

When a convention, constraint, or danger was discovered, create in each enabled target format. Use the appropriate frontmatter for each tool:

- **Claude Code** (`.claude/rules/*.md`): `paths: ["glob/pattern/**"]` — if `claude` target enabled
- **VS Code Copilot** (`.github/instructions/*.instructions.md`): `applyTo: "glob/pattern/**"` — if `copilot` target enabled
- **Cursor** (`.cursor/rules/*.mdc`): `globs: glob/pattern/**` — if `cursor` target enabled

The rule body is the same across tools — only the frontmatter differs. Choose the appropriate structure:

**Pattern Rules** — for conventions and reusable approaches. Follow the **full pattern template** from `skills/_references/patterns/_TEMPLATE.md`. Include: Category, When to Use / Not Good For, Problem It Solves, The Pattern (step-by-step with code), Best Practices, Common Mistakes (wrong/fix code), Variations, Related.

**Landmine Rules** — for danger zones and known traps. Follow the **full landmine template** from `skills/_references/landmines/_TEMPLATE.md`. Include: Severity, Category, Quick Summary, Symptoms, Root Cause, The Trap, Safe Approach (Don't/Do with code), Validation, Prevention, Related.

**Guidelines:**
- Scope the glob as narrowly as possible
- Include code examples — abstract rules without code are not actionable
- State the rule as a clear instruction, not a description
- Include the "why" — it helps AI tools apply the rule correctly
- Name the file after the concern: `api-error-handling.md`, `sync-race-condition.md`
- Pattern and landmine rules should be as detailed as the templates require — typically 50-150 lines

#### Module CLAUDE.md Update

When a danger zone or module-specific pattern was found (if `claude` target enabled):

- If no `CLAUDE.md` exists in that directory, create one
- If one exists, add to the relevant section (Caution, Patterns)
- Keep it concise — only what someone working in this module needs to know

#### Root Context File Update

When project-level understanding changed, update root context files for enabled targets:

- `CLAUDE.md` — Claude Code (if `claude` target enabled)
- `AGENTS.md` — Universal (always)
- `.github/copilot-instructions.md` — VS Code Copilot (if `copilot` target enabled)

For each:
- Update the specific section (Architecture, Grain, Module Map, Vocabulary)
- Don't rewrite sections that haven't changed

#### Agent Update

When the explorer agent's prompt should include new knowledge, update enabled target formats:

- `.claude/agents/*.md` — Claude Code (if `claude` target enabled)
- `.github/agents/*.agent.md` — VS Code Copilot (if `copilot` target enabled)

For each:
- Add to the agent's context about architecture, danger zones, or module relationships
- Keep the agent prompt focused — it's context, not a manual

### Step 3.5: Clean Up Deprecated Artifacts

After generating new artifacts, check whether any `[DEPRECATED]` tags in the project can be resolved:

1. Scan existing rules and CLAUDE.md files for `[DEPRECATED — migrate to X]` markers
2. For each deprecated artifact, check whether the old pattern still exists in the codebase (search for actual usage, not just the rule)
3. If the old pattern is fully gone — delete the deprecated rule and inform the user
4. If the old pattern still exists — leave the `[DEPRECATED]` tag in place

This keeps the configuration clean over time without losing guidance for code that still needs it.

### Step 4: Present Summary

```
## Learnings Applied

**Task:** [what was completed]
**Trigger:** [manual | auto: user correction | auto: repeated fix | auto: change broke something | auto: rejected code | auto: undocumented convention | auto: undocumented trap]

**Created:**
- [artifact type]: [file path] — [what was created]

**Updated (existing):**
- [artifact type]: [file path] — [what was added/changed in the existing file]

**Conflicts Resolved:**
- [file path] — [resolution: replaced / deprecated / scope-split / kept existing] — [reason]

**Deprecated Rules Cleaned Up:**
- [file path] — old pattern no longer found in codebase, rule deleted

**Why:** [brief explanation of what triggered these updates]
```

Omit sections that have no entries (Created, Updated, Conflicts Resolved, Deprecated Rules Cleaned Up).

## Notes

- This skill is language-agnostic — works for any project type
- This skill respects tool target preferences — check `.north-starr.json` for enabled targets. Only generate artifacts for enabled tools.
- Every output must be a native artifact for the enabled AI tools (rules, agents, context files) — not a log or template file
- Read existing files before updating — build on what's there, don't duplicate
- **Never create a duplicate rule** — always check for existing patterns/landmines covering the same concern before creating new files (Step 2.7)
- **Never silently replace or delete existing rules** — always prompt the user when new knowledge contradicts existing content
- **Always prompt the user before updating an existing pattern or landmine** — show what will change and let them choose (update, create separate, or skip)
- Auto-update without prompting is fine only for additive changes to context files (CLAUDE.md, AGENTS.md) that deepen existing content
- Keep rules and CLAUDE.md content concise — verbose documentation goes stale and wastes context
- Not every task produces learnings — it's fine to run this and conclude "nothing to update"
- When auto-triggered, finish the immediate fix first, then capture the learning — don't interrupt the user's flow
- When unsure whether something deserves a rule vs. a CLAUDE.md note: rules are for constraints that should always be enforced, CLAUDE.md is for context that helps Claude make better decisions
- `[DEPRECATED]` tags have a lifecycle — they should be cleaned up once the old pattern is fully gone, not left indefinitely
- When both old and new patterns coexist, prefer **scope-split** over **replace** — old code still needs its rules until it's actually changed
