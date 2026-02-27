---
name: cursor-rules
description: This skill should be used when creating or editing Cursor rules files (.mdc format) in the .cursor/rules directory. Provides guidance on rule types, frontmatter configuration, effective rule writing patterns, and best practices for structuring AI instructions for Cursor IDE.
---

# Cursor Rules

Guide for creating and editing Cursor rules files (.mdc format) that provide context-aware instructions to the AI in Cursor IDE.

## Overview

Cursor rules are instructions that guide AI behavior in Cursor IDE. Rules use the MDC format (`.mdc`) with YAML frontmatter and markdown content, stored in `.cursor/rules/` directories.

**Key capabilities:**

- Define project-specific coding standards and patterns
- Auto-attach context based on file patterns
- Create reusable instruction components
- Control when and how rules are applied

**Context:** All Cursor rules guidance in this skill applies to `.mdc` files only, not `.cursorrules` (deprecated) or `AGENTS.md` files.

## When to Use This Skill

Use this skill when:

- Creating new Cursor rule files for project standards
- Editing existing rules to improve clarity or effectiveness
- Deciding which rule type to use (Always, Auto Attached, Agent Requested, Manual)
- Structuring rule content for optimal AI comprehension
- Applying AI instruction writing best practices to Cursor rules

## MDC File Format

Cursor rules use MDC format combining YAML frontmatter with markdown content.

### Structure

```yaml
---
description: Brief explanation of rule's purpose
globs: ["**/*.ts", "**/*.tsx"]
alwaysApply: false
---
# Rule Content

Markdown instructions for the AI...
```

### Frontmatter Options

**`description`** (string, required for Agent Requested rules)

- Explains the rule's purpose
- Used by AI to decide if rule is relevant
- Keep concise (1-2 sentences)

```yaml
description: TypeScript coding standards including type safety and error handling patterns
```

**`globs`** (array of strings, optional)

- File patterns that trigger auto-attachment
- Uses standard glob syntax
- Applied when matching files are referenced in chat

```yaml
globs: ["**/*.ts", "**/*.tsx"]  # TypeScript files
globs: ["src/api/**/*"]         # API directory
globs: ["**/*.test.ts"]         # Test files
```

**`alwaysApply`** (boolean, default: false)

- `true`: Rule always included in model context
- `false`: Rule applied based on type and conditions

```yaml
alwaysApply: true   # For critical standards that always apply
alwaysApply: false  # For context-specific guidance
```

## Rule Types

Cursor provides four rule types controlling when rules are applied. Choose the type based on how broadly the rule should apply.

### Always

**When to use:** Critical standards that apply to all AI interactions in the project.

**Configuration:**

```yaml
---
description: Core coding standards
alwaysApply: true
---
```

**Characteristics:**

- Permanently included in model context
- Use sparingly (impacts token budget)
- Best for: Project-wide standards, critical constraints, fundamental patterns

**Example use cases:**

- "Never commit secrets or credentials"
- "All functions must have TypeScript return types"
- "Follow company security review process before external API calls"

### Auto Attached

**When to use:** Context-specific guidance that applies when working with certain file types or directories.

**Configuration:**

```yaml
---
description: React component patterns
globs: ["**/*.tsx", "src/components/**/*"]
alwaysApply: false
---
```

**Characteristics:**

- Applied when glob patterns match referenced files
- Token-efficient (only loads when relevant)
- Best for: File-type standards, directory-specific patterns, framework conventions

**Example use cases:**

- React component patterns (attached for `.tsx` files)
- API endpoint standards (attached for `src/api/**/*`)
- Test conventions (attached for `*.test.ts` files)

### Agent Requested

**When to use:** Guidance that AI should evaluate for relevance based on the task.

**Configuration:**

```yaml
---
description: Database migration patterns for PostgreSQL schema changes
alwaysApply: false
---
# Note: No globs - AI decides based on description
```

**Characteristics:**

- AI determines if rule is relevant to current task
- Requires descriptive `description` field
- Best for: Specialized workflows, domain-specific patterns, optional guidance

