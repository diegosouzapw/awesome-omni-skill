---
name: skill-manager
description: Manages Claude Skills lifecycle - creating, updating, maintaining, and versioning project-specific and cross-IDE knowledge bases. Invoke when user wants to create new skills, update existing ones, or sync skill content across tools.
---

# Skill Manager - Claude Skills Lifecycle Management

You are an expert in managing Claude Skills, which are project-specific knowledge bases that provide specialized guidance. This skill helps you create, update, maintain, and version skills effectively.

## What are Claude Skills?

Claude Skills are markdown-based knowledge modules stored in `.claude/skills/` that:

- Provide domain-specific expertise (frameworks, tools, patterns)
- Are automatically invoked based on trigger descriptions
- Can reference supporting documentation files
- Enable consistent, expert-level guidance across sessions
- Support project-specific best practices and patterns

## When to Use This Skill

Invoke this skill when the user:

- Wants to **create a new skill** from documentation or knowledge
- Needs to **update an existing skill** with new information
- Asks to **maintain or refactor skills** for better organization
- Wants to **version control skills** or track changes
- Needs to **validate skill structure** and completeness
- Asks about **skill best practices** or conventions
- Wants to **migrate knowledge** into skill format

## Skill Structure

### Required Files

Every skill must have at minimum:

```
.claude/skills/<skill-name>/
└── SKILL.md                    # Main skill definition (REQUIRED)
```

### Recommended Files

Well-documented skills should include:

```
.claude/skills/<skill-name>/
├── SKILL.md                    # Main definition with triggers
├── README.md                   # Documentation about the skill
├── quick-reference.md          # Quick lookups and cheat sheets
├── guide.md                    # Comprehensive deep-dive
└── examples.md                 # Code examples and patterns
```

### SKILL.md Format

The main skill file must follow this structure:

```markdown
---
name: skill-name
description: Clear description that triggers invocation. Mention key terms that should activate this skill.
---

# Skill Title - One Line Summary

You are an expert in [domain]. This skill provides [purpose].

## What is [Topic]?

Brief introduction to what this skill covers.

## When to Use This Skill

Invoke this skill when the user:

- Lists specific triggers
- Mentions key concepts
- Asks about related topics

## Core Concepts

### Main Concept 1
Explanation...

### Main Concept 2
Explanation...

## Common Patterns

Practical examples and code templates...

## Best Practices

Guidelines and recommendations...

## Instructions for Assistance

When helping users with [topic]:

1. Step-by-step guidance approach
2. What to consider
3. How to validate

## Additional Resources

References to other skill files or external docs...
```

## Creating a New Skill

### Step 1: Identify the Domain

Determine:
- **What expertise** does this skill provide?
- **What triggers** should invoke it? (frameworks, tools, patterns, file types)
- **What scope** - project-specific or general knowledge?
- **What audience** - beginner, intermediate, or expert?

### Step 2: Gather Source Material

Collect:
- Official documentation
- Project conventions and patterns
- Common pitfalls and solutions
- Code examples and templates
- Best practices and anti-patterns

### Step 3: Structure the Content

Organize into:

**SKILL.md** (Main file):
- Clear, concise overview
- Core concepts (2-5 key ideas)
- Common patterns and templates
- Best practices
- When to use guidance

**quick-reference.md** (Optional):
- Cheat sheets
- Quick lookup tables
- Code templates
- Command references

**guide.md** (Optional):
- Comprehensive explanations
- Architecture and design
- Advanced topics
- Detailed examples

**examples.md** (Optional):
- Real-world use cases
- Complete code examples
- Step-by-step tutorials

### Step 4: Write Effective Triggers

The `description` field is critical - it determines when the skill activates.

**Good triggers:**
```yaml
description: Expert in React hooks and state management. Invoke when user asks about useState, useEffect, custom hooks, or React component patterns.
```

**Bad triggers:**
```yaml
description: Helps with React.
```

**Key principles:**
- Mention specific terms users will say
- Include framework/tool names
- List key concepts and APIs
- Mention file patterns if relevant

### Step 5: Validate and Test

After creating the skill:

1. **Check structure**: Ensure SKILL.md exists with proper frontmatter
2. **Test triggers**: Try sample questions that should invoke it
3. **Review content**: Is it clear, concise, and actionable?
4. **Check links**: Do references to other files work?
5. **Test examples**: Do code snippets run correctly?

## Updating Existing Skills

### When to Update

Update skills when:
- New versions of tools/frameworks release
- Best practices evolve
- Common issues are discovered
- User feedback reveals gaps
- Project conventions change

### Update Process

1. **Identify changes**:
   ```bash
   # Check skill age
   ls -lah .claude/skills/<skill-name>/

   # Review recent framework changes
   # Check official docs for updates
   ```

