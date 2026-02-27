---
name: create-ticket
description: Turns bug reports, feature requests, technical debt, and ideas into well-structured tickets with acceptance criteria, estimates, subtasks, and labels. Use when the user says "create a ticket", "write a ticket", "file a bug", "log this issue", "create a story", "make a task", "write a Jira ticket", "create an issue", or describes a bug or feature that needs tracking.
allowed-tools: Read, Grep, Glob, Bash
---

# Create Ticket Skill

When creating a ticket, follow this structured process. A good ticket gives the assignee everything they need to understand, implement, and verify the work without back-and-forth questions.

## 1. Classify the Ticket Type

Determine what kind of ticket this is:

| Type | When | Icon |
|------|------|------|
| **Bug** | Something is broken or behaving incorrectly | 🐛 |
| **Feature** | New functionality or capability | ✨ |
| **Improvement** | Enhancing existing functionality | 💡 |
| **Tech Debt** | Refactoring, cleanup, or infrastructure improvement | 🔧 |
| **Security** | Vulnerability fix or security hardening | 🔒 |
| **Performance** | Speed, memory, or scalability improvement | ⚡ |
| **Documentation** | Missing or outdated docs | 📝 |
| **Infrastructure** | CI/CD, deployment, monitoring, DevOps | 🏗️ |
| **Design** | UI/UX changes, visual updates | 🎨 |
| **Research / Spike** | Investigation before implementation | 🔍 |
| **Test** | Adding or fixing tests | ✅ |
| **Dependency** | Library updates, version bumps | 📦 |

## 2. Gather Context

Before writing the ticket, gather relevant information:

### For Bugs
```bash
# Check recent commits that might be related
git log --oneline -20

# Check for related error handling
grep -rn "catch\|throw\|Error\|Exception" --include="*.ts" --include="*.js" --include="*.py" src/ | grep -i "[relevant keyword]"

# Check related files
find src -name "*[relevant keyword]*" -type f

# Check for existing related issues
gh issue list --search "[keyword]" --state all 2>/dev/null
```

### For Features
```bash
# Check if any related code or patterns exist
grep -rn "[related pattern]" --include="*.ts" --include="*.js" --include="*.py" src/

# Check existing models/services that would be affected
find src -name "*[related area]*" -type f

# Check for related tickets already filed
gh issue list --search "[keyword]" --state all 2>/dev/null
```

## 3. Ticket Templates

### 🐛 Bug Report
```markdown
## Title
[Component/Area]: [Clear description of the bug]

Examples:
- "Auth: Users get logged out after password change on other devices"
- "Checkout: Duplicate charges when clicking Pay button twice quickly"
- "Search: Results return 0 items for queries containing special characters"

## Type
Bug

## Priority
[Critical / High / Medium / Low]

## Description

### What Happens
[Describe the actual behavior — what the user sees or experiences]

### What Should Happen
[Describe the expected behavior — what the correct outcome should be]

### Steps to Reproduce
1. [First step — be specific about starting state]
2. [Second step — include exact inputs, clicks, actions]
3. [Third step — include any timing or sequence requirements]
4. **Observe**: [What goes wrong]

### Reproduction Rate
[Always / Intermittent (~X% of the time) / Only under specific conditions]

### Environment
- **Browser/Client**: [Chrome 120, Safari 17, iOS app v3.2, etc.]
- **OS**: [macOS 14.2, Windows 11, iOS 17, Android 14]
- **Environment**: [Production / Staging / Local]
- **User role**: [Admin / Editor / Viewer]
- **Account**: [test-user@example.com or account ID if relevant]

### Error Details
[Include any of the following if available:]
- Error message (exact text)
- Console errors
- Network request/response (status code, body)
- Stack trace
- Log entries
- Screenshot or recording

### Impact
- **Users affected**: [All users / Specific group / Edge case]
- **Severity**: [Data loss / Broken feature / Degraded experience / Visual only]
- **Workaround available**: [Yes — describe / No]
- **Revenue impact**: [Yes — describe / No / Unknown]

### Root Cause (if known)
[If you've investigated and have a hypothesis, include it here]

### Suggested Fix (if known)
[If you have an idea of how to fix it, describe the approach]

## Acceptance Criteria
- [ ] Bug no longer occurs when following the reproduction steps
- [ ] Existing related tests still pass
- [ ] New test added covering this specific scenario
- [ ] Fix doesn't introduce regressions in [related area]
- [ ] [Any specific verification criteria]

## Related
- Related to: #[issue-number]
- Introduced in: [commit/PR if known]
- Affected files: [list key files if known]

## Labels
bug, [component], [priority]
```

