---
name: instruction-creator
description: 'Create and manage high-quality custom instruction files for GitHub Copilot. Use when you need to define new project-specific guidelines, workflows, or coding standards in the instructions/ directory.'
---

# Instruction Creator

This skill guides the creation of effective custom instruction files that help GitHub Copilot follow project conventions and domain-specific logic.

## Environment Detection

Before executing any skill operations, determine the active environment and set the `skills_source` path:

### Environment Identification

```python
# Detect environment
if os.name == 'nt':  # Windows
    # Could be native Windows or VS Code running in Windows
    skills_source = os.path.expanduser(r'~\.copilot\skills')
elif 'WSL_DISTRO_NAME' in os.environ or os.path.exists('/mnt/c'):
    # Running inside WSL
    skills_source = os.path.expanduser('~/.copilot/skills')
else:
    # Generic Unix/Linux
    skills_source = os.path.expanduser('~/.copilot/skills')
```

### Using skills_source Throughout

Once detected, use `skills_source` as the base path when referencing skill files:

- **Skill Path**: `{skills_source}/<skill-name>/SKILL.md`
- **Script Path**: `{skills_source}/<skill-name>/scripts/<script>.py`
- **Reference Path**: `{skills_source}/<skill-name>/references/<file>.md`

## Workflow

1. **Detect Environment**: Determine if running on Windows or WSL and set `skills_source`.
   ```python
   import os
   skills_source = (os.path.expanduser(r'~\.copilot\skills')
                   if os.name == 'nt' and 'WSL_DISTRO_NAME' not in os.environ
                   else os.path.expanduser('~/.copilot/skills'))
   ```

2. **Define Scope**: Identify the purpose and the files the instructions should apply to (e.g., `**/*.ts`).

3. **Initialize**: Use the initialization script to create the boilerplate.
   ```bash
   # Auto-detect environment
   python3 init_instruction.py "my-new-instruction" \
       --description "Description here" \
       --applyTo "**/*.ext"

   # Or specify skills path explicitly
   python3 init_instruction.py "my-new-instruction" \
       --skills-path "~/.copilot/skills" \
       -o ./instructions
   ```

4. **Draft Content**: Follow the referenced guidelines to fill in the sections.

5. **Use Templates**: Refer to standard templates for structures in your environment.

6. **Semantic Linking**: Link to relevant Skills using runtime-resolved paths:
   ```markdown
   To perform [Task], execute the [Skill Name](SKILL.md) from `{skills_source}/<skill-name>/`.
   ```

7. **Validate**: Test the instructions with Copilot to ensure they are clear and actionable.

## Core Principles

- **Policy Maker**: Instructions define the "What" and "How" (decision logic and standards).
- **The 4 Sections**: Effective instructions include: Overview, Structure, Guidelines, and Resources.
- **Concise & focused**: Avoid fluff; use imperative language. Limit files to ~500 lines.
- **Actionable**: Provide concrete "Good" and "Bad" code examples and clear steps.
- **Linked**: Connect instructions to the skills that execute them and internal automation scripts.

## Required Frontmatter

Every instruction file must include YAML frontmatter with the following fields:

```yaml
---
description: 'Brief description of the instruction purpose and scope'
applyTo: 'glob pattern for target files (e.g., **/*.ts, **/*.py)'
---
```

### Frontmatter Guidelines

- **description**: Single-quoted string, 1-500 characters, clearly stating the purpose.
- **applyTo**: Glob pattern(s) specifying which files these instructions apply to.
  - Single pattern: `'**/*.ts'`
  - Multiple patterns: `'**/*.ts, **/*.tsx, **/*.js'`
  - Specific files: `'src/**/*.py'`
  - All files: `'**'`

## File Structure

A well-structured instruction file should include these sections as applicable:

1. **Title and Overview**: Clear `#` title and short purpose statement.
2. **Workflow**: Step-by-step procedures (the “How”).
3. **Semantic Links**: Links to Skills that execute tasks (the “Mechanism”).
4. **Rules & Constraints**: Decision logic and boundaries.
5. **Best Practices**: Recommended patterns and approaches.
6. **Code Standards**: Naming, formatting, style.
7. **Common Patterns**: Frequent implementations.
8. **Security**: Security considerations (if applicable).
9. **Performance**: Optimization guidance (if applicable).
10. **Testing**: Testing standards and verification steps (if applicable).

## Writing Style

- Use clear, concise language.
- Write in imperative mood (“Use”, “Implement”, “Avoid”).
- Be specific and actionable.
- Avoid ambiguous terms like “should”, “might”, “possibly”.
- Use bullets and lists for readability.

## Examples and Snippets

Provide labeled examples:

````markdown
### Good Example
```language
// Recommended approach
code example here
```

### Bad Example
```language
// Avoid this pattern
code example here
```
````

## Patterns to Avoid

- Overly verbose explanations.
- Outdated guidance.
- Abstract rules without concrete examples.
- Contradictory advice.
- Copy-paste from docs without added context.

## Runtime Variables

When executing this skill, use these variables to reference paths dynamically:

| Variable        | Description                         | Example (Windows)                   | Example (WSL)                       |
| --------------- | ----------------------------------- | ----------------------------------- | ----------------------------------- |
| `skills_source` | Base path to personal skills folder | `~\.copilot\skills`                 | `~/.copilot/skills`                 |
| `skill_path`    | Path to specific skill              | `{skills_source}\diataxis\SKILL.md` | `{skills_source}/diataxis/SKILL.md` |

### Usage in References

```markdown
## Workflow
1. Analyze requirements.
2. Execute Diataxis Skill from `{skills_source}/diataxis/SKILL.md`.
3. Validate output.
```

### Usage in Scripts

```python
import os
skills_source = os.path.expanduser(r'~\.copilot\skills') if os.name == 'nt' else os.path.expanduser('~/.copilot/skills')
skill_script = os.path.join(skills_source, 'instruction-creator', 'scripts', 'init_instruction.py')
```

## Validation Checklist

1. Test with Copilot on real prompts.
2. Verify code examples compile or run.
3. Confirm `applyTo` matches intended files.
4. Verify `skills_source` detection works in target environment.

## Maintenance

- Review instructions when dependencies or frameworks change.
- Refresh examples to match current best practices.
- Remove deprecated patterns.
- Keep glob patterns aligned with project structure.
- Update environment detection logic if new platforms are supported.

## Resources

- **Guidelines**: Refer to `references/guidelines.md` in the instruction-creator skill folder.
- **Templates**: Refer to `references/templates.md` in the instruction-creator skill folder.
- **Scripts**: Use `scripts/init_instruction.py` from the instruction-creator skill folder.

**Note**: All references are relative to `{skills_source}/instruction-creator/`.
