---
name: agent-teams
description: Orchestrate Gemini CLI agent teams for parallel multi-agent collaboration
category: orchestration
---

# Agent Teams Skill

## Purpose
Guide effective use of Gemini CLI's agent teams feature for parallel multi-agent collaboration with best practices, templates, and orchestration patterns.

## Prerequisites

**Enable agent teams** in `~/.gemini/settings.json`:
```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

## When to Use Agent Teams

### ✅ Good Use Cases

**Research and Review** (Best ROI):
- Multiple teammates investigate different aspects simultaneously
- Teammates challenge each other's findings
- Example: "Spawn 3 reviewers - security, performance, test coverage"

**New Features** (Parallel Development):
- Each teammate owns separate layer/module
- Minimal file overlap
- Example: "Spawn frontend, backend, and security reviewer teammates"

**Debugging with Competing Hypotheses**:
- Teammates test different theories in parallel
- Converge on answer faster through debate
- Example: "Spawn 5 teammates to investigate different root causes"

**Cross-Layer Coordination**:
- Changes span frontend, backend, tests
- Each teammate owns different concern
- Example: "Spawn API designer, implementer, and test engineer"

### ❌ Poor Use Cases

- Sequential tasks (use single session)
- Same-file edits (conflict risk - use single session)
- Simple tasks (overhead not worth it - use subagents)
- Work with many dependencies (coordination overhead)

## Team Templates

### 1. Code Review Team (3 reviewers)

```
Create an agent team to review PR #{PR_NUMBER}. Spawn three reviewers:

1. **Security Reviewer** (Opus for critical thinking)
   - Focus: Security vulnerabilities, auth, input validation
   - Check: SQL injection, XSS, CSRF, secrets exposure
   - Severity: Rate findings as critical/high/medium/low

2. **Performance Reviewer** (Sonnet)
   - Focus: Performance implications, scalability
   - Check: N+1 queries, memory leaks, algorithm complexity
   - Measure: Impact on latency, throughput, resource usage

3. **Test Coverage Reviewer** (Sonnet)
   - Focus: Test adequacy, edge cases
   - Check: Missing tests, untested paths, flaky tests
   - Ensure: All new code has tests, critical paths covered

Have each reviewer:
- Review independently
- Share findings with other reviewers
- Challenge assumptions
- Update shared findings document

After all reviews complete, synthesize into single report with:
- Critical blockers (must fix before merge)
- Important issues (should fix)
- Suggestions (nice to have)
```

### 2. Feature Development Team (3 specialists)

```
Create an agent team to implement [FEATURE_NAME]. Use delegate mode.

