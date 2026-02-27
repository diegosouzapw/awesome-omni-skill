---
name: github-issue-triage
description: Analyze GitHub issues for the Nx repository and provide assignment recommendations based on technology stack, team expertise, and priority classification rules.
allowed-tools: Bash, Read, Grep, Glob
---

# GitHub Issue Triage Skill

This skill provides comprehensive logic for triaging GitHub issues in the Nx repository.

## Team Expertise Mapping

### Primary Team Members
- **@barbados-clemens** - Documentation, lifecycle hooks, developer experience, API docs
- **@lourw** - Java/Gradle
- **@leosvelperez** - Angular (primary), TypeScript, testing tools (Jest/Cypress/Playwright), ESLint issues
- **@Coly010** - Webpack/Rollup/Rspack configs, Module Federation, Storybook, Vue, bundler optimization, React Native, Nx Release, Publishing, migration utilities, React, Node/NestJS/Express, NextJS, Remix, create-nx-workspace, preset issues
- **@AgentEnder** - Nx Core (caching/daemon/graph), installation issues, plugin system, devkit, affected calculation

## Assignment Logic (Check in Order)

### 1. Technology Keywords in Title/Body

Look for these keywords and match to assignees:

| Keywords | Assignee | Reason |
|----------|----------|--------|
| "Angular", "ng serve", "@angular" | @leosvelperez | Angular specialist |
| "Nest", "NestJS", "@nx/nest" | @Coly010 | Backend framework support |
| "Next", "NextJS", "@nx/next" | @Coly010 | Framework support |
| "React", "@nx/react" | @Coly010 | Framework support |
| "webpack", "rollup", "rspack", "bundler" | @Coly010 | Bundler expertise |
| "Module Federation", "MF" | @Coly010 | Advanced bundling feature |
| "create-nx-workspace", "preset" | @Coly010 | Workspace setup |
| "cache", "daemon", "affected", "nx reset" | @AgentEnder | Nx core functionality |
| "plugin", "generator", "executor" | @AgentEnder | Plugin system & devkit |
| "docs", "documentation", "lifecycle" | @barbados-clemens | Documentation focus |

### 2. Error Pattern Analysis

Examine error messages and stack traces to identify:

- **Compilation/build failures with Angular** → @leosvelperez
- **Webpack configuration errors** → @Coly010
- **Installation/dependency issues** → @AgentEnder
- **Preset/generator failures** → Check technology first, fallback to @AgentEnder

### 3. Scope Assignment

Match scope labels to technology:

- Use `scope: angular` for Angular-specific issues
- Use `scope: react` for React-specific issues
- Use `scope: node` for Node/NestJS/Express issues
- Use `scope: bundlers` for webpack/rollup/rspack issues
- Use `scope: core` only for caching/daemon/graph issues
- Use `scope: misc` for installation/setup issues
- Use `scope: dx` for documentation/developer experience issues

## Priority Classification Rules

Apply priorities conservatively:

| Priority | Criteria |
|----------|----------|
| **High** | Blocks many users (workspace creation failures, compilation blockers, security CVEs) |
| **Medium** | Standard bugs, configuration issues, generator problems |
| **Low** | Documentation improvements, UI enhancements, edge cases |

## Validation Steps

Before finalizing assignments:

1. **Check issue body** for actual error messages and stack traces
2. **Review reproduction steps** to identify the root problem area
3. **Consider scope** - does this affect a specific framework or Nx core?
4. **Verify priority** - how many users would be blocked?
5. **Cross-check** - does the technology keyword match the actual problem?

## Bulk Operations

### Assignment Command Template

```bash
# Assign single issue
gh issue edit 12345 --repo nrwl/nx --add-assignee username

# Batch assign to same person
gh issue edit 12345 12346 12347 --repo nrwl/nx --add-assignee username

# Batch assign to different people (sequential)
gh issue edit 12345 12346 --repo nrwl/nx --add-assignee user1
gh issue edit 12347 12348 --repo nrwl/nx --add-assignee user2
```

### Label Command Template

```bash
# Apply scope labels
gh issue edit 12345 12346 --repo nrwl/nx --add-label "scope: angular"

# Apply priority labels
gh issue edit 12345 --repo nrwl/nx --add-label "priority: high"
gh issue edit 12346 12347 --repo nrwl/nx --add-label "priority: medium"
```

## Browser Verification (MANDATORY)

After all assignments are complete, open every assigned issue in the browser for manual review.

### Command by OS

**macOS:**
```bash
for issue in ISSUE_NUMBERS; do open "https://github.com/nrwl/nx/issues/$issue"; done
```

**Linux:**
```bash
for issue in ISSUE_NUMBERS; do xdg-open "https://github.com/nrwl/nx/issues/$issue"; done
```

**Windows:**
```bash
for issue in ISSUE_NUMBERS; do start "https://github.com/nrwl/nx/issues/$issue"; done
```

**Alternative (CLI-based, one at a time):**
```bash
gh issue view ISSUE_NUMBER --repo nrwl/nx --web
```

## Workflow Best Practices

1. **Always inspect the full issue** - Don't assign based on title alone
2. **Check comments** - Users often provide context in discussions
3. **Look for duplicate markers** - Sometimes issues reference others
4. **Consider workload** - Don't overload one assignee
5. **Validate assignments** - Browser verification is non-negotiable
6. **Document reasoning** - Leave a comment explaining the assignment if complex

## Assignment Examples

**Example 1: Angular + Build Issue**
- Title: "Angular build fails with Module Federation setup"
- Keywords: "Angular", "Module Federation", "build fails"
- Recommendation: @Coly010 (bundler + framework specialist)
- Scope: `scope: bundlers`
- Priority: `priority: high` (blocks users from building)

**Example 2: Installation Problem**
- Title: "npm install fails with peer dependency conflict"
- Keywords: "install", "dependency"
- Recommendation: @AgentEnder (installation issues)
- Scope: `scope: misc`
- Priority: `priority: high` (blocks workspace setup)

**Example 3: Documentation Request**
- Title: "Add lifecycle hooks documentation"
- Keywords: "docs", "documentation", "lifecycle"
- Recommendation: @barbados-clemens (documentation specialist)
- Scope: `scope: dx`
- Priority: `priority: low` (enhancement, not blocking)
