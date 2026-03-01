---
name: unified-cicd-platform
description: "This skill should be used when the user is choosing a CI/CD platform, migrating between CI/CD providers, consolidating build and deployment pipelines, designing pipeline architecture across application and infrastructure code, setting up drift detection, configuring OIDC authentication for pipelines, or discussing the operational cost of multiple CI/CD systems. Covers platform selection, the cost of multi-platform CI/CD, what 'everything on one platform' means, OIDC pipeline authentication, Jenkins migration, and scheduled pipeline jobs like drift detection."
version: 1.0.0
---

# One CI/CD Platform for Everything, or Pay the Tax Forever

Running CircleCI for application builds, GitLab CI for infrastructure pipelines, and GitHub Actions for deployments is an operational tax that compounds daily. Every platform has different YAML syntax, different secrets management, different caching mechanisms, different debugging workflows, and different monitoring dashboards. Engineers carry three mental models instead of one. During incidents -- when context-switching costs are highest -- crossing platform boundaries wastes critical minutes. A deployment failed. Which platform ran it? Where are the logs? How do you re-trigger it? The answer changes depending on which system was responsible.

This is not a theoretical concern. Teams that have lived through multi-platform CI/CD report the same pattern: initial adoption is easy (each tool does its job), but maintenance becomes a death spiral. Secret rotation requires updating three platforms. A new engineer must learn three systems. Pipeline debugging requires three browser tabs with three different log viewers. The consolidation from three providers to one is consistently described as one of the highest-impact operational improvements a team can make.

## The Cost of Multi-Platform CI/CD

| Dimension | Single Platform | Multiple Platforms |
|-----------|-----------------|-------------------|
| Syntax to learn | One YAML dialect | 2-3 incompatible dialects |
| Secrets management | One secrets store | Secrets duplicated across platforms, rotation requires N updates |
| Debugging | One log viewer, one interface | Context-switch between platforms to trace a deployment |
| Monitoring | One dashboard for all pipelines | Separate monitoring per platform, gaps between them |
| OIDC setup | One trust relationship per cloud account | One trust relationship per platform per account |
| Onboarding | "Here is how our CI/CD works" | "It depends on which repo and which kind of pipeline" |
| Incident response | One place to check | "Which platform ran the failing deployment?" |
| Runner management | One pool of runners | Separate runner infrastructure per platform |
| Cost visibility | One bill | Multiple vendor bills, hard to compare |
| Migration effort | Already consolidated | Every consolidation is a multi-week project |

### The Pattern That Creates This Mess

It usually starts innocently. The team uses GitHub for code, so someone sets up GitHub Actions for linting. An infrastructure engineer prefers GitLab CI's Terraform integration, so infrastructure pipelines go there. A data team keeps Jenkins around because their scheduled ETL jobs already work. Each decision is locally rational. The global result is chaos.

## Platform Selection: Follow Your Code Host

The simplest rule: **use the CI/CD platform native to your code hosting platform**. The integration is tightest, the context switching is lowest, and the team already has accounts.

| Code Host | CI/CD Platform | Why |
|-----------|---------------|-----|
| GitHub | GitHub Actions | Native integration, OIDC built-in, environment protection rules, PR-triggered workflows |
| GitLab | GitLab CI | Native integration, built-in container registry, merge request pipelines, environment management |
| Bitbucket | Bitbucket Pipelines | Native integration, deployment environments, PR-triggered pipelines |
| Self-hosted Git | Choose one: GitHub Actions (with self-hosted runners), GitLab CI, or Buildkite | Evaluate runner management, OIDC support, and team familiarity |

**If your code is on GitHub but your CI/CD is on CircleCI or Jenkins**, you are paying an integration tax for every webhook, every status check, and every deployment notification. Migrate to GitHub Actions. The effort is finite. The operational benefit is permanent.