1. **Backend Engineer** (Sonnet)
   - Owns: API endpoints, business logic, database
   - Files: src/api/*, src/services/*, migrations/*
   - Delivers: Tested API with docs

2. **Frontend Engineer** (Sonnet)
   - Owns: UI components, state management, routing
   - Files: src/components/*, src/pages/*, src/hooks/*
   - Delivers: Tested components with Storybook

3. **Security Reviewer** (Opus)
   - Monitors: Both backend and frontend work
   - Reviews: Each implementation before merge
   - Blocks: Insecure patterns, missing validation

Workflow:
1. Backend implements API first
2. Security reviews backend (blocks if issues)
3. Frontend builds UI against API
4. Security reviews frontend (blocks if issues)
5. Integration testing
6. Synthesize PR description

Task dependencies:
- Frontend blocked by backend completion
- All blocked by security approval
```

### 3. Research Team (5 investigators)

```
Research [TOPIC] using agent team with competing perspectives:

1. **Optimist** - Find why this is the best approach
2. **Pessimist** - Find why this won't work
3. **Pragmatist** - Find middle ground, real-world constraints
4. **Architect** - Evaluate technical feasibility, integration
5. **Devil's Advocate** - Challenge all assumptions, find edge cases

Process:
1. Each teammate researches independently (30 min)
2. Share findings via broadcast
3. Debate findings - try to disprove each other
4. Iterate: respond to challenges with evidence
5. Converge on consensus (or documented disagreement)

Output: Research document with:
- Consensus findings (what everyone agrees on)
- Disagreements (what's still unclear)
- Recommendations (with confidence levels)
- Action items (what to build/test next)
```

### 4. Bug Investigation Team (4 theories)

```
Investigate [BUG_DESCRIPTION] with parallel hypothesis testing:

Spawn 4 teammates, each investigating a different theory:

1. **Frontend Theory** - "Bug is in client-side state management"
   - Check: React state, Redux, local storage
   - Test: Reproduce with different browsers

2. **Backend Theory** - "Bug is in API response handling"
   - Check: API logs, database queries, error handling
   - Test: Reproduce with API mocking

3. **Race Condition Theory** - "Bug is timing-dependent"
   - Check: Async operations, promises, event handlers
   - Test: Reproduce with delays, fast/slow networks

4. **Data Theory** - "Bug is specific data triggering edge case"
   - Check: Data validation, edge cases, null handling
   - Test: Reproduce with different data samples

Process:
1. Each teammate investigates their theory independently
2. Share findings every 15 minutes
3. **Actively try to disprove other theories**
4. Theory that survives scrutiny is likely root cause
5. Once consensus reached, implement fix

Output: Root cause analysis with evidence + fix PR
```

### 5. Refactoring Team (3 layers)

```
Refactor [MODULE_NAME] using layered team approach:

1. **Architect Teammate** (Opus) - Plan approval required
   - Design: New architecture, interfaces, data flow
   - Output: ADR (Architecture Decision Record)
   - Waits: For lead approval before implementation

2. **Implementer Teammate** (Sonnet)
   - Implements: Architecture from ADR
   - Follows: Patterns from ADR exactly
   - Tests: Each component as built

3. **Migration Teammate** (Sonnet)
   - Migrates: Existing code to new architecture
   - Ensures: Backward compatibility
   - Validates: All existing tests pass

Workflow:
1. Architect designs (in plan mode)
2. Lead reviews and approves design
3. Implementer builds new architecture
4. Migration teammate updates existing code
5. Both run full test suite
6. Lead synthesizes migration guide
```

## Best Practices

### Task Sizing

**Good task size**: 30-60 minutes per task
- ✅ Self-contained unit (function, file, review)
- ✅ Clear deliverable
- ✅ Minimal dependencies

**If lead not creating enough tasks**:
```
Split the work into smaller pieces. Aim for 5-6 tasks per teammate.
```

### Context Provision

**Bad** (too vague):
```
Spawn a security reviewer
```

**Good** (specific context):
```
Spawn a security reviewer with the prompt: "Review src/auth/ for
security vulnerabilities. Focus on token handling and session management.
App uses JWT in httpOnly cookies. Check for: timing attacks, token leakage,
session fixation, CSRF. Rate findings as critical/high/medium/low."
```

### File Ownership

**Prevent conflicts** by assigning file ownership:
```
Backend teammate owns: src/api/*, src/services/*
Frontend teammate owns: src/components/*, src/pages/*
Security teammate: read-only review, no file edits
```

### Monitor Progress

**Check in regularly**:
```
What's the status of each teammate? Show me their progress.
```

**Redirect if stuck**:
```
Tell the frontend teammate to use the existing Button component
instead of creating a new one.
```

### Lead Management

**If lead starts implementing instead of delegating**:
```
Wait for your teammates to complete their tasks before proceeding.
Focus on coordination only.
```

**Use delegate mode** (Shift+Tab) to prevent lead from coding.

## Common Patterns

### Pattern 1: Adversarial Review

```
Create agent team to review [DESIGN_DOC]:
- Spawn 2 reviewers with opposing mandates
- Reviewer 1: Find why this design is brilliant
- Reviewer 2: Find why this design will fail
- Have them debate each point
- Lead synthesizes balanced perspective
```

### Pattern 2: Parallel Exploration

```
Create agent team to explore [N] different approaches to [PROBLEM]:
- Spawn N teammates (3-5 optimal)
- Each teammate prototypes different approach
- All share findings after 30 minutes
- Team votes on best approach
- Lead synthesizes decision rationale
```

### Pattern 3: Layered Implementation

```
Create agent team with staged approval:
1. Architect (plan approval required) → designs
2. Lead reviews and approves design
3. Implementer (Sonnet) → builds
4. Tester (Sonnet) → validates
5. Lead synthesizes docs
```

### Pattern 4: Swarm Intelligence

```
Create agent team to solve [HARD_PROBLEM]:
- Spawn 5 teammates with diverse approaches
- Each shares progress every 10 minutes
- Teammates borrow successful strategies from others
- Evolves toward optimal solution through iteration
- Lead tracks convergence
```

## Troubleshooting

### Teammates Not Spawning

**Check if task is complex enough**:
- Agent teams only for parallel-suitable work
- For simple tasks, Claude uses single session or subagents

**Explicitly request team**:
```
Create an agent team for this. I want to see parallel work.
```

### Too Many Permission Prompts

**Pre-approve in `~/.gemini/permissions.json`**:
```json
{
  "allow": {
    "Write": ["src/**/*"],
    "Edit": ["src/**/*"],
    "Bash": ["npm test", "npm run build"]
  }
}
```

### Teammates Conflicting

**Assign clear file ownership**:
```
Backend owns src/api/
Frontend owns src/components/
No overlap allowed.
```

### Team Runs Too Long

**Set time boundaries**:
```
Each teammate has 30 minutes to complete their investigation.
Report findings even if incomplete.
```

### Lead Doing Work Instead of Delegating

**Enable delegate mode**:
```
Press Shift+Tab to enable delegate mode
```

Or remind:
```
Stop implementing. Your job is to coordinate teammates only.
```

## Architecture

### Storage Locations

- **Team config**: `~/.gemini/teams/{team-name}/config.json`
- **Task list**: `~/.gemini/tasks/{team-name}/`
- **Teammate members**: Listed in config.json

### Communication Methods

**Direct message** (one-to-one):
```
message({recipientId: "backend-engineer", message: "..."})
```

**Broadcast** (one-to-all):
```
broadcast({message: "All teammates: share your progress"})
```

**Shared task list** (implicit coordination):
- Teammates see all tasks
- Can claim available work
- File locking prevents conflicts

## Token Cost Management

**Agent teams use ~3-5x tokens of single session**

### Minimize Cost

1. **Use Haiku for simple teammates**:
   ```
   Spawn test runner teammate with Haiku model
   ```

2. **Start small, scale up**:
   - Begin with 2-3 teammates
   - Add more only if needed

3. **Time-box work**:
   ```
   Each teammate has 30 minutes max
   ```

4. **Avoid broadcast overuse**:
   - Broadcast = message to ALL teammates
   - Costs scale with team size
   - Use direct messages when possible

### When Cost is Worth It

- ✅ Critical security review (parallel expertise)
- ✅ Complex feature (parallel implementation)
- ✅ Research (diverse perspectives)
- ❌ Simple bug fix (use single session)
- ❌ Documentation update (use single session)

## Examples

### Example 1: Security-First Feature Development

```
I'm adding payment processing. Create an agent team with security review at each step:

