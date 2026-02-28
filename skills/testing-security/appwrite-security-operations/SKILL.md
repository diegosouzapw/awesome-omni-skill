---
name: appwrite-security-operations
description: Production security operations for Appwrite services. Use when defining security quality gates, runtime hardening, API key lifecycle management, dependency vulnerability control, and incident response workflows.
---

# Appwrite Security Operations

Use this skill for production-grade security governance beyond basic app implementation.

## Scope

- Security design quality gates before release.
- Operational hardening for Cloud and self-hosted Appwrite.
- API key lifecycle and secret handling.
- Dependency and supply-chain vulnerability control.
- Ongoing monitoring and incident response.

## Security Quality Gates

1. Define threat model and trust boundaries per service path.
2. Map every endpoint to required auth mode and permission boundary.
3. Run negative authorization tests for cross-tenant and cross-role abuse.
4. Validate all privileged write paths execute only in trusted server runtimes.
5. Block release if any critical security control is missing.

## Appwrite Operational Hardening

### Cloud and self-hosted

- Keep abuse protections enabled in production and monitor 429 patterns.
- Register only approved platforms to enforce CORS boundaries.
- Use Appwrite Activity and audit logs, then mirror security evidence externally because native retention is short.
- Keep Appwrite versions up to date and follow migration steps during upgrades.

### Self-hosted specific

- Set `_APP_OPENSSL_KEY_V1` immediately after installation and protect it as a critical secret.
- Do not rotate `_APP_OPENSSL_KEY_V1` casually, because changing it breaks access to encrypted values.
- Force HTTPS in production with `_APP_OPTIONS_FORCE_HTTPS`.
- Restrict console access with IP and email allowlists where possible.
- Re-enable abuse protections in production with `_APP_OPTIONS_ABUSE=enabled`.
- Use tested backup and restore procedures for database, volumes, and `.env`.

## API Key Lifecycle

1. Use API keys only on trusted server paths and never in client apps.
2. Create one key per service boundary and environment.
3. Grant least scopes required for each service.
4. Rotate by creating a replacement key, deploying new secret, validating traffic, then deleting old key.
5. Keep break-glass revocation procedure ready for emergency key exposure.

## Dev Keys and Environment Separation

- Use Dev keys only for local development and testing.
- Never ship Dev keys in production clients.
- Keep dev, stage, and prod projects and secrets fully isolated.

## Dependency Vulnerability Program

### Repository controls

- Enable dependency graph, Dependabot alerts, and Dependabot security updates.
- Use `.github/dependabot.yml` to tune update cadence and grouping.
- Gate pull requests with dependency review checks at a defined severity threshold.
- Enable secret scanning to catch leaked credentials early.

### Continuous scanning

- Run package-manager audit commands in CI for each ecosystem in use.
- Use OSV-Scanner for source and container image scanning when operating containers or mixed ecosystems.
- Treat lockfiles as mandatory for reliable dependency graph and vulnerability resolution.

### Remediation policy

- Fix critical and high vulnerabilities within strict SLA.
- Document accepted-risk exceptions with expiry and owner.
- Prefer minimal-version upgrades first, then controlled major upgrades when needed.
- Verify runtime behavior and authorization tests after each security patch release.

## Monitoring and Response

1. Define alert routing for authentication abuse, permission failures, and suspicious traffic spikes.
2. Keep incident runbooks for key leak, account takeover, and data exposure scenarios.
3. Preserve forensic evidence from Appwrite activity logs and external logging pipeline.
4. Run incident drills and permission regression tests on a regular cadence.

## Repository Automation Defaults

- Use `.github/workflows/security-ci.yml` as the default CI security baseline.
- Use `.github/dependabot.yml` to keep Actions dependencies updated.
- Use `.github/pull_request_template.md` to enforce review-time security checks.
- Use `SECURITY_GATES.md` as readiness scoring criteria with evidence-backed assessment.

## References

- https://appwrite.io/docs/advanced/platform/api-keys
- https://appwrite.io/docs/advanced/security/abuse-protection
- https://appwrite.io/docs/advanced/platform/rate-limits
- https://appwrite.io/docs/advanced/security/audit-logs
- https://appwrite.io/docs/advanced/self-hosting/production/security
- https://appwrite.io/docs/advanced/self-hosting/tls-certificates
- https://appwrite.io/docs/advanced/self-hosting/production/backups
- https://appwrite.io/docs/advanced/self-hosting/production/updates
- https://docs.github.com/en/code-security/concepts/supply-chain-security/about-the-dependency-graph
- https://docs.github.com/en/code-security/concepts/supply-chain-security/about-dependabot-security-updates
- https://docs.github.com/en/code-security/dependabot/dependabot-security-updates/configuring-dependabot-security-updates
- https://docs.github.com/en/code-security/concepts/supply-chain-security/about-the-dependabot-yml-file
- https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/configuring-the-dependency-review-action
- https://docs.github.com/github/administering-a-repository/about-token-scanning
- https://docs.npmjs.com/cli/v9/commands/npm-audit/
- https://yarnpkg.com/cli/npm/audit
- https://bun.sh/docs/pm/cli/audit
- https://google.github.io/osv-scanner/usage/
- https://google.github.io/osv-scanner/usage/scan-image
