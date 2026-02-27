---
name: artifact-creation
description: "Guide the creation of artifacts (rules, skills, commands, subagents) from patterns. Use templates appropriately, ensure proper structure, add cross-references, and update documentation. Use when creating new artifacts from extracted patterns."
---

# Artifact Creation Skill

Guide the creation of artifacts from patterns, ensuring proper structure, cross-referencing, and documentation.

## When to Use

- Use when creating artifacts from extracted patterns
- Use when converting patterns to reusable artifacts
- Use when setting up new artifacts with proper structure
- Use when ensuring artifacts follow standards

## Instructions

### Creation Process

1. **Select template**: Choose appropriate template (rule/skill/command/subagent)
2. **Determine type**: Confirm artifact type is correct
3. **Create structure**: Use template to create proper structure
4. **Add content**: Fill in pattern-specific content
5. **Add cross-references**: Link to related artifacts
6. **Update documentation**: Add to relevant documentation
7. **Validate**: Verify artifact follows standards

### Template Usage

#### For Rules

- Use `.cursor/templates/rule-template.md`
- Include proper frontmatter (description, alwaysApply, globs)
- Follow rule structure and formatting
- Add cross-references section

#### For Skills

When creating a **skill**, you MUST use the [skill-creator](.cursor/skills/skill-creator/SKILL.md) skill first. Follow its guidance for SKILL.md anatomy, frontmatter (name, description), progressive disclosure, and when to add scripts/references/assets. Then use `.cursor/templates/skill-template.md` for this repo’s structure and cross-references.

- Create folder structure: `skill-name/SKILL.md`
- Apply skill-creator before filling content

#### For Commands

- Use `.cursor/templates/command-template.md`
- Create markdown file with command structure
- Include Overview, Steps, Checklist sections
- Add cross-references section

#### For Subagents

- Use `.cursor/templates/subagent-template.md`
- Create markdown file with frontmatter
- Include proper configuration (model, readonly, is_background)
- Write clear, focused prompt

### Structure Validation

Ensure artifacts have:
- Proper frontmatter/metadata
- Clear structure and organization
- Appropriate sections for type
- Cross-references to related artifacts
- Examples where helpful

### Cross-Reference Addition

When creating artifacts:
1. **Identify related artifacts**: Find rules, skills, commands, subagents that relate
2. **Add forward references**: Include in new artifact
3. **Add backward references**: Update related artifacts
4. **Link to documentation**: Add documentation links

### Documentation Updates

When creating artifacts:
1. **Update category docs**: Add to category documentation if applicable
2. **Update master index**: Add to master artifact index
3. **Update usage guides**: Add to relevant usage documentation
4. **Update examples**: Add to example sections

## Examples

### Example 1: Creating a Rule

**Pattern**: TypeScript strict mode preference

**Creation**:
1. Use rule template
2. Create `.cursor/rules/organization/typescript-strict-mode.mdc`
3. Add frontmatter with description
4. Add guidelines for TypeScript strict mode
5. Add cross-references to TypeScript-related artifacts
6. Update organization documentation

### Example 2: Creating a Command

**Pattern**: Code review checklist workflow

**Creation**:
1. Use command template
2. Create `.cursor/commands/meta/code-review-checklist.md`
3. Add overview and steps
4. Add checklist items
5. Add cross-references to review-related artifacts
6. Update commands documentation

## Best Practices

- Always use templates for consistency
- Follow naming and organization standards
- Add cross-references from the start
- Update documentation immediately
- Validate structure before finalizing
- Test artifacts after creation

## Related Artifacts

- [Skill Creator](.cursor/skills/skill-creator/SKILL.md) – Use when creating skills (anatomy, frontmatter, progressive disclosure)
- [Artifact Creation Rule](.cursor/rules/meta/artifact-creation.mdc)
- [Organization Rule](.cursor/rules/meta/organization.mdc)
- [Cross-Referencing Rule](.cursor/rules/meta/cross-referencing.mdc)
- [Create Artifact Command](.cursor/commands/meta/create-artifact.md)
- [Artifact Creator Subagent](.cursor/agents/meta/artifact-creator.md)
- [Templates](.cursor/templates/)