2. **Document updates**:
   - Note what changed (version, features, deprecations)
   - Identify affected sections
   - Plan content additions/modifications

3. **Update content**:
   - Modify SKILL.md for core changes
   - Update examples with new patterns
   - Add new best practices
   - Deprecate old patterns (but explain why)

4. **Validate**:
   - Test updated code examples
   - Verify documentation links
   - Check for contradictions
   - Test skill invocation

5. **Version tracking** (optional):
   - Add version note to README.md
   - Document change history
   - Note source documentation versions

### Update Template

Add version information to README.md:

```markdown
## Version History

### v2.0 - 2025-01-09
- Updated for Framework v5.0
- Added new pattern: [pattern name]
- Deprecated: [old pattern] (use [new pattern] instead)
- Source: Framework v5.0 docs

### v1.0 - 2024-11-08
- Initial skill creation
- Source: Framework v4.0 docs
```

## Maintaining Skills

### Regular Maintenance Tasks

**Monthly:**
- Review for outdated information
- Check external documentation links
- Update examples if dependencies changed

**Quarterly:**
- Compare with official docs for changes
- Review invocation patterns - are they working?
- Gather user feedback
- Consider splitting large skills

**Annually:**
- Major refactor if needed
- Archive deprecated content
- Consolidate overlapping skills

### Skill Quality Checklist

- [ ] Clear, specific description with good triggers
- [ ] Core concepts are concise (< 5 main topics)
- [ ] Examples are tested and working
- [ ] Best practices are current
- [ ] No contradictions with other skills
- [ ] Links to external resources work
- [ ] README documents the skill purpose
- [ ] Code examples have proper syntax
- [ ] Appropriate scope (not too broad/narrow)

### Refactoring Large Skills

When a skill grows too large (> 1000 lines), consider:

**Option 1: Split by Subtopic**
```
react-hooks/              →  react-state-hooks/
  SKILL.md                    SKILL.md (useState, useReducer)
  (too large)
                              react-effect-hooks/
                                SKILL.md (useEffect, useLayoutEffect)

                              react-custom-hooks/
                                SKILL.md (creating custom hooks)
```

**Option 2: Separate by Level**
```
django/                   →  django-basics/
  SKILL.md                    SKILL.md (getting started)
  (too comprehensive)
                              django-advanced/
                                SKILL.md (advanced patterns)
```

**Option 3: Extract Reference Material**
```
api-guide/                →  api-guide/
  SKILL.md (huge)             SKILL.md (concise overview)
                                reference.md (detailed API docs)
                                examples.md (code samples)
```

## Best Practices

### Content Guidelines

**Do:**
- ✅ Use clear, concise language
- ✅ Provide working code examples
- ✅ Include common pitfalls
- ✅ Show both good and bad patterns
- ✅ Link to official documentation
- ✅ Use consistent formatting
- ✅ Test all code examples
- ✅ Keep scope focused

**Don't:**
- ❌ Copy entire documentation sites
- ❌ Include untested code
- ❌ Make skills too broad
- ❌ Duplicate information across skills
- ❌ Use vague trigger descriptions
- ❌ Include deprecated patterns without context
- ❌ Create overlapping skills

### Naming Conventions

**Skill directory names:**
- Use lowercase with hyphens: `skill-name`
- Be specific: `textual` not `tui-framework`
- Avoid version numbers: `django` not `django-5`

**File names:**
- `SKILL.md` - Always uppercase (required)
- `README.md` - Uppercase (convention)
- Other files: lowercase with hyphens

### Documentation Standards

Each skill should document:
- **Purpose**: What expertise does it provide?
- **Scope**: What's included and excluded?
- **Prerequisites**: Required knowledge or tools
- **Version**: What version of tools/frameworks it covers
- **Last updated**: When was it last reviewed?
- **Sources**: Where information came from

## Converting Documentation to Skills

### From Official Docs

**Process:**
1. Identify core concepts (5-10 key topics)
2. Extract common patterns and examples
3. Note best practices and anti-patterns
4. Create concise summaries
5. Link to official docs for details

**Example:**
```
Official Docs (100 pages)  →  Skill Structure:
├── Getting Started             SKILL.md (core concepts)
├── Core Concepts              quick-reference.md (patterns)
├── API Reference              guide.md (architecture)
├── Advanced Topics            + links to official docs
└── Examples
```

### From Internal Knowledge

**Process:**
1. Document team conventions
2. Capture tribal knowledge
3. Record common solutions
4. Include project-specific patterns
5. Add context for decisions

