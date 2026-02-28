---
name: integration-workflows
description: Cross-MCP workflows that coordinate multiple systems (Linear, GitHub, n8n, Slack) for end-to-end automation. Captures patterns that span tool boundaries.
---

# Integration Workflows

## Purpose
This skill captures workflows that use MULTIPLE MCP servers together. When a pattern emerges that coordinates Linear + GitHub + n8n + Slack (or any combination), document it here.

## Current Status
**Placeholder skill - Will grow as we discover integration patterns**

This skill intentionally starts small and will evolve as we build real integrations.

## When to Add to This Skill

Add a workflow here when:
- ✅ It uses 2+ different MCP servers
- ✅ The workflow has been tested and works
- ✅ It's repeatable (not one-off)
- ✅ It saves significant time/effort

## Documented Workflows

### [Future: YouTube Knowledge Extractor]
**Status:** In Development
**MCPs Used:** n8n, Linear, GitHub
**Description:** Will document once workflow is complete and working

### [Future: Feature Development Pipeline]
**Status:** Planned
**MCPs Used:** Linear, GitHub, Slack
**Description:** Issue creation → branch → PR → review → merge → notify

### [Future: Bug Triage Automation]
**Status:** Planned  
**MCPs Used:** Linear, GitHub, n8n
**Description:** Bug report → analysis → assignment → tracking

## Template for New Workflows

When adding a workflow, use this template:

```markdown
### Workflow Name

**MCPs Used:** [List]
**Trigger:** [What starts this workflow]
**Outcome:** [What's accomplished]

#### Steps:
1. **[MCP Name]**: [Action]
   - Tool: [specific tool]
   - Purpose: [why this step]

2. **[MCP Name]**: [Action]
   - Tool: [specific tool]
   - Purpose: [why this step]

[etc.]

#### Error Handling:
- If [condition]: [fallback]

#### Success Criteria:
- [How to verify it worked]
```

## Meta-Pattern: Skill Improvement Loop

**When improving any skill:**
1. Update the skill itself
2. If related to forked MCP → update MCP repo
3. Document the pattern that led to improvement
4. Consider if it's actually an integration pattern (belongs here)

## Notes
- Version: 1.0.0 (Placeholder)
- Created: 2025-10-18
- Author: Kurt Anderson
- Will grow organically as integration patterns emerge
