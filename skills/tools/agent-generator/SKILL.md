---
name: agent-generator
description: This skill should be used when the user asks to "create an agent", "generate an agent", "build a new subagent", "make an agent for Claude Code", or wants to create autonomous agent configurations for Claude Code or plugins.
version: 0.1.0
allowed-tools: ["Read", "Grep", "Write", "Bash", "Edit", "AskUserQuestion"]
---

## Purpose

Generate well-structured, production-ready Claude Code agent configurations (`.md` files with YAML frontmatter) that follow official standards, include proper triggering conditions with examples, and contain comprehensive system prompts for autonomous operation.

## When to Use This Skill

Use this skill when:
- Creating a new agent for Claude Code (personal or plugin-level)
- Generating autonomous subprocess configurations for specific tasks
- Building domain-specialized agents with restricted tool access
- Converting workflows into reusable agent configurations
- The user wants an agent that follows current best practices

## Core Workflow

### Step 1: Latest Documentation

!`curl -s https://code.claude.com/docs/en/sub-agents.md`

Read and internalize this documentation to ensure the generated agent follows current best practices, structure requirements, and writing conventions.

### Step 2: Understand Requirements

Gather concrete information about the agent's purpose:

**Essential questions:**
- What specific task should this agent handle autonomously?
- What triggering scenarios should cause Claude to delegate to this agent?
- What tools should the agent have access to? (Use principle of least privilege)
- What domain expertise should the system prompt include?
- Should this agent be in a plugin or personal configuration?
- Should this agent run in foreground or background?
- Which skills should the agent pre-load, if any?

**Example dialogue:**
```
User: "Create an agent for database schema validation"

Ask: "What specific validation tasks should this agent handle?
For example:
- 'Check schema files for syntax errors'
- 'Validate foreign key relationships'
- 'Verify migration files are properly ordered'

What tools should it need? (Read, Grep, Bash for validation scripts?)

Should this be a personal agent or part of a plugin?"
```

Do not overwhelm with questions. Focus on the most critical aspects first: purpose, triggering conditions, and tool access.

### Step 3: Plan Agent Structure

Based on requirements, identify the agent configuration:

**Identifier:**
- 3-50 characters, lowercase letters, numbers, hyphens only
- Must start and end with alphanumeric
- Use descriptive names: `code-reviewer`, `security-analyzer`, `test-generator`

**Model Selection:**
- `inherit` - Use same model as parent (recommended for most cases)
- `sonnet` - Balanced performance/cost
- `opus` - Maximum capability for complex reasoning
- `haiku` - Fast, cheap for simple tasks

**Color Selection:**
- Blue/cyan: Analysis, review, exploration
- Green: Success-oriented, generation
- Yellow: Validation, verification
- Red: Critical, security, error handling
- Magenta: Creative, content generation

**Tools:**
- Read-only analysis: `["Read", "Grep", "Glob"]`
- Code generation: `["Read", "Write", "Grep", "Edit"]`
- Testing/validation: `["Read", "Bash", "Grep"]`
- Full access: Omit `tools` field or use `["*"]


### Step 4: Create Directory Structure

For plugin agents:
```bash
mkdir -p ./path-to-plugin/agents/
```

For personal agents:
```bash
mkdir -p ~/.claude/agents/
```

Delete unnecessary directories. Create only what is needed.

### Step 5: Write Agent Configuration File

#### Frontmatter Requirements

**Critical:** The `description` field determines when Claude triggers the agent.

```yaml
---
name: agent-identifier
description: Use this agent when the user asks to "specific phrase 1", "specific phrase 2", "specific phrase 3", or mentions "key concept". Be concrete and specific about triggers.
model: inherit
color: blue
tools: ["Read", "Write", "Grep"]
---
```

**Description field guidelines:**
- Start with "This skill should be used when..." or "Use this agent when..."
- Include 3-5 specific trigger phrases users would actually say
- Be concrete and specific about what triggers the agent
- Mention key concepts that should trigger the agent
- Do NOT include `<example>` blocks in YAML frontmatter (breaks parsing)

#### System Prompt Design

The markdown body becomes the agent's system prompt. Write in second person.

**Standard template:**
```markdown
You are [role] specializing in [domain]. Run in the [background or foreground].

**Your Core Responsibilities:**
1. [Primary responsibility with clear success criteria]
2. [Secondary responsibility]
3. [Additional responsibilities as needed]

**Analysis Process:**
1. [Step one - what to do first]
2. [Step two - how to proceed]
3. [Step three - how to conclude]

**Quality Standards:**
- [Specific quality criterion 1]
- [Specific quality criterion 2]

**Output Format:**
Provide results in this format:
- [What information to include]
- [How to structure the response]

