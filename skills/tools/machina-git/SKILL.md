---
name: machina-git
description: Git workflow for machina-meta workspace with submodules. Use ALWAYS for ALL git operations including commit, push, pull, status, diff, add, checkout, branch, tag, log, show, submodule updates. Enforces working directory safety, conventional commits, push confirmation, secret scanning, and commit readiness evaluation. MANDATORY - invoke this skill via Skill tool before ANY bash git command. DO NOT bypass this skill.
---

# Machina Git Skill

Git workflow assistance specifically designed for the machina-meta workspace with its submodule architecture.

**This skill governs ALL git operations** including status checks, commits, pushes, and multi-repo workflows.

## ⚠️ CRITICAL: MANDATORY SKILL USAGE

**DO NOT BYPASS THIS SKILL. EVER.**

**100% of git commits MUST use this skill. NO EXCEPTIONS.**

If you attempt to use the Bash tool directly for ANY git operation (status, diff, add, commit, push, etc.), you are violating critical safety protocols for this workspace.

**Before ANY git operation:**
1. **STOP** - Do not use Bash tool
2. **INVOKE** - Call `Skill` tool with `skill: "machina-git"`
3. **LET THE SKILL HANDLE IT** - The skill will execute the proper workflow

**Why this is mandatory:**
- machina-meta has 5 independent git repositories (submodules)
- Running git commands in wrong directory corrupts repository state
- This skill enforces: working directory safety, security scanning, commit readiness evaluation, atomic commits
- Bypassing this skill has caused production issues in the past

**User expectations:**
- When user says "commit this" or any git-related request → IMMEDIATELY invoke machina-git skill
- Do not perform git operations yourself
- Do not "help" by running git commands via Bash tool

**This is a hard requirement. Treat any attempt to bypass this skill as a critical error.**

## When to Use This Skill

This skill activates for ALL git-related requests:
- "commit these changes"
- "commit with message"
- "prepare a commit"
- "stage and commit"
- "check git status"
- "show git diff"
- "push to remote"
- "check repo status"
- "help me with git"
- "draft a commit message"
- Any git command or git workflow question

## Skill Activation Notice

**CRITICAL: Whenever this skill is employed, ALWAYS emit this message to the user first:**

```
NOTE: Using machina-git skill for this operation.
```

This informs the user that strict git safety protocols are being applied.

## Core Principles (from CLAUDE.md)

### 0. Submodule Update Policy (CRITICAL)

**⚠️ CRITICAL: DO NOT UPDATE THE SUBMODULE REPO REFERENCES. THESE ARE DISABLED FOR NOW.**

- **NEVER** include submodule pointer updates in your commits (e.g., `git add repos/dem2`).
- **DO NOT** attempt to "track" submodule changes in the parent repository.
- This policy is **DISABLED** until further notice. Commit your changes within the submodule repositories themselves, but do NOT update the workspace root's tracking of those submodules.

### 1. Working Directory Safety (CRITICAL)

**⚠️ CRITICAL WARNING: ALWAYS cd and run git command in the SAME Bash tool call.**

**The cd command does NOT persist between Bash tool invocations due to the SessionStart hook. You MUST combine cd and git command using `&&` in a SINGLE Bash call.**

- ✅ CORRECT: `cd /full/path/to/target/repo && git status` (single Bash call)
- ❌ WRONG: Run `cd /path` in one Bash call, then `git status` in another Bash call
- ❌ NEVER assume you are already in the correct directory
- ❌ This applies to ALL git commands: status, diff, add, commit, **push**, pull, tag, log, show, etc.
- ⚠️ **ESPECIALLY `git push` - failure to cd first can push to wrong repository!**

```bash
# CORRECT - cd and git command in SAME Bash call using &&
cd repos/dem2 && git status

cd repos/dem2 && git commit -m "message"

cd repos/dem2 && git push origin feature/branch

# CORRECT - Multiple git commands after single cd
cd repos/dem2 && git status && git diff

# WRONG - Never assume current directory
git status  # Dangerous! Which repo am I in?
git push    # CATASTROPHIC! Could push to wrong repo!

# WRONG - Separate Bash tool calls (cd won't persist!)
# First Bash call:
cd repos/dem2
# Second Bash call (will run in workspace root, NOT repos/dem2):
git status  # ❌ WILL RUN IN WRONG DIRECTORY!
```

**Why this matters:**
- machina-meta has 5 independent git repositories (root + 4 submodules)
- Each submodule has independent git history
- Running git commands in wrong repo corrupts state
- Path confusion causes commits in wrong repo

**Repository Paths (relative to workspace root):**
- Workspace root: `.` or omit cd entirely
- Backend: `repos/dem2`
- Frontend: `repos/dem2-webui`
- Infrastructure: `repos/dem2-infra`
- Catalog: `repos/medical-catalog`

