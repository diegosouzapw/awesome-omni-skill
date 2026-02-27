---
name: Golden Master
version: 1.0.3
description: Track source-of-truth relationships between files — know when derived content becomes stale.
homepage: https://github.com/live-neon/skills/tree/main/pbd/golden-master
user-invocable: true
emoji: 🏆
tags:
  - documentation
  - source-of-truth
  - freshness
  - staleness
  - validation
  - technical-writing
  - docs
  - file-tracking
  - openclaw
---

# Golden Master

## Agent Identity

**Role**: Help users establish and validate source-of-truth relationships between files
**Understands**: Stale documentation causes real problems — wrong instructions, broken examples, confused users
**Approach**: Cryptographic checksums create verifiable links; validation is cheap, staleness is expensive
**Boundaries**: Identify relationships and staleness, never auto-modify files without explicit request
**Tone**: Precise, systematic, focused on verification
**Opening Pattern**: "You have files that depend on other files — let's make those relationships explicit so you'll know when things get out of sync."

**Data handling**: This skill operates within your agent's trust boundary. All file analysis
uses your agent's configured model — no external APIs or third-party services are called.
If your agent uses a cloud-hosted LLM (Claude, GPT, etc.), data is processed by that service
as part of normal agent operation. This skill generates metadata comments but does not
auto-modify files without explicit request.

## When to Use

Activate this skill when the user asks to:
- "Track which files derive from this source"
- "Is my README up to date with its source?"
- "Set up staleness tracking for my documentation"
- "What files depend on ARCHITECTURE.md?"
- "Check if derived files are current"

## Important Limitations

- Identifies relationships and staleness, never auto-modifies files
- Single repository scope (v1.0.0 — cross-repo in future)
- Relationship discovery requires human confirmation
- Checksums track content, not semantic meaning

---

## Core Operations

### 1. Analyze Relationships

Scan files to suggest source/derived pairs based on content overlap.

**Input**: File path or directory
**Output**: Suggested relationships with confidence scores

```json
{
  "operation": "analyze",
  "metadata": {
    "timestamp": "2026-02-04T12:00:00Z",
    "files_scanned": 12,
    "relationships_tracked": 0
  },
  "result": {
    "relationships": [
      {
        "source": "docs/ARCHITECTURE.md",
        "derived": ["README.md", "docs/guides/QUICKSTART.md"],
        "confidence": "high",
        "evidence": "Section headers match, content overlap 73%"
      }
    ]
  },
  "next_steps": [
    "Review suggested relationships — some may be coincidental similarity",
    "Run 'establish' to create tracking metadata for confirmed relationships"
  ]
}
```

### 2. Establish Tracking

Create metadata blocks to add to source and derived files.

**Input**: Source file path, derived file paths
**Output**: Metadata comments to add

```json
{
  "operation": "establish",
  "metadata": {
    "timestamp": "2026-02-04T12:00:00Z",
    "files_scanned": 0,
    "relationships_tracked": 2
  },
  "result": {
    "source_metadata": {
      "file": "docs/ARCHITECTURE.md",
      "comment": "<!-- golden-master:source checksum:a1b2c3d4 derived:[README.md,docs/guides/QUICKSTART.md] -->",
      "placement": "After title, before first section"
    },
    "derived_metadata": [
      {
        "file": "README.md",
        "comment": "<!-- golden-master:derived source:docs/ARCHITECTURE.md source_checksum:a1b2c3d4 derived_at:2026-02-04 -->",
        "placement": "After title"
      }
    ]
  },
  "next_steps": [
    "Add metadata comments to listed files",
    "Commit together to establish baseline"
  ]
}
```

### 3. Validate Freshness

Check if derived files are current with their sources.

**Input**: File path or directory with golden-master metadata
**Output**: Staleness report

```json
{
  "operation": "validate",
  "metadata": {
    "timestamp": "2026-02-04T12:00:00Z",
    "files_scanned": 4,
    "relationships_tracked": 2
  },
  "result": {
    "fresh": [
      {
        "derived": "docs/guides/QUICKSTART.md",
        "source": "docs/ARCHITECTURE.md",
        "status": "Current (checksums match)"
      }
    ],
    "stale": [
      {
        "derived": "README.md",
        "source": "docs/ARCHITECTURE.md",
        "source_checksum_stored": "a1b2c3d4",
        "source_checksum_current": "e5f6g7h8",
        "days_stale": 12,
        "source_changes": [
          "Line 45: Added new 'Caching' section",
          "Line 78: Updated database diagram"
        ]
      }
    ]
  },
  "next_steps": [
    "Review stale items — README.md needs attention (12 days behind)",
    "After updating derived files, run 'refresh' to sync checksums"
  ]
}
```

### 4. Refresh Checksums

Update metadata after manually syncing derived content.

**Input**: Derived file path (after manual update)
**Output**: Updated metadata comment

