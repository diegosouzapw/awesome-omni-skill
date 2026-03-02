---
name: archive-work
description: Archive completed scratchpads and session logs to project history. Invoke when user says "archive this work", "clean up scratchpad", "archive scratchpad", or after PR is merged.
tools:
  - Read
  - Write
  - Bash:mkdir *
  - Bash:mv *
  - Bash:git *
  - Glob
  - AskUserQuestion
  - Skill
---

# Archive Work Skill

## Purpose

Archive completed scratchpads and development artifacts to maintain clean project roots while preserving work history for future reference. This skill organizes completed work into a structured archive.

## Natural Language Triggers

This skill activates when the user says things like:
- "Archive this work"
- "Clean up the scratchpad"
- "Archive scratchpad"
- "Move scratchpad to archive"
- "We're done, archive everything"
- After PR merge: "PR merged, let's clean up"

## Workflow Execution

### Phase 1: Detect Artifacts (Parallel)

**Execute these searches in parallel** for faster detection:

1. **Find Scratchpads:**
   - `Glob: SCRATCHPAD_*.md` in project root
   - Identify issue numbers from filenames

2. **Find Session Logs:**
   - `Glob: SESSION_LOG_*.md` in project root
   - These are created by the PreCompact hook before auto-compaction
   - Associate with scratchpad (same issue context)

3. **Find Other Related Files:**
   - Related temporary files
   - Claude Code conversation exports

4. **Check Git Status:**
   - Current branch for context
   - Recent commits for PR detection

**After parallel detection, verify completion:**
- Check if scratchpad tasks are all complete
- Check if PR was created/merged
- Warn if work appears incomplete

### Phase 2: Determine Archive Location

**Default Structure:**
```
docs/dev/cc-archive/
└── {YYYYMMDDHHMM}-{issue-number}-{brief-description}/
    ├── SCRATCHPAD_{issue_number}.md
    ├── session-log.md (if exists)
    └── README.md (summary)
```

**Timestamp Prefix:** Archives use `YYYYMMDDHHMM` prefix for chronological ordering.
This ensures archives sort by completion date, not ticket number.

**Check Project Conventions:**
- Read CLAUDE.md for custom archive location
- Check if `docs/dev/cc-archive/` exists
- Create directory structure if needed

### Phase 3: Prepare Archive

1. **Generate Timestamp and Directory Name:**
   ```bash
   # Generate timestamp prefix
   TIMESTAMP=$(date +%Y%m%d%H%M)
   ARCHIVE_DIR="${TIMESTAMP}-{issue-number}-{description}"
   ```

2. **Create Archive Directory:**
   ```bash
   mkdir -p docs/dev/cc-archive/${ARCHIVE_DIR}
   ```

3. **Generate Archive Summary:**
   Create `README.md` in archive folder:
   ```markdown
   # Issue #{issue_number} - {title}

   **Archived:** {date}
   **PR:** #{pr_number} (if applicable)
   **Status:** {Completed/Merged/Abandoned}

   ## Summary
   {Brief description of what was accomplished}

   ## Key Decisions
   {Extract from scratchpad Decisions Made section}

   ## Files Changed
   {List of files that were modified}

   ## Lessons Learned
   {Any notable insights from Work Log}
   ```

4. **Move Files (using git mv for proper tracking):**
   ```bash
   git mv SCRATCHPAD_{issue_number}.md docs/dev/cc-archive/${ARCHIVE_DIR}/
   ```

   **Important:** Use `git mv` instead of `mv` to ensure both the addition to
   archive AND the removal from project root are tracked in the same commit.

### Phase 4: Confirm with User

```
AskUserQuestion:
  question: "Ready to archive this work?"
  header: "Archive"
  options:
    - "Yes, archive and commit"
      description: "Move files to archive and create commit"
    - "Archive without commit"
      description: "Move files but don't commit yet"
    - "Show me what will be archived"
      description: "Preview the archive operation"
    - "Cancel"
      description: "Keep scratchpad in current location"
```

### Phase 5: Execute Archive

1. **Move Files (with git tracking):**
   ```bash
   # Use git mv to track both addition and removal in same commit
   git mv SCRATCHPAD_{issue_number}.md docs/dev/cc-archive/${ARCHIVE_DIR}/

   # Move session logs (created by PreCompact hook)
   # These are untracked, so use mv then git add
   for log in SESSION_LOG_*.md; do
     if [ -f "$log" ]; then
       mv "$log" docs/dev/cc-archive/${ARCHIVE_DIR}/
     fi
   done
   git add docs/dev/cc-archive/${ARCHIVE_DIR}/SESSION_LOG_*.md 2>/dev/null || true
   ```
   - Create summary README in archive directory
   - Stage the new README: `git add docs/dev/cc-archive/${ARCHIVE_DIR}/README.md`

