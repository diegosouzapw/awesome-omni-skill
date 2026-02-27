---
name: edit-claude
description: Creates, updates, or optimizes CLAUDE.md files following Anthropic best practices. Use when user requests creating, updating, improving, or optimizing CLAUDE.md files for project context, coding standards, or persistent memory.
---

# Instructions for Managing CLAUDE.md Files

## Determine Action Type

**CREATE**: New CLAUDE.md file requested
**UPDATE**: Modify existing CLAUDE.md (keywords: "update", "add", "modify", "change")
**OPTIMIZE**: Improve token efficiency (keywords: "optimize", "reduce tokens", "improve")

## Updating Existing CLAUDE.md Files

1. **Read current file**: Always read before editing
2. **Identify section**: Locate relevant section or create new heading
3. **Make surgical edits**: Use Edit tool for precise changes
4. **Preserve structure**: Maintain existing organization patterns
5. **Validate**: Ensure markdown is valid and clear

## Optimizing CLAUDE.md Files

1. **Audit current content**:
   - Identify verbose prose that can become bullets
   - Find repetitive information
   - Locate outdated or irrelevant content

2. **Apply compression techniques**:
   - Convert paragraphs → bullets or tables
   - Remove unnecessary words and filler
   - Use abbreviations where context is clear
   - Group similar items together

3. **Remove anti-patterns**:
   - Delete sensitive information (credentials, tokens)
   - Remove frequently changing data
   - Extract verbose documentation to separate files
   - Remove duplicate information

4. **Validate token efficiency**: Aim for maximum signal, minimum tokens

See `reference.md` for optimization strategies and examples.

## Creating New CLAUDE.md Files

1. **Gather context**: Ask user for project details if missing:
   - Coding standards (indentation, naming conventions)
   - Build/test/deployment commands
   - Architectural patterns
   - Security requirements

2. **Organize around WHAT/WHY/HOW**:
   - **WHAT**: Tech stack, codebase map, key packages
   - **WHY**: Project purpose, component responsibilities
   - **HOW**: Build/test/deploy commands, verification methods

