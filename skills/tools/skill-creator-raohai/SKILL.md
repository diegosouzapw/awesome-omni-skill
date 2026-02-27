---
name: skill-creator
description: Guides users through creating new Claude Code skills using the agent-skill-npm-boilerplate template. Use when users want to create a new skill, build a skill, make a skill, or learn how to create skills.
allowed-tools: Bash, Read, Edit, Write, WebFetch
---

# Skill Creator

A meta skill that helps you create new Claude Code skills using the [agent-skill-npm-boilerplate](https://github.com/RaoHai/agent-skill-npm-boilerplate) template.

## Instructions

When a user asks to create a skill, guide them through this systematic process:

### Step 1: Create Project from Template

1. **Guide them to use the GitHub template**:
   - Visit https://github.com/RaoHai/agent-skill-npm-boilerplate
   - Click "Use this template" button
   - Or go directly to /generate

2. **Or help them clone**:
   ```bash
   git clone https://github.com/RaoHai/agent-skill-npm-boilerplate.git [skill-name]
   cd [skill-name]
   rm -rf .git
   git init
   ```

### Step 2: Gather Requirements

Ask the user these essential questions:

**About the Skill:**
- What should the skill do? (Get clear description of purpose)
- When should it trigger? (What keywords or phrases?)
- What are the main use cases?

**About Tools:**
- Does it need to read files? (Read, Grep, Glob)
- Execute commands? (Bash)
- Modify files? (Edit, Write)
- Access web? (WebFetch)

**About Packaging:**
- npm scope or no scope?
  - With scope: `@mycompany/skill-name`
  - Without: `skill-name`
- Skill name (lowercase with hyphens only, max 64 chars)

**About Target Tools:**
- Which AI tools do they use?
  - Claude Code (always enabled)
  - Cursor?
  - Windsurf?
  - Other?

### Step 3: Customize Configuration

Help them update these files:

#### package.json
```json
{
  "name": "@user-org/skill-name",
  "description": "[Clear description]",
  "author": "[Name <email>]",
  "keywords": ["claude-code", "skill", "[relevant-keywords]"],
  "repository": {
    "url": "https://github.com/[username]/[repo].git"
  }
}
```

#### .claude-skill.json
```json
{
  "name": "skill-name",
  "package": "@user-org/skill-name",
  "targets": {
    "claude-code": { "enabled": true },
    "cursor": { "enabled": [true/false] }
  }
}
```

### Step 4: Write SKILL.md (MOST IMPORTANT)

Help them create a comprehensive SKILL.md:

**Frontmatter (CRITICAL):**
```yaml
---
name: skill-name
description: [What it does] and when to use it. Use when [scenarios]. Include trigger keywords.
allowed-tools: [Tools it needs]
---
```

**✅ Good descriptions:**
- "Generates conventional commit messages from git diffs. Use when writing commits, creating commit messages, or reviewing staged changes."
- "Analyzes Python code for type errors. Use when checking Python types, debugging type issues, or validating .py files."

**❌ Bad descriptions:**
- "Helps with files" (too vague)
- "A useful tool" (no keywords)

**Description Guidelines:**
1. Start with action verb + what it does
2. Add "Use when [scenarios]"
3. Include natural keywords users would say
4. Be specific about file types, actions, domains
5. Keep under 1024 characters

**Main Content Structure:**
```markdown
# Skill Name

[Brief introduction]

## Instructions

When [user scenario]:

1. **First Step**: [Action]
   - [Details]

2. **Second Step**: [Action]
   - [Details]

3. **Final Step**: [Result]
   - [Output format]

## Examples

### Example 1: [Scenario]

User asks: "[Question]"

What happens:
1. [Step]
2. [Step]
3. [Result]

## Best Practices

- [Practice 1]
- [Practice 2]
```

### Step 5: Add Supporting Files (If Needed)

For complex skills, use progressive disclosure:

**reference.md** - Detailed docs (loaded only when needed):
- Complete API reference
- Advanced patterns
- Configuration options

**examples.md** - More examples:
- Real-world scenarios
- Edge cases
- Integration patterns

**scripts/** - Utility scripts (zero-context execution):
- Validation logic
- Data processing

Link from SKILL.md:
```markdown
For details, see [reference.md](reference.md)
```

**Keep SKILL.md under 500 lines!**

### Step 6: Test Locally

Help them test:

```bash
# Test installation
npm test

# Or manually
node install-skill.js

# Verify installed
ls -la ~/.claude/skills/[skill-name]/
cat ~/.claude/skills/[skill-name]/SKILL.md
```

**In Claude Code:**
- Ask: "What skills are available?"
- Verify skill appears
- Test with trigger phrases
- Check tool access
- Verify behavior

**Testing checklist:**
- [ ] Appears in skills list
- [ ] Triggers on keywords
- [ ] Has tool access
- [ ] Works as intended
- [ ] Clear documentation

### Step 7: Publish to npm

Guide publishing:

```bash
# One-time setup
npm login

# Publish (scoped packages need --access public)
npm publish --access public

# Verify
npm view @user-org/skill-name
```

**Pre-publish checklist:**
- [ ] All placeholders replaced
- [ ] README updated
- [ ] Tested locally
- [ ] Git committed
- [ ] Version tagged

### Step 8: Share and Iterate

Encourage them to:
- Share with community
- Gather feedback
- Improve iteratively
- Contribute examples

## Common Skill Patterns

### Pattern 1: File Analyzer

**When**: Analyze specific file types

```yaml
name: file-analyzer
description: Analyzes [file-type] for [purpose]. Use when checking [file-type], analyzing [aspect], or validating [file-type] files.
allowed-tools: Read, Grep, Glob
```

**Structure:**
1. Find files
2. Read and parse
3. Analyze
4. Report findings

### Pattern 2: Code Generator

**When**: Generate code/docs

```yaml
name: code-generator
description: Generates [artifact] from [input]. Use when creating [thing], generating [stuff], or scaffolding [structure].
allowed-tools: Read, Write, Bash
```

**Structure:**
1. Gather requirements
2. Read templates
3. Generate content
4. Write output
5. Verify

### Pattern 3: Workflow Automation

**When**: Automate processes

```yaml
name: workflow-helper
description: Automates [workflow]. Use when [action]ing [thing], running [process], or executing [workflow].
allowed-tools: Bash, Read, Edit
```

**Structure:**
1. Check preconditions
2. Execute steps
3. Verify completion

## Troubleshooting

### Skill Not Triggering

**Cause**: Poor description

**Fix:**
1. Add more trigger keywords to description
2. Make description more specific
3. Test with exact keywords
4. Or ask explicitly: "Use the [skill-name] skill"

### Tool Access Issues

**Cause**: Missing from allowed-tools

**Fix:**
1. Add to `allowed-tools` in frontmatter
2. Or remove field to allow all tools

### Installation Fails

**Cause**: Config errors

**Fix:**
1. Validate .claude-skill.json (must be valid JSON)
2. Check name consistency across files
3. Verify file permissions
4. Run `node install-skill.js` for details

## Best Practices

1. **Start Simple**: Basic version first, enhance later
2. **Clear Descriptions**: Critical for skill discovery
3. **Test Thoroughly**: Various inputs and edge cases
4. **Document Well**: Good docs attract users
5. **Progressive Disclosure**: SKILL.md < 500 lines
6. **Semantic Versioning**: Follow semver
7. **Gather Feedback**: Improve from real usage

## Quick Reference Checklist

**Skill Creation:**
- [ ] Clone template
- [ ] Update package.json
- [ ] Update .claude-skill.json
- [ ] Write SKILL.md (focus on description!)
- [ ] Add examples
- [ ] Test locally
- [ ] Commit to git
- [ ] Publish to npm
- [ ] Test installation
- [ ] Share!

**SKILL.md Essentials:**
- [ ] Clear, keyword-rich description
- [ ] Correct name (lowercase, hyphens, <64 chars)
- [ ] Appropriate allowed-tools
- [ ] Step-by-step instructions
- [ ] Concrete examples
- [ ] Best practices
- [ ] Links to reference docs (if needed)

## Resources

- [Boilerplate](https://github.com/RaoHai/agent-skill-npm-boilerplate)
- [Official Docs](https://code.claude.com/docs/en/skills)
- [Best Practices](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/best-practices)
- [Multi-Tool Support](https://github.com/RaoHai/agent-skill-npm-boilerplate/blob/main/MULTI-TOOL-SUPPORT.md)

---

**Remember**: The `description` field is everything. Spend time making it clear, specific, and keyword-rich. This single field determines when Claude uses the skill!
