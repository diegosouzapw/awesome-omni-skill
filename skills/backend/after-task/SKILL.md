---
name: after-task
description: Mandatory knowledge capture after completing work. Documents solution in Graphiti and tracks effectiveness for system improvement.
---

# @after-task - Knowledge Capture

**Use AFTER:**
- Spec completed and tests passing
- Pre-push hook succeeded
- Before creating PR or merging
- Complex bug fixed
- Major refactoring done

**MANDATORY** - Don't skip this! Future you will thank you.

---

## What It Does

```
1. Document solution in Graphiti (group_id="screengraph")
   - Problem statement
   - Solution approach
   - Key learnings and gotchas
   - Files modified
   - Related specs/bugs
   
2. Optional: Track MCP effectiveness
   - Which MCPs were actually used
   - What worked well
   - What could be better
```

**Token cost**: ~600 tokens  
**Frequency**: Once per spec/major-task  
**ROI**: Future specs benefit = exponential improvement

---

## Execution

### Mandatory: Document in Graphiti

```typescript
add_memory({
  name: "[Spec/Bug Number]: [Short Title]",
  episode_body: `
    [Tags: domain, type, technology]
    
    **Problem**: [What we were trying to solve]
    
    **Solution**: [High-level approach, not code details]
    
    **Key Learnings**:
    - [Learning 1]
    - [Learning 2]
    
    **Gotchas**:
    - [Gotcha 1 with workaround]
    - [Gotcha 2 with workaround]
    
    **Files Modified**:
    - [file 1]
    - [file 2]
    
    **Tests Added**:
    - [test file 1]
    
    **Related**: [Spec-XXX, BUG-YYY, FR-ZZZ]
    
    **Date**: [YYYY-MM-DD]
  `,
  group_id: "screengraph",
  source: "text"
});
```

### Optional: Track MCP Effectiveness

```typescript
track_effectiveness({
  task: "[Original task description]",
  mcps_used: ["graphiti", "encore-mcp", "svelte"],
  outcome: "helpful",  // or "partially_helpful" or "not_helpful"
  feedback: "[What worked well or what was missing]"
});
```

**This helps the orchestrator learn and improve recommendations!**

---

## Template for Common Scenarios

### Completed Spec

```typescript
add_memory({
  name: "Spec-001: Automate Appium Lifecycle",
  episode_body: `
    [Tags: spec, backend, appium, devops, lifecycle]
    
    **Problem**: Manual Appium server management was error-prone and time-consuming
    
    **Solution**: 
    - Created lifecycle management service
    - Automated start/stop/health-check
    - Integration with agent setup flow
    
    **Key Learnings**:
    - Appium sessions need explicit health monitoring
    - Port conflicts must be detected before starting
    - Process management requires PID tracking
    
    **Gotchas**:
    - Appium doesn't expose ready signal (must poll /status)
    - Zombie processes if not cleaned up properly
    - Port 4723 conflicts with other Appium instances
    
    **Files Modified**:
    - backend/appium/lifecycle.service.ts
    - backend/agent/nodes/setup/EnsureDevice/node.ts
    - .cursor/commands/backend/Taskfile.yml
    
    **Tests Added**:
    - backend/appium/tests/lifecycle.test.ts
    
    **Related**: Spec-001
    
    **Date**: 2025-11-13
  `,
  group_id: "screengraph",
  source: "text"
});
```

### Bug Fixed

```typescript
add_memory({
  name: "BUG-015: Agent Stalls on Privacy Consent Dialog",
  episode_body: `
    [Tags: bug, backend, agent, appium, dialogs]
    
    **Problem**: Agent hung when app showed privacy consent modal
    
    **Solution**: 
    - Added pre-flight dialog detection
    - User prompted to handle consent manually
    - Check runs before policy execution starts
    
    **Key Learnings**:
    - First-run experience often has modals
    - Can't automate all user interactions
    - Human-in-loop pattern works well
    
    **Gotchas**:
    - Different apps have different consent flows
    - Some dialogs block entire UI hierarchy
    - Must check BEFORE starting automation
    
    **Files Modified**:
    - backend/agent/nodes/setup/EnsureDevice/device-check.ts
    
    **Related**: BUG-015, Spec-001
    
    **Date**: 2025-11-13
  `,
  group_id: "screengraph",
  source: "text"
});
```

### Refactoring

```typescript
add_memory({
  name: "Refactor: Agent State Machine to Single Sink Pattern",
  episode_body: `
    [Tags: refactor, backend, agent, architecture]
    
    **Problem**: Multiple terminal states added complexity
    
    **Solution**: 
    - Single "completed" state
    - stopReason field captures why (success/error/canceled)
    - Frontend uses stopReason for UI decisions
    
    **Key Learnings**:
    - Simpler state machines are easier to debug
    - stopReason pattern is flexible
    - Database schema aligned with code
    
    **Gotchas**:
    - Must migrate existing runs to new pattern
    - Frontend needs update to check stopReason
    
    **Files Modified**:
    - backend/agent/machine/AgentMachine.ts
    - backend/db/migrations/008_single_sink.up.sql
    - frontend/src/routes/runs/[runId]/+page.svelte
    
    **Related**: Architecture decision docs
    
    **Date**: 2025-11-05
  `,
  group_id: "screengraph",
  source: "text"
});
```

---

## Integration Points

### After Pre-Push
```bash
# Pre-push succeeded
git push origin feature-X

# Now document (BEFORE creating PR)
@after-task [completed spec/task]

# Review documentation template
# Fill in learnings
# Add to Graphiti
```

### After PR Merge
```bash
# PR merged to main

# Document if not done already
@after-task [completed work]
```

---

## Quality Standards

**Good documentation includes:**
- ✅ Clear problem statement
- ✅ High-level solution (not code details)
- ✅ Specific gotchas with workarounds
- ✅ ALL files modified
- ✅ Related specs/bugs linked
- ✅ Proper tags for categorization

**Bad documentation:**
- ❌ Just code snippets
- ❌ No gotchas mentioned
- ❌ Vague problem statement
- ❌ Missing file references
- ❌ No tags

---

## Why This is Mandatory

**If you skip @after-task:**
- ❌ Knowledge is lost
- ❌ Next person hits same gotchas
- ❌ Patterns aren't discovered
- ❌ System doesn't improve

**If you run @after-task:**
- ✅ Knowledge compounds
- ✅ Future specs are faster
- ✅ Gotchas documented once, avoided forever
- ✅ System gets exponentially smarter

**Future you will thank present you.**

---

## Enforcement

From `founder_rules.mdc`:
```markdown
**After Completing Task:**
1. ✅ Document solution via Graphiti
2. ✅ Include: problem, solution, gotchas, files
3. ✅ Use tags for organization
```

**This isn't optional. It's how the system improves.**

---

**Purpose**: Capture institutional knowledge so future work benefits from past work. The self-improvement loop closes here.


