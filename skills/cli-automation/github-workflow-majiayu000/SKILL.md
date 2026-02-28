---
name: github-workflow
description: GitHub workflow automation for LiquidationHeatmap. Generate standardized PR descriptions, issue templates, and commit messages. Reduces token usage by 70% on repetitive GitHub operations.
---

# GitHub Workflow Automation

Automate common GitHub operations with standardized templates to reduce token consumption.

## Token Consumption Analysis

### **Without Skill** (Direct GitHub MCP Tools)

| Operation | MCP Tool | Avg Tokens | Notes |
|-----------|----------|-----------|-------|
| Create PR | `mcp__github__create_pull_request` | ~2,500 | Tool metadata + body generation |
| Create Issue | `mcp__github__create_issue` | ~1,800 | Template + label reasoning |
| Add PR Comment | `mcp__github__add_comment_to_pending_review` | ~1,200 | Review comment context |
| List Issues | `mcp__github__list_issues` | ~2,000 | Pagination + filtering logic |
| Search Code | `mcp__github__search_code` | ~3,500 | Query construction + results |
| Update Issue | `mcp__github__update_issue` | ~1,500 | State reasoning |

**Estimated Total**: ~12,500 tokens for typical PR workflow

### **With Skill** (Template-Driven)

| Operation | Template | Tokens | Savings |
|-----------|----------|--------|---------|
| Create PR | `liquidation_pr_template` | ~600 | **76%** ✅ |
| Create Issue | `task_issue_template` | ~500 | **72%** ✅ |
| Add PR Comment | `review_comment_template` | ~300 | **75%** ✅ |
| Commit Message | `commit_msg_template` | ~200 | **87%** ✅ |

**Estimated Total**: ~1,600 tokens (87% reduction)

---

## Quick Start

### Create Pull Request
```
User: "Create PR for Task 01 implementation"

Skill: github-workflow
└─ Generates PR with LiquidationHeatmap template:
   Title: "[Task 01] Bitcoin Interface: ZMQ Listener"
   Body: Auto-generated from template
   Labels: enhancement, task-01
   Draft: true (MVP incomplete)
```

**Token Cost**: ~600 (vs ~2,500 with MCP tool)

---

## Templates

### 1. Pull Request Template

```yaml
# .claude/skills/github-workflow/templates/pr_template.md

Title Pattern: "[Task XX] Module Name: Brief Description"

Body:
## Summary
{description}

## Changes
- **Deliverable**: `{file_path}`
- **Tests**: `{test_file}` ({test_count} tests)
- **Coverage**: {coverage}%
- **Integration**: Task {prev} → Task {next}

## Test Plan
- [ ] Unit tests passing
- [ ] Integration tests passing
- [ ] Coverage >{threshold}%
- [ ] Manual testing completed

## Checklist
- [ ] Code follows LiquidationHeatmap style
- [ ] Black box interface maintained
- [ ] Documentation updated
- [ ] TDD cycle completed (RED-GREEN-REFACTOR)

## Related
- Closes #{issue_number}
- Part of #epic-mempool-live

🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

**Usage**:
```python
skill.create_pr(
    task="01",
    module="Bitcoin Interface",
    description="ZMQ listener for mempool transactions",
    file_path="live/backend/zmq_listener.py",
    test_file="tests/test_zmq_listener.py",
    coverage=87,
    issue_number=5
)
```

**Output**: Ready-to-use PR via `mcp__github__create_pull_request`

---

### 2. Issue Template

```yaml
# .claude/skills/github-workflow/templates/issue_task.md

Title: "[Task XX] Module Name"

Body:
## Goal
{goal}

## Deliverables
- [ ] `{deliverable_file}`
- [ ] `{test_file}`
- [ ] Coverage >{threshold}%
- [ ] Integration with Task {prev_task}

## Acceptance Criteria
{criteria_list}

## Dependencies
- Requires: Task {prev_task}
- Blocks: Task {next_task}

## Resources
- Task file: `docs/tasks/{task_number}_{module}.md`
- Agent: {agent_name}
- Estimated: {estimate}

