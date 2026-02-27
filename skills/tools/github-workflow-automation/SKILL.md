---
name: github-workflow-automation
description: Automates GitHub workflows including batch issue creation from JSON/CSV, PR creation with issue linking, pre-PR testing, and WordPress-specific validation (PHP linting, WPCS). Use when creating multiple issues, automating PR workflows, or validating WordPress code before commits.
---

# GitHub Workflow Automation Skill

This skill provides comprehensive automation for GitHub issue management, pull request workflows, and WordPress-specific code validation.

## Core Capabilities

### 1. Batch Issue Creation
Create multiple GitHub issues from structured data sources (JSON or CSV).

**When to use:**
- Converting audit results to issues
- Importing issues from spreadsheets
- Bulk issue creation from templates

**How it works:**
```bash
python scripts/batch_create_issues.py --input issues.json --repo owner/repo
python scripts/batch_create_issues.py --input issues.csv --format csv
```

See `references/batch-issues-guide.md` for detailed JSON/CSV schemas.

### 2. Automated PR Creation
Create pull requests from issues with automatic "Fixes #N" linking.

**When to use:**
- Converting issues to PRs with proper linking
- Automating PR workflow from feature branches
- Ensuring issue closure on PR merge

**How it works:**
```bash
python scripts/create_pr_from_issue.py --issue 123 --base main
python scripts/create_pr_from_issue.py --issue 456 --draft
```

See `references/pr-workflow-guide.md` for best practices.

### 3. Pre-PR Testing
Execute tests before creating pull requests to ensure code quality.

**When to use:**
- Before creating any PR
- As part of CI/CD pipeline
- Validating WordPress code changes

**How it works:**
```bash
bash scripts/run_tests.sh
bash scripts/run_tests.sh --verbose
```

Automatically detects: PHPUnit, pytest, npm test, WordPress test suite.

### 4. WordPress Validation
Run WordPress Coding Standards (PHPCS) checks with optional auto-fix.

**When to use:**
- Before committing WordPress PHP code
- Ensuring WPCS compliance
- Automated code quality gates

**How it works:**
```bash
bash scripts/validate_wordpress.sh
bash scripts/validate_wordpress.sh --fix
bash scripts/validate_wordpress.sh --standard WordPress-Extra
```

See `references/wordpress-standards.md` for configuration details.

## Common Workflows

### Workflow 1: Convert Website Audit to Issues
1. Create CSV/JSON file with audit findings
2. Validate input: `python scripts/validate_input.py --input audit.json`
3. Create issues: `python scripts/batch_create_issues.py --input audit.json`

### Workflow 2: Issue-to-PR with Tests
1. Check GitHub health: `bash scripts/gh_health_check.sh`
2. Run tests: `bash scripts/run_tests.sh`
3. Create PR: `python scripts/create_pr_from_issue.py --issue 123`

### Workflow 3: WordPress Code Contribution
1. Validate code: `bash scripts/validate_wordpress.sh --fix`
2. Run tests: `bash scripts/run_tests.sh`
3. Create PR: `python scripts/create_pr_from_issue.py --issue 456`

## Scripts Reference

All scripts are in the `scripts/` directory:

- **batch_create_issues.py** - Create multiple issues from JSON/CSV
- **create_pr_from_issue.py** - Create PR linked to issue(s)
- **run_tests.sh** - Execute test suites with auto-detection
- **validate_wordpress.sh** - Run PHPCS with WordPress standards
- **validate_input.py** - Validate JSON/CSV before processing
- **gh_health_check.sh** - Verify gh CLI authentication and permissions

## Detailed References

For comprehensive documentation, see the `references/` directory:

- **batch-issues-guide.md** - JSON/CSV schemas and examples
- **wordpress-standards.md** - WPCS configuration and best practices
- **github-actions-integration.md** - CI/CD workflow examples
- **pr-workflow-guide.md** - PR creation best practices
- **error-handling.md** - Common errors and solutions
- **templates/** - Reusable issue and PR templates

## Error Handling

All scripts include comprehensive error handling:

- Input validation before execution
- Clear error messages with suggested actions
- Graceful handling of rate limits
- Partial success support (continue on individual failures)

See `references/error-handling.md` for troubleshooting guidance.

## Prerequisites

Required tools (scripts will check for these):
- `gh` CLI installed and authenticated
- Git configured
- For WordPress validation: `phpcs` with WPCS installed
- For testing: Appropriate test framework (PHPUnit, etc.)

Run health check: `bash scripts/gh_health_check.sh`

## Examples

### Create Issues from JSON
```json
{
  "issues": [
    {
      "title": "Fix contact form",
      "body": "Contact form not submitting...",
      "labels": ["bug", "priority:high"],
      "assignees": ["username"]
    }
  ]
}
```

```bash
python scripts/batch_create_issues.py --input issues.json
```

### Create PR with Multiple Issue Links
```bash
python scripts/create_pr_from_issue.py --issue 123,124,125 --base main
```

### Validate and Auto-Fix WordPress Code
```bash
bash scripts/validate_wordpress.sh --fix --standard WordPress-Extra
```

## Integration Points

This skill integrates with:
- **GitHub CLI** - All GitHub operations
- **GitHub Actions** - See `references/github-actions-integration.md`
- **WordPress Coding Standards** - Code quality enforcement
- **Test Frameworks** - Automated testing before PRs

## Best Practices

1. **Always validate input** before batch operations
2. **Run tests** before creating PRs
3. **Use dry-run mode** to preview changes
4. **Check health** before starting workflows
5. **Follow WordPress standards** for all PHP code
6. **Link PRs to issues** using "Fixes #N" syntax
7. **Review automated PRs** even when tests pass

## Troubleshooting

**Scripts not executable?**
```bash
chmod +x scripts/*.sh
```

**gh CLI not authenticated?**
```bash
gh auth login
```

**PHPCS not found?**
```bash
composer global require wp-coding-standards/wpcs
phpcs --config-set installed_paths ~/.composer/vendor/wp-coding-standards/wpcs
```

For more help, see `references/error-handling.md`.
