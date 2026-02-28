---
name: agents
description: How to spawn, manage, message, and command subagents via gobby-agents.
version: "2.0.0"
category: Orchestration
alwaysApply: false
injectionFormat: full
---

# Agents Skill: Working with Subagents

This skill covers the full `gobby-agents` MCP server — spawning agents, managing their lifecycle, configuring agent definitions, and inter-agent communication.

## Server Clarity

All agent tools live on **`gobby-agents`**, NOT `gobby-sessions`.
- `gobby-sessions` — session lifecycle (CRUD, handoffs)
- `gobby-agents` — spawning, lifecycle, definitions, messaging, commands

Session IDs can be passed as `#N`, `N`, full UUID, or UUID prefix throughout.

---

## 1. Spawning & Lifecycle

### spawn_agent(prompt, ...)

Spawn a subagent to execute a task.

| Parameter | Required | Description |
|-----------|----------|-------------|
| `prompt` | yes | The task/instruction for the agent |
| `agent` | no | Named agent definition to use (e.g. `"default"`, `"claude-cli-sonnet"`) |
| `task_id` | no | Gobby task ID to associate with this run |
| `isolation` | no | `"none"` (current dir), `"worktree"` (git worktree), `"clone"` (shallow clone) |
| `branch_name` | no | Branch name for worktree/clone isolation |
| `base_branch` | no | Base branch to create from |
| `clone_id` / `worktree_id` | no | Reuse an existing clone or worktree |
| `workflow` | no | Workflow to attach to the agent session |
| `mode` | no | Workflow mode (e.g. `"plan"`, `"execute"`) |
| `initial_step` | no | Starting step for the workflow |
| `provider` | no | LLM provider override |
| `model` | no | Model override |
| `timeout` | no | Timeout in seconds |
| `max_turns` | no | Maximum agentic turns |
| `sandbox` | no | Enable sandboxing |
| `sandbox_mode` | no | Sandbox mode string |
| `sandbox_allow_network` | no | Allow network in sandbox |
| `sandbox_extra_paths` | no | Extra paths to mount in sandbox |
| `parent_session_id` | no | Override parent session (defaults to current) |
| `project_path` | no | Override project path |

### can_spawn_agent(parent_session_id)

Check if spawning is allowed from a session. Returns whether the depth limit or other restrictions would block a spawn.

### evaluate_spawn(...)

Dry-run a spawn — validates agent definition, workflow, isolation, and runtime without actually executing. Accepts the same key parameters as `spawn_agent` (`agent`, `workflow`, `task_id`, `isolation`, `mode`, `provider`, `branch_name`, `base_branch`, `parent_session_id`, `project_path`).

### wait_for_agent(run_id, timeout?, poll_interval?)

Block until an agent reaches a terminal status (`success`, `error`, `timeout`, `cancelled`) or the wait timeout expires.

| Parameter | Required | Description |
|-----------|----------|-------------|
| `run_id` | yes | The agent run ID to wait on |
| `timeout` | no | Max wait time in seconds |
| `poll_interval` | no | Polling interval in seconds |

### stop_agent(run_id)

Mark a running agent as cancelled in the DB. Does **not** kill the process — use `kill_agent` for that.

### kill_agent(run_id?, session_id?, signal?, force?, debug?)

Kill a running agent's process and close its terminal.

| Parameter | Required | Description |
|-----------|----------|-------------|
| `run_id` | no | Parent kills child by run ID |
| `session_id` | no | Self-termination by session ID |
| `signal` | no | Signal to send (default: SIGTERM) |
| `force` | no | Force kill if graceful fails |
| `debug` | no | Enable debug output |

Use `run_id` when a parent wants to kill a child. Use `session_id` for self-termination.

---

## 2. Querying Agents

### list_agents(parent_session_id, status?, limit?)

List agent runs for a session (DB records). Filter by `status` and limit results.

### list_running_agents(parent_session_id?, mode?)

List agents with active in-memory processes. Unlike `list_agents`, this only shows agents that are currently running, not historical records.

### get_running_agent(run_id)

Get the in-memory process state for a specific running agent.

### get_agent_result(run_id)

Get the result of a completed agent run (output, status, timing).

### running_agent_stats()

Get aggregate statistics about all running agents (counts, resource usage).

### unregister_agent(run_id)

Remove an agent from the in-memory running registry. Internal use — prefer `stop_agent` or `kill_agent`.

---

## 3. Agent Definitions

Agent definitions are named configurations stored in the DB that control how agents are spawned (CLI type, model, allowed tools, workflows, etc.).

### list_agent_definitions(enabled?, project_id?)

List definitions, optionally filtered by enabled status or project scope.

### get_agent_definition(name)

Get the full definition body for a named agent.

### create_agent_definition(name, definition)

Create a new agent definition. The `definition` object is validated against `AgentDefinitionBody` before insertion.

### toggle_agent_definition(name, enabled)

Enable or disable a definition by name.

### delete_agent_definition(name, force?)

Soft-delete a definition. Template-sourced agents are protected unless `force=True`.

### update_agent_rules(name, add?, remove?)

Add or remove rules from an agent definition's `workflows.rules` list.

### update_agent_variables(name, set_vars?, remove?)

Set or remove variables from an agent definition's `workflows.variables` dict.

---

## 4. Messaging (P2P)

Peer-to-peer messaging between any two sessions in the same project.

### send_message(from_session, to_session, content, priority?)

Send a message. When sending to a parent session, auto-writes to `agent_runs.result`.

| Parameter | Required | Description |
|-----------|----------|-------------|
| `from_session` | yes | Sender session ID |
| `to_session` | yes | Recipient session ID |
| `content` | yes | Message content |
| `priority` | no | Priority level |

### deliver_pending_messages(session_id)

Fetch all undelivered messages for a session and mark them as delivered. Call this often as an inbox check — especially before starting long-running work.

---

## 5. Commands (Parent-to-Child)

Structured task delegation from an ancestor session to a descendant. Only one active command per target at a time.

### send_command(from_session, to_session, command_text, ...)

Issue a command to a descendant agent. Validates ancestry. Rejects if the target already has an active command.

| Parameter | Required | Description |
|-----------|----------|-------------|
| `from_session` | yes | Commanding (ancestor) session |
| `to_session` | yes | Target (descendant) session |
| `command_text` | yes | The command instruction |
| `allowed_tools` | no | Restrict which tools the agent can use |
| `allowed_mcp_tools` | no | Restrict which MCP tools the agent can use |
| `exit_condition` | no | Condition for the agent to auto-complete |

### activate_command(session_id, command_id)

Accept a pending command — marks it as running and sets session variables (`command_id`, `command_text`, `allowed_tools`, `exit_condition`).

### complete_command(session_id, command_id, result)

Complete a command — marks it done, clears session variables, and sends the result back to the commanding session.

---

## Dos and Don'ts

- **DO** use `#N` shorthand for session IDs (e.g. `#5`).
- **DO** call `deliver_pending_messages` before starting long-running work.
- **DO** use `evaluate_spawn` to dry-run before spawning complex agents.
- **DO** use `can_spawn_agent` to check depth limits before spawning.
- **DO** use `kill_agent` (not bash tools or `/quit`) to terminate agents.
- **DON'T** use `unregister_agent` directly — prefer `stop_agent` or `kill_agent`.
- **DON'T** poll constantly — use hooks and event-driven patterns where possible.
- **DON'T** send commands to non-descendant sessions — ancestry is validated.
