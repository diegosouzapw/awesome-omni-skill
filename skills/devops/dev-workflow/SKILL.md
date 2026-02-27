---
name: dev-workflow
description: Developer productivity workflows - PR review, CI monitoring, deploy tracking, code metrics
allowed-tools: Read, Write, Edit, Bash, Grep, Glob, WebFetch
---

<Purpose>
Automate common developer workflows that span multiple tools and data sources -- GitHub PRs, CI pipelines, error tracking, code metrics, and system health. This skill gathers information from disparate sources in parallel, cross-references it (which commits caused which errors, which PRs affect which issues), generates actionable developer reports, and sends critical alerts. It replaces the manual morning routine of checking GitHub, CI, Sentry, and Slack.
</Purpose>

<Use_When>
- User says "check PRs", "CI status", "what's the deploy status?"
- User asks "code metrics", "developer report", "project health"
- User says "what happened while I was away?", "catch me up", "morning brief"
- User wants to review pull requests with context from CI and error tracking
- User asks "which commits broke the build?", "what errors are new?"
- User wants to track deployment status or release readiness
- User says "dev report", "status report", "weekly summary"
</Use_When>

<Do_Not_Use_When>
- Pure code editing or implementation -- use executor agents or `ralph` instead
- Git operations only (commit, branch, merge) -- use `git-master` skill instead
- Deep code analysis or architecture review -- use `analyze` skill instead
- Debugging a specific bug -- use `analyze` or delegate to architect agent
- Writing documentation -- use writer agent instead
</Do_Not_Use_When>

<Why_This_Exists>
Developers waste 30-60 minutes daily checking multiple tools for status updates. GitHub notifications pile up, CI failures are missed, errors go unnoticed until users report them, and nobody knows which deploy is in production. This skill consolidates all developer-relevant information into a single workflow, cross-references data across tools, and surfaces what matters -- critical failures, blocking PRs, new errors, and deployment state.
</Why_This_Exists>

<Execution_Policy>
- Gather data from all sources in parallel (GitHub, git, build, error tracking)
- Cross-reference data to surface actionable insights, not raw dumps
- Prioritize critical items: build failures > new errors > blocking PRs > informational
- Store reports in memory for trend tracking over time
- Send critical alerts to Telegram only for actionable issues
- Default model routing: data gathering at haiku tier, cross-referencing at sonnet tier
- Respect rate limits on external APIs (GitHub, Sentry)
</Execution_Policy>

<Steps>
1. **Collect Data in Parallel**: Gather from multiple sources simultaneously

   **GitHub** (via gh CLI):
   ```bash
   # Open PRs
   gh pr list --state open --json number,title,author,createdAt,reviewDecision,statusCheckRollup
   # Recent merged PRs
   gh pr list --state merged --json number,title,mergedAt --limit 10
   # Open issues assigned to user
   gh issue list --assignee @me --state open --json number,title,labels
   # CI status on current branch
   gh run list --limit 5 --json status,conclusion,name,createdAt
   ```

   **Git** (local state):
   ```bash
   # Recent commits
   git log --oneline -20 --format="%h %s (%an, %ar)"
   # Branch status
   git branch -vv
   # Uncommitted changes
   git status --short
   ```

   **Build/Tests** (project-specific):
   ```bash
   # Run build check
   npm run build 2>&1 | tail -20  # or equivalent
   # Run test suite
   npm test 2>&1 | tail -30  # or equivalent
   ```

   **Error Tracking** (if Sentry MCP available):
   - Recent unresolved errors
   - Error frequency trends
   - Errors correlated with recent deploys

2. **Cross-Reference and Analyze**: Connect data across sources
   - Which commits are in which PRs?
   - Which CI failures correspond to which commits?
   - Which errors appeared after which deployments?
   - Which PRs are blocked by failing CI checks?
   - Which issues are addressed by open PRs?

