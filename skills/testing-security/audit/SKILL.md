---
name: audit
description: Code quality and compliance audits — dead code detection, PII handling, and SOC 2 readiness
disable-model-invocation: true
arguments:
  - name: type
    description: "Comma-separated list of audit types (e.g., 'dead-code,pii,soc2') or 'all'. If omitted, you will be asked."
    required: false
---

# Code Quality & Compliance Audit

You are performing code quality and compliance audits on this project. These audits are **read-only** — they report findings but do not modify code. This skill is distinct from `/security-audit` (which focuses on vulnerabilities) and `/dependency-audit` (which focuses on npm packages).

## Available Audit Types

| Key         | Audit               | Focus                                                                   |
| ----------- | ------------------- | ----------------------------------------------------------------------- |
| `dead-code` | Dead Code Detection | Unused files, exports, dependencies, and types                          |
| `pii`       | PII Handling Audit  | Personal data flows, logging safety, GDPR compliance                    |
| `soc2`      | SOC 2 Readiness     | Change management, access control, monitoring, vulnerability management |

## Instructions

### 1. Determine audit scope

If `$ARGUMENTS` is provided, parse it as a comma-separated list of audit keys, or `all` for everything. Otherwise, ask the user which audits to run.

### 2. Load context

- Read `package.json` for project metadata and dependencies
- Read `CLAUDE.md` and `.claude/docs/TypeScript Coding Standard for Mission-Critical Systems.md` if present
- Determine source file scope: all `.ts` files under `src/`

### 3. Execute selected audits

---

### Audit: `dead-code` — Dead Code Detection

Analyze the codebase for unused code that increases maintenance burden and attack surface.

1. **Unused exports**: For each exported function, type, constant, and class in `src/`, search the rest of the codebase for imports of that export. Flag exports that are never imported anywhere (excluding `index.ts` barrel re-exports that are themselves unused).

2. **Unused files**: Identify `.ts` files that are not imported by any other file and are not entry points (`src/index.ts`, test files, config files).

3. **Unused dependencies**: For each package in `dependencies` and `devDependencies` in `package.json`:
   - Search `src/` and config files for `import ... from '<package>'` or `require('<package>')`
   - Search config files (`.eslintrc*`, `vitest.config.*`, `tsconfig.json`, etc.) for references
   - Flag packages with zero references

4. **Unused types**: TypeScript interfaces and type aliases that are defined but never referenced.

5. **Commented-out code**: Search for multi-line commented code blocks (3+ consecutive lines of `//` comments that look like code, or `/* */` blocks containing code patterns).

6. **Dead branches**: Code after `return`, `throw`, `break`, or `continue` statements within the same block.

7. **TODO/FIXME/HACK**: Catalog all TODO, FIXME, and HACK comments with file:line locations — these indicate incomplete or temporary code.

If Knip is available (`npx knip --version` succeeds), also run `npx knip --reporter json` and incorporate its findings.

**Output**: Table of dead code items with file, line, category, and suggested action (remove/review).

---

### Audit: `pii` — PII Handling Audit

Analyze the codebase for personally identifiable information handling and GDPR compliance concerns.

1. **PII field detection**: Search for TypeScript interfaces, types, Zod schemas, and variable declarations containing fields commonly associated with PII:
   - Names: `name`, `firstName`, `lastName`, `fullName`, `displayName`
   - Contact: `email`, `phone`, `phoneNumber`, `mobile`, `address`, `city`, `state`, `zip`, `postalCode`, `country`
   - Identity: `ssn`, `socialSecurityNumber`, `nationalId`, `passport`, `driverLicense`, `dateOfBirth`, `dob`, `birthDate`, `age`
   - Financial: `creditCard`, `cardNumber`, `cvv`, `bankAccount`, `iban`, `routingNumber`
   - Digital: `ip`, `ipAddress`, `userAgent`, `deviceId`, `mac`, `geolocation`, `latitude`, `longitude`

2. **Branded type check** (Rule 7.3): For each PII field found, check if it uses a branded type (e.g., `Email` instead of raw `string`). Flag raw primitive PII fields.