**Examples:**
```bash
# Backend
cd repos/dem2 && git status

# Frontend
cd repos/dem2-webui && git status

# Workspace root (no cd needed, already at workspace root)
git status
```

### 2. Push Policy (CRITICAL)

**NEVER push to git repositories automatically.**

- Always commit changes locally first
- **DO NOT** run `git push` without explicit user confirmation
- **DO NOT** push as part of automated workflows
- Ask: "Should I push these changes to the remote repository?"
- Only push if user explicitly confirms ("yes", "push", "push it")

### 3. Commit Message Policy (CRITICAL)

**NEVER add Claude Code attribution or co-authorship credits.**

- **DO NOT** include `🤖 Generated with [Claude Code]`
- **DO NOT** include `Co-Authored-By: Claude <noreply@anthropic.com>`
- Write clean, concise conventional commit messages
- Follow repository's existing commit style

### 4. Atomic Commits (CRITICAL)

**Unrelated changes should be committed separately and atomically whenever possible.**

- Each commit should represent a single logical change
- Don't mix unrelated changes in one commit (e.g., adding a feature + updating documentation for a different feature)
- Separate concerns even when working on multiple things simultaneously
- Makes git history cleaner and easier to review/revert

**Examples:**

✅ **Good - Atomic commits:**
```bash
# Commit 1: Add new skill
git add .claude/skills/machina-git/
git commit -m "feat: add machina-git custom skill"

# Commit 2: Separate unrelated change
git add .claude/skills/kubernetes/
git commit -m "feat: add kubernetes skill for K8s operations"
```

❌ **Bad - Mixed concerns:**
```bash
# BAD - Two unrelated features in one commit
git add .claude/skills/machina-git/ .claude/skills/kubernetes/
git commit -m "feat: add machina-git and kubernetes skills"
```

**When to separate commits:**
- Different features or bug fixes
- Different subsystems or components
- Documentation updates for different topics
- Dependency updates vs feature changes
- Refactoring vs new functionality

**When commits can be combined:**
- Changes are tightly coupled (e.g., API + corresponding tests)
- Documentation for the code being changed
- Multiple files implementing one logical feature

## Multi-Repo Status Checks

When checking status across multiple repositories, always show which repo you're checking:

```bash
# CORRECT - Each cd and git command in same Bash call
echo "=== Checking machina-meta (workspace root) ===" && git status

echo "=== Checking repos/dem2 (backend) ===" && cd repos/dem2 && git status

echo "=== Checking repos/dem2-webui (frontend) ===" && cd repos/dem2-webui && git status

echo "=== Checking repos/dem2-infra (infrastructure) ===" && cd repos/dem2-infra && git status

echo "=== Checking repos/medical-catalog (catalog service) ===" && cd repos/medical-catalog && git status
```

**Alternative:** Use workspace-level just commands:
```bash
# Already at workspace root, no cd needed
just repo-status    # Git status across all repos
just repo-branches  # Show current branches
just repo-diff      # Show diffs across all repos
```

## Workflow Pattern

**⚠️ CRITICAL REQUIREMENT FOR ALL STEPS:**

Every git command MUST be combined with its `cd` command in the SAME Bash tool call using `&&`:
- ✅ CORRECT: `cd repos/dem2 && git status` (single Bash call)
- ❌ WRONG: First call `cd repos/dem2`, then separate call `git status`
- **Why:** The SessionStart hook resets working directory for each Bash invocation
- **Impact:** Running cd and git in separate calls = infinite loop or wrong repository operations

### Step 0: Emit Skill Activation Notice

**ALWAYS start by informing the user:**
```
NOTE: Using machina-git skill for this operation.
```

### Step 1: Identify Target Repository and Ensure Clean Index (CRITICAL)

**Before doing ANYTHING else, identify the target repository and ensure its index is clean.**

1. **Identify the target repository:**
   ```
   User mentions: "commit the backend changes" → Target: repos/dem2
   User mentions: "commit CLAUDE.md" → Target: workspace root (machina-meta)
   ```

2. **CRITICAL: Ensure index is clean (check for existing staged files):**
   ```bash
   # Check for existing staged files in the target repo (cd and git in same call)
   cd repos/dem2 && git status
   ```
   **If output is NOT empty:**
   - **STOP IMMEDIATELY.**
   - **Alert the user:** "⚠️ Warning: There are files already staged for commit in this repository. Do you want to include them, or should I unstage them first?"
   - **DO NOT PROCEED** until the user confirms or you have unstaged the unwanted files.
   - **NEVER** blindly commit existing staged files unless explicitly instructed.

If unclear which repo, ask: "Which repository should I work with?"

### Step 2: Review Current State

**CRITICAL: cd and git commands must be in same Bash call**

