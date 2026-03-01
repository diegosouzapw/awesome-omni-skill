---
name: anchor-based-context-recovery
description: Use when starting work after context compaction, switching features, or gathering context without re-reading entire files - searches anchor tags to find relevant architecture/pattern documentation
---

# Anchor-Based Context Recovery

## When to Use

**ALWAYS use when:**
- Starting work on a feature/bug after context compaction
- Switching between different features in same session
- User says "work on feature X" or "fix bug Y"
- About to modify code you haven't read in current context

**DO NOT use when:**
- You just read the relevant files in current conversation
- Working on trivial changes (<10 lines)
- User is asking general questions, not requesting implementation

## The Problem

After context compaction or in new sessions:
- Claude loses track of what files were already modified
- Must re-read 500+ line files to understand architecture
- Forgets critical constraints and validation rules
- Wastes context re-learning project structure

## The Solution: Searchable Anchors

Projects embed HTML comment anchors in their documentation:

```markdown
<!-- anchor: validation-rules -->
## Validation Architecture
- Foreign language: Warning only
- AP Calculus: Hard block
- Located: App.jsx:229-474

<!-- anchor: state-management -->
## Core State Structure
- courses array: Primary schedule data
- Located: App.jsx:285-290
```

**Key principle:** Anchors survive file splits and refactoring. Code locations change, but anchor names remain stable.

## Phase 1: Discover Relevant Anchors

**Step 1: Search for anchor documentation**

```bash
# Find all anchor files
find docs/ -name "*anchor*" -o -name "architecture.md" -o -name "context-map.md" 2>/dev/null

# Or search all docs for anchors
grep -r "<!-- anchor:" docs/ 2>/dev/null | head -20
```

**Step 2: Search for task-specific anchors**

If working on feature F012:
```bash
grep -r "anchor:.*F012" docs/ 2>/dev/null
grep -r "anchor:.*refactor" docs/ 2>/dev/null
```

If working on validation bug:
```bash
grep -r "anchor:.*validation" docs/ 2>/dev/null
grep -r "anchor:.*rules" docs/ 2>/dev/null
```

**Step 3: List all available anchors**

```bash
grep -r "<!-- anchor:" docs/ 2>/dev/null | sed 's/.*anchor: //' | sed 's/ -->.*//' | sort -u
```

## Phase 2: Collect Context

**Step 1: Read anchor sections ONLY**

Do NOT read entire files. Use grep with context:

```bash
# Read validation anchor + 20 lines after
grep -A 20 "<!-- anchor: validation-rules -->" docs/architecture/core.md

# Or read multiple related anchors
grep -A 15 "<!-- anchor: state-management -->" docs/architecture/core.md
grep -A 15 "<!-- anchor: credit-calculation -->" docs/architecture/core.md
```

**Step 2: Find related code locations**

Anchors should reference exact file locations. Extract them:

```markdown
<!-- anchor: validation-rules -->
Located: App.jsx:229-474
Related: SchedulingEngine.js:150-200
```

Then use Read tool with those specific line ranges, or use Grep to find the relevant sections.

**Step 3: Search for related patterns**

Based on anchor info, search for related code:

```bash
# If anchor mentions "validation rules", find all validators
grep -r "validate.*course" src/ --include="*.js"

# Find related hooks
grep -r "use.*Validation" src/hooks/
```

## Phase 3: Verify Understanding

**Before making changes, verify:**

1. **Architecture constraints from anchors**
   - File size limits?
   - Required patterns?
   - Validation rules?

2. **Current state from code search**
   - Where is the relevant code?
   - What are the dependencies?
   - Are there existing tests?

3. **Related components from anchor links**
   - What else might be affected?
   - Are there anchor references to related features?

## Phase 4: Document Your Changes (Optional)

**When adding new patterns, suggest anchor additions:**

If you extract validation logic to a new file:
```markdown
<!-- anchor: course-validator -->
## Course Validator Module
- Extracted from: App.jsx:229-474 (Nov 2024)
- Located: src/domain/validators/courseValidator.js
- Used by: App.jsx, SchedulingEngine.js
- Related: validation-rules anchor (original pattern)
```

