---
name: commit-feature
description: Stage changes, create conventional commit (no co-author), push to origin, and add detailed PR comment with session context
argument-hint: [type] [scope] — e.g., "feat api" or "fix validation"
---

# Commit Feature Without Co-Authoring

Stage all changes, create a conventional commit message without Claude co-authoring, push to origin, and add a detailed PR comment with session context including TODOs and known issues.

## Arguments

`$ARGUMENTS` should contain:
- **type** (required): `feat`, `fix`, `chore`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `revert`
- **scope** (optional): Component or area affected (e.g., `api`, `ui`, `auth`, `database`)

Examples:
- `/commit-feature feat api` — New API feature
- `/commit-feature fix validation` — Bug fix in validation
- `/commit-feature chore deps` — Dependency updates

## Step 1 — Parse Arguments

Extract type and optional scope from `$ARGUMENTS`:

```bash
ARGS="$ARGUMENTS"
TYPE=$(echo "$ARGS" | awk '{print $1}')
SCOPE=$(echo "$ARGS" | awk '{print $2}')
```

Validate type is one of: `feat`, `fix`, `chore`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `revert`

If invalid or missing:
```
❌ Invalid or missing commit type

Usage: /commit-feature <type> [scope]

Valid types:
  feat      - New feature
  fix       - Bug fix
  chore     - Maintenance tasks
  docs      - Documentation changes
  style     - Code style/formatting
  refactor  - Code restructuring
  perf      - Performance improvements
  test      - Test additions/changes
  build     - Build system changes
  ci        - CI/CD changes
  revert    - Revert previous commit

Example: /commit-feature feat api
```

## Step 2 — Check Git Status

```bash
git status --porcelain
```

If no changes found:
```
ℹ️ No changes to commit

Working directory is clean. Make some changes first.
```

## Step 3 — Show Staged and Unstaged Changes

Display what will be committed:

```bash
echo "📋 Files to be committed:"
git status --short

echo ""
echo "📊 Summary of changes:"
git diff --stat
```

## Step 4 — Stage All Changes

```bash
git add .
```

Confirm staging:
```bash
git status --short
```

## Step 5 — Analyze Changes for Commit Message

Read the git diff to understand what was changed:

```bash
git diff --cached
```

Based on the diff, create a concise commit message following conventional commit format:

**Format:**
```
<type>[(scope)]: <subject>

[optional body]

[optional footer]
```

**Rules:**
- Subject: Imperative mood, lowercase, no period, max 50 chars
- Body: Explain WHAT and WHY (not HOW), wrap at 72 chars
- Footer: Breaking changes, issue references

**Examples:**
```
feat(api): add vulnerability triage endpoint

Implements POST /api/triage for bulk finding updates with
analysis state, justification, and response tracking.

Closes #123
```

```
fix(validation): handle null values in SBOM parser

Prevents TypeError when component.version is undefined in
CycloneDX SBOM processing.
```

```
chore(deps): update prisma to 5.20.0

Includes performance improvements for PostgreSQL queries
and fixes KV cache invalidation race condition.
```

## Step 6 — Get Current Branch and PR

```bash
BRANCH=$(git branch --show-current)
echo "Current branch: $BRANCH"

# Check if PR exists
PR_JSON=$(gh pr list --head "$BRANCH" --base main --json number,title,url,state --limit 1)
PR_NUMBER=$(echo "$PR_JSON" | jq -r '.[0].number // empty')
PR_URL=$(echo "$PR_JSON" | jq -r '.[0].url // empty')
```

## Step 7 — Create Commit (No Co-Author)

Create commit with the conventional message (WITHOUT Claude co-authoring):

```bash
git commit -m "$(cat <<'EOF'
<type>[(scope)]: <subject>

<optional body>

<optional footer>
EOF
)"
```

**CRITICAL:** Do NOT include `Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>` line.

Confirm commit created:
```bash
git log -1 --oneline
```

## Step 8 — Push to Origin

```bash
git push origin "$BRANCH"
```

If push fails (no upstream), set upstream and push:
```bash
git push -u origin "$BRANCH"
```

## Step 9 — Collect Session Context

Gather detailed information for PR comment:

### 9.1 — Get Session Todos

Use TaskList tool to retrieve all tasks with their status, subjects, and descriptions.

Categorize by status:
- ✅ **Completed** — Tasks marked as completed
- 🔄 **In Progress** — Tasks currently being worked on
- ⏳ **Pending** — Tasks not yet started
- ⚠️ **Blocked** — Tasks with unresolved blockers

### 9.2 — Identify Known Issues Outside Session Scope

Review the conversation history to identify:
- Pre-existing bugs or issues mentioned but not fixed
- Technical debt noted during implementation
- Code patterns that should be improved later
- Performance concerns identified but deferred
- Security considerations for future work
- Missing tests or documentation
- Commented "TODO" or "FIXME" in changed files

Search for these patterns in the diff:
```bash
git diff --cached | grep -E "(TODO|FIXME|XXX|HACK|NOTE:|WARNING:)"
```

### 9.3 — Extract Implementation Details

From the conversation and code changes, document:
- **Approach taken**: High-level implementation strategy
- **Key decisions**: Why certain patterns or libraries were chosen
- **Trade-offs**: What was prioritized and what was deferred
- **Edge cases**: Known limitations or boundary conditions
- **Dependencies**: New packages or services added
- **Configuration**: Environment variables or settings changed

## Step 10 — Create or Update PR Comment

If PR exists (`$PR_NUMBER` is not empty), add detailed comment:

