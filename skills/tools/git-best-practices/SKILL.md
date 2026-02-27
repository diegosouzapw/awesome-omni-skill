---
name: git-best-practices
description: Git best practices for ensuring proper .gitignore setup and git workflow management. Use this skill when initializing new projects, working with package managers (npm, pip, etc.), or making git commits.
---

# Git Best Practices Skill

This skill provides comprehensive guidelines for proper git workflow management, with a focus on `.gitignore` setup and preventing accidental commits of unwanted files.

## .gitignore Management

### Critical: Always Check Before Initializing Package Managers

**BEFORE** initializing npm, pip, poetry, cargo, or any other package manager:

1. **Check if `.gitignore` exists**: Use `ls -la .gitignore` or check via file system
2. **If `.gitignore` doesn't exist**: Create it with appropriate patterns for your project type
3. **If `.gitignore` exists**: Verify it includes the necessary patterns for your package manager
4. **After creating/updating `.gitignore`**: Verify it's working with `git status` to ensure ignored files don't appear

### Required Patterns by Project Type

#### Node.js / npm / yarn / pnpm

```
node_modules/
npm-debug.log*
yarn-debug.log*
yarn-error.log*
pnpm-debug.log*
.pnpm-store/
```

#### Python / pip / poetry

```
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
env/
venv/
ENV/
env.bak/
venv.bak/
.venv
pip-log.txt
pip-delete-this-directory.txt
.pytest_cache/
.coverage
htmlcov/
*.egg-info/
dist/
build/
```

#### Rust / cargo

```
target/
Cargo.lock
```

#### Go

```
*.exe
*.exe~
*.dll
*.so
*.dylib
*.test
*.out
go.work
```

#### Java / Maven / Gradle

```
*.class
*.log
*.jar
*.war
*.ear
target/
build/
.gradle/
.idea/
*.iml
```

### Universal Patterns (All Projects)

These should be included in every `.gitignore`:

```
# Environment files
.env
.env.local
.env.*.local
.env.development
.env.production
.env.test
*.env

# Build outputs
dist/
build/
.next/
out/
*.tsbuildinfo

# OS files
.DS_Store
.DS_Store?
._*
.Spotlight-V100
.Trashes
ehthumbs.db
Thumbs.db
Desktop.ini

# Logs
*.log
logs/
npm-debug.log*
yarn-debug.log*
yarn-error.log*
pnpm-debug.log*
lerna-debug.log*

# Editor files (optional - include if not using shared IDE config)
.idea/
.vscode/
*.swp
*.swo
*~
.project
.classpath
.settings/

# Temporary files
*.tmp
*.temp
.cache/
```

## Git Workflow Best Practices

### Before Committing

1. **Always check `git status`** before committing to see what will be committed:

   ```bash
   git status
   ```

2. **Review the output carefully**:
   - Ensure no sensitive files are staged (`.env`, secrets, API keys)
   - Verify `node_modules/` or other build artifacts aren't included
   - Check that only intended files are staged

3. **If unwanted files appear**:
   - Check if `.gitignore` exists and has the correct patterns
   - Update `.gitignore` if needed
   - Remove files from staging: `git reset HEAD <file>`
   - Verify with `git status` again

### Commit Message Format

Use the existing project format:

```
feat: TASK-{id} - {description}
```

Examples:

- `feat: TASK-123 - Add user authentication`
- `feat: TASK-456 - Fix memory leak in data processor`

### Never Commit

- **Sensitive files**: `.env`, `.env.local`, any file containing secrets, API keys, passwords
- **Build artifacts**: `node_modules/`, `dist/`, `build/`, compiled binaries
- **OS files**: `.DS_Store`, `Thumbs.db`
- **Large binary files**: Unless using Git LFS (Large File Storage)
- **IDE-specific files**: Unless the project explicitly includes them in version control

### Verifying .gitignore Effectiveness

After creating or updating `.gitignore`:

1. Run `git status` to see what files git is tracking
2. If ignored files still appear, they may have been previously tracked:
   ```bash
   # Remove from git cache (but keep local files)
   git rm --cached <file>
   # Or for directories:
   git rm -r --cached <directory>
   ```
3. Commit the removal: `git commit -m "chore: Remove tracked files that should be ignored"`
4. Verify again with `git status`

## Project-Specific Considerations

### When Initializing a New Project

1. Create `.gitignore` **before** running `npm init`, `pip install`, etc.
2. Include patterns for your package manager
3. Initialize git: `git init` (if not already initialized)
4. Verify `.gitignore` is working before proceeding

### When Adding New Dependencies

1. Check `git status` after installing packages
2. Ensure new build artifacts are ignored
3. If new file types appear, update `.gitignore` accordingly

