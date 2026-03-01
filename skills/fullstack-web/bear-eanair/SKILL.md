---
name: bear
description: Interacts with Bear note-taking app on macOS via X-Callback-URL. Use when user asks to create Bear notes, search notes, add text to notes, manage tags, capture web pages to Bear, or perform any other Bear note management tasks. Supports note creation, text appending, tag management, note search, and web page capture.
---

# Bear Note Integration

Integrates Claude with the Bear note-taking app on macOS, enabling automated note creation, searching, editing, and tag management through X-Callback-URL.

## Quick Start

### Prerequisites

1. **Bear app installed** on macOS
2. **API Token** (optional, for search/tags features):
   - Help > Advanced > API Token > Copy Token
   - Set environment variable: `export BEAR_API_TOKEN="your-token"`
3. **xcall tool** (optional, for response handling):
   - Download from [xcall releases](https://github.com/martinfinke/xcall/releases)
   - Install to `/Applications/xcall.app`

### Basic Usage

Create a note:

```python
from scripts.bear import create_note

# Create note without response
create_note(title="My Note", text="Content here", tags="work,important")

# Get response with note ID (requires xcall)
result = create_note(title="My Note", text="Content", return_id=True)
print(result['identifier'])
```

Search notes:

```python
from scripts.bear import search_notes

# Requires xcall and token
results = search_notes(term="project-x", tag="work")
for note in results:
    print(f"{note['title']} - {note['identifier']}")
```

## Core Tasks

### 1. Create Notes

Create new notes in Bear with optional tags and timestamps.

```python
from scripts.bear import create_note

# Basic note creation
create_note(
    title="Daily Standup",
    text="## Progress\n- Task 1 complete\n- Task 2 in progress",
    tags="work,standup"
)

# With timestamp
create_note(
    title="Meeting Notes",
    text="Key discussion points...",
    tags="meetings",
    add_timestamp=True
)

# With response (returns note ID)
result = create_note(
    title="Important Note",
    text="Critical information",
    return_id=True
)
note_id = result['identifier']
```

### 2. Search Notes

Find notes by keywords with optional tag filtering.

```python
from scripts.bear import search_notes

# Keyword search
results = search_notes(term="Python")

# Tag filtering
results = search_notes(term="bug", tag="development")

# Process results
for note in results:
    print(f"Title: {note['title']}")
    print(f"ID: {note['identifier']}")
    print(f"Modified: {note['modificationDate']}")
```

### 3. Add Text to Notes

Append or replace text in existing notes.

```python
from scripts.bear import add_text

# Append text
add_text(
    note_id="7E4B681B",
    text="\n\n## Update\nNew content added",
    mode="append"
)

# Append to specific header
add_text(
    note_id="7E4B681B",
    text="- New item",
    mode="append",
    header="TODO"
)

# Replace all content
add_text(
    note_id="7E4B681B",
    text="New complete content",
    mode="replace_all"
)
```

### 4. Manage Tags

List, rename, and delete tags.

```python
from scripts.bear import get_tags, rename_tag, delete_tag

# List all tags
tags = get_tags()
for tag in tags:
    print(tag['name'])

# Rename tag
rename_tag(old_name="todo", new_name="inbox")

# Delete tag
delete_tag(name="archive")
```

### 5. Capture Web Pages

Save web page content as new Bear notes.

```python
from scripts.bear import grab_url

# Capture URL as note
result = grab_url(
    url="https://example.com/article",
    tags="reference,articles"
)

# With response
result = grab_url(
    url="https://docs.python.org",
    tags="docs",
    return_id=True
)
note_id = result['identifier']
```

## Bundled Resources

### scripts/

Python module `bear.py` provides functions for all Bear X-Callback-URL actions:

- `create_note()` - Create notes
- `search_notes()` - Search notes
- `add_text()` - Append/replace text
- `open_note()` - Open notes
- `get_tags()` - List all tags
- `rename_tag()` - Rename tags
- `delete_tag()` - Delete tags
- `grab_url()` - Capture web pages
- `trash_note()`, `archive_note()` - Organize notes

### references/

- **actions.md**: Complete X-Callback-URL API reference with all parameters
- **workflows.md**: Common workflows and integration patterns
- **troubleshooting.md**: Setup help and common issues

See these files for detailed API documentation and advanced usage patterns.
