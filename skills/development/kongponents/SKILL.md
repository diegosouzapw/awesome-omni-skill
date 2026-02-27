---
name: kongponents
description: Kong Vue component library reference - provides props, slots, events, and code examples for Kongponents components
---

## Setup Check

Before using this skill, verify the documentation is synced:

1. Check if `docs/VERSION` file exists
2. If not found: Display error message and stop
   ```
   ❌ Kongponents documentation not synced.

   Run the following command to download documentation:
   /kongponents sync
   ```
3. If found: Proceed to help user with Kongponents

## Commands

### /kongponents sync

Downloads and indexes Kongponents component documentation.

**What it does:**
1. Clones Kong/kongponents repository (docs/components only)
2. Generates component index
3. Creates searchable reference in `docs/components/`

**Usage:**
```
/kongponents sync
```

**Implementation:**
- Use Bash tool to execute: `bash scripts/sync.sh`
- Display progress output to user
- Report success or failure

**First time setup:**
Run this command once after installing the skill.

## Component Reference

After syncing, component documentation is available in `docs/components/`.

**To help users:**

1. **Check if synced** - Verify `docs/VERSION` exists first
2. **Read component index** - Use `docs/component-index.md` to see all components
3. **Read component docs** - Use `docs/components/<component-name>.md` for details
4. **Provide examples** - Generate Vue code using Kongponents patterns

**Common components:**
- KButton - Button with various appearances and sizes
- KInput - Text input with validation states
- KModal - Modal dialog with overlay
- KSelect - Dropdown select with search
- KTable - Data table component (deprecated, use KTableData)
- KAlert - Alert/notification messages
- KBadge - Status badges
- KCard - Card container
- KIcon - Icon component
- ... and 30+ more

**Example workflow:**

User asks: "Show me KButton props"

1. Check `docs/VERSION` exists ✓
2. Read `docs/components/button.md`
3. Extract props section
4. Provide formatted answer with examples

User asks: "Create a danger button"

1. Check `docs/VERSION` exists ✓
2. Read `docs/components/button.md`
3. Find danger appearance prop
4. Generate code:
   ```vue
   <KButton appearance="danger">
     Delete
   </KButton>
   ```

## Update Check (Optional)

To check if documentation is outdated:

1. Read `docs/last-check.txt` timestamp
2. If more than 86400 seconds (24 hours) old:
   - Show message: "Kongponents docs may be outdated (last synced X days ago). Run `/kongponents sync` to update."
   - User can choose to continue or sync

## Error Handling

**Missing sync:**
```
❌ Kongponents documentation not synced.
Run: /kongponents sync
```

**Sync failures:**
- Git not installed: "Error: git not found. Please install git first."
- Network issues: "Error: Failed to clone repository. Check your internet connection."
- Permission issues: "Error: Cannot write to docs/ directory."

**Component not found:**
- Check `docs/component-index.md` for available components
- Suggest similar component names if typo detected
