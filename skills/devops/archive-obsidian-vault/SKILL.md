---
name: archive-obsidian-vault
description: Archives monthly work items from Obsidian vault Area/ and Projects/ to Archive/ with proper organization and attachment handling. Use when archiving completed monthly work.
---

# Archive Obsidian Vault

## Overview

This skill automates the archival process for monthly work items in an Obsidian vault. It moves completed work from `Area/` and `Projects/` directories to `Archive/` following a consistent naming pattern, handles attachments properly, and optionally processes journal files.

## Vault Structure

The Obsidian vault follows this structure:
- `Area/` - Active work areas (Discussions, Drawings, Journal, People)
- `Projects/` - Active projects
- `Archive/` - Archived work organized by YYYY-MM [Category] pattern

## Quick Start

To archive work for a specific month:

1. **Request month to archive** - Ask user for the month in YYYY-MM format (e.g., 2025-10)
2. **Identify items to archive** - Search for all files and folders matching the date pattern
3. **Process journal cleanup** - Run delete_empty_journals.py if requested
4. **Create archive directories** - Set up destination folders
5. **Move files with attachments** - Transfer files and their associated attachments
6. **Verify completion** - Confirm all items were archived successfully

## Archival Process

### Step 1: Request Month to Archive

**Ask the user for the month:**

- Request the month in YYYY-MM format (e.g., "2025-10")
- Validate the format matches YYYY-MM pattern
- Store this as the `$MONTH` variable for the rest of the process

**Ask about journal processing preferences:**

- Should empty journals be deleted before archiving?
- Should journals be combined into monthly files?
- Should bookmarks be extracted from journals?
- Or should journals be moved as-is?

### Step 2: Identify Items to Archive

**Search for items in Area/ directories:**

Navigate to the vault location:
```
/Users/paul/Library/CloudStorage/GoogleDrive-paul@meandu.com/My Drive/Obsidian/meandu/
```

Find items matching the month pattern:
- `Area/Discussions/$MONTH *.md` files
- `Area/Drawings/$MONTH-*.md` files
- `Area/Journal/$MONTH-*.md` files
- `Projects/$MONTH */` folders

**Identify attachment references:**

For each Drawings file found:
- Extract attachment references using pattern: `![[*-tldrawFile.*]]`
- Build list of attachment files that need to be moved
- Attachments are typically in previous months' Archive/*/attachments/ folders or Projects/attachments/

### Step 3: Process Journal Cleanup (Optional)

If user requested empty journal deletion:

```bash
cd 'Area/Journal'
python ~/bin/obsidian/delete_empty_journals.py --dry-run . $MONTH
# Review output, then run without --dry-run
python ~/bin/obsidian/delete_empty_journals.py . $MONTH
```

Available journal processing scripts in `~/bin/obsidian/`:
- `delete_empty_journals.py` - Removes journal files containing only whitespace
- `combine_journals.py` - Merges daily YYYY-MM-DD.md files into monthly YYYY-MM Journal.md
- `extract_bookmarks.py` - Extracts links into separate YYYY-MM Links.md files

### Step 4: Create Archive Directories

Create destination directories following the established pattern:

```bash
cd Archive
mkdir -p "$MONTH Discussions"
mkdir -p "$MONTH Drawings/attachments"
mkdir -p "$MONTH Journals"
# Create project-specific folders as needed
mkdir -p "$MONTH [Project Name]"
```

### Step 5: Move Files with Attachments

**Move Discussion files:**
```bash
mv Area/Discussions/$MONTH*.md Archive/$MONTH\ Discussions/
```

**Move Drawing files and attachments:**

1. Move markdown files:
```bash
mv Area/Drawings/$MONTH-*.md Archive/$MONTH\ Drawings/
```

2. Extract list of referenced attachments from moved files
3. Find and copy/move attachments from their current locations to Archive/$MONTH Drawings/attachments/
4. Verify all referenced attachments are present

**Move Journal files:**
```bash
mv Area/Journal/$MONTH-*.md Archive/$MONTH\ Journals/
```

**Move Project folders:**
```bash
mv Projects/$MONTH\ [Project\ Name] Archive/
```

### Step 6: Verify Completion

**Count and verify:**
- Count files in each archive directory
- Verify attachment references match available files
- Report final counts to user

**Summary format:**
```
Archived for $MONTH:
- X discussion files → Archive/$MONTH Discussions/
- X drawing files → Archive/$MONTH Drawings/
- X attachment files → Archive/$MONTH Drawings/attachments/
- X journal files → Archive/$MONTH Journals/
- X project folders

Total: X items archived
```

## File Paths Reference

**Vault base path:**
```
/Users/paul/Library/CloudStorage/GoogleDrive-paul@meandu.com/My Drive/Obsidian/meandu/
```

**Key directories:**
- `Area/Discussions/` - Discussion files
- `Area/Drawings/` - Drawing files with tldraw attachments
- `Area/Journal/` - Daily journal entries
- `Projects/` - Project folders
- `Archive/` - Archived work

**Journal processing scripts:**
```
~/bin/obsidian/delete_empty_journals.py
~/bin/obsidian/combine_journals.py
~/bin/obsidian/extract_bookmarks.py
```

## Archive Naming Pattern

All archive folders follow this pattern:
```
YYYY-MM [Category]
```

**Examples:**
- `2025-10 Discussions` - For discussion files
- `2025-10 Drawings` - For drawing files (includes attachments/ subdirectory)
- `2025-10 Journals` - For journal entries
- `2025-10 On-call training` - For specific projects

## Attachment Handling

**Attachment reference format:**
```
![[uuid-tldrawFile.webp]]
![[uuid-tldrawFile.png]]
```

**Finding attachments:**
- Check Archive/YYYY-MM Drawings/attachments/ directories from previous months
- Check Projects/attachments/ directory
- Use find command to locate specific files by UUID

**Organizing attachments:**
- All attachments for a month go in Archive/YYYY-MM Drawings/attachments/
- Maintain flat structure (no nested directories)
- Preserve original filenames

## Error Handling

**Common issues:**

1. **Nested attachments directory** - If attachments end up in attachments/attachments/, flatten:
   ```bash
   cd Archive/$MONTH\ Drawings/attachments
   mv attachments/* .
   rmdir attachments
   ```

2. **Shared attachments** - If files reference attachments from previous months, copy (don't move) them:
   ```bash
   cp Archive/YYYY-MM\ Drawings/attachments/uuid.png Archive/YYYY-MM\ Drawings/attachments/
   ```

3. **Google Drive sync** - If files aren't visible in Obsidian after archiving:
   - Wait for Google Drive to complete sync
   - Tell user to reload vault in Obsidian (Cmd+R)

## Best Practices

1. **Always run journal cleanup scripts with --dry-run first** to preview changes
2. **Verify attachment counts** before and after moving to ensure nothing is lost
3. **Use comm or similar tools** to verify all referenced attachments are present
4. **Report detailed summaries** to user showing what was archived
5. **Check for empty directories** after moving files to keep vault clean
