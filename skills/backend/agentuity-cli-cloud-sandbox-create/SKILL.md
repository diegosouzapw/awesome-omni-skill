---
name: agentuity-cli-cloud-sandbox-create
description: Create an interactive sandbox for multiple executions. Requires authentication. Use for Agentuity cloud platform operations
version: "0.0.110"
license: Apache-2.0
allowed-tools: "Bash(agentuity:*)"
metadata:
  command: "agentuity cloud sandbox create"
  tags: "slow requires-auth"
---

# Cloud Sandbox Create

Create an interactive sandbox for multiple executions

## Prerequisites

- Authenticated with `agentuity auth login`
- Organization context required (`--org-id` or default org)

## Usage

```bash
agentuity cloud sandbox create [options]
```

## Options

| Option | Type | Required | Default | Description |
|--------|------|----------|---------|-------------|
| `--memory` | string | Yes | - | Memory limit (e.g., "500Mi", "1Gi") |
| `--cpu` | string | Yes | - | CPU limit in millicores (e.g., "500m", "1000m") |
| `--disk` | string | Yes | - | Disk limit (e.g., "500Mi", "1Gi") |
| `--network` | boolean | Yes | - | Enable outbound network access |
| `--idleTimeout` | string | Yes | - | Idle timeout before sandbox is reaped (e.g., "10m", "1h") |
| `--env` | array | Yes | - | Environment variables (KEY=VALUE) |
| `--file` | array | Yes | - | Files to create in sandbox (sandbox-path:local-path) |
| `--snapshot` | string | Yes | - | Snapshot ID or tag to restore from |
| `--dependency` | array | Yes | - | Apt packages to install (can be specified multiple times) |

## Examples

Create a sandbox with default settings:

```bash
bunx @agentuity/cli cloud sandbox create
```

Create a sandbox with resource limits:

```bash
bunx @agentuity/cli cloud sandbox create --memory 1Gi --cpu 1000m
```

Create a sandbox with network and custom timeout:

```bash
bunx @agentuity/cli cloud sandbox create --network --idle-timeout 30m
```

Create a sandbox with a specific environment variable:

```bash
bunx @agentuity/cli cloud sandbox create --env KEY=VAL
```

## Output

Returns JSON object:

```json
{
  "sandboxId": "string",
  "status": "string",
  "stdoutStreamUrl": "string",
  "stderrStreamUrl": "string"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `sandboxId` | string | Unique sandbox identifier |
| `status` | string | Current sandbox status |
| `stdoutStreamUrl` | string | URL to the stdout output stream |
| `stderrStreamUrl` | string | URL to the stderr output stream |