### ✨ Feature Request
```markdown
## Title
[Area]: [Clear description of the feature]

Examples:
- "Auth: Add SSO login via SAML for enterprise customers"
- "Dashboard: Add date range filter to analytics charts"
- "API: Add bulk create endpoint for products"

## Type
Feature

## Priority
[Critical / High / Medium / Low]

## Description

### User Story
As a [role/persona],
I want to [action/capability],
so that [benefit/value].

### Background / Context
[Why is this feature needed? What problem does it solve?
Link to customer feedback, support tickets, or product brief.]

### Detailed Requirements

#### Functional Requirements
1. [Specific requirement — what the system must do]
2. [Specific requirement]
3. [Specific requirement]

#### Non-Functional Requirements
- **Performance**: [Response time, throughput expectations]
- **Scale**: [Expected data volume, concurrent users]
- **Security**: [Auth requirements, data sensitivity]
- **Accessibility**: [WCAG level, screen reader support]
- **Internationalization**: [Languages, locales, RTL support]

### User Flow
1. User navigates to [page/section]
2. User clicks [element]
3. System displays [response]
4. User enters [input]
5. System processes and shows [result]

### UI/UX
[Link to Figma/design mockups if available]

[If no designs yet, describe the expected interface:]
- Where does this feature appear in the UI?
- What does the user see?
- What interactions are available?
- What feedback does the user get?

### API Changes (if applicable)

#### New Endpoints
```
POST /api/v1/[resource]
Request: { field1: string, field2: number }
Response: 201 { id: string, field1: string, field2: number, createdAt: string }
```

#### Modified Endpoints
```
GET /api/v1/[resource]
New query params: ?dateFrom=ISO&dateTo=ISO
```

### Database Changes (if applicable)
- New table: `[table_name]` with columns: [list]
- Modified table: `[table_name]` — add column: [column] ([type])
- New index: `[index_name]` on [table].[columns]

### Notifications / Emails (if applicable)
- When: [trigger condition]
- To: [recipient]
- Channel: [email / push / in-app / SMS]
- Content: [brief description]

### Configuration (if applicable)
- New env var: `[VAR_NAME]` — [purpose]
- New feature flag: `[flag_name]` — [purpose]
- New admin setting: [description]

## Acceptance Criteria
- [ ] [Specific, testable criterion — "User can ___"]
- [ ] [Specific, testable criterion]
- [ ] [Specific, testable criterion]
- [ ] [Edge case handled — "System handles ___ by ___"]
- [ ] [Error case handled — "When ___ fails, user sees ___"]
- [ ] [Performance criterion — "Response time under ___ms"]
- [ ] Unit tests added for new business logic
- [ ] Integration tests added for new endpoints
- [ ] Documentation updated
- [ ] Feature flag controls rollout (if applicable)

## Out of Scope
[Explicitly list what this ticket does NOT include.
This prevents scope creep and sets clear boundaries.]
- Not including [related but separate feature]
- Not supporting [specific edge case — future ticket]
- Not changing [related area that stays as-is]

## Dependencies
- Blocked by: #[issue-number] — [why]
- Depends on: [external service / team / decision]
- Blocks: #[issue-number] — [why]

## Design / Technical Approach (optional)
[If the approach has been discussed or decided, document it here.
Include alternatives considered and why this approach was chosen.]

### Option A: [Approach name]
- Pros: ...
- Cons: ...

### Option B: [Approach name] ✅ Recommended
- Pros: ...
- Cons: ...

## Subtasks
- [ ] [Subtask 1 — e.g., "Create database migration"]
- [ ] [Subtask 2 — e.g., "Implement API endpoint"]
- [ ] [Subtask 3 — e.g., "Build UI component"]
- [ ] [Subtask 4 — e.g., "Add tests"]
- [ ] [Subtask 5 — e.g., "Update documentation"]

## Estimation
- **Size**: [XS / S / M / L / XL] or [Story Points: 1/2/3/5/8/13]
- **Estimated time**: [X hours / X days]
- **Complexity**: [Low / Medium / High]
- **Risk**: [Low / Medium / High — describe risk factors]

## Related
- Product brief: [link]
- Design: [Figma link]
- Customer request: [support ticket / feedback link]
- Related tickets: #[number], #[number]

## Labels
feature, [component], [priority], [team]
```