```bash
gh pr comment "$PR_NUMBER" --body "$(cat <<'EOF'
## 🤖 Session Summary

### Commit Details
**Type:** <type>
**Scope:** <scope>
**Message:** <commit subject>

---

### 📝 Implementation Context

#### What Changed
<Detailed explanation of the changes in this commit, including:>
- Key functionality added/modified
- Files affected and why
- Integration points with existing code
- Data model or API changes

#### Approach & Decisions
<Document the implementation approach:>
- Why this solution was chosen over alternatives
- Technical decisions made during implementation
- Libraries or patterns introduced
- Performance or security considerations

#### Testing Notes
<Testing considerations:>
- Test coverage added
- Manual testing performed
- Edge cases validated
- Known test gaps (if any)

---

### ✅ Session Tasks

<If tasks exist, format as markdown checklist grouped by status>

#### Completed
- [x] Task 1
- [x] Task 2

#### In Progress
- [ ] Task 3

#### Pending
- [ ] Task 4

<If no tasks: "No tracked tasks for this session">

---

### ⚠️ Known Issues & Technical Debt

<List issues identified but not addressed in this commit:>

#### Pre-Existing Issues
- Issue 1: Description and why it wasn't addressed
- Issue 2: Description and potential impact

#### Deferred Work
- TODO 1: What needs to be done and why it was deferred
- TODO 2: Suggested approach for future implementation

#### Code Quality Notes
- Pattern 1: Existing pattern that could be improved
- Pattern 2: Technical debt to address in future refactor

<If none: "No significant issues identified outside session scope">

---

### 🔗 Related Work

<If applicable, link to:>
- Related issues: #123, #456
- Documentation: Link to relevant docs
- Design docs: Link to technical design
- Dependent PRs: #789

---

<Timestamp>
📅 Generated: <current date/time>
🤖 [Claude Code](https://claude.com/claude-code) Session
EOF
)"
```

If NO PR exists:
```
ℹ️ No PR found for branch '$BRANCH'

Changes committed and pushed. Create a PR with:
  gh pr create --base main --fill

The session context can be added as a comment after PR creation.
```

## Step 11 — Final Summary

Output comprehensive summary:

```
╔═══════════════════════════════════════════════════╗
║           COMMIT & PUSH SUMMARY                   ║
╚═══════════════════════════════════════════════════╝

COMMIT:
  Type:     <type>
  Scope:    <scope>
  Message:  <subject>
  Hash:     <commit sha>

BRANCH:     <branch name>
STATUS:     ✅ Committed and pushed

FILES CHANGED:
  Modified:   <count>
  Added:      <count>
  Deleted:    <count>

PULL REQUEST:
  Number:     #<pr number>
  URL:        <pr url>
  Comment:    ✅ Added detailed session context

SESSION CONTEXT:
  ✅ Completed tasks:     <count>
  🔄 In progress:         <count>
  ⏳ Pending:             <count>
  ⚠️ Known issues:        <count>
  📝 Implementation details included

NEXT STEPS:
  • Review PR comments for full session context
  • Address any remaining TODOs or known issues
  • Ensure CI/CD checks pass
  • Request review when ready to merge
```

## Error Handling

### Invalid Commit Type
```
❌ Invalid commit type: '<type>'

Must be one of: feat, fix, chore, docs, style, refactor, perf, test, build, ci, revert
```

### No Changes to Commit
```
ℹ️ Working directory is clean

No changes to commit. Make changes first, then run:
  /commit-feature <type> [scope]
```

### Push Failed
```
❌ Failed to push to origin

Possible causes:
1. Network connectivity issues
2. Authentication required
3. Remote branch conflicts
4. No upstream configured

Try:
  git push -u origin <branch>
  # or resolve conflicts with:
  git pull --rebase origin <branch>
```

### No PR Found
```
ℹ️ No open PR found for branch '<branch>'

Changes committed and pushed successfully. Create PR with:
  gh pr create --base main --fill

Run /commit-feature again after PR creation to add session context.
```

## Usage Examples

```bash
# Feature commit with scope
/commit-feature feat api
# → Creates: "feat(api): add triage endpoint"
# → Pushes to origin
# → Adds detailed PR comment with session context

# Bug fix without scope
/commit-feature fix
# → Creates: "fix: handle null values in parser"
# → Pushes and comments on PR

# Chore with dependency scope
/commit-feature chore deps
# → Creates: "chore(deps): update prisma to 5.20.0"
# → Documents dependency changes in PR comment

# Documentation update
/commit-feature docs
# → Creates: "docs: update API authentication guide"
# → Links to related documentation in PR comment
```

## Conventional Commit Reference

| Type | Description | Example |
|------|-------------|---------|
| `feat` | New feature | feat(auth): add OAuth2 support |
| `fix` | Bug fix | fix(validation): prevent XSS in input |
| `chore` | Maintenance | chore(deps): upgrade dependencies |
| `docs` | Documentation | docs(api): update endpoint examples |
| `style` | Formatting | style: apply prettier to codebase |
| `refactor` | Code restructure | refactor(db): extract query helpers |
| `perf` | Performance | perf(api): optimize database queries |
| `test` | Tests | test(auth): add integration tests |
| `build` | Build system | build: update webpack config |
| `ci` | CI/CD | ci: add automated deployment |
| `revert` | Revert commit | revert: revert "add feature X" |

## Safety Notes

- **No Claude co-authoring**: Commits will NOT include `Co-Authored-By` line
- **Review before push**: Always verify staged changes match intent
- **PR comments are append-only**: Multiple runs will add multiple comments
- **Session context is detailed**: PR comments include implementation notes, TODOs, and known issues
- **Push is automatic**: Changes will be pushed immediately after commit
- **Conventional commits**: Follow the standard for automated changelog generation
- **No destructive operations**: This skill only adds commits and comments
