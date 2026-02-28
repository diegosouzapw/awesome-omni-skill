---
name: terraform-infra
description: Terraform infrastructure operations with safety controls
version: 1.0.0
model: sonnet
invoked_by: both
user_invocable: true
tools: [Bash, Read, Glob]
best_practices:
  - Always run plan before apply
  - Review plan output carefully
  - Never force push to production
error_handling: graceful
streaming: supported
verified: true
lastVerifiedAt: 2026-02-22T00:00:00.000Z
---

# Terraform Infrastructure Skill

## Installation

The skill invokes the **Terraform** CLI. Install:

- **macOS**: `brew tap hashicorp/tap && brew install hashicorp/tap/terraform`
- **Windows**: `choco install terraform` or download from [HashiCorp](https://developer.hashicorp.com/terraform/install)
- **Linux (apt)**: Add HashiCorp repo then `sudo apt update && sudo apt install terraform` (see [HashiCorp install](https://developer.hashicorp.com/terraform/install))

Verify: `terraform --version`

## Cheat Sheet & Best Practices

**Workflow:** `terraform init` → `terraform fmt` → `terraform validate` → `terraform plan -out=tfplan` → review → `terraform apply tfplan`. Use `terraform show tfplan` to inspect.

**Hacks:** Always run `plan` before `apply`; never `apply` blind. Use remote state (e.g. S3 + lock) for team work. Prefer `-auto-approve` only in CI with reviewed plans. Use `terraform state list` and `terraform state show <resource>` to debug. Use service accounts / workload identity in pipelines; avoid static keys.

## Certifications & Training

**HashiCorp Terraform Associate (004):** IaC concepts, Terraform fundamentals, state, modules, Terraform Cloud. [Learning path](https://developer.hashicorp.com/terraform/tutorials/certification-003). **Skill data:** init → fmt → validate → plan -out → apply; remote state; no blind apply.

## Hooks & Workflows

**Suggested hooks:** Pre-apply: run `terraform plan -out=tfplan` and gate on review. CI: apply only after plan approval. Use with **devops** (primary).

**Workflows:** Use with **devops** (primary). Flow: init → plan → review → apply; use state commands for debugging. See `ci-cd-implementation-rule` for pipeline integration.

## Overview

Provides 90%+ context savings vs raw Terraform MCP server. Includes critical safety controls for infrastructure operations.

## Requirements

- Terraform CLI (v1.0+)
- Cloud provider credentials configured
- Working directory with .tf files

## Tools (Progressive Disclosure)

### Planning & Validation

| Tool     | Description             | Confirmation |
| -------- | ----------------------- | ------------ |
| plan     | Generate terraform plan | No           |
| validate | Validate configuration  | No           |
| fmt      | Format terraform files  | No           |

### State Operations

| Tool     | Description            | Confirmation |
| -------- | ---------------------- | ------------ |
| show     | Display current state  | No           |
| list     | List state resources   | No           |
| state-mv | Move resource in state | Yes          |

### Workspace Operations

| Tool             | Description      | Confirmation |
| ---------------- | ---------------- | ------------ |
| workspace-list   | List workspaces  | No           |
| workspace-select | Select workspace | No           |
| workspace-new    | Create workspace | Yes          |

### Execution (⚠️ Dangerous)

| Tool  | Description   | Confirmation |
| ----- | ------------- | ------------ |
| apply | Apply changes | **REQUIRED** |

### Blocked Operations

| Tool     | Status      |
| -------- | ----------- |
| destroy  | **BLOCKED** |
| state-rm | **BLOCKED** |

## Quick Reference

```bash
# Initialize
terraform init

# Plan changes
terraform plan -out=tfplan

# Validate
terraform validate

# Apply (requires -auto-approve for automation)
terraform apply tfplan
```

## Configuration

- Working directory: Must contain terraform files
- TF*VAR*\*: Variable values via environment
- TF_WORKSPACE: Active workspace

## Safety Controls

⚠️ **terraform apply ALWAYS requires confirmation**
⚠️ **terraform destroy is BLOCKED by default**
⚠️ **State modifications require confirmation**
⚠️ **Review plan output before apply**

## Agent Integration

- **devops** (primary): Infrastructure management
- **architect** (secondary): Infrastructure design
- **cloud-integrator** (secondary): Cloud provisioning

## Troubleshooting

| Issue        | Solution                      |
| ------------ | ----------------------------- |
| Init failed  | Check provider credentials    |
| State locked | Check for other operations    |
| Plan failed  | Review error output carefully |

## Iron Laws

1. **ALWAYS** run `terraform plan` and review the output before executing `terraform apply`
2. **NEVER** hardcode credentials or secrets in `.tf` files — use secret managers (AWS Secrets Manager, Azure Key Vault, HashiCorp Vault)
3. **ALWAYS** use remote state with encryption and state locking to prevent concurrent modifications
4. **NEVER** edit state files directly — use `terraform state` commands exclusively
5. **ALWAYS** pin provider and module versions for fully reproducible infrastructure deployments

## Anti-Patterns

| Anti-Pattern                          | Why It Fails                                 | Correct Approach                                      |
| ------------------------------------- | -------------------------------------------- | ----------------------------------------------------- |
| Hardcoded credentials in .tf files    | Secret exposure in VCS, compliance failure   | Use variables with secret manager backend             |
| No state locking                      | Concurrent applies corrupt state             | Enable backend locking (S3+DynamoDB, Azure Blob, GCS) |
| `terraform apply` without plan review | Unexpected resource deletion or recreation   | Always `plan` first, review diff, then `apply`        |
| Unversioned providers and modules     | Non-reproducible builds and breaking changes | Pin versions: `version = "~> 4.0"`                    |
| Untagged resources                    | Untrackable costs and compliance failure     | Tag all resources with env, owner, cost-center        |

## Memory Protocol (MANDATORY)

**Before starting:**
Read `.claude/context/memory/learnings.md`

**After completing:**

- New pattern -> `.claude/context/memory/learnings.md`
- Issue found -> `.claude/context/memory/issues.md`
- Decision made -> `.claude/context/memory/decisions.md`

> ASSUME INTERRUPTION: If it's not in memory, it didn't happen.
