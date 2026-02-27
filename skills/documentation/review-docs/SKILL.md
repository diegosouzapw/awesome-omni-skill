---
name: review-docs
description: Review documentation (README.md and CLAUDE.md) for quality, completeness, and consistency. Use when asked to review docs, check documentation, validate README files, or audit CLAUDE.md coverage.
---

# Documentation Review

Review documentation in the specified path (default: entire repository).

> [!IMPORTANT]
> Consult `REFERENCE.md` in this skill directory for the expected output format and level of detail.

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Argument Parsing

Parse `$ARGUMENTS` for:
- **Path**: Optional path to review (default: repo root, reviews all docs)
- **`--create-beads`**: Create beads for findings (default: report only)

## Workflow

1. Run deterministic validation script for structural issues
2. Find all README and CLAUDE.md files in scope
3. Evaluate against quality criteria
4. Cross-reference enumerations in docs against codebase sources of truth (see Enumeration Completeness)
5. Check for existing beads: `bd list --status=open`
6. For each issue found, skip if a bead already exists with matching file/issue
7. Create beads only for new issues

### Deterministic Validation

Before manual review, run the validation script for CLAUDE.md structural issues:

```bash
python3 ${CLAUDE_PLUGIN_ROOT}/skills/review-docs/scripts/validate-claude-md.py $0 --json
```

Parse the JSON output and include structural issues (missing tables, missing references, index drift) in your findings. The script checks:
- Presence of markdown tables with required columns
- Whether referenced files/directories exist
- Index drift (files not indexed, or index entries pointing to missing files)
- `@path` import targets resolve to existing files
- `.claude/rules/` file structure, frontmatter globs, and empty files
- Leaked local preferences (hardcoded user paths, localhost URLs) in shared CLAUDE.md

### Deduplication

Before creating a bead, check if one already exists:
- Run `bd list --status=open` to see all open issues
- Run `bd list --status=closed` to see previously resolved issues
- Compare your findings against existing bead titles and descriptions
- Skip creating beads for issues that already have matching open beads
- **Skip findings where a closed bead's fix introduced the pattern you're flagging.** Use `bd show <id>` on relevant closed beads. If the current state is the intentional resolution of a prior issue, do not re-raise it.
- When in doubt, show the user the potential duplicate rather than creating

## Quality Criteria

### Structure (Progressive Disclosure)

README.md files belong at **component boundaries** — the repo root plus the top-level directory of each major component (e.g., `frontend/`, `backend/`, `infra/`, `docs/`). Most subdirectories within a component should not have their own README. If a directory only needs a brief description and a pointer to its files, a CLAUDE.md index is sufficient.

**Root README.md**:
- Project purpose (why it exists)
- What it does (brief overview)
- Quick start instructions
- Links to component docs
- NO deep architectural details

**Component READMEs** (top-level component dirs only):
- Component's purpose in detail
- Architectural decisions
- Component-specific setup
- Internal APIs/interfaces

Do **not** flag missing READMEs in nested subdirectories (e.g., `src/utils/`, `lib/internal/`). Navigation within a component is handled by CLAUDE.md.

### Content Quality

- **Concise**: No unnecessary words or redundancy
- **Clear**: Jargon explained, unambiguous
- **Accurate**: Examples work, paths exist, commands valid, lists reflect what actually exists in code
- **Complete**: Prerequisites listed, all steps included, enumerations cover all items (see Enumeration Completeness below)
- **Consistent**: Uniform terminology and formatting
- **Internally consistent**: Numbers, counts, and facts agree across all mentions within the same document

Content Quality applies primarily to README prose. For CLAUDE.md, focus on accuracy (index entries point to real files) and consistency (uniform table format).

### Enumeration Completeness

When documentation contains a list of things — components, features, agents, services, environment variables, log files, database tables, API endpoints — verify the list is **complete and accurate** by cross-referencing the codebase.

**Workflow**:
1. Identify every enumeration in the docs (any list that claims to describe "all" or "the" items of a type)
2. Find the canonical source in code (e.g., config directory for agents, env template for env vars, schema definitions for DB tables)
3. Diff the two: items in code but missing from docs, items in docs but removed from code
4. Flag gaps as P3 (stale or incomplete)

