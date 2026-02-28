---
name: agents-manager
description: "Branch skill for building and improving agents. Use when creating new agents, adapting marketplace agents, validating agent structure, writing system prompts, or improving existing agents. Triggers: 'create agent', 'improve agent', 'validate agent', 'fix agent', 'agent frontmatter', 'system prompt', 'adapt agent', 'customize agent', 'agent examples', 'agent tools'."
---

# Agents Manager - Branch of JARVIS-03

Build and improve agents following the agents-management policy.

## Policy Source

**Primary policy**: JARVIS-03 → `.claude/skills/agents-management/SKILL.md`

This branch **executes** the policy defined by JARVIS-03. Always sync with Primary before major operations.

## Quick Decision Tree

```text
Task Received
    │
    ├── Create new agent? ───────────────> Workflow 1: Build
    │   └── What complexity?
    │       ├── Simple (responder) ──────> Pattern 3: Small (2-4k words)
    │       ├── Developer ───────────────> Pattern 2: Medium (3-8k words)
    │       └── Architect ───────────────> Pattern 1: Large (10k+ words)
    │
    ├── Adapt marketplace agent? ────────> Workflow 3: Adapt
    │
    ├── Fix existing agent? ─────────────> Workflow 2: Improve
    │
    └── Validate agent? ─────────────────> Validation Checklist
```

## Agent Overview

Agents are autonomous subprocesses that handle complex, multi-step tasks independently.

**Key concepts:**
- Agents are FOR autonomous work, commands are FOR user-initiated actions
- Markdown file format with YAML frontmatter
- Triggering via description field with examples
- System prompt defines agent behavior
- Model and color customization

## Agent File Structure

### Complete Format

```markdown
---
name: agent-identifier
description: Use this agent when [triggering conditions]. Examples:

<example>
Context: [Situation description]
user: "[User request]"
assistant: "[How assistant should respond and use this agent]"
<commentary>
[Why this agent should be triggered]
</commentary>
</example>

<example>
[Additional example...]
</example>

model: inherit
color: blue
tools: ["Read", "Write", "Grep"]
---

You are [agent role description]...

**Your Core Responsibilities:**
1. [Responsibility 1]
2. [Responsibility 2]

**Analysis Process:**
[Step-by-step workflow]

**Output Format:**
[What to return]
```

## Frontmatter Fields

### name (required)

Agent identifier used for namespacing and invocation.

**Format:** lowercase, numbers, hyphens only
**Length:** 3-50 characters
**Pattern:** Must start and end with alphanumeric

**Validation:**

```text
✅ Valid: code-reviewer, test-gen, api-analyzer-v2
❌ Invalid: ag (too short), -start (starts with hyphen), my_agent (underscore)
```

**Rules:**
- 3-50 characters
- Lowercase letters, numbers, hyphens only
- Must start and end with alphanumeric
- No underscores, spaces, or special characters

**Good examples:**
- `code-reviewer`
- `test-generator`
- `api-docs-writer`
- `security-analyzer`

**Bad examples:**
- `helper` (too generic)
- `-agent-` (starts/ends with hyphen)
- `my_agent` (underscores not allowed)
- `ag` (too short, < 3 chars)

### description (required)

Defines when Claude should trigger this agent. **This is the most critical field.**

**Must include:**
1. Triggering conditions ("Use this agent when...")
2. Multiple `<example>` blocks showing usage
3. Context, user request, and assistant response in each example
4. `<commentary>` explaining why agent triggers

**Length:** 10-5,000 characters
**Best:** 200-1,000 characters with 2-4 examples

**Format:**

```text
Use this agent when [conditions]. Examples:

<example>
Context: [Scenario description]
user: "[What user says]"
assistant: "[How Claude should respond]"
<commentary>
[Why this agent is appropriate]
</commentary>
</example>

[More examples...]
```

**Best practices:**
- Include 2-4 concrete examples
- Show proactive and reactive triggering
- Cover different phrasings of same intent
- Explain reasoning in commentary
- Be specific about when NOT to use the agent

### model (required)

Which model the agent should use.

| Option | Description | Use For |
|--------|-------------|---------|
| `inherit` | Same as parent (recommended) | Default choice |
| `sonnet` | Claude Sonnet (balanced) | Developers, debuggers |
| `opus` | Claude Opus (most capable) | Architects, complex decisions |
| `haiku` | Claude Haiku (fast, cheap) | Simple validators, quick tasks |

### color (required)

Visual identifier for agent in UI.

| Color | Use For |
|-------|---------|
| blue/cyan | Analysis, review, architecture |
| green | Generation, creation, success |
| yellow | Validation, caution, warnings |
| red | Security, critical, destructive |
| magenta | Creative, transformation |

### tools (optional)

Restrict agent to specific tools.

**Format:** Array of tool names

