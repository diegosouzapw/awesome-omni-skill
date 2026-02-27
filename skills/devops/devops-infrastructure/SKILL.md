---
name: devops-infrastructure
description: "Use when provisioning infrastructure, building containers, configuring CI/CD, or deploying services - ensures all infrastructure is codified, versioned, and reviewable with repeatable deployment strategies and proper secrets management | インフラのプロビジョニング、コンテナのビルド、CI/CDの構成、サービスのデプロイ時に使用 - すべてのインフラがコード化、バージョン管理、レビュー可能であることを保証し、再現可能なデプロイ戦略と適切なシークレット管理を実現"
---

# DevOps Infrastructure

## Overview

Manual infrastructure changes are incidents waiting to happen. Unversioned configs are undocumented debt.

**Core principle:** EVERY piece of infrastructure is defined in code, stored in version control, and deployed through automation.

**Violating the letter of this process is violating the spirit of infrastructure engineering.**

## The Iron Law

```
NO MANUAL INFRASTRUCTURE CHANGES - EVERYTHING IS CODE, VERSIONED, AND REVIEWABLE
```

If you clicked through a console UI to create it, it doesn't exist yet. Write the code.

## When to Use

Use for ANY infrastructure work:
- Provisioning cloud resources
- Building container images
- Configuring CI/CD pipelines
- Setting up deployment strategies
- Managing secrets and credentials
- Configuring monitoring and alerting
- Setting up networking and DNS

**Use this ESPECIALLY when:**
- Under deadline pressure ("just create it in the console for now")
- A quick manual fix seems faster
- Someone says "we'll codify it later"
- You're setting up a "temporary" environment
- Debugging a production issue by hand

**Don't skip when:**
- It's "just one resource" (one resource becomes twenty)
- It's a dev environment (dev environments become templates)
- It's a one-time setup (nothing is one-time)

## The Five Phases

You MUST complete each phase before proceeding to the next.

### Phase 1: Define Infrastructure as Code

**BEFORE creating ANY resource:**

1. **Choose Your IaC Tool**
   - Terraform: Multi-cloud, mature ecosystem, declarative
   - Pulumi: General-purpose languages, strong typing
   - CloudFormation: AWS-native, deep integration
   - Pick one per project and stick with it

2. **Structure Your Code**
   ```
   infrastructure/
   ├── modules/           # Reusable components
   │   ├── networking/
   │   ├── compute/
   │   └── database/
   ├── environments/      # Per-environment configs
   │   ├── dev/
   │   ├── staging/
   │   └── production/
   ├── variables.tf       # Input definitions
   └── outputs.tf         # Exported values
   ```

3. **State Management**
   - Remote state (S3 + DynamoDB, GCS, Terraform Cloud)
   - State locking enabled - always
   - Never commit state files to git
   - One state per environment minimum

4. **Plan Before Apply**
   - Always run plan/preview first
   - Review every change in the plan
   - Automate plan output in PRs
   - Never apply without reviewing the plan

### Phase 2: Container Best Practices

**Every container image follows these rules:**

1. **Multi-Stage Builds**

   <Good>
   ```dockerfile
   FROM node:20-alpine AS builder
   WORKDIR /app
   COPY package*.json ./
   RUN npm ci --production=false
   COPY . .
   RUN npm run build

   FROM node:20-alpine AS runtime
   WORKDIR /app
   RUN addgroup -g 1001 appgroup && adduser -u 1001 -G appgroup -s /bin/sh -D appuser
   COPY --from=builder /app/dist ./dist
   COPY --from=builder /app/node_modules ./node_modules
   USER appuser
   EXPOSE 3000
   CMD ["node", "dist/index.js"]
   ```
   Small image, non-root user, only production artifacts
   </Good>

   <Bad>
   ```dockerfile
   FROM node:20
   WORKDIR /app
   COPY . .
   RUN npm install
   CMD ["npm", "start"]
   ```
   Bloated image, root user, dev dependencies included, source code exposed
   </Bad>

