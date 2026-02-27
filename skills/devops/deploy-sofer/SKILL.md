---
name: deploy
description: Ship code to target environment. Use after code is merged to deploy to staging or production. Handles build, artifact creation, and deployment orchestration.
---

# Deploy

Ship code to a target environment (staging, production, or custom).

## Purpose

Deploy handles the release process:
- Build application artifacts
- Run pre-deployment checks
- Deploy to target environment
- Verify deployment success
- Handle rollback if needed

## Input

Expect from orchestrator or user:
- Target environment (staging, production, custom)
- Version/tag to deploy (or "latest")
- Deployment configuration
- Rollback preferences

## Process

### 1. Pre-deployment checks

Verify readiness before deploying:

```yaml
pre_checks:
  - check: "All tests passing"
    command: "npm test"
    required: true

  - check: "Build succeeds"
    command: "npm run build"
    required: true

  - check: "No pending migrations"
    command: "npm run db:status"
    required: false

  - check: "Environment variables set"
    command: "scripts/check-env.sh"
    required: true
```

### 2. Build artifacts

Create deployable artifacts:

```bash
# Build application
npm run build

# Create Docker image (if applicable)
docker build -t app:${VERSION} .

# Package artifacts
tar -czf dist-${VERSION}.tar.gz dist/
```

### 3. Tag release

Create version tag:

```bash
# Semantic versioning
git tag -a v${VERSION} -m "Release ${VERSION}"
git push origin v${VERSION}
```

### 4. Deploy to environment

#### Container-based deployment

```bash
# Push to registry
docker push registry.example.com/app:${VERSION}

# Update deployment
kubectl set image deployment/app app=registry.example.com/app:${VERSION}

# Or using Helm
helm upgrade app ./charts/app --set image.tag=${VERSION}
```

#### Platform-specific deployment

**Vercel/Netlify:**
```bash
vercel --prod
# or
netlify deploy --prod
```

**AWS:**
```bash
aws s3 sync dist/ s3://bucket-name/
# or
aws ecs update-service --cluster prod --service app --force-new-deployment
```

**Heroku:**
```bash
git push heroku main
```

**Custom server:**
```bash
rsync -avz dist/ user@server:/var/www/app/
ssh user@server 'sudo systemctl restart app'
```

### 5. Database migrations (if applicable)

If the story artifacts include migration files:

1. **Validate**: Dry-run migration against staging/production-like environment
   ```bash
   npm run db:migrate:dry-run
   ```

2. **Human approval**: Present migration plan for explicit approval
   - Show: tables affected, columns added/removed, data transformations
   - Show: rollback plan
   - This is a mandatory human-in-the-loop checkpoint

3. **Apply**: Run migration against production
   ```bash
   npm run db:migrate
   ```

4. **Verify**: Confirm schema is correct
   ```bash
   npm run db:status
   ```

5. **On failure**: Execute rollback migration immediately
   ```bash
   npm run db:migrate:rollback
   ```
   Then halt deployment and escalate.

**Critical**: Production migrations must be applied AFTER merge and DURING deployment, never before merge. Applying migrations before merge creates a window where production schema doesn't match running code.

### 6. Verify deployment

Confirm deployment succeeded:

```yaml
verification:
  - check: "Health endpoint responds"
    command: "curl -f https://app.example.com/health"
    expected: "200 OK"

  - check: "Version matches"
    command: "curl https://app.example.com/version"
    expected: "${VERSION}"

  - check: "Key functionality works"
    command: "npm run smoke-test"
    expected: "All smoke tests pass"
```

### 7. Handle failure

If deployment fails:

```yaml
rollback:
  trigger: "Health check fails after 3 attempts"
  actions:
    - "Revert to previous version"
    - "Notify team"
    - "Log failure details"

  commands:
    kubernetes: "kubectl rollout undo deployment/app"
    docker: "docker service update --rollback app"
    heroku: "heroku rollback"
```

## Environment configuration

```yaml
environments:
  staging:
    url: "https://staging.example.com"
    auto_deploy: true
    approval_required: false
    rollback_on_failure: true

  production:
    url: "https://app.example.com"
    auto_deploy: false
    approval_required: true
    rollback_on_failure: true
    notify:
      - "ops-team@example.com"
```

## Output

```yaml
deploy:
  version: "1.2.3"
  environment: "production"
  status: "success | failed | rolled_back"

  timeline:
    started: "2024-01-15T10:00:00Z"
    completed: "2024-01-15T10:05:32Z"
    duration: "5m32s"

  pre_checks:
    - name: "tests"
      status: "pass"
    - name: "build"
      status: "pass"

  artifacts:
    - type: "docker_image"
      location: "registry.example.com/app:1.2.3"
    - type: "source_map"
      location: "s3://artifacts/app-1.2.3-sourcemaps.tar.gz"

  verification:
    health_check: "pass"
    version_check: "pass"
    smoke_tests: "pass"

  url: "https://app.example.com"
  commit: "abc123def"
  deployed_by: "orchestrator"

  notes: ""
```

Update manifest:
```yaml
releases:
  - version: "1.2.3"
    environment: "production"
    date: "2024-01-15T10:05:32Z"
    stories: ["US-001", "US-002", "US-003"]
    status: "deployed"
```

## Deployment strategies

### Blue-green deployment
```yaml
strategy: "blue-green"
process:
  - Deploy to inactive environment (green)
  - Run verification tests
  - Switch traffic from blue to green
  - Keep blue as rollback target
```

### Canary deployment
```yaml
strategy: "canary"
process:
  - Deploy to canary (small % of traffic)
  - Monitor for errors
  - Gradually increase traffic
  - Full rollout or rollback based on metrics
```

### Rolling deployment
```yaml
strategy: "rolling"
process:
  - Update instances one at a time
  - Verify each instance before proceeding
  - Maintain availability throughout
```

## Notifications

```yaml
notifications:
  on_start:
    - channel: "slack"
      message: "🚀 Deploying v${VERSION} to ${ENV}"

  on_success:
    - channel: "slack"
      message: "✅ v${VERSION} deployed to ${ENV}"
    - channel: "email"
      recipients: ["team@example.com"]

  on_failure:
    - channel: "slack"
      message: "❌ Deployment failed: ${ERROR}"
    - channel: "pagerduty"
      severity: "high"
```

## Tips

- Always deploy to staging first
- Have a rollback plan before deploying
- Monitor closely after production deploys
- Keep deployments small and frequent
- Automate as much as possible
- Document any manual steps required
