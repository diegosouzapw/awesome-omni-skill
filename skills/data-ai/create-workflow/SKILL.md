---
name: create-workflow
description: Create Jazz workflow automation files (WORKFLOW.md). Use this for scheduling Jazz agents to run recurring tasks. For OS-level scripts/commands, use create-system-routine.
---

# Create Workflow

Generate workflow automation files that schedule jazz agents to run recurring tasks.

## When to Use

- User wants to automate a recurring task using a Jazz Agent
- User asks to "create a workflow" or "schedule an agent"
- User says "I want to check X every Y" (e.g., "check emails every hour")
- User wants unattended agent execution

**Note:** If the user wants to schedule a simple shell script or system command *without* using a Jazz Agent, use the `create-system-routine` skill instead.

## Gathering Information (Questionnaire)

**Do not create the workflow file until you have enough information.** If the user's prompt is vague (e.g. "create a workflow", "I want to automate emails") or missing any of the items below, guide them through a short questionnaire instead of guessing.

**You have enough info when you know:**

1. **Task**: What should the agent do? (Concrete, not vague.)
2. **Schedule**: When should it run? (So you can convert to cron.)
3. **Catch-up**: Should missed runs execute on startup? (If not specified, ask or choose a sensible default.)
4. **Auto-approve**: What risk level? (read-only / low-risk / high-risk / false—default to conservative.)
5. **Skills**: Which skills does the agent need? (e.g. email, calendar, deep-research.)
6. **Instructions**: Clear, safety-first prompt with explicit criteria and "when in doubt" rules.
7. **Location**: Where to save? (Local `./workflows/<name>/` or global `~/.jazz/workflows/<name>/`.)

**How to run the questionnaire:**

- Ask **one or a few questions at a time**; don't dump a long list.
- Use the user's words to refine (e.g. "You said 'clean my inbox'—should that be archive only, or also delete? How old is 'old'?").
- After each answer, confirm what you have and ask only what's still missing.
- Once you have all seven items above, proceed to create the WORKFLOW.md.

## Workflow

1. **Understand the task**: What should the agent do? When should it run? If unclear, ask.
2. **Determine schedule**: Convert user intent to cron format; if not stated, ask when they want it to run.
3. **Decide catch-up behavior**: Should missed runs execute on startup? Ask if not specified.
4. **Choose auto-approve policy**: Based on risk level of operations; default to conservative.
5. **Identify required skills**: What skills will the agent need?
6. **Write clear instructions**: Safety-first prompt with explicit guidelines.
7. **Create WORKFLOW.md**: Place in appropriate directory only after you have enough info from above.

## Workflow File Structure

```markdown
---
name: workflow-name
description: Brief one-line summary
schedule: "0 8 * * *"
autoApprove: read-only
catchUpOnStartup: true
maxCatchUpAge: 43200
agent: default
skills:
  - skill-name
---

# Workflow Title

[Clear instructions for the agent, including safety guidelines]
```

## Frontmatter Fields

| Field              | Required | Description                                     | Example                                               |
| ------------------ | -------- | ----------------------------------------------- | ----------------------------------------------------- |
| `name`             | ✓        | Kebab-case identifier                           | `email-cleanup`                                       |
| `description`      | ✓        | One-line summary                                | `Clean up old newsletters`                            |
| `schedule`         |          | Cron expression                                 | `0 * * * *` (hourly)                                  |
| `agent`            |          | Agent to use (optional, will prompt if omitted) | `research-agent`                                      |
| `autoApprove`      |          | Auto-approval policy                            | `read-only`, `low-risk`, `high-risk`, `true`, `false` |
| `skills`           |          | Skills to load                                  | `["email", "calendar"]`                               |
| `catchUpOnStartup` |          | Run missed workflows on startup                 | `true`                                                |
| `maxCatchUpAge`    |          | Max age (seconds) for catch-up runs             | `43200` (12 hours)                                    |

## Cron Schedule Guide

Ask user when they want it to run, then convert to cron:

| User Intent          | Cron Expression | Description               |
| -------------------- | --------------- | ------------------------- |
| Every hour           | `0 * * * *`     | At minute 0 of every hour |
| Every morning at 8am | `0 8 * * *`     | Daily at 8:00 AM          |
| Every 15 minutes     | `*/15 * * * *`  | Every 15 minutes          |
| Weekdays at 9am      | `0 9 * * 1-5`   | Mon-Fri at 9:00 AM        |
| Every Monday at 9am  | `0 9 * * 1`     | Weekly on Monday          |
| First of month       | `0 0 1 * *`     | Monthly at midnight       |

Cron format: `minute hour day-of-month month day-of-week`

## Auto-Approve Policy

Choose based on what tools the workflow will use:

| Policy      | When to Use                           | Example Workflows                                |
| ----------- | ------------------------------------- | ------------------------------------------------ |
| `read-only` | Only reads/searches, no modifications | Weather check, news digest, monitoring           |
| `low-risk`  | Modifies data but reversible          | Email archiving, labeling, calendar events       |
| `high-risk` | Deletes, sends, executes commands     | Email cleanup with deletion, automated responses |
| `false`     | Always ask for approval               | Testing, development                             |

**Default to the most restrictive policy that allows the workflow to function.**

## Safety Guidelines

Always include safety rules in the workflow prompt:

```markdown
**Safety Rules:**
- When in doubt, DO NOTHING
- Only perform actions you're 100% confident about
- Leave uncertain items for manual review
- [Add task-specific safety rules]
```