1. Backend Engineer (Sonnet)
   - Implement payment API
   - Use Stripe SDK
   - Store minimal PII

2. Frontend Engineer (Sonnet)
   - Build payment form
   - Use Stripe Elements (no card data on our server)
   - Handle errors gracefully

3. Security Reviewer (Opus)
   - Review each implementation before allowing next step
   - Check: PCI DSS compliance, no card data logged, HTTPS only
   - Block: Any violations

Use delegate mode. Backend goes first, must pass security review before frontend starts.
```

### Example 2: Adversarial Design Review

```
Review this architecture design document: [DOC_URL]

Create agent team with adversarial approach:

1. **Advocate** - Find strengths, why this design is optimal
2. **Critic** - Find weaknesses, why this design will fail
3. **Moderator** - Synthesize debate, propose improvements

Process:
- Advocate and Critic read doc independently
- Each presents case (5 minutes)
- Debate each point (20 minutes)
- Moderator synthesizes balanced view
- Lead creates updated design with improvements
```

### Example 3: Parallel Hypothesis Testing

```
Our API is slow. Create agent team to test different optimization theories:

1. **Database Theory** - "Queries are slow"
   - Profile queries with EXPLAIN ANALYZE
   - Test: Add indexes, check improvement

2. **Network Theory** - "Network latency is high"
   - Profile with network inspector
   - Test: Add CDN, check improvement

3. **Code Theory** - "Algorithm is inefficient"
   - Profile with performance tools
   - Test: Optimize hot paths, check improvement

4. **Scale Theory** - "System is overloaded"
   - Check: CPU, memory, disk I/O
   - Test: Vertical scaling, check improvement

Each teammate:
- Measures baseline performance
- Implements their optimization
- Measures improvement
- Shares results with team

Winner: Theory with biggest measurable improvement
```

## Integration with Other Skills

**Combine with**:
- **brain-briefing**: Get context before spawning team
- **task-processor**: Break PRD into tasks for team
- **pr-review-loop**: Use team for multi-reviewer approval

## Limitations (Experimental)

- No session resumption with in-process teammates
- Task status can lag (manually update if needed)
- Shutdown can be slow (teammates finish current work)
- One team per session (clean up before new team)
- No nested teams (teammates can't spawn teams)
- Lead is fixed (can't promote teammate)

## Quick Reference

### Start Team
```
Create an agent team for [TASK]. Spawn [N] teammates: [ROLES]
```

### Talk to Teammate
```
Shift+Up/Down to select, then type message
```

### Check Progress
```
What's the status of each teammate?
```

### Shut Down Teammate
```
Ask [teammate-name] to shut down
```

### Clean Up
```
Clean up the team
```

### Enable Delegate Mode
```
Press Shift+Tab
```

---

**Created**: 2026-02-06
**Requires**: `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`
**Documentation**: `.gemini/docs/AGENT_TEAMS.md`
