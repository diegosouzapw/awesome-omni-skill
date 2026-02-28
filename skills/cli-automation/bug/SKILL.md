---
name: bug
description: "Quickly create a bug task."
category: core
triggers: bug, file bug, report bug
metadata:
  gobby:
    audience: all
    format_overrides:
      autonomous: full
---

# /gobby:bug - Create Bug Task

Create a bug/defect task with the provided title and optional description.

## Usage

```text
/gobby:bug <title>
/gobby:bug <title> - <description>
```

## Examples

```text
/gobby:bug Fix login timeout
/gobby:bug Database connection drops - Users report intermittent connection failures after 5 minutes of inactivity
```

## Tool Schema Reminder

**First time calling a tool this session?** Use `get_tool_schema(server_name, tool_name)` before `call_tool` to get correct parameters. Schemas are cached per session—no need to refetch.

## Action

Create a bug task with the following parameters:

- `title`: The bug title from user input
- `task_type`: "bug"
- `priority`: 1 (high - bugs are important)

Parse the user input:

- Try splitting on common delimiter patterns: ` - `, ` -- `, `—` (em dash), `–` (en dash)
- Use regex pattern `\s*[-–—]+\s*` to split once (first match only)
- If a delimiter is found, assign the left part as title and right part as description
- If no delimiter is found, use entire input as title and set description to null

```python
call_tool(
    server_name="gobby-tasks",
    tool_name="create_task",
    arguments={
        "title": "<parsed title>",
        "description": "<parsed description or null>",  # Use null, not empty string
        "task_type": "bug",
        "priority": 1,
        "session_id": "<session_id>"  # Required - from session context
    }
)
```

## Error Handling

### Missing session_id

The `session_id` parameter is required. If session context is unavailable:

1. **User-facing error**: Display "Session context unavailable — please sign in or retry"
2. **Behavior**: Fail gracefully with a clear error message; do not create partial tasks
3. **Fallback options**:
   - Prompt the user to provide a session ID manually
   - Implement retry logic with exponential backoff if the session service is temporarily unavailable
   - Check `get_current_session` from `gobby-sessions` to obtain or refresh the session

```python
# Example error handling
session_id = get_session_context()
if not session_id:
    return error("Session context unavailable — please sign in or retry")
```

After creating, confirm with the task reference (e.g., "Created bug #123: Fix login timeout").