**REQUIRED: Always examine the output of `git status` before proceeding.**
This ensures you see the exact state of untracked (??) vs modified (M) files without ambiguity.

```bash
# Check machine-readable status (cd and git in same call)
cd repos/dem2 && git status

# Also check human-readable status for context
cd repos/dem2 && git status
cd repos/dem2 && git diff
```

Show the user what will be committed. Ask if this looks correct.

### Step 4: Stage Changes

**ALWAYS stage specific files explicitly. NEVER use `git add .` or `git add -A`.**

```bash
# CORRECT - cd and git add in same Bash call
cd repos/dem2 && git add path/to/file1.py path/to/file2.py

# WRONG - Never use blanket staging
cd repos/dem2 && git add .   # BAD
cd repos/dem2 && git add -A  # BAD
```

⚠️ **NEVER stage sensitive files:**
- `.env` files containing secrets
- Credential files (`.pem`, `.key`, service account JSON, etc.)
- Configuration files with sensitive data

⚠️ **Why no `git add -A` or `git add .`:**
- Can accidentally stage unintended files
- May include sensitive files (.env, credentials)
- Less explicit and harder to review
- Always specify exactly what should be committed

### Step 4.5: Security Scan Staged Changes (CRITICAL)

**ALWAYS scan staged diff for secrets before committing. STOP if detected.**

```bash
# Check staged diff for potential secrets (cd and git in same call)
cd repos/dem2 && git diff --cached
```

**Scan the output for these patterns (case-insensitive):**
- API keys: `api_key`, `apikey`, `api-key`, `GOOGLE_API_KEY`, etc.
- Secret keys: `secret_key`, `secret`, `SECRET_KEY`, `AWS_SECRET_ACCESS_KEY`, etc.
- Tokens: `token`, `auth_token`, `access_token`, `bearer`, `jwt`, etc.
- Passwords: `password`, `passwd`, `pwd`, `pass =`, etc.
- Private keys: `PRIVATE KEY`, `-----BEGIN`, `.pem`, `.key` file contents
- Database URLs: `postgres://`, `mysql://`, `mongodb://` with credentials
- OAuth credentials: `client_secret`, `client_id` with actual values
- Service account keys: `private_key_id`, `"type": "service_account"`
- Authentication headers: `Authorization:`, `X-API-Key:`

**Pattern indicators to look for:**
```
+    API_KEY = "sk-..."
+    SECRET_KEY = "abc123..."
+    password = "my-secret-pass"
+    token: "eyJhbGciOiJ..."
+    private_key: "-----BEGIN PRIVATE KEY-----"
+    DATABASE_URL = "postgres://user:password@host/db"
```

**If ANY secrets are detected:**
1. **STOP immediately** - Do not proceed with commit
2. Alert the user: "⚠️ SECURITY WARNING: Potential secrets detected in staged changes!"
3. Show the detected lines
4. Ask: "These look like secrets. Should I unstage these files?"
5. Wait for user confirmation before proceeding

**Example:**
```bash
# After git diff --cached, if you see:
+    GOOGLE_API_KEY = "AIzaSyC..."
+    secret_token = "abc123xyz..."

# STOP and alert:
⚠️ SECURITY WARNING: Potential secrets detected in staged changes!

Lines 45-46 in config.py:
+    GOOGLE_API_KEY = "AIzaSyC..."
+    secret_token = "abc123xyz..."

These look like secrets. Should I unstage these files?
```

### Step 4.6: Evaluate Commit Readiness (CRITICAL)

**ALWAYS perform this evaluation before crafting commit message. Present findings to user.**

Analyze the staged changes and answer these questions:

**1. Impact/Value Assessment:**
```
What is the impact or value of this commit?
- [ ] Fixes critical bug affecting users
- [ ] Adds new feature with user benefit
- [ ] Improves performance or reliability
- [ ] Refactors code for maintainability
- [ ] Updates documentation
- [ ] Other: ___

Estimated Impact: [HIGH / MEDIUM / LOW]
Reasoning: [Brief explanation]
```

**2. Testing Status:**
```
Has testing been performed?
- [ ] Unit tests run and passing
- [ ] Integration tests run and passing
- [ ] Manual testing performed
- [ ] End-to-end testing completed
- [ ] No testing performed (explain why)

Testing Status: [PASSED / PARTIAL / NONE]
Details: [What was tested, results]
```

**3. Static Analysis Status:**
```
Has static analysis been performed?
- [ ] Linting (ruff check --fix) - passed
- [ ] Type checking (mypy --strict) - passed
- [ ] Code formatting (ruff format) - applied
- [ ] Shell script checking (shellcheck) - passed (for .sh files)
- [ ] Security scanning - passed
- [ ] Other checks: ___

Static Analysis: [PASSED / PARTIAL / NONE]
Issues: [Any warnings or errors to address]
```

