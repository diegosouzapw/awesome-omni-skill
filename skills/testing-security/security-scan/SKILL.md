---
name: security-scan
description: Comprehensive security scanning for CVE vulnerabilities, OWASP Top 10 code patterns, and dependency audits. Use when the user wants to check code security, find vulnerabilities, or audit dependencies.
license: MIT
compatibility: Requires semgrep. Optional language-specific tools - npm/yarn/pnpm audit (Node.js), pip-audit (Python), govulncheck (Go), cargo-audit (Rust), trivy (multi-language fallback). Network access needed for NVD API.
metadata:
  author: Ray Tien
  version: "1.0.0"
---

# Security Scan

Comprehensive security scanner combining CVE vulnerability detection, OWASP Top 10 code pattern analysis, and dependency audits.

## Usage

```
/security-scan                           # Full scan
/security-scan --deps-only               # Dependencies only
/security-scan --code-only               # Code patterns only
/security-scan --owasp A03               # Specific OWASP category
/security-scan --severity critical,high  # Filter by severity
/security-scan --auto-remind on|off      # Toggle auto-remind (default: off)
/security-scan --export-bypass           # Export false positive report (DOCX)
/security-scan --export-bypass --pdf     # Export as PDF
/security-scan --export-bypass --template ./template.docx  # Use custom DOCX template
/security-scan --export-bypass --pdf --template ./template.pdf  # Use custom PDF template
```

## Execution Steps

### Step 1: Codebase Detection

Scan project root to identify languages and frameworks:

```
Detection File           -> Language           -> Tool
---------------------------------------------------------
package.json             -> Node.js            -> npm audit
yarn.lock                -> Node.js            -> yarn audit
pnpm-lock.yaml           -> Node.js            -> pnpm audit
requirements.txt         -> Python             -> pip-audit
Pipfile.lock             -> Python             -> pip-audit
pyproject.toml           -> Python             -> pip-audit
go.mod                   -> Go                 -> govulncheck
Cargo.toml               -> Rust               -> cargo audit
composer.json            -> PHP                -> composer audit
Gemfile.lock             -> Ruby               -> bundler-audit
pom.xml                  -> Java/Maven         -> trivy
build.gradle             -> Java/Gradle        -> trivy
Dockerfile               -> Container          -> trivy
```

Run this detection:
```bash
# Detect project types
ls -la package.json yarn.lock pnpm-lock.yaml requirements.txt Pipfile.lock pyproject.toml go.mod Cargo.toml composer.json Gemfile.lock pom.xml build.gradle Dockerfile 2>/dev/null
```

### Step 2: Parallel Subagent Dispatch

Launch these subagents in parallel using Task tool with `run_in_background: true`:

**Subagent 1 - Dependency Scanner:**
```
Scan dependencies for known CVE vulnerabilities.

Based on detected project type, run appropriate commands:

Node.js:
  npm audit --json 2>/dev/null || yarn audit --json 2>/dev/null || pnpm audit --json 2>/dev/null

Python:
  pip-audit --format json 2>/dev/null

Go:
  govulncheck -json ./... 2>/dev/null

Rust:
  cargo audit --json 2>/dev/null

Fallback (any):
  trivy fs --format json --scanners vuln .

Collect all vulnerabilities with: CVE ID, package name, current version, fixed version, severity.
```

**Subagent 2 - OWASP Code Scanner:**
```
Scan code for OWASP Top 10 vulnerabilities using semgrep.

Run:
  semgrep --config "p/owasp-top-ten" --json .

If semgrep not installed, inform user:
  "semgrep required: npm install -g semgrep"

OWASP categories to check:
- A01: Broken Access Control
- A02: Cryptographic Failures
- A03: Injection (SQL, XSS, Command)
- A04: Insecure Design
- A05: Security Misconfiguration
- A06: Vulnerable Components
- A07: Auth Failures
- A08: Data Integrity Failures
- A09: Logging Failures
- A10: SSRF

Collect all findings with: OWASP category, file path, line number, code snippet, severity.
```

### Step 3: CVE Enrichment

After Subagent 1 and 2 complete, query NVD API for detailed CVE information:

```bash
# For each CVE found, query NVD API
curl -s "https://services.nvd.nist.gov/rest/json/cves/2.0?cveId=CVE-XXXX-YYYY"
```

Extract:
- CVSS score and severity
- Description
- References
- Fix recommendations

Rate limit: 5 requests per 30 seconds (no API key)

### Step 4: Generate Reports

**CLI Summary Output:**
```
Security Scan Results
=====================

Scanned: [detected languages]
Duration: [time]

CRITICAL  X   [!!] Immediate action required
HIGH      X   [!]  Fix soon
MEDIUM    X   [ ]  Review recommended
LOW       X   [ ]  Minor issues

Top Issues:
-------------------------------------------------------------
[SEVERITY] CVE-XXXX-YYYY - description
           Package: name@version -> Fix: upgrade to X.X.X
           OWASP: category

[SEVERITY] OWASP-Category - issue description
           File: path/to/file.ts:line
           Code: `code snippet`
-------------------------------------------------------------

Full report: ./security-report.md
```

**Generate security-report.md:**

Write detailed Markdown report with:
1. Summary table (severity counts)
2. OWASP Top 10 coverage status
3. Dependency vulnerabilities (grouped by severity)
4. Code vulnerabilities (grouped by OWASP category)
5. Remediation recommendations

### Step 5: Handle Missing Tools

If a required tool is not installed, provide installation command:

```bash
# semgrep (required)
npm install -g semgrep

# pip-audit (Python)
pip install pip-audit

# govulncheck (Go)
go install golang.org/x/vuln/cmd/govulncheck@latest

# cargo-audit (Rust)
cargo install cargo-audit

# trivy (fallback)
brew install trivy  # macOS
# or
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin
```

## Auto-Remind Feature (Default: OFF)

When enabled with `--auto-remind on`, store setting and remind user:
- Before git commit
- When package.json/requirements.txt/etc. changes detected
- When files in auth/, db/, api/ directories are modified

Check setting:
```bash
cat ~/.claude/settings/security-scan-auto-remind 2>/dev/null || echo "off"
```

Toggle setting:
```bash
echo "on" > ~/.claude/settings/security-scan-auto-remind   # enable
echo "off" > ~/.claude/settings/security-scan-auto-remind  # disable
```

## False Positive Report (Bypass Documentation)

### Workflow

1. Run `/security-scan` to get scan results
2. User reviews results and discusses with Claude which items are false positives
3. User provides reason for each false positive
4. Run `/security-scan --export-bypass` to generate DOCX report

### During Review

When user identifies a false positive, record it in memory:
- Vulnerability ID (CVE or OWASP rule)
- File location (if applicable)
- Severity
- User-provided reason for false positive

Example conversation:
```
User: "CVE-2024-1234 is a false positive, we don't use the affected merge function"
Claude: "Noted. CVE-2024-1234 marked as false positive. Reason: does not use affected merge function"

User: "The SQL injection in src/db.ts:45 is safe because id comes from internal system"
Claude: "Noted. A03-Injection in src/db.ts:45 marked as false positive. Reason: id from internal system, validated as integer"
```

### Export Report

When user runs `--export-bypass`, generate report using existing skills:

**Option 1: DOCX (recommended)**
Use the `docx` skill from https://github.com/anthropics/skills/tree/main/skills/docx

**Option 2: PDF**
Use the `pdf` skill from https://github.com/anthropics/skills/tree/main/skills/pdf

### Report Content Structure

```
# Security Scan - False Positive Report

**Scan Date:** [date]
**Project:** [project name from package.json or directory name]
**Scope:** [scanned languages and tools used]

---

## False Positives

### 1. [CVE-ID or OWASP Rule] - [Title]

| Field | Value |
|-------|-------|
| Type | Dependency / Code Pattern |
| File | [file path if applicable] |
| Severity | [CRITICAL/HIGH/MEDIUM/LOW] |
| Original Finding | [brief description] |

**False Positive Reason:**
[User-provided reason]

---

### 2. [Next item...]
```

### Output

- DOCX: `./false-positive-report.docx`
- PDF: `./false-positive-report.pdf`

### Report Style

- Title: "Security Scan - False Positive Report"
- Table format for structured data
- Clear separation between items
- Professional formatting suitable for audit/compliance

### Custom Template (Optional)

User can provide their own DOCX/PDF template with `--template` flag.

**Template placeholders:**
```
{{scan_date}}        - Scan date (YYYY-MM-DD)
{{project_name}}     - Project name
{{scope}}            - Scanned languages/tools
{{false_positives}}  - List of false positive items
```

**Each false positive item:**
```
{{item.id}}          - CVE ID or OWASP rule
{{item.title}}       - Vulnerability title
{{item.type}}        - Dependency / Code Pattern
{{item.file}}        - File path (if applicable)
{{item.severity}}    - CRITICAL/HIGH/MEDIUM/LOW
{{item.finding}}     - Original finding description
{{item.reason}}      - User-provided false positive reason
```

If no template provided, use default formatting from docx/pdf skill.

## OWASP Top 10 Reference (2025)

See [references/OWASP.md](references/OWASP.md) for detailed information.

| Code | Category                    | Common Issues                          |
|------|-----------------------------|----------------------------------------|
| A01  | Broken Access Control       | Missing auth checks, IDOR              |
| A02  | Security Misconfiguration   | Debug enabled, default credentials     |
| A03  | Software Supply Chain       | Vulnerable dependencies, typosquatting |
| A04  | Cryptographic Failures      | Weak encryption, hardcoded secrets     |
| A05  | Injection                   | SQL injection, XSS, command injection  |
| A06  | Insecure Design             | Missing rate limits, trust boundaries  |
| A07  | Authentication Failures     | Weak passwords, session fixation       |
| A08  | Integrity Failures          | Insecure deserialization, unsigned JWTs|
| A09  | Logging & Alerting Failures | Missing audit logs, no alerting        |
| A10  | Exceptional Conditions      | Fail-open errors, unhandled exceptions |

## Example Output

```
Security Scan Results
=====================

Scanned: Node.js (npm)
Duration: 8.2s

CRITICAL  1   [!!] Immediate action required
HIGH      3   [!]  Fix soon
MEDIUM    5   [ ]  Review recommended
LOW       2   [ ]  Minor issues

Top Issues:
-------------------------------------------------------------
[CRITICAL] CVE-2024-1234 - Prototype pollution in lodash
           Package: lodash@4.17.20 -> Fix: upgrade to 4.17.21
           OWASP: A03 Injection

[HIGH] A03-Injection - SQL Injection detected
       File: src/db/users.ts:45
       Code: `db.query(\`SELECT * FROM users WHERE id = ${id}\`)`

[HIGH] A02-Crypto - Hardcoded secret detected
       File: src/config/auth.ts:12
       Code: `const SECRET = "hardcoded-secret-key"`
-------------------------------------------------------------

Full report: ./security-report.md
```