```yaml
tools: ["Read", "Write", "Grep", "Bash"]
```

**Default:** If omitted, agent has access to all tools

**Common tool sets:**

| Use Case | Tools |
|----------|-------|
| Read-only analysis | `["Read", "Grep", "Glob"]` |
| Code generation | `["Read", "Write", "Grep"]` |
| Testing | `["Read", "Bash", "Grep"]` |
| Full access | Omit field or use `["*"]` |

**Best practice:** Limit tools to minimum needed (principle of least privilege)

## Workflow 1: Build New Agent

### Step 1: Define Agent Purpose

Answer these questions:

- What domain does this agent specialize in?
- When should Claude invoke this agent?
- What tools does it need access to?
- How complex are its tasks? (determines pattern)

### Step 2: Choose System Prompt Pattern

| Pattern | Word Count | Model | Use For |
|---------|------------|-------|---------|
| Architect | 10,000-15,000 | opus | Backend, cloud, database, K8s architects |
| Developer | 3,000-8,000 | sonnet/inherit | Frontend, mobile, feature developers |
| Responder | 2,000-4,000 | sonnet/haiku | Incident response, debugging, quick tasks |

### Step 3: Write Frontmatter

```yaml
---
name: agent-name
description: Use this agent when [specific conditions]. [Expertise description]. Masters [technologies]. Use PROACTIVELY when [trigger scenarios]. Examples:

<example>
Context: [Situation that triggers agent]
user: "[User's request]"
assistant: "I'll use the [agent-name] agent to [action]."
<commentary>
[Why this agent is appropriate]
</commentary>
</example>

<example>
Context: [Another scenario]
user: "[Request]"
assistant: "[Response using agent]"
<commentary>
[Reasoning]
</commentary>
</example>

model: inherit
color: blue
tools: ["Read", "Write", "Grep"]
---
```

### Step 4: Write System Prompt

**Pattern 1: Architect (Large - 10k+ words)**

```markdown
You are [Domain] Architect specializing in [specific areas].

**Expert Purpose:**
[Comprehensive description of expertise - 2-3 paragraphs]

**Core Capabilities:**

### [Area 1]
- **[Sub-topic]**: [Details with sub-items]
- **[Sub-topic]**: [Details]

### [Area 2]
[Continue with 8-12 capability areas]

**Behavioral Traits:**
- [6-10 personality/approach traits]
- Proactively identifies architectural risks
- Balances ideal solutions with pragmatic constraints
- Documents decisions and rationale

**Knowledge Base:**
- [Technology 1]: [Expertise level and specifics]
- [Technology 2]: [Expertise level]

**Response Approach:**
1. Understand the full context and constraints
2. Identify architectural implications
3. Consider multiple approaches
4. Evaluate trade-offs
5. Recommend with clear rationale
6. Provide implementation guidance
7. Document decisions
8. Consider future maintainability

**Example Interactions:**
- "Design an API for..." → Analyze requirements, propose structure, document decisions
- "How should we scale..." → Evaluate options, recommend approach, plan implementation

**Workflow Position:**
- **After**: Requirements gathering, initial planning
- **Complements**: Backend developers, DevOps engineers
- **Enables**: Implementation teams, code reviewers

**Output Format:**
Provide architectural recommendations as:
- Executive summary (2-3 sentences)
- Detailed analysis (structured sections)
- Decision rationale (why this approach)
- Implementation guidance (next steps)
- Risk considerations (what could go wrong)
```

**Pattern 2: Developer (Medium - 3-8k words)**

```markdown
You are [Domain] Developer specializing in [frameworks/technologies].

**Expert Purpose:**
[Clear focus statement - 1 paragraph]

**Core Capabilities:**
1. [Primary capability with details]
2. [Secondary capability]
3. [Additional capabilities - 5-8 total]

**Modern Stack Focus:**
- [Framework 1]: [Version/approach]
- [Framework 2]: [Details]

**Best Practices:**
- [Practice 1]
- [Practice 2]

**Response Approach:**
1. Understand requirements and constraints
2. Check existing patterns in codebase
3. Implement following established standards
4. Verify functionality works correctly

**Example Interactions:**
- "Build a component..." → Check existing patterns, implement, test
- "Fix this issue..." → Diagnose, implement fix, verify

**Output Format:**
- Working code with inline comments
- Explanation of key decisions
- Usage examples if applicable
```

**Pattern 3: Responder (Small - 2-4k words)**

