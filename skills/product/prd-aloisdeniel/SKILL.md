---
name: prd
description: "A CLI tool to manage Product Requirements Documents (PRDs) with features, user stories, and acceptance criteria. Triggers on: prd, product requirements, feature management, feature, user story."
---

	# PRD — Product Requirements Document Manager

Use the `prd` CLI to manage feature PRDs, user stories, and acceptance criteria. PRD documents live under the `prd/` directory as markdown files.

## Workflow

1. Before starting work, run `prd feat ls` to see all features and their progress.
2. Identify the current feature with `prd feat current` (reads the git branch name).
3. Find the next incomplete user story with `prd feat <id> us next`.
4. Implement the user story, then mark acceptance criteria as completed.
5. Add notes to track decisions or context discovered during implementation.

## Commands

### List features

```
prd feat ls
```

Outputs tab-separated lines: `ID  Name  [completed/total]`

### Show a feature

```
prd feat <feature-id>
```

Prints the full PRD markdown for the given feature.

### Create a feature

```
prd feat new --name "Feature name" --description "Description"
```

Creates a new `prd/<NNN-slug>/prd.md` with a skeleton template. Prints the new ID.

### Get current feature from git branch

```
prd feat current
```

Prints the feature ID extracted from branches named `feat/<NNN>-*` or `feature/<NNN>-*`.

### Start a feature branch

```
prd feat start <feature-id>
```

Creates and checks out a git branch `feat/<dir-name>`.

### List user stories

```
prd feat <feature-id> us ls
```

Outputs tab-separated lines: `US-N  Name  [status]`

### Show a user story

```
prd feat <feature-id> us <story-id>
```

Prints the markdown section for a single user story.

### Get next incomplete user story

```
prd feat <feature-id> us next
```

Prints the ID of the first user story with incomplete acceptance criteria.

### Create a user story

```
prd feat <feature-id> us new --name "Story name" --description "Description" \
  --acceptance-criteria "First criterion" --acceptance-criteria "Second criterion" \
  --technical-considerations "Notes" --priority 2
```

Appends a new user story to the feature's PRD file.

### List acceptance criteria

```
prd feat <feature-id> us <story-id> accept ls
```

Lists all acceptance criteria with their number (1-based), status, and text.

### Complete a user story

```
prd feat <feature-id> us <story-id> complete
```

Toggles the user story completion status in `progress.md`.

### Delete a user story

```
prd feat <feature-id> us <story-id> delete
```

Removes the user story section from the PRD file.

### Bump version

```
prd feat <feature-id> bump
```

Increments the document version in the footer and updates the date to today.

### List notes

```
prd feat <feature-id> note ls
```

### Add a note

```
prd feat <feature-id> note add --content "Note text"
```

Adds a dated note to the feature's Notes section.

## Typical session

```bash
# See what features exist
prd feat ls

# Check which feature you're on
FEAT=$(prd feat current)

# Find the next story to work on
US=$(prd feat $FEAT us next)

# Read the story details
prd feat $FEAT us $US

# After implementing, mark story as done
prd feat $FEAT us $US complete

# Add a note about a decision made
prd feat $FEAT note add --content "Chose JWT over session cookies for auth"
```

## Document structure

PRD files follow this markdown structure:

```markdown
# Feature Name

Description text.

## Goals

- Goal 1
- Goal 2

## User Stories

### US-1 | P1 | Story name

Description of the user story.

#### Acceptance Criteria

- [ ] Criterion 1
- [ ] Criterion 2
- [x] Completed criterion

#### Technical Considerations

Implementation notes.

## Functional Requirements

- FR-1: Requirement

## Notes

### 2026-01-15

Note content.

---
*Document Version: 1.0*
*Last Updated: 2026-01-15*
```

The document footer (version and last updated date) is automatically maintained whenever the file is modified through `prd` commands.

## Progress tracking

Progress is tracked in a separate `progress.md` file next to `prd.md`. The PRD file is immutable for progress operations. Completing user stories and adding notes writes to `progress.md`:

```markdown
# Progress

- [ ] US-1
- [x] US-2
- [ ] US-3

## Notes

### 2026-01-28

Note content.
```