## Labels
- `enhancement`
- `task-{task_number}`
- `{priority}`
```

**Usage**:
```python
skill.create_issue(
    task="02",
    module="Transaction Processor",
    goal="Implement liquidation heatmap visualization component",
    deliverable_file="live/backend/tx_processor.py",
    agent_name="transaction-processor",
    estimate="2-3 weeks"
)
```

---

### 3. PR Review Comment Template

```yaml
# Review comment for specific code

## Pattern: Suggest Improvement
🔍 **Code Review**: {file}:{line}

**Issue**: {issue_description}

**Suggestion**:
```{language}
{suggested_code}
```

**Reason**: {explanation}
```

**Usage**:
```python
skill.add_review_comment(
    file="live/backend/zmq_listener.py",
    line=45,
    issue="Missing error handling for connection loss",
    suggestion="Add try/except with exponential backoff",
    language="python"
)
```

---

### 4. Commit Message Template

```yaml
# Standardized commit format

Pattern:
[Task XX] Module: Brief description

{detailed_changes}

- Deliverable: {file}
- Tests: {test_file}
- Coverage: {coverage}%
- Integration: {integration}

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

**Usage**:
```python
skill.generate_commit_msg(
    task="01",
    module="Bitcoin Interface",
    description="Implement ZMQ listener",
    file="live/backend/zmq_listener.py",
    coverage=87
)
```

**Output**:
```
[Task 01] Bitcoin Interface: Implement ZMQ listener

- Add async ZMQ subscription to Bitcoin Core
- Implement reconnection logic with exponential backoff
- Add health check monitoring

- Deliverable: live/backend/zmq_listener.py
- Tests: tests/test_zmq_listener.py (3 tests)
- Coverage: 87%
- Integration: Task 01 → Task 02

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

---

## Workflow Automation

### Workflow 1: Complete PR Creation
```
User: "Create PR for Task 01"

Skill Execution:
1. Read git diff
2. Extract changes summary
3. Count tests
4. Check coverage
5. Generate PR body from template
6. Call mcp__github__create_pull_request with template
7. Add labels (enhancement, task-01)
8. Set draft=true (if MVP incomplete)

Token Cost: ~600 (vs ~2,500 manual)
```

### Workflow 2: Issue + Branch + PR Draft
```
User: "Setup Task 02 workflow"

Skill Execution:
1. Create issue from template → mcp__github__create_issue
2. Create branch task-02/transaction-processor → mcp__github__create_branch
3. Create draft PR → mcp__github__create_pull_request
4. Link PR to issue

Token Cost: ~1,200 (vs ~4,500 manual)
```

### Workflow 3: Review Comment Batch
```
User: "Add review comments for PR #12"

Skill Execution:
1. Read PR diff → mcp__github__pull_request_read
2. Generate 5 review comments from template
3. Batch submit → mcp__github__add_comment_to_pending_review
4. Submit review → mcp__github__submit_pending_pull_request_review

Token Cost: ~900 (vs ~3,000 manual)
```

---

## Token Savings Comparison

### Scenario: Full Task Implementation (Task 01)

**Without Skill**:
```
1. Create branch: manual                      ~500 tokens
2. Implement code: (subagent)                ~8,000 tokens
3. Create PR with reasoning:                 ~2,500 tokens
4. Add review comments:                      ~1,800 tokens
5. Update issue:                             ~1,500 tokens
6. Generate commit msg:                      ~1,500 tokens

Total: ~15,800 tokens
```

**With Skill**:
```
1. Create branch: template                     ~200 tokens
2. Implement code: (subagent)                ~8,000 tokens
3. Create PR from template:                    ~600 tokens
4. Add review comments:                        ~300 tokens
5. Update issue:                               ~400 tokens
6. Generate commit msg:                        ~200 tokens

Total: ~9,700 tokens (38% reduction)
```

**Savings**: ~6,100 tokens per task (38% reduction on GitHub ops)

---

## Integration with Existing Workflow

### Before (Subagents Only)
```
User: "Implement Task 01"
├─ Subagent: bitcoin-onchain-expert (implement)
├─ Manual: Create PR (reasoning + GitHub MCP)
├─ Manual: Write commit message
└─ Manual: Update issue

