---
name: markdown-consolidator
description: "Intelligent consolidation and synthesis of multiple markdown files with overlapping content and different update dates. Use when: (1) Multiple AI-generated markdown files need merging, (2) Knowledge bases have fragmented or duplicate content, (3) Documentation requires recency-aware synthesis, (4) Supporting documents need re-synthesis after AI task completion, (5) Project documentation has semantic overlap across files, (6) Periodic knowledge base maintenance and deduplication is needed."
---

# Markdown Consolidator

Consolidate and synthesize multiple markdown files with intelligent handling of overlapping content, different update dates, and semantic deduplication.

## Core Problem

AI-assisted workflows generate fragmented documentation:
- Each AI session creates task-specific markdown files
- AI references supporting docs but doesn't update them post-task
- Knowledge becomes scattered across files with overlapping content
- Different timestamps make version reconciliation complex

## Workflow Overview

```
1. ANALYZE  → Inventory files, extract metadata, identify relationships
2. CLUSTER  → Group semantically related files using content analysis
3. PLAN     → Create merge strategy based on recency, overlap, authority
4. SYNTHESIZE → Merge content with intelligent conflict resolution
5. VALIDATE → Verify completeness and coherence of output
```

## Analysis Phase

### Step 1: File Inventory

Run the inventory script to analyze all markdown files:

```bash
python scripts/inventory.py <directory> --output inventory.json
```

The script extracts:
- File paths and sizes
- Modification timestamps (file system and YAML frontmatter)
- Section headers (H1-H6 structure)
- Word/token counts per section
- Internal links (`[[wikilinks]]` and `[markdown](links)`)
- YAML frontmatter metadata
- Content fingerprints for similarity detection

### Step 2: Relationship Mapping

```bash
python scripts/analyze_relationships.py inventory.json --output relationships.json
```

Identifies:
- **Semantic clusters**: Files covering similar topics (via TF-IDF/embedding similarity)
- **Temporal chains**: Files that evolved from each other (via timestamp + similarity)
- **Reference graphs**: Which files reference which (via link analysis)
- **Conflict zones**: Sections with contradictory or overlapping content

## Clustering Phase

### Clustering Strategies

Choose based on your consolidation goal:

**Topic-based clustering** (default)
Groups files by semantic similarity of content.
```bash
python scripts/cluster.py relationships.json --method topic --threshold 0.6
```

**Temporal clustering**
Groups files by modification date ranges.
```bash
python scripts/cluster.py relationships.json --method temporal --window 7d
```

**Hierarchical clustering**
Groups by directory structure + content similarity.
```bash
python scripts/cluster.py relationships.json --method hierarchical
```

### Cluster Output

Creates `clusters.json` with structure:
```json
{
  "clusters": [
    {
      "id": "cluster_001",
      "theme": "API Authentication",
      "files": ["auth-design.md", "oauth-notes.md", "token-handling.md"],
      "primary_file": "auth-design.md",
      "overlap_score": 0.72,
      "conflicts": ["token-handling.md:L45 vs oauth-notes.md:L23"]
    }
  ]
}
```

## Planning Phase

### Merge Strategy Selection

**Authority-based** (recommended for documentation)
- Most recent file is authoritative for conflicts
- Older unique content is preserved with attribution
- Use when files represent evolving understanding

**Comprehensive** (for knowledge bases)
- Union of all unique information
- Conflicts flagged for manual review
- Use when completeness matters more than consistency

**Canonical** (for specifications)
- Designate one file as canonical
- Others provide supplementary/historical context
- Use when single source of truth is required

### Create Merge Plan

```bash
python scripts/plan_merge.py clusters.json --strategy authority --output merge_plan.json
```

Generates actionable merge plan:
```json
{
  "cluster_id": "cluster_001",
  "output_file": "consolidated/authentication.md",
  "sections": [
    {
      "heading": "## Overview",
      "sources": [{"file": "auth-design.md", "lines": "1-25", "action": "primary"}],
      "conflicts": []
    },
    {
      "heading": "## Token Handling",
      "sources": [
        {"file": "token-handling.md", "lines": "10-45", "action": "primary"},
        {"file": "oauth-notes.md", "lines": "20-35", "action": "supplement"}
      ],
      "conflicts": [
        {
          "description": "Token expiry differs: 24h vs 1h",
          "resolution": "Use most recent (token-handling.md: 24h)"
        }
      ]
    }
  ]
}
```

## Synthesis Phase

### Execute Merge

```bash
python scripts/synthesize.py merge_plan.json --output consolidated/
```

