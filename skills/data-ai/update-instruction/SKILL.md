---
name: update-instruction
description: Create, update, or manage universal-ai-config instruction templates. Handles finding existing instructions, deciding whether to create or modify, and writing the template.
userInvocable: false
---

# Manage Instruction Templates

Instructions are persistent context and rules that apply to AI conversations, scoped by file patterns or always-on.

## Finding the Right Instruction

### 1. Search existing instructions

List files in `<%= instructionTemplatePath() %>/` and read their frontmatter (`description`, `globs`) to understand what each covers and its scope.

### 2. Match the user's intent to existing files

Look for instructions that already cover the same topic or a closely related topic. Consider:

- **Exact match**: an instruction about the same subject exists → update it
- **Partial overlap**: the topic fits within the scope of a broader instruction → add to it rather than creating a separate file
- **Multiple candidates**: several instructions touch on the topic → pick the one whose `globs` and purpose align best with the user's intent

### 3. Determine scope for new instructions

If no existing instruction fits, investigate the project to decide where the instruction belongs:

1. **Search the codebase** for how the topic is used — find which files and directories are relevant
2. **Cross-reference with the user's wording** — the instruction may apply broadly (e.g. "always validate env vars") or narrowly (e.g. "feature flags in API routes should use env vars")
3. **Choose the right scoping**:
   - If the rule is universal across the project → use `alwaysApply: true`
   - If it applies to a specific area → use `globs` matching only the relevant files/directories
   - If the topic appears in many places but the instruction only makes sense for a subset, scope to that subset

**Example:** The user says "feature flags should be loaded from env vars." Feature flags might appear in 10 places across the codebase, but if only the API layer loads them from config, the right scope is `globs: ["src/api/**"]` rather than `alwaysApply: true`.

## Deciding What to Do

- **Create new**: when the topic is distinct from all existing instructions
- **Update existing**: when an instruction already covers the topic but needs changes — modify its content or frontmatter
- **Add per-target override**: when a frontmatter field needs different values per target, use the override object syntax:
  ```yaml
  description:
    claude: Claude-specific description
    copilot: Copilot-specific description
    default: Default description
  ```
- **Delete**: when an instruction is obsolete or fully superseded by another

## Creating a New Instruction

1. Create a `.md` file in `<%= instructionTemplatePath() %>/` with a descriptive name (e.g. `error-handling.md`)
2. Add YAML frontmatter with at minimum a `description`
3. Write the instruction body

### Frontmatter Fields

See the **Instructions** section in `<%= instructionPath('uac-template-guide') %>` for the complete field reference. Key fields: `description`, `globs`, `alwaysApply`, `excludeAgent`.

### When to use `alwaysApply` vs `globs`

- Use `alwaysApply: true` for project-wide conventions that should always be active
- Use `globs` to scope instructions to specific file types or directories (e.g. `["src/api/**"]` for API-specific rules)
- If neither is set, the instruction may still be applied by some targets based on relevance

### Example

```markdown
---
description: TypeScript coding conventions
globs: ["**/*.ts", "**/*.tsx"]
---

Follow these TypeScript conventions:

- Use strict mode
- Prefer interfaces over type aliases for object shapes
- Use explicit return types on exported functions
```