**Common patterns to cross-reference**:
- Agent/service/component lists → config files or directory listings
- Feature/capability lists → tool, handler, or route files
- Environment variable documentation → `.env.example`, `.env.template`, or equivalent
- Log file lists → logging configuration entries
- Database table/schema documentation → schema definition files or migrations
- CLI command/Makefile target documentation → actual command definitions

**What to skip**: Exhaustive enumeration is not always expected. Use judgement — a "Key features" section need not list every minor capability. But a section titled "Agents" or "Configuration" that presents itself as comprehensive should be complete.

### Hidden Knowledge

Good docs surface things newcomers can't discover on their own. Flag when these are missing:

**Prerequisites & Environment**:
- System dependencies (e.g., "requires Docker 20+", "needs libssl-dev on Linux")
- Required tool versions (node, python, go, etc.)
- Required accounts or API keys (with signup links, not the keys themselves)
- Environment variables with example values (use `YOUR_API_KEY` placeholders)

**Platform Differences**:
- OS-specific instructions when behavior differs (Mac vs Linux vs Windows)
- Architecture notes if relevant (ARM vs x86)

**Gotchas & Failure Modes**:
- Common setup errors and fixes ("If you see X, run Y")
- Non-obvious side effects ("This command also resets the database")
- Known limitations or unsupported scenarios

**Magic Values**:
- Default ports, timeouts, limits that aren't obvious from code
- Config file locations that vary by platform
- Implicit ordering dependencies ("Run A before B")

### Quick Start Quality

The quick start should let someone succeed in minutes, not hours. Evaluate:

**Copy-Pasteable Commands**:
- Commands can be copied and run verbatim (no unexplained `<placeholders>`)
- If placeholders are needed, explain how to get the real value
- Shell-specific syntax noted if it matters (bash vs zsh vs fish)

**Expected Output**:
- Show what success looks like ("You should see: ...")
- Include sample output for key commands
- Note how to verify it worked

**Minimal Path**:
- Doesn't require reading other sections first
- Optional features clearly marked as optional
- "Hello world" possible before diving into configuration

**First-Run Troubleshooting**:
- Top 2-3 things that go wrong on first run
- One-liner fixes for each
- Link to more detailed troubleshooting if it exists

### CLAUDE.md Navigation Index

CLAUDE.md files provide progressive disclosure via tabular indexes, pointing readers to the right file at the right time. README.md holds invisible knowledge (architecture, design decisions, invariants); CLAUDE.md holds navigation.

> CLAUDE.md conventions inspired by solatis/claude-config (MIT).

**Coverage** — Non-trivial directories should have a CLAUDE.md. Skip generated dirs, vendored deps, stubs (`.gitkeep` only), `.git`, `node_modules`, `__pycache__`, `dist`, `build`, and similar. Also skip `CLAUDE.local.md` and `MEMORY.md` — these are personal/auto-managed and not part of shared project documentation.

**Tabular index format** — Each CLAUDE.md must contain at least one markdown table with columns: `File` (or `Directory`), `What`, `When to read`. Entries should use:
- Backtick-wrapped file/directory names (e.g., `` `src/` ``)
- Noun-based "What" descriptions (e.g., "Entry point", "Test utilities")
- Action-verb "When to read" triggers (e.g., "Adding a new route", "Debugging test failures")

**Content separation (subdirectory CLAUDE.md only)** — Subdirectory CLAUDE.md files should contain only:
- A one-sentence overview of the directory's purpose
- One or more tabular indexes

They should **not** contain architecture explanations, design decisions, invariants, or multi-paragraph prose. That content belongs in README.md.

**Root CLAUDE.md operational sections** — The root CLAUDE.md may additionally contain operational sections (Build, Test, Lint commands), agent instruction sections (e.g., beads workflow, tool configuration), and other project-wide guidance. This is expected and should not be flagged.

### CLAUDE.md Progressive Disclosure

**Detail belongs where the code lives.** CLAUDE.md depth should match directory depth. The goal is to minimize context loading at each level — an AI working in `internal/cloud/aws/` should not need to read the entire root CLAUDE.md to understand that directory.

The hierarchy works as follows:
- **Root CLAUDE.md**: Navigation index + dev commands. Just enough to find where to go.
- **Branch CLAUDE.md** (e.g., `internal/cloud/`): Architecture and patterns for that area + index to children.
- **Leaf CLAUDE.md** (e.g., `internal/cloud/aws/`): Implementation specifics for that directory.

