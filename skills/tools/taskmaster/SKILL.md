---
name: taskmaster
description: "Use when any agent must operate Taskmaster with an MCP-first approach and automatic CLI fallback for init, PRD parsing, task lifecycle, dependencies, and status workflows."
license: MIT
compatibility: OpenCode
metadata:
  version: "1.2.0"
  owner: agent-skills
  references:
    - https://skills.sh/sfc-gh-dflippo/snowflake-dbt-demo/task-master-install
    - https://skills.sh/sfc-gh-dflippo/snowflake-dbt-demo/task-master
    - https://skills.sh/anombyte93/prd-taskmaster/prd-taskmaster
    - https://github.com/eyaltoledano/claude-task-master
    - https://raw.githubusercontent.com/eyaltoledano/claude-task-master/main/docs/command-reference.md
---

# Taskmaster MCP-First Operations

## When to Use

Use this skill when an agent needs to execute any Taskmaster workflow through Taskmaster MCP first, with Taskmaster CLI fallback only when MCP is unavailable or fails.

MCP scope for this skill:

- Allowed MCP integration: `mcp_task-master-ai` only.
- Do not use any other MCP tool while executing this skill.

Typical triggers:

- Initialize Taskmaster in a repository.
- Parse a PRD and generate tasks.
- Expand tasks into subtasks.
- Create, update, move, prioritize, or delete tasks.
- Change task status (`pending`, `in-progress`, `done`, `cancelled`, and other status values supported by the installed CLI version).
- Add, remove, or validate dependencies.
- Work with tags or contexts.
- Query progress, next task, or task details.

Do not use this skill for source-code implementation or git workflows.

## Inputs

Inputs:

- Repository absolute path.
- Requested action intent (for example: `init`, `parse-prd`, `expand`, `set-status`, `add-dependency`, `next-task`).
- Action parameters (task IDs, PRD path, status values, dependency IDs, tag/context names, and flags).
- Safety flags when needed:
  - `allow_install` (boolean)
  - `confirmed` (boolean for destructive actions)

If any required input is missing, stop and return a structured input error.

## Procedure

Procedure:

1. Resolve the CLI executable (test one by one and only test the next if first fails) in this order:
   - `task-master`
   - `taskmaster`
   - `npx -y task-master-ai`
   Try to run the first one and if it fails, try the next one, but if one of them succeeds, continue with that one.
2. Route the intent using the command catalog below.
3. If the exact command is not available in the installed version, use the closest official alias shown in `<tm> --help`.
4. For destructive operations (delete, clear, overwrite, force replace), require `confirmed=true`.
5. Execute only Taskmaster CLI commands, do not edit tasks.json manually or with any other tool.
6. Run post-action verification with one read command (`list`, `show`, or equivalent) on the same channel used for execution (CLI).
7. Return a structured execution report with exact commands/methods, outcomes

### Command Catalog (CLI Reference)

Use these commands as the default reference. Always confirm with `--help` before execution.

Project bootstrap and setup:

- `task-master init`
- `task-master init --rules cursor,windsurf,vscode`
- `task-master migrate` (when present in installed version)

PRD and planning:

- `task-master parse-prd <prd-file.txt>`
- `task-master parse-prd <prd-file.txt> --num-tasks=5`
- `task-master parse-prd <prd-file.txt> --num-tasks=0`

Task inspection:

- `task-master list`
- `task-master list --status=<status>`
- `task-master list --with-subtasks` // Shows all tasks with their subtasks
- `task-master list --status=<status> --with-subtasks`
- `task-master next`
- `task-master show <id>` // Shows the task defailt, with the subtasks
- `task-master show --id=<id>`
- `task-master show 1,3,5`
- `task-master show 1.2`

Task creation and updates:

- `task-master add-task --prompt="<description>"`
- `task-master add-task --prompt="<description>" --research`
- `task-master add-task --prompt="<description>" --dependencies=1,2,3`
- `task-master add-task --prompt="<description>" --priority=high`
- `task-master update --from=<id> --prompt="<prompt>"`
- `task-master update --from=<id> --prompt="<prompt>" --research`
- `task-master update-task --id=<id> --prompt="<prompt>"`
- `task-master update-task --id=<id> --prompt="<prompt>" --research`
- `task-master update-subtask --id=<parentId.subtaskId> --prompt="<prompt>"`
- `task-master update-subtask --id=<parentId.subtaskId> --prompt="<prompt>" --research`

Task lifecycle and status:

- `task-master set-status --id=<id> --status=<status>`
- `task-master set-status --id=1,2,3 --status=<status>`
- `task-master set-status --id=1.1,1.2 --status=<status>`

Task expansion and decomposition:

- `task-master expand --id=<id> --num=<number>`
- `task-master expand --id=<id> --num=0`
- `task-master expand --id=<id> --prompt="<context>"`
- `task-master expand --id=<id> --research`
- `task-master expand --all`
- `task-master expand --all --force`
- `task-master expand --all --research`
- `task-master clear-subtasks --id=<id>`
- `task-master clear-subtasks --id=1,2,3`
- `task-master clear-subtasks --all`

Complexity analysis:

- `task-master analyze-complexity`
- `task-master analyze-complexity --output=my-report.json`
- `task-master analyze-complexity --model=<model>`
- `task-master analyze-complexity --threshold=6`
- `task-master analyze-complexity --file=custom-tasks.json`
- `task-master analyze-complexity --research`
- `task-master complexity-report`
- `task-master complexity-report --file=my-report.json`

Dependencies:

- `task-master add-dependency --id=<id> --depends-on=<id>`
- `task-master remove-dependency --id=<id> --depends-on=<id>`
- `task-master validate-dependencies`
- `task-master fix-dependencies`

Task moves and restructuring:

- `task-master move --from=<id> --to=<id>`
- `task-master move --from=5.2 --to=7.3`
- `task-master move --from=10,11,12 --to=16,17,18`
- `task-master move --from=5 --from-tag=backlog --to-tag=in-progress`
- `task-master move --from=5,6,7 --from-tag=backlog --to-tag=done --with-dependencies`
- `task-master move --from=5 --from-tag=backlog --to-tag=in-progress --ignore-dependencies`

Tag and context management:

- `task-master tags`
- `task-master tags --show-metadata`
- `task-master add-tag <tag-name>`
- `task-master add-tag <tag-name> --description="<description>"`
- `task-master add-tag --from-branch`
- `task-master add-tag <new-tag> --copy-from-current`
- `task-master add-tag <new-tag> --copy-from=<source-tag>`
- `task-master use-tag <tag-name>`
- `task-master rename-tag <old-name> <new-name>`
- `task-master copy-tag <source-tag> <target-tag>`
- `task-master copy-tag <source-tag> <target-tag> --description="<description>"`
- `task-master delete-tag <tag-name>`
- `task-master delete-tag <tag-name> --yes`

Rules management:

- `task-master rules add <profile1,profile2,...>`
- `task-master rules remove <profile1,profile2,...>`
- `task-master rules remove <profile1,profile2,...> --force`
- `task-master rules setup`

Model configuration:

- `task-master models`
- `task-master models --set-main=<model>`
- `task-master models --set-research=<model>`
- `task-master models --set-fallback=<model>`
- `task-master models --set-main=<model> --ollama`
- `task-master models --set-research=<model> --openrouter`
- `task-master models --set-main=<model> --codex-cli`
- `task-master models --set-fallback=<model> --codex-cli`
- `task-master models --setup`

Research:

- `task-master research "<query>"`
- `task-master research "<query>" --id=15,16`
- `task-master research "<query>" --files=src/a.js,src/b.js`
- `task-master research "<query>" --context="<context>" --tree`
- `task-master research "<query>" --detail=high`
- `task-master research "<query>" --file=custom-tasks.json`
- `task-master research "<query>" --tag=<tag-name>`
- `task-master research "<query>" --save-file`
- `task-master research "<query>" --save-to=15`
- `task-master research "<query>" --save-to=15.2`

Task file generation:

- `task-master generate`

### Intent-to-Command Routing Hints

Use this mapping when parent agents provide high-level intent:

- `initialize taskmaster` -> `init`
- `create tasks from PRD` -> `parse-prd`
- `expand task` -> `expand --id=<id>`
- `expand pending tasks` -> `expand --all`
- `mark task started/done/cancelled` -> `set-status`
- `change downstream tasks after scope change` -> `update --from=<id>`
- `update one task only` -> `update-task`
- `append implementation notes to a subtask` -> `update-subtask`
- `add/remove dependency` -> `add-dependency` or `remove-dependency`
- `fix dependency graph` -> `fix-dependencies`
- `switch work context` -> `use-tag`
- `view next executable task` -> `next`

## Outputs

Outputs:

- Markdown report with these sections:
  - `Preconditions`
  - `Command Resolution`
  - `Executed Commands`
  - `Result Data`
  - `Validation`
  - `Final Status`

`Final Status` must be one of:

- `success`
- `partial`
- `failed`

## Examples

Input:

- Repository: `/repo/app`
- Action: `set-status`
- Params: `task_id=12`, `status=in-progress`

Output:

- `Preconditions`: Taskmaster executable resolved to `task-master`; repository contains `.taskmaster/`.
- `Command Resolution`: intent `set-status` mapped to installed version command `<tm> set-status`.
- `Executed Commands`: `<tm> set-status --id=12 --status=in-progress`.
- `Result Data`: task `12` status updated.
- `Validation`: `<tm> show 12` confirms `in-progress`.
- `Final Status`: `success`.

Input:

- Repository: `/repo/app`
- Action: `parse-prd`
- Params: `input=.taskmaster/docs/prd.md`, `num_tasks=20`, `research=true`

Output:

- `Preconditions`: Taskmaster initialized.
- `Command Resolution`: intent `parse-prd` mapped to `<tm> parse-prd`.
- `Executed Commands`: `<tm> parse-prd .taskmaster/docs/prd.md --num-tasks=20 --research`.
- `Result Data`: 20 tasks created.
- `Validation`: `<tm> list` shows new tasks.
- `Final Status`: `success`.

## Notes and Edge Cases

Notes:

- Do not execute source-code changes, git writes, or unrelated shell commands in this skill.
- **NEVER** manipulate the tasks.json file directly or using any other tool except the task-master CLI. Use the provided commands to manage tasks.
