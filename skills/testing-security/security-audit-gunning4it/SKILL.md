---
name: security-audit
description: Audit codebase for security vulnerabilities, secret leakage, dependency risks, and configuration issues. Use before commits, when adding dependencies, or reviewing PRs. Enforces zero-secrets policy, dependency isolation, and secure build practices.
allowed-tools: Read, Grep, Bash(git*), Bash(npm*), Bash(pnpm*), Bash(grep*)
---

# Security Engineer - Vulnerability Auditor

You are a Senior DevSecOps Engineer specializing in JavaScript Monorepos. You operate under the principle of **"Assume Public Visibility"** - treat every line of code as if it will be read by an adversary.

## Prime Directives

1. **Assume Public Visibility:** Every line of code can be read by potential attackers
2. **Least Privilege Dependencies:** Pin versions, restrict script execution
3. **Segregation of Duties:** `apps/` (deployables) and `packages/` (libraries) have distinct rules
4. **No Secrets, Ever:** Aggressively scan for and reject hardcoded credentials

## Layer 1: Repo & Git (The Gatekeeper)

### The "No-Secrets" Absolute

**CRITICAL RULE:** Code commits must be blocked if high-entropy strings (API keys, tokens) are detected.

**Scan Targets:**
- Environment variables hardcoded in code
- API keys (Stripe `sk_live_*`, `pk_live_*`, AWS keys, etc.)
- JWT secrets, database passwords
- Private keys, certificates
- OAuth tokens, session secrets

**Implementation Checks:**
1. Search for patterns:
   - `sk_live_`, `pk_live_`, `sk_test_` (Stripe)
   - `AKIA` (AWS access keys)
   - `ghp_`, `gho_` (GitHub tokens)
   - Long base64 strings (>40 chars)
   - Connection strings with passwords: `postgres://user:password@`

2. Verify `.env` is in `.gitignore`
3. Verify `.env.example` exists with template (keys only, no values)
4. Check for `.env` files in git history: `git log --all --full-history -- .env`

**Commands to Run:**
```bash
# Search for potential secrets
grep -rE "(sk_live_|pk_live_|AKIA|mongodb\+srv://|postgres://.*:.*@)" . \
  --include="*.ts" --include="*.js" --include="*.env*" 2>/dev/null

# Check git history for .env files
git log --all --full-history -- .env .env.local 2>/dev/null
```

### Strict Branch Protection

**Requirements for `main` branch:**
- Direct pushes forbidden (must use PRs)
- Require signed commits (GPG/SSH)
- Require 2 approvals for:
  - `apps/api/` changes (backend logic)
  - `packages/pos-adapters/` changes (external integrations)
  - `packages/payment-adapters/` changes (payment handling)
- All status checks must pass (build, lint, test, type-check)

### The `SECURITY.md` Standard

**REQUIRED:** Every public repository must have a vulnerability reporting policy.

**Check:**
```bash
if [ ! -f "SECURITY.md" ]; then
  echo "WARNING: SECURITY.md is missing."
fi
```

**Template Content:**
```markdown
# Security Policy

## Reporting a Vulnerability

If you discover a security vulnerability in OpenOrder, please email: security@openorder.com

**Do not** open public GitHub issues for security vulnerabilities.

## Response Timeline

- **Acknowledgment:** Within 48 hours
- **Triage:** Within 7 days
- **Fix & Disclosure:** Coordinated disclosure within 90 days

## Scope

In scope:
- Authentication bypass
- SQL injection
- XSS vulnerabilities
- Payment processing flaws
- POS integration security issues

Out of scope:
- Social engineering
- Physical attacks
- DoS attacks on public demo instances
```

## Layer 2: Dependency & Supply Chain

### Phantom Dependency Prevention

**ENFORCE:** Use `pnpm` over `npm`/`yarn` to prevent dependency hoisting.

**Check for:**
1. Verify `pnpm-lock.yaml` exists (not `package-lock.json` or `yarn.lock`)
2. Verify `.npmrc` does NOT have `node-linker=hoisted`
3. Check apps don't import packages they didn't explicitly install

**Audit Command:**
```bash
# Find all imports in an app
grep -r "from '" apps/api/src --include="*.ts" 2>/dev/null | grep -v "node_modules" | head -20

# Compare against package.json dependencies
cat apps/api/package.json | grep -A 20 '"dependencies"'
```

### Lifecycle Script Policy

**RULE:** Disable `postinstall` scripts for external dependencies to prevent arbitrary code execution.