**Anti-pattern**: A root CLAUDE.md that contains multi-paragraph explanations of how `internal/cloud/` works, what patterns `internal/broker/` uses, or how `test/` is structured. That detail should live in the respective subdirectory CLAUDE.md files, not the root.

**What root CLAUDE.md should contain**:
- Project overview (1-2 sentences)
- Tabular navigation index pointing to major areas
- Build/test/lint commands
- Agent workflow instructions (if applicable)
- Common gotchas (brief, not exhaustive)

**What root CLAUDE.md should NOT contain**:
- Multi-paragraph architecture explanations for subdirectories
- Implementation patterns specific to a component
- Detailed testing approaches (belongs in `test/CLAUDE.md`)
- Provider-specific details (belongs in provider directories)

**Index drift** — Flag when:
- A file or subdirectory exists in the directory but is missing from the CLAUDE.md index (excluding dotfiles, generated artifacts, and files covered by glob patterns in the index)
- An index entry references a file or directory that does not exist
- A CLAUDE.md index links to a README that doesn't exist, or a README references a component whose CLAUDE.md is missing

**Misplaced detail** — Flag when:
- Root CLAUDE.md contains multi-paragraph explanations about subdirectory internals
- Architectural detail about a component appears in the root instead of that component's CLAUDE.md
- A branch directory lacks a CLAUDE.md but its detail is front-loaded in a parent CLAUDE.md

### CLAUDE.md Imports (`@path` Syntax)

CLAUDE.md files can import other files using `@path/to/file` syntax. The skill validates these references:

- Import target must exist (P2 if missing)
- Relative paths resolve relative to the containing file's directory
- Maximum 5 hops for recursive imports (A imports B imports C...); flag deeper chains as P3
- Circular imports are P2
- Imports inside fenced code blocks and inline code spans are not evaluated — do not flag them
- Exclude email addresses (user@example.com), npm scopes (`@org/pkg`), and social handles (`@username`) from import checking

### `.claude/rules/` Directory

The `.claude/rules/` directory contains per-topic rule files that scope instructions to specific file paths. Do not flag an absent `.claude/rules/` directory — it is optional.

- Each file should cover one topic (P4 if unfocused)
- Filenames should be descriptive — flag generic names like `rules.md`, `misc.md` (P4)
- `paths` frontmatter: P2 for invalid glob syntax, P3 for globs matching no files, P4 if universal rules are unnecessarily scoped to specific paths
- Flag duplication between rules files and CLAUDE.md (P3)

### Memory Location Awareness

**`.claude/CLAUDE.md` alternative** — Both `./CLAUDE.md` and `./.claude/CLAUDE.md` are valid project instruction locations. Do not flag the choice of one over the other. Check both locations when reviewing.

