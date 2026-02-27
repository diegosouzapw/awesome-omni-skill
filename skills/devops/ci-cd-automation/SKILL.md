---
version: 1.0.0
name: CI/CD Automation
description: Automated generation and management of deployment pipelines for Astro projects.
---

# CI/CD Automation: DevOps Agent

> **ACTIVATION PHRASE**: "Activate CI/CD mode" or "Set up deployment pipeline"

This agent automates the creation and configuration of CI/CD pipelines, ensuring safe, performant, and reliable deployments for Astro-based websites.

## 1. Core Capabilities

### Pipeline Generation
- **GitHub Actions**: Generates `.github/workflows/deploy.yml` with caching and parallel jobs.
- **Vercel/Netlify**: Configures environment variables and build commands.
- **Preview Deployments**: Sets up automated previews for pull requests.

### Performance Budgeting
- **Lighthouse CI**: Integrates performance checks into the pipeline.
- **Bundle Size Watch**: Alerts on significant bundle size increases.
- **Build Caching**: Optimizes build times for Astro projects.

### Safety & Compliance
- **Sentinel Sync**: Runs `npm run sentinel` (Astro check) before every deploy.
- **HIPAA Scan**: Integrates with `sentinel-auditor` to check for PHI in logs.
- **Rollback Logic**: Defines emergency rollback procedures.

## 2. Standard Templates

### GitHub Actions (Astro + Vercel)
The agent generates a hardened `.github/workflows/deploy.yml` that includes:
1. **Checkout & Setup**: Node.js environment.
2. **Cache**: npm/pnpm and Astro cache.
3. **Build**: Production build simulation.
4. **Audit**: Lighthouse CI + Type check.
5. **Deploy**: Vercel/Netlify CLI deployment.

## 3. Workflow Logic

### Phase 1: Environment Audit
1. Identify the hosting provider (Vercel, Netlify, or Custom VPS).
2. Check for required secrets (VERCEL_TOKEN, GITHUB_TOKEN).
3. Verify `astro.config.mjs` output settings.

### Phase 2: Configuration
1. Write the YAML/Configuration files.
2. Set up Lefthook/Git Hooks for pre-commit validation.
3. Define the deployment strategy (e.g., Blue-Green if supported).

### Phase 3: Verification
1. Run a dry-run build.
2. Validate the workflow syntax.

## 4. Example Prompts

```text
CI/CD Automation: Generate a GitHub Actions workflow that runs Lighthouse CI and deploys to Vercel on every push to main.
```

```text
CI/CD Automation: Set up a pre-deployment safety check that blocks the build if there are any accessibility errors.
```

---
**Version**: 1.0.0
**Dependencies**: github-manager, sentinel-auditor
