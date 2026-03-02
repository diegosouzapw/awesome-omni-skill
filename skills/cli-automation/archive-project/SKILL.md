---
name: archive-project
description: Archive a completed project by creating an AI-generated summary in Knowledge/, storing the git commit hash for reference, and deleting the original project file. Use when user says a project is "done", "complete", "finished", or wants to "archive" it.
---

# Archive Project

Archive a completed project to preserve its knowledge while cleaning up the Projects/ folder.

## Instructions

When the user wants to archive a project:

### 1. Identify the Project File

- If not specified, ask which project to archive
- List projects in the relevant pillar: `ls [Pillar]/Projects/`
- Confirm the file path (e.g., `Personal/Projects/Some Project.md`)

### 2. Get Current Git State

```bash
git rev-parse HEAD
git log -1 --format="%H %s"
```

Store the commit hash - this is the reference point for restoration.

### 3. Read and Summarise the Project

Read the project file and create an intelligent summary covering:
- **What the project was about** - goals, context, purpose
- **Key outcomes and learnings** - what was achieved, lessons learned
- **Important decisions made** - architectural choices, trade-offs, rationale

### 4. Create Archived Summary

Create a new file at `[Pillar]/Knowledge/[Project Name] (Archived).md`:

```markdown
# [Project Name] (Archived)

Archived: YYYY-MM-DD
Commit: `[hash]`

## Summary

[AI-generated summary of what the project was about, its goals, and context]

## Outcomes & Learnings

[AI-generated summary of key outcomes, what was achieved, lessons learned]

## Key Decisions

[AI-generated list of important decisions made during the project]

## Reference

To restore or view the original project:

```bash
git show [hash]
git checkout [hash] -- "[Pillar]/Projects/[Project Name].md"
```

#type/archive #context/[pillar]
```

### 5. Update Manifest

Add entry to `[Pillar]/Knowledge/MANIFEST.md`:
- If the auto-update hook is configured, this happens automatically
- Otherwise, manually add the new file to the manifest table

### 6. Delete Original Project

```bash
rm "[Pillar]/Projects/[Project Name].md"
```

The knowledge is now preserved in the archived summary.

### 7. Commit Changes

```bash
git add -A
git commit -m "Archive project: [Project Name]"
```

## Example

User: "The Marketing Campaign project is done, archive it"

Steps:
1. Read `Personal/Projects/Marketing Campaign.md`
2. Get commit hash: `abc123def`
3. Create AI-generated summary at `Personal/Knowledge/Marketing Campaign (Archived).md`
4. Manifest updates automatically (hook) or manually
5. Delete `Personal/Projects/Marketing Campaign.md`
6. Commit: "Archive project: Marketing Campaign"

## Notes

- Always get the commit hash BEFORE making any changes
- The archived summary should be comprehensive enough to understand the project without the original file
- The commit hash allows full restoration if needed via `git checkout`
