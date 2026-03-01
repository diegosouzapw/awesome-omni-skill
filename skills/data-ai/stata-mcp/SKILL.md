---
name: stata-mcp
description: Run or debug Stata workflows through the local io.github.tmonk/mcp-stata server. Use when users mention Stata commands, .do files, r()/e() results, dataset inspection, Stata graph exports, or data browsing with sorting/filtering.
---

# Stata MCP Skill

## Instructions
1. Ensure the `stata` MCP server is registered (see project README for config) and request it if not already active.
2. When the user asks for Stata work:
   - Use `run_command` for ad-hoc syntax (`trace=True` for call stacks, `raw=True` for plain output).
   - Use `load_data` before analyses that require datasets.
   - Use `get_data`, `describe`, `codebook`, or `get_variable_list` to inspect data.
   - Use `run_do_file` for provided `.do` scripts.
   - Use `export_graph`/`export_graphs_all` for visualization requests.
   - Use `get_help` when the user wants Stata documentation.
   - Use `get_stored_results` to return `r()`/`e()` scalars/macros after commands for validation.
   - Use `read_log` to tail or retrieve output from long-running commands.
   - Use `get_ui_channel` to obtain a localhost HTTP endpoint for high-volume data browsing.
3. Surface `rc`/`stderr` info back to the user, referencing `r()`/`e()` codes.
4. If Stata isn't auto-discovered, remind the user to set `STATA_PATH` (examples in README).

## Tool quick reference

### Command Execution
- `run_command(code, echo=True, as_json=True, trace=False, raw=False, max_output_lines=None)`: Run Stata syntax.
  - `code`: The Stata command(s) to execute.
  - `echo`: Include the command itself in output (default: True).
  - `as_json`: Return JSON envelope with rc/stdout/stderr/error (default: True).
  - `trace`: Enable `set trace on` for deeper error diagnostics (default: False).
  - `raw`: Return plain stdout/error message instead of JSON (default: False).
  - `max_output_lines`: Truncate output to this many lines (default: None for no truncation).
  - Note: Always writes output to a temporary log file and emits a `notifications/logMessage` with `{"event":"log_path","path":"..."}` so the client can tail it locally.

- `run_do_file(path, echo=True, as_json=True, trace=False, raw=False, max_output_lines=None)`: Execute .do files.
  - `path`: Path to the .do file.
  - `echo`: Include commands in output (default: True).
  - `as_json`: Return JSON envelope (default: True).
  - `trace`: Enable trace mode for debugging (default: False).
  - `raw`: Return plain output instead of JSON (default: False).
  - `max_output_lines`: Truncate output to this many lines (default: None).
  - Note: Always writes output to a temporary log file and emits incremental `notifications/progress` when the client provides a progress token/callback.

- `read_log(path, offset=0, max_bytes=65536)`: Read a slice of a previously-provided log file.
  - `path`: Path to the log file (from `notifications/logMessage`).
  - `offset`: Byte offset to start reading from (default: 0).
  - `max_bytes`: Maximum bytes to read (default: 65536).
  - Returns JSON: `path`, `offset`, `next_offset`, `data`.

### Data Loading & Inspection
- `load_data(source, clear=True, as_json=True, raw=False, max_output_lines=None)`: Load data using sysuse/webuse/use heuristics.
  - `source`: Dataset name, URL, or file path (e.g., "auto", "webuse nlsw88", "/path/to/file.dta").
  - `clear`: Append `, clear` to replace existing data (default: True).
  - `as_json`: Return JSON envelope (default: True).
  - `raw`: Return plain output (default: False).
  - `max_output_lines`: Truncate output to this many lines (default: None).
  - Note: After loading, use UI channel for advanced filtering/sorting at scale.

- `get_data(start=0, count=50)`: Retrieve a slice of the active dataset as JSON.
  - `start`: Zero-based index of first observation (default: 0).
  - `count`: Number of observations to retrieve (default: 50, max: 500).
  - Note: For advanced sorting/filtering at scale, use the UI channel endpoints (see `get_ui_channel()`).

- `describe()`: Return variable descriptions, storage types, and labels.

- `get_variable_list()`: Return JSON list of all variables with names, labels, and types.

- `codebook(variable, as_json=True, trace=False, raw=False, max_output_lines=None)`: Return codebook/summary for a specific variable.
  - `variable`: Variable name to describe.
  - `as_json`: Return JSON envelope (default: True).
  - `trace`: Enable trace mode (default: False).
  - `raw`: Return plain output (default: False).
  - `max_output_lines`: Truncate output to this many lines (default: None).

### Graph Management
- `list_graphs()`: List all graphs in Stata's memory with active graph marked.
  - Note: Graphs are automatically cached during command execution for instant exports.

- `export_graph(graph_name=None, format="pdf")`: Export a stored graph to file.
  - `graph_name`: Name of graph to export (from `list_graphs`); if None, exports active graph.
  - `format`: Output format—"pdf" (default) or "png". Use "png" to view plots directly.

- `export_graphs_all()`: Export all graphs in memory. Returns file paths.

### Help & Results
- `get_help(topic, plain_text=False)`: Return Stata help text.
  - `topic`: Command or help topic (e.g., "regress", "graph").
  - `plain_text`: Return plain text instead of Markdown (default: False).

- `get_stored_results()`: Return current `r()` and `e()` results as JSON after a command.