```markdown
You are [Domain] Responder specializing in [area].

**Expert Purpose:**
Rapid [problem type] resolution with [approach].

**Immediate Actions:**
1. [First 5 minutes actions]
2. [Triage steps]

**Severity Matrix:**

| Level | Impact | Response Time | Actions |
|-------|--------|---------------|---------|
| P0 | Critical | Immediate | [Actions] |
| P1 | High | 15 min | [Actions] |
| P2 | Medium | 1 hour | [Actions] |

**Diagnostic Process:**
1. [Step 1]
2. [Step 2]
3. [Step 3]

**Common Pitfalls:**
- [Mistake 1]: [How to avoid]
- [Mistake 2]: [Correct approach]

**Output Format:**
- Status: [Current state]
- Diagnosis: [What's wrong]
- Action: [What to do]
- Timeline: [Expected resolution]
```

### Step 5: Save and Validate

Save to: `agents/[name].md`

Run validation checklist.

## Workflow 2: Improve Existing Agent

### Step 1: Analyze Current State

```bash
# Read agent file
cat agents/[name].md

# Check for issues:
# - Missing examples in description?
# - System prompt too short/long?
# - Wrong model for complexity?
# - Missing tools restriction?
```

### Step 2: Gap Analysis

| Component | Check | Common Issues |
|-----------|-------|---------------|
| name | lowercase-hyphens? | Spaces, uppercase, too short |
| description | Has examples? | Missing `<example>` blocks |
| description | PROACTIVELY triggers? | Only reactive triggers |
| model | Matches complexity? | opus for simple, haiku for complex |
| color | Semantic meaning? | Random color choice |
| system prompt | Has sections? | Missing capabilities/output format |
| system prompt | Right length? | Architect <500 words = too short |

### Step 3: Apply Fixes

**Adding examples to description:**

```yaml
description: ... Examples:

<example>
Context: [Scenario]
user: "[Request]"
assistant: "I'll use [agent] to [action]."
<commentary>
[Why appropriate]
</commentary>
</example>
```

**Adding Workflow Position:**

```markdown
**Workflow Position:**
- **After**: [What happens before this agent]
- **Complements**: [Related agents]
- **Enables**: [What this agent enables]
```

**Expanding Core Capabilities:**

Add 8-12 capability areas for architects, 5-8 for developers.

**Adding JARVIS Integration:**

```markdown
**JARVIS Integration:**
- Reference category's Primary Skill for domain knowledge
- Use category's MCP tools when available
- Follow category's established patterns
```

### Step 4: Validate

Run full validation checklist.

## Workflow 3: Adapt Marketplace Agent

When taking an agent from wshobson-agents, obra-superpowers, or similar:

### Step 1: Read Original Agent

```bash
cat marketplace-plugin/agents/[agent].md
```

Note:
- System prompt structure
- Capabilities covered
- Behavioral traits
- Response patterns

### Step 2: Identify JARVIS Fit

| Original Focus | JARVIS Target |
|----------------|---------------|
| Orchestration | Plugin-Orchestrator |
| Self-improvement | plugin-dev |
| Data/analytics | Plugin-BigQuery-[Cat] |
| Domain-specific | Plugin-Category-[Cat] |

### Step 3: Adapt Description

**Original (generic):**

```yaml
description: Expert backend architect for designing scalable APIs...
```

**Adapted (JARVIS-specific):**

```yaml
description: Expert backend architect for JARVIS ecosystem. Use when designing APIs for MCP servers, planning microservices architecture, or establishing backend patterns for categories. Use PROACTIVELY when starting backend development. Examples:

<example>
Context: Creating MCP server for new category
user: "Design the API for the Asana MCP server"
assistant: "I'll use the backend-architect agent to design the API structure."
<commentary>
MCP server creation requires careful API design - this agent specializes in this.
</commentary>
</example>
```

### Step 4: Add Workflow Position

```markdown
**Workflow Position:**
- **After**: Category creation, requirements gathering
- **Complements**: frontend-developer, database-architect
- **Enables**: MCP implementation, testing
```

### Step 5: Adjust for Category Context

Add category-specific references:

```markdown
**JARVIS Integration:**
- Reference category's Primary Skill for domain knowledge
- Use category's MCP tools when available
- Follow category's established patterns
- Check BigQuery for relevant data
```

### Step 6: Validate Adaptation

Run full validation checklist.

## Agent Organization

### Plugin Agents Directory

```
plugin-name/
└── agents/
    ├── analyzer.md
    ├── reviewer.md
    └── generator.md
```

All `.md` files in `agents/` are auto-discovered.

### Namespacing

Agents are namespaced automatically:
- Single plugin: `agent-name`
- With subdirectories: `plugin:subdir:agent-name`

### Multiple Plugins

When multiple plugins have agents:
- Each plugin's agents have distinct namespace
- Claude combines all available agents
- Avoid name conflicts across plugins

## Testing Agents

### Test Triggering

Create test scenarios to verify agent triggers correctly:

1. Write agent with specific triggering examples
2. Use similar phrasing to examples in test
3. Check Claude loads the agent
4. Verify agent provides expected functionality

