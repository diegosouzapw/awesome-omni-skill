---
name: things3
description: This skill should be used when the user asks to "create a task in Things", "add a todo to Things 3", "manage Things projects", "schedule a task", "complete a todo", "delete a task from Things", "move task to Today", "add to Things inbox", "create a project in Things", "manage Things areas", or mentions Things 3 task management.
version: 0.1.0
---

# Things 3 Task Management

This skill enables interaction with Things 3 on macOS using AppleScript. It provides scripts for creating, reading, updating, and deleting tasks, projects, areas, and tags.

## Prerequisites

- Things 3 must be installed on macOS
- AppleScript permissions must be granted to the terminal

## Core Concepts

### Built-in Lists
Things 3 has these built-in lists:
- **Inbox** - Default landing spot for new tasks
- **Today** - Tasks scheduled for today
- **Upcoming** - Tasks with future dates
- **Anytime** - Tasks available anytime (no specific date)
- **Someday** - Tasks for later consideration
- **Logbook** - Completed tasks
- **Trash** - Deleted items

### Task Properties
To-dos can have:
- `name` - Task title (required)
- `notes` - Additional details
- `due date` - When the task is due
- `when date` - When to start/show the task
- `tag names` - Comma-separated tags
- `status` - open, completed, or canceled

### Organization Hierarchy
- **Areas** - High-level life categories (Work, Personal, Health)
- **Projects** - Collections of related tasks with a goal
- **To-dos** - Individual actionable items

## Available Scripts

All scripts are in `scripts/` directory and output JSON for easy parsing.

### Task Operations

#### create-todo.sh
Create a new to-do in Things 3.
```bash
~/.claude/skills/things3/scripts/create-todo.sh "Task name" ["notes"] ["due:YYYY-MM-DD"] ["tags:tag1,tag2"] ["list:Today|Inbox|..."] ["project:ProjectName"]
```

#### list-todos.sh
List to-dos from a specific list or project.
```bash
~/.claude/skills/things3/scripts/list-todos.sh [list_name]
# list_name: Today, Inbox, Upcoming, Anytime, Someday, or project/area name
```

#### update-todo.sh
Update properties of an existing to-do.
```bash
~/.claude/skills/things3/scripts/update-todo.sh "task name or id" ["name:New Name"] ["notes:New notes"] ["tags:tag1,tag2"]
```

#### complete-todo.sh
Mark a to-do as complete.
```bash
~/.claude/skills/things3/scripts/complete-todo.sh "task name or id"
```

#### delete-todo.sh
Move a to-do to trash.
```bash
~/.claude/skills/things3/scripts/delete-todo.sh "task name or id"
```

#### move-todo.sh
Move a to-do to a different list, project, or area.
```bash
~/.claude/skills/things3/scripts/move-todo.sh "task name or id" "destination"
# destination: Today, Inbox, Someday, project name, or area name
```

#### schedule-todo.sh
Schedule a to-do for a specific date.
```bash
~/.claude/skills/things3/scripts/schedule-todo.sh "task name or id" "YYYY-MM-DD"
```

### Project & Area Operations

#### create-project.sh
Create a new project.
```bash
~/.claude/skills/things3/scripts/create-project.sh "Project Name" ["notes"] ["area:AreaName"] ["tags:tag1,tag2"]
```

#### create-area.sh
Create a new area.
```bash
~/.claude/skills/things3/scripts/create-area.sh "Area Name"
```

### Tag Operations

#### create-tag.sh
Create a new tag.
```bash
~/.claude/skills/things3/scripts/create-tag.sh "Tag Name" ["parent:ParentTagName"]
```

### UI Operations

#### show-todo.sh
Show and select a to-do in Things 3 UI.
```bash
~/.claude/skills/things3/scripts/show-todo.sh "task name or id"
```

## Common Workflows

### Add a task to Today
```bash
~/.claude/skills/things3/scripts/create-todo.sh "Review quarterly report" "Check sales figures" "list:Today"
```

### Create a project with tasks
```bash
# Create the project
~/.claude/skills/things3/scripts/create-project.sh "Website Redesign" "Complete by Q2"

# Add tasks to it
~/.claude/skills/things3/scripts/create-todo.sh "Design mockups" "" "" "" "project:Website Redesign"
~/.claude/skills/things3/scripts/create-todo.sh "Review with stakeholders" "" "" "" "project:Website Redesign"
```

### Schedule a task for next week
```bash
~/.claude/skills/things3/scripts/create-todo.sh "Prepare presentation" "" "due:2025-01-03"
```

### Complete and clean up
```bash
~/.claude/skills/things3/scripts/complete-todo.sh "Review quarterly report"
```

## Error Handling

All scripts return JSON with:
- `success`: boolean indicating operation result
- `message`: description of what happened
- `data`: relevant data (task info, list of tasks, etc.)

Example success:
```json
{"success": true, "message": "Task created", "data": {"name": "My Task", "id": "ABC123"}}
```

Example error:
```json
{"success": false, "message": "Task not found", "data": null}
```

## Additional Resources

### Reference Files
- **`references/applescript-api.md`** - Complete Things 3 AppleScript API documentation

### Things 3 Documentation
- [Things AppleScript Guide](https://culturedcode.com/things/support/articles/2803572/)
- [AppleScript Commands Reference](https://culturedcode.com/things/support/articles/4562654/)
