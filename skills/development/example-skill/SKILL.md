---
name: example-skill
version: "1.0.0"
description: "An example skill template showing the SKILL.md format. Use this when you need to understand skill structure."
license: "MIT"
dependencies:
  - typescript
---

# Example Skill

This is an example skill that demonstrates the SKILL.md format used by Claude and GitHub Copilot.

## When to Use This Skill

Use this skill when:
- You need to understand the skill file format
- You're creating a new skill
- You want to see best practices for skill documentation

## Core Instructions

1. **Always use YAML frontmatter** at the top of the file with at least `name` and `description`
2. **Write clear, actionable instructions** in the markdown body
3. **Include examples** to demonstrate expected behavior
4. **List guidelines** for consistent results

## Examples

### Good Skill Instruction
```markdown
When creating a new React component:
1. Use functional components with hooks
2. Define props interface with TypeScript
3. Export as named export for better tree-shaking
```

### Bad Skill Instruction
```markdown
Make good components.
```

## Guidelines

- Keep instructions specific and actionable
- Include code examples where helpful
- Reference related skills in dependencies
- Use consistent formatting throughout
- Test the skill with various prompts

## Resources

You can include additional files in a `resources/` subdirectory:
- Scripts for automation
- Templates for code generation
- Configuration files

## Related Skills

- typescript - For TypeScript-specific patterns
- react - For React component patterns
- testing - For testing best practices