2. **Minimal Base Images**
   - Alpine or distroless
   - Pin exact versions (not `latest`)
   - Rebuild regularly for security patches

3. **Security Scanning**
   - Scan images in CI (Trivy, Snyk, Grype)
   - Block deployment on critical/high CVEs
   - No secrets in images - ever
   - No secrets in build args - they leak in layer history

4. **Image Hygiene**
   - `.dockerignore` for every project
   - One process per container
   - Health checks defined
   - Graceful shutdown handling (SIGTERM)

### Phase 3: Kubernetes Deployment Patterns

**Match the workload to the right abstraction:**

1. **Deployments** - Stateless services
   - Rolling updates by default
   - Resource limits set (CPU and memory)
   - Readiness and liveness probes configured
   - Pod Disruption Budgets defined

2. **StatefulSets** - Databases, caches, queues
   - Stable network identities
   - Persistent volume claims
   - Ordered startup/shutdown
   - Don't use for stateless workloads

3. **CronJobs** - Scheduled tasks
   - Concurrency policy set
   - Failure history limits
   - Deadline seconds configured
   - Idempotent by design

4. **Resource Definitions**

   <Good>
   ```yaml
   resources:
     requests:
       cpu: "100m"
       memory: "128Mi"
     limits:
       cpu: "500m"
       memory: "512Mi"
   ```
   Explicit requests and limits
   </Good>

   <Bad>
   ```yaml
   # No resource limits defined - hope the node has enough
   ```
   Unbounded resource usage, noisy neighbor problems
   </Bad>

### Phase 4: CI/CD Pipeline Design

**Every pipeline follows this structure:**

1. **Pipeline Stages**
   ```
   Lint → Test → Build → Scan → Deploy(staging) → Verify → Deploy(production)
   ```
   - No stage can be skipped
   - Failure at any stage stops the pipeline
   - Production deploy requires explicit approval

2. **Deployment Strategies**

   | Strategy | When | Risk | Rollback |
   |----------|------|------|----------|
   | **Rolling** | Default for most services | Medium | Automatic |
   | **Blue-Green** | Zero-downtime critical services | Low | Instant switch |
   | **Canary** | High-traffic, risk-sensitive | Lowest | Route back to stable |

   - Rolling: Good default, gradual replacement
   - Blue-Green: Full parallel environment, instant cutover
   - Canary: Percentage-based traffic shifting, metrics-driven promotion

3. **Rollback Plan**
   - Every deployment MUST have a rollback plan
   - Automated rollback on health check failure
   - Database migrations must be backward-compatible
   - "We'll figure it out" is not a rollback plan

4. **Pipeline Security**
   - Secrets injected at runtime, never in code
   - Least-privilege service accounts
   - Signed artifacts and images
   - Audit trail for every deployment

### Phase 5: Secrets Management and Monitoring

**Secrets:**

1. **Never in Code**
   - Not in source files
   - Not in Dockerfiles
   - Not in CI config files
   - Not in environment variable definitions committed to git
   - Use vault (HashiCorp Vault, AWS Secrets Manager, 1Password)
   - Inject at runtime via environment or mounted files

2. **Rotation**
   - Secrets have expiration dates
   - Automated rotation where possible
   - Revoke immediately when compromised

**Monitoring and Alerting:**

3. **The Four Golden Signals**
   - Latency: How long requests take
   - Traffic: How many requests per second
   - Errors: Rate of failed requests
   - Saturation: How full your resources are

4. **Alert Design**
   - Alert on symptoms, not causes
   - Every alert has a runbook
   - No alert fatigue - if you ignore it, delete it
   - Page for user-facing impact only

## Red Flags - STOP and Follow Process

