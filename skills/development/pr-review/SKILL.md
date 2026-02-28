---
name: pr-review
description: Analyze pull requests and provide structured code review feedback. Use when asked to review PR, analyze pull request, check code changes, provide PR feedback, or code review.
allowed-tools: [list_dir, read_file, grep_search, semantic_search, get_changed_files, get_errors, list_code_usages]
---

# PR Review Skill

Provides comprehensive code review analysis for pull requests with structured feedback format.

## When to Activate

This skill activates when the user requests:
- "review this PR"
- "analyze pull request #123"
- "check the code changes"
- "provide feedback on my PR"
- "code review needed"

## Required Workflow

### Step 1: Gather PR Context

1. **Ask for PR number** (if not provided)
2. **Read changed files** using `get_changed_files`
3. **Check for errors** using `get_errors`
4. **Read relevant files** using `read_file` for context

### Step 2: Load Project Configuration (Optional)

Try to read configuration in this order:
1. `.github/skills/pr-review/config.md` (project-specific)
2. `~/.copilot/skills/pr-review/config.md` (user defaults)
3. `.github/copilot-instructions.md` (project context)

**Config format**:
```markdown
- **Code Style Guide**: [path/to/style-guide.md]
- **Architecture Patterns**: [docs/Architecture/]
- **Testing Requirements**: Unit tests required for all new features
- **Review Priorities**: Security, Performance, Offline-first patterns
- **Code Quality Rules**: Logger usage, hook patterns, etc.
```

If config doesn't exist, proceed with general review guidelines.

### Step 3: Analyze Against Standards

Use references/review-checklist.md to evaluate:

1. **PR Metadata**:
   - Title follows Conventional Commits format
   - Description includes Jira reference
   - Scope is clear and focused
   - Breaking changes documented

2. **Code Quality**:
   - TypeScript types are correct
   - React best practices followed
   - No performance anti-patterns
   - Security vulnerabilities checked

3. **Architecture Alignment**:
   - Offline-first patterns (if applicable)
   - API contract compliance
   - Database schema compatibility
   - Sync conflict handling

4. **Project Conventions**:
   - File structure follows project layout
   - Git conventions (branch naming, commit messages)
   - CI/CD checks passing

### Step 4: Structured Feedback Format

**Output structure**:

```markdown
## Quick Assessment

**Strengths** ✅:
- [List 2-3 positive aspects]

**Concerns** ⚠️:
- [List any blocking issues]

---

## Detailed Feedback

### Critical Issues ❌ (Must fix before merge)
- **[File:Line]**: [Issue description]
  - **Why**: [Impact/reasoning]
  - **Fix**: [Specific action to take]

### Major Issues 🔶 (Should fix)
- **[File:Line]**: [Issue description]
  - **Suggestion**: [How to improve]

### Minor Issues 📝 (Consider fixing)
- **[File:Line]**: [Issue description]
  - **Optional**: [Improvement idea]

---

## Checklist Review

- [ ] PR title follows Conventional Commits
- [ ] Jira ticket referenced in description
- [ ] TypeScript types are sound
- [ ] Tests added/updated
- [ ] Documentation updated
- [ ] No console.log/debugging code
- [ ] Offline-first patterns preserved
- [ ] API contract unchanged (or documented)
- [ ] CI checks passing

---

## Recommendation

[Approve ✅ | Request Changes 🔄 | Reject ❌]

**Rationale**: [1-2 sentence summary]
```

### Step 5: Ask About Posting Review

After generating feedback:

```
Would you like me to post this review to GitHub PR #XXX?
(I can use GitHub API to add review comments)
```

## Stop Rules

**STOP and ask user if**:
1. ❌ PR number not provided and cannot be inferred
2. ❌ No changed files found
3. ❌ Critical errors detected that make review impossible
4. ❌ User has pending uncommitted changes (might affect review)

## References

- `references/review-checklist.md` - Comprehensive review criteria
- Project-specific: `.github/skills/pr-review/config.md`

## Success Criteria

✅ **Review is complete when**:
- All changed files analyzed
- Feedback categorized by severity
- Specific file/line references provided
- Actionable items identified
- Recommendation given with rationale