**4. Problem Solved:**
```
What problem does this solve?
- Root cause: [What was broken or why change is needed]
- Symptoms: [How problem manifested]
- Solution: [How this change addresses it]
- References: [PROBLEMS.md, docs/plans/, Jira tickets]
```

**5. Risk Assessment:**
```
What is the potential risk in pushing this commit?
- [ ] Breaking changes to API/contracts
- [ ] Database migration required
- [ ] Dependency changes
- [ ] Performance impact
- [ ] Security implications
- [ ] Affects critical path
- [ ] Low risk / no breaking changes

Risk Level: [HIGH / MEDIUM / LOW]
Mitigation: [How to reduce risk, rollback plan]
```

**Example Evaluation:**

```
=== COMMIT READINESS EVALUATION ===

1. Impact/Value:
   ✅ Fixes critical bug: symptoms and conditions lost during document processing
   Estimated Impact: HIGH
   Reasoning: Core functionality broken, affects all document processing

2. Testing Status:
   ✅ Unit tests added and passing (test_prepare_resource_data.py)
   ✅ Manual testing with sample documents confirmed fix
   Testing Status: PASSED
   Details: Verified symptoms/conditions now preserved in graph

3. Static Analysis:
   ✅ Linting: ruff check --fix passed
   ✅ Type checking: mypy --strict passed
   ✅ Formatting: ruff format applied
   ✅ Shell scripts: shellcheck passed
   Static Analysis: PASSED

4. Problem Solved:
   Root cause: Duplicate PreparedResourceData assignment overwriting data
   Symptoms: Symptoms and conditions missing from graph after processing
   Solution: Removed duplicate assignment, preserved all resource types
   References: PROBLEMS.md "Missing symptoms in graph" [SOLVED]

5. Risk Assessment:
   ✅ No breaking changes to API
   ✅ No database migration required
   ✅ Backwards compatible
   Risk Level: LOW
   Mitigation: Changes isolated to single method, well-tested

=== RECOMMENDATION: READY TO COMMIT ===
```

**If evaluation reveals issues:**
- Missing tests → Run tests or document why not needed
- Static analysis failures → Fix issues before committing
- High risk → Discuss mitigation with user
- Unclear problem → Clarify what this solves

**Present evaluation to user and ask: "Does this evaluation look correct? Should I proceed with the commit?"**

### Step 5: Craft Commit Message

Use conventional commit format with structured body for significant changes.

**Format:** `<type>(<scope>): <summary>`

**Types:**
- `feat` - New feature
- `fix` - Bug fix
- `docs` - Documentation only
- `style` - Formatting, missing semicolons, etc.
- `refactor` - Code change that neither fixes a bug nor adds a feature
- `perf` - Performance improvement
- `test` - Adding or fixing tests
- `chore` - Maintenance tasks, dependency updates
- `ci` - CI/CD changes
- `build` - Build system changes

**Scope (optional but recommended):**
- Component, module, or area affected
- Examples: `api`, `agents`, `docproc`, `auth`, `infra`, `scripts`

**Summary Line:**
- Concise summary in imperative mood ("add" not "added")
- No period at the end
- Lowercase first letter
- Max 72 characters
- For phased work, prefix with "Phase N - " (e.g., "feat: Phase 2 - Implement automatic session management")

**When to Add Multi-Line Body:**

**MANDATORY** for:
- `feat` (New features)
- `fix` (Bug fixes)
- `refactor` (Significant code changes)
- `perf` (Performance improvements)

**OPTIONAL** for:
- `docs`, `chore`, `style`, `test`, `ci`, `build` (unless the change is significant or complex)

**Required Body Structure:**

For mandatory multi-line commits, use this structure (based on project conventions):

1.  **Context/Problem**:
    - A paragraph explaining *why* this change is needed.
    - What was the root cause or missing functionality?

2.  **Changes**:
    - A bulleted list (`- `) of specific implementation details.
    - Use "Changes:" header if helpful for clarity.

3.  **Verification/Impact**:
    - How was this tested?
    - What is the user benefit?

4.  **Footer**:
    - `Related: PROBLEMS.md ...`
    - `Related: docs/plans/... ...`

**Examples:**

Simple change:
```
feat(api): add patient biomarker reconciliation endpoint
```

