---
model: haiku
created: 2025-12-16
modified: 2026-02-10
reviewed: 2025-12-16
description: Check and configure feature flag infrastructure (OpenFeature + providers)
allowed-tools: Glob, Grep, Read, Write, Edit, Bash, AskUserQuestion, TodoWrite, WebSearch, WebFetch
args: "[--check-only] [--fix] [--provider <goff|flagd|launchdarkly|split>]"
argument-hint: "[--check-only] [--fix] [--provider <goff|flagd|launchdarkly|split>]"
name: configure-feature-flags
---

# /configure:feature-flags

Check and configure feature flag infrastructure using the OpenFeature standard with pluggable providers.

## When to Use This Skill

| Use this skill when... | Use another approach when... |
|------------------------|------------------------------|
| Adding feature flag infrastructure to a new project | Creating or editing individual flag definitions in YAML |
| Setting up OpenFeature SDK with a provider (GOFF, flagd, LaunchDarkly) | Debugging why a specific flag evaluation returns unexpected values |
| Auditing existing feature flag configuration for completeness | Writing application logic that consumes feature flags |
| Configuring relay proxy infrastructure (Docker, Kubernetes) | Managing LaunchDarkly or Split dashboard settings |
| Adding feature flag test helpers and in-memory providers | Configuring error tracking (`/configure:sentry` instead) |

## Context

- Package JSON: !`find . -maxdepth 1 -name \'package.json\'`
- Python project: !`find . -maxdepth 1 -name \'pyproject.toml\'`
- Go project: !`find . -maxdepth 1 -name \'go.mod\'`
- Cargo project: !`find . -maxdepth 1 -name \'Cargo.toml\'`
- OpenFeature SDK: !`grep -l 'openfeature' package.json pyproject.toml Cargo.toml go.mod`
- GOFF config: !`find . -maxdepth 2 -name 'flags.goff.yaml' -o -name 'flags.goff.yml'`
- Docker compose: !`find . -maxdepth 1 -name 'docker-compose*.yml' -o -name 'docker-compose*.yaml'`
- Project standards: !`find . -maxdepth 1 -name \'.project-standards.yaml\'`

## Parameters

Parse from `$ARGUMENTS`:

- `--check-only`: Report compliance status without modifications
- `--fix`: Apply all fixes automatically without prompting
- `--provider <provider>`: Override provider detection (goff, flagd, launchdarkly, split)

## Version Checking

**CRITICAL**: Before configuring feature flags, verify latest SDK and provider versions using WebSearch or WebFetch:

