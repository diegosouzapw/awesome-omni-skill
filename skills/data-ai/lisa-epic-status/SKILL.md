---
name: lisa-epic-status
description: Show detailed status for an epic
---

# Epic Status

**Epic name:** Provided via arguments (already parsed by command)

Use the `get_epic_status` tool to get detailed status.

Display with checkbox format:

```
Epic: initial-setup
Status: Spec phase complete

Phase Progress:
  ✓ Spec - complete (.lisa/epics/initial-setup/spec.md)
  ○ Research - not started
  ○ Plan - not started
  ○ Execute - not started

Spec Summary:
[First 3-4 lines of spec.md]

Next Actions:
  • /lisa-continue initial-setup - Continue to research phase
  • /lisa-yolo initial-setup - Auto-execute all remaining phases
```

**If epic doesn't exist:**
```
Epic '<name>' not found.

Create it with: /lisa-create-epic <name>
```