**Edge Cases:**
Handle these situations:
- [Edge case 1]: [How to handle it]
- [Edge case 2]: [How to handle it]
```

**System prompt best practices:**
- Write in second person ("You are...", "You will...")
- Be specific about responsibilities and success criteria
- Provide clear step-by-step process
- Define exact output format
- Include quality standards
- Address edge cases explicitly
- Keep under 10,000 characters (ideal: 500-3,000)

### Step 6: Validate and Test

**Structure validation:**
- [ ] Agent file has valid YAML frontmatter
- [ ] All required fields present: `name`, `description`, `model`, `color`
- [ ] `name` follows identifier rules (3-50 chars, lowercase-hyphens)
- [ ] `description` includes specific trigger phrases
- [ ] System prompt uses second person
- [ ] `tools` field uses correct format (array of strings) if present

**Content validation:**
- [ ] Triggering phrases in description are specific and realistic
- [ ] System prompt defines clear responsibilities
- [ ] Process steps are actionable
- [ ] Output format is specified
- [ ] Edge cases are addressed

**Test the agent:**
1. Create a scenario matching one of the triggering examples
2. Verify Claude recognizes and delegates to the agent
3. Confirm agent follows its system prompt
4. Check output matches specified format
5. Test edge cases mentioned in system prompt

### Step 7: Handle Skills Pre-loading

If the user specifies that the agent should pre-load certain skills, add the `skills` frontmatter field:

```yaml
---
name: agent-identifier
description: Use this agent when...
skills: ["skill-name-1", "skill-name-2"]
---
```

If no skills are specified by the user, ask:
"Should this agent pre-load any specific skills for its task? For example, should it have access to specialized skills for code review, testing, or documentation?"

**Skills field guidelines:**
- Only include skills that are directly relevant to the agent's task
- Don't preload skills unless they provide necessary domain expertise
- The agent will have access to all tools specified in `tools` field
- Skills enhance the agent with specialized capabilities

## Writing Style Rules

### Second Person for System Prompt

Address the agent directly:

**Correct:**
```
You are a code reviewer specializing in security vulnerabilities.
You will analyze the code for common OWASP Top 10 issues.
You must provide specific line references for each finding.
```

**Incorrect:**
```
This agent is a code reviewer.
The agent will analyze code.
Findings should include line references.
```

### Third Person in Description

Frontmatter description must use third person:

**Correct:**
```yaml
description: Use this agent when the user asks to "review code", "check for security issues", or mentions "code review" or "security analysis".
```

**Incorrect:**
```yaml
description: Use when you need to review code.
description: Load for code review tasks.
```

### Specific Triggering Examples

Include concrete, realistic examples:

**Correct:**
```yaml
<example>
Context: User has made changes and wants quality feedback
user: "Review my recent changes for bugs"
assistant: "I'll use the code-reviewer agent to analyze your changes for potential bugs and issues."
<commentary>
User is requesting code review, which is the primary purpose of this agent.
</commentary>
</example>
```

**Incorrect:**
```yaml
<example>
user: "Help with code"
assistant: "Using agent"
<commentary>
Too vague to be useful.
</commentary>
</example>
```

## Common Mistakes to Avoid

### Mistake 1: Vague Triggering Description

❌ **Bad:**
```yaml
description: Helps with code review.
```

**Why:** No triggering conditions, no examples, not specific

✅ **Good:**
```yaml
description: Use this agent when the user asks to "review code", "check for bugs", "analyze my changes", or mentions "code review" or "static analysis". Examples: [...]
```

**Why:** Clear triggers, specific user phrases, includes examples

### Mistake 2: Missing Example Components

❌ **Bad:**
```yaml
<example>
user: "Review this"
assistant: "OK"
</example>
```

**Why:** Missing Context and commentary, too vague

✅ **Good:**
```yaml
<example>
Context: User has just committed changes and wants validation
user: "Can you review the changes I just made?"
assistant: "I'll delegate to the code-reviewer agent to analyze your recent changes for potential issues."
<commentary>
User is explicitly requesting review of their changes, which matches this agent's purpose.
</commentary>
</example>
```

**Why:** Complete example with context and reasoning

### Mistake 3: Over-Permissive Tool Access

❌ **Bad:**
```yaml
tools: ["*"]
```

**Why:** Agent can access all tools, including destructive ones

✅ **Good:**
```yaml
tools: ["Read", "Grep", "Glob"]
```

**Why:** Agent restricted to read-only analysis (principle of least privilege)

### Mistake 4: First Person System Prompt

❌ **Bad:**
```markdown
I am a code reviewer. I will analyze code.
```

**Why:** First person instead of second person

✅ **Good:**
```markdown
You are a code reviewer specializing in security analysis.
You will examine code for potential vulnerabilities.
```

**Why:** Second person, direct instructions to agent

## Validation Checklist

Before finalizing a generated agent:

**Structure:**
- [ ] File named appropriately (lowercase, hyphens, `.md` extension)
- [ ] Valid YAML frontmatter with all required fields
- [ ] `name` follows identifier rules (3-50 chars, alphanumeric start/end)
- [ ] `description` includes "Use this agent when..." prefix with specific trigger phrases
- [ ] `model` is set (usually `inherit`)
- [ ] `color` is specified
- [ ] `tools` array is valid (if present)

**Content Quality:**
- [ ] Triggering phrases are specific and realistic
- [ ] System prompt uses second person throughout
- [ ] System prompt defines clear responsibilities
- [ ] Process steps are actionable and specific
- [ ] Output format is clearly specified
- [ ] Edge cases are addressed
- [ ] System prompt length is reasonable (500-3,000 characters)

**Skills Configuration:**
- [ ] If user specified skills, `skills` field includes them
- [ ] If no skills specified, user was asked about them
- [ ] Listed skills are relevant to agent's task

## Quick Reference Templates

### Minimal Agent Structure

```markdown
---
name: simple-agent
description: Use this agent when the user asks to "do X", "perform Y", or mentions "Z concept".
model: inherit
color: blue
---