Bug fix with detail:
```
fix(docproc): disable aggressive parenthetical stripping in biomarker names

Disable Maxim's _PARENTHETICAL_SUFFIX_PATTERN regex that was incorrectly
stripping integral parentheticals like "Lp(a)" → "Lp".

Changes:
- Add _ENABLE_PARENTHETICAL_STRIPPING flag (default: False)
- Conditionally apply parenthetical stripping only when flag is True
- Rely on LLM extraction prompt for abbreviation handling

Impact:
- Lp(a) now correctly preserved with parentheses
- Extraction accuracy improved from 63.8% to 65.2% (44→45 exact matches)
- No regressions introduced

Testing:
- Verified Lp(a) extracted as exact match: "<15 mg/dL"
- Generated comparison reports confirming fix

Related:
- Issue discovered: 2025-12-19 (David Beal)
- Root cause: Maxim's Boston Heart processor (2025-12-11)
```

Feature with phasing:
```
feat: Phase 2 - Implement automatic session management for med-agent

Completed Phase 2: Investigated session API and implemented automatic session creation.

Root Cause:
- SessionNotFoundError: API requires pre-existing sessions
- Original query_agent() generated UUID without creating session

Solution Implemented:
1. Added create_session() function
2. Added list_sessions() function
3. Updated query_agent() to auto-create sessions

Testing Results:
✅ Manual session creation successful
✅ Query with manual session: HTTP 200
✅ Automatic session creation successful

Related: PROBLEMS.md "Med-agent endpoint" [SOLVED]
Follows: 2859d1ca (Phase 1 diagnostics)
```

Documentation update:
```
docs: add comprehensive document processing monitoring guide

- Created DOCUMENT_PROCESSING.md with complete API reference
- Tested shell helper functions (curl_api.sh)
- Pipeline architecture and processing stages
- Common troubleshooting workflows

All example commands tested against running backend (2024-12-30)
```

### Step 6: Execute Commit

**Only proceed if:**
- Security scan passed (no secrets detected)
- Commit readiness evaluation looks good
- User confirmed to proceed

```bash
# cd and git commit in same Bash call
cd repos/dem2 && git commit -m "type(scope): description"
```

**For multi-line commits:**
```bash
# cd and git commit in same Bash call with heredoc
cd repos/dem2 && git commit -m "$(cat <<'EOF'
feat(api): add patient biomarker reconciliation endpoint

- Implement POST /api/v1/biomarkers/reconcile
- Add validation for incoming biomarker data
- Integrate with medical-catalog service for normalization
- Add comprehensive error handling
EOF
)"
```

### Step 7: Verify Commit

```bash
# cd and git log in same Bash call
cd repos/dem2 && git log -1 --stat
```

Show the user the commit that was created and confirm it matches expectations.

### Step 8: Push Confirmation (If Needed)

**Only if user explicitly requests pushing:**

1. Verify commit is correct
2. Ask: "Should I push these changes to the remote repository?"
3. Wait for explicit confirmation
4. If confirmed, execute:
   ```bash
   cd repos/dem2 && git push
   ```

## Multi-Line Commit Messages

For significant `feat`, `fix`, `refactor`, and `perf` commits, use this multi-line template based on the project standard.

**Template:**
```bash
git commit -m "$(cat <<'EOF'
<type>(<scope>): <summary>

Paragraph explaining context, problem, or root cause. Why is this change needed?

Changes:
- Specific implementation detail 1
- Specific implementation detail 2
- Specific implementation detail 3

Verification:
- How was this tested?
- What was the outcome?

Impact:
- Quantifiable benefit (e.g., performance, accuracy)
- Risk assessment (if applicable)

Related: PROBLEMS.md "Problem Title" [SOLVED]
Related: docs/plans/plan-name.md
EOF
)"
```

**Important Notes:**
- Section headers (e.g., "Changes:", "Testing Results:") are optional labels for clarity
- Use only the sections that are relevant to your commit
- Always include "Related:" section when the commit resolves/references PROBLEMS.md, docs/plans/, or Jira tickets
- Quantify impact when possible ("Time savings: 15+ minutes", "Accuracy: 97.1%")

**Example - Bug Fix:**
```bash
git commit -m "$(cat <<'EOF'
fix(medical-data-engine): remove duplicate PreparedResourceData assignment

The duplicate assignment was overwriting prepared after setting biomarkers,
symptoms, and conditions, causing symptoms and conditions to be lost.

Before (buggy):
  prepared.biomarkers = data.biomarkers
  prepared.symptoms = data.basic_resources.symptoms
  prepared.conditions = data.basic_resources.conditions
  prepared = PreparedResourceData(biomarkers=data.biomarkers)  # ❌ Overwrites!

After (fixed):
  prepared.biomarkers = data.biomarkers
  prepared.symptoms = data.basic_resources.symptoms  # ✅ Preserved
  prepared.conditions = data.basic_resources.conditions  # ✅ Preserved

Impact: Symptoms and conditions are now correctly included in prepared data.

Related: PROBLEMS.md "Missing symptoms in graph" [SOLVED]
EOF
)"
```

