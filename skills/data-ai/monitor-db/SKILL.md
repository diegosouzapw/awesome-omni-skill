---
name: monitor-db
description: "Workflow for monitor-db"
metadata: {"clawdbot":{"emoji":"🤖","always":false}}
---

# monitor-db

This skill executes the **monitor-db** workflow from .agent/workflows/monitor-db.md.

## Usage

```
"Run monitor-db"
```

Or via Discord slash command:
```
/skill monitor-db
```

## Implementation

This skill reads and executes the workflow file:

```bash
cat .agent/workflows/monitor-db.md
```

Then follows the steps defined in that file.
