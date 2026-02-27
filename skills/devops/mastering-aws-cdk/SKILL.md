---
name: mastering-aws-cdk
description: Guides AWS CDK v2 infrastructure-as-code development in TypeScript with patterns, troubleshooting, and deployment workflows. Use when creating or refactoring CDK stacks, debugging CloudFormation or CDK deploy errors, setting up CI/CD with GitHub Actions OIDC, or integrating AWS services (Lambda, API Gateway, ECS/Fargate, S3, DynamoDB, EventBridge, Aurora, MSK).
---

# Mastering AWS CDK v2 (TypeScript)

Focused guidance for building, deploying, and troubleshooting AWS CDK v2 infrastructure in TypeScript.

## Contents

- [Use This Skill When](#use-this-skill-when)
- [Trigger Terms](#trigger-terms)
- [Quick Start](#quick-start)
- [Workflow](#workflow)
- [Reference Map](#reference-map)
- [Guardrails](#guardrails)
- [Debugging Checklist](#debugging-checklist)
- [When Not to Use](#when-not-to-use)
- [Reference Files](#reference-files)

## Use This Skill When

- Building new CDK apps or stacks in TypeScript
- Refactoring or splitting stacks to manage limits
- Debugging synth/diff/deploy failures or CloudFormation rollbacks
- Importing existing resources into CDK management
- Driving stacks from JSON/YAML configuration files
- Setting up GitHub Actions OIDC deployments
- Implementing service patterns across AWS managed services
- Writing CDK tests and running security checks

## Trigger Terms

Use for queries mentioning: `cdk`, `cdk deploy`, `cdk diff`, `cdk synth`, `cdk import`, `cdk watch`, `cdk refactor`, `cdk bootstrap`, `cdk-nag`, `hotswap`, `CloudFormation`, `stack rollback`, `cdk.context.json`, `cdk.json`, `SSM Parameter Store`, `hnb659fds`, or `OIDC GitHub Actions`.

## Quick Start

1. Confirm target account, region, and environment (dev/stage/prod).
2. Run `cdk synth` then `cdk diff` to validate changes.
3. Deploy with `cdk deploy --require-approval=never` in CI.

## Workflow

### 1) Intake

Collect:
- account and region
- environment name and stage
- target services and integrations
- existing resources to import or avoid replacement

### 2) Stack Design

- Keep stacks under 500 resources (split or use nested stacks)
- Pass outputs via props or explicit exports
- Set removal policies for stateful resources (retain by default)

### 3) Implement

- Prefer L2 constructs; use L1 only for gaps
- Apply least-privilege IAM grants
- Keep resource names deterministic

### 4) Validate

- `cdk synth` to inspect the template
- `cdk diff` to review changes
- `cdk doctor` for environment issues

### 5) Deploy

- Ensure bootstrap completed for the account/region
- Review CloudFormation events on failure
- Use `--require-approval=never` only for CI

### 6) Observability

- Add log retention, alarms, and dashboards early
- Use X-Ray where distributed tracing matters
- See [observability.md](references/observability.md)

## Reference Map

| Task | Reference |
|------|-----------|
| Troubleshooting errors | [troubleshooting.md](references/troubleshooting.md) |
| CI/CD with GitHub Actions | [cicd-github.md](references/cicd-github.md) |
| Service-specific patterns | [services.md](references/services.md) |
| Observability setup | [observability.md](references/observability.md) |
| Architecture and operations | [architecture-ops.md](references/architecture-ops.md) |
| Testing and security | [testing-security.md](references/testing-security.md) |
| Latest features | [latest-features.md](references/latest-features.md) |

## Guardrails

- Do not modify CloudFormation-managed resources in the console
- Avoid dynamic values (Date.now, random) in resource definitions
- Use `env: { account, region }` for lookups (VPC/AZ/AMI)
- Use stable IDs when generating constructs from config data
- Use `cdk import` (adopt) for existing resources; use `fromXxx` only for read-only references
- Do not use hotswap in production pipelines

## Debugging Checklist

Copy and track progress:
```
Debugging Progress:
- [ ] Check CloudFormation events (Console -> Stack -> Events)
- [ ] Re-run with verbose output: `cdk deploy --progress events`
- [ ] Inspect template: `cdk synth > template.yaml`
- [ ] Run diff: `cdk diff`
- [ ] Check service logs (Lambda: CloudWatch, ECS: task events)
- [ ] Run `cdk doctor`
```

## When Not to Use

- Terraform/Pulumi or raw CloudFormation templates
- Manual console-driven resource management
- CDK in Python/Java/Go/C# (TypeScript only)

## Reference Files

- [references/troubleshooting.md](references/troubleshooting.md) -- Error messages and fixes
- [references/cicd-github.md](references/cicd-github.md) -- GitHub Actions OIDC setup
- [references/services.md](references/services.md) -- Lambda, ECS, MSK, DynamoDB, Aurora, S3, EventBridge patterns
- [references/observability.md](references/observability.md) -- CloudWatch, X-Ray, dashboards, alarms
- [references/architecture-ops.md](references/architecture-ops.md) -- Determinism, configuration-driven patterns, imports, drift, and operational workflows
- [references/testing-security.md](references/testing-security.md) -- CDK testing, cdk-nag, and compliance checks
- [references/latest-features.md](references/latest-features.md) -- New constructs, CLI capabilities, and recent patterns
