---
name: code-review-assistant
description: Comprehensive code review assistant that analyzes code for security vulnerabilities, performance issues, and code quality. Use when reviewing pull requests, conducting code audits, or analyzing code changes. Supports Python, JavaScript/TypeScript, and general code patterns. Includes automated analysis scripts and structured checklists.
allowed-tools: Read, Grep, Glob, Bash
---

# Code Review Assistant

Perform structured code reviews using checklists and automated analysis tools.

## Review Workflow

1. **Gather context** - Understand the scope of changes
2. **Run automated analysis** - Execute scripts for metrics and security scans
3. **Apply checklists** - Review using category-specific checklists
4. **Synthesize findings** - Compile issues with severity and recommendations

## Quick Start

For a standard code review:

```bash
# 1. View changes
git diff HEAD~1

# 2. Analyze code complexity and metrics
python scripts/analyze.py <file_or_directory>

# 3. Scan for security patterns (optional)
python scripts/security_scan.py <file_or_directory>
```

Then apply the appropriate checklists based on the code type.

## Automated Analysis

### Code Metrics Analysis

Run `scripts/analyze.py` to get code metrics:

```bash
python scripts/analyze.py path/to/code --output json
python scripts/analyze.py src/ --recursive
```

Outputs:
- Lines of code (total, code, comments, blank)
- Function/method count and average length
- Cyclomatic complexity estimates
- File-level metrics summary

### Security Pattern Scan

Run `scripts/security_scan.py` for quick security checks:

```bash
python scripts/security_scan.py path/to/code
python scripts/security_scan.py src/ --severity high
```

Detects:
- Dangerous function calls (eval, exec, shell injection)
- Hardcoded credentials patterns
- SQL injection indicators
- XSS vulnerability patterns

## Review Checklists

Select checklists based on the type of changes being reviewed:

### Security Review
**When to use**: Authentication changes, user input handling, API endpoints, database queries

See [SECURITY.md](references/SECURITY.md) for complete security checklist covering:
- Injection vulnerabilities (SQL, XSS, command injection)
- Authentication and authorization
- Data exposure and encryption
- Input validation

### Performance Review
**When to use**: Database operations, loops, API calls, data processing

See [PERFORMANCE.md](references/PERFORMANCE.md) for performance checklist covering:
- N+1 query detection
- Memory management
- Algorithmic complexity
- Caching opportunities

### Code Quality Review
**When to use**: All code changes, especially new features and refactoring

See [QUALITY.md](references/QUALITY.md) for quality checklist covering:
- Naming conventions
- Function complexity
- DRY principle adherence
- Error handling patterns

## Review Output Format

Structure findings using this format:

```markdown
## Code Review Summary

**Files reviewed**: [count]
**Issues found**: Critical: X | High: Y | Medium: Z | Low: W

### Critical Issues
1. **[File:Line]** Description
   - Code: `snippet`
   - Fix: Recommendation

### High Priority Issues
[Same format]

### Positive Observations
- [Note well-implemented patterns]

### Recommendations
1. [Prioritized action items]
```
