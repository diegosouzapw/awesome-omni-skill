---
name: flow-engineer-subagent
description: Guide for creating effective Subagents (custom agent definitions). Use when users want to create a new subagent, set up task-specific agents, configure code reviewers, debuggers, or domain-specific assistants. Works across IDEs (Cursor, OpenCode).
disable-model-invocation: true
---

# Subagent Creator

This skill guides through creating effective Subagents — custom agent definitions that provide specialized AI assistants with focused system prompts.

## About Subagents

Subagents are specialized AI assistants that extend agent capabilities with:
1. **Focused behavior** — custom system prompts for specific domains or tasks
2. **Context isolation** — separate conversations that preserve your main context
3. **Reusable configurations** — shareable agent definitions across projects or users

## IDE Detection and Subagent Placement

Subagents work across multiple IDEs. Before creating a subagent, determine the current environment and ask the user where to place it.

### Control Primitives Map by IDE

| Primitive | Scope | Cursor | Claude Code | OpenCode |
| :--- | :--- | :--- | :--- | :--- |
| **Custom Agents (Subagents)** | User | `~/.cursor/agents/*.md` | `~/.claude/agents/*.md` | `~/.config/opencode/agents/*.md` |
| | Project | `.cursor/agents/*.md` | `.claude/agents/*.md` | `.opencode/agents/*.md` |

### Subagent-Specific Paths

| IDE | Personal Subagents | Project Subagents |
|-----|-------------------|-------------------|
| **Cursor** | `~/.cursor/agents/*.md` | `.cursor/agents/*.md` |
| **Claude Code** | `~/.claude/agents/*.md` | `.claude/agents/*.md` |
| **OpenCode** | `~/.config/opencode/agents/*.md` | `.opencode/agents/*.md` |

### Detection Strategy

1. Check for IDE-specific markers in the project:
   - `.cursor/` directory → Cursor
   - `.claude/` directory → Claude Code
   - `.opencode/` directory or `opencode.json` → OpenCode
2. If multiple detected or none → ask the user
3. Ask: personal subagent (user-level) or project subagent (shared via repo)?

**IMPORTANT**: Never create subagents in `~/.cursor/agents-cursor/` (reserved for Cursor internals) or other IDE-reserved directories.

## Core Principles

### Focused Specialization

Each subagent should excel at **one specific task**.

- Good: `code-reviewer` — focused on reviewing code quality
- Bad: `helper` — too broad, overlaps with default agent

### Clear Delegation Triggers

The description determines when the main agent delegates to this subagent.

| Freedom Level | When to Use | Example |
|---------------|-------------|---------|
| **High** (manual) | User explicitly requests | "Use the data-analyst to..." |
| **Medium** (suggested) | Agent suggests based on context | "Consider using debugger for errors" |
| **Low** (proactive) | Automatic delegation on pattern match | "Use code-reviewer after code changes" |

### Anatomy of a Subagent

Every subagent is a `.md` file with YAML frontmatter and a markdown body:

```
my-agent.md
├── Frontmatter (YAML)  # Metadata: name, description, mode, etc.
└── Body (Markdown)     # System prompt: behavior, workflow, constraints
```

### Frontmatter Structure

#### Cursor Format

```yaml
---
name: code-reviewer
description: Expert code review specialist. Proactively reviews code for quality, security, and maintainability. Use immediately after writing or modifying code.
model: inherit
readonly: false
---
```

#### Cursor Additional Fields

| Field        | Description                                      | Default                    |
|-------------|---------------------------------------------------|----------------------------|
| `name`      | Agent name                                       | Required                   |
| `description` | Task description and role definition            | Required                   |
| `model`     | Suggested model (`inherit`, `fast`, `slow`, or model ID) | Inherits from conversation |
| `readonly`  | If `true`, restricts agent to read-only tools (no file edits, shell writes) | `false` |

#### OpenCode Format

```yaml
---
description: Expert code review specialist. Proactively reviews code for quality, security, and maintainability. Use immediately after writing or modifying code.
mode: subagent
model: gpt-4o
temperature: 0.3
permission: auto
---
```

