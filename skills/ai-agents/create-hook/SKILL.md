---
name: create-hook
description: Creates or adds Cursor agent lifecycle hooks. Use when the user asks to create a hook, add a hook, set up a lifecycle hook, or automate something on agent stop, afterFileEdit, afterAgentResponse, or other Cursor hook events. Hooks live in .cursor/hooks/ and are registered in .cursor/hooks.json.
---

# Create Hook

**Type:** Utility skill (procedural, no agentic lead or sub-agents).

Adds a Cursor lifecycle hook: a script that runs when a specific agent event fires. Hooks are project-level only (`.cursor/hooks/` and `.cursor/hooks.json`).

## 1. Choose the lifecycle event

Register the hook under one of these keys in `hooks.json`:

| Event                                           | When it runs                          |
| ----------------------------------------------- | ------------------------------------- |
| `stop`                                          | When the agent stops (end of turn)    |
| `afterFileEdit`                                 | After the agent edits a file          |
| `afterAgentResponse`                            | After the agent sends a response      |
| `beforeSubmitPrompt`                            | Before the user's prompt is submitted |
| `beforeReadFile` / `afterShellExecution` / etc. | See Cursor docs for full list         |

## 2. Add the script

- **Path**: `.cursor/hooks/<name>.js` (or `.sh` if shell). Use a clear name (e.g. `memory-reminder.js`, `on-stop.js`).
- **Runtime**: Node.js is typical; command in `hooks.json` is run with Cursor's working directory set so that `node hooks/<name>.js` resolves (run from `.cursor`).
- **Input**: The hook receives a single JSON object on **stdin**. Read it like this:

```javascript
let input = "";
process.stdin.setEncoding("utf8");
process.stdin.on("data", (chunk) => {
  input += chunk;
});
process.stdin.on("end", () => {
  const payload = JSON.parse(input);
  const roots = payload.workspace_roots || [process.cwd()];
  const root = path.resolve(roots[0]); // repo root
  // ... use root for paths like path.join(root, '.cursor', 'next-step.md')
});
```

- **Payload**: Often includes `workspace_roots` (array of repo roots). Use the first root as the project root for paths. Event-specific payloads may include `file_path`, `prompt`, etc.
- **Output**: Hook can write files (e.g. `.cursor/next-step.md`), log to stderr, or exit; avoid writing to stdout if the protocol expects JSON there.
- **Shebang**: Start Node scripts with `#!/usr/bin/env node`.

## 3. Register in hooks.json

- **Path**: `.cursor/hooks.json`
- **Format**:

```json
{
  "version": 1,
  "hooks": {
    "stop": [
      { "command": "node hooks/on-stop.js" },
      { "command": "node hooks/my-new-hook.js" }
    ]
  }
}
```

- Append to the array for the chosen event; do not remove existing hooks unless the user asks to replace them.

## Checklist

- [ ] Script created under `.cursor/hooks/<name>.js` (or `.sh`)
- [ ] Script reads JSON from stdin and uses `workspace_roots` for repo root when needed
- [ ] Entry added to `.cursor/hooks.json` under the correct event key
- [ ] Command is `node hooks/<name>.js` (or `./hooks/<name>.sh`); path is relative to `.cursor`

## Reference

- Existing hooks in this repo: `.cursor/hooks/on-stop.js`, `.cursor/hooks/after-file-edit.js`, `.cursor/hooks/memory-reminder.js` — use them as patterns for stdin handling and `path.join(root, ...)`.
- Project memory (`.cursor/memory.md`) notes that new hooks should be added under `.cursor/hooks/` and registered in `.cursor/hooks.json`.