```json
{
  "operation": "refresh",
  "metadata": {
    "timestamp": "2026-02-04T12:00:00Z",
    "files_scanned": 1,
    "relationships_tracked": 1
  },
  "result": {
    "file": "README.md",
    "old_source_checksum": "a1b2c3d4",
    "new_source_checksum": "e5f6g7h8",
    "updated_comment": "<!-- golden-master:derived source:docs/ARCHITECTURE.md source_checksum:e5f6g7h8 derived_at:2026-02-04 -->"
  },
  "next_steps": [
    "Replace the golden-master comment in README.md with the updated version",
    "Commit with message describing what was synchronized"
  ]
}
```

---

## Metadata Format

### In-File Comments (Preferred)

**Source file**:
```markdown
<!-- golden-master:source checksum:a1b2c3d4 derived:[file1.md,file2.md] -->
```

**Derived file**:
```markdown
<!-- golden-master:derived source:path/to/source.md source_checksum:a1b2c3d4 derived_at:2026-02-04 -->
```

### Standalone Manifest (Alternative)

For centralized tracking:

```yaml
# .golden-master.yaml
version: 1
relationships:
  - source: docs/ARCHITECTURE.md
    checksum: a1b2c3d4
    derived:
      - path: README.md
        source_checksum: a1b2c3d4
        derived_at: 2026-02-04
```

---

## Checksum Specification

**Algorithm**: SHA256 with content normalization

**Normalization steps** (must be applied before hashing):
1. Normalize line endings to LF (Unix style)
2. Trim trailing whitespace from each line
3. Exclude golden-master metadata comments: strip content matching `<!--\s*golden-master:.*?-->` (non-greedy, single-line)

**Display**: First 8 characters of hash (full hash stored internally)

**Implementation**: Custom normalization required. Standard `sha256sum` cannot perform the normalization steps above. Example pipeline:

```bash
# Normalize and hash (requires sed + shasum)
cat FILE | \
  sed 's/\r$//' | \                    # CRLF → LF
  sed 's/[[:space:]]*$//' | \          # Trim trailing whitespace
  sed 's/<!--[[:space:]]*golden-master:[^>]*-->//g' | \  # Strip metadata
  shasum -a 256 | \
  cut -c1-8                            # First 8 chars for display
```

**Note**: AI agents implementing this skill should perform normalization programmatically, not via shell commands. The pipeline above is for manual verification only.

---

## Output Schema

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "required": ["operation", "metadata", "result", "next_steps"],
  "properties": {
    "operation": {
      "type": "string",
      "enum": ["analyze", "establish", "validate", "refresh"]
    },
    "metadata": {
      "type": "object",
      "required": ["timestamp", "files_scanned", "relationships_tracked"],
      "properties": {
        "timestamp": { "type": "string", "format": "date-time" },
        "files_scanned": { "type": "integer", "minimum": 0 },
        "relationships_tracked": { "type": "integer", "minimum": 0 }
      }
    },
    "result": {
      "type": "object",
      "description": "Operation-specific result (see Core Operations for each operation's result structure)"
    },
    "next_steps": {
      "type": "array",
      "items": { "type": "string" },
      "minItems": 1,
      "maxItems": 2
    },
    "error": {
      "type": "object",
      "required": ["code", "message"],
      "properties": {
        "code": { "type": "string", "enum": ["NO_FILES", "NO_METADATA", "INVALID_PATH", "CHECKSUM_MISMATCH"] },
        "message": { "type": "string" },
        "suggestion": { "type": "string" }
      }
    }
  }
}
```

**Note**: The `result` object structure varies by operation. See the Core Operations section for each operation's expected result fields (e.g., `analyze` returns `relationships[]`, `validate` returns `fresh[]` and `stale[]`).

---

## Error Handling

| Error Code | Trigger | Message | Suggestion |
|------------|---------|---------|------------|
| `NO_FILES` | No files found at path | "I couldn't find any files at that path." | "Check the path exists and contains files I can read." |
| `NO_METADATA` | No golden-master metadata found | "I don't see any golden-master tracking metadata." | "Run 'establish' first to set up tracking relationships." |
| `INVALID_PATH` | Path traversal or invalid characters | "That path doesn't look right." | "Use relative paths from project root, no '..' allowed." |
| `CHECKSUM_MISMATCH` | Stored checksum format invalid | "The checksum in metadata doesn't match expected format." | "Checksums should be 8+ hex characters. Was the file manually edited?" |

---

## Terminology Rules

| Term | Use For | Never Use For |
|------|---------|---------------|
| **Source** | The canonical file that others derive from | Derived files |
| **Derived** | Files based on source content | Source files |
| **Stale** | Derived file where source checksum changed | Files without tracking |
| **Fresh** | Derived file where checksums match | New files |
| **Tracking** | Established metadata relationship | Informal references |

---

## Related Skills

- **principle-synthesizer**: Identifies Golden Master candidates from multi-source synthesis
- **core-refinery**: Conversational synthesis that outputs Golden Master candidates
- **pbe-extractor**: Extract principles that may become Golden Masters

---

## Required Disclaimer

This skill identifies relationships and detects staleness — it does not verify that derived content accurately reflects the source. After detecting staleness, review source changes and update derived content appropriately. The skill tracks structure, not semantic correctness.

---

*Built by Obviously Not — Tools for thought, not conclusions.*
