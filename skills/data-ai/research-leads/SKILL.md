---
name: research-leads
description: Research new capabilities and changes for tracked AI coding agents. Use this skill when assigned a research-leads issue to discover new features, or when asked to revise a research PR.
---

# Research Leads Skill

You proactively research developments for AI coding agents tracked in this repository.

## Context

This repository tracks capabilities of these AI coding agents:
- **claude-code** (Anthropic) — CLI agent for code generation, editing, debugging
- **copilot-cli** (GitHub) — CLI companion for terminal workflows
- **gemini-cli** (Google) — CLI agent powered by Gemini models
- **vscode-copilot** (GitHub) — VS Code integrated AI coding assistant

Each agent's capabilities are stored in `agents/<name>/capabilities/current.json`.

## Lead Types

Research these categories of changes:

1. **New capability** — A feature not yet tracked (e.g. a new tool, mode, or integration)
2. **Changed capability** — An existing feature that evolved significantly (renamed, expanded, restructured)
3. **Version / model release** — A new version, model update, or platform change
4. **Deprecation notice** — A feature being removed or replaced
5. **New agent** — A new AI coding agent worth adding to the tracker

## Confidence Levels

- **High** — Official documentation or changelog explicitly confirms the change
- **Medium** — Blog post, release notes, or credible announcement confirms it
- **Low** — Inference from indirect evidence only (pre-uncheck these in the PR)

## Research Mode

When assigned an issue labelled `research-leads`:

### Step 1: Understand current state

Read each `agents/<name>/capabilities/current.json` to understand what's already tracked. Note capability names, descriptions, and source URLs.

### Step 2: Research each agent

For each tracked agent, check:
- Official documentation sites for new or restructured pages
- Changelog / release notes for recent updates
- GitHub releases (for open-source agents)
- Product announcement blogs
- Any other authoritative sources you find

Look for capabilities, features, or integrations not yet in the data.

### Step 3: Create the PR

Open a PR with the actual proposed changes to `current.json` files in the diff.

**PR description format — this IS the triage interface:**

```markdown
## Research Leads — YYYY-MM-DD

Uncheck items to skip. Edit inline to correct a source URL.
Comment `@copilot revise` after making changes.

---

### claude-code (N leads)
- [x] **New: <Capability Name>** · High confidence
      Source: <url>
      Action: Add entry to current.json

- [x] **Update: <Capability Name>** · Medium confidence
      Source: <url>
      Action: Update description / add source

- [ ] **New: <Capability Name>** · Low confidence
      Source: <url>
      Action: Add entry (excluded by default — check to include)

### gemini-cli (N leads)
...

### No leads
- vscode-copilot: No new developments found
```

### Step 4: Validate

Before creating the PR, run:
```bash
python3 framework/scripts/validate_framework.py
```

Fix any errors before committing.

## Revise Mode

When someone comments `@copilot revise` on a research PR:

1. Re-read the PR description to see which items are checked/unchecked
2. Remove commits/changes for unchecked items
3. Apply any inline edits the reviewer made (corrected URLs, descriptions)
4. Re-run `python3 framework/scripts/validate_framework.py`
5. Push the updated commits

## Schema Reference

Each capability entry in `current.json` must have:

```json
{
  "category": "<one of: code-completion, code-generation, chat-assistance, code-explanation, code-refactoring, testing, debugging, documentation, command-line, multi-file-editing, context-awareness, language-support, ide-integration, api-integration, customization, security, performance, collaboration, model-selection, agent-orchestration, observability>",
  "name": "Capability Name",
  "description": "What this capability does, in 1-2 sentences",
  "available": true,
  "tier": "<free|pro|business|enterprise>",
  "maturityLevel": "<experimental|beta|stable|deprecated>",
  "status": "active",
  "sources": [
    {
      "url": "https://...",
      "description": "Brief source label",
      "verifiedDate": "YYYY-MM-DD",
      "sourceGranularity": "<dedicated|section|excerpt>",
      "excerpt": "Required 50-300 char verbatim quote when sourceGranularity is excerpt"
    }
  ]
}
```

## Adding a New Agent

If you discover a new AI coding agent worth tracking:

1. Create the directory: `agents/<agent-name>/capabilities/`
2. Create `current.json` with the agent info and initial capabilities
3. Create `agents/<agent-name>/docs-registry.json` listing authoritative doc sources
4. Run `python3 framework/scripts/validate_framework.py` to confirm schema compliance
5. Mark this lead as "New agent" in the PR description with Low confidence

## Important Rules

- NEVER fabricate capabilities. Every lead must cite a real, accessible source URL.
- Verify all source URLs return HTTP 200: `curl -Is URL | head -1`
- Pre-uncheck low-confidence leads so the reviewer must opt in.
- When adding a source with `sourceGranularity: "excerpt"`, include a verbatim 50-300 character quote from the page.
- Commit messages: `research: add <capability> to <agent>` or `research: update <capability> for <agent>`