**`CLAUDE.local.md` (personal, not reviewed)** — `CLAUDE.local.md` contains personal preferences and is gitignored. Do not flag it as missing. Do not review its content. Flag personal/local preferences leaked into shared CLAUDE.md as P3 (hardcoded user home paths like `/Users/<name>/`, `/home/<name>/`, `C:\Users\<name>\`; personal sandbox URLs; machine-specific values).

**`MEMORY.md` / auto memory (excluded)** — Files in `~/.claude/projects/<project>/memory/` including `MEMORY.md` are managed by Claude Code directly. The skill must:
- Not suggest changes to MEMORY.md
- Not flag it for any criteria
- Not recommend creating or restructuring it
- Skip it entirely during discovery

### Instruction Specificity

Instructions in CLAUDE.md and `.claude/rules/` should be specific and actionable. Flag vague directives as P4:
- "follow best practices"
- "write clean code"
- "use appropriate naming"
- "ensure quality"

Do not flag intentionally high-level guidance that communicates a genuine design preference (e.g., "prefer composition over inheritance", "use dependency injection for external services").

### Formatting Best Practices

Instructions should be structured for fast scanning:
- Instructions should use bullet points, not prose paragraphs (P4)
- Related bullets should be grouped under descriptive headings — flag 10+ ungrouped consecutive items (P4)

Applies to CLAUDE.md and `.claude/rules/` instruction content only. Does not apply to tabular indexes, README prose, or code block sections.

## Checklist

### Structure & Navigation
- [ ] Root README has: purpose, overview, quick start
- [ ] Root README links to component docs
- [ ] Each major component root has a README (not nested subdirs)
- [ ] Progressive disclosure maintained
- [ ] Non-trivial directories have CLAUDE.md files
- [ ] Each CLAUDE.md has a tabular index (`File`/`Directory`, `What`, `When to read`)
- [ ] Subdirectory CLAUDE.md files contain only overview + index (no architecture prose)
- [ ] CLAUDE.md indexes are in sync with actual directory contents (no drift)
- [ ] Root CLAUDE.md contains navigation + commands, not subdirectory architecture details
- [ ] Detail lives where the code lives (no front-loading in parent CLAUDE.md files)

### Content Quality
- [ ] No outdated references (paths, commands)
- [ ] Code examples syntactically correct
- [ ] Referenced files/commands exist
- [ ] Consistent terminology
- [ ] No duplicate information
- [ ] Numbers and facts consistent across all mentions within the same document

### Codebase Consistency
- [ ] Enumerations (agents, features, env vars, etc.) cross-referenced against code
- [ ] No undocumented components in sections that present themselves as comprehensive
- [ ] No documented items that no longer exist in code

### Memory Ecosystem
- [ ] `@path` imports resolve to existing files
- [ ] No import chains exceed 5 hops
- [ ] `.claude/rules/` files focused and descriptively named
- [ ] `paths` frontmatter valid and matching real files
- [ ] No duplication between rules files and CLAUDE.md
- [ ] No personal preferences leaked into shared CLAUDE.md
- [ ] MEMORY.md excluded from review scope
- [ ] Instructions specific and actionable
- [ ] Instructions use bullets grouped under headings

### Hidden Knowledge
- [ ] Prerequisites documented (system deps, tool versions)
- [ ] Required env vars listed with example values
- [ ] Platform-specific instructions where behavior differs
- [ ] Common errors and fixes documented
- [ ] Magic values (ports, timeouts, defaults) explained

### Quick Start
- [ ] Commands are copy-pasteable (no unexplained placeholders)
- [ ] Expected output shown for key steps
- [ ] Minimal path to first success (no detours)
- [ ] First-run troubleshooting for common failures

## Output

You MUST produce a report following the exact structure shown in `REFERENCE.md`.

**Severity guide**:
- **P1** — Security-relevant: docs omit auth steps, expose secrets in examples, or give dangerous command examples
- **P2** — Broken: code examples that error, paths/commands that don't exist, quick start fails on copy-paste, index entries pointing to missing files, broken `@path` imports, circular imports, invalid glob syntax in rules frontmatter
- **P3** — Stale or incomplete: outdated references, missing prerequisites, missing env var docs, missing CLAUDE.md coverage, index drift, no expected output shown, misplaced detail (architecture in root instead of subdirectory CLAUDE.md), enumeration gaps (components/features/env vars in code but missing from docs), internal fact contradictions within the same document, leaked local preferences in shared CLAUDE.md, orphan globs in rules frontmatter, duplication between rules files and CLAUDE.md, import chains exceeding 5 hops
- **P4** — Polish: formatting inconsistencies, verbose wording, missing "When to read" triggers, missing platform-specific notes, vague instructions ("follow best practices"), generic rule filenames, ungrouped instruction lists, unnecessarily scoped universal rules

**`--create-beads` mode**:

Each bead description MUST be structured in three parts:

```
<file:line (or just filename if no specific line)>

<Explanation of the problem: what is wrong or missing and why it matters.>

Fix: <Concrete prescription of exactly what to change or add.>

Done when: <A verifiable criterion checkable by reading the file.
For bugs: "The link at README.md:45 resolves to an existing file."
For tasks: "The Quick Start section lists the minimum required Docker version
with a link to the install page."
NOT: "The docs are accurate.">
```

- Errors/inconsistencies → bug beads:
  ```bash
  bd create --type=bug --priority=<1-3> --title="Docs: <issue>" \
    --description="<file:line>

  <explanation>

  Fix: <concrete prescription>

  Done when: <verifiable criterion>"
  ```

- Missing content → task beads:
  ```bash
  bd create --type=task --priority=<2-4> --title="Docs: <what to add>" \
    --description="<file>

  <explanation>

  Fix: <concrete prescription>

  Done when: <verifiable criterion>"
  ```