### 💡 Improvement
```markdown
## Title
[Area]: [Clear description of the improvement]

Examples:
- "Checkout: Add address autocomplete to reduce form abandonment"
- "Dashboard: Show loading skeletons instead of spinners"
- "Email: Add unsubscribe link to marketing emails"

## Type
Improvement

## Priority
[Critical / High / Medium / Low]

## Description

### Current Behavior
[How the feature works today]

### Proposed Improvement
[How it should work after this change]

### Motivation
[Why is this improvement needed? Metrics, user feedback, or observations that support it.]
- User feedback: [quote or link]
- Metric: [current value → target value]
- Competitive: [what competitors do]

### Impact
- **Users affected**: [who benefits]
- **Expected outcome**: [metric improvement, reduced friction, etc.]

## Acceptance Criteria
- [ ] [Specific, measurable criterion]
- [ ] [Specific, measurable criterion]
- [ ] Existing functionality not degraded
- [ ] Tests updated to cover the improvement

## Subtasks
- [ ] [Subtask 1]
- [ ] [Subtask 2]
- [ ] [Subtask 3]

## Estimation
- **Size**: [XS / S / M / L / XL]
- **Complexity**: [Low / Medium / High]

## Labels
improvement, [component], [priority]
```

### 🔧 Tech Debt
```markdown
## Title
[Area]: [Clear description of the tech debt]

Examples:
- "Auth: Refactor UserService into separate AuthService and ProfileService"
- "API: Replace manual validation with Zod schemas across all endpoints"
- "Tests: Fix flaky order integration tests caused by race conditions"

## Type
Tech Debt

## Priority
[Critical / High / Medium / Low]

## Description

### Current State
[Describe the current problematic code/architecture]
- What's wrong with it?
- Where in the codebase? [file paths]
- How long has it been this way?

### Problems It Causes
[Why does this matter? Be specific about the pain.]
- Slows down development: [how]
- Causes bugs: [which ones]
- Makes onboarding harder: [how]
- Increases risk: [what risk]
- Performance impact: [what impact]

### Proposed Solution
[Describe the target state]
- What should the code look like after?
- What pattern/approach should replace the current one?
- What files will be affected?

### Migration Strategy
[How to get from current state to target state]
1. [Step 1]
2. [Step 2]
3. [Step 3]

### Risk Assessment
- **Behavior changes**: [None / Minimal — describe / Significant — describe]
- **Regression risk**: [Low / Medium / High — describe]
- **Rollback plan**: [How to revert if something goes wrong]

## Acceptance Criteria
- [ ] Code follows the new pattern/structure
- [ ] All existing tests pass without modification (behavior preserved)
- [ ] No new warnings or linting errors
- [ ] Performance not degraded (benchmark if applicable)
- [ ] [Specific structural criterion]

## Subtasks
- [ ] [Subtask 1]
- [ ] [Subtask 2]
- [ ] [Subtask 3]

## Estimation
- **Size**: [XS / S / M / L / XL]
- **Complexity**: [Low / Medium / High]
- **Risk**: [Low / Medium / High]

## Labels
tech-debt, refactor, [component], [priority]
```

