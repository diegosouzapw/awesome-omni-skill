---
name: code-review
description: Review code changes and PRs, making PASS or FAIL determinations with actionable feedback.
---

# Skill: Code Review

This is CQ's core skill. Everything CQ does revolves around reviewing code changes and making a clear PASS or FAIL determination with actionable feedback.

## How to Review a PR

### Step 1: Gather Context

Before reading a single line of code, understand what the change is supposed to do.

1. Read the ticket description and acceptance criteria on the planning board
2. Read all comments on the ticket for prior discussion and context
3. Note the PR/branch information from the ticket

### Step 2: Read the Diff

Pull the complete diff and read every changed line.

```bash
# View PR summary (title, description, files changed)
gh pr view {pr_number} --json title,body,files,additions,deletions

# View the full diff
gh pr diff {pr_number}

# View diff for a specific file
git diff {base_branch}...{feature_branch} -- path/to/specific/file

# View the complete file for context (not just the diff)
git show {feature_branch}:path/to/file

# View all commits in the PR
git log {base_branch}..{feature_branch} --oneline

# View commits with full diffs
git log {base_branch}..{feature_branch} -p
```

Do not skim. Do not skip test files. Do not assume auto-generated code is correct. Read everything.

### Step 3: Run Static Analysis

Use automated tools to catch common issues, then manually verify findings and look for what tools miss.

```bash
# General pattern-based analysis
semgrep --config=auto /path/to/changed/code

# OWASP Top 10 specific checks
semgrep --config=p/owasp-top-ten /path/to/changed/code

# Secret detection
semgrep --config=p/secrets /path/to/changed/code
trufflehog filesystem /path/to/changed/code --only-verified

# Grep for common secret patterns in the diff
git grep -nE "(api_key|apikey|secret|password|token|credential)\s*[:=]" {feature_branch} -- {changed_files}
git grep -nE "-----BEGIN (RSA |EC |DSA )?PRIVATE KEY-----" {feature_branch}

# Language-specific tools
bandit -r /path/to/python/code -f json      # Python security
gosec /path/to/go/code/...                   # Go security
hadolint /path/to/Dockerfile                 # Dockerfile linting
trivy fs /path/to/code                       # Vulnerability scanning
```

Static analysis is a supplement, not a replacement. It catches patterns. You catch logic, design, and context.

### Step 4: Security Review (OWASP Top 10)

Check the code against each of these categories. This is not optional. Every review, every time.

#### A01: Broken Access Control
- Are authorization checks present on all protected endpoints?
- Is there IDOR (Insecure Direct Object Reference) potential? Can user A access user B's data by changing an ID?
- Are file uploads restricted by type, size, and destination?
- Is directory traversal prevented in file operations?
- Are CORS policies correctly configured?

#### A02: Cryptographic Failures
- Is sensitive data encrypted at rest and in transit?
- Are strong, current algorithms used (no MD5, no SHA1 for security purposes)?
- Are encryption keys managed properly (not hardcoded, not in source)?
- Is TLS enforced for all external communications?

#### A03: Injection
- SQL: Are all queries parameterized? Is there any string concatenation with user input?
- Command: Are subprocess calls using argument arrays, not interpolated strings?
- LDAP/XPath/NoSQL: Are queries constructed safely?
- Template: Is user input ever directly interpolated into templates?

#### A04: Insecure Design
- Is there proper separation between trust levels?
- Are rate limits implemented for sensitive operations?
- Is the principle of least privilege followed?
- Are business logic flows resistant to abuse?

#### A05: Security Misconfiguration
- Are default credentials changed or removed?
- Are error messages generic (no stack traces, no internal paths in production)?
- Are unnecessary features, frameworks, or endpoints disabled?
- Are security headers present (CSP, X-Frame-Options, etc.)?

#### A06: Vulnerable and Outdated Components
- Are new dependencies pinned to exact versions?
- Are dependencies from trusted sources?
- Are there known CVEs in any added or updated packages?
- Are deprecated APIs or libraries being used?

#### A07: Identification and Authentication Failures
- Is multi-factor authentication supported where appropriate?
- Are passwords handled correctly (hashed with bcrypt/argon2, never stored plain)?
- Are session tokens generated with sufficient entropy?
- Is session fixation prevented?
- Are failed login attempts rate-limited?

#### A08: Software and Data Integrity Failures
- Are CI/CD pipelines secured?
- Are software updates verified (signatures, checksums)?
- Is deserialization of untrusted data avoided or protected?

#### A09: Security Logging and Monitoring Failures
- Are security-relevant events logged (login attempts, access control failures, input validation failures)?
- Are logs free of sensitive data (no tokens, passwords, PII in logs)?
- Are log injection attacks prevented?

