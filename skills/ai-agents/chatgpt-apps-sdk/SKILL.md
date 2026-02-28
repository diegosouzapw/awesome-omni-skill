---
name: chatgpt-apps-sdk
description: "Build ChatGPT apps using OpenAI's Apps SDK. This skill leverages OpenAI's Docs MCP server to fetch the latest documentation, ensuring guidance is always current. Use when creating a new ChatGPT app, building an MCP server for ChatGPT, designing widgets/UI for ChatGPT apps, preparing an app for submission, or any question about ChatGPT Apps SDK or Agentic Commerce."
---

# ChatGPT Apps SDK Builder

Build ChatGPT apps using live documentation from OpenAI's Docs MCP.
Note: the docs and steps here are intentionally minimal. If you are ever unsure,
always search the docs for additional code patterns and guidance before proceeding.

## Required: Connect to OpenAI Docs MCP

**Server URL:** `https://developers.openai.com/mcp`

Before ANY work, verify connection and test with: `search_openai_docs("Apps SDK quickstart")`

## Deterministic TODO List (always use, in this order)

1. Verify OpenAI Docs MCP connectivity with `search_openai_docs("Apps SDK quickstart")`. If this fails, stop the TODO list and notify the user.
2. Inspect Apps SDK skill templates and docs sources, then dynamically enumerate available docs and decide which additional docs are required based on the user's scope.
3. Scaffold the app using the skill template (generic, not domain-specific).
4. Implement app-specific tools and UI flow based on the user's requirements.
5. App submission checklist (per docs): generate `app-submission-checklist.md` with checkboxes; leave items unchecked if not verified and list them as open items.
6. Run checks/tests and review lint.

## Workflow



### 1. Verify OpenAI Docs MCP connectivity

Run `search_openai_docs("Apps SDK quickstart")`. If this fails, stop the workflow and notify the user.

### 2. Inspect templates and docs sources + dynamic doc discovery

Review `templates/` and confirm doc URLs in "Key Doc URLs" are current.

Then dynamically enumerate docs and decide additional required docs:
- Call `list_openai_docs()` (page if needed) to see all available docs.
- Map the user's scope to doc topics; use targeted `search_openai_docs()` to locate relevant docs.
- Fetch only what's required for the user's ask; avoid static long lists.

Examples of scope-to-doc triggers (not exhaustive):
- Commerce, payments, revenue, transactions → fetch Agentic Commerce docs.
- External systems/tools → fetch MCP server + tool annotations docs.
- Custom UI/widgets → fetch UI guidelines / UI kit docs.
- Release/submission → fetch submission guidelines.

Always fetch the minimum Docs MCP set:
- `fetch_openai_doc("https://developers.openai.com/apps-sdk/quickstart/")`
- `search_openai_docs("Apps SDK MCP server registerTool")`

### 3. Scaffold the app

Use the skill template in `templates/` as the base scaffold.

### 4. Implement app-specific tools and UI flow

**Always fetch current design guidelines first:**
- `fetch_openai_doc("https://developers.openai.com/apps-sdk/concepts/ux-principles/")`
- `fetch_openai_doc("https://developers.openai.com/apps-sdk/concepts/ui-guidelines/")`

Key UX principles (verify against live docs):
- App must provide value that requires ChatGPT (conversational leverage)
- UI should enhance conversation, not replace it
- Tools should be atomic and self-contained

Key UI guidelines (verify against live docs):
- Use system fonts only (SF Pro/Roboto) - no custom fonts
- Use system colors - no custom backgrounds
- Consider using official UI kit: https://openai.github.io/apps-sdk-ui/

### 5. Before Setting Tool Annotations

**Fetch current annotation requirements:**
- `search_openai_docs("Apps SDK tool annotations readOnlyHint openWorldHint destructiveHint")`
- `fetch_openai_doc("https://developers.openai.com/apps-sdk/build/mcp-server/")` - see "Tool annotations and elicitation"

Annotation rules (verify against live docs):
- `readOnlyHint: true` → tool only reads data, no elicitation needed
- `openWorldHint: true` → tool affects external systems, requires elicitation  
- `destructiveHint: true` → tool deletes/overwrites data
- Omitting hints defaults to worst case (open world destructive write)

### 6. App submission checklist

**Fetch current submission guidelines:**
- `fetch_openai_doc("https://developers.openai.com/apps-sdk/app-submission-guidelines/")`

Checklist (verify against live docs). **Output a markdown file named `app-submission-checklist.md`** in the workspace root that:
- Uses checkbox items.
- Only checks items you have verified from code or artifacts.
- Leaves unverified items unchecked and lists them under an **Open items** section.
- Notes the exact doc URLs used for verification.

Required checklist items to include (adapt as needed to latest docs):
- Tool annotations are correct and justified
- App name/description are accurate and non-misleading
- Screenshots reflect actual functionality and required dimensions
- Tool descriptions match behavior
- App does not duplicate native ChatGPT capabilities
- UI follows design guidelines

### 7. Run checks/tests and review lint

Run project checks/tests and review lints before finalizing.

## Templates

Minimal starting points in `templates/`. **Always query Docs MCP for current patterns before modifying.**

## Key Doc URLs

| Topic | URL to Fetch |
|-------|--------------|
| Quickstart | `https://developers.openai.com/apps-sdk/quickstart/` |
| UX Principles | `https://developers.openai.com/apps-sdk/concepts/ux-principles/` |
| UI Guidelines | `https://developers.openai.com/apps-sdk/concepts/ui-guidelines/` |
| MCP Server | `https://developers.openai.com/apps-sdk/build/mcp-server/` |
| Submission | `https://developers.openai.com/apps-sdk/app-submission-guidelines/` |
| App Name/Description + Screenshots | `https://developers.openai.com/apps-sdk/app-submission-guidelines/` |
| Reference | `https://developers.openai.com/apps-sdk/reference/` |
| Examples | `https://github.com/openai/openai-apps-sdk-examples` |