### When Working with Multiple Languages

If a project uses multiple languages/frameworks, include patterns for all of them:

- Node.js + Python: Include both `node_modules/` and `__pycache__/`
- React + Python backend: Include `.next/` or `dist/` plus Python patterns

## Quick Reference Checklist

Before committing:

- [ ] `.gitignore` exists and is appropriate for the project type
- [ ] Ran `git status` and reviewed output
- [ ] No sensitive files (`.env`, secrets) are staged
- [ ] No build artifacts (`node_modules/`, `dist/`, etc.) are staged
- [ ] Commit message follows format: `feat: TASK-{id} - {description}`
- [ ] `.gitignore` is working correctly (ignored files don't appear in `git status`)

## Common Mistakes to Avoid

1. **Initializing package managers before creating `.gitignore`**
   - This can lead to `node_modules/` or similar directories being tracked

2. **Assuming `.gitignore` works retroactively**
   - Files already tracked by git will continue to be tracked even if added to `.gitignore`
   - Use `git rm --cached` to untrack them

3. **Committing without checking `git status`**
   - Always verify what you're committing

4. **Including sensitive data in commits**
   - Double-check for `.env` files, API keys, passwords
   - If accidentally committed, use `git reset` or `git commit --amend` (if not pushed)

5. **Forgetting project-specific patterns**
   - Different projects need different `.gitignore` patterns
   - Tailor the `.gitignore` to the project's technology stack

## Pre-Commit Checklist

Before committing, verify:

- [ ] TypeScript compilation passes (`npx tsc --noEmit`)
- [ ] No console errors in browser testing
- [ ] All success criteria verified
- [ ] Progress.txt updated

## Pre-Commit Hooks

Consider adding pre-commit hooks for:
- TypeScript compilation check
- Linter checks
- Test execution

Document hook setup in project README if implemented.

## Pre-Commit Validation

**Run TypeScript compilation check before commit:**

```bash
# Before committing, verify compilation
npx tsc --noEmit

# If errors exist, fix them before committing
```

**Verify no console errors:**

```bash
# Check for console errors in browser
agent-browser console | grep -i "error"

# Or check build output
npm run build 2>&1 | grep -i "error"
```

**Check git status before staging:**

```bash
# Always check what will be committed
git status

# Review changes
git diff

# Stage only relevant files
git add src/scenes/GameScene.ts
```

**Review changes with git diff:**

```bash
# Review all changes
git diff

# Review staged changes
git diff --staged

# Review specific file
git diff src/scenes/GameScene.ts
```

## Commit Message Patterns

**Format: "feat: TASK-ID - Description"**

```bash
# Examples
git commit -m "feat: US-033 - Add wizard character sprite"
git commit -m "feat: US-034 - Implement timer countdown"
git commit -m "feat: US-035 - Add scene transition animations"
```

**Include task ID for traceability:**

```bash
# Always include task ID
git commit -m "feat: TASK-{id} - {description}"

# Examples
git commit -m "feat: US-036 - Fix maze generation error handling"
```

**Use conventional commit types:**

```bash
# Types: feat, fix, chore, docs, refactor, test
git commit -m "feat: US-037 - Add new feature"
git commit -m "fix: US-038 - Fix bug"
git commit -m "chore: US-039 - Update dependencies"
```

**Keep messages descriptive but concise:**

```bash
# ✅ GOOD: Descriptive but concise
git commit -m "feat: US-040 - Add coin collection sound effect"

# ❌ BAD: Too vague
git commit -m "feat: US-040 - Update code"

# ❌ BAD: Too verbose
git commit -m "feat: US-040 - Add coin collection sound effect that plays when player collects coin and updates score and triggers animation"
```

## Workflow Patterns

**Check git status before operations:**

```bash
# Before any git operation, check status
git status

# See what files changed
git status --short

# See untracked files
git status --untracked-files=all
```

**Stage only relevant files:**

```bash
# Stage specific files
git add src/scenes/GameScene.ts
git add src/scenes/GameOverScene.ts

# Don't stage everything
# ❌ git add .  # Only if you're sure
```

**Commit after verification passes:**

```bash
# 1. Make changes
# 2. Verify TypeScript compilation
npx tsc --noEmit

# 3. Verify browser testing
agent-browser open http://localhost:3000

# 4. Stage files
git add src/

# 5. Commit
git commit -m "feat: TASK-ID - Description"
```

**Update progress.txt before commit:**

```bash
# Update progress.txt with task completion
echo "## Task Complete" >> tasks/progress.txt
echo "- [x] All criteria met" >> tasks/progress.txt

# Then commit
git add tasks/progress.txt
git commit -m "feat: TASK-ID - Description"
```