3. **Generate Report**: Format developer status report
   ```markdown
   # Developer Report - YYYY-MM-DD

   ## Critical (Action Required)
   - CI FAILING: main branch build broken since commit abc123 (2h ago)
   - ERROR SPIKE: 500 errors up 300% in auth service since deploy v1.2.3

   ## PRs Requiring Attention
   | PR | Title | Author | Status | CI | Age |
   |----|-------|--------|--------|-----|-----|
   | #42 | Fix auth timeout | @alice | Approved | Passing | 2d |
   | #38 | Refactor DB layer | @bob | Changes requested | Failing | 5d |

   ## Recent Activity
   - 5 commits merged today
   - 2 new issues opened
   - 1 deployment completed (v1.2.3)

   ## Code Health
   - Build: PASSING
   - Tests: 142 passed, 0 failed
   - Coverage: 78.5%
   ```

4. **Store for Trend Tracking**: Log report data in memory
   ```
   sc_memory_store(
     content="Dev report YYYY-MM-DD: build=pass, tests=142/142, open_prs=5, errors=3",
     category="dev-report",
     confidence=1.0
   )
   ```
   - Enables: "show me build health over the past month"
   - Enables: "when did test failures start increasing?"

5. **Alert on Critical Items**: Send urgent items to Telegram
   ```
   sc_send_message(
     message="ALERT: CI broken on main. Commit abc123 by @alice. Error: TypeScript compilation failed.",
     priority="high"
   )
   ```
   - Only alert for: build failures on main, error rate spikes, security vulnerabilities
   - Never alert for: informational updates, non-blocking PR status
</Steps>

<Tool_Usage>
- `Bash` -- Execute gh CLI commands, git commands, build/test commands
- `sc_send_message` -- Send critical alerts to Telegram
- `sc_memory_store` -- Log report data for trend tracking
- `sc_memory_search` -- Query historical reports for trend analysis
- `sc_memory_add_entity` -- Create report entities in knowledge graph
- `sc_memory_add_relation` -- Link reports to deployments, errors, PRs
- `sc_memory_graph_query` -- Query deployment history and error trends
- `Read` -- Load configuration files, build outputs, test results
- `Write` -- Save generated reports to ~/superclaw/data/reports/
- `Grep` -- Search logs and outputs for specific error patterns
- `Glob` -- Find log files, test results, coverage reports
- `WebFetch` -- Access external dashboards or status pages
- `Edit` -- Update configuration files if needed
</Tool_Usage>

<Examples>
<Good>
User: "catch me up - what happened overnight?"
Action:
1. Parallel data collection:
   - gh pr list: 3 new PRs opened, 2 merged
   - gh run list: All CI passing on main, 1 failure on feature/auth branch
   - git log: 8 commits merged since yesterday 6PM
   - Memory search: Compare with yesterday's report
2. Cross-reference: The CI failure on feature/auth is from PR #45 (Bob's auth refactor)
3. Report: "Since yesterday 6PM: 2 PRs merged (both passing CI), 3 new PRs opened. One CI failure on feature/auth branch (PR #45) -- TypeScript error in auth.service.ts. No new errors in production. Build and tests healthy on main."
4. Store report in memory
Why good: Parallel collection, cross-referenced insights, actionable summary, historical context.
</Good>

<Good>
User: "which commits broke the build?"
Action:
1. Get CI failure details: `gh run list --status failure`
2. Get commit list for the failing run
3. Cross-reference with git log
4. Identify the specific commit and author
5. Show the error message and affected files
Why good: Specific, actionable answer with full context.
</Good>

<Bad>
User: "catch me up"
Action: Running `gh pr list` and pasting raw JSON output.
Why bad: Raw data dump without cross-referencing, prioritization, or formatting. Should produce a human-readable report with actionable insights.
</Bad>

<Bad>
User: "fix the CI failure"
Action: Running dev-workflow to generate a report.
Why bad: The user wants a fix, not a report. Route to executor or `ralph` for code changes.
</Bad>
</Examples>

<Escalation_And_Stop_Conditions>
- If gh CLI is not authenticated, instruct the user to run `gh auth login` first
- If the project has no CI pipeline configured, skip CI sections and note the gap
- If Sentry MCP is not available, skip error tracking sections and note the gap
- If git is not initialized in the project, skip git sections
- If critical alerts cannot be sent (Telegram not configured), display them prominently in the report instead
- If data collection from any source fails, proceed with available sources and note what is missing
</Escalation_And_Stop_Conditions>

