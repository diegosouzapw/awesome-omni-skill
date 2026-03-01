---
name: praxis
description: Interact with the Praxis C2 framework for orchestrating AI coding agents. Use when the user wants to manage nodes, agents, sessions, run operations or chains, or search intercepted traffic on the Praxis network.
---

Praxis is a Command & Control (C2) framework for orchestrating AI coding agents. It provides a unified interface to manage, monitor, and interact with AI agents (like Claude Code, Cursor, Windsurf, etc.) running on remote machines.

## First Step

Before using any commands, discover the full capabilities by running:

```bash
praxis_cli --fullhelp
```

This outputs comprehensive documentation for all commands and subcommands.

## Key Concepts

- **Node**: A machine running the Praxis node agent
- **Agent**: An AI coding agent (e.g., Claude Code) discovered on a node
- **Session**: An active connection to an agent for sending prompts
- **Operation**: A pre-configured prompt/workflow for common tasks
- **Chain**: A sequence of operations executed as a workflow

## Requirements

The Praxis service must be running and accessible via RabbitMQ. The default connection is `amqp://praxis:praxis@localhost:5672`.

To specify a different RabbitMQ URL:
```bash
praxis_cli --rabbitmq amqp://user:pass@host:5672 node list
```

Or set the environment variable:
```bash
export PRAXIS_RABBITMQ_URL=amqp://user:pass@host:5672
```

## Output Formats

Use `--output json` for machine-readable output suitable for scripting and parsing.
