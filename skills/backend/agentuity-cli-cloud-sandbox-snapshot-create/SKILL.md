---
name: agentuity-cli-cloud-sandbox-snapshot-create
description: Create a snapshot from a sandbox. Requires authentication. Use for Agentuity cloud platform operations
version: "0.0.110"
license: Apache-2.0
allowed-tools: "Bash(agentuity:*)"
argument-hint: "<sandboxId>"
metadata:
  command: "agentuity cloud sandbox snapshot create"
  tags: "slow requires-auth"
---

# Cloud Sandbox Snapshot Create

Create a snapshot from a sandbox

## Prerequisites

- Authenticated with `agentuity auth login`
- Organization context required (`--org-id` or default org)

## Usage

```bash
agentuity cloud sandbox snapshot create <sandboxId> [options]
```

## Arguments

| Argument | Type | Required | Description |
|----------|------|----------|-------------|
| `<sandboxId>` | string | Yes | - |

## Options

| Option | Type | Required | Default | Description |
|--------|------|----------|---------|-------------|
| `--tag` | string | Yes | - | Tag for the snapshot |

## Examples

Create a snapshot from a sandbox:

```bash
bunx @agentuity/cli cloud sandbox snapshot create sbx_abc123
```

Create a tagged snapshot:

```bash
bunx @agentuity/cli cloud sandbox snapshot create sbx_abc123 --tag latest
```

## Output

Returns JSON object:

```json
{
  "snapshotId": "string",
  "sandboxId": "string",
  "tag": "unknown",
  "sizeBytes": "number",
  "fileCount": "number",
  "createdAt": "string"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `snapshotId` | string | Snapshot ID |
| `sandboxId` | string | Source sandbox ID |
| `tag` | unknown | Snapshot tag |
| `sizeBytes` | number | Snapshot size in bytes |
| `fileCount` | number | Number of files in snapshot |
| `createdAt` | string | Snapshot creation timestamp |
