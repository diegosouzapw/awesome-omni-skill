---
name: week-skill
description: Manage WEEEK tasks, boards, and users. View boards, create/move tasks, assign users, and track time via CLI. Use when you need to interact with the WEEEK project management system.
---

# WEEEK Management Skill

This skill provides a comprehensive CLI wrapper for the WEEEK API, enabling efficient task and board management directly from the terminal.

## Usage

The main entry point is `weeek-cli.js`.

### 0. 🔍 Discovery (Start here)
**Goal:** Find Project or Board IDs to work with.
- **List Projects:** `node weeek-cli.js projects`
- **List Boards:** `node weeek-cli.js boards [projectId]`
- **Usage:**
  ```bash
  node weeek-cli.js projects
  node weeek-cli.js boards 1
  ```

### 1. 📋 Board & Context
**Goal:** Understand the current state of tasks on a specific board.
- **Command:** `board <name>`
- **Description:** Searches for a board by name (fuzzy match) and displays all columns with their tasks.
- **Usage:**
  ```bash
  node weeek-cli.js board "Release 2.0"
  ```

### 2. 🆕 Task Creation
**Goal:** Create new tasks quickly without knowing internal IDs.
- **Command:** `create <title>`
- **Options:**
  - `--board <name>`: Target board name (required for column).
  - `--col <name>`: Target column name.
  - `--assignee <name>`: User name to assign (fuzzy match).
  - `--prio <0-3>`: Priority (0=Low, 1=Med, 2=High, 3=Hold).
  - `--desc <text>`: Task description.
  - `--subtasks "ST1, ST2"`: List of subtasks to create.
- **Usage:**
  ```bash
  node weeek-cli.js create "Fix login bug" --board "App" --col "Backlog" --assignee "Ivan" --prio 2 --desc "Bug in oauth flow" --subtasks "Check logs, Fix bug, Test"
  ```

### 3. 👁️ Task Inspection
**Goal:** Get detailed info about a specific task.
- **Command:** `show <taskId>`
- **Description:** Shows clean description (no HTML), status, priority, subtasks, and assignees.
- **Usage:**
  ```bash
  node weeek-cli.js show 893
  ```

### 4. 🚚 Task Movement
**Goal:** Move a task through the workflow (e.g., to "Done").
- **Command:** `move <taskId> <columnName>`
- **Description:** Moves a task to a different column on the same board.
- **Usage:**
  ```bash
  node weeek-cli.js move 893 "Done"
  ```

### 5. 👥 Team Management
**Goal:** Find user IDs or assign people to tasks.
- **List Users:** `node weeek-cli.js users`
- **Assign User:** `node weeek-cli.js assign <taskId> <userName>`

### 6. ⏱️ Time Tracking
**Goal:** Log work time.
- **Start:** `node weeek-cli.js timer start <taskId>`
- **Stop:** `node weeek-cli.js timer stop <taskId>`

## 🔧 Raw API Access (Advanced)
If a specific action is not covered by the high-level commands above, you can explore and execute raw API requests directly using the underlying generic tools.

### 1. Explore the API
- **List Categories:** See all available API tags.
  ```bash
  node weeek-cli.js tags
  ```
- **List Endpoints:** See all paths for a specific tag (e.g., `Task`, `Project`, `User`).
  ```bash
  node weeek-cli.js endpoints Task
  ```
- **Describe Endpoint:** Get details on parameters and request body for a specific route.
  ```bash
  node weeek-cli.js describe GET tm/tasks
  ```

### 2. Execute Raw Requests
- **Command:** `run <Method> <Path>`
- **Options:**
  - `-p, --params "key=value"`: URL query parameters.
  - `-d, --data '{"json": "body"}'`: Request body.
- **Examples:**
  ```bash
  # Get all projects
  node weeek-cli.js run GET tm/projects

  # Filter tasks via API params directly
  node weeek-cli.js run GET tm/tasks -p "projectId=1&day=2023-10-01"
  ```

## Best Practices
1.  **Use `board` first**: Before moving or creating tasks, run `board` to see the exact column names.
2.  **Fuzzy Search**: You don't need exact names for boards or users. "Release" will find "Trionix App - release 2.0".
3.  **Check IDs**: When using `show`, `move`, or `assign`, use the numeric Task ID (e.g., `893`) visible in the `board` output.

## Troubleshooting
- If a board is not found, try a shorter keyword.
- Ensure `.env` contains a valid `WEEEK_API_KEY`.
