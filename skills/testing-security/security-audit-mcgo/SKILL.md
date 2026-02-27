---
name: security-audit
description: Comprehensive security audit with findings classification, epic generation, and PDF report
disable-model-invocation: true
---

# Security Audit Skill

You perform a comprehensive security audit of the current codebase. The audit consists of three phases: Analysis, Epic creation, and PDF report generation.

**Scope:** `$ARGUMENTS` (default: `full` – all categories). Possible restrictions: `docker`, `api`, `auth`, `dependencies`, `config`, `network`.

**Language:** Parse `$ARGUMENTS` for a `lang=XX` parameter (e.g. `lang=de`, `lang=fr`). Default language is **English**. If a `lang` parameter is found, produce ALL output (epics, PDF report content, section headings, findings, summary) in the specified language. The `lang` parameter can be combined with a scope, e.g. `/security-audit docker lang=de`.

---

## Phase 1 – Analysis

Examine the codebase systematically across these categories:

### 1.1 Analysis Categories

| Category | What to check |
|----------|---------------|
| **Source Code** | Injection vulnerabilities (SQL, Command, SSRF, XSS), insecure deserialization, missing input validation, hardcoded secrets, insecure crypto |
| **Authentication & Authorization** | Token handling, session management, missing auth checks, token leakage (logs, errors), credential storage |
| **Docker & Containers** | Root user, unnecessary packages, missing security options (read_only, no-new-privileges, cap_drop), secret handling, base image currency |
| **CI/CD Pipeline** | Secret exposure in logs, missing image scans, insecure registry configuration, missing SAST/DAST |
| **Dependencies** | Known CVEs (check package.json/requirements.txt/go.mod etc.), outdated packages, unnecessary dependencies |
| **Configuration** | Missing TLS enforcement, open CORS, missing rate limits, insecure defaults, missing security headers |
| **Network & Transport** | Cleartext transmission, missing timeouts, unlimited body sizes, DNS rebinding |

### 1.2 Procedure

1. Read `CLAUDE.md`, `README.md`, `package.json` (or equivalent) for project overview
2. Search `src/` (or main source directory) recursively
3. Check `Dockerfile`, `docker-compose*.yml`, `.gitlab-ci.yml`, `.github/workflows/`
4. Check configuration files (`.env*`, `config.*`, etc.)
5. Check `package-lock.json`/`yarn.lock`/`go.sum` for known vulnerable versions

### 1.3 Finding Classification

Classify each finding by severity:

| Severity | Criteria | Prefix |
|----------|----------|--------|
| **CRITICAL** | Directly exploitable, Remote Code Execution, credential theft, full compromise | C1, C2, ... |
| **HIGH** | Exploitable with preconditions, significant impact, data loss possible | H1, H2, ... |
| **MEDIUM** | Defense-in-depth gap, best-practice violation with concrete risk | M1, M2, ... |
| **LOW** | Hardening measure, minimal direct impact, improvement suggestion | L1, L2, ... |

For each finding document:
- **ID** and **title**
- **File and line** (where possible)
- **Description** of the problem
- **Current code** (relevant excerpt)
- **Recommended fix** with concrete code suggestion
- **OWASP Top 10** mapping (if applicable)
- **CWE number** (consult the references.md in this skill directory)

### 1.4 Positive Findings

Also document what is already well implemented. Examples:
- Existing input validation
- Correct secret handling
- Security headers present
- Dependency management up to date

---

## Phase 2 – Create or Update Epics

Create or update one epic per severity level with concrete tickets in `docs/epics/`:

### 2.1 Existing Epics – Read First

Before creating or modifying epics, check if `docs/epics/epic-security-*.md` files already exist. If they do:

1. **Read all existing epic files** to understand previously documented findings
2. **Compare** each existing ticket against the current analysis results
3. **Update** each ticket according to these rules:
   - **Fixed issue:** Mark the ticket as `✓ RESOLVED`. Add a `**Resolution:**` section describing what was done. Check off the acceptance criteria. **NEVER delete the ticket.**
   - **Still open (unchanged):** Keep the ticket as-is
   - **Worsened or changed:** Re-evaluate the ticket. Update the description, severity, affected code, and recommended fix. Add a `**Re-evaluation ([Date]):**` note explaining what changed. If severity changed, move the ticket to the appropriate epic file (and leave a cross-reference in the old location)
   - **New finding:** Add as a new ticket with the next available ID
4. **NEVER delete any ticket or finding** – resolved issues serve as audit trail
5. Set the epic **Status** to `closed` when all tickets within it are resolved

### 2.2 Epic Structure

For each severity level (where findings exist) create a file:

- `docs/epics/epic-security-critical.md` – Critical findings
- `docs/epics/epic-security-high.md` – High findings
- `docs/epics/epic-security-medium.md` – Medium findings
- `docs/epics/epic-security-low.md` – Low findings

### 2.3 Epic Format