3. **Logging exposure**: Search all logging statements (`console.log`, `console.info`, `console.warn`, `console.error`, `logger.info`, `logger.warn`, `logger.error`, `logger.debug`) for:
   - Direct references to PII-typed variables
   - Logging of entire request objects (`logger.info(req)`, `logger.info({ body: req.body })`)
   - Logging of entire user objects
   - String interpolation containing PII field names

4. **Error message exposure**: Check error handlers and Error constructors for PII in messages. Check HTTP error responses for PII leakage.

5. **Validation at boundaries** (Rule 7.2): For each API endpoint or external input handler that receives PII, verify Zod schema validation is present and is the first operation.

6. **Data flow map**: Generate a text-based map showing:
   - Where PII enters the system (API endpoints, file reads, env vars)
   - Where PII is stored (database writes, cache, file system)
   - Where PII exits the system (API responses, emails, logs, exports)
   - Where PII is missing encryption or redaction

**Output**: PII inventory table, compliance gaps, and a data flow summary.

---

### Audit: `soc2` — SOC 2 Readiness Assessment

Evaluate the repository for SOC 2 Trust Services Criteria evidence. Uses `gh` CLI where available for GitHub configuration checks.

1. **Change Management (CC8.1)**:
   - Check branch protection on `main`: `gh api repos/{owner}/{repo}/branches/main/protection` (if `gh` is available)
   - Verify PR reviews are required
   - Verify CI status checks are required before merge
   - Search git history for direct pushes to `main` without PRs: `git log --first-parent main --no-merges --oneline`
   - Verify CI workflows exist and enforce the coding standard

2. **Access Control (CC6.1-CC6.8)**:
   - Check for `.env` files committed to the repository (should be in `.gitignore`)
   - Scan source code for hardcoded secrets (API keys, tokens, passwords — same patterns as `/security-audit secrets`)
   - Check for `CODEOWNERS` file existence
   - Check for Dependabot or Renovate configuration

3. **Monitoring and Logging (CC7.1-CC7.4)**:
   - Check for structured logging implementation (Pino, Winston)
   - Check for error monitoring integration patterns (Sentry, Datadog imports)
   - Check for health check endpoints (`/health`, `/healthz`, `/readyz`)
   - Check for request/response logging middleware

4. **Vulnerability Management (CC3.1-CC3.4)**:
   - Verify `npm audit` is in CI pipeline
   - Check for SAST tools (CodeQL, Semgrep, SonarQube configuration)
   - Check for container scanning if Docker is used (Trivy, Snyk)
   - Verify dependency update automation (Dependabot, Renovate)

5. **Availability (A1.1-A1.3)**:
   - Check for health check endpoint implementations
   - Check for runbook/incident documentation in `docs/`
   - Check for graceful shutdown handling (`SIGTERM`/`SIGINT` handlers)
   - Check for Dockerfile with `HEALTHCHECK`

**Output**: SOC 2 control matrix with status (PASS/FAIL/PARTIAL/NOT APPLICABLE), evidence locations, and gap remediation suggestions.

---

### 4. Generate consolidated report

Output a structured report to the console:

```markdown
# Code Quality & Compliance Audit Report

**Date**: YYYY-MM-DD
**Project**: <name>
**Audits performed**: <list>

## Summary

- Dead code items: N
- PII compliance gaps: N
- SOC 2 controls passing: N/M

## Dead Code Findings

| File | Line | Category | Item | Action |
| ---- | ---- | -------- | ---- | ------ |

## PII Handling Findings

| File | Line | Field | Issue | Severity |
| ---- | ---- | ----- | ----- | -------- |

### PII Data Flow Map

<text-based flow diagram>

## SOC 2 Readiness

| Control | Category | Status | Evidence | Gap |
| ------- | -------- | ------ | -------- | --- |

## Recommendations

1. Prioritized action list
2. ...

---

_Generated by /audit_
```

### 5. Do not modify any files — this is a read-only audit.