Total: ~15,800 tokens
```

### After (Subagents + GitHub Skill)
```
User: "Implement Task 01"
├─ Subagent: bitcoin-onchain-expert (implement)
├─ Skill: github-workflow (create PR)        ✅ -76% tokens
├─ Skill: github-workflow (commit msg)       ✅ -87% tokens
└─ Skill: github-workflow (update issue)     ✅ -73% tokens

Total: ~9,700 tokens (38% reduction)
```

---

## Automatic Invocation

**Triggers**:
- "create PR for [task/module]"
- "create issue for Task [XX]"
- "generate commit message for [changes]"
- "add review comment to PR [number]"
- "setup workflow for Task [XX]"

**Does NOT trigger**:
- Complex PR review reasoning (use human or subagent)
- Code conflict resolution (use subagent)
- Strategic issue prioritization (use human)

---

## Configuration

### Enable GitHub MCP
```json
// .mcp.codebase.json (already configured)
{
  "github": {
    "type": "http",
    "url": "https://api.githubcopilot.com/mcp",
    "headers": {
      "Authorization": "Bearer ${GITHUB_PAT}"
    }
  }
}
```

### Environment Variable
```bash
export GITHUB_PAT="your_github_pat_token"
```

### Project Config
```yaml
# .claude/skills/github-workflow/config.yaml
repository: "gptprojectmanager/LiquidationHeatmap"
default_branch: "main"
labels:
  - "enhancement"
  - "task-01"
  - "task-02"
  - "task-03"
  - "task-04"
  - "task-05"
reviewers:
  - "gptprojectmanager"
```

---

## Best Practices

### PR Best Practices
- Always use draft PRs for incomplete work
- Link to related issue (#XX)
- Include test coverage in description
- Use conventional commit style in title

### Issue Best Practices
- One task per issue
- Clear acceptance criteria
- Link to task file in docs/
- Assign appropriate labels

### Commit Message Best Practices
- Follow [Task XX] prefix
- Be concise but descriptive
- Include deliverable file paths
- Always add Claude attribution

---

## Example Output

### Generated PR Description
```markdown
## Summary
Implement ZMQ listener for real-time mempool transaction streaming from Bitcoin Core.

## Changes
- **Deliverable**: `live/backend/zmq_listener.py`
- **Tests**: `tests/test_zmq_listener.py` (3 tests)
- **Coverage**: 87%
- **Integration**: Task 01 → Task 02 (raw tx bytes output)

## Test Plan
- [x] Unit tests passing
- [x] Integration tests passing
- [x] Coverage >80% (87%)
- [x] Manual testing with Bitcoin Core v27

## Checklist
- [x] Code follows LiquidationHeatmap style
- [x] Black box interface maintained
- [x] Documentation updated (CLAUDE.md)
- [x] TDD cycle completed (RED-GREEN-REFACTOR)

## Related
- Closes #5
- Part of #epic-mempool-live

🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

---

## Token Economics Summary

| Operation | Without Skill | With Skill | Savings |
|-----------|--------------|------------|---------|
| Create PR | 2,500 | 600 | **76%** |
| Create Issue | 1,800 | 500 | **72%** |
| PR Comments | 1,200 | 300 | **75%** |
| Commit Msg | 1,500 | 200 | **87%** |
| **Total/Task** | **7,000** | **1,600** | **77%** |

**Full Pipeline (5 Tasks)**:
- Without: 35,000 tokens
- With: 8,000 tokens
- **Savings: 27,000 tokens (77%)**

---

## Limitations

### What Skill Does NOT Do
- ❌ Complex code review reasoning (use subagent)
- ❌ Merge conflict resolution (manual)
- ❌ Strategic issue prioritization (human decision)
- ❌ Security vulnerability analysis (specialized tool)

### What Skill DOES Do
- ✅ Generate standardized PR/issue templates
- ✅ Auto-fill metadata (labels, assignees, milestones)
- ✅ Batch GitHub operations
- ✅ Reduce token consumption on repetitive ops