You are an agent that [does X].

Process:
1. [Step 1]
2. [Step 2]

Output: [What to provide]
```

### Analysis Agent (Read-Only)

```markdown
---
name: analyzer
description: Use this agent when the user asks to "analyze code", "examine structure", "map dependencies", or mentions code analysis, structure analysis, or dependency mapping.
model: inherit
color: blue
tools: ["Read", "Grep", "Glob"]
---

You are a code analyst specializing in [domain].

**Analysis Process:**
1. Read and understand the codebase structure
2. Identify patterns and relationships
3. Report findings with specific references

**Output:**
- Summary of key findings
- Specific file and line references
- Recommendations with rationale
```

### Generation Agent (Write Access)

```markdown
---
name: generator
description: Use this agent when the user asks to "generate X", "create Y", "build Z", or mentions content generation, code generation, or creating new artifacts.
model: inherit
color: green
tools: ["Read", "Write", "Grep", "Edit"]
---

You are a content generator specializing in [domain].

**Generation Process:**
1. Understand requirements from context
2. Read existing code/patterns for consistency
3. Generate new content following conventions
4. Validate output quality

**Quality Standards:**
- Follow project naming conventions
- Maintain consistency with existing code
- Include appropriate error handling
```

### Complete Agent with Skills

```markdown
---
name: specialist
description: Use this agent when the user asks to "specialize task", "handle X", or mentions specific domain expertise or specialized processing.
model: inherit
color: cyan
tools: ["Read", "Write", "Grep", "Bash"]
skills: ["relevant-skill-1", "relevant-skill-2"]
---

You are a specialist agent with expertise in [domain].

**Core Responsibilities:**
1. [Responsibility 1]
2. [Responsibility 2]

**Process:**
[Step-by-step workflow]

**Output Format:**
[Detailed format specification]

**Edge Cases:**
[Edge case handling]
```

## Frontmatter Fields Summary

| Field | Required | Format | Example |
|-------|----------|--------|---------|
| name | Yes | lowercase-hyphens (3-50 chars) | code-reviewer |
| description | Yes | Text with specific trigger phrases | Use this agent when the user asks to "review code", "check for bugs"... |
| model | Yes | inherit/sonnet/opus/haiku | inherit |
| color | Yes | blue/cyan/green/yellow/magenta/red | blue |
| tools | No | Array of tool names | ["Read", "Grep"] |
| skills | No | Array of skill names | ["skill-name"] |

## Implementation Steps Summary

To generate an agent:

1. **Fetch docs**: Run `!curl -s https://code.claude.com/docs/en/sub-agents.md`
2. **Understand**: Ask about purpose, triggering conditions, tools, and skills
3. **Plan**: Determine identifier, model, color, and tool access
4. **Structure**: Create agent file in appropriate directory
5. **Write configuration**:
   - Frontmatter with all required fields
   - Description with specific trigger phrases
   - System prompt in second person
6. **Add skills**: Include `skills` field if user specified relevant skills
7. **Validate**: Check structure, content, and completeness
8. **Test**: Verify triggering and system prompt execution

## Additional Resources

### Reference Files

For detailed guidance, consult:
- **`references/agent-patterns.md`** - Common agent patterns and architectures
- **`references/system-prompts.md`** - System prompt design patterns

### Example Files

Working examples in `examples/`:
- **`examples/read-only-agent.md`** - Analysis agent with restricted tools
- **`examples/generation-agent.md`** - Content generation with write access
- **`examples/skilled-agent.md`** - Agent with pre-loaded skills

## Best Practices

✅ **DO:**
- Fetch latest documentation before generating
- Ask about skills pre-loading if not specified
- Use specific triggering examples with complete structure
- Write system prompts in second person
- Restrict tools using principle of least privilege
- Define clear output formats
- Address edge cases explicitly
- Validate before finalizing

❌ **DON'T:**
- Skip fetching latest documentation
- Use vague triggering descriptions
- Write system prompts in first person
- Grant unnecessary tool access
- Omit example components (Context, commentary)
- Skip validation
- Forget to ask about skills if not specified
