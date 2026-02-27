---
name: php-security-audit
description: Analyze a PHP web application or codebase for security vulnerabilities and OWASP compliance. Use when the user asks to audit, check, review, or analyze the security, vulnerabilities, OWASP compliance, or hardening of a PHP, Laravel, Kirby, Livewire, or Blade application. Also use when the user mentions "securite", "security", "OWASP", "injection SQL", "XSS", "CSRF", "faille", "vulnerabilite", "pentest", "hardening", "authentication", or "authorization". Specialized for PHP, Laravel, Kirby CMS, Livewire, Blade, Vite, Tailwind CSS, and SQL databases.
---

# PHP Security Review

Audit a PHP web application's security from the codebase, based on the OWASP Top 10 (2021) and PHP/Laravel-specific security best practices. Focuses on what is verifiable in code.

## How to Conduct the Audit

### 1. Explore the Codebase

Before auditing, understand the project:

- Identify the framework (Laravel, Kirby, vanilla PHP) and version
- Locate entry points: routes, controllers, API endpoints, middleware
- Find authentication and authorization logic (guards, policies, gates)
- Locate database queries (Eloquent, Query Builder, raw SQL)
- Check `.env` handling and configuration
- Find file upload logic
- Identify third-party packages (`composer.json`)
- Locate frontend build config (Vite, Tailwind, Livewire)
- Check for scheduled tasks, queue workers, artisan commands

### 2. Load the Checklist

Read `references/checklist.md` in this skill's directory for the full list of practices to check.

### 3. Analyze Each Category

For each category in the checklist, search the codebase for relevant patterns. Use Grep, Glob, and Read tools to find:

- Raw SQL queries without parameter binding (`DB::raw`, `whereRaw`, string concatenation in queries)
- Unescaped output in Blade templates (`{!! !!}` vs `{{ }}`)
- Missing CSRF protection on forms and state-changing routes
- Mass assignment vulnerabilities (missing `$fillable` or `$guarded`)
- Insecure deserialization (`unserialize()` on user input)
- Dangerous PHP functions (`eval`, `exec`, `system`, `shell_exec`, `passthru`)
- File upload without validation (type, size, extension)
- Hardcoded secrets, API keys, passwords in source code
- `.env` file exposure risks
- Missing rate limiting on authentication endpoints
- Insecure session configuration
- Missing security headers (CSP, X-Frame-Options, HSTS)
- Livewire: exposed properties, missing authorization on actions
- Kirby: panel access, content file permissions, API exposure
- Outdated dependencies with known CVEs

### 4. Generate the Report

Produce a structured report with these sections:

```
## Audit Securite PHP - [Project Name]

### Score global
X / Y pratiques conformes (Z%)

### Resume
[2-3 sentences summarizing the main findings]

### Resultats par categorie

#### [Category Name] (OWASP AXX)
| # | Pratique | Statut | Commentaire |
|---|----------|--------|-------------|
| 1 | ...      | OK/KO/NA | ...       |

### Vulnerabilites critiques
[List any findings that could be actively exploited]

### Top priorites
[5-10 most impactful fixes, ordered by risk/effort ratio]

### Recommandations detaillees
[For each KO finding, explain the vulnerability, exploitation scenario, fix, and relevant OWASP category]
```

**Status values:**

- **OK** - Practice is followed
- **KO** - Practice is not followed (include fix + OWASP reference)
- **NA** - Not applicable to this project
- **PARTIAL** - Partially followed (explain what's missing)

### 5. Prioritization

When listing recommendations, prioritize by exploitability:

1. **Critical** - Actively exploitable, data breach risk (SQL injection, auth bypass, RCE)
2. **High** - Exploitable with some effort (XSS stored, CSRF, IDOR, mass assignment)
3. **Medium** - Exploitable under specific conditions (information disclosure, missing headers)
4. **Low** - Defense-in-depth improvements (logging, rate limiting, dependency updates)

### 6. Export the Review Report

Once the audit is complete, save the full analysis as a Markdown document at the root of the audited codebase:

```
/docs/YYMMDD_php-security-audit.md
```

Where `YYMMDD` is the current date (e.g., `260206` for February 6, 2026). Create the `/docs/` directory if it does not exist.

The document must include:

1. The complete audit report (as generated in step 4)
2. An **action plan** section at the end, listing all tasks derived from the findings:
   - Grouped by priority (Critical, High, Medium, Low)
   - Each task broken down into sub-tasks of **2 to 4 hours** (assuming AI-assisted development)
   - Time estimate for each sub-task
   - Total estimated time per priority level

```
### Plan d'action

#### Critique
- [ ] [Tache] — ~Xh (avec IA)
  - [ ] [Sous-tache 1] — ~2h
  - [ ] [Sous-tache 2] — ~3h

#### Haute priorite
...

#### Moyenne priorite
...

#### Basse priorite
...

**Total estime : ~Xh**
```

## Important Notes

- This is a **static code audit**, not a penetration test. Recommend DAST/pentest for full coverage.
- Focus on what is verifiable in the codebase. Flag what needs runtime testing.
- Be specific: point to exact files, lines, and code patterns.
- For Laravel: leverage framework security features (Eloquent, Blade escaping, CSRF middleware, policies).
- For Kirby: check panel config, blueprint permissions, content API exposure.
- For Livewire: check exposed public properties, action authorization, file uploads.
- When a practice cannot be verified from code alone (e.g., WAF configuration), mark as "needs verification".