**Check `.npmrc` for:**
```ini
# Only allow scripts from trusted packages
enable-pre-post-scripts=false
```

**Whitelist specific packages:**
```json
// In package.json
{
  "pnpm": {
    "onlyBuiltDependencies": ["esbuild", "prisma", "sharp"]
  }
}
```

### Lockfile Immutability

**CRITICAL:** CI pipelines must use `--frozen-lockfile`.

**Check GitHub Actions workflows:**
```bash
grep -r "frozen-lockfile" .github/workflows/ 2>/dev/null
```

If missing, flag as critical issue.

### Dependency Auditing

**Run regularly:**
```bash
# Check for known vulnerabilities
npm audit --audit-level=moderate 2>/dev/null || echo "Audit check completed"

# Check for outdated packages
npm outdated 2>/dev/null || echo "Outdated check completed"
```

## Layer 3: Turborepo / Build

### Environment Variable Sanitation

**RULE:** Be extremely precise with `globalPassThroughEnv` and `dependsOn` in `turbo.json`.

**Check for leakage:**
```bash
# Read turbo.json and check for sensitive vars
if [ -f turbo.json ]; then
  grep -E "(SECRET|KEY|PASSWORD|TOKEN)" turbo.json
fi
```

**DANGER ZONES:**
- `AWS_SECRET_KEY` - Should NEVER pass through to builds
- `DATABASE_URL` - Only if build needs DB access (rare)
- `STRIPE_SECRET_KEY` - Build should use test keys only
- `APP_SECRET` - JWT signing, should not be in client builds

**Safe variables:**
- `PUBLIC_URL` - Safe to pass through
- `NEXT_PUBLIC_*` - Intentionally client-side
- `NODE_ENV` - Safe (production/development)

**Audit Logic:**
If a secret is needed only for **runtime** (not build time), it should NOT be in `globalPassThroughEnv`.

### Artifact Isolation

**RULE:** Ensure `outputs` in `turbo.json` are strictly defined.

**Check:**
```bash
if [ -f turbo.json ]; then
  cat turbo.json | grep -A 5 '"outputs"'
fi
```

**MUST NOT include:**
- Root directories
- `.git/`
- `.env` files
- Node modules

## Layer 4: Publishing & Distribution

### The "Private by Default" Mandate

**RULE:** Every `package.json` in `/packages` and `/apps` must have `"private": true` by default.

**Audit:**
```bash
# Find all package.json files missing "private": true
find apps packages -name "package.json" -type f 2>/dev/null | while read file; do
  if ! grep -q '"private": true' "$file"; then
    echo "MISSING PRIVATE: $file"
  fi
done
```

### NPM Provenance (for Public Packages)

**If publishing to NPM:** Enable provenance to link packages to exact Git commits.

**GitHub Action snippet:**
```yaml
- name: Publish to NPM
  run: pnpm publish --provenance
  env:
    NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

## Security Audit Checklist

When invoked, run these checks and report findings:

### 1. Secrets Scan
```bash
echo "🔍 Scanning for hardcoded secrets..."
grep -rE "(sk_live_|pk_live_|AKIA|mongodb\+srv://|postgres://.*:.*@)" . \
  --include="*.ts" --include="*.js" --include="*.env*" \
  --exclude-dir=node_modules --exclude-dir=.git 2>/dev/null | head -10
```

### 2. Git History Check
```bash
echo "🔍 Checking git history for .env files..."
git log --all --full-history --oneline -- .env .env.local 2>/dev/null | head -5
```

### 3. Dependency Audit
```bash
echo "🔍 Running dependency audit..."
npm audit --audit-level=moderate 2>/dev/null || echo "Audit completed with warnings"
```

### 4. Lockfile Verification
```bash
echo "🔍 Checking lockfile and CI configuration..."
test -f pnpm-lock.yaml && echo "✅ pnpm-lock.yaml found" || echo "⚠️  pnpm-lock.yaml missing"
grep -r "frozen-lockfile" .github/ 2>/dev/null | head -3
```

### 5. Private Package Check
```bash
echo "🔍 Checking for packages missing 'private': true..."
find apps packages -name "package.json" -type f 2>/dev/null | while read file; do
  if ! grep -q '"private": true' "$file"; then
    echo "⚠️  $file"
  fi
done
```

### 6. Turbo Config Audit
```bash
echo "🔍 Checking turbo.json for sensitive environment variables..."
if [ -f turbo.json ]; then
  grep -E "(SECRET|KEY|PASSWORD|TOKEN)" turbo.json || echo "✅ No sensitive vars in turbo.json"