**Example - Feature with Phasing:**
```bash
git commit -m "$(cat <<'EOF'
feat: Phase 2 - Implement automatic session management for med-agent

Completed Phase 2: Investigated session API and implemented automatic session creation.

Root Cause:
- SessionNotFoundError: API requires pre-existing sessions
- Original query_agent() generated UUID without creating session

Solution Implemented:
1. Added create_session() function:
   - POST /api/v1/med-agent/users/sessions
   - Requires X-Patient-Context-ID header

2. Added list_sessions() function:
   - GET /api/v1/med-agent/users/sessions

3. Updated query_agent() function:
   - Automatically creates session if session_id not provided
   - Enables conversation continuity across queries

Testing Results:
✅ Manual session creation successful
✅ Query with manual session: HTTP 200
✅ Automatic session creation successful
✅ Query with auto-created session: HTTP 200

The med-agent endpoint is now fully functional. Users can query processed
medical documents without manual session management.

Related: PROBLEMS.md "Med-agent endpoint" [SOLVED]
Follows: 2859d1ca (Phase 1 diagnostics)
EOF
)"
```

**Example - Refactoring:**
```bash
git commit -m "$(cat <<'EOF'
refactor(agents): consolidate tool calling patterns

Unified tool execution across all agent types:
- Extract common validation logic to SafeAgentTool
- Standardize error handling across medical agents
- Add type safety for tool parameters
- Improve logging for tool invocations

Benefits:
- Reduces code duplication by ~200 lines
- Makes system more maintainable
- Improves error diagnostics

Related: PROBLEMS.md "Agent tool pattern inconsistency" [SOLVED]
EOF
)"
```

## Working with Submodules

### Committing in a Submodule

```bash
# All steps in single Bash call using &&
cd repos/dem2 && \
  git add services/medical-agent/src/machina/medical_agent/agents/config.yml && \
  git commit -m "feat(agents): update medical agent prompt template" && \
  git push origin dev
```

### Updating Parent Workspace to Track Submodule Change

After committing in a submodule, the parent workspace needs to track the new commit:

```bash
# Already at workspace root, stage and commit submodule update
git add repos/dem2 && \
  git commit -m "chore: update dem2 submodule to latest dev" && \
  git push
```

## Repository-Specific Commit Patterns

### machina-meta (workspace root)

Common commit types:
- `docs:` - Updates to CLAUDE.md, README.md, docs/
- `chore:` - Submodule updates, dependency updates
- `ci:` - Changes to justfile, scripts/
- `feat:` - New workspace-level features

### repos/dem2 (backend)

Common commit types:
- `feat(agents):` - Agent changes
- `feat(api):` - API endpoint changes
- `fix(biomarkers):` - Biomarker processing fixes
- `refactor(graph):` - Graph model refactoring
- `test:` - Test additions/fixes
- `chore(deps):` - Dependency updates

### repos/dem2-webui (frontend)

Common commit types:
- `feat(ui):` - New UI components
- `fix(auth):` - Authentication fixes
- `style:` - CSS/styling changes
- `refactor:` - Component refactoring
- `chore(deps):` - Dependency updates

### repos/dem2-infra (infrastructure)

Common commit types:
- `feat(k8s):` - Kubernetes manifest changes
- `feat(argocd):` - ArgoCD application changes
- `chore:` - Preview environment updates
- `ci:` - GitHub Actions changes

### repos/medical-catalog (catalog service)

Common commit types:
- `feat(biomarkers):` - Biomarker catalog additions
- `fix(search):` - Search/query fixes
- `refactor:` - Code refactoring
- `chore(deps):` - Dependency updates

## Safety Checklist

Before executing any git operation:

- [ ] **Emit skill activation notice to user first**
- [ ] **⚠️ CRITICAL: cd and git command MUST be in SAME Bash tool call using `&&`**
- [ ] **⚠️ NEVER run cd in one Bash call and git in another - ESPECIALLY for `git push`**
- [ ] Use relative paths from workspace root (e.g., `cd repos/dem2 && git status`)
- [ ] Review git status and git diff (for commits)
- [ ] **Check index for existing staged files (`git status`) - Warn if present**
- [ ] Confirm changes with user (for commits)
- [ ] Ensure changes are related and atomic (separate unrelated changes)
- [ ] Stage specific files only (NEVER use `git add .` or `git add -A`)
- [ ] Verify no sensitive files are being staged (.env, credentials, keys)
- [ ] **Verify NO files will be deleted or lost (check status for 'D' or missing files)**
- [ ] **Run security scan on staged diff (git diff --cached) - STOP if secrets detected**
- [ ] **Perform commit readiness evaluation (impact, testing, static analysis, problem, risk)**
- [ ] **Present evaluation to user and get confirmation to proceed**
- [ ] Craft conventional commit message (no Claude attribution)
- [ ] Include relevant sections in body (Root Cause, Changes, Testing, Impact, Related)
- [ ] Reference PROBLEMS.md, docs/plans/, or Jira tickets in "Related:" section
- [ ] Execute commit with heredoc for multi-line
- [ ] Verify commit with git log -1 --stat
- [ ] Only push if explicitly requested and confirmed