**Self-hosted runners for specialized workloads**: If you need GPU runners, ARM builds, or other exotic compute, use self-hosted GitHub Actions runners rather than switching to a different CI/CD platform. The [terraform-aws-github-runner](https://github.com/github-aws-runners/terraform-aws-github-runner) module provides auto-scaling, ephemeral runners on AWS with full Terraform management.

## What "Everything" Means

One platform should handle all of these pipeline types. Not just application builds -- everything.

| Pipeline Type | Description | Example |
|--------------|-------------|---------|
| Application CI | Lint, test, build, push container image | Run tests on PR, build Docker image on merge |
| Infrastructure CI | Validate, lint, plan Terraform changes | `terraform plan` on PR, post plan as comment |
| Infrastructure CD | Apply Terraform changes to environments | `terraform apply` on tag push with approval gate |
| Application CD | Deploy application to environments | Update container service with new image tag |
| Scheduled jobs | Recurring operational tasks | Drift detection, certificate expiry checks, cost reports |
| Security scans | Vulnerability and compliance scanning | Container image scanning, dependency audits |

### Drift Detection: A Scheduled Pipeline Example

Drift detection compares actual cloud infrastructure against the Terraform state file. It runs on a schedule (daily or weekly) and alerts when resources have been modified outside of Terraform. This is a perfect example of a job that must live on the same platform as the rest of your pipelines.

```yaml
# .github/workflows/drift-detection.yml
name: Terraform Drift Detection
on:
  schedule:
    - cron: "0 6 * * 1-5"  # Weekdays at 6am UTC
  workflow_dispatch: {}      # Allow manual trigger

permissions:
  id-token: write
  contents: read
  issues: write

jobs:
  detect-drift:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        layer: [network, security, compute, databases]
    steps:
      - uses: actions/checkout@v4

      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::role/GithubActionsPlanRole
          aws-region: eu-west-1

      - uses: hashicorp/setup-terraform@v3

      - name: Detect Drift
        id: drift
        run: |
          cd infrastructure/prod/${{ matrix.layer }}
          terraform init
          terraform plan -detailed-exitcode -no-color 2>&1 | tee drift-output.txt
          # Exit code 2 = changes detected (drift)
          echo "exit_code=$?" >> $GITHUB_OUTPUT

      - name: Alert on Drift
        if: steps.drift.outputs.exit_code == '2'
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const drift = fs.readFileSync(
              `infrastructure/prod/${{ matrix.layer }}/drift-output.txt`, 'utf8'
            );
            github.rest.issues.create({
              owner: context.repo.owner,
              repo: context.repo.repo,
              title: `Drift detected: ${{ matrix.layer }} (prod)`,
              body: `Terraform plan detected changes in the \`${{ matrix.layer }}\` layer.\n\n\`\`\`\n${drift.substring(0, 60000)}\n\`\`\``,
              labels: ['drift', 'infrastructure']
            });
```

## OIDC Authentication for Pipelines

A single CI/CD platform means one OIDC trust relationship per cloud account instead of one per platform per account. This is one of the strongest arguments for consolidation: with three platforms, you manage three sets of trust policies, three sets of subject claim restrictions, and three sets of credential scoping rules. With one platform, you manage one.

Every pipeline that interacts with a cloud provider must authenticate via OIDC federation -- no stored credentials, no long-lived API keys. For the complete OIDC setup (trust policies, flow diagrams, subject claim restrictions, per-provider Terraform code), see the `zero-static-credentials` skill.

## Migrating from Jenkins

Jenkins is the most common migration source. The migration is worth it despite the effort, because Jenkins carries unique operational costs: JVM maintenance, plugin version conflicts, Groovy pipeline syntax (which few engineers enjoy debugging), and the Jenkins controller as a single point of failure.

### Migration Strategy

| Phase | Action | Duration |
|-------|--------|----------|
| 1. Inventory | List all Jenkins jobs, triggers, and secrets | 1 week |
| 2. Classify | Group by type: CI, CD, scheduled, one-off | 1 day |
| 3. Pilot | Migrate 2-3 simple CI jobs to the new platform | 1 week |
| 4. Templates | Create reusable workflow templates from the pilots | 1 week |
| 5. Bulk migrate | Use templates to migrate remaining jobs | 2-4 weeks |
| 6. Parallel run | Run both platforms for 2 weeks, verify parity | 2 weeks |
| 7. Cutover | Disable Jenkins jobs, redirect all triggers | 1 day |
| 8. Decommission | Remove Jenkins infrastructure after 30-day grace period | 1 day |

**Do not attempt a big-bang migration**. Run both platforms in parallel during the transition. Disable Jenkins jobs one at a time as their replacements prove stable.

## Good vs. Bad Patterns

```
BAD: Multiple CI/CD platforms
- CircleCI for application builds (team A set it up in 2021)
- GitLab CI for infrastructure (the infra engineer preferred it)
- GitHub Actions for deployments (added later for OIDC support)
- Jenkins for scheduled jobs (legacy, nobody wants to touch it)
- Result: 4 platforms, 4 syntaxes, 4 secrets stores, 4 bills