### Session Management
- `create_session(session_id)`: Manually create a new Stata session.
- `list_sessions()`: List all active sessions and their status (running, idle, etc.).
- `stop_session(session_id)`: Terminate and clean up a specific session.
- `break_session(session_id="default")`: Interrupt the currently executing command in a session.
  - Use this tool when a command is taking too long or you want to stop a long-running loop without losing data already in memory.
  - Follow-up with `read_log` to see where execution stopped.

### UI Data Browser
- `get_ui_channel()`: Return a short-lived localhost HTTP endpoint + bearer token for the UI-only data browser.
  - Returns JSON with `baseUrl`, `token`, `expiresAt`, and `capabilities`.
  - Intended for VS Code extension UI to browse data at high volume (paging, filtering, sorting) without sending large payloads over MCP.
  - Loopback only (binds to `127.0.0.1`), requires bearer auth.
  - **Key endpoints** (all require `Authorization: Bearer <token>` header):
    - `GET /v1/dataset`: Dataset identity and state
    - `GET /v1/vars`: Variable metadata
    - `POST /v1/page`: Page data with optional sorting (`sortBy` parameter)
    - `POST /v1/arrow`: Binary Arrow IPC stream
    - `POST /v1/views`: Create filtered view
    - `POST /v1/views/:viewId/page`: Page within filtered view (supports sorting)
    - `POST /v1/views/:viewId/arrow`: Arrow stream from filtered view
    - `DELETE /v1/views/:viewId`: Delete view
    - `POST /v1/filters/validate`: Validate filter expression
  - **Sorting**: Use `sortBy` array in page requests (e.g., `["price"]` for ascending, `["-price"]` for descending, `["foreign", "-price"]` for multi-level)
  - **Filtering**: Filter expressions use Python boolean operators (`==`, `!=`, `<`, `>`, `and`, `or`); Stata-style `&`/`|` also accepted
  - **Server limits**: maxLimit=500, maxVars=32767, maxChars=500, maxRequestBytes=1000000, maxArrowLimit=1000000
  - **Dataset tracking**: `datasetId` used for cache invalidation; changing dataset invalidates view handles

## Cancellation
- Clients may cancel an in-flight request by sending the MCP notification `notifications/cancelled` with `params.requestId` set to the original tool call ID.
- Pass a `_meta.progressToken` when invoking the tool if you want progress updates (optional).
- Cancellation is best-effort and depends on Stata surfacing `BreakError`.

## Error Reporting
- All tools executing Stata commands support JSON envelopes (`as_json=true`) containing:
  - `rc`: Return code from r()/c(rc)
  - `stdout`: Standard output
  - `stderr`: Standard error (captures "red text")
  - `message`: Error message
  - `line`: Line number (when Stata reports it)
  - `command`: The command that was executed
  - `log_path`: Path to log file for streaming (when applicable)
  - `snippet`: Excerpt of error output
- Stata-specific error codes (`r(XXX)`) are parsed and preserved
- Use `trace=true` to enable `set trace on` for detailed program-defined error diagnostics
- Set `MCP_STATA_LOGLEVEL` environment variable (e.g., `DEBUG`, `INFO`) to control server logging

## MCP Resources
The server exposes these resources for MCP clients:
- `stata://data/summary` → `summarize`
- `stata://data/metadata` → `describe`
- `stata://graphs/list` → graph list
- `stata://variables/list` → variable list
- `stata://results/stored` → stored r()/e() results

## Graph review workflow
1. Call `list_graphs()` to see available plots and identify the active graph.
2. Use `export_graphs_all()` to fetch file paths for every graph; view them directly in the client.
3. For a single plot, call `export_graph(graph_name="GraphName", format="png")` to get a viewable file.
4. Compare the rendered PNGs to the user spec (titles, axes labels, legends, colors, filters); state whether the graph matches and what to change.

## Examples

### Run a regression
```
# Load sample data and run regression
load_data("auto")
run_command("regress price mpg")
get_stored_results()  # Retrieve coefficients and statistics
```

### Export a histogram
```
# Create and export a graph
run_command("histogram price")
list_graphs()  # Confirm graph exists
export_graph(graph_name="Graph", format="png")  # Export for viewing
```

### Debug a do-file
```
run_do_file("/path/to/analysis.do", trace=True)
```

### Inspect data structure
```
load_data("nlsw88", clear=True)
describe()
get_variable_list()
codebook("wage")
get_data(start=0, count=10)
```

### Read log output from long-running command
```
# After run_command emits a log_path notification
read_log("/tmp/stata_log_abc123.log", offset=0)
# Continue reading with next_offset for incremental output
read_log("/tmp/stata_log_abc123.log", offset=4096)
```

### Advanced data browsing with sorting and filtering
```
# Get UI channel for high-volume data operations
get_ui_channel()  # Returns baseUrl, token, expiresAt

# Example UI channel usage (requires HTTP client):
# POST {baseUrl}/v1/page with Authorization: Bearer {token}
# Body: {"datasetId":"...","offset":0,"limit":50,"vars":["price","mpg"],"sortBy":["-price"]}

# Create filtered view for price < 5000
# POST {baseUrl}/v1/views
# Body: {"datasetId":"...","frame":"default","filterExpr":"price < 5000"}

# Page through filtered view with sorting
# POST {baseUrl}/v1/views/{viewId}/page
# Body: {"offset":0,"limit":50,"vars":["price","mpg"],"sortBy":["-price"]}
```