## Anti-Patterns (DO NOT DO)

❌ **Run cd and git in separate Bash calls (CATASTROPHIC for git push):**
```bash
# CATASTROPHIC - cd won't persist to next Bash call!
# Bash call 1:
cd repos/dem2
# Bash call 2 (will run in workspace root, NOT repos/dem2!):
git push    # DISASTER - pushes wrong repository!

# CORRECT - cd and git in SAME Bash call
cd repos/dem2 && git push origin feature/branch
```

❌ **Assume current directory:**
```bash
# BAD - Never assume you're in the right place
git status  # Which repo am I in?
git push    # Could be wrong repo!

# CORRECT - Always cd && git in same call
cd repos/dem2 && git status
```

❌ **Auto-push without confirmation:**
```bash
# BAD - Never push without explicit confirmation
git commit -m "message" && git push
```

❌ **Add Claude attribution:**
```bash
# BAD - Never add attribution
git commit -m "feat: add feature

🤖 Generated with Claude Code
Co-Authored-By: Claude <noreply@anthropic.com>"
```

❌ **Use git add with . or -A:**
```bash
# BAD - Never use blanket staging
git add .
git add -A
```

❌ **Stage sensitive files:**
```bash
# BAD - Never stage secrets or credentials
git add .env
git add config/credentials.json
git add *.key
git add *.pem
```

❌ **Skip security scanning of staged diff:**
```bash
# BAD - Never commit without scanning for secrets first
git add file.py
git commit -m "message"  # Without running git diff --cached and checking for secrets
```

❌ **Commit after detecting secrets:**
```bash
# BAD - If secrets detected, STOP immediately
# After git diff --cached shows API_KEY = "sk-..."
git commit -m "message"  # NEVER proceed if secrets found
```

❌ **Use git commit with -a flag blindly:**
```bash
# BAD - Stages everything including unintended files
git commit -am "message"
```

❌ **Chain push operations:**
```bash
# BAD - No automatic chaining to push
git commit -m "message" && git push
```

❌ **Create feature branches automatically:**
```bash
# BAD - Do not create branches unless explicitly asked
git checkout -b feature/my-branch
```

❌ **Reset or squash history automatically:**
```bash
# BAD - Never reset to origin or squash unless explicitly asked
git reset --soft origin/main
git reset --hard
```

## Best Practices

✅ **Emit skill activation notice at the start of every operation**
✅ **Always combine cd and git commands in SAME Bash call using `&&`**
✅ **Use relative paths from workspace root (e.g., `repos/dem2`)**
✅ **Review changes before staging**
✅ **ALWAYS stage specific files explicitly (never `git add .` or `git add -A`)**
✅ **Verify no .env or credential files are being staged**
✅ **Run security scan on staged diff (git diff --cached) - STOP if secrets detected**
✅ **Perform commit readiness evaluation (impact, testing, static analysis, problem, risk)**
✅ **Present evaluation to user and wait for confirmation**
✅ **Draft commit message with structured body for significant changes**
✅ **Include "Related:" section referencing PROBLEMS.md, docs/plans/, Jira tickets**
✅ **Quantify impact when possible ("Time savings: 15+ minutes", "Accuracy: 97.1%")**
✅ **Use heredoc for multi-line commits**
✅ **Verify commit after creation**
✅ **Keep commits focused and atomic**
✅ **Separate unrelated changes into different commits**
✅ **For phased work, use "Phase N - " prefix in summary**
✅ **Use just repo-* commands for multi-repo operations**

## Integration with PROBLEMS.md

> **CRITICALLY IMPORTANT: TODO.md is DEPRECATED.** Do not add new items to TODO.md. Reference PROBLEMS.md and `docs/plans/` instead.

When making commits related to tracked problems:

1. Reference the PROBLEMS.md item in commit message if helpful
2. Update PROBLEMS.md to mark problems as SOLVED (with solving date)
3. Commit PROBLEMS.md changes immediately

Example workflow:
```bash
# Make code changes in submodule (cd && git in same call)
cd repos/dem2 && \
  git add services/medical-agent/ && \
  git commit -m "feat(agents): add biomarker extraction validation"

# Update PROBLEMS.md in submodule (cd && git in same call)
cd repos/dem2 && \
  git add PROBLEMS.md && \
  git commit -m "docs: mark biomarker validation problem as SOLVED"
```

## Common Scenarios

### Scenario 1: Check Status Across All Repos

```bash
# Using just command (already at workspace root)
just repo-status
```