**When modifying existing patterns, note anchor updates needed:**

Suggest updating the anchor documentation when you make significant changes to referenced code.

## Anchor Naming Conventions

**Good anchor names:**
- `<!-- anchor: validation-rules -->` - Describes the concept
- `<!-- anchor: F012-refactor-plan -->` - References feature ID
- `<!-- anchor: state-management -->` - Clear, searchable
- `<!-- anchor: 300-line-enforcement -->` - References constraint

**Bad anchor names:**
- `<!-- anchor: important -->` - Too vague
- `<!-- anchor: app-stuff -->` - Not searchable
- `<!-- anchor: temp-notes -->` - Implies temporary
- `<!-- anchor: section-3 -->` - Meaningless number

## Expected Workflow Example

```bash
# User says: "Fix the foreign language validation bug"

# Phase 1: Find anchors
grep -r "anchor:.*validation" docs/
# Result: <!-- anchor: validation-rules --> in docs/architecture/core.md

# Phase 2: Collect context
grep -A 30 "<!-- anchor: validation-rules -->" docs/architecture/core.md
# Reads: Validation rules are in App.jsx:229-474

# Use Grep to find the exact code
grep -n "foreign.*language" src/App.jsx

# Phase 3: Verify understanding
# - Rule: Foreign language validation is WARNING only (from anchor)
# - Location: App.jsx:229-293 (from anchor)
# - Current behavior: Need to check implementation

# Phase 4: Make the fix with full context
# - Know where the code is
# - Know the expected pattern
# - Know related constraints from anchors
```

## Benefits

**Compared to re-reading full files:**
- 940-line file → 50 lines of anchor context (95% reduction)
- Survives refactoring (anchors update when files split)
- Searchable by concept, not by file path
- Works across sessions and compaction

**Compared to manual summarization:**
- Consistent structure across features
- Easy to update when code changes
- Machine-readable (grep-able)
- No information loss from summarization

## Common Anchor Patterns

### Pattern 1: Feature Development
```markdown
<!-- anchor: F012-refactor-plan -->
## Feature F012: Extract Validation Logic
- Status: in_progress
- Files to modify: App.jsx (reduce 940→300 lines)
- Extract to: src/domain/validators/
- Tests: tests/validators/
- Related: validation-rules, 300-line-enforcement anchors
```

### Pattern 2: Bug Fixes
```markdown
<!-- anchor: bug-foreign-language-validation -->
## Bug: Foreign Language Prerequisites Not Checking
- Reported: 2024-11-27
- Expected: Warning when skipping levels
- Actual: No warning shown
- Root cause: TBD
- Related: validation-rules anchor
```

### Pattern 3: Architecture Constraints
```markdown
<!-- anchor: 300-line-rule -->
## Architecture Constraint: File Size Limit
- Hard limit: 300 lines per file
- Enforcement: .github/workflows/architecture-check.yml
- Current violations: App.jsx (940 lines)
- Refactor priority: HIGH
- Related: F012-refactor-plan
```

## Checklist

Before claiming you've used this skill:

- [ ] Searched for relevant anchors (`grep -r "anchor:.*keyword" docs/`)
- [ ] Read ONLY anchor sections (not full files)
- [ ] Found code locations from anchor references
- [ ] Searched for related patterns in codebase
- [ ] Verified architecture constraints from anchors
- [ ] Ready to implement without re-reading full context

## Anti-Patterns to Avoid

**DON'T:**
- Read entire files when anchors exist
- Assume anchor content from previous sessions
- Skip anchor search because "I remember this"
- Create anchors for trivial code (<20 lines)
- Use anchors as TODO lists (use feature roadmap instead)

**DO:**
- Search for anchors at start of EVERY task
- Reference exact file locations in anchors
- Link related anchors together
- Keep anchor descriptions concise (20-50 lines)
- Update anchor docs when making significant changes