#### OpenCode Additional Fields

| Field | Description | Default |
|-------|-------------|---------|
| `mode` | `primary` / `subagent` / `all` | `subagent` |
| `model` | Specific model string | Inherits from conversation |
| `temperature` | Creativity (0.0-2.0) | Inherits |
| `top_p` | Nucleus sampling (0.0-1.0) | Inherits |
| `steps` | Max reasoning steps | Inherits |
| `tools` | Allowed tools | Inherits |
| `permission` | `auto` / `ask` / `none` | Inherits |
| `color` | UI color code | Inherited |
| `hidden` | Hide from UI | `false` |
| `disable` | Disable subagent | `false` |

## Writing Effective Descriptions

The description is **critical** - the AI uses it to decide when to delegate.

### Rules

1. **Third person** (injected into system prompt):
   - Good: "Reviews code for quality and best practices"
   - Bad: "I review code for quality and best practices"

2. **Specific with trigger terms**:
   - Good: "Debugging specialist for errors, test failures, and unexpected behavior. Use proactively when encountering any issues."
   - Bad: "Helps with debugging"

3. **Include WHAT and WHEN**:
   - WHAT: specific capabilities
   - WHEN: trigger scenarios (proactive, suggested, manual)

4. **Action verbs**: Start with verbs describing what the subagent does.

### Examples

```yaml
# ❌ Too vague
description: Helps with code

# ❌ Not specific enough
description: Code reviewer

# ✅ Specific and actionable
description: Expert code review specialist. Proactively reviews code for quality, security, and maintainability. Use immediately after writing or modifying code.
```

## Common Subagent Types

### Code Reviewer

```markdown
---
name: code-reviewer
description: Expert code review specialist. Proactively reviews code for quality, security, and maintainability. Use immediately after writing or modifying code.
---

You are a senior code reviewer ensuring high standards of code quality and security.

When invoked:
1. Run git diff to see recent changes
2. Focus on modified files
3. Begin review immediately

Review checklist:
- Code is clear and readable
- Functions and variables are well-named
- No duplicated code
- Proper error handling
- No exposed secrets or API keys
- Input validation implemented
- Good test coverage
- Performance considerations addressed

Provide feedback organized by priority:
- Critical issues (must fix)
- Warnings (should fix)
- Suggestions (consider improving)

Include specific examples of how to fix issues.
```

### Debugger

```markdown
---
name: debugger
description: Debugging specialist for errors, test failures, and unexpected behavior. Use proactively when encountering any issues.
---

You are an expert debugger specializing in root cause analysis.

When invoked:
1. Capture error message and stack trace
2. Identify reproduction steps
3. Isolate the failure location
4. Implement minimal fix
5. Verify solution works

Debugging process:
- Analyze error messages and logs
- Check recent code changes
- Form and test hypotheses
- Add strategic debug logging
- Inspect variable states

For each issue, provide:
- Root cause explanation
- Evidence supporting the diagnosis
- Specific code fix
- Testing approach
- Prevention recommendations

Focus on fixing the underlying issue, not the symptoms.
```

### Data Scientist

```markdown
---
name: data-scientist
description: Data analysis expert for SQL queries, BigQuery operations, and data insights. Use proactively for data analysis tasks and queries.
---

You are a data scientist specializing in SQL and BigQuery analysis.

When invoked:
1. Understand the data analysis requirement
2. Write efficient SQL queries
3. Use BigQuery command line tools (bq) when appropriate
4. Analyze and summarize results
5. Present findings clearly

Key practices:
- Write optimized SQL queries with proper filters
- Use appropriate aggregations and joins
- Include comments explaining complex logic
- Format results for readability
- Provide data-driven recommendations

For each analysis:
- Explain the query approach
- Document any assumptions
- Highlight key findings
- Suggest next steps based on data

Always ensure queries are efficient and cost-effective.
```

### Documentation Specialist

