---
name: writing-agents
description: "Creates effective agent definitions for Claude Code, Cursor, and OpenCode. Covers system prompt design, frontmatter configuration, orchestration rules, and multi-tool deployment. This skill should be used when the user asks to 'write an agent', 'create a subagent', 'build an agent roster', 'design agent orchestration', 'improve an agent prompt', or mentions agent definitions, subagent configuration, or multi-agent workflows."
user-invocable: true
disable-model-invocation: false
---

# Writing Agents

Create effective agent definitions that specialize AI behavior for specific tasks. Agents are markdown files with YAML frontmatter that define a subagent's identity, capabilities, and constraints.

## Quick Start

Every agent requires a markdown file with YAML frontmatter:

```yaml
---
name: code-reviewer
description: Reviews code for quality, security, and best practices. Use proactively after code changes.
---

You are a code reviewer. Analyze code for security vulnerabilities, performance issues, and maintainability.
```

Read the `portable-agents` skill first for cross-tool frontmatter compatibility. This skill focuses on writing effective agent **content**.

## Core Principles

### 1. Description Drives Delegation

The `description` field is the most important line in the file. The orchestrating agent reads it to decide when to delegate. Write it as a clear trigger condition.

```yaml
# Good - specific triggers, clear scope
description: Reviews code for quality and best practices. Use proactively after code changes.
description: Specialized for multi-repository analysis and finding implementation examples in open source. MUST BE USED when users ask to look up code in remote repositories.

# Bad - vague, no trigger conditions
description: A helpful coding assistant.
description: Does various tasks.
```

Include "Use proactively" or "MUST BE USED when" to control delegation aggressiveness.

### 2. System Prompts Are Identity Documents

The markdown body defines who the agent is, what it does, and how it works. Structure it as a complete operating manual, not a suggestion list.

**Effective prompts have four layers:**

| Layer | Purpose | Example |
|-------|---------|---------|
| **Identity** | Who the agent is | "You are Atlas - the Systematic Task Executor" |
| **Workflow** | Step-by-step process | Phase 0: Classify → Phase 1: Explore → Phase 2: Execute |
| **Constraints** | Hard boundaries | "Never suppress type errors", "Read-only" |
| **Output contract** | What to return | Structured format, evidence requirements |

### 3. Constrain Aggressively

Agents without constraints drift. Every agent should have explicit hard blocks:

```markdown
## Hard Constraints (NEVER violate)

| Constraint | No Exceptions |
|------------|---------------|
| Type error suppression (`as any`, `@ts-ignore`) | Never |
| Commit without explicit request | Never |
| Speculate about unread code | Never |
| Leave code in broken state | Never |
```

### 4. Demand Evidence

Agents that claim completion without proof are unreliable. Require verification artifacts:

```markdown
## Verification Requirements

Task is NOT complete without:
- `ReadLints` clean on changed files
- Build passes (exit code 0)
- All todos marked completed

**No evidence = not complete.**
```

## Authoring Workflow

### Step 1: Define the Role

Answer three questions:
1. **What does this agent do?** (one sentence)
2. **When should the orchestrator delegate to it?** (trigger conditions)
3. **What should it NOT do?** (boundaries)

### Step 2: Design the Workflow

Map the agent's process as numbered phases. Each phase should have:
- **Entry condition**: When to enter this phase
- **Actions**: What to do
- **Exit condition**: When to move to next phase

```markdown
## Phase 0: Intent Gate (EVERY request)

Classify the request type:
| Type | Signal | Action |
|------|--------|--------|
| Trivial | Single file, known location | Direct tools only |
| Exploratory | "How does X work?" | Parallel search |
| Open-ended | "Improve", "Refactor" | Full execution loop |

## Phase 1: Exploration
...

## Phase 2: Execution
...

## Phase 3: Verification
...
```

### Step 3: Write Constraints

Two categories:

**Hard blocks** (NEVER violate -- use a table for visual weight):
```markdown
| Constraint | No Exceptions |
|------------|---------------|
| Suppress type errors | Never |
| Commit without request | Never |
```

**Behavioral constraints** (agent-specific boundaries):
```markdown
## Constraints
- **Read-only**: Cannot create, modify, or delete files
- **No delegation**: Cannot spawn other agents
- **No emojis**: Keep output clean and parseable
```

### Step 4: Define Output Contract

Specify exactly what the agent returns. Structured output formats prevent ambiguity:

```markdown
## Output Contract

Always end with this format:

<results>
<files>
- /absolute/path/file.ts — [why relevant]
</files>
<answer>
[Direct answer to the actual need]
</answer>
</results>
```

### Step 5: Add Communication Style

Prevent AI slop with explicit style rules:

```markdown
## Communication Style
- Start work immediately. No acknowledgments.
- Don't summarize unless asked.
- One word answers are acceptable.
- Never start with flattery ("Great question!")
```

### Step 6: Deploy