**Example use cases:**

- "Database migration patterns for PostgreSQL"
- "Performance optimization techniques for React"
- "Accessibility guidelines for web components"

### Manual

**When to use:** On-demand guidance invoked explicitly in chat.

**Configuration:**

```yaml
---
description: Debugging workflow for production issues
alwaysApply: false
---
# Note: No globs - invoked with @ruleName
```

**Characteristics:**

- Invoked explicitly using `@ruleName` in chat
- Full control over when rule applies
- Best for: Workflows, checklists, specialized procedures

**Example use cases:**

- "@deploy-checklist" - Pre-deployment verification
- "@debug-production" - Production debugging workflow
- "@refactor-guide" - Code refactoring procedures

**Usage:**

```
@debug-production help me investigate the API timeout issue
```

## Writing Effective Rules

Apply AI instruction writing best practices to create clear, actionable Cursor rules.

### Core Principles

**Keep rules under 500 lines**

- Cursor's recommendation for optimal performance
- Decompose large rules into focused, composable components
- Split by domain (e.g., separate rules for testing, API, UI patterns)

**Be specific and actionable**

- Provide concrete patterns and examples
- Avoid vague guidance like "write good code"
- Include code snippets showing expected patterns

**Use progressive disclosure**

- Structure: Overview → Core patterns → Detailed guidance → Examples
- Front-load essential information
- Layer complexity for detailed edge cases

**Include concrete examples**

- Show input/output pairs for patterns
- Use `@filename.ts` syntax to reference existing project files
- Demonstrate both correct and incorrect approaches

### Writing Style

**Imperative/infinitive form** (verb-first), not second person:

```markdown
✓ Good: "Validate input before API requests"
✗ Bad: "You should validate input before you make API requests"
```

**Consistent terminology:**

```markdown
✓ Good: Always use "component" for React components
✗ Bad: Mix "component", "element", "widget" interchangeably
```

**Objective, instructional language:**

```markdown
✓ Good: "Return explicit error types rather than throwing exceptions"
✗ Bad: "Hey, try to return errors instead of throwing them if possible"
```

### Using XML Tags in Rules

XML tags help structure complex rules for clearer parsing. Use them for:

**Multiple examples requiring distinction:**

```markdown
<example type="valid">
const result = await fetchData();
if (!result) {
  return { error: "Not found" };
}
return result;
</example>

<example type="invalid">
// Don't assume success
return (await fetchData()).data;
</example>
```

**Separating context from instructions:**

```markdown
<context>
This project uses Zod for runtime validation with React Query for data fetching.
</context>

<instructions>
When creating API clients:
1. Define Zod schema for response
2. Create React Query hook using the schema
3. Export both for component consumption
</instructions>
```

**Defining templates:**

```markdown
<template>
export const use[EntityName] = () => {
  return useQuery({
    queryKey: ['[entity]'],
    queryFn: async () => {
      const response = await fetch('/api/[endpoint]');
      return [Schema].parse(await response.json());
    }
  });
};
</template>
```

Skip XML tags when markdown structure is sufficient for simple content.

### Reference Project Files

Use `@filename` syntax to reference existing project files as examples:

```markdown
## Component Patterns

Follow the patterns in @src/components/Button.tsx for all interactive components:

- Props interface with TypeScript
- Forwarded refs for accessibility
- Consistent event handler naming
```

This grounds the AI in actual project code rather than abstract descriptions.

## Common Patterns

### Project-Wide Standards (Always)

**Purpose:** Critical standards applying to all code.

```yaml
---
description: TypeScript and code quality standards
alwaysApply: true
---
# Code Standards

## Type Safety

- All functions must have explicit return types
- No `any` types without justification comment
- Use discriminated unions for complex state

## Error Handling

- Return Result<T, E> types for operations that can fail
- Never silently catch and ignore errors
- Log errors with context before re-throwing

## Testing

- All exported functions must have unit tests
- Integration tests for all API endpoints
- Minimum 80% code coverage
```

### Framework Patterns (Auto Attached)

