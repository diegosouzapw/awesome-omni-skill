---
name: github-workflow
description: Write and edit GitHub Actions workflow files. Use when creating new workflows, editing existing .github/workflows/*.yml files, or setting up CI/CD pipelines. Triggers on requests like "create a workflow", "add GitHub Actions", "set up CI", or "edit the workflow file".
---

# GitHub Workflow

Write clear, minimal GitHub Actions workflow files.

## Naming Guidelines

### Do NOT name

- `run` steps that are self-explanatory from the command itself
- `uses` steps for common, obvious actions like:
  - `actions/checkout`
  - `actions/setup-node`
  - `oven-sh/setup-bun`
  - `pnpm/action-setup`
  - `actions/cache`

### DO name

- `uses` steps with `actions/github-script` (explain what the script does)
- Complex multi-line shell scripts (summarize the purpose)
- Steps where the intent is not obvious from the code

## Examples

### Good

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: oven-sh/setup-bun@v2
      - run: bun install
      - run: bun run build
      - run: bun test

      - name: Post build status to Slack
        uses: actions/github-script@v7
        with:
          script: |
            const webhook = process.env.SLACK_WEBHOOK;
            // ... complex logic
```

### Bad

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
      - name: Setup Bun
        uses: oven-sh/setup-bun@v2
      - name: Install dependencies
        run: bun install
      - name: Build project
        run: bun run build
      - name: Run tests
        run: bun test
```

## Best Practices

- Pin action versions with full SHA or major version tag (`@v4`, not `@main`)
- Use `workflow_dispatch` for manual triggers when useful
- Set appropriate `permissions` to follow least privilege
- Use `concurrency` to cancel redundant runs
- Prefer `${{ github.token }}` over PAT when possible
- Avoid emoji in workflow names and step names
- Use `$GITHUB_STEP_SUMMARY` to output execution results in Markdown format
- Avoid obvious comments; only add comments to explain complex logic

## Step Summary Example

```yaml
- name: Report test results
  run: |
    echo "## Test Results" >> $GITHUB_STEP_SUMMARY
    echo "| Suite | Passed | Failed |" >> $GITHUB_STEP_SUMMARY
    echo "|-------|--------|--------|" >> $GITHUB_STEP_SUMMARY
    echo "| Unit  | 42     | 0      |" >> $GITHUB_STEP_SUMMARY
```