3. **Determine organization strategy** (memory hierarchy):

   **Main CLAUDE.md** (universal, <200 tokens ideal, <500 acceptable):
   - Build/test/deploy commands
   - Universal code style applying to all files
   - Critical patterns used everywhere
   - Cohesive project-wide conventions (Git, Security, Planning, Style)
   - `CLAUDE.md` in project root (shared via git)

   **.claude/rules/** (modular, 100-300 tokens each):
   - Path/language-specific files (auto-loaded): `python.md`, `javascript.md`
   - Domain-specific patterns: `frontend/`, `backend/`
   - Path-specific rules with frontmatter (see reference.md)

   **CLAUDE.local.md** (personal, auto-gitignored):
   - Personal preferences not shared with team
   - Local dev shortcuts, experimental rules

   **~/.claude/CLAUDE.md** (cross-project personal):
   - Universal personal preferences across all projects

   **@imports** (lazy-loaded reference):
   - External docs: `@README`, `@docs/architecture.md`
   - Home directory: `@~/.claude/my-prefs.md`

   **Memory load order** (later overrides earlier):
   1. Enterprise policy → 2. Project memory → 3. Project rules (.claude/rules/) → 4. User memory (~/.claude/) → 5. Project local (CLAUDE.local.md)

4. **Organization decision tree**:
   - Universal + cohesive (Git/Security/Planning)? → Main CLAUDE.md (even if 200-500 tokens)
   - Path/language-specific (Python/JS/Bash rules)? → .claude/rules/lang.md with frontmatter
   - Domain-specific (frontend/backend patterns)? → .claude/rules/domain/
   - Topic >300 tokens standalone? → Consider .claude/rules/topic.md
   - Personal preferences? → CLAUDE.local.md or ~/.claude/
   - Detailed reference docs? → @import external docs

5. **Universal vs Path-Specific Decision**:

   **Keep in main CLAUDE.md:**
   - Universal conventions applying to ALL files/operations
   - Cohesive conceptual units (Git workflow, Security policies, Style guides)
   - Even if combined total is 200-500 tokens
   - Examples: commit format, pre-commit flow, security exclusions, output formatting

   **Extract to .claude/rules/:**
   - Path/language-specific rules (Python for `*.py`, React for `*.tsx`)
   - Domain-specific patterns (`frontend/`, `backend/`, `infra/`)
   - When single topic exceeds ~300 tokens standalone
   - Examples: `python.md` with `paths: "**/*.py"`, `bash-scripting.md` with `paths: "**/*.sh"`

   **Key principle:** Cohesion and semantic grouping matter more than strict token limits. A well-organized 430-token CLAUDE.md with universal sections (Git 90 + Security 50 + Planning 45 + Style 200 = 385 tokens) is better than fragmenting conceptually related content across multiple files.

6. **Structure content** (token-efficient):
   - Use markdown headings for organization
   - Use tables and bullets over prose
   - Be specific (e.g., "Use 2-space indentation" not "Format code properly")
   - Group related items logically

7. **Include sanity marker** (optional but recommended):
   ```
   sanity check: [random-number]
   ```

8. **Write file** with appropriate sections based on user context

See `reference.md § Templates` for starter examples and `§ Modular Rules` for .claude/rules/ patterns.

## Key Principles

- **Specific over generic**: "Run `npm test`" not "Test the code"
- **Persistent not temporary**: Coding standards yes, current bug no
- **Concise not verbose**: Bullets and tables over paragraphs
- **Modular organization**: Main CLAUDE.md + .claude/rules/ + @imports
- **Path-specific when needed**: Frontmatter with `paths:` glob patterns
- **Secure**: Never include credentials or sensitive data

## MANDATORY Validation (CREATE only)

**STOP**: Before creating new CLAUDE.md, answer YES/NO for each:

- **Q1: Persistent** (not temporary)? [YES/NO]
- **Q2: Frequently referenced** (coding standards, workflows)? [YES/NO]
- **Q3: Concise** (avoid verbose docs)? [YES/NO]
- **Q4: Non-sensitive** (no credentials/tokens)? [YES/NO]

**If ANY answer is NO:**
→ STOP. Explain why inappropriate.
→ Recommend alternatives: README.md (docs), environment variables (secrets), direct request (one-time), .claude/rules/ (detailed guidelines)
→ EXIT immediately.

**If ALL answers are YES:**
→ Proceed to "Creating New CLAUDE.md Files" section above.

---

## Progressive Disclosure

Keep main CLAUDE.md lean (<200 tokens). Distribute content:

**Modular rules** (.claude/rules/ - auto-loaded):
```
.claude/rules/
  |- code-style.md
  |- security.md
  |- frontend/react.md
  |- backend/api.md
```

**Imports** (lazy-loaded when referenced):
```markdown
@README
@docs/architecture.md
@~/.claude/my-project-prefs.md
```

**Reference docs** (external):
```
reference/
  |- runbooks/building.md
  |- standards/conventions.md
```

Use `/memory` command during session to view/edit loaded memories.

## Constraints

- **Instruction budget**: LLMs follow ~150-200 instructions reliably. Claude Code's system prompt uses ~50, leaving ~100 for CLAUDE.md
- **Token target**: Main CLAUDE.md <200 tokens ideal, <500 acceptable for universal cohesive content
- **Universal relevance**: Every line should apply to most sessions, not task-specific work
- **Modular distribution**: Use .claude/rules/ for path/language/domain-specific content, not to fragment universal cohesive sections
- **Cohesion over tokens**: Keep conceptually related universal sections together (Git, Security, Planning, Style) even if combined total is 200-500 tokens

See `reference.md § Content Guidelines` for inclusion/exclusion rules and anti-patterns.

## Validation Checklist

- [ ] Information is persistent and frequently referenced
- [ ] No sensitive credentials or tokens included
- [ ] Content is concise and token-efficient (<200 tokens for main CLAUDE.md)
- [ ] Markdown structure is clear with headings
- [ ] Specific guidelines (not generic advice)
- [ ] Appropriate organization: main vs .claude/rules/ vs @imports
- [ ] Path-specific rules use frontmatter (if applicable)
- [ ] Sanity marker included (optional)

See `reference.md § Templates`, `§ Modular Rules`, and `§ Import Syntax` for detailed examples.