**Purpose:** File-type or directory-specific conventions.

```yaml
---
description: React component patterns and conventions
globs: ["**/*.tsx", "src/components/**/*"]
alwaysApply: false
---

# React Component Standards

## Component Structure

<template>
interface [ComponentName]Props {
  // Props definition
}

export const [ComponentName] = forwardRef<
  HTMLElement,
  [ComponentName]Props
>((props, ref) => {
  // Implementation
});

[ComponentName].displayName = '[ComponentName]';
</template>

## Examples

<example type="valid">
// Interactive component with forwarded ref
interface ButtonProps {
  onClick: () => void;
  children: React.ReactNode;
}

export const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ onClick, children }, ref) => (
    <button ref={ref} onClick={onClick}>
      {children}
    </button>
  )
);

Button.displayName = 'Button';
</example>

<example type="invalid">
// Missing ref forwarding and display name
export const Button = ({ onClick, children }) => (
  <button onClick={onClick}>{children}</button>
);
</example>
```

### Workflow Procedures (Manual)

**Purpose:** Step-by-step procedures invoked on-demand.

```yaml
---
description: Pre-deployment verification checklist
alwaysApply: false
---

# Deployment Checklist

Invoke with: @deploy-checklist

## Pre-Deployment Verification

### 1. Code Quality

- [ ] All tests passing (`npm run test`)
- [ ] No TypeScript errors (`npm run type-check`)
- [ ] Linter passing (`npm run lint`)
- [ ] No console.log or debugger statements

**Validation:** Check CI pipeline status

### 2. Security Review

- [ ] No hardcoded secrets or API keys
- [ ] Environment variables properly configured
- [ ] Dependencies updated (no critical vulnerabilities)

**Validation:** Run `npm audit` and review results

### 3. Database Migrations

- [ ] Migrations tested locally
- [ ] Rollback plan documented
- [ ] Backup completed

**Validation:** Verify migration scripts in `db/migrations/`

### 4. Deployment

Follow standard deployment process:
1. Deploy to staging
2. Run smoke tests
3. Get approval from lead
4. Deploy to production
5. Monitor for 15 minutes
```

### Specialized Guidance (Agent Requested)

**Purpose:** Domain-specific patterns AI loads when relevant.

```yaml
---
description: Performance optimization patterns for React applications
alwaysApply: false
---

# React Performance Optimization

## When to Optimize

Optimize when:
- Components re-render unnecessarily (use React DevTools Profiler)
- User interactions feel laggy (>100ms response)
- Large lists without virtualization

## Optimization Techniques

### Memoization

<instructions>
Use `useMemo` for expensive computations:
- Complex filtering/sorting operations
- Derived state calculations
- Object/array creation in render
</instructions>

<example type="valid">
const filteredItems = useMemo(
  () => items.filter(item => item.category === selectedCategory),
  [items, selectedCategory]
);
</example>

<example type="invalid">
// Recreates array on every render
const filteredItems = items.filter(item =>
  item.category === selectedCategory
);
</example>

### Component Memoization

<instructions>
Use `React.memo` for expensive presentational components:
- Complex rendering logic
- Large component trees
- Components receiving stable props
</instructions>

Refer to @src/components/DataTable.tsx for production example.
```

## Anti-Patterns to Avoid

**❌ Vague, high-level guidance**

```markdown
✗ Bad: "Write clean, maintainable code following best practices"
✓ Good: "Extract functions exceeding 50 lines into smaller units with single responsibilities"
```

**❌ Overly broad Always rules**

```markdown
✗ Bad: 500-line rule with alwaysApply: true covering all coding standards
✓ Good: Focused 100-line rule for critical standards + separate Auto Attached rules for context-specific patterns
```

**❌ Missing concrete examples**

```markdown
✗ Bad: "Use proper error handling"
✓ Good: Show specific error handling pattern with code example
```

**❌ Inconsistent terminology**

```markdown
✗ Bad: Mix "API endpoint", "route", "path", "URL" for same concept
✓ Good: Choose "API endpoint" and use consistently
```

