---
name: rules-migration
description: MIGRATE CLAUDE.md into modular `.claude/rules/` directory structure following Claude Code's rules system. Converts monolithic CLAUDE.md into organized, path-specific rule files with glob patterns. Use when migrating to rules system, modularizing project instructions, splitting CLAUDE.md, organizing memory files. Triggers on "migrate claudemd to rules", "convert claude.md to rules", "modularize claude.md", "split claude.md into rules", "migrate to rules system".
---

# Refactor CLAUDE.md to Rules

Converts a monolithic CLAUDE.md file into Claude Code's modular `.claude/rules/` directory structure. This enables better organization, path-specific rules, and progressive disclosure of project instructions.

## Trigger Phrases

Users invoke this skill by saying:
- "migrate claudemd to rules"
- "convert claude.md to rules"
- "modularize claude.md"
- "split claude.md into rules"
- "migrate to rules system"
- "organize claude.md into rules"

## When to Use This Skill

Use this skill when:
- Migrating from CLAUDE.md to `.claude/rules/` system
- CLAUDE.md has become too large or complex
- Need path-specific rules for different parts of codebase
- Want better organization of project instructions
- Multiple team members need different rule subsets

## Core Principles

1. **Content Preservation**: Never lose content during conversion
2. **Logical Grouping**: Group related instructions into cohesive files
3. **Path Specificity**: Apply rules only where relevant using glob patterns
4. **Progressive Disclosure**: Keep core rules concise; detailed content in subdirectories
5. **Maintainability**: Make rules easy to find and update

## Workflow

### Step 1: Read and Analyze CLAUDE.md