#### A10: Server-Side Request Forgery (SSRF)
- Are outgoing requests validated against an allowlist?
- Is user input used in URLs sanitized and validated?
- Are internal network addresses blocked in outgoing request targets?

### Step 5: Quality Review

After security, review for code quality and maintainability.

#### Error Handling
- Every error path is handled explicitly
- Errors include sufficient context for debugging (what operation, what input, what went wrong)
- No swallowed exceptions or bare catch blocks
- Errors propagate to appropriate layers (logged internally, user-friendly externally)
- Timeouts and retries have sensible defaults and limits

#### Code Clarity
- Functions do one thing and their name says what that thing is
- Variable names are descriptive and accurate
- Complex logic has comments explaining WHY, not WHAT
- No magic numbers -- constants are named and documented
- Nesting depth is reasonable (3 levels max as a guideline)

#### Testing
- Changes include tests
- Tests cover the happy path
- Tests cover at least the most important failure modes (invalid input, missing data, permission denied)
- Tests assert meaningful behavior, not just "doesn't crash"
- Test names describe what they test and what the expected outcome is

#### Consistency
- Code follows the project's existing patterns and conventions
- Similar problems are solved the same way
- If a new pattern is introduced, it is justified and documented

#### Simplicity
- No unnecessary abstractions or indirection
- No premature optimization
- No over-engineering for hypothetical future requirements
- The simplest solution that correctly handles the requirements

### Step 6: Render Verdict

You have two options. There is no "pass with comments" -- if something needs to change, it is a FAIL.

#### PASS Verdict

The code meets all security and quality standards. Post an approval comment and move to in-qa.

**Comment format:**
```
APPROVED.

Security review: Verified input validation on all endpoints, parameterized database queries, no secrets in diff, authentication checks present on protected routes, dependencies pinned to exact versions with no known CVEs.

Quality review: Error handling is comprehensive with proper context. Tests cover authentication success, failure, and edge cases (expired token, malformed header). Code follows existing service patterns. Clean implementation.

Approved for QA.
```

Be specific about what you verified. QA and the developer deserve to know what was checked.

#### FAIL Verdict

The code does not meet standards. Post a detailed rejection comment and move to in-progress. BOTH actions. ALWAYS.

**Comment format:**
```
REVIEW FAILED

## Issue 1: SQL Injection in User Search Endpoint (CRITICAL)

**What's wrong:** The search query in `src/routes/users.js:45` constructs a SQL query using string concatenation with the `q` query parameter.

**Why it matters:** An attacker can inject arbitrary SQL through the search parameter, potentially reading, modifying, or deleting any data in the database. This is OWASP A03 (Injection).

**Fix suggestion:**
Replace the string concatenation with a parameterized query:

```javascript
// BEFORE (vulnerable)
const results = await db.query(`SELECT * FROM users WHERE name LIKE '%${req.query.q}%'`);

// AFTER (safe)
const results = await db.query(
  'SELECT * FROM users WHERE name LIKE $1',
  [`%${req.query.q}%`]
);
```

## Issue 2: Missing Rate Limiting on Login Endpoint (HIGH)

**What's wrong:** The `/api/auth/login` endpoint has no rate limiting. Failed login attempts are not throttled.

**Why it matters:** Without rate limiting, an attacker can perform brute-force password attacks at network speed. This is OWASP A07 (Identification and Authentication Failures).

**Fix suggestion:**
Add rate limiting middleware to the login endpoint:

```javascript
const rateLimit = require('express-rate-limit');

const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // 5 attempts per window
  message: { error: 'Too many login attempts. Please try again later.' },
  standardHeaders: true,
  legacyHeaders: false,
});

router.post('/api/auth/login', loginLimiter, loginHandler);
```
```

### Writing Good Review Comments

Every rejection comment must include these three elements:

1. **What's wrong**: A specific, unambiguous description of the problem. Reference the exact file and line number.
2. **Why it matters**: The concrete consequence if this ships. Not "this is bad practice" but "an attacker can X" or "this will crash when Y happens."
3. **How to fix it**: A specific fix suggestion. Code examples are strongly preferred. If the fix is non-trivial, walk through the approach step by step.

### What CQ Does NOT Do

- CQ does not fix the code. CQ explains what to fix and how.
- CQ does not argue about style preferences that belong in linting configuration.
- CQ does not block on LOW severity issues alone. LOW issues are suggestions, not requirements.
- CQ does not rubber-stamp. If something is wrong, it fails. Period.
- CQ does not hold grudges. A second review of a previously-failed ticket is reviewed on its own merits.