### 🔒 Security
```markdown
## Title
[Security]: [Clear description of the vulnerability or hardening]

Examples:
- "Security: Fix XSS vulnerability in user profile rendering"
- "Security: Add rate limiting to authentication endpoints"
- "Security: Rotate exposed API key in commit history"

## Type
Security

## Priority
[Critical / High / Medium / Low]

## ⚠️ Sensitivity
[Is this ticket sensitive? Should it be private/confidential?]
- If actively exploitable: Make ticket PRIVATE
- If theoretical/hardening: Can be public

## Description

### Vulnerability (if applicable)
- **Type**: [XSS / SQL Injection / IDOR / CSRF / etc.]
- **CWE**: [CWE-XXX if known]
- **CVSS Score**: [if assessed]
- **Affected component**: [file path / endpoint]
- **Exploitability**: [Easy / Moderate / Difficult / Theoretical]

### Current Risk
[What could an attacker do with this vulnerability?]

### Proposed Fix
[How to remediate]

### Discovery
[How was this found? Security audit, penetration test, dependency scan, etc.]

## Acceptance Criteria
- [ ] Vulnerability is remediated
- [ ] Regression test added for the specific attack vector
- [ ] Security audit passes on the affected area
- [ ] No new vulnerabilities introduced by the fix
- [ ] Incident response completed (if applicable)

## Labels
security, [severity], [component]
```

### ⚡ Performance
```markdown
## Title
[Perf]: [Clear description of the performance issue]

Examples:
- "Perf: Reduce /api/products response time from 2.3s to <500ms"
- "Perf: Fix N+1 query in order listing page"
- "Perf: Reduce initial bundle size from 1.8MB to under 500KB"

## Type
Performance

## Priority
[Critical / High / Medium / Low]

## Description

### Current Performance
- **Metric**: [Response time / Load time / Memory usage / Bundle size]
- **Current value**: [measured value with tool used]
- **Target value**: [acceptable threshold]
- **Affected area**: [endpoint / page / operation]

### Root Cause (if known)
[What's causing the slowness?]

### Proposed Solution
[How to improve performance]

### Measurement Plan
- **Tool**: [Lighthouse / Chrome DevTools / APM / custom benchmark]
- **Baseline**: [current measurement]
- **Target**: [goal measurement]
- **How to verify**: [specific steps to measure after fix]

## Acceptance Criteria
- [ ] [Metric] improved from [current] to [target]
- [ ] No functionality regression
- [ ] Performance test added to prevent future regression
- [ ] Measured in production-like environment

## Labels
performance, [component], [priority]
```

### 🔍 Research / Spike
```markdown
## Title
[Spike]: [Clear description of what needs investigation]

Examples:
- "Spike: Evaluate real-time notification options (WebSocket vs SSE vs polling)"
- "Spike: Investigate feasibility of migrating from REST to GraphQL"
- "Spike: Research GDPR compliance requirements for user data export"

## Type
Research / Spike

## Priority
[High / Medium / Low]

## Description

### Question to Answer
[The specific question(s) this spike should answer]

1. [Question 1]
2. [Question 2]
3. [Question 3]

### Context
[Why do we need this research? What decision depends on it?]

### Scope
[What should be investigated and what's out of scope]

### Time Box
[Maximum time to spend: e.g., 2 days, 4 hours, 1 sprint]

## Expected Deliverables
- [ ] Written summary of findings
- [ ] Pros/cons comparison of options (if applicable)
- [ ] Recommendation with reasoning
- [ ] Proof of concept (if applicable)
- [ ] Estimated effort for implementation
- [ ] Follow-up tickets created based on findings

## Labels
spike, research, [component]
```