Place the file in the appropriate directory:

| Tool | Project | User |
|------|---------|------|
| Claude Code | `.claude/agents/` | `~/.claude/agents/` |
| Cursor | `.cursor/agents/` | `~/.cursor/agents/` |
| OpenCode | `.opencode/agents/` | `~/.config/opencode/agents/` |

For multi-tool deployment, see the `portable-agents` skill.

## System Prompt Patterns

### Intent Gate Pattern

Start every agent with request classification. Prevents the agent from using a sledgehammer on a thumbtack:

```markdown
## Phase 0: Intent Gate

| Type | Signal | Action |
|------|--------|--------|
| **Trivial** | Single file, direct answer | Direct tools only |
| **Explicit** | Specific file/line, clear command | Execute directly |
| **Exploratory** | "How does X work?" | Parallel search |
| **Open-ended** | "Improve", "Refactor" | Assess → plan → execute |
| **Ambiguous** | Unclear scope | Ask ONE clarifying question |
```

### Parallel-First Pattern

For search/research agents, mandate parallel tool execution:

```markdown
## Parallel Execution (DEFAULT behavior)

Launch **3+ tools simultaneously** in your first action:

Grep(pattern="auth", path="src/")
Glob(glob_pattern="**/auth*.ts")
SemanticSearch(query="Where is authentication implemented?")

Never sequential unless output depends on prior result.
```

### Todo Obsession Pattern

For execution agents, enforce progress tracking:

```markdown
## Todo Management (NON-NEGOTIABLE)

- 2+ steps → Create todos FIRST
- Mark `in_progress` before starting (ONE at a time)
- Mark `completed` IMMEDIATELY after each step
- NEVER batch completions

Skipping todos on non-trivial tasks = INCOMPLETE WORK.
```

### Failure Recovery Pattern

Prevent agents from spiraling on broken code:

```markdown
## Failure Recovery

1. Fix root causes, not symptoms
2. Re-verify after EVERY fix attempt
3. Never shotgun debug

**After 3 consecutive failures:**
1. STOP all edits
2. REVERT to last working state
3. DOCUMENT what failed
4. ASK USER for guidance
```

### Codebase Assessment Pattern

For agents that modify code, assess before following patterns:

```markdown
## Codebase Assessment

| State | Signals | Behavior |
|-------|---------|----------|
| **Disciplined** | Consistent patterns, configs, tests | Follow existing style |
| **Transitional** | Mixed patterns | Ask which pattern to follow |
| **Legacy/Chaotic** | No consistency | Propose conventions |
| **Greenfield** | New/empty project | Apply modern best practices |
```

## Writing Orchestrator Rules

Orchestrator rules tell the root thread when and how to delegate. They're separate from agent definitions.

### Cursor: `.cursor/rules/orchestrator.mdc`

```yaml
---
description: Root-level orchestration brain
alwaysApply: true
---

# Orchestration Rules
[delegation logic]
```

### Claude Code / OpenCode: `AGENTS.md`

Put orchestration rules in `AGENTS.md` at the project root. Both tools read it automatically.

### Key Orchestrator Components

1. **Auto-triggers**: Signals that fire agents proactively
2. **Request classification**: Map request types to agent dispatch
3. **Phase chaining**: Sequential agent handoffs with context passing
4. **Verification gates**: Evidence requirements between phases
5. **Failure recovery**: Escalation paths when agents fail

See [references/orchestration-patterns.md](references/orchestration-patterns.md) for complete examples.

## Anti-Patterns

| Anti-Pattern | Problem | Fix |
|--------------|---------|-----|
| Vague description | Never gets delegated to | Add specific trigger conditions |
| No constraints | Agent drifts, makes unauthorized changes | Add hard blocks table |
| No verification | Claims completion without evidence | Require lint/build/test evidence |
| Monolithic prompt | Too long, agent ignores later sections | Use phases with clear entry/exit |
| "You should" language | Inconsistent, weak | Use imperative form |
| Asking questions by default | Blocks progress | Mandate explore-first, ask-last |
| No failure recovery | Spirals on broken code | Add 3-failure stop protocol |
| Same prompt everywhere | Ignores tool differences | Use portable subset + tool-specific frontmatter |

## Validation Checklist

Before finalizing an agent:

- [ ] `description` includes specific trigger conditions
- [ ] System prompt has identity, workflow, constraints, and output contract
- [ ] Hard constraints table present with "No Exceptions" column
- [ ] Verification requirements defined with evidence types
- [ ] Communication style section prevents AI slop
- [ ] Failure recovery protocol included (for execution agents)
- [ ] Tested delegation from orchestrator
- [ ] Prompt under 500 lines (move details to references)

## References

- **[references/orchestration-patterns.md](references/orchestration-patterns.md)** - Complete orchestrator rule examples
- **[references/prompt-templates.md](references/prompt-templates.md)** - Starter templates for common agent roles