```markdown
---
name: docs-specialist
description: Technical documentation expert for API docs, README files, and inline comments. Use proactively when writing or updating documentation.
---

You are a technical documentation specialist focused on clarity and completeness.

When invoked:
1. Understand the code or feature being documented
2. Identify target audience (developers, users, etc.)
3. Structure documentation logically
4. Write clear, concise explanations
5. Include examples and use cases

Documentation principles:
- Start with user's goal, not implementation
- Provide concrete examples
- Keep explanations simple and direct
- Use consistent terminology
- Link to related topics

For each documentation piece:
- Overview (what and why)
- Quick start (minimal example)
- Reference (detailed parameters)
- Examples (common use cases)
- Troubleshooting (common issues)
```

## Subagent Creation Workflow

### Phase 1: Discovery

Gather from user:
1. Purpose and primary use case
2. Target IDE and storage location (personal vs project)
3. Delegation behavior (proactive, suggested, or manual)
4. Domain knowledge the agent needs
5. Workflow or process to follow
6. Output format preferences

If context from prior conversation exists, infer the subagent from discussed workflows or specialized tasks.

### Phase 2: Design

1. Draft subagent name (lowercase letters and hyphens only)
2. Write specific, third-person description with WHAT + WHEN
3. Define workflow process (checklists, steps)
4. Identify constraints or guidelines
5. Determine delegation triggers

### Phase 3: Implementation

1. Create the file in the IDE-appropriate location:
   ```bash
   # Cursor (project-level)
   mkdir -p .cursor/agents && touch .cursor/agents/my-agent.md

   # Claude Code (project-level)
   mkdir -p .claude/agents && touch .claude/agents/my-agent.md

   # OpenCode (project-level)
   mkdir -p .opencode/agents && touch .opencode/agents/my-agent.md
   ```

2. Write the frontmatter with required fields:
   - Cursor: `name`, `description`; optional `model`, `readonly`
   - Claude Code: `name`, `description`; optional `model`
   - OpenCode: `description`, optional `mode`, `model`, etc.

3. Write the system prompt body:
   - Purpose and role
   - When/how to invoke
   - Workflow process
   - Output format
   - Constraints and guidelines

### Phase 4: Verification

Test the subagent:

```bash
# Test by asking the main agent to use it
"Use the my-agent subagent to [task description]"
```

Checklist:
- [ ] Description is specific, includes trigger terms, WHAT + WHEN
- [ ] Written in third person
- [ ] Consistent terminology
- [ ] Clear workflow or process
- [ ] Appropriate delegation behavior (proactive/suggested/manual)
- [ ] File placed in correct location
- [ ] YAML frontmatter is valid

### Phase 5: Iterate

1. Use subagent on real tasks
2. Notice struggles or inefficiencies
3. Update description or system prompt
4. Test again

## Best Practices

1. **One task per subagent**: Each should excel at one specific task
2. **Write detailed descriptions**: Include trigger terms so the AI knows when to delegate
3. **Check into version control**: Share project subagents with your team
4. **Use proactive language**: Include "use proactively" in descriptions for automatic delegation
5. **Define clear workflows**: Use checklists or step-by-step processes
6. **Provide examples**: Show what good output looks like

## Anti-Patterns

- **Overly broad scope**: `general-helper` not `code-reviewer`
- **Vague descriptions**: "Helps with code" vs "Reviews code for quality and best practices"
- **Missing delegation triggers**: No "use proactively" or trigger terms
- **Too many options**: Provide a preferred approach with escape hatch
- **Inconsistent terminology**: Pick one term, use throughout
- **Windows paths**: Use `scripts/helper.py`, not `scripts\helper.py`

## Comparison with Skills

| Aspect | Skills | Subagents |
|--------|--------|-----------|
| **Purpose** | Teach agent how to do something | Define a specialized assistant |
| **Format** | Directory with `SKILL.md` | Single `.md` file |
| **Context** | Loaded as reference when triggered | Isolated conversation context |
| **Use case** | Reusable workflows and patterns | Specialized behavior or domain expertise |
| **Scope** | Procedural knowledge | Behavioral specialization |

Choose **Skills** when you want to extend the main agent's capabilities with new workflows or knowledge.

Choose **Subagents** when you want a separate assistant with a focused role and isolated context.
