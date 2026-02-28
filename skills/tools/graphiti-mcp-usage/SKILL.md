---
name: graphiti-mcp-usage
description: Workflow for using Graphiti MCP tools to capture preferences, procedures, and facts consistently.
---

# Graphiti MCP Usage Skill

This skill explains how to use the Graphiti knowledge graph effectively. Follow it whenever you need to look up or store ScreenGraph preferences, procedures, or factual relationships.

## Before You Start

1. **Search for existing knowledge.**
   - Use `search_nodes` to look for Preferences or Procedures tied to your task.
   - Use `search_facts` to explore relationships and factual context.
   - Filter by entity type (`"Preference"`, `"Procedure"`) to narrow the results.
   - Review every match before making new assumptions.
2. **Capture requirements immediately.**
   - When a user states a requirement or preference, call `add_episode` right away.
   - Split long requirements into logical chunks.
   - Explicitly mark when you are updating existing knowledge instead of creating new entries.
3. **Document procedures and facts.**
   - Confirmed workflows become Procedures.
   - Relationships between entities become Facts.
   - Tag each entry with specific categories for easier retrieval later.

## During Your Work

- **Respect preferences.** Align your work with every preference you discover.
- **Follow procedures exactly.** Execute the stored steps without deviation.
- **Apply factual context.** Use recorded facts to inform implementation details and recommendations.
- **Stay consistent.** Ensure your narrative and actions match the graph’s established knowledge.

## Best Practices

| Core Principle | Quick Action |
| --- | --- |
| Know Your Context | Run `search_nodes` for Preferences & Procedures before starting. |
| Know the Relationships | Use `search_facts` to surface supporting data. |
| Be Efficient | Filter nodes immediately by specifying `"Preference"` or `"Procedure"`. |
| Be Thorough | Review all matches to avoid redundant knowledge. |
| Don’t Lose Context | Use `add_episode` immediately for new or updated requirements. |
| Define the “How” | Record workflows as Procedures. |
| Define the “What” | Record entity relationships as Facts. |
| Tag Everything | Use clear categories for every Preference and Procedure. |

## Summary Checklist

- [ ] Search nodes for Preferences/Procedures.
- [ ] Search facts for supporting relationships.
- [ ] Capture new requirements or updates with `add_episode`.
- [ ] Store workflows as Procedures and relationships as Facts.
- [ ] Tag entries with clear categories for future retrieval.

Remember: **the Graphiti knowledge graph *is* your memory.** Use it consistently to deliver personalized, context-aware assistance.

---

## ScreenGraph-Specific Patterns

### Project Identifier (CRITICAL)

**ALWAYS use `group_id="screengraph"` for ALL Graphiti operations.**

- ✅ Graphiti is shared across multiple projects
- ✅ `group_id` provides project isolation
- ✅ Use tags in `episode_body` to categorize content (e.g., `[Tags: backend, agent, appium]`)
- ❌ **NEVER use different group_ids** - ScreenGraph = `"screengraph"` always

**Common Tags for Organization:**
- `backend` - Backend/API patterns
- `frontend` - UI components, Svelte patterns
- `testing` - E2E tests, integration tests
- `debugging` - Bug fixes, workarounds
- `architecture` - Design decisions
- `devops` - CI/CD, automation
- `appium` - Device connections, WebDriver

### Common Workflow: Before Implementing

```typescript
// Step 1: Search for existing patterns
const nodes = await search_memory_nodes({
  query: "agent recovery after device disconnect",
  group_ids: ["screengraph"],
  max_nodes: 10
});

// Step 2: Search for specific facts
const facts = await search_memory_facts({
  query: "WebDriver session timeout handling",
  group_ids: ["screengraph"],
  max_facts: 10
});

// Step 3: Review results → Implement → Document
```

### Common Workflow: After Solving Bug

```typescript
// Document the solution immediately
await add_memory({
  name: "Agent Stalls on Privacy Consent Dialog",
  episode_body: `
    [Tags: backend, agent, appium, debugging]
    
    **Problem**: Agent hangs when app shows privacy consent modal
    
    **Root Cause**: Policy engine doesn't handle modal overlays
    
    **Solution**: 
    - Added pre-flight dialog detection in device-check.ts
    - Prompt user to handle consent before starting
    - Added check in EnsureDevice node
    
    **Gotchas**: 
    - Must check BEFORE starting policy execution
    - Different apps have different consent flows
    - Some consent dialogs block UI hierarchy
    
    **Files Modified**:
    - backend/agent/nodes/setup/EnsureDevice/device-check.ts
    - backend/agent/nodes/setup/EnsureDevice/node.ts
    
    **Related Issues**: BUG-015
    
    **Lessons**: Always assume first-run experience needs human intervention
  `,
  group_id: "screengraph",
  source: "text"
});
```

### Common Workflow: Before Building UI Component

```typescript
// Search for similar UI patterns
const uiPatterns = await search_memory_nodes({
  query: "Svelte 5 component with real-time updates",
  group_ids: ["screengraph"],
  max_nodes: 5
});

// Check for specific gotchas
const gotchas = await search_memory_facts({
  query: "Svelte 5 runes reactivity issues",
  group_ids: ["screengraph"],
  max_facts: 5
});
```

### Common Workflow: Documenting Architecture Decision

```typescript
// After major refactoring or design decision
await add_memory({
  name: "Agent State Machine: Single Sink Pattern",
  episode_body: `
    [Tags: backend, agent, architecture]
    
    **Decision**: Use single terminal state ("completed") instead of multiple end states
    
    **Rationale**: 
    - Simplifies state transitions
    - Easier to track run completion
    - Aligns with database schema
    
    **Alternatives Considered**:
    - Multiple terminal states (success/failure/canceled)
    - Rejected: Added complexity without clear benefit
    
    **Implementation**:
    - State machine has 1 terminal: "completed"
    - stopReason field captures why (success/error/canceled)
    - Frontend uses stopReason for UI decisions
    
    **References**: 
    - backend/agent/machine/AgentMachine.ts
    - agent_single_sink_design.md
    
    **Date**: 2025-11-05
  `,
  group_id: "screengraph",
  source: "text"
});
```

### Quick Reference: API Usage

**Search by topic:**
```typescript
search_memory_nodes({
  query: "topic",
  group_ids: ["screengraph"],
  max_nodes: 10
})
```

**Search for specific facts:**
```typescript
search_memory_facts({
  query: "question",
  group_ids: ["screengraph"],
  max_facts: 10
})
```

**Document solution:**
```typescript
add_memory({
  name: "Short Title",
  episode_body: "[Tags: category1, category2]\n\nProblem:\nSolution:\nGotchas:\nFiles:",
  group_id: "screengraph",
  source: "text"
})
```

**Retrieve recent episodes:**
```typescript
get_episodes({
  group_id: "screengraph",
  last_n: 10
})
```

---

## ScreenGraph Integration Points

### With Other MCPs

1. **Context7** → Use when you need external library docs
2. **Sequential-thinking** → Use for complex reasoning
3. **Encore-mcp** → Use for backend introspection, then document findings
4. **Playwright** → Use for frontend debugging, then document patterns

### With Task Commands

After running important Task commands, document learnings:

```typescript
// After task founder:workflows:db-reset
add_memory({
  name: "Database Reset Procedure",
  episode_body: "[Tags: devops, database]\n\nRun 'task founder:workflows:db-reset' to wipe DB. Automatically runs migrations after. Safe for dev, dangerous for prod.",
  group_id: "screengraph",
  source: "text"
})
```

### With Claude Skills

Before using complex skills (backend-debugging, e2e-testing), search Graphiti for related past issues to inform your approach.
