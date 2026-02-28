---
name: lettactl
description: Manage Letta AI agent fleets with kubectl-style CLI
---

## Overview
kubectl-style CLI and SDK for deploying AI agents declaratively from YAML.

## Quick Start
```bash
lettactl apply -f fleet.yaml        # Deploy agents
lettactl get agents                 # List agents
lettactl send <agent> "Hello"       # Message agent
lettactl health                     # Check server
```

## Environment
```bash
export LETTA_BASE_URL=http://localhost:8283
export LETTA_API_KEY=<optional-for-cloud>
```

## Skills Index
- `fleet-deployment/` - Deploy agents from YAML configs
- `agent-management/` - Create, update, delete, list agents
- `message-operations/` - Send messages, view history
- `resource-management/` - Manage blocks, tools, folders, files
- `bulk-operations/` - Cleanup, delete-all, bulk messaging
- `observability/` - Health, context usage, async runs
- `release-cycle/` - Release workflow: issues, branches, tests, commits, PRs, versioning