<Final_Checklist>
- [ ] Data collected from all available sources (GitHub, git, build, errors)
- [ ] Critical items identified and prioritized
- [ ] Cross-referencing performed (commits to PRs, PRs to CI, commits to errors)
- [ ] Report formatted with clear sections and priorities
- [ ] Report stored in memory for trend tracking
- [ ] Critical alerts sent via Telegram (if applicable)
- [ ] No raw data dumps -- all output is human-readable and actionable
- [ ] Missing data sources noted in the report
</Final_Checklist>

<Advanced>
## Developer Morning Brief Template

```markdown
# Morning Brief - {{DATE}}

## Overnight Summary
- Commits merged: {{N}}
- PRs opened: {{N}} | merged: {{N}} | closed: {{N}}
- CI status: {{PASSING/FAILING}} (main branch)
- New errors: {{N}} ({{SEVERITY}})

## Action Items
1. {{CRITICAL_ITEM}} - {{CONTEXT}}
2. {{REVIEW_NEEDED}} - PR #{{N}} by {{AUTHOR}}
3. {{FOLLOW_UP}} - {{CONTEXT}}

## Your Open PRs
| PR | Status | CI | Last Activity |
|----|--------|-----|--------------|
{{PR_TABLE}}

## Team Activity
{{ACTIVITY_SUMMARY}}
```

## PR Review Checklist

When reviewing PRs via this workflow:
- [ ] CI checks passing
- [ ] No merge conflicts
- [ ] Code coverage maintained or improved
- [ ] No security warnings in dependencies
- [ ] Related issues linked
- [ ] Description explains the "why" not just the "what"

## CI Failure Triage Guide

```
CI Failure
├── Build failure
│   ├── TypeScript error → Check recent type changes
│   ├── Import error → Check for missing dependencies
│   └── Compilation error → Check for syntax issues
├── Test failure
│   ├── Assertion error → Check expected vs actual values
│   ├── Timeout → Check for async issues or infinite loops
│   └── Setup error → Check test fixtures and mocks
└── Lint/Format failure
    ├── ESLint → Run auto-fix: npx eslint --fix
    └── Prettier → Run auto-fix: npx prettier --write
```

## Custom Report Templates

Users can customize reports by modifying templates in:
`~/superclaw/config/report-templates/`

Available template variables:
- `{{DATE}}`, `{{TIME}}` -- Current date/time
- `{{BRANCH}}` -- Current git branch
- `{{PR_COUNT}}`, `{{ISSUE_COUNT}}` -- Counts
- `{{CI_STATUS}}` -- Build status
- `{{ERROR_COUNT}}` -- Error tracker count

## Team-Wide Dashboards

For multi-developer projects:
```bash
# All team members' PR status
gh pr list --state open --json number,title,author,reviewDecision

# Contribution stats
git shortlog -sn --since="1 week ago"

# File hotspots (most changed files)
git log --since="1 week ago" --name-only --format="" | sort | uniq -c | sort -rn | head -20
```

## Integration with Automation Pipeline

Schedule daily reports via SuperClaw's cron system:
```
sc_cron_add(
  name="daily-dev-report",
  schedule="0 9 * * 1-5",  # 9 AM weekdays
  command="superclaw dev-workflow morning-brief"
)
```

## Deployment Tracking

Track deployment history:
```
sc_memory_add_entity(
  name="deploy-v1.2.3",
  type="deployment",
  properties={version: "1.2.3", timestamp: "...", commit: "abc123", environment: "production"}
)
sc_memory_add_relation(from="deploy-v1.2.3", to="pr-42", type="includes")
sc_memory_add_relation(from="deploy-v1.2.3", to="pr-38", type="includes")
```

Query: "what PRs were in the last production deploy?"
```
sc_memory_graph_query(query="deployment production latest includes")
```

## Troubleshooting

**gh CLI not found?**
- Install: `brew install gh` (macOS) or see https://cli.github.com/
- Authenticate: `gh auth login`

**Build command unknown?**
- Check package.json for scripts (npm/yarn/pnpm)
- Check Makefile for targets
- Check for CI config files (.github/workflows/, .gitlab-ci.yml)

**Rate limited by GitHub API?**
- gh CLI handles authentication automatically
- For heavy usage, consider using `--limit` flags
- Cache responses in memory for repeated queries within the same session
</Advanced>