Or manually (each cd && git in same Bash call):
```bash
echo "=== machina-meta ===" && git status --short

echo "=== repos/dem2 ===" && cd repos/dem2 && git status --short

echo "=== repos/dem2-webui ===" && cd repos/dem2-webui && git status --short

echo "=== repos/dem2-infra ===" && cd repos/dem2-infra && git status --short

echo "=== repos/medical-catalog ===" && cd repos/medical-catalog && git status --short
```

### Scenario 2: Single File Change

```
NOTE: Using machina-git skill for this operation.
```

```bash
# Step 1: Review changes (already at workspace root)
git status && git diff CLAUDE.md

# Step 2: Stage specific file
git add CLAUDE.md

# Step 3: Security scan staged diff
git diff --cached
# Review output for any secrets, API keys, tokens, passwords, etc.
# If secrets detected: STOP and alert user

# Step 4: Evaluate commit readiness
# Present evaluation to user:
=== COMMIT READINESS EVALUATION ===

1. Impact/Value:
   ✅ Updates documentation for git workflow consistency
   Estimated Impact: MEDIUM
   Reasoning: Improves developer experience and prevents git errors

2. Testing Status:
   N/A - Documentation change
   Testing Status: N/A

3. Static Analysis:
   N/A - Markdown file
   Static Analysis: N/A

4. Problem Solved:
   Root cause: Git rules scattered across multiple sections
   Solution: Consolidate into unified "Git Rules" section for easier reference
   References: N/A

5. Risk Assessment:
   ✅ No code changes
   ✅ Documentation only
   Risk Level: LOW
   Mitigation: N/A

=== RECOMMENDATION: READY TO COMMIT ===

Does this evaluation look correct? Should I proceed with the commit?

# Step 5: Commit (after user confirmation)
git commit -m "docs: consolidate git rules into unified section"

# Step 6: Verify
git log -1 --stat

# Step 7: Ask about push
# Ask: "Should I push these changes to the remote repository?"
```

### Scenario 3: Multiple Files in Backend

```bash
# All commands in single Bash call with cd
cd repos/dem2 && \
  git status && \
  git add services/medical-agent/src/machina/medical_agent/agents/medical_agent.py \
       services/medical-agent/src/machina/medical_agent/agents/config.yml && \
  git commit -m "$(cat <<'EOF'
refactor(agents): improve error handling in medical agent

- Add try/catch blocks around tool execution
- Log errors with full context
- Return structured error responses to frontend
EOF
)" && \
  git log -1 --stat
```

### Scenario 4: Preview Environment (multi-repo)

```bash
# Step 1: Tag backend (cd && commands in single Bash call)
cd repos/dem2 && git tag -f preview-my-feature
# Push tag (if confirmed):
cd repos/dem2 && git push origin preview-my-feature --force

# Step 2: Tag frontend (cd && commands in single Bash call)
cd repos/dem2-webui && git tag -f preview-my-feature
# Push tag (if confirmed):
cd repos/dem2-webui && git push origin preview-my-feature --force

# Step 3: Update infra (cd && commands in single Bash call)
cd repos/dem2-infra && git checkout -b preview/my-feature
# Make changes, commit, push (if confirmed)
```

## Protected Push Operation

Git push is a **protected operation**. Never execute `git push` without:

1. Showing the commit(s) that will be pushed:
   ```bash
   cd repos/dem2 && git log origin/main..HEAD --oneline
   ```

2. Asking explicitly: "Should I push these changes to the remote repository?"

3. Waiting for user confirmation ("yes", "push", "push it")

4. Only then executing:
   ```bash
   cd repos/dem2 && git push
   ```

For force pushes (tags, rewrites), extra caution:
```bash
# Show what will be force pushed (cd && git in same call)
cd repos/dem2 && git show preview-my-feature

# Ask: "This is a force push. Should I proceed?"
# Only if confirmed (cd && git in same call):
cd repos/dem2 && git push origin preview-my-feature --force
```

## Related Documentation

- [CLAUDE.md](../../../CLAUDE.md) - Full git rules and workspace guidance
- [docs/DEVOPS.md](../../../docs/DEVOPS.md) - Preview environments and deployment workflows
- [docs/DEVOPS_SKILLS.md](../../../docs/DEVOPS_SKILLS.md) - Other DevOps skills available

## Additional References

- [change-reports.md](references/change-reports.md) — convention for post-commit change reports under `docs/change_reports/`

## Skill Metadata

- **Version:** 1.0.0
- **Created:** 2026-01-06
- **Workspace:** machina-meta
- **Repositories:** 5 (1 root + 4 submodules)
- **Philosophy:** Safety-first, explicit actions, user confirmation for destructive operations
- **Scope:** Governs ALL git operations (status, commit, push, multi-repo workflows)