If you catch yourself thinking:
- "Just create it in the console, we'll codify later"
- "This is a one-time setup"
- "I'll hardcode the secret for now"
- "We don't need monitoring yet"
- "Skip the staging deploy, push straight to prod"
- "The rollback plan is to fix forward"
- "It's just a small config change, no PR needed"
- "We'll add resource limits later"
- "Root user is fine for now"

**ALL of these mean: STOP. Follow the process.**

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "Console is faster for one resource" | One resource becomes twenty. Codify from the start. |
| "We'll codify it later" | You won't. "Later" means "never" in infrastructure. |
| "It's just a dev environment" | Dev environments are production templates. Treat them the same. |
| "Hardcode the secret for now" | Secrets in code get committed, pushed, leaked. Use a vault. |
| "We don't need monitoring yet" | You need monitoring BEFORE the first incident, not after. |
| "Skip staging, it works on my machine" | Your machine is not production. Deploy to staging first. |
| "Fix forward is our rollback" | Fix forward under pressure creates new incidents. Have a real rollback. |
| "Resource limits slow us down" | Unbounded containers slow everyone down when they consume the node. |
| "Manual change, just this once" | Snowflake servers start with "just this once." |
| "One-time setup doesn't need code" | Nothing is one-time. You'll rebuild, migrate, or recover. |

## Anti-Patterns

| Anti-Pattern | Consequence | Correct Approach |
|-------------|-------------|-----------------|
| **Snowflake servers** | Unreproducible, undocumented, irreplaceable | Everything in IaC, immutable infrastructure |
| **Secrets in code** | Credential leaks, security incidents | Vault/env injection, runtime secrets |
| **No rollback plan** | Extended outages, panic-driven fixes | Automated rollback, backward-compatible migrations |
| **Deploy without approval** | Unreviewed changes in production | PR-based deployments, required approvals |
| **No resource limits** | Noisy neighbors, node exhaustion, cascading failures | Explicit requests and limits on every workload |
| **`latest` tag** | Unreproducible builds, surprise breaking changes | Pin exact versions, rebuild intentionally |

## Quick Reference

| Phase | Key Activities | Success Criteria |
|-------|---------------|------------------|
| **1. IaC** | Define resources in code, remote state, plan before apply | All infrastructure in version control |
| **2. Containers** | Multi-stage builds, minimal images, security scanning | Small, secure, non-root images |
| **3. Kubernetes** | Right abstraction, resource limits, probes | Workloads are resilient and bounded |
| **4. CI/CD** | Pipeline stages, deployment strategy, rollback plan | Automated, gated, reversible deployments |
| **5. Secrets/Monitoring** | Vault injection, four golden signals, alert runbooks | No secrets in code, actionable alerts |

## Verification Checklist

Before marking infrastructure work complete:

- [ ] All resources defined in IaC (no console-created resources)
- [ ] State stored remotely with locking
- [ ] Container images are multi-stage, minimal, non-root
- [ ] Images scanned for vulnerabilities in CI
- [ ] No secrets in code, Dockerfiles, or CI configs
- [ ] Secrets injected from vault/secrets manager at runtime
- [ ] Resource limits set on all workloads
- [ ] Health checks and probes configured
- [ ] Deployment strategy chosen with rollback plan documented
- [ ] Production deploy requires approval
- [ ] Monitoring covers four golden signals
- [ ] Every alert has a runbook

Can't check all boxes? You're not done.

## Integration with Other Skills

**This skill requires using:**
- **test-driven-development** - REQUIRED for testing IaC changes (Terratest, infrastructure integration tests)

**Complementary skills:**
- **systematic-debugging** - Use when deployments fail or infrastructure behaves unexpectedly
- **documentation-generation** - Generate runbooks, architecture diagrams, and operational docs

## Final Rule

```
If it's not in code, it doesn't exist.
If it's not in version control, it's not real.
If it can't be reviewed, it can't be trusted.
```

No exceptions without your human partner's explicit approval.

