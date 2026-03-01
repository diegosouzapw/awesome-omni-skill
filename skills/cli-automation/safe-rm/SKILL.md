---
name: safe-rm
description: Safe file deletion for Git projects. Classifies files (ALLOW/TRASH/BLOCK), uses trash for untracked source, logs all operations. Use when deleting files, cleaning build artifacts, running rm -rf, or when the user mentions cleanup, trash, or deletion in repositories.
license: MIT
compatibility: Requires git, trash-cli; works with Cursor, VSCode, Claude Code
metadata:
  author: kayaman
  version: "1.0"
---

# Safe RM - Protected File Deletion for Git Projects

## Purpose

This skill teaches safe file deletion practices to prevent accidental loss of unstaged/untracked files in Git repositories, while allowing efficient cleanup of build artifacts and dependencies. It works seamlessly with AI coding agents and human users.

## When to Use This Skill

Use this skill whenever you need to delete files or directories, especially:
- Cleaning up build artifacts or dependencies
- Removing temporary files in Git repositories
- Any `rm -rf` operation in project directories
- Deleting files that might include untracked source code

## Critical Safety Principles

### 1. NEVER directly delete in Git repositories without checking

Before any deletion operation in a Git repository, you MUST:
- Check if files are tracked, staged, or untracked
- Classify files by type and risk level
- Use appropriate deletion method based on classification

### 2. Classification System

Files fall into three categories:

**AUTO_ALLOW** (safe to delete directly, just log):
- Build artifacts: `dist/`, `build/`, `target/`, `out/`, `.next/`, `public/build/`
- Dependencies: `node_modules/`, `vendor/`, `.venv/`, `venv/`, `__pycache__/`
- Cache: `.cache/`, `.pytest_cache/`, `.mypy_cache/`, `.tox/`
- Compiled: `*.pyc`, `*.class`, `*.o`, `*.so`, `*.dll`
- Temp: `*.tmp`, `*.log`, `*.swp`, `.DS_Store`, `Thumbs.db`

**AUTO_TRASH** (move to trash, highly recoverable):
- Untracked source code: `*.py`, `*.js`, `*.ts`, `*.jsx`, `*.tsx`, `*.rs`, `*.go`, `*.java`, `*.c`, `*.cpp`, `*.h`
- Untracked docs: `*.md`, `*.txt`, `*.rst`, `*.adoc`
- Untracked configs: `*.json`, `*.yaml`, `*.toml`, `*.ini` (not in .gitignore)
- Local env files: `*.env.local`, `*.env.development`, `.env.*.local`
- Any files modified in last 24 hours
- Any file with "important", "draft", "wip", "todo" in name

**BLOCK** (refuse deletion, return error):
- `.git/` directory
- Root directories: `/`, `/home`, `/etc`, `/usr`, `/var`
- System directories: `/bin`, `/sbin`, `/lib`
- Files matching protection patterns: `**/*important*/**`, `**/*backup*/**`

## Implementation Guide

### Step 1: Detection (Required First Step)

```bash
# Check if in Git repository
if git rev-parse --is-inside-work-tree >/dev/null 2>&1; then
    IN_GIT_REPO=true
    
    # Get list of untracked files in target path
    UNTRACKED=$(git ls-files --others --exclude-standard "$TARGET_PATH")
    
    # Get list of files in .gitignore
    IGNORED=$(git check-ignore "$TARGET_PATH"/* 2>/dev/null)
fi
```

### Step 2: Classification

```bash
# Classify each file/directory
classify_file() {
    local file="$1"
    local basename=$(basename "$file")
    
    # Check BLOCK patterns first (highest priority)
    case "$file" in
        .git|.git/*|/|/home|/etc|/usr|/var|/bin|/sbin|/lib)
            echo "BLOCK"
            return
            ;;
        *important*|*backup*)
            echo "BLOCK"
            return
            ;;
    esac
    
    # Check AUTO_ALLOW patterns (build artifacts, etc.)
    case "$basename" in
        node_modules|dist|build|target|out|.next|__pycache__|.cache|vendor|.venv|venv)
            echo "ALLOW"
            return
            ;;
    esac
    
    case "$file" in
        *.pyc|*.class|*.o|*.so|*.dll|*.tmp|*.log|*.swp|.DS_Store)
            echo "ALLOW"
            return
            ;;
    esac
    
    # Check if in .gitignore (AUTO_ALLOW)
    if git check-ignore -q "$file" 2>/dev/null; then
        echo "ALLOW"
        return
    fi
    
    # Check if untracked source code (AUTO_TRASH)
    if [[ "$IN_GIT_REPO" == "true" ]] && echo "$UNTRACKED" | grep -q "^$file$"; then
        case "$file" in
            *.py|*.js|*.ts|*.jsx|*.tsx|*.rs|*.go|*.java|*.c|*.cpp|*.h|*.md|*.txt)
                echo "TRASH"
                return
                ;;
        esac
    fi
    
    # Default: TRASH (safe mode)
    echo "TRASH"
}
```

