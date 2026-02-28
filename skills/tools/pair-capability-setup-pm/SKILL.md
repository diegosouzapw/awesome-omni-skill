---
name: pair-capability-setup-pm
description: "Configures the project management tool by guiding selection from supported options, applying the implementation guide, updating way-of-working adoption, and recording the decision via /pair-capability-record-decision. Idempotent — detects existing configuration."
version: 0.4.1
author: Foomakers
---

# /pair-capability-setup-pm — PM Tool Configuration

Configure the project management tool for the current project. Guides the developer through tool selection, applies the implementation guide, updates adoption files, and records the decision.

## Arguments

| Argument | Required | Description                                                                                 |
| -------- | -------- | ------------------------------------------------------------------------------------------- |
| `$tool`  | No       | PM tool to configure (e.g., `github`, `filesystem`). If omitted, presents selection options. |

## Composed Skills

| Skill              | Type       | Required                                                    |
| ------------------ | ---------- | ----------------------------------------------------------- |
| `/pair-capability-record-decision` | Capability | Yes — records PM tool choice as ADL entry + adoption update |

## Algorithm

### Step 1: Detect Existing Configuration

1. **Check**: Read [adoption/tech/way-of-working.md](../../../.pair/adoption/tech/way-of-working.md). Does it contain a PM tool configuration (e.g., "Github Projects is adopted" or "Filesystem is adopted" for project management)?
2. **Skip** (not configured): Proceed to Step 2.
3. **Act** (already configured): Present current configuration:

   > PM tool already configured: **[tool name]**.
   > - Current setup: [summary of PM section from way-of-working.md]
   >
   > Options:
   > 1. **Keep current** — no changes needed
   > 2. **Reconfigure** — switch to a different PM tool

   - If **Keep current** → stop, output current state.
   - If **Reconfigure** → proceed to Step 2.

4. **Verify**: Mode is `configure` or `done`.

### Step 2: Select PM Tool

1. **Check**: Is `$tool` provided and valid?
2. **Skip**: If valid `$tool`, proceed to Step 3 with that tool.
3. **Act**: Present PM tool options using the [selection framework](../../../.pair/knowledge/guidelines/collaboration/project-management-tool/README.md):

   > **Select a project management tool:**
   >
   > | Tool | Best For | Implementation Guide |
   > |------|----------|---------------------|
   > | **GitHub Projects** | Teams using GitHub, remote collaboration, any size | Available |
   > | **Filesystem** | Small teams, offline work, high security | Available |
   > | **Other** (Jira, Linear, Azure DevOps, etc.) | Enterprise, complex workflows | No implementation guide yet |
   >
   > Which tool does your team use or want to adopt?

4. **Act**: If developer selects a tool without an implementation guide → **HALT**:

   > No implementation guide available for **[tool name]**. To add support:
   > - Create `guidelines/collaboration/project-management-tool/<tool>-implementation.md`
   > - Follow the structure of existing implementation guides
   >
   > For now, you can manually configure `adoption/tech/way-of-working.md`.

5. **Verify**: Tool selected with available implementation guide.

### Step 3: Apply Implementation Guide

1. **Act**: Read the implementation guide for the selected tool:
   - GitHub: [github-implementation.md](../../../.pair/knowledge/guidelines/collaboration/project-management-tool/github-implementation.md)
   - Filesystem: [filesystem-implementation.md](../../../.pair/knowledge/guidelines/collaboration/project-management-tool/filesystem-implementation.md)

2. **Act**: Walk the developer through the setup steps from the implementation guide:
   - For **GitHub Projects**: project creation, board configuration, label setup, automation rules, MCP integration
   - For **Filesystem**: directory structure creation, status tracking files, workflow templates

3. **Act**: Gather project-specific details needed for configuration:
   - Project/organization name
   - Board columns and workflow methodology (Kanban, Scrum, etc.)
   - Label taxonomy
   - Automation preferences

4. **Verify**: PM tool is configured and accessible.

### Step 4: Update Way-of-Working

1. **Check**: Read current [adoption/tech/way-of-working.md](../../../.pair/adoption/tech/way-of-working.md).
2. **Act**: Add or update the PM tool section with:
   - Tool name and version/tier
   - Workflow methodology (Kanban, Scrum, etc.)
   - Project identifier (e.g., GitHub org/project name, filesystem path)
   - Access method (e.g., MCP, CLI, direct)
   - Reference to implementation guide
3. **Verify**: Way-of-working reflects current PM configuration.

### Step 5: Record Decision

1. **Act**: Compose `/pair-capability-record-decision` with:
   - `$type`: `non-architectural`
   - `$topic`: `pm-tool-choice`
   - `$summary`: "[Tool] adopted for project management with [methodology] workflow"

2. **Verify**: ADL entry created at `adoption/decision-log/YYYY-MM-DD-pm-tool-choice.md` and adoption files updated.

## Output Format

```text
PM CONFIGURED:
├── Tool:       [tool name]
├── Methodology: [Kanban | Scrum | etc.]
├── Project:    [project identifier]
├── Access:     [MCP | CLI | filesystem]
├── Adoption:   [way-of-working.md updated]
├── Decision:   [ADL entry path]
└── Status:     [Configured | Already configured (unchanged) | Reconfigured]
```

## Composition Interface

When composed by `/pair-process-bootstrap`:

- **Input**: `/pair-process-bootstrap` reaches PM configuration phase and invokes `/pair-capability-setup-pm` (optionally with `$tool` if developer pre-selected).
- **Output**: Returns tool name, configuration status, and ADL entry path.
- `/pair-process-bootstrap` includes the adoption and ADL changes in the next commit.

When invoked **independently**:

- Interactive: full Step 1-5 flow. Developer commits changes when satisfied.

## Edge Cases

- **PM tool already configured + reconfigure**: Old configuration is replaced, not appended. The new ADL entry references the previous decision if one exists.
- **No MCP connection for GitHub**: Warn that GitHub Projects requires MCP or CLI access. Offer to configure the adoption file manually and validate connectivity later.
- **Multiple PM tools**: Not supported in a single project. One tool per project. If developer needs multiple, suggest primary + secondary pattern and document in way-of-working.

## Graceful Degradation

- If [way-of-working.md](../../../.pair/adoption/tech/way-of-working.md) doesn't exist, create it with PM section as initial content. Warn: "Created way-of-working.md — this appears to be a new project."
- If implementation guide not found for selected tool, HALT with contribution instructions (Step 2.4).
- If `/pair-capability-record-decision` is not installed, warn and skip ADL recording: "Decision not recorded — /pair-capability-record-decision not installed. Please manually document the PM tool choice."

## Notes

- Supported tools with implementation guides: **GitHub Projects**, **Filesystem**.
- The [selection framework](../../../.pair/knowledge/guidelines/collaboration/project-management-tool/README.md) provides decision matrix and cost-benefit analysis for tool selection.
- This skill modifies: `adoption/tech/way-of-working.md` and creates an ADL entry via `/pair-capability-record-decision`.
- PM tool configuration is a project-level decision — it applies to all team members and workflows.
