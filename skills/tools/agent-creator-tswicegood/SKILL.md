---
name: agent-creator
description: Interview-based system for creating specialized Claude Code agents. Use when users want to create a Claude Code agent, need help defining agent roles and responsibilities, mention "create an agent" or "build an agent", or want to generate a CLAUDE.md file for agentic coding workflows. Guides through interactive interview to gather requirements and generates complete agent configurations following Claude Code best practices.
---

# Agent Creator for Claude Code

Create specialized Claude Code agents through an interactive interview process. Generates complete CLAUDE.md configurations with workflows, constraints, and quality gates tailored to specific roles.

## Interview Process

Follow this 7-phase interview to gather requirements, then generate a complete agent configuration:

### Phase 1: Role & Purpose (2-3 questions)

**Goal**: Understand the agent's identity and primary purpose.

**Questions**:
1. "What would you like this agent to do? Describe the main role in a sentence or two."
2. "What problem are you trying to solve with this agent?"
3. "How would you name this agent role?" (e.g., 'Code Reviewer', 'Test Engineer', 'API Guardian')

**Learning**: The agent's identity, purpose, and the gap it fills.

**Follow-up if vague**: "Can you give me an example of a task this agent would handle?"

### Phase 2: Responsibilities (3-4 questions)

**Goal**: Define MUST do vs SHOULD do vs MUST NOT do.

**Questions**:
1. "What are the 3-5 core duties this agent MUST perform every time?"
2. "Are there secondary responsibilities it SHOULD handle when relevant?"
3. "What is explicitly outside this agent's scope? What should it NOT do?"
4. "Are there situations where the agent should stop and ask before proceeding?"

**Learning**: Clear boundaries and priorities.

**Follow-up if too broad**: "Let's focus on the most important duty - can you break that down into specific steps?"

### Phase 3: Autonomy & Constraints (2-3 questions)

**Goal**: Determine autonomy level and risk tolerance.

**Questions**:
1. "How much autonomy should this agent have?"
   - **High**: Makes decisions and implements independently
   - **Medium**: Proposes plan, gets approval, then implements
   - **Low**: Only analyzes and advises, never modifies
2. "What mistakes would be unacceptable for this agent to make?"
3. "Are there specific files, directories, or systems the agent should never touch?"

**Learning**: Risk tolerance and guardrails.

**Follow-up if unsure**: "Would you be comfortable with this agent committing code while you're away?"

### Phase 4: Technical Context (3-5 questions)

**Goal**: Understand the codebase and technical environment.

**Questions**:
1. "What's your primary programming language(s)?" (e.g., TypeScript, Python, Go)
2. "What framework or stack are you using?" (e.g., Next.js, Django, FastAPI)
3. "What testing tools do you use?" (e.g., Jest, pytest, go test)
4. "Do you have specific code style preferences or standards?"
5. "Are there key architectural patterns this agent should follow?"

**Learning**: Technical environment and conventions.

**Follow-ups**:
- "Where can I find examples of 'good' code in your project?"
- "Do you have documentation files the agent should reference?"
- "What commands does the agent need to run regularly?" (test, lint, build)

### Phase 5: Workflow & Process (3-4 questions)

**Goal**: Define the step-by-step operational procedure.

**Questions**:
1. "When you use this agent, what's the typical workflow? Walk me through the steps."
2. "What should the agent do FIRST every time?"
3. "How do you want the agent to communicate findings or results?"
4. "What must be true before the agent considers its work 'complete'?" (quality gates)

**Learning**: Operational procedure and definition of done.

**Offer common patterns**:
- **Research → Plan → Implement**: Good for features
- **Analyze → Report**: Good for audits/reviews
- **Test → Code → Test**: Good for TDD
- **Scan → Flag → Suggest**: Good for quality checks

### Phase 6: Success Criteria (2-3 questions)

**Goal**: Define measurable success.

**Questions**:
1. "How will you know if this agent is working well? What does success look like?"
2. "Can you describe an example of a great outcome from this agent?"
3. "What would a poor outcome look like? What should I watch out for?"

**Learning**: How to measure effectiveness and failure modes to avoid.

**Follow-up**: "What metrics matter most? Time saved? Issues caught? Code quality?"

### Phase 7: Custom Commands (1-2 questions, optional)

**Goal**: Identify repeated workflows needing shortcuts.

**Questions**:
1. "Are there specific workflows you'll run repeatedly that deserve a shortcut command?"
2. "What would you name these commands, and what would they do?"

**Examples to offer**:
- `/project:review` - Full review with checklist
- `/project:quick-scan` - Fast critical-issues-only scan
- `/project:generate-tests` - Create test suite for a file

**Note**: Many agents work fine without custom commands.

## After Interview: Generate Agent

Once all phases are complete, generate these artifacts:

### 1. Complete CLAUDE.md File

Structure using XML-style semantic sections. See references/claude-md-template.md for the complete template.

Key sections:
- `<role>` - Agent identity and expertise
- `<responsibilities>` - Core and secondary duties
- `<constraints>` - Must NOT do, autonomy level, when to ask
- `<workflow>` - Step-by-step process with quality gates
- `<patterns>` - Good/bad code examples
- `<communication_style>` - How to interact with user
- `<success_criteria>` - What good looks like

### 2. Custom Command Files (if any)

Format: `.claude/commands/command-name.md`

Example structure:
```markdown
[Command description]

Usage: /project:command-name [ARGS]

Steps:
1. [Step 1]
2. [Step 2]

$ARGUMENTS will be replaced with user input
```

### 3. Quick Start Guide

Brief usage instructions showing how to deploy and use the agent.

## Key Principles for Agent Design

### Token Efficiency
- Write "what" not "how" (Claude already knows how to code)
- Use examples from actual codebase, not generic examples
- Document only what Claude doesn't already know

### Progressive Disclosure
- Put frequently-needed info at top of CLAUDE.md
- Reference detailed docs via file paths (don't inline everything)
- Use XML tags to create scannable sections

### Clear Boundaries
- Make constraints explicit with ❌ symbols
- Define quality gates with checkboxes
- Specify when to ask for approval

### Concrete Examples
- Show good vs bad code patterns
- Use actual code from the codebase when possible
- Prefer specific examples over abstract rules

## Resources

For detailed examples and templates:

- **references/agent-templates.md** - Pre-built agent templates for common roles
- **references/claude-md-template.md** - Complete CLAUDE.md structure template
- **references/best-practices.md** - Tips, troubleshooting, and advanced patterns

## Interview Best Practices

**Do**:
- ✅ Ask one question at a time
- ✅ Use the user's language and terminology
- ✅ Provide examples when concepts are abstract
- ✅ Confirm understanding before moving to next phase
- ✅ Take note of specific examples they mention

**Don't**:
- ❌ Jump ahead - follow phases in order
- ❌ Make assumptions - ask clarifying questions
- ❌ Rush through - this is a design process
- ❌ Generate anything until all phases complete

## Adaptation

**For experienced users**: Move faster, skip basic explanations

**For new users**: Provide more context and examples

**For technical leads**: Focus on team consistency and standards

**For individual devs**: Focus on personal workflow optimization