**Example: Project Patterns Skill**
```markdown
---
name: project-patterns
description: Project-specific conventions and patterns for vcr-tui. Invoke when working on this codebase.
---

## Our Conventions

### File Organization
[How we structure code...]

### Testing Strategy
[Our testing approach...]

### Common Patterns
[Patterns we use repeatedly...]
```

## Integration with Other Tools

Skills can be referenced by other tools:

### For Documentation

**docs/DEVELOPMENT.md:**
```markdown
## Development Guides

For detailed guidance, see Claude Skills in `.claude/skills/`:

- **Textual Framework**: `.claude/skills/textual/` - TUI development patterns
- **Git Hooks**: `.claude/skills/hk/` - Pre-commit hook management
- **Project Patterns**: `.claude/skills/project-patterns/` - Our conventions
```

### For Cursor/Copilot

Create `.cursorrules` or similar that references skills:

```markdown
# Project Context

This project uses Claude Skills for detailed guidance.
See `.claude/skills/` for:

- textual: Textual TUI framework patterns
- hk: Git hook management

When suggesting code, follow patterns documented in these skills.
```

### For CI/CD

Reference skills in CI configuration:

```yaml
# .github/workflows/validate.yml
- name: Validate Skills
  run: |
    # Check skill structure
    python scripts/validate_skills.py
```

## Skill Templates

### Minimal Skill Template

```markdown
---
name: skill-name
description: Brief description with trigger keywords
---

# Skill Name

You are an expert in [topic]. This skill provides [purpose].

## When to Use This Skill

Invoke when the user asks about [triggers].

## Core Concepts

### Concept 1
[Explanation...]

## Common Patterns

[Code examples...]

## Best Practices

[Guidelines...]
```

### Comprehensive Skill Template

See `.claude/skills/textual/SKILL.md` for a complete example of a well-structured skill with:
- Clear triggers
- Supporting files (quick-reference.md, guide.md)
- Multiple sections with depth
- Code examples and patterns
- Testing guidance

## Troubleshooting

### Skill Not Invoking

**Check:**
1. Is `name` in frontmatter lowercase with hyphens?
2. Does `description` include specific trigger words?
3. Is the skill in `.claude/skills/<name>/SKILL.md`?
4. Are there typos in the frontmatter YAML?

**Fix:**
```bash
# Verify structure
ls -la .claude/skills/<skill-name>/SKILL.md

# Check frontmatter format
head -10 .claude/skills/<skill-name>/SKILL.md
```

### Overlapping Skills

**Problem:** Multiple skills trigger for the same query

**Solution:**
- Make descriptions more specific
- Narrow skill scope
- Split broad skills into focused ones
- Use clear boundaries in descriptions

### Outdated Content

**Problem:** Skill contains deprecated information

**Solution:**
- Add version tracking to README
- Mark deprecated sections clearly
- Update with current best practices
- Document migration paths

## Example: Creating a Skill from Scratch

**Scenario:** Create a pytest skill for this project

**Step 1: Gather information**
```bash
# Check what testing framework is used
grep -r "pytest" . --include="*.toml" --include="*.txt"

# Look at existing tests
find . -name "test_*.py" | head -5
```

**Step 2: Create structure**
```bash
mkdir -p .claude/skills/pytest-patterns
```

**Step 3: Write SKILL.md**
```markdown
---
name: pytest-patterns
description: Expert in pytest testing patterns and best practices. Invoke when writing tests, using fixtures, or debugging test failures.
---

# Pytest Testing Patterns

You are an expert in pytest testing for Python applications...

[Continue with core concepts, patterns, etc.]
```

**Step 4: Add supporting files**
- `quick-reference.md`: Common fixture patterns, assertion examples
- `examples.md`: Real test examples from the project

**Step 5: Test**
- Ask: "How do I write a pytest fixture?"
- Verify skill is invoked
- Check if guidance is helpful

## Instructions for Using This Skill

When helping users manage skills:

1. **Assess current state**: Check existing skills and their quality
2. **Understand goals**: What knowledge needs to be captured?
3. **Plan structure**: Which format best fits the content?
4. **Create/update systematically**: Follow the templates and processes
5. **Validate thoroughly**: Test invocation and content quality
6. **Document clearly**: Explain what was done and why

Always consider:
- Is this the right level of detail?
- Will the triggers work effectively?
- Is the content maintainable long-term?
- Does it integrate with existing skills?
- Is it accessible to the intended audience?

## Additional Resources

For examples of well-structured skills, see:
- `.claude/skills/textual/` - Comprehensive framework skill
- `.claude/skills/hk/` - Tool-specific skill with detailed reference

## Version Tracking

This skill documents Claude Skills format as of January 2025. The format may evolve with future Claude Code releases.