### Step 3: Execution

```bash
# Execute deletion based on classification
delete_safely() {
    local file="$1"
    local classification=$(classify_file "$file")
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    
    case "$classification" in
        BLOCK)
            echo "ERROR: Blocked deletion of protected path: $file" >&2
            echo "$timestamp | BLOCK | $file | REFUSED" >> ~/.local/state/safe-rm/audit.log
            return 1
            ;;
            
        ALLOW)
            # Direct deletion, but log it
            rm -rf "$file"
            echo "$timestamp | ALLOW | $file | deleted" >> ~/.local/state/safe-rm/audit.log
            ;;
            
        TRASH)
            # Move to trash for recovery
            trash-put "$file"
            echo "$timestamp | TRASH | $file | trashed" >> ~/.local/state/safe-rm/audit.log
            ;;
    esac
}
```

## Correct Usage Examples

### Example 1: Clean Build Artifacts
```bash
# AI Agent internal reasoning:
# "User wants to clean project. Let me check what's safe to delete."

# Check Git status first
git status --short

# Clean node_modules (AUTO_ALLOW - in .gitignore)
rm -rf node_modules/
echo "$(date '+%Y-%m-%d %H:%M:%S') | ALLOW | node_modules/ | deleted" >> ~/.local/state/safe-rm/audit.log

# Clean dist (AUTO_ALLOW - build artifact)
rm -rf dist/
echo "$(date '+%Y-%m-%d %H:%M:%S') | ALLOW | dist/ | deleted" >> ~/.local/state/safe-rm/audit.log
```

### Example 2: Mixed Content Directory
```bash
# AI Agent internal reasoning:
# "Directory contains both source code and build files. Must classify individually."

# Check untracked files
UNTRACKED=$(git ls-files --others --exclude-standard src/)

# Process each file
for file in src/*; do
    if [[ "$file" == *".pyc" ]] || [[ "$file" == *"__pycache__"* ]]; then
        # Compiled files - safe to delete
        rm -rf "$file"
        echo "$(date '+%Y-%m-%d %H:%M:%S') | ALLOW | $file | deleted" >> ~/.local/state/safe-rm/audit.log
    elif echo "$UNTRACKED" | grep -q "^$file$"; then
        # Untracked source - move to trash
        trash-put "$file"
        echo "$(date '+%Y-%m-%d %H:%M:%S') | TRASH | $file | trashed" >> ~/.local/state/safe-rm/audit.log
    else
        # Tracked file - probably shouldn't delete, but if needed:
        rm "$file"
        echo "$(date '+%Y-%m-%d %H:%M:%S') | ALLOW | $file | deleted" >> ~/.local/state/safe-rm/audit.log
    fi
done
```

### Example 3: Complete Project Cleanup
```bash
# AI Agent internal reasoning:
# "User wants fresh start. Must preserve untracked work-in-progress."

# First, identify and trash untracked source files
git ls-files --others --exclude-standard | while read -r file; do
    case "$file" in
        *.py|*.js|*.ts|*.md|*.txt)
            trash-put "$file"
            echo "$(date '+%Y-%m-%d %H:%M:%S') | TRASH | $file | trashed (untracked source)" >> ~/.local/state/safe-rm/audit.log
            ;;
    esac
done

# Then clean build artifacts
rm -rf node_modules/ dist/ .cache/
echo "$(date '+%Y-%m-%d %H:%M:%S') | ALLOW | build-artifacts | deleted" >> ~/.local/state/safe-rm/audit.log

# Inform user
echo "Cleanup complete. Untracked source files moved to trash. Use 'trash-list' to review."
```

## Recovery Procedures

### View Trashed Files
```bash
# List all trashed files
trash-list

# List trashed files from today
trash-list | grep "$(date '+%Y-%m-%d')"

# Search for specific file
trash-list | grep "filename"
```

### Restore Trashed Files
```bash
# Interactive restore
trash-restore

# Direct restore (if you know the path)
trash-restore /path/to/file
```

### Check Audit Log
```bash
# View recent deletions
tail -50 ~/.local/state/safe-rm/audit.log

# View all TRASH operations today
grep "$(date '+%Y-%m-%d')" ~/.local/state/safe-rm/audit.log | grep TRASH

# View all BLOCK attempts
grep BLOCK ~/.local/state/safe-rm/audit.log
```

## Integration with Btrfs Snapshots

