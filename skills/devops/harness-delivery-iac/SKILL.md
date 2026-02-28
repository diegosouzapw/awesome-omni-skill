---
name: harness-delivery-iac
description: Use this skill for Harness CI/CD, IaCM, templates, connectors, integrations, pipelines, pull requests, APIs, feature flags, internal developer portal, and governance workflows.
---

# Harness Delivery and IaC Skill

## Workflow
1. Identify module: CI, CD, IaCM, IDP, FF, or Platform API.
2. Load only the matching reference file(s) below.
3. Return concrete pipeline/spec snippets and step-by-step setup.

## References
- `references/core-categories.md`
- `references/iacm-and-workspaces.md`
- `references/ci-cd-and-prs.md`
- `references/platform-api-and-integrations.md`
# Harness Delivery IaC Skill

This skill provides repeatable workflows for defining Harness CI/CD and governance automation as code.

## Standard Task 1: Create a CI pipeline with build + test stages

### Required inputs
- Harness account/org/project identifiers.
- Service/repository name and connector reference.
- Build command(s) and test command(s).
- Target branch and trigger conditions.

### Command/snippet workflow
1. Create pipeline YAML:
   ```bash
   mkdir -p .harness
   cat > .harness/ci-pipeline.yaml <<'YAML'
   pipeline:
     name: app-ci
     identifier: app_ci
     projectIdentifier: <project_id>
     orgIdentifier: <org_id>
     tags: {}
     stages:
       - stage:
           name: Build
           identifier: Build
           type: CI
           spec:
             cloneCodebase: true
             execution:
               steps:
                 - step:
                     type: Run
                     name: Install
                     identifier: Install
                     spec:
                       shell: Sh
                       command: |
                         <install_command>
                 - step:
                     type: Run
                     name: Test
                     identifier: Test
                     spec:
                       shell: Sh
                       command: |
                         <test_command>
   YAML
   ```
2. Validate YAML locally:
   ```bash
   yq e '.pipeline.identifier' .harness/ci-pipeline.yaml
   ```
3. Commit and push pipeline definition:
   ```bash
   git add .harness/ci-pipeline.yaml
   git commit -m "Add Harness CI pipeline"
   git push
   ```

### Expected output format
- Versioned pipeline YAML in `.harness/`.
- Identifier extraction output from validation command.
- Commit with clear message and pipeline metadata.

### Validation checklist
- [ ] Pipeline YAML includes `pipeline`, `stages`, and executable CI steps.
- [ ] Project/org identifiers are populated.
- [ ] Test step command is non-empty.
- [ ] YAML parses successfully with `yq`.

## Standard Task 2: Add PR automation flow (CI trigger on pull requests)

### Required inputs
- Existing pipeline identifier.
- SCM connector reference.
- Source repository and target branch filters.
- Optional path-based include/exclude rules.

### Command/snippet workflow
1. Create trigger YAML:
   ```bash
   cat > .harness/ci-pr-trigger.yaml <<'YAML'
   trigger:
     name: app-ci-pr
     identifier: app_ci_pr
     enabled: true
     orgIdentifier: <org_id>
     projectIdentifier: <project_id>
     pipelineIdentifier: app_ci
     source:
       type: Webhook
       spec:
         type: Github
         spec:
           type: PullRequest
           spec:
             connectorRef: <github_connector>
             autoAbortPreviousExecutions: true
             payloadConditions:
               - key: targetBranch
                 operator: Regex
                 value: ^main$
   YAML
   ```
2. Validate trigger schema fields:
   ```bash
   yq e '.trigger.pipelineIdentifier' .harness/ci-pr-trigger.yaml
   ```
3. Commit trigger definition:
   ```bash
   git add .harness/ci-pr-trigger.yaml
   git commit -m "Add Harness PR trigger for CI"
   git push
   ```

### Expected output format
- `.harness/ci-pr-trigger.yaml` with webhook PR trigger config.
- Validation output containing pipeline identifier.
- Commit showing trigger addition.

### Validation checklist
- [ ] Trigger is `enabled: true`.
- [ ] `pipelineIdentifier` matches an existing pipeline.
- [ ] PR event type and branch filters are correctly configured.
- [ ] YAML validation command returns expected value.

## Standard Task 3: Provision Harness entities with Terraform (pipeline + trigger refs)

### Required inputs
- Harness provider credentials (`account_id`, platform API key/token).
- Org/project names.
- Paths to pipeline/trigger YAML files.
- Desired Terraform state backend configuration.

### Command/snippet workflow
1. Define Terraform provider and resources:
   ```bash
   mkdir -p infra/harness
   cat > infra/harness/main.tf <<'TF'
   terraform {
     required_providers {
       harness = {
         source  = "harness/harness"
         version = "~> 0.30"
       }
     }
   }

   provider "harness" {
     endpoint         = "https://app.harness.io/gateway"
     account_id       = var.harness_account_id
     platform_api_key = var.harness_platform_api_key
   }

   resource "harness_platform_pipeline" "app_ci" {
     identifier  = "app_ci"
     org_id      = var.org_id
     project_id  = var.project_id
     yaml        = file("${path.module}/../../.harness/ci-pipeline.yaml")
     name        = "app-ci"
   }

   resource "harness_platform_triggers" "app_ci_pr" {
     identifier   = "app_ci_pr"
     org_id       = var.org_id
     project_id   = var.project_id
     target_id    = harness_platform_pipeline.app_ci.identifier
     yaml         = file("${path.module}/../../.harness/ci-pr-trigger.yaml")
     name         = "app-ci-pr"
   }
   TF
   ```
2. Initialize and validate Terraform:
   ```bash
   cd infra/harness
   terraform init
   terraform validate
   terraform plan -out=tfplan
   ```
3. Apply in controlled environments:
   ```bash
   terraform apply tfplan
   ```

### Expected output format
- Terraform plan showing create/update actions for Harness resources.
- Successful apply log with resource IDs.
- Deployment summary:
  ```text
  Terraform validate: PASS
  Planned resources: harness_platform_pipeline.app_ci, harness_platform_triggers.app_ci_pr
  Apply: SUCCESS
  ```

### Validation checklist
- [ ] `terraform validate` succeeds.
- [ ] Plan includes expected Harness resources only.
- [ ] Trigger references created pipeline.
- [ ] Apply output returns resource identifiers without errors.