**Actions:**
- Read the complete CLAUDE.md file
- Identify major sections by headers (##, ###)
- Categorize content by concern (code style, testing, security, frontend, backend, etc.)
- Detect path references (src/api/, frontend/, *.ts patterns)
- Note any execution rules, non-negotiables, critical protocols

**What to expect:**
- Large file with multiple concerns mixed together
- Headers like "Code Quality", "Testing", "Security", "API Guidelines"
- References to specific directories or file patterns

**Common Sections to Look For:**
- Execution rules / orchestration
- Code style and conventions
- Testing requirements
- Security protocols
- Frontend guidelines
- Backend/API guidelines
- Git/commit standards
- Environment setup
- Stack configuration

### Step 2: Design Rules Directory Structure

**Actions:**
- Create logical grouping plan based on sections
- Identify which rules are path-specific vs global
- Determine subdirectory organization (frontend/, backend/, etc.)
- Plan file names using kebab-case

**Decision Points:**
- If section references specific paths → Create path-specific rule with frontmatter
- If section applies globally → Create global rule without frontmatter
- If section is large (200+ lines) → Consider subdirectory with multiple files
- If section is small (< 50 lines) → Combine with related content

**Standard Directory Pattern:**
```
.claude/rules/
├── core.md                # Core execution rules (orchestration, agents)
├── code-style.md          # Language-agnostic code conventions
├── testing.md             # Testing requirements
├── security.md            # Security protocols
├── git.md                 # Git and commit standards
├── stack.md               # Tech stack defaults
├── frontend/
│   ├── react.md           # React conventions (path: src/**/*.tsx)
│   └── styles.md          # CSS/styling rules (path: **/*.css, **/*.scss)
└── backend/
    ├── api.md             # API guidelines (path: src/api/**/*.ts)
    └── database.md        # Database rules (path: src/db/**/*.ts)
```

### Step 3: Extract and Create Rule Files

**Actions:**
- Create `.claude/rules/` directory
- Create subdirectories for organized concerns (frontend/, backend/)
- Extract each section into appropriate file
- Add YAML frontmatter for path-specific rules
- Preserve formatting and structure

**YAML Frontmatter Format:**
```yaml
---
paths: src/api/**/*.ts
---
```

**Glob Pattern Examples:**
- `**/*.ts` - All TypeScript files
- `src/api/**/*` - Everything under src/api/
- `{src,lib}/**/*.tsx` - TSX files in src or lib
- `*.md` - Markdown files in root only

**Content Organization Tips:**
- Start each file with clear purpose statement
- Keep most important rules at the top
- Use headers (##, ###) for subsections
- Include examples inline for quick reference
- Link to detailed documentation if needed

### Step 4: Handle Special Cases

**Non-Negotiable Rules:**
- Create `core.md` for critical execution rules
- Keep these concise and scannable
- Use clear numbering and headers

**Large Sections:**
- Split into multiple files in subdirectory
- Example: `frontend/components.md`, `frontend/hooks.md`, `frontend/state.md`

**Cross-Cutting Concerns:**
- Security rules might apply to both frontend and backend
- Options:
  - Create global `security.md`
  - Or create both `frontend/security.md` and `backend/security.md` with specific rules

**Environment-Specific Rules:**
- Create `development.md` and `production.md` if needed
- Or use path patterns to scope deployment-specific rules

### Step 5: Verify the Migration

**Actions:**
- Compare line count: Original CLAUDE.md vs total rules files
- Check all headers/sections are accounted for
- Validate YAML frontmatter syntax
- Test glob patterns match intended files
- Verify no content was lost or duplicated

**Validation Checklist:**
```bash
# Check rules are loaded
ls -la .claude/rules/

# Verify frontmatter syntax (no errors when Claude loads)
# Claude Code will validate on next invocation

# Compare content (line counts should be similar)
wc -l .claude/CLAUDE.md
find .claude/rules/ -name "*.md" -exec wc -l {} + | tail -1
```

### Step 6: Archive Original CLAUDE.md

**Actions:**
- Move original CLAUDE.md to `.claude/CLAUDE.md.backup`
- Add a note explaining the migration in README or .claude/README.md
- Commit the changes with descriptive message

**Commit Message Format:**
```
refactor(rules): migrate CLAUDE.md to modular rules system

- Split monolithic CLAUDE.md into organized rule files
- Created .claude/rules/ with core, frontend, backend structure
- Added path-specific rules using glob patterns
- Preserved all original content

BREAKING: CLAUDE.md moved to .claude/rules/ directory
```

## Quick Example

**Before (CLAUDE.md):**
```markdown
# Claude Code Configuration

## Code Quality Checks
Before marking ANY task complete:
1. Run getDiagnostics on all modified files
2. Fix ALL linting/type errors

## React Guidelines
Use functional components with hooks.
Props should be typed with interfaces.

## API Development
All endpoints must have error handling.
Use Zod for request validation.
```

**After (.claude/rules/):**
```
.claude/rules/
├── core.md
│   ## Code Quality Checks
│   Before marking ANY task complete:
│   1. Run getDiagnostics on all modified files
│   2. Fix ALL linting/type errors
│
├── frontend/
│   └── react.md
│       ---
│       paths: src/**/*.tsx
│       ---
│       # React Guidelines
│       Use functional components with hooks.
│       Props should be typed with interfaces.
│
└── backend/
    └── api.md
        ---
        paths: src/api/**/*.ts
        ---
        # API Development
        All endpoints must have error handling.
        Use Zod for request validation.
```

## Success Indicators

This skill is successful when:
- [ ] All content from CLAUDE.md preserved in rules files
- [ ] Rules are logically organized by concern
- [ ] Path-specific rules have valid YAML frontmatter
- [ ] Glob patterns correctly target intended files
- [ ] Original CLAUDE.md is backed up
- [ ] Changes are committed with descriptive message

## Quality Checklist

Before marking complete, verify:
- [ ] `.claude/rules/` directory exists
- [ ] All rule files use `.md` extension
- [ ] YAML frontmatter syntax is valid (no syntax errors)
- [ ] Glob patterns follow documented syntax
- [ ] No duplicate content across files
- [ ] File names are descriptive and kebab-case
- [ ] Core rules are in root, specialized in subdirectories

## Common Issues & Solutions

See [reference.md](./reference.md) for:
- Glob pattern syntax and examples
- YAML frontmatter structure
- Directory organization patterns
- Migration strategies for complex CLAUDE.md files

See [examples.md](./examples.md) for:
- Complete before/after conversions
- Real-world CLAUDE.md migrations
- Path-specific rule examples
- Complex glob pattern use cases

## Next Steps After Completion

1. Restart Claude Code to load new rules system
2. Verify rules are applied correctly by checking Claude's behavior
3. Iterate on organization if needed (rules can be moved/renamed)
4. Document the new structure in project README
5. Update team documentation about rules location

## Integration Points

**Works with:**
- Claude Code's memory system (auto-loads `.claude/rules/*.md`)
- Path-specific rule matching (uses glob patterns)
- Progressive disclosure (subdirectories for detailed rules)

**Related Skills:**
- `context-management` - for managing project context
- `documentation-suite` - for documenting the rules structure

---

*rules-migration | Created: 2025-12-10 | Updated: 2025-12-10*
