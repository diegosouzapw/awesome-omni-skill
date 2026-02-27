---
name: headless-automation
description: Run Claude Code in CI/CD pipelines, pre-commit hooks, and batch processing. Covers -p flag, fan-out migrations, and pipeline patterns.
---

# Headless Automation Skill

## Trigger
Use when you want Claude to work autonomously in CI/CD pipelines, pre-commit hooks, batch processing, or any automated workflow.

## The Insight
From Anthropic's Claude Code best practices: "Use `-p` flag with prompts for CI/CD, pre-commit hooks, and infrastructure. Employ 'fanning out' for large migrations or 'pipelining' for data processing workflows."

## Basic Headless Usage

### The `-p` Flag
Run Claude with a prompt, get output, exit:

```bash
# Simple prompt
claude -p "Explain what this function does" < src/utils.ts

# With file context
claude -p "Review this PR for security issues" --files $(git diff --name-only main)

# Output to file
claude -p "Generate API documentation" > docs/api.md
```

### Print Mode (`--print`)
Get raw output without interactive formatting:

```bash
claude -p "List all TODO comments" --print
```

## Automation Patterns

### Pattern 1: CI/CD Integration

**GitHub Actions Example:**
```yaml
name: AI Code Review
on: [pull_request]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run Claude Review
        run: |
          claude -p "Review the changes in this PR. Focus on:
          - Security vulnerabilities
          - Performance issues
          - Code style violations

          Output as GitHub PR comment markdown." \
          --files $(git diff --name-only origin/main)
```

**Pre-commit Hook:**
```bash
#!/bin/bash
# .git/hooks/pre-commit

# Get staged files
FILES=$(git diff --cached --name-only --diff-filter=ACM | grep -E '\.(ts|tsx|js|jsx)$')

if [ -n "$FILES" ]; then
  claude -p "Check these files for obvious bugs or issues.
  Output only if problems found, otherwise output nothing." \
  --files $FILES

  if [ $? -ne 0 ]; then
    echo "Claude found issues. Please review."
    exit 1
  fi
fi
```

### Pattern 2: Fan-Out for Large Migrations
Process many items in parallel:

```bash
#!/bin/bash
# migrate-all.sh

# Step 1: Generate task list
claude -p "List all files that need migration from API v1 to v2.
Output as plain file paths, one per line." --print > migration-tasks.txt

# Step 2: Process in parallel
cat migration-tasks.txt | xargs -P 4 -I {} bash -c '
  claude -p "Migrate this file from API v1 to v2.
  Apply changes directly." --files {}
'

# Step 3: Verify
claude -p "Verify all migrations completed successfully.
Check for any remaining v1 API usage."
```

### Pattern 3: Pipeline Processing
Chain Claude operations:

```bash
#!/bin/bash
# data-pipeline.sh

# Stage 1: Extract
claude -p "Extract all API endpoint definitions from src/api/" \
  --print > /tmp/endpoints.json

# Stage 2: Transform
claude -p "Convert these endpoints to OpenAPI 3.0 format" \
  < /tmp/endpoints.json > /tmp/openapi.json

# Stage 3: Generate
claude -p "Generate TypeScript client from this OpenAPI spec" \
  < /tmp/openapi.json > src/generated/api-client.ts
```

### Pattern 4: Batch Processing

```bash
#!/bin/bash
# process-batch.sh

# Process each item in a list
while IFS= read -r item; do
  claude -p "Process: $item" --print >> results.txt
done < items.txt
```

### Pattern 5: Scheduled Tasks

**Cron job for daily reports:**
```bash
# crontab -e
0 9 * * * cd /path/to/project && claude -p "Generate daily code health report" > reports/$(date +%Y-%m-%d).md
```

## Safe Automation Practices

### Use Containers for Risky Operations
```bash
# Run in isolated Docker container
docker run --rm -v $(pwd):/workspace \
  claude-code -p "Refactor all deprecated API calls" \
  --dangerously-skip-permissions
```

### Limit Scope
```bash
# Only touch specific directories
claude -p "Update imports" --files src/components/*.tsx
```

### Dry Run First
```bash
# Preview changes without applying
claude -p "Show what changes would be made to migrate to React 18.
Don't make any changes, just list them."
```

### Capture Output for Review
```bash
# Log all output
claude -p "Apply linting fixes" 2>&1 | tee automation.log
```

## Useful Flags for Automation

| Flag | Purpose |
|------|---------|
| `-p "prompt"` | Run with prompt, non-interactive |
| `--print` | Raw output, no formatting |
| `--files` | Specify files to include |
| `--dangerously-skip-permissions` | Skip permission prompts (use in containers) |
| `--output-format json` | JSON output for parsing |

## Template: Migration Script

```bash
#!/bin/bash
set -e

TASK="Migrate from moment.js to date-fns"
PATTERN="*.ts *.tsx"

echo "=== Starting: $TASK ==="

# 1. Discover affected files
echo "Finding affected files..."
FILES=$(claude -p "List files using moment.js" --print)
echo "Found $(echo "$FILES" | wc -l) files"

# 2. Process each file
echo "$FILES" | while read -r file; do
  echo "Processing: $file"
  claude -p "Migrate $file from moment.js to date-fns.
  Preserve all functionality." --files "$file"
done

# 3. Verify
echo "Verifying migration..."
claude -p "Check for any remaining moment.js usage"

# 4. Run tests
echo "Running tests..."
npm test

echo "=== Complete: $TASK ==="
```