GOOD: Single CI/CD platform
- GitHub Actions for application CI
- GitHub Actions for infrastructure CI/CD
- GitHub Actions for application CD
- GitHub Actions for scheduled jobs (drift detection, cert checks)
- GitHub Actions for security scans
- Result: 1 platform, 1 syntax, 1 secrets store, 1 bill
```

```
BAD: Stored credentials for pipeline auth
- AWS_ACCESS_KEY_ID in CI/CD secrets
- Same key used across all pipelines
- Key is 2 years old, never rotated
- Any repo can use it (no scoping)

GOOD: OIDC federation for pipeline auth
- No credentials stored in CI/CD platform
- Each job gets unique, short-lived tokens
- Trust policy restricts access to specific repos and refs
- Revoking access = update trust policy (no pipeline changes)
```

## Cloud Provider Translation

| Concept | AWS | GCP | Azure |
|---------|-----|-----|-------|
| Scheduled pipelines | GitHub Actions `schedule`, EventBridge | Cloud Scheduler + Cloud Build | Azure Pipelines schedules |
| Approval gates | GitHub Environments (protection rules) | Cloud Build Approval | Azure DevOps Approvals |
| Self-hosted runners | GitHub Actions self-hosted runners | Cloud Build private pools | Azure DevOps self-hosted agents |
| Pipeline secrets | GitHub Actions Secrets (per-repo or org) | Secret Manager + WIF access | Azure Key Vault + federated auth |
| OIDC setup | See `zero-static-credentials` skill | See `zero-static-credentials` skill | See `zero-static-credentials` skill |

## Examples

Working implementations in `examples/`:
- **`examples/platform-consolidation-plan.md`** -- Step-by-step migration plan for consolidating from multiple CI/CD providers to a single platform, including inventory template, parallel run strategy, and cutover checklist
- **`examples/oidc-trust-policies.md`** -- Role-per-pipeline-type pattern (plan, build, deploy) with subject claim restrictions scoping which branches and events can assume each role
- For OIDC provider setup and federation configuration, see `zero-static-credentials` skill (`examples/oidc-federation.md`)

## Review Checklist

When designing or reviewing CI/CD platform architecture:

- [ ] All pipeline types (app CI, infra CI, infra CD, app CD, scheduled jobs) run on a single platform
- [ ] The CI/CD platform is native to the code hosting platform (GitHub -> GitHub Actions, GitLab -> GitLab CI)
- [ ] No pipeline authenticates to cloud providers using stored credentials (OIDC only -- see `zero-static-credentials` skill)
- [ ] OIDC trust policies include subject claim restrictions (see `zero-static-credentials` skill)
- [ ] Production deploy roles are only assumable from tag refs, not branch refs (see `tag-based-production-deploys` skill)
- [ ] Drift detection runs on a schedule (daily or weekly) and creates alerts when changes are found
- [ ] No Jenkins jobs remain in production (or a migration plan with timeline exists)
- [ ] Reusable workflow templates exist for common pipeline patterns (CI, CD, scheduled)
- [ ] Secrets are managed in one place (the CI/CD platform's native secrets store or external vault)
- [ ] Pipeline monitoring and alerting covers all pipeline types, not just application deployments
- [ ] New engineers can understand the entire CI/CD system by learning one platform
- [ ] Runner infrastructure (if self-hosted) is managed on the single platform, not split across providers
- [ ] Cost visibility exists for CI/CD spend (one bill, not three)
- [ ] The decision to use a single platform is documented in an ADR (see `architecture-decision-records` skill)