**❌ Conversational language**

```markdown
✗ Bad: "Hey, when you're working with APIs, make sure you handle errors properly, okay?"
✓ Good: "Handle API errors by returning Result<T, Error> types"
```

**❌ Temporal references**

```markdown
✗ Bad: "Use the new authentication system" or "Recently refactored to async/await"
✓ Good: "Use OAuth 2.0 authentication" (evergreen, specific)
```

**❌ No glob patterns for context-specific rules**

```markdown
✗ Bad: React-specific rule without globs (always loads unnecessarily)
✓ Good: React rule with globs: ["**/*.tsx"] (loads only when relevant)
```

## Quality Checklist

Before finalizing a Cursor rule:

**Format & Structure:**

- [ ] Uses `.mdc` format with YAML frontmatter
- [ ] Saved in `.cursor/rules/` directory
- [ ] Frontmatter includes required fields for rule type
- [ ] Rule under 500 lines (decompose if longer)

**Rule Type Configuration:**

- [ ] Appropriate rule type chosen (Always/Auto Attached/Agent Requested/Manual)
- [ ] `alwaysApply: true` only for critical project-wide standards
- [ ] `globs` configured for Auto Attached rules
- [ ] `description` clear and specific for Agent Requested rules

**Content Quality:**

- [ ] Imperative/infinitive form (not second person)
- [ ] Consistent terminology throughout
- [ ] Concrete examples with code snippets
- [ ] References to project files using `@filename` syntax
- [ ] Progressive disclosure (overview → details)

**Specificity:**

- [ ] Actionable patterns, not vague guidance
- [ ] Specific constraints and requirements
- [ ] No temporal references ("new", "old", "recently")
- [ ] Clear anti-patterns identified

**Token Efficiency:**

- [ ] Large rules decomposed into focused components
- [ ] Auto Attached rules use appropriate globs
- [ ] Always rules limited to essential standards
- [ ] No redundant information

## Example: Complete Rule File

Comprehensive example showing best practices:

```yaml
---
description: API client patterns using Zod schemas and React Query hooks
globs: ["src/api/**/*", "**/*-client.ts"]
alwaysApply: false
---

# API Client Patterns

## Overview

This project separates API clients (Zod schemas + React Query hooks) from UI components.

**Pattern:** Schema definition → Client function → React Query hook → Component consumption

## Client Structure

<template>
// 1. Define Zod schema
const [Entity]Schema = z.object({
  id: z.string(),
  // ... fields
});

type [Entity] = z.infer<typeof [Entity]Schema>;

// 2. Create client function
async function fetch[Entity](id: string): Promise<[Entity]> {
  const response = await fetch(`/api/[entity]/${id}`);
  return [Entity]Schema.parse(await response.json());
}

// 3. Create React Query hook
export function use[Entity](id: string) {
  return useQuery({
    queryKey: ['[entity]', id],
    queryFn: () => fetch[Entity](id),
  });
}
</template>

## Error Handling

<instructions>
All client functions must handle errors consistently:
1. Parse responses with Zod (throws on invalid data)
2. Catch network errors and re-throw with context
3. React Query handles loading/error states
</instructions>

<example type="valid">
async function fetchUser(id: string): Promise<User> {
  try {
    const response = await fetch(`/api/users/${id}`);
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }
    const data = await response.json();
    return UserSchema.parse(data);
  } catch (error) {
    console.error('Failed to fetch user:', { id, error });
    throw error;
  }
}
</example>

<example type="invalid">
// Missing error context and response validation
async function fetchUser(id: string): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  return UserSchema.parse(await response.json());
}
</example>

## Reference Implementation

See @src/api/user-client.ts for production example implementing this pattern.

## Validation

Before completing API client:
- [ ] Zod schema covers all response fields
- [ ] Client function includes error handling
- [ ] React Query hook properly configured
- [ ] Types exported for component consumption
```

---

**Remember:** Effective Cursor rules are specific, actionable, and scoped appropriately. When in doubt, create focused rules with Auto Attached globs rather than broad Always rules.
