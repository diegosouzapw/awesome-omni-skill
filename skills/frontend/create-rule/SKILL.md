---
name: create-rule
description: Create rules files for file-scoped AI instructions. Interactive wizard for .claude/rules/ or .cursor/rules/ files with proper frontmatter.
---

# Create Rule File

## Purpose

Rules files provide **file-scoped AI instructions** that automatically activate when you read, write, or edit files matching specific patterns. This wizard guides users through creating properly formatted rule files.

## How Rules Work

1. Rule files live in `.claude/rules/` or `.cursor/rules/` directories
2. Each rule file has YAML frontmatter specifying which files trigger it
3. When you touch a matching file, the rule content is injected into context
4. The AI then follows those instructions for that specific file

## Wizard Process

Follow these steps **in order**, asking **one question at a time** using the `askquestion` tool. Wait for the user's response before proceeding to the next step.

---

### Step 1: Rule Location

Ask where the rule should be saved:

```
Use askquestion with these options:
- .claude/rules/ (project) - "Project-specific rules in .claude/rules/"
- .cursor/rules/ (project) - "Project-specific rules in .cursor/rules/ (Cursor compatibility)"
- ~/.claude/rules/ (global) - "User-level rules that apply to all projects"
```

Store the chosen location for later.

---

### Step 2: Rule Format

Ask which frontmatter format to use:

```
Use askquestion with these options:
- claude (recommended) - "Claude Code format using 'paths:' field - simpler and official"
- cursor - "Cursor format using 'globs:' array with optional 'description' field"
```

**Claude Code format** is recommended as it's the official format.

---

### Step 3: File Patterns

Ask what files should trigger this rule. Offer common patterns:

```
Use askquestion with these options:
- test - "Test files: **/*.test.ts, **/*.spec.ts"
- api - "API files: src/api/**/*.ts, **/api/**/*.ts"
- components - "Component files: src/components/**/*.tsx, **/*.component.ts"
- docs - "Documentation: **/*.md, docs/**/*"
- styles - "Style files: **/*.css, **/*.scss, **/*.styled.ts"
- config - "Config files: *.config.ts, *.config.js, .env*"
- custom - "Custom pattern (I'll type my own)"
```

If user selects "custom", ask them to type the glob pattern(s).

**Important**: After getting patterns, ask if they want to add more patterns. Keep asking until they say no. Collect all patterns into an array.

---

### Step 4: Rule Name

Suggest a filename based on the patterns chosen, or let user provide custom name:

```
Use askquestion with options like:
- testing.md (if test patterns)
- api-guidelines.md (if API patterns)
- component-rules.md (if component patterns)
- code-style.md (general)
- custom - "Custom name (I'll type my own)"
```

Ensure the name ends with `.md`.

---

### Step 5: Rule Type

Ask what kind of instructions this rule should contain:

```
Use askquestion with these options:
- code-style - "Code style & formatting conventions"
- testing - "Testing patterns, frameworks, and best practices"
- security - "Security requirements and vulnerability prevention"
- performance - "Performance optimization guidelines"
- documentation - "Documentation and comment requirements"
- architecture - "Architecture patterns and design principles"
- error-handling - "Error handling and logging practices"
- accessibility - "Accessibility (a11y) requirements"
- custom - "Custom (I'll describe what I need)"
```

---

### Step 6: Rule Details

Based on the rule type selected, ask follow-up questions to gather specifics:

**For code-style:**
- Indentation preference (spaces/tabs, size)
- Quote style (single/double)
- Naming conventions
- Import ordering

**For testing:**
- Testing framework (vitest, jest, pytest, etc.)
- Testing patterns (AAA, BDD, etc.)
- Coverage requirements
- Mocking preferences

**For security:**
- Input validation requirements
- Authentication/authorization patterns
- Sensitive data handling
- Specific vulnerabilities to prevent

**For performance:**
- Optimization targets (bundle size, runtime, memory)
- Caching strategies
- Lazy loading requirements

**For documentation:**
- Comment style (JSDoc, docstrings, etc.)
- Required sections
- Example requirements

**For architecture:**
- Design patterns to follow
- Layer boundaries
- Dependency rules