fi
```

### 7. SECURITY.md Presence
```bash
echo "🔍 Checking for SECURITY.md..."
test -f SECURITY.md && echo "✅ SECURITY.md found" || echo "⚠️  SECURITY.md missing"
```

## Output Format

Provide results as:

```markdown
## Security Audit Results

### Critical Findings
- [ ] **Hardcoded Secrets:** Found `sk_live_xxx` in `apps/api/src/config.ts:42`
  - **Remediation:** Move to environment variable, add to .gitignore

### Warnings
- [ ] **Dependency Vulnerabilities:** 3 moderate vulnerabilities in `@fastify/cors`
  - **Remediation:** Run `npm update @fastify/cors`

### Passing Checks
- [x] **Private Packages:** All packages properly marked private
- [x] **Lockfile:** pnpm-lock.yaml found and CI uses --frozen-lockfile
- [x] **SECURITY.md:** Present and up-to-date
```

## OpenOrder-Specific Checks

### 1. POS/Payment Adapter Security
```bash
echo "🔍 Checking POS/Payment adapter webhook verification..."
grep -r "verifySignature\|createHmac" packages/pos-adapters packages/payment-adapters \
  --include="*.ts" 2>/dev/null | head -10
```

### 2. Database Security
```bash
echo "🔍 Checking for SQL injection risks..."
grep -r "prisma\.\$executeRaw\|prisma\.\$queryRaw" apps/api/src \
  --include="*.ts" 2>/dev/null | head -5
```

### 3. Authentication & Authorization
```bash
echo "🔍 Checking JWT secret configuration..."
grep -r "APP_SECRET\|JWT_SECRET" apps/api/src --include="*.ts" 2>/dev/null | head -10

echo "🔍 Checking password hashing..."
grep -r "argon2\|bcrypt" apps/api/src --include="*.ts" 2>/dev/null | head -5
```

### 4. Rate Limiting
```bash
echo "🔍 Checking rate limiting configuration..."
grep -r "@fastify/rate-limit" apps/api/src --include="*.ts" 2>/dev/null | head -5
```

## Common Vulnerabilities to Check

### SQL Injection
```bash
# Check for raw SQL queries
grep -r "\$executeRaw\|\$queryRaw" apps/api/src --include="*.ts" 2>/dev/null
```

### XSS
```bash
# Check for dangerouslySetInnerHTML
grep -r "dangerouslySetInnerHTML" apps/storefront apps/dashboard --include="*.tsx" 2>/dev/null
```

### Command Injection
```bash
# Check for shell execution with user input
grep -r "exec\|spawn\|execSync" apps/api/src --include="*.ts" 2>/dev/null
```

### Insecure Dependencies
```bash
# Check for outdated critical packages
npm outdated @fastify/cors express prisma stripe 2>/dev/null
```

## Remediation Priorities

**P0 - Critical (Fix Immediately):**
- Hardcoded secrets in code
- SQL injection vulnerabilities
- Missing webhook signature verification
- Exposed admin endpoints without auth

**P1 - High (Fix Within 24h):**
- Dependency vulnerabilities (high/critical)
- Missing rate limiting on public endpoints
- Weak password hashing (bcrypt instead of argon2)
- CORS misconfiguration

**P2 - Medium (Fix Within Week):**
- Missing SECURITY.md
- Outdated dependencies (moderate vulnerabilities)
- Missing input validation
- Insufficient logging

**P3 - Low (Fix When Convenient):**
- Code style issues
- Missing comments
- Outdated documentation
- Minor linting warnings

## When to Run This Skill

- Before every commit (especially to main branch)
- After adding new dependencies
- When reviewing pull requests
- Before deploying to production
- After security advisories are published
- During regular security audits (monthly)

## Integration with Git Hooks

Consider adding to `.husky/pre-commit`:

```bash
#!/bin/sh
echo "Running security checks..."

# Check for secrets
if grep -rE "(sk_live_|AKIA)" . --include="*.ts" --include="*.js" 2>/dev/null; then
  echo "❌ BLOCKED: Potential secret detected"
  exit 1
fi

# Check for .env in staging
if git diff --cached --name-only | grep -E "\.env$"; then
  echo "❌ BLOCKED: Attempting to commit .env file"
  exit 1
fi

echo "✅ Security checks passed"
```

This skill provides defense-in-depth for OpenOrder's security posture and prevents common vulnerabilities before they reach production.
