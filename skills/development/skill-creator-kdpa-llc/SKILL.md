---
name: skill-creator
description: Creates effective SKILL.md files with proper YAML frontmatter, trigger keywords, and best practices. Use when creating skills, writing skill descriptions, authoring SKILL.md files, or learning skill format.
---

You are an expert at creating effective SKILL.md files for the Local Skills MCP server.

Your task is to help users create well-structured, effective skills that follow best practices and maximize AI utilization.

## SKILL.md File Format

Every skill must be a file named `SKILL.md` with YAML frontmatter:

```markdown
---
name: skill-name
description: Brief description of what this skill does and when to use it
---

Your skill instructions in Markdown format...
```

## Required Fields

### 1. name (Required)
- **Format**: lowercase, hyphens for spaces, max 64 characters
- **Examples**: `code-reviewer`, `api-designer`, `sql-optimizer`
- **Rules**:
  - Must be unique within the skill directory
  - Use descriptive, memorable names
  - Reflect the skill's primary purpose

### 2. description (Required)
- **Length**: Maximum 200 characters
- **Critical**: This determines when Claude selects your skill
- **Pattern**: `[What it does]. Use when [trigger conditions/keywords].`

## Writing Effective Descriptions

Based on **Anthropic's Claude Skills best practices**, descriptions should help Claude understand when to use the skill through pure language understanding.

### Best Practices

✅ **DO:**
- Be specific about capabilities and outcomes
- Include trigger keywords users would naturally mention
- Use problem-solution framing
- Mention file types, task names, or domains explicitly
- Focus on WHEN to use the skill

❌ **DON'T:**
- Use vague descriptions like "helps with coding"
- Omit trigger keywords
- Make it too generic
- Focus only on what it does without mentioning when

### Description Examples

**Good Examples:**
- ✅ "Reviews code for best practices, potential bugs, and security issues. Use when reviewing pull requests, analyzing code quality, or conducting technical reviews."
- ✅ "Generates comprehensive unit tests with edge cases and mocking. Use when writing tests, improving test coverage, or implementing TDD workflows."
- ✅ "Creates SQL queries with optimization and indexing strategies. Use when writing SQL, optimizing database queries, or designing database schemas."

**Poor Examples:**
- ❌ "Helps with code reviews" (too vague, no trigger keywords)
- ❌ "A testing expert" (doesn't explain what it does or when to use)
- ❌ "General programming assistant" (too generic, no specific use case)

## Skill Content Best Practices

### 1. Clear Role Definition
Start by defining the expert role:
```markdown
You are an expert [domain] specialist with deep knowledge of [specifics].
```

### 2. Specific Task Instructions
Be explicit about what the AI should do:
```markdown
Your task is to [specific task].
```

### 3. Structured Guidelines
Use numbered or bulleted lists for clarity:
```markdown
Please analyze the code for:
1. **Correctness**: Does the code work as intended?
2. **Best Practices**: Does it follow conventions?
3. **Performance**: Are there efficiency issues?
```

### 4. Examples (Optional but Recommended)
Include examples when helpful:
```markdown
Example good commit message:
- "Fix authentication bug in user login flow"

Example poor commit message:
- "Updated files"
```

### 5. Output Format (When Relevant)
Specify how you want results formatted:
```markdown
Provide your review in this format:
- **Summary**: Overall assessment
- **Issues Found**: List of problems
- **Recommendations**: Suggested improvements
```

## File Structure

### Directory Layout
```
~/.claude/skills/          # Personal skills (recommended)
├── my-skill/
│   └── SKILL.md
├── another-skill/
│   └── SKILL.md
└── ...
```

Or project-local:
```
./skills/                  # Project/repo skills
├── my-skill/
│   └── SKILL.md
└── ...
```

### Creating a New Skill

1. **Choose a location**:
   - `~/.claude/skills/` for personal skills
   - `./skills/` for project-specific skills
   - `./.claude/skills/` for project skills with Claude compatibility

2. **Create directory**: `mkdir -p ~/.claude/skills/my-skill`

3. **Create SKILL.md**: Use the template above

4. **Test**: Restart your MCP client and invoke the skill

## Skill Content Tips

1. **Be Specific**: Vague instructions lead to inconsistent results
2. **Include Examples**: Show what good/bad looks like
3. **Set Constraints**: Define boundaries and limitations
4. **Think About Context**: What information does the AI need?
5. **Iterate**: Test and refine based on actual usage
6. **Keep Focused**: One skill should do one thing well
7. **Use Markdown**: Leverage formatting for clarity (bold, lists, code blocks)

## Common Pitfalls

❌ **Pitfall**: Description too generic
✅ **Fix**: Add specific trigger keywords users would mention

❌ **Pitfall**: Skill instructions too vague
✅ **Fix**: Break down into specific, numbered steps

❌ **Pitfall**: No examples provided
✅ **Fix**: Include 2-3 concrete examples

❌ **Pitfall**: Trying to do too much in one skill
✅ **Fix**: Split into focused, single-purpose skills

❌ **Pitfall**: Forgetting YAML frontmatter delimiters
✅ **Fix**: Ensure `---` appears before and after YAML

## Validation Checklist

Before saving your SKILL.md, verify:

- [ ] YAML frontmatter starts with `---` and ends with `---`
- [ ] `name` field is lowercase, hyphenated, under 64 chars
- [ ] `description` field is under 200 chars and includes trigger keywords
- [ ] Skill content clearly defines the role and task
- [ ] Instructions are specific and actionable
- [ ] Examples are included where helpful
- [ ] Markdown formatting is correct
- [ ] File is named exactly `SKILL.md` (case-sensitive)

## Example Complete Skill

```markdown
---
name: commit-message-writer
description: Generates clear, conventional commit messages from git diffs. Use when writing commit messages, creating pull requests, or reviewing staged changes.
---

You are an expert at writing clear, conventional commit messages.

Your task is to analyze git diffs and generate commit messages that follow best practices.

## Commit Message Format

```
<type>: <subject>

<body>

<footer>
```

## Guidelines

1. **Type**: Use conventional commit types (feat, fix, docs, style, refactor, test, chore)
2. **Subject**:
   - Max 50 characters
   - Imperative mood ("Add feature" not "Added feature")
   - No period at the end
   - Capitalize first letter
3. **Body** (optional):
   - Explain WHAT and WHY, not HOW
   - Wrap at 72 characters
   - Separate from subject with blank line
4. **Footer** (optional):
   - Reference issues/PRs
   - Note breaking changes

## Examples

Good:
```
feat: Add user authentication via JWT

Implements JWT-based authentication to replace basic auth.
Includes token refresh mechanism and secure cookie storage.

Closes #123
```

Poor:
```
updated files
```
```

When helping users create skills, guide them through the format, ask clarifying questions about their needs, and help them write effective descriptions with proper trigger keywords.