```markdown
# Epic: Security Hardening – [Severity] Findings

**Status:** open | closed
**Priority:** [CRITICAL/HIGH/MEDIUM/LOW]
**Source:** Security Audit, [Date]

## Description

[1-2 sentence summary]

---

## Tickets

### [Epic-Nr].[Ticket-Nr] – [Title]

**File:** `[Path]`
**Finding:** [ID] – [Short description]

**Current Code ([File]:[Lines]):**
\`\`\`[language]
[code]
\`\`\`

**Required Changes:**

1. [Concrete instruction with code example]
2. [...]

---

### [Epic-Nr].[Ticket-Nr] – [Title] ✓ RESOLVED

**File:** `[Path]`
**Finding:** [ID] – [Short description]

**Resolution:** [What was done to fix the issue]

---

## Acceptance Criteria

- [ ] [Testable criterion 1]
- [x] [Resolved criterion]
```

### 2.4 Rules for Epics

- Every ticket must reference **file and line**
- Every ticket must contain **concrete fix code** (not just "should be fixed")
- Acceptance criteria must be **testable** (e.g. "Request with X returns Y")
- Cross-reference tickets when fixes overlap
- Order within an epic: by effort/impact descending
- **NEVER delete tickets or findings** – they are part of the audit trail
- Resolved tickets stay in the epic with their resolution documented

---

## Phase 3 – Generate PDF Report

Generate a professional security audit report as PDF.

### 3.1 Procedure

1. Read the HTML template from this skill directory: `report-template.html`
2. Read the references from this skill directory: `references.md`
3. Populate the template with the audit results (replace the placeholder comments). Set the `<html lang="...">` attribute to the active language code (e.g. `en`, `de`, `fr`)
4. Create the output directory `docs/security-audit/` if it does not exist
5. Determine the output filename. Use the timestamp format `YYYY-MM-DD-HHmmSS` (current date and time):
   - **First audit** (no existing PDF in `docs/security-audit/`): `[projectname]-security-audit-YYYY-MM-DD.pdf`
   - **Subsequent reviews** (an audit PDF already exists): `[projectname]-security-review-YYYY-MM-DD-HHmmSS.pdf`

   This keeps the original audit as baseline and creates timestamped reviews alongside it for traceability.
6. Write the populated HTML file to `docs/security-audit/[filename].html`
7. Detect the platform and resolve the Chrome binary path:
   - **macOS:** `"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome"`
   - **Linux:** `google-chrome-stable` or `google-chrome` (whichever is found in PATH)
   - **Windows (WSL/Git Bash):** `"/mnt/c/Program Files/Google/Chrome/Application/chrome.exe"` or `"C:\Program Files\Google\Chrome\Application\chrome.exe"`

   If Chrome is not found, inform the user and skip PDF generation (epics and summary are still produced).

   Convert via Chrome headless to PDF:
   ```bash
   <resolved-chrome-path> \
     --headless --disable-gpu --no-sandbox \
     --print-to-pdf="docs/security-audit/[filename].pdf" \
     --print-to-pdf-no-header \
     "docs/security-audit/[filename].html"
   ```
8. Delete the temporary HTML file

### 3.2 Report Content

The report must contain:

1. **Title page** – Project name, audit date, auditor (Claude Code), overall rating
2. **Executive Summary** – 3-5 sentence summary for management
3. **Scoring** – Overall score as rating (A/B+/B/C/D) based on:
   - A: No critical/high findings, max 2 medium
   - B+: No critical, max 2 high, few medium
   - B: No critical, several high
   - C: 1-2 critical or many high
   - D: Multiple critical findings
4. **Findings table** – All findings with ID, severity, title, OWASP/CWE
5. **Detailed findings** – Per finding: description, affected code, recommendation
6. **Positive findings** – What is already well implemented
7. **Risk matrix** – Likelihood vs. impact grid
8. **References** – OWASP, CWE, NIST references (from references.md)

### 3.3 Score Ring

Calculate the score as percentage:

```
Score = 100 - (Critical * 20) - (High * 10) - (Medium * 4) - (Low * 1)
Score = max(0, min(100, Score))
```

Mapping:
- 90-100: A (dark green)
- 75-89: B+ (green)
- 60-74: B (yellow)
- 40-59: C (orange)
- 0-39: D (red)

---

## Output

At the end of the audit show a summary:

```
Security Audit completed.

Result: [Rating] ([Score]/100)
- Critical: [n] findings
- High: [n] findings
- Medium: [n] findings
- Low: [n] findings
- Positive: [n] findings

Generated files:
- docs/epics/epic-security-critical.md (if findings exist)
- docs/epics/epic-security-high.md (if findings exist)
- docs/epics/epic-security-medium.md (if findings exist)
- docs/epics/epic-security-low.md (if findings exist)
- docs/security-audit/[projectname]-security-audit-YYYY-MM-DD.pdf (first audit)
- docs/security-audit/[projectname]-security-review-YYYY-MM-DD-HHmmSS.pdf (subsequent reviews)
```