**For error-handling:**
- Error types to use
- Logging requirements
- Recovery strategies

**For accessibility:**
- WCAG level target
- Specific requirements (keyboard nav, screen readers, etc.)

**For custom:**
- Ask user to describe what instructions they need

Use `askquestion` for structured choices or let user type freely for open-ended details.

---

### Step 7: Generate & Write

Now generate the rule file:

1. **Construct the frontmatter** based on format choice:

   **Claude Code format:**
   ```markdown
   ---
   paths: pattern1, pattern2, pattern3
   ---
   ```

   **Cursor format:**
   ```markdown
   ---
   globs: ["pattern1", "pattern2", "pattern3"]
   description: "<brief description based on rule type>"
   ---
   ```

2. **Generate rule content** based on type and details gathered. Write clear, actionable instructions.

3. **Create the directory** if it doesn't exist:
   ```bash
   mkdir -p <chosen-location>
   ```

4. **Write the file** using the Write tool.

5. **Confirm success** by showing the user:
   - The full path of the created file
   - The complete file contents
   - A reminder that the rule will activate when they touch matching files

---

## Frontmatter Reference

### Claude Code Format (Recommended)

```markdown
---
paths: **/*.test.ts, **/*.spec.ts
---

Your rule content here...
```

- Uses `paths:` field with comma-separated patterns
- Simpler syntax, official Claude Code format
- No additional fields

### Cursor Format

```markdown
---
globs: ["**/*.test.ts", "**/*.spec.ts"]
description: "Testing guidelines for all test files"
---

Your rule content here...
```

- Uses `globs:` field with JSON array
- Optional `description:` field for documentation
- More verbose but allows description

---

## Example Generated Rules

### Testing Rule (Claude Code format)

```markdown
---
paths: **/*.test.ts, **/*.spec.ts, tests/**/*
---

# Testing Guidelines

When writing or modifying tests:

1. **Framework**: Use Vitest for all tests
2. **Pattern**: Follow AAA (Arrange, Act, Assert) pattern
3. **Naming**: Use descriptive names: "should [action] when [condition]"
4. **Isolation**: Mock external dependencies, never call real APIs
5. **Coverage**: Include edge cases - empty inputs, null values, boundaries
6. **Assertions**: One primary assertion per test when possible
```

### API Security Rule (Cursor format)

```markdown
---
globs: ["src/api/**/*.ts", "**/routes/**/*.ts", "**/controllers/**/*.ts"]
description: "Security requirements for API endpoints"
---

# API Security Requirements

All API endpoints must:

1. **Input Validation**: Validate all inputs with Zod schemas
2. **Authentication**: Verify JWT tokens before processing
3. **Authorization**: Check user permissions for the resource
4. **Rate Limiting**: Apply rate limits to prevent abuse
5. **Error Handling**: Never expose internal errors to clients
6. **Logging**: Log all requests with correlation IDs (no sensitive data)
```

### Component Accessibility Rule

```markdown
---
paths: src/components/**/*.tsx, **/*.component.tsx
---

# Accessibility Requirements

All components must meet WCAG 2.1 AA:

1. **Keyboard Navigation**: All interactive elements must be keyboard accessible
2. **Focus Management**: Visible focus indicators, logical tab order
3. **ARIA Labels**: Provide aria-label for icon-only buttons
4. **Color Contrast**: Minimum 4.5:1 for normal text, 3:1 for large text
5. **Screen Readers**: Test with VoiceOver/NVDA, use semantic HTML
6. **Motion**: Respect prefers-reduced-motion for animations
```

---

## Glob Pattern Reference

| Pattern | Matches |
|---------|---------|
| `**/*.ts` | All .ts files at any depth |
| `src/**/*.ts` | All .ts files under src/ |
| `*.config.ts` | Config files in root only |
| `**/*.{ts,tsx}` | Both .ts and .tsx files |
| `tests/**/*` | All files under tests/ |
| `!**/*.test.ts` | Exclude test files (negation) |

---

## After Creation

Remind the user:

1. The rule is now active for matching files
2. They can edit the rule anytime to refine instructions
3. Rules from closer directories take precedence over parent directories
4. They can create multiple rules for different file patterns