On systems with btrfs and Snapper (like OpenSUSE Tumbleweed), you have an additional recovery layer:

```bash
# List available snapshots
sudo snapper list

# Compare current state with snapshot
sudo snapper diff <snapshot-number>..0

# Restore from snapshot if needed
sudo snapper rollback <snapshot-number>
```

## Common Patterns for AI Agents

### Pattern 1: Dependency Cleanup
```bash
# Safe pattern for removing dependencies
if [[ -d "node_modules" ]]; then
    rm -rf node_modules/
    echo "$(date '+%Y-%m-%d %H:%M:%S') | ALLOW | node_modules/ | deleted" >> ~/.local/state/safe-rm/audit.log
fi
```

### Pattern 2: Build Cleanup
```bash
# Safe pattern for build artifacts
for dir in dist build target out .next; do
    if [[ -d "$dir" ]]; then
        rm -rf "$dir/"
        echo "$(date '+%Y-%m-%d %H:%M:%S') | ALLOW | $dir/ | deleted" >> ~/.local/state/safe-rm/audit.log
    fi
done
```

### Pattern 3: Untracked File Check Before Deletion
```bash
# Always check for untracked source before rm -rf
if git rev-parse --is-inside-work-tree >/dev/null 2>&1; then
    UNTRACKED_SRC=$(git ls-files --others --exclude-standard | grep -E '\.(py|js|ts|md|txt)$')
    if [[ -n "$UNTRACKED_SRC" ]]; then
        echo "WARNING: Found untracked source files. Moving to trash:"
        echo "$UNTRACKED_SRC"
        echo "$UNTRACKED_SRC" | while read -r file; do
            trash-put "$file"
            echo "$(date '+%Y-%m-%d %H:%M:%S') | TRASH | $file | auto-protected" >> ~/.local/state/safe-rm/audit.log
        done
    fi
fi
```

## Error Handling

### If trash-cli is not available:
```bash
if ! command -v trash-put >/dev/null 2>&1; then
    echo "ERROR: trash-cli not installed. Cannot safely delete files." >&2
    echo "Install with: sudo zypper install trash-cli" >&2
    exit 1
fi
```

### If audit log is not writable:
```bash
AUDIT_LOG="${SAFE_RM_AUDIT_LOG:-${XDG_STATE_HOME:-$HOME/.local/state}/safe-rm/audit.log}"
if [[ ! -w "$AUDIT_LOG" ]] && [[ ! -w "$(dirname "$AUDIT_LOG")" ]]; then
    # Fallback to stderr
    echo "$(date '+%Y-%m-%d %H:%M:%S') | WARNING | Could not write to audit log" >&2
fi
```

## Performance Considerations

For AI agents running many operations:
- Cache Git repo detection: `git rev-parse --show-toplevel`
- Batch operations where possible
- Use simple pattern matching before expensive Git commands
- Skip Git checks for known build directories

## Critical Reminders

1. **ALWAYS log deletions** to `~/.local/state/safe-rm/audit.log`
2. **ALWAYS classify before deleting** - never assume
3. **PREFER trash over rm** when uncertain
4. **NEVER delete .git directory**
5. **VERIFY untracked files** before bulk operations
6. **TEST with dry-run** when implementing new deletion logic

## Testing Your Implementation

Before executing deletion code:

```bash
# Dry run - show what would be deleted
echo "Would delete: $TARGET"
classify_file "$TARGET"

# Verify classification matches expectations
# Only then proceed with actual deletion
```

## Summary Decision Tree

```
File to delete?
├─ Is .git or system path? → BLOCK (error)
├─ Is build artifact/dependency? → ALLOW (delete, log)
├─ Is in .gitignore? → ALLOW (delete, log)
├─ Is untracked source code? → TRASH (move, log)
└─ Uncertain? → TRASH (safe default, log)
```

## Tool Requirements

Required tools (check before execution):
- `git` - for repository detection
- `trash-put` - for safe deletion (from trash-cli package)
- Standard shell utilities: `grep`, `basename`, `date`

Installation on OpenSUSE Tumbleweed:
```bash
sudo zypper install trash-cli git
```

## Support and Maintenance

Audit log: `~/.local/state/safe-rm/audit.log` (override with `SAFE_RM_AUDIT_LOG`)
Trash: `~/.local/share/Trash/`
Configuration: `~/.config/safe-rm/safe-rm-rules.yaml` (override with `SAFE_RM_CONFIG_DIR`)

Regular maintenance:
```bash
# Empty trash older than 30 days
trash-empty 30

# Review audit log monthly
grep "$(date -d '1 month ago' '+%Y-%m')" ~/.local/state/safe-rm/audit.log | wc -l
```