The synthesizer:
1. Creates section-by-section merged content
2. Preserves original attribution via HTML comments
3. Resolves conflicts per strategy
4. Maintains internal link consistency
5. Updates frontmatter with merge metadata

### Synthesis Rules

**Content Deduplication**
- Exact duplicates: Remove, keep first occurrence
- Near duplicates (>80% similarity): Merge, note sources
- Partial overlap: Keep both with clear section breaks

**Conflict Resolution**
```
Authority strategy:
  1. Prefer most recently modified source
  2. Prefer explicitly dated content over undated
  3. Prefer longer/more detailed explanations
  4. Flag unresolvable conflicts for review

Comprehensive strategy:
  1. Include all non-contradictory content
  2. Present conflicts as "Version A / Version B" blocks
  3. Add TODO markers for manual resolution
```

**Link Handling**
- Internal links updated to point to consolidated files
- Broken links flagged with `<!-- BROKEN: original-target.md -->`
- External links preserved as-is

### Output Format

Consolidated files include:
```markdown
---
title: Authentication System
consolidated_from:
  - file: auth-design.md
    modified: 2024-12-01T10:30:00
  - file: oauth-notes.md
    modified: 2024-11-28T15:45:00
  - file: token-handling.md
    modified: 2024-12-02T09:00:00
consolidated_at: 2024-12-03T14:00:00
strategy: authority
---

# Authentication System

<!-- SOURCE: auth-design.md:1-25 -->
## Overview
...

<!-- SOURCE: token-handling.md:10-45, SUPPLEMENTED: oauth-notes.md:20-35 -->
## Token Handling
...

<!-- CONFLICT RESOLVED: Used token-handling.md (most recent) -->
Token expiry is set to 24 hours...
```

## Validation Phase

```bash
python scripts/validate.py consolidated/ --original <source_dir>
```

Validates:
- **Completeness**: All source content represented or explicitly excluded
- **Link integrity**: All internal links resolve
- **Coherence**: No contradictions in final output
- **Metadata**: Proper attribution and timestamps

Generates `validation_report.md`:
```markdown
## Consolidation Validation Report

### Coverage
- 47/47 source files processed
- 3 files excluded (empty/invalid)
- 12 clusters created
- 8 consolidated files produced

### Content Coverage
- 98.3% of source content preserved
- 1.7% deduplicated (exact matches)
- 5 conflicts resolved automatically
- 2 conflicts flagged for review

### Issues
- [ ] REVIEW: consolidated/auth.md:L145 - conflicting token formats
- [ ] REVIEW: consolidated/api.md:L67 - unclear which version is correct
```

## Quick Start

For immediate consolidation of a directory:

```bash
# Full pipeline
python scripts/consolidate.py <source_dir> <output_dir> --strategy authority

# This runs: inventory → analyze → cluster → plan → synthesize → validate
```

## Advanced: Incremental Updates

For ongoing maintenance:

```bash
# Detect changes since last consolidation
python scripts/detect_changes.py <source_dir> --since "2024-12-01"

# Re-consolidate only affected clusters
python scripts/consolidate.py <source_dir> <output_dir> --incremental
```

## Configuration

Create `.consolidator.yaml` in project root:

```yaml
# Files/directories to exclude
exclude:
  - "**/archive/**"
  - "**/.obsidian/**"
  - "**/templates/**"

# Similarity threshold for clustering (0-1)
similarity_threshold: 0.6

# Default merge strategy
default_strategy: authority

# Preserve original files
keep_originals: true
archive_path: .consolidated-archive/

# Frontmatter fields to preserve
preserve_frontmatter:
  - tags
  - aliases
  - created

# Output format
output:
  add_source_comments: true
  add_merge_frontmatter: true
  update_internal_links: true
```

## Integration Patterns

### With Claude Code Sessions

Add to your CLAUDE.md:
```markdown
## Post-Task Consolidation

After completing any task that creates or modifies markdown files:
1. Run `/project:consolidate` to update knowledge base
2. Review flagged conflicts in validation report
3. Archive original files if consolidation successful
```

### With Basic Memory MCP

The consolidator can output in Basic Memory format:
```bash
python scripts/synthesize.py merge_plan.json --format basic-memory
```

Outputs files with observation/relation syntax compatible with Basic Memory's knowledge graph.

## Reference Documentation

- [ALGORITHMS.md](references/ALGORITHMS.md) - Detailed similarity/clustering algorithms
- [CONFLICT-RESOLUTION.md](references/CONFLICT-RESOLUTION.md) - Conflict handling patterns
- [INTEGRATION.md](references/INTEGRATION.md) - Integration with other tools