2. **Commit Archive:**
   If user opted to commit:
   ```
   Skill: commit-changes

   # Commit message will be:
   # 📚🗃️ chore(docs): Archive work for issue #{issue_number}
   #
   # Completed work archived to docs/dev/cc-archive/
   # PR: #{pr_number}
   ```

   **The commit will include:**
   - Removal of SCRATCHPAD from project root (via git mv)
   - Addition of SCRATCHPAD in archive directory
   - Session logs (SESSION_LOG_*.md) if present
   - New README.md summary

### Phase 6: Report Result

```
✓ Work archived successfully!

📁 Archive location:
   docs/dev/cc-archive/{YYYYMMDDHHMM}-{issue-number}-{description}/

📄 Files archived:
   - SCRATCHPAD_{issue_number}.md
   - SESSION_LOG_*.md (if any existed)
   - README.md (summary generated)

🗑️ Cleaned up:
   - Removed scratchpad from project root (tracked via git mv)
   - Removed session logs from project root

{If committed}
📝 Committed: {commit hash}
   - Added: archive directory with scratchpad, session logs, README
   - Removed: SCRATCHPAD_{issue_number}.md from project root
   - Removed: SESSION_LOG_*.md from project root
```

## Archive Options

### Option 1: Full Archive (Default)
- Move scratchpad to archive
- Generate summary README
- Commit the archive

### Option 2: Delete Only
If user prefers not to keep history:
```
AskUserQuestion:
  question: "How to handle the scratchpad?"
  options:
    - "Archive (keep history)"
    - "Delete (no history)"
    - "Keep in place"
```

### Option 3: Custom Location
Allow user to specify different archive location:
```
AskUserQuestion:
  question: "Archive to default location?"
  options:
    - "Yes, use docs/dev/cc-archive/"
    - "Specify custom location"
```

## Error Handling

### No Scratchpad Found
```
ℹ️ No scratchpad found to archive.
   Looking for: SCRATCHPAD_*.md in project root
```

### Work Incomplete
```
⚠️ Scratchpad has incomplete tasks:
   - {unchecked task 1}
   - {unchecked task 2}

   Archive anyway?
   1. Yes, archive incomplete work
   2. No, continue working first
```

### Archive Directory Exists
```
⚠️ Archive already exists for issue #{number}

   Options:
   1. Overwrite existing archive
   2. Create numbered version (archive-2/)
   3. Cancel
```

### No PR Created
```
ℹ️ No PR found for this work.

   Archive anyway?
   1. Yes, archive without PR reference
   2. No, create PR first
```

## Integration with Other Skills

**Invoked by:**
- `do-work` skill - After completing all tasks
- User directly after PR is merged

**Invokes:**
- `commit-changes` skill - To commit archive

**Reads from:**
- Scratchpad - Content to archive
- Git history - PR information

## Archive Structure Best Practices

### Recommended Directory Layout
```
docs/
└── dev/
    └── cc-archive/
        ├── 202512281430-42-add-authentication/
        │   ├── SCRATCHPAD_42.md
        │   └── README.md
        ├── 202512281545-43-fix-login-bug/
        │   ├── SCRATCHPAD_43.md
        │   ├── SESSION_LOG_1.md
        │   └── README.md
        └── 202512290900-44-refactor-api/
            ├── SCRATCHPAD_44.md
            ├── SESSION_LOG_1.md
            ├── SESSION_LOG_2.md
            └── README.md
```

### Archive Naming Convention
`{YYYYMMDDHHMM}-{issue-number}-{slugified-description}/`

**Format breakdown:**
- `YYYYMMDDHHMM` - Timestamp when archived (enables chronological sorting)
- `{issue-number}` - GitHub issue number for reference
- `{slugified-description}` - Brief description from issue title

Examples:
- `202512281430-42-add-user-authentication/`
- `202512290915-123-fix-payment-bug/`
- `202512271000-7-initial-project-setup/`

**Why timestamp prefix?**
- Archives sort chronologically regardless of ticket number order
- Easy to scan for recent work
- Preserves actual completion order

## Best Practices

### ✅ DO:
- Archive after PR is merged
- Include summary README
- Preserve decision history
- Use consistent archive location
- Commit archives to repo
- Use `git mv` to move scratchpads (tracks removal properly)
- Use timestamp prefix for chronological ordering

### ❌ DON'T:
- Archive incomplete work without noting it
- Delete without archiving (lose history)
- Mix archives from different projects
- Skip the summary README
- Leave scratchpads in project root long-term
- Use plain `mv` for tracked files (leaves unstaged deletion)

---

**Version:** 1.3.0
**Last Updated:** 2025-12-31
**Maintained By:** Escapement
**Changelog:**
- v1.3.0: Added parallel execution for artifact detection
- v1.2.0: Added SESSION_LOG_*.md detection and archiving (from PreCompact hook)
- v1.1.0: Added timestamp prefix for chronological sorting; use git mv for proper tracking
- v1.0.0: Initial conversion from commands/archive-dev.md