### 🏗️ Infrastructure
```markdown
## Title
[Infra]: [Clear description of the infrastructure change]

Examples:
- "Infra: Set up staging environment with automated deployments"
- "Infra: Add Datadog APM monitoring to production services"
- "Infra: Migrate database from RDS t3.medium to r6g.large"

## Type
Infrastructure

## Priority
[Critical / High / Medium / Low]

## Description

### Current State
[Current infrastructure setup for the affected area]

### Desired State
[Target infrastructure setup]

### Changes Required
1. [Change 1]
2. [Change 2]
3. [Change 3]

### Impact
- **Downtime required**: [Yes — estimated duration / No]
- **Cost change**: [+$X/month / -$X/month / Neutral]
- **Services affected**: [list]
- **Rollback plan**: [how to revert]

## Acceptance Criteria
- [ ] [Infrastructure change verified]
- [ ] Monitoring/alerting configured
- [ ] Documentation updated
- [ ] Team notified of changes
- [ ] Runbook updated (if applicable)

## Labels
infrastructure, devops, [priority]
```

## 4. Estimation Guide

### Size Estimation

| Size | Story Points | Time Estimate | Characteristics |
|------|-------------|---------------|-----------------|
| **XS** | 1 | < 2 hours | Config change, copy update, one-line fix |
| **S** | 2 | 2-4 hours | Simple bug fix, small component, add test |
| **M** | 3-5 | 1-2 days | New feature within existing patterns, moderate refactor |
| **L** | 8 | 3-5 days | Complex feature, new patterns, multiple components |
| **XL** | 13 | 1-2 weeks | Large feature, architectural change, cross-team work |
| **XXL** | 21+ | 2+ weeks | Should be broken into smaller tickets |

### Complexity Factors
- **Low**: Follows existing patterns, well-understood area, minimal dependencies
- **Medium**: Some new patterns needed, touches 2-3 modules, moderate testing effort
- **High**: New architecture, cross-team dependencies, unfamiliar territory, high risk

### Signals That a Ticket Is Too Big
- More than 5 subtasks
- Estimated at L or larger
- Touches more than 3 unrelated areas
- Requires multiple reviewers from different teams
- Can't be described in a single user story
- Estimated at more than 1 week

**If too big**: Break it into smaller tickets and create an epic to group them.

## 5. Priority Framework

| Priority | Criteria | Response |
|----------|----------|----------|
| 🔴 **Critical** | Production down, data loss, security breach, revenue blocking | Fix immediately, drop everything |
| 🟠 **High** | Major feature broken, significant user impact, approaching deadline | Fix this sprint |
| 🟡 **Medium** | Minor feature issue, workaround exists, improvement with clear value | Schedule for next sprint |
| 🟢 **Low** | Cosmetic, nice-to-have, minor tech debt, edge case | Add to backlog, address when convenient |

## 6. Auto-Generate Subtasks

Based on the ticket type and scope, generate appropriate subtasks:

### Feature Subtasks Pattern
```
- [ ] Design: Create/review UI mockups
- [ ] DB: Create migration for [new table/columns]
- [ ] API: Implement [endpoint(s)]
- [ ] Service: Implement business logic in [service]
- [ ] UI: Build [component(s)]
- [ ] Tests: Add unit tests for [service/component]
- [ ] Tests: Add integration tests for [endpoint(s)]
- [ ] Docs: Update API documentation
- [ ] Docs: Update README / user docs
- [ ] Review: Code review and QA
```

### Bug Fix Subtasks Pattern
```
- [ ] Investigate: Reproduce and identify root cause
- [ ] Fix: Implement the fix in [file(s)]
- [ ] Test: Add regression test for this specific scenario
- [ ] Test: Verify fix in staging environment
- [ ] Verify: Confirm related features not affected
```

### Tech Debt Subtasks Pattern
```
- [ ] Analyze: Map current code and dependencies
- [ ] Plan: Document migration strategy
- [ ] Implement: Refactor [component/module] (phase 1)
- [ ] Implement: Refactor [component/module] (phase 2)
- [ ] Test: Verify behavior is preserved
- [ ] Cleanup: Remove deprecated code
- [ ] Docs: Update architecture documentation
```

## 7. Platform-Specific Issue Creation

