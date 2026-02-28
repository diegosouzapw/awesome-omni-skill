---
name: creating-skills
description: Creates new Agent Skills with proper YAML frontmatter, progressive disclosure architecture, and best practices. Use when the user asks to create a new skill, generate a skill template, or build custom capabilities.
---

# Creating Skills

This skill helps you create new Claude Agent Skills following Anthropic's official specifications and best practices.

This skill includes comprehensive resources:
- **BEST_PRACTICES.md**: Detailed authoring guidelines
- **REFERENCE.md**: Technical specifications and detailed examples
- **README.md**: Overview and quick start guide
- **templates/**: Ready-to-use skill templates
- **examples/**: Sample skills for reference
- **scripts/validate_skill.py**: Skill validation tool

## When to Use This Skill

Invoke this skill when the user:
- Asks to create a new Claude skill
- Wants to generate a skill template
- Needs help structuring a custom capability
- Requests skill scaffolding or boilerplate
- Wants to validate an existing skill

## Skill Creation Workflow

### Step 1: Gather Requirements

Ask the user for:

1. **Skill name**: What should the skill be called?
   - Lowercase letters, numbers, hyphens only
   - Maximum 64 characters
   - Use gerund form (e.g., "processing-data", "analyzing-logs")
   - Avoid vague names like "helper" or "utils"

2. **Skill description**: What does the skill do and when should it be used?
   - Maximum 1024 characters
   - Write in third person
   - Include specific triggers
   - Format: "[What it does]. Use when [conditions]."

3. **Skill complexity**: Simple or complex?
   - Simple: Single SKILL.md file (under 300 lines)
   - Complex: SKILL.md + REFERENCE.md + FORMS.md + scripts

4. **Core functionality**: What are the main tasks?

### Step 2: Choose and Copy Template

**Simple Template** (for focused, single-file skills):
```bash
cp -r templates/simple-skill-template/ ../your-skill-name/
```

**Complex Template** (for multi-file skills with extensive docs):
```bash
cp -r templates/complex-skill-template/ ../your-skill-name/
```

See REFERENCE.md section "Template Selection Guide" for detailed criteria.

### Step 3: Customize the Template

**Edit SKILL.md frontmatter:**
```yaml
---
name: your-skill-name
description: What it does. Use when triggers.
---
```

**Fill in required sections:**
1. When to Use This Skill
2. Core functionality with steps
3. Common patterns
4. Error handling
5. Examples
6. Validation checklist

**For Complex Skills:**
- Edit REFERENCE.md for technical details
- Edit FORMS.md for output templates
- Create scripts for automation

See REFERENCE.md section "Customization Guide" for detailed instructions.

### Step 4: Validate the Skill

Run the validation script:

```bash
python scripts/validate_skill.py ../your-skill-name/SKILL.md --strict
```

The validator checks:
- YAML frontmatter syntax and fields
- Name format (lowercase-with-hyphens, ≤64 chars, gerund form)
- Description (≤1024 chars, includes triggers)
- No reserved words ("anthropic", "claude")
- File structure and references
- Best practices compliance

See REFERENCE.md section "Validation Tool" for detailed usage.

### Step 5: Review Best Practices

Before finalizing:

```bash
cat BEST_PRACTICES.md
```

Key principles:
- **Conciseness**: Only include non-standard information
- **Progressive Disclosure**: Keep SKILL.md under 500 lines
- **Freedom Levels**: Match specificity to task fragility
- **Consistent Terminology**: Use same terms throughout
- **Validation Steps**: Include checklists for complex workflows

See BEST_PRACTICES.md for complete guidelines.

## Frontmatter Requirements

Every SKILL.md must start with:

```yaml
---
name: skill-name
description: Clear description. Use when triggers.
---
```

**Quick Rules:**
- `name`: lowercase-with-hyphens, ≤64 chars, gerund form
- `description`: ≤1024 chars, includes "Use when..."
- No XML tags or reserved words

See REFERENCE.md section "Frontmatter Specifications" for complete rules and examples.

## Progressive Disclosure

Skills load in three levels:

**Level 1 - Metadata** (~100 tokens, always loaded):
- YAML frontmatter for skill discovery

**Level 2 - Instructions** (<5,000 tokens, triggered):
- Main SKILL.md content

**Level 3 - Resources** (on-demand):
- REFERENCE.md, FORMS.md, scripts
- Load only when referenced

**Example:**
```markdown
## Basic Processing
[Instructions for common case]

## Advanced Techniques
See REFERENCE.md section "Advanced Methods" for details.
```

## Example Skills

**Simple Skill**: Code Reviewer (examples/simple-skill-example/)
- Single SKILL.md file
- Clear workflow with checklists
- ~350 lines

**Complex Skill**: Data Analyzer (examples/complex-skill-example/)
- SKILL.md + REFERENCE.md + FORMS.md + scripts
- Statistical methods in REFERENCE.md
- Report templates in FORMS.md

View examples:
```bash
cat examples/simple-skill-example/SKILL.md
cat examples/complex-skill-example/SKILL.md
```

## Quick Start

**1. Choose template based on complexity:**
   - Simple: Single focused task, <300 lines
   - Complex: Multiple features, needs extensive docs

**2. Copy template:**
   ```bash
   cp -r templates/simple-skill-template/ ../my-skill/
   ```

**3. Edit frontmatter and fill sections:**
   - Replace all [placeholders]
   - Add specific examples
   - Create validation checklist

**4. Validate:**
   ```bash
   python scripts/validate_skill.py ../my-skill/SKILL.md --strict
   ```

**5. Review BEST_PRACTICES.md and test**

## Validation Checklist

Before using a new skill:

**Frontmatter:**
- [ ] Valid name (gerund form, ≤64 chars)
- [ ] Description with triggers (≤1024 chars)
- [ ] No prohibited content

**Content:**
- [ ] "When to Use This Skill" section
- [ ] Core functionality with steps
- [ ] Examples included
- [ ] Validation checklist
- [ ] Concise (no unnecessary info)

**Structure:**
- [ ] Progressive disclosure applied
- [ ] SKILL.md under 500 lines (or split)
- [ ] Referenced files exist

**Testing:**
- [ ] Passed validator
- [ ] Functionally tested
- [ ] Works as expected

## Common Patterns

**Pattern 1: Simple Single-File Skill**
- Use simple-skill-template
- Focus on one capability
- Include 2-3 examples
- Add validation checklist

**Pattern 2: Complex Multi-File Skill**
- Use complex-skill-template
- SKILL.md: High-level workflow with references
- REFERENCE.md: Technical details, algorithms
- FORMS.md: Output templates
- scripts/: Automation utilities

**Pattern 3: Skill with Automation**
- Include scripts/ directory
- Document script usage in SKILL.md
- Scripts execute without loading to context
- Use only pre-installed packages

## Runtime Constraints

Remember when designing skills:

- ❌ No network access or external API calls
- ❌ No runtime package installation
- ✅ Only pre-installed packages
- ✅ Scripts execute via bash without context loading
- ✅ Progressive disclosure minimizes context usage

## Error Handling

**Common Issue: Validation Fails**
- Check YAML syntax
- Verify name format (lowercase-with-hyphens)
- Ensure description includes "Use when..."
- Remove any reserved words

**Common Issue: Skill Too Long**
- Split into SKILL.md + REFERENCE.md
- Move technical details to REFERENCE.md
- Move templates to FORMS.md
- Keep SKILL.md under 500 lines

**Common Issue: References Not Found**
- Ensure referenced files exist
- Use relative paths
- Check file names match exactly

## Additional Resources

**Internal:**
- `REFERENCE.md`: Technical specs, detailed examples, troubleshooting
- `BEST_PRACTICES.md`: Complete authoring guidelines
- `README.md`: Quick start and overview
- `templates/`: Ready-to-use templates
- `examples/`: Working sample skills

**External:**
- [Claude Skills Documentation](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview)
- [Skills Cookbook](https://github.com/anthropics/claude-cookbooks/tree/main/skills)

## Getting Help

**Review templates:**
```bash
cat templates/simple-skill-template/SKILL.md
cat templates/complex-skill-template/SKILL.md
```

**Study examples:**
```bash
cat examples/simple-skill-example/SKILL.md
cat examples/complex-skill-example/SKILL.md
```

**Read best practices:**
```bash
cat BEST_PRACTICES.md
```

**Check technical specs:**
```bash
cat REFERENCE.md
```

**Validate your skill:**
```bash
python scripts/validate_skill.py ../your-skill/SKILL.md --strict
```

For detailed technical specifications, troubleshooting, and comprehensive examples, see REFERENCE.md.
