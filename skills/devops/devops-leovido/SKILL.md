---
name: devops
description: DevOps standards, CI/CD pipelines, and containerization practices. Use this when users need guidance on setting up GitHub Actions workflows, Docker containerization, monitoring with Prometheus and Grafana, or implementing CI/CD pipelines for React and React Native projects.
license: MIT - Complete terms in LICENSE.txt
---

# DevOps Skills & Best Practices

DevOps standards, CI/CD pipelines, and containerization practices.

## Table of Contents

- [CI/CD](#cicd)
- [Docker](#docker)
- [GitHub CLI (gh) for DevOps](#github-cli-gh-for-devops)

---

## CI/CD

### GitHub Actions

**Strict Requirements**: When creating a Pull Request, the following checks **MUST** run and **MUST** pass before merging:

1. **Security Checks** (Strict)
   - Dependency vulnerability scanning
   - Use tools like `pnpm audit`, Dependabot, or Snyk

2. **SonarQube Analysis** (Strict)
   - Code quality and security analysis
   - Requires initial SonarQube project setup
   - Configure quality gates and thresholds
   - Use latest stable SonarQube version

3. **Unit Tests** (Strict)
   - Run full test suite
   - Generate coverage reports
   - Fail if tests fail or coverage drops below threshold

4. **Linting** (Strict)
   - Run Biome linting
   - Fail on linting errors
   - Optionally auto-fix and commit changes

**Required GitHub Actions versions:**
- `actions/checkout@v4`
- `pnpm/action-setup@v2` (with pnpm version >= 10)
- `actions/setup-node@v4` (with Node.js 22.x)

**Example GitHub Actions workflow:**

**Note**: The following PR checks are **strict requirements** - all must be included and pass before merging.

```yaml
name: PR Checks

on:
  pull_request:
    branches: [main, develop]

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
        with:
          version: 10
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'pnpm'
      - run: pnpm install --frozen-lockfile
      - run: pnpm audit --audit-level=moderate

  sonarqube:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: pnpm/action-setup@v2
        with:
          version: 10
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'pnpm'
      - run: pnpm install --frozen-lockfile
      - name: SonarQube Scan
        uses: sonarsource/sonarqube-scan-action@master
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
        with:
          version: 10
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'pnpm'
      - run: pnpm install --frozen-lockfile
      - run: pnpm run test:ci
      - uses: codecov/codecov-action@v3
        with:
          token: ${{ secrets.CODECOV_TOKEN }}

  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
        with:
          version: 10
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'pnpm'
      - run: pnpm install --frozen-lockfile
      - run: pnpm run lint
      - run: pnpm run format:check
```

---

## Docker

### Containerization

Set up Docker and Docker Compose for local development and deployment.

**Docker Compose Structure:**

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
    volumes:
      - .:/app
      - /app/node_modules
    depends_on:
      - prometheus
      - grafana

  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus:/etc/prometheus
      - prometheus_data:/prometheus

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3001:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana_data:/var/lib/grafana
    depends_on:
      - prometheus

volumes:
  prometheus_data:
  grafana_data:
```

**Dockerfile Best Practices:**

- Use multi-stage builds for smaller images
- Leverage layer caching
- Use specific version tags for base images
- Run as non-root user when possible
- Include health checks

**Additional Services:**
- Database services (PostgreSQL, MySQL, MongoDB) can be added to docker-compose.yml as needed
- Configure service dependencies appropriately

---

## GitHub CLI (gh) for DevOps

The GitHub CLI (`gh`) provides powerful commands for managing GitHub Actions workflows, runs, secrets, and other DevOps operations.

### GitHub Actions Workflow Management

**List and View Workflows:**
```bash
# List all workflows in repository
gh workflow list

# View workflow details
gh workflow view <workflow-id>

# View workflow YAML file
gh workflow view <workflow-id> --yaml

# View workflow runs
gh run list

# View specific workflow run
gh run view <run-id>

# View workflow run logs
gh run view <run-id> --log

# Watch workflow run in real-time
gh run watch <run-id>
```

**Run and Manage Workflows:**
```bash
# Manually trigger a workflow
gh workflow run <workflow-id>

# Run workflow with inputs
gh workflow run <workflow-id> --field key=value

# Rerun a failed workflow
gh run rerun <run-id>

# Cancel a running workflow
gh run cancel <run-id>

# Delete workflow runs
gh run delete <run-id>
```

**For AI Agents:**
- Monitor CI/CD pipeline status after PR creation
- Automatically rerun failed workflows after fixes
- Cancel long-running workflows when needed
- Retrieve workflow logs for debugging

### Cache Management

**Manage GitHub Actions Caches:**
```bash
# List all caches
gh cache list

# Delete specific cache
gh cache delete <cache-id>

# Delete all caches (use with caution)
gh cache list --json id --jq '.[].id' | xargs -I {} gh cache delete {}
```

**For AI Agents:**
- Clear corrupted caches that cause build failures
- Monitor cache usage and efficiency
- Clean up old caches to free space

### Secrets and Variables Management

**Repository Secrets:**
```bash
# List secrets
gh secret list

# Set a secret
gh secret set SECRET_NAME --body "secret-value"

# Delete a secret
gh secret delete SECRET_NAME
```

**Repository Variables:**
```bash
# List variables
gh variable list

# Get variable value
gh variable get VARIABLE_NAME

# Set variable
gh variable set VARIABLE_NAME --body "variable-value"

# Delete variable
gh variable delete VARIABLE_NAME
```

**For AI Agents:**
- Securely manage secrets for CI/CD pipelines
- Configure environment-specific variables
- Rotate secrets when needed
- Verify secret configuration before workflow runs

### Workflow Status Monitoring

**Check PR Status:**
```bash
# Check all checks for a PR
gh pr checks <pr-number>

# Wait for checks to complete
gh pr checks <pr-number> --watch

# View check details
gh run view <run-id> --log
```

**For AI Agents:**
- Wait for CI checks before merging PRs
- Report check status to users
- Retry failed checks automatically
- Block merges until all checks pass

### Advanced Workflow Operations

**Workflow Enable/Disable:**
```bash
# Disable a workflow
gh workflow disable <workflow-id>

# Enable a workflow
gh workflow enable <workflow-id>
```

**Download Artifacts:**
```bash
# List artifacts from a run
gh run view <run-id> --json artifacts

# Download artifacts
gh run download <run-id>
```

**For AI Agents:**
- Temporarily disable workflows during maintenance
- Download build artifacts for testing
- Archive artifacts for deployment

### Integration with CI/CD Pipeline

**Example: Automated Deployment Workflow**

```bash
#!/bin/bash
# Automated deployment script using gh CLI

# 1. Check if PR is ready
PR_NUMBER=$(gh pr list --head $(git branch --show-current) --json number -q '.[0].number')
if [ -z "$PR_NUMBER" ]; then
  echo "No PR found for current branch"
  exit 1
fi

# 2. Wait for all checks to pass
echo "Waiting for CI checks..."
gh pr checks $PR_NUMBER --watch

# 3. Verify checks passed
CHECKS_STATUS=$(gh pr checks $PR_NUMBER --json status -q '.[].status')
if [[ "$CHECKS_STATUS" == *"FAILURE"* ]]; then
  echo "Some checks failed. Deployment aborted."
  exit 1
fi

# 4. Merge PR
gh pr merge $PR_NUMBER --squash --delete-branch

# 5. Wait for deployment workflow
echo "Waiting for deployment workflow..."
DEPLOY_RUN=$(gh run list --workflow=deploy.yml --limit 1 --json databaseId -q '.[0].databaseId')
gh run watch $DEPLOY_RUN

# 6. Verify deployment
DEPLOY_STATUS=$(gh run view $DEPLOY_RUN --json conclusion -q '.conclusion')
if [ "$DEPLOY_STATUS" != "success" ]; then
  echo "Deployment failed!"
  exit 1
fi

echo "Deployment successful!"
```

**For AI Agents:**
- Automate complete CI/CD pipeline from PR to deployment
- Monitor each stage and handle failures appropriately
- Provide status updates throughout the process
- Rollback on deployment failures

### Environment and Deployment Management

**Deployment Status:**
```bash
# View deployment status (via API)
gh api repos/:owner/:repo/deployments

# View deployment environments
gh api repos/:owner/:repo/environments
```

**For AI Agents:**
- Track deployment status across environments
- Verify deployments before proceeding
- Manage environment-specific configurations

### Best Practices for AI Agents

1. **Always verify workflow status**: Check `gh pr checks` before merging
2. **Monitor long-running workflows**: Use `gh run watch` for real-time updates
3. **Handle failures gracefully**: Implement retry logic for transient failures
4. **Secure secret management**: Never log or expose secret values
5. **Use appropriate workflow triggers**: Understand when to use manual vs automatic triggers
6. **Clean up resources**: Delete old caches and artifacts periodically
7. **Provide clear status**: Report workflow status in user-friendly format
8. **Respect rate limits**: Implement delays for bulk operations
9. **Use JSON output**: Parse `--json` output for programmatic processing
10. **Error handling**: Check exit codes and provide meaningful error messages

### Troubleshooting CI/CD Issues

**Common Commands for Debugging:**
```bash
# View recent failed runs
gh run list --status failure --limit 10

# View logs for failed run
gh run view <run-id> --log --failed

# Check workflow syntax
gh workflow view <workflow-id> --yaml

# Verify secrets are set
gh secret list

# Check cache status
gh cache list
```

**For AI Agents:**
- Automatically diagnose CI/CD failures
- Suggest fixes based on error logs
- Verify configuration before retrying
- Report issues with actionable solutions

---

## Additional Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [GitHub CLI Documentation](https://cli.github.com/manual/)
- [GitHub Actions Workflow Syntax](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)

---

## Notes

- This document should be reviewed and updated regularly as best practices evolve
- Team-specific additions and modifications are encouraged
- When in doubt, refer to official documentation and community standards