### Test System Prompt

Ensure system prompt is complete:

1. Give agent typical task
2. Check it follows process steps
3. Verify output format is correct
4. Test edge cases mentioned in prompt
5. Confirm quality standards are met

### Test Commands

```bash
# Validate agent structure
# Check frontmatter fields
cat agents/my-agent.md | head -20

# Check for required sections in system prompt
grep -E "Core Capabilities|Response Approach|Output Format" agents/my-agent.md
```

## Minimal Agent Template

For quick agent creation:

```markdown
---
name: simple-agent
description: Use this agent when [condition]. Examples:

<example>
Context: [Scenario]
user: "[Request]"
assistant: "Using simple-agent to [action]."
<commentary>
[Why this agent fits]
</commentary>
</example>

model: inherit
color: blue
---

You are an agent that [does X].

**Process:**
1. [Step 1]
2. [Step 2]
3. [Step 3]

**Output:** [What to provide]
```

## Validation Checklist

### Frontmatter

- [ ] File in `agents/` directory with `.md` extension
- [ ] `name`: lowercase, hyphens only, 3-50 characters
- [ ] `name`: starts and ends with alphanumeric
- [ ] `description`: starts with "Use this agent when..."
- [ ] `description`: includes "Use PROACTIVELY when..."
- [ ] `description`: has 2-4 `<example>` blocks
- [ ] `description`: each example has Context, user, assistant, commentary
- [ ] `model`: appropriate for complexity (inherit/sonnet/opus/haiku)
- [ ] `color`: matches agent purpose semantically
- [ ] `tools`: restricted appropriately (if needed)

### System Prompt

- [ ] Opens with expert identity ("You are...")
- [ ] Has Expert Purpose section
- [ ] Has Core Capabilities (5-12 areas depending on pattern)
- [ ] Has Behavioral Traits (6-10 traits for architects)
- [ ] Has Response Approach (numbered steps)
- [ ] Has Example Interactions (5-10 examples)
- [ ] Has Output Format specification
- [ ] Has Edge Cases section (optional but recommended)
- [ ] Length appropriate for model:
  - Architect: 10,000-15,000 words
  - Developer: 3,000-8,000 words
  - Responder: 2,000-4,000 words

### Integration

- [ ] Workflow Position defined (After/Complements/Enables)
- [ ] References JARVIS tools and patterns where relevant
- [ ] No conflicts with existing agents in same plugin
- [ ] JARVIS Integration section if adapted from marketplace

## Model Selection Guide

| Agent Type | Recommended Model | Reason |
|------------|-------------------|--------|
| Architects (backend, cloud, database) | opus | Complex decisions, long prompts |
| Developers (frontend, mobile) | sonnet or inherit | Balanced speed/quality |
| Debuggers, responders | sonnet | Speed matters |
| Validators, simple checks | haiku | Fast, focused |
| Unknown/general | inherit | Use parent's model |

## Common Issues & Fixes

| Issue | Diagnosis | Fix |
|-------|-----------|-----|
| Agent never triggers | Description too vague | Add specific trigger conditions and examples |
| Agent triggers incorrectly | Examples too broad | Make examples more specific |
| Wrong complexity | haiku running architect tasks | Change model to opus |
| No examples | description lacks `<example>` | Add 2-4 real scenarios |
| Vague output | No Output Format section | Add explicit format spec |
| Generic prompt | Missing JARVIS context | Add Workflow Position, integration notes |
| Too short for architect | <500 words system prompt | Expand capabilities, add sections |
| Tools too broad | No restrictions | Add appropriate tool limits |

## Best Practices

**DO:**
- ✅ Include 2-4 concrete examples in description
- ✅ Write specific triggering conditions
- ✅ Use `inherit` for model unless specific need
- ✅ Choose appropriate tools (least privilege)
- ✅ Write clear, structured system prompts
- ✅ Test agent triggering thoroughly
- ✅ Add Workflow Position for context
- ✅ Match system prompt length to complexity

**DON'T:**
- ❌ Use generic descriptions without examples
- ❌ Omit triggering conditions
- ❌ Give all agents same color
- ❌ Grant unnecessary tool access
- ❌ Write vague system prompts
- ❌ Skip testing
- ❌ Use opus for simple tasks (wasteful)
- ❌ Use haiku for complex tasks (inadequate)

## When to Use This Skill

- User asks to create a new agent
- User asks to adapt a marketplace agent
- User asks to validate agent structure
- User asks to improve agent description or prompt
- User asks about agent frontmatter or tools
- DEV-Manager detects agent issues during improvement cycle
- Regular improvement cycle (~6 sessions)

## Sync Protocol

Before executing any workflow:

1. Read JARVIS-03's agents-management SKILL.md
2. Check for policy updates
3. Apply current policy, not cached knowledge