1. **OpenFeature JS SDK**: Check [npm](https://www.npmjs.com/package/@openfeature/js-sdk)
2. **OpenFeature Python SDK**: Check [PyPI](https://pypi.org/project/openfeature-sdk/)
3. **GO Feature Flag**: Check [GitHub releases](https://github.com/thomaspoignant/go-feature-flag/releases)
4. **flagd**: Check [GitHub releases](https://github.com/open-feature/flagd/releases)

## Execution

Execute this feature flag configuration workflow:

### Step 1: Detect project language and existing setup

Check for existing feature flag infrastructure:

| Indicator | Language | Detected Provider |
|-----------|----------|-------------------|
| `@openfeature/server-sdk` in package.json | Node.js | OpenFeature (check for provider) |
| `@openfeature/web-sdk` in package.json | Browser JS | OpenFeature Web |
| `@openfeature/react-sdk` in package.json | React | OpenFeature React |
| `openfeature-sdk` in pyproject.toml | Python | OpenFeature Python |
| `@openfeature/go-feature-flag-provider` | Node.js | GO Feature Flag |
| `go-feature-flag-relay-proxy` in docker-compose | Any | GO Feature Flag Relay |
| `flagd` in docker-compose/k8s | Any | flagd provider |

### Step 2: Analyze current state

Check for complete feature flag setup:

1. Verify OpenFeature SDK installed for project language
2. Check provider package installed
3. Verify provider initialized in application startup
4. Check evaluation context configuration
5. Check hooks configured (logging, telemetry)
6. For GOFF: verify flag configuration file exists, relay proxy configured
7. For flagd: verify container/service configured, gRPC/HTTP endpoints

### Step 3: Generate compliance report

Print a formatted compliance report:

```
Feature Flag Compliance Report
==============================
Project: [name]
Language: [detected]
Provider: [detected or None]

OpenFeature SDK:    [status per check]
Provider Config:    [status per check]
Infrastructure:     [status per check]

Overall: [X issues found]
Recommendations: [list specific fixes]
```

If `--check-only`, stop here.

### Step 4: Install SDK and configure provider (if --fix or user confirms)

Based on detected language, install and configure the OpenFeature SDK with the selected provider. Use code templates from [REFERENCE.md](REFERENCE.md).

1. Install OpenFeature SDK and provider packages
2. Create feature flag client wrapper module
3. Create evaluation context helper
4. Create middleware for HTTP frameworks (Express, FastAPI, etc.)

### Step 5: Create flag configuration

Create `flags.goff.yaml` with example flags covering common patterns:
- Simple boolean flag
- Percentage rollout
- Multi-variant flag (A/B test)
- Environment-specific flag
- User-specific override
- Scheduled rollout

Use flag templates from [REFERENCE.md](REFERENCE.md).

### Step 6: Configure infrastructure

1. Create docker-compose entry for relay proxy (local development)
2. Optionally create Kubernetes manifests for production

### Step 7: Create test configuration

Create test file using in-memory provider for feature flag unit tests.

### Step 8: Update standards tracking

Update `.project-standards.yaml`:

```yaml
components:
  feature_flags: "2025.1"
  feature_flags_sdk: "openfeature"
  feature_flags_provider: "[goff|flagd|launchdarkly]"
```

### Step 9: Print completion report

Print a summary of all changes made including SDK installed, provider configured, flag file created, and next steps (start relay, initialize in app, use flags in code).

For detailed code templates, flag configuration patterns, and infrastructure manifests, see [REFERENCE.md](REFERENCE.md).

## Agentic Optimizations

| Context | Command |
|---------|---------|
| Quick compliance check | `/configure:feature-flags --check-only` |
| Auto-fix with GOFF provider | `/configure:feature-flags --fix --provider goff` |
| Validate flag config | `goff lint --file flags.goff.yaml` |
| Check relay proxy health | `curl -s http://localhost:1031/health | jq -c` |
| List configured flags | `curl -s http://localhost:1031/v1/feature/flags | jq -r 'keys[]'` |
| Check SDK installed (JS) | `jq -r '.dependencies | keys[] | select(contains("openfeature"))' package.json` |

## Flags

| Flag | Description |
|------|-------------|
| `--check-only` | Report status without offering fixes |
| `--fix` | Apply all fixes automatically without prompting |
| `--provider <provider>` | Override provider detection (goff, flagd, launchdarkly, split) |

## Examples

```bash
# Check compliance and offer fixes
/configure:feature-flags

# Check only, no modifications
/configure:feature-flags --check-only

# Auto-fix with GO Feature Flag provider
/configure:feature-flags --fix --provider goff

# Configure for LaunchDarkly
/configure:feature-flags --fix --provider launchdarkly
```

## Error Handling

- **No package manager found**: Cannot install SDK, provide manual steps
- **Provider not supported**: List supported providers, suggest alternatives
- **Relay proxy unreachable**: Check Docker/K8s configuration
- **Invalid flag syntax**: Validate with `goff lint` before deployment

## See Also

- `/configure:all` - Run all compliance checks
- `/configure:sentry` - Error tracking (often used with feature flags for rollback)
- **OpenFeature documentation**: https://openfeature.dev
- **GO Feature Flag documentation**: https://gofeatureflag.org