For workflows that modify or delete:
- Emphasize conservative behavior
- Specify exact criteria (e.g., "only archive newsletters older than 2 weeks")
- Add "if unsure, skip it" instructions

## File Location

Ask user where they want the workflow (or choose based on context):

| Location   | Use Case                      | Path                                   |
| ---------- | ----------------------------- | -------------------------------------- |
| **Local**  | Project-specific automation   | `./workflows/<name>/WORKFLOW.md`       |
| **Global** | User-wide personal automation | `~/.jazz/workflows/<name>/WORKFLOW.md` |

Built-in workflows are shipped with Jazz and shouldn't be modified.

## Complete Example

User request: *"I want to clean up my email inbox every hour, archiving old newsletters"*

**Questions to ask:**
1. ✓ What counts as "old"? (2 weeks)
2. ✓ Should it only archive or also delete? (archive only)
3. ✓ Any specific senders to target? (newsletters, promotional)
4. ✓ Where to save it? (global: `~/.jazz/workflows/`)

**Generated workflow:**

```markdown
---
name: email-cleanup
description: Archive old newsletters and promotional emails
schedule: "0 * * * *"
autoApprove: low-risk
skills:
  - email
---

# Email Cleanup

Review my inbox from the last hour and archive emails matching these criteria:

**Criteria for archiving:**
- Newsletters older than 2 weeks
- Promotional emails older than 3 days
- GitHub notifications already read

**Safety Rules:**
- When in doubt, DO NOTHING
- Only archive emails you're 100% confident match the criteria
- If an email might be important, leave it in inbox
- Never delete, only archive

**Output:**
Log count of archived emails to console.
```

## Skills Integration

If the workflow needs specific capabilities, suggest relevant skills:

| Task Type           | Skills to Include             |
| ------------------- | ----------------------------- |
| Email management    | `email`                       |
| Calendar tasks      | `calendar`                    |
| Research/analysis   | `deep-research`               |
| Code/git operations | (none needed, built-in tools) |
| Web searches        | (none needed, built-in)       |

Include skills in frontmatter:
```yaml
skills:
  - email
  - calendar
```

And reference in prompt:
```markdown
Use the `email` skill to access my inbox efficiently.
```

## After Creation

Tell the user what to do next:

```bash
# Test the workflow manually first
jazz workflow run <name>

# Once confident, schedule it
jazz workflow schedule <name>

# Monitor logs
tail -f ~/.jazz/logs/<name>.log

# View run history
jazz workflow history <name>
```

## Common Workflow Patterns

### Daily Research Digest

```markdown
---
name: tech-digest
description: Daily AI and tech news digest
schedule: "0 8 * * *"
autoApprove: true
skills:
  - deep-research
---

# Daily Tech Digest

Research the most important AI and tech news from the last 24 hours.

**Sources:** Twitter, Reddit, Hacker News, Hugging Face, TechCrunch

**Output:** Save summary to `~/digests/YYYY/Month/DD.md`

**Format:** Brief bullet points with source links
```

### Morning Briefing

```markdown
---
name: morning-briefing
description: Weather, calendar, and news brief
schedule: "0 7 * * 1-5"
autoApprove: read-only
---

# Morning Briefing

Provide a concise morning brief:

1. Today's weather and outfit suggestion
2. Calendar events for today
3. Top 3 news headlines

Keep it under 100 words - this is a quick glance.
```

### GitHub Issue Triage

```markdown
---
name: github-triage
description: Label and prioritize new GitHub issues
schedule: "0 9 * * 1-5"
autoApprove: low-risk
---

# GitHub Issue Triage

Review new GitHub issues from the last 24 hours:

1. Add appropriate labels (bug, feature, documentation)
2. Set priority (P0-P3) based on severity
3. Add to project board if critical

**Safety:** Only add labels and project assignments. Never close issues.
```

### Weekly Standup Report

```markdown
---
name: weekly-standup
description: Generate weekly progress report
schedule: "0 17 * * 5"
autoApprove: read-only
---

# Weekly Standup Report

Generate a weekly summary:

1. Completed tasks (from GitHub, calendar, notes)
2. In-progress work
3. Blockers or challenges
4. Next week's priorities

**Output:** Save to `~/standups/YYYY-WW.md`
```

## Validation Checklist

Before creating the file, verify:

- ✓ Name is kebab-case, descriptive
- ✓ Description is one clear sentence
- ✓ Schedule is valid cron (5 fields)
- ✓ Auto-approve matches task risk level
- ✓ Safety rules are explicit and conservative
- ✓ Skills are listed if needed
- ✓ Output location is specified if workflow saves files
- ✓ Instructions are clear enough for an AI agent

## Anti-Patterns

- ❌ Vague instructions: "clean my inbox" → specify exactly what to clean
- ❌ Overly aggressive auto-approve: Don't use `high-risk` unless necessary
- ❌ Missing safety rules: Always include "when in doubt" guidelines
- ❌ No schedule: If automating, include a schedule (or explain it's manual-only)
- ❌ Wrong location: Project-specific workflows shouldn't be global
- ❌ Too complex: Break into multiple simple workflows instead

## Testing Guidance

Always recommend testing before scheduling:

1. **Manual run first**: `jazz workflow run <name>` to verify logic
2. **Check with `--auto-approve`**: Test the auto-approval behavior
3. **Monitor initial runs**: Watch logs for first few scheduled executions
4. **Iterate**: Adjust safety rules and criteria based on actual behavior

## Documentation

After creating the workflow, optionally create a README in the workflow directory explaining:
- What it does
- When it runs
- What to expect
- How to customize

Example: `workflows/email-cleanup/README.md`