### GitHub Issues
```bash
# Create issue
gh issue create \
  --title "feat(auth): Add OAuth2 login with Google and GitHub" \
  --body "$(cat ticket.md)" \
  --label "feature,auth,size/M" \
  --assignee "@me" \
  --milestone "v2.4.0"

# Create with project board
gh issue create \
  --title "fix(checkout): Prevent duplicate charges on retry" \
  --body "$(cat ticket.md)" \
  --label "bug,critical,checkout" \
  --project "Sprint 24"
```

### GitLab Issues
```bash
# Create issue
glab issue create \
  --title "feat(auth): Add OAuth2 login with Google and GitHub" \
  --description "$(cat ticket.md)" \
  --label "feature,auth" \
  --assignee "@me" \
  --milestone "v2.4.0"
```

### Jira (via API)
```bash
# Create via Jira REST API
curl -X POST "https://your-domain.atlassian.net/rest/api/3/issue" \
  -H "Content-Type: application/json" \
  -H "Authorization: Basic $(echo -n email:api-token | base64)" \
  -d '{
    "fields": {
      "project": { "key": "PROJ" },
      "summary": "feat(auth): Add OAuth2 login with Google and GitHub",
      "description": { "type": "doc", "version": 1, "content": [...] },
      "issuetype": { "name": "Story" },
      "priority": { "name": "High" },
      "labels": ["feature", "auth"],
      "story_points": 5
    }
  }'
```

### Linear (via API)
```bash
# Create via Linear API
curl -X POST "https://api.linear.app/graphql" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $LINEAR_API_KEY" \
  -d '{
    "query": "mutation { issueCreate(input: { title: \"...\", description: \"...\", teamId: \"...\", priority: 2 }) { success issue { id identifier url } } }"
  }'
```

## 8. Quality Checklist

Before finalizing any ticket, verify:
```markdown
### Ticket Quality Check
- [ ] **Title is clear** — someone can understand the scope from the title alone
- [ ] **Type is correct** — bug vs feature vs tech debt etc.
- [ ] **Priority is justified** — based on impact and urgency, not gut feeling
- [ ] **Description is complete** — no critical information missing
- [ ] **Acceptance criteria are testable** — each one can be verified as done/not done
- [ ] **Scope is bounded** — out of scope is explicitly stated
- [ ] **Dependencies are listed** — blockers and related work identified
- [ ] **Estimation is reasonable** — sized appropriately, broken down if too large
- [ ] **Labels are applied** — type, component, priority, size
- [ ] **Assignee or team is clear** — someone knows this is theirs
- [ ] **No jargon without explanation** — a new team member could understand this
- [ ] **Subtasks are actionable** — each subtask can be independently worked on
```

## Output Format

Present the ticket as:

### 1. Ticket Preview
Show the complete formatted ticket ready to be filed.

### 2. Metadata Summary
```
Type: Feature
Priority: High
Size: M (3-5 story points, ~2 days)
Complexity: Medium
Labels: feature, auth, size/M
Subtasks: 6
Dependencies: None
```

### 3. Filing Options
- Command to create on GitHub/GitLab/Jira
- Or the formatted markdown ready to paste

### 4. Related Tickets (if applicable)
- Suggest related tickets that should also be created
- Identify if this should be part of an epic

## Adaptation Rules

- **Detect project tracker** — check for `.github/`, `.gitlab-ci.yml`, or ask the user
- **Match existing ticket style** — look at recent issues to match team conventions
- **Detect from conversation** — if the user describes a bug casually, extract the structured information
- **Detect from code** — if pointed at a file or error, analyze the code to fill in technical details
- **Break down large scope** — if the request is too big for one ticket, suggest an epic with multiple tickets
- **Ask only what's missing** — if the user provides most details, don't ask for information you can infer
- **Security sensitivity** — if it's a vulnerability, recommend making the ticket private
- **Suggest priority** — based on impact analysis, suggest a priority with reasoning

## Summary

End every ticket creation with:
1. **Ticket ready** — formatted and ready to file
2. **Filing command** — platform-specific command to create it
3. **Suggested assignee** — based on code ownership if detectable
4. **Related work** — other tickets that should be created alongside this one
5. **Questions** — anything that needs clarification before filing
