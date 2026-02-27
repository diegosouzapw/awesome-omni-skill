---
name: search-copilot-chats
description: Search across archived Copilot chat sessions (VS Code + CLI) using the copilot-session-tools CLI. Use when the user says "search my chats", "find in chat history", "what did we discuss about X", "look up past sessions", "scan chats", or references a session-state path or session GUID. Also covers exporting sessions as markdown or HTML and launching the web viewer.
---

# Search Copilot Chats

Search, browse, and export archived GitHub Copilot chat sessions from VS Code (Stable & Insiders) and the Copilot CLI. Uses the **copilot-session-tools** CLI (`copilot-session-tools`) backed by a local SQLite database with FTS5 full-text search.

## Installation

Install this skill for any supported AI coding agent:

```bash
# Claude Code (default)
npx skills-installer install @Arithmomaniac/copilot-session-tools/search-copilot-chats

# Other clients (Cursor, VS Code, Codex, etc.)
npx skills-installer install @Arithmomaniac/copilot-session-tools/search-copilot-chats --client cursor
```

For project-local installation (current directory only):

```bash
npx skills-installer install @Arithmomaniac/copilot-session-tools/search-copilot-chats --local
```

See [claude-plugins.dev](https://claude-plugins.dev) for browsing available skills.

## When to use

- User asks to search past Copilot conversations ("what did we discuss about X?")
- User pastes a `session-state` path or session GUID and wants context from it
- User pastes a web viewer URL like `http://127.0.0.1:5000/session/{uuid}`
- User wants to find prior decisions, code patterns, or troubleshooting steps from past sessions
- User asks to export a session as markdown, HTML, or JSON
- User wants to refresh the chat archive (scan for new sessions)
- User says "search my chats", "look up past sessions", "find in chat history"

## Prerequisites

The `copilot-session-tools` CLI must be installed and available on PATH. It uses the Copilot CLI's built-in session store at `~/.copilot/session-store.db` (requires Copilot CLI v0.0.412+).

### Step 0: Ensure CLI is installed

Check if the CLI is available:

```powershell
copilot-session-tools --help
```

If the command is not found, install it using whichever package manager is available (in preference order):

```bash
# Option 1: uv (recommended — fast, isolated)
uv tool install copilot-session-tools[all]

# Option 2: pipx (isolated)
pipx install copilot-session-tools[all]

# Option 3: pip (global)
pip install copilot-session-tools[all]
```

The `[all]` extra includes both the CLI and web viewer dependencies.

## Favor CLI over Python

**Always prefer CLI execution** (`copilot-session-tools` commands) over writing Python scripts or importing the library directly. The CLI handles all common workflows. Only drop into Python/SQL when:
- You need a custom join or aggregation the CLI doesn't support
- You need to process results programmatically (e.g., pipe into another tool)
- The CLI's output format doesn't fit the task

For direct SQL queries, use `sqlite3` on the database file rather than writing Python:

```powershell
sqlite3 "$HOME/.copilot/session-store.db" "SELECT cs.session_id, cs.workspace_name, cs.created_at FROM cst_sessions cs ORDER BY cs.created_at DESC LIMIT 10"
```

## Instructions

### Search strategy (critical)

The FTS5 search uses **AND semantics**: every keyword in the query MUST appear in the **same message**. More keywords = stricter matching = fewer results. This is the #1 cause of empty search results.

**Start broad, narrow later.** Always begin with 1–2 core keywords. Only add more if you get too many results.

**Iterative approach.** If no results are returned, **remove** keywords (don't add more). Try synonyms. Drop field filters. The goal is to widen the net, not narrow it further.

**Field filters don't reduce keyword strictness.** A filter like `workspace:ZTS` narrows the *session pool*, but all FTS keywords still must match within a single message.

**Keyword budget.** Aim for **2–3 keywords max**. Use exact phrases for multi-word concepts (e.g., `"resource graph"` counts as 1 match unit).

### Search dos and don'ts

**❌ DON'T — these all returned zero results in real usage:**

| Failed query | Problem |
|---|---|
| `customer tenant query build SPN` | 5 loose keywords — ALL must appear in one message |
| `workspace:ZTS "resource graph" integration test permission` | Workspace filter + exact phrase + 3 keywords = too strict |
| `"build service" "resource graph" reader` | Two exact phrases + a keyword — very unlikely all in one message |
| `spec driven proposal specs design tasks` | 6 keywords, too many AND conditions |
| `workspace:ZTS Docker SSL ARG mock` | 4 keywords after workspace filter |
| `"customer managed identity" ARG build` | Exact phrase + 2 keywords, still too strict |

**✅ DO — these succeeded:**

| Successful query | Why it works |
|---|---|
| `ARG build pipeline` | 3 focused keywords |
| `"resource graph" auth` | 1 exact phrase + 1 keyword |
| `workspace:ZTS ARG RBAC` | Workspace filter + only 2 keywords |
| `workspace:ZTS CollectVmData` | Workspace + 1 specific identifier |
| `docker build` | 2 keywords, broad |

**Iterative search workflow:**

```
Step 1: copilot-session-tools search "resource graph" --full --limit 20
  → Too many results? Add ONE keyword:
Step 2: copilot-session-tools search "resource graph" auth --full --limit 20  
  → Still too many? Add a workspace filter:
Step 3: copilot-session-tools search 'workspace:ZTS "resource graph" auth' --full --limit 10
  → No results? Back up and try different keyword:
Step 4: copilot-session-tools search 'workspace:ZTS "resource graph" permission' --full --limit 10
```

### Step 1: Ensure the archive is fresh

Before searching, scan for new sessions (incremental — only imports changed files):

```powershell
copilot-session-tools scan --verbose
```

This scans:
- VS Code Stable: `%APPDATA%\Code\User\workspaceStorage\`
- VS Code Insiders: `%APPDATA%\Code - Insiders\User\workspaceStorage\`
- Copilot CLI: `~\.copilot\session-state\` and `~\.copilot\history-session-state\`

For a full reimport (slow, use only if data seems stale):

```powershell
copilot-session-tools scan --full --verbose
```

### Step 2: Search for content

#### Basic search

```powershell
copilot-session-tools search "your search terms" --full --limit 30
```

Key flags:
| Flag | Purpose |
|------|---------|
| `--full` / `-F` | Show full content (default truncates to 200 chars) |
| `--limit N` / `-l N` | Max results (default 20) |
| `--sort date` / `-s date` | Sort by date instead of relevance |
| `--role user` / `-r user` | Only user messages |
| `--role assistant` / `-r assistant` | Only assistant messages |
| `--tools-only` | Search only tool invocations |
| `--files-only` | Search only file changes |
| `--no-tools` | Exclude tool invocations |
| `--no-files` | Exclude file changes |

#### Inline field filters

Field filters go directly in the query string (extracted before FTS processing):

```powershell
# Filter by workspace
copilot-session-tools search "workspace:my-project error handling" --full

# Filter by repository
copilot-session-tools search "repo:github.com/owner/repo authentication" --full

# Filter by session title
copilot-session-tools search "title:build-loop pipeline" --full

# Filter by date range
copilot-session-tools search "start_date:2025-01-01 end_date:2025-06-30 docker" --full

# Filter by source edition
copilot-session-tools search "edition:cli ralph loop" --full

# Filter by role (inline)
copilot-session-tools search "role:user pipeline trigger" --full

# Combine multiple filters
copilot-session-tools search "role:user workspace:zts start_date:2025-06-01 docker build" --full
```

#### Exact phrase matching

Wrap phrases in double quotes (escape for shell):

```powershell
copilot-session-tools search '"exact phrase match"' --full
```

#### FTS5 search tips

- `word1 word2` → AND (both must appear)
- `"exact phrase"` → phrase match
- Field filters (`role:`, `workspace:`, `repo:`, `title:`, `edition:`, `start_date:`, `end_date:`) are extracted before FTS processing
- Sort by `relevance` (FTS5 rank, default) or `date` (session created_at DESC)
- Tool invocations and file changes use LIKE matching, not FTS

> **⚠️ Hyphenated terms crash FTS5.** A bare query like `copilot-session-tools` is parsed as `copilot MINUS chat MINUS archive`, producing `no such column: chat`. Always wrap hyphenated terms in double quotes:
> ```powershell
> copilot-session-tools search '"copilot-session-tools"' --full
> ```

### Step 3: Browse a specific session

When you have a session ID (from search results, a pasted path, or a URL):

#### Export as markdown (default — use this)

```powershell
copilot-session-tools export-markdown --session-id <session-guid> --output-dir . --verbose
```

Optional detail flags:
- `--include-diffs` — Include file change diffs
- `--include-tool-inputs` — Include tool invocation inputs
- `--include-thinking` — Include reasoning/thinking blocks

#### Export as self-contained HTML (opt-in — for user presentation only)

```powershell
copilot-session-tools export-html --session-id <session-guid> --output-dir . --verbose
```

Only use when the user explicitly wants a visual artifact to browse, share, or attach somewhere. The HTML is self-contained (embedded CSS/JS, dark mode, collapsible tool details) but adds no information beyond what markdown already provides. For LLM reading, always use markdown.

Alternatively, if the web viewer is running, download markdown via its API:
```
http://127.0.0.1:5000/api/markdown/<session-guid>?download=true&include_diffs=true&include_tool_inputs=true&include_thinking=true
```

#### Get session statistics

```powershell
copilot-session-tools stats
```

#### Launch web viewer (for visual browsing)

```powershell
copilot-session-tools web
```

Additional flags: `--host` (default `127.0.0.1`), `--port` (default `5000`), `--title`, `--debug`, `--db` (default `~/.copilot/session-store.db`).

Then browse: `http://127.0.0.1:5000/session/<session-guid>`

The web viewer supports:
- Full-text search with highlight
- Workspace and edition filtering
- **Download Markdown** and **Copy URL** buttons per session
- Dark mode
- Syntax highlighting for code blocks
- Sticky header, multi-select filters, first-prompt display

### Step 4: Extract session references from user input

Users reference past sessions in several ways. Recognize and extract the session GUID:

| User provides | Extract |
|---------------|---------|
| `C:\Users\avilevin\.copilot\session-state\{uuid}` | The `{uuid}` portion |
| `~\.copilot\session-state\{uuid}` | The `{uuid}` portion |
| `http://127.0.0.1:5000/session/{uuid}` | The `{uuid}` portion |
| A bare GUID like `67894303-8571-...` | Use directly as session ID |
| A short prefix like `67894303` | Search: `copilot-session-tools search "67894303"` |

Then use the session ID with `export-markdown --session-id` or `raw-json` to retrieve context.

### Step 5: Get archive statistics

```powershell
copilot-session-tools stats
```

Shows: session count, message count, workspace breakdown, edition breakdown, top repositories.

### Step 6: Direct SQL for advanced queries

When the CLI search doesn't support your query shape, use SQLite directly:

```powershell
# Find sessions by workspace name
sqlite3 "$HOME/.copilot/session-store.db" `
  "SELECT session_id, workspace_name, created_at FROM cst_sessions WHERE workspace_name LIKE '%zts%' ORDER BY created_at DESC LIMIT 20"

# Count messages per session (find long conversations)
sqlite3 "$HOME/.copilot/session-store.db" `
  "SELECT s.session_id, s.workspace_name, COUNT(m.id) as msg_count FROM cst_sessions s JOIN cst_messages m ON s.session_id = m.session_id GROUP BY s.session_id ORDER BY msg_count DESC LIMIT 20"

# Find sessions with tool invocations of a specific tool
sqlite3 "$HOME/.copilot/session-store.db" `
  "SELECT DISTINCT s.session_id, s.workspace_name, s.created_at FROM cst_sessions s JOIN cst_messages m ON s.session_id = m.session_id JOIN cst_tool_invocations ti ON m.id = ti.message_id WHERE ti.name LIKE '%build%' ORDER BY s.created_at DESC LIMIT 20"

# Find file changes by path pattern
sqlite3 "$HOME/.copilot/session-store.db" `
  "SELECT DISTINCT s.session_id, s.workspace_name, fc.path FROM cst_sessions s JOIN cst_messages m ON s.session_id = m.session_id JOIN cst_file_changes fc ON m.id = fc.message_id WHERE fc.path LIKE '%Dockerfile%' ORDER BY s.created_at DESC LIMIT 20"

# Find command runs
sqlite3 "$HOME/.copilot/session-store.db" `
  "SELECT s.session_id, cr.command, cr.status FROM cst_sessions s JOIN cst_messages m ON s.session_id = m.session_id JOIN cst_command_runs cr ON m.id = cr.message_id WHERE cr.command LIKE '%az pipelines%' ORDER BY cr.timestamp DESC LIMIT 20"
```

## Database schema (quick reference)

The tool extends `~/.copilot/session-store.db` with `cst_*` enrichment tables. Built-in tables are never modified.

**Built-in tables** (managed by Copilot CLI, read-only):
| Table | Key columns | Notes |
|-------|-------------|-------|
| `sessions` | id, cwd, repository, branch, summary, created_at | CLI sessions only |
| `turns` | session_id, turn_index, user_message, assistant_response | Basic conversation data |

**Enrichment tables** (managed by this tool, prefixed `cst_`):
| Table | Key columns | Notes |
|-------|-------------|-------|
| `cst_sessions` | session_id, workspace_name, vscode_edition, custom_title, parser_version, source_format | All enriched sessions |
| `cst_messages` | session_id, message_index, role, content, timestamp | Parsed messages with content blocks |
| `cst_messages_fts` | content | FTS5 virtual table, synced via triggers |
| `cst_tool_invocations` | message_id, name, input, result, status | Tool calls within messages |
| `cst_file_changes` | message_id, path, diff, content | File edits made by assistant |
| `cst_command_runs` | message_id, command, result, status, output | Shell commands executed |

## Common search patterns

These are real patterns observed from actual usage:

| Goal | Command |
|------|---------|
| Find past discussions about a topic | `copilot-session-tools search "docker build" --full` |
| Find what the user asked (not assistant) | `copilot-session-tools search "role:user pipeline" --full` |
| Find tool usage patterns | `copilot-session-tools search "az pipelines" --tools-only --full` |
| Find file edits to a specific file | `copilot-session-tools search "Dockerfile" --files-only --full` |
| Find sessions in a specific repo | `copilot-session-tools search "repo:visualstudio.com/One/_git/ZTS" --full` |
| Find recent CLI sessions | `copilot-session-tools search "edition:cli" --sort date --full` |
| Replay a prior workflow | Export the session markdown, then follow the same steps |
| Find sessions with errors | `copilot-session-tools search "error failed exception" --role assistant --full` |

## Workflow: "Continue where we left off"

When a user pastes a session path or URL and wants to resume work:

1. Extract the session GUID (see Step 4)
2. Export as markdown: `copilot-session-tools export-markdown --session-id <guid> -o .`
   Or as HTML: `copilot-session-tools export-html --session-id <guid> -o .`
3. Read the exported file to understand context
4. Summarize what was done and what remains
5. Continue the work in the current session

## Workflow: "Find how we solved X before"

When a user wants to find a prior solution:

1. Search: `copilot-session-tools search "<topic keywords>" --full --limit 30`
2. Identify the most relevant session from results
3. Export that session: `copilot-session-tools export-markdown --session-id <guid> -o .`
4. Extract the relevant technique/solution
5. Apply it to the current situation

## Known issues

- **Hyphenated terms crash FTS5**: Queries like `copilot-session-tools` fail because `-` is an FTS5 NOT operator. Always quote hyphenated terms: `'"copilot-session-tools"'`.
- **Rich console crashes on pipe**: The CLI uses Rich for console output, which can crash on Unicode box-drawing characters when piped. Workaround: don't pipe output; read it directly.
- **Missing sessions**: If a session doesn't appear after scanning, check that the source file exists and wasn't cleaned up by VS Code. Use `--full` scan to force reimport.
- **Requires Copilot CLI v0.0.412+**: The session store database (`~/.copilot/session-store.db`) is created by the Copilot CLI's Chronicle feature, introduced in v0.0.412.

## Related skills

- **ado-build-ralph-loop**: Uses this tool to find past build iteration patterns
- **diagnose-ado-build-failures**: Pairs with chat archive to find prior diagnosis approaches
- **search-microsoft-repos-for-patterns**: Broader search across engineering systems (not just local chats)
