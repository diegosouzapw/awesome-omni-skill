---
name: inquisitor-mode
description: Multi-agent autonomous workspace organizing system for Claude Code. Triggers on "Inquisitor Mode". Orchestrates parallel reader/classifier/renamer agents to scan directories recursively, read file contents (PDF, DOCX, XLSX, images, TXT, CSV, JSON, code files), extract dates and metadata, rename files by content and date, generate structured JSON manifests, detect duplicates, link related documents, and reorganize directory structures. Features dry-run preview before execution, rollback capability, incremental processing, OCR for scanned documents, confidence-based routing to human review, and parallel worker dispatch via Task tool with circuit breakers. Adapted from Loki Mode orchestration patterns.
---

# Inquisitor Mode - Multi-Agent Workspace Organizing System

> **Version 1.0.0** | Unstructured Files to Organized Knowledge | Dry-Run First
> Adapted from Loki Mode v2.35.0 orchestration patterns

---

## Quick Reference

### Critical First Steps (Every Turn)
1. **READ** `.inquisitor/CONTINUITY.md` - Your working memory
2. **CHECK** `.inquisitor/state/orchestrator.json` - Current phase/metrics
3. **REVIEW** `.inquisitor/queue/pending.json` - Next files to process
4. **FOLLOW** RARV cycle: REASON, ACT, REFLECT, **VERIFY**
5. **OPTIMIZE** Sonnet=classification/orchestration, Haiku=reading/renaming - 10+ Haiku readers in parallel
6. **TRACK** Efficiency metrics: files processed, time, agent count

### Key Files (Priority Order)
| File | Purpose | Update When |
|------|---------|-------------|
| `.inquisitor/CONTINUITY.md` | Working memory - what am I doing NOW? | Every turn |
| `.inquisitor/config/settings.json` | User configuration (categories, naming, thresholds) | Setup/reconfigure |
| `.inquisitor/state/orchestrator.json` | Master state (phase, metrics, progress) | Every phase change |
| `.inquisitor/queue/*.json` | File processing states | Every file change |
| `.inquisitor/manifest/workspace-manifest.json` | Final structured output | After each file processed |
| `.inquisitor/rollback/rollback.json` | Undo map (new name -> original) | After each rename |
| `.inquisitor/memory/semantic/patterns.json` | Learned document patterns | After batch completion |

### Decision Tree: What To Do Next?

```
START
  |
  +-- Read CONTINUITY.md ----------+
  |                                |
  +-- Task in-progress?            |
  |   +-- YES: Resume              |
  |   +-- NO: Check pending queue  |
  |                                |
  +-- Pending files?               |
  |   +-- YES: Dispatch readers    |
  |   +-- NO: Check phase          |
  |                                |
  +-- Phase done?                  |
  |   +-- YES: Advance             |
  |       (scan -> read -> classify |
  |        -> dry-run -> execute    |
  |        -> verify -> report)     |
  |   +-- NO: Generate tasks       |
  |                                |
LOOP <-----------------------------+
```

---

## Processing Phases

```
Scan -> Read -> Classify -> Plan -> Dry-Run -> [Approve] -> Execute -> Verify -> Report
  |       |        |         |        |            |           |         |         |
(Find  (Extract (Type,    (Rename  (Preview    (User      (Rename   (Check    (Generate
files)  content) dates,    plan,    changes,    says OK)   + move)   integrity) manifest)
               entities) folders)  show user)
```

### Phase Details

| Phase | Description | Agents Used |
|-------|-------------|-------------|
| **Scan** | Recursively walk target directory, build file inventory | orchestrator (Haiku) |
| **Read** | Extract text content from each file in parallel | reader-workers (Haiku x N) |
| **Classify** | Determine document type, extract dates, entities, summary | classifier (Sonnet) |
| **Plan** | Generate rename plan, folder structure, detect duplicates | planner (Sonnet) |
| **Dry-Run** | Present full preview to user: renames, moves, duplicates | orchestrator |
| **Approve** | HALT - Wait for user approval (only human interaction point) | - |
| **Execute** | Perform renames and directory reorganization | renamer (Haiku) |
| **Verify** | Validate all files accessible, hashes match, no data loss | verifier (Haiku) |
| **Report** | Generate workspace-manifest.json and summary | orchestrator |

---

## Core Autonomy Rules (Modified from Loki Mode)

**This system runs autonomously EXCEPT at the Approve phase.**

1. **NEVER ask questions during processing** - Make best-effort decisions
2. **NEVER wait for confirmation EXCEPT at dry-run approval** - This is the ONE human checkpoint
3. **NEVER delete files** - Only rename and move. Originals recoverable via rollback.json
4. **NEVER overwrite files** - If name collision, append incrementing suffix
5. **ALWAYS generate dry-run first** - No file operations without preview
6. **ALWAYS maintain rollback capability** - Every operation is reversible
7. **ALWAYS preserve file integrity** - Hash before and after, verify match

---

## Model Selection Strategy

| Model | Use For | Examples |
|-------|---------|----------|
| **Sonnet** | Classification, orchestration, planning | Document type classification, rename planning, relationship linking |
| **Haiku** | Reading, renaming, verification, scanning | File content extraction, applying renames, hash verification |

### Parallelization Strategy (Memory-Aware)

Batches are sized by **estimated token weight**, not file count:

```python
# During Scan: estimate load per file
for file in scanned_files:
    file.load_estimate = estimate_file_weight(file.size, file.reader_type)
    # Returns: estimated_tokens, weight_class (light/medium/heavy/very_heavy/oversized)

# During Read: build batches within memory budget
# OCR: max 120K tokens/batch, 3 workers  |  Direct: max 200K tokens/batch, 10 workers
batch, weight = build_memory_aware_batch(pending_files, reader_type)
parallelism = calculate_dynamic_parallelism(batch, reader_type)

# Dispatch batch (only as many as budget allows)
for file in batch[:parallelism]:
    Task(subagent_type="general-purpose", model="haiku",
         description=f"Read file: {file.name}",
         run_in_background=True,
         prompt=f"Extract all text content from {file.path}...")

# Oversized files (>80K estimated tokens): chunked reading, one at a time
# Text: line ranges | PDF: summary-only OCR | DOCX: condensed | Excel: sheet-by-sheet
```

See `references/tool-orchestration.md` for full load estimation and memory-aware dispatch.

---

## Agent Types Overview

| Agent | Count | Model | Role |
|-------|-------|-------|------|
| `orchestrator` | 1 | Sonnet | Scans dir, builds queue, dispatches workers, manages phases |
| `reader-worker` | N (parallel) | Haiku | Reads file content via appropriate tool (docx, pdf/OCR, xlsx, etc.) |
| `classifier` | 1-3 | Sonnet | Determines doc type, extracts dates/entities/summary, confidence score |
| `planner` | 1 | Sonnet | Generates rename plan, folder structure, duplicate detection |
| `renamer` | N (parallel) | Haiku | Applies renames and moves per approved plan |
| `verifier` | N (parallel) | Haiku | Hash verification, accessibility checks, manifest validation |
| `linker` | 1 | Sonnet | Discovers relationships between documents (references, invoices<->contracts) |

See `references/agent-types.md` for complete definitions.

---

## Supported File Types

| Category | Extensions | Reading Method |
|----------|-----------|----------------|
| Documents | `.pdf` | `doctr-ocr` MCP (OCR) or text extraction |
| Documents | `.docx` | `office` MCP (`read_docx`) |
| Spreadsheets | `.xlsx`, `.xls` | `excel` MCP (`read_data_from_excel`) |
| Text | `.txt`, `.md`, `.csv`, `.json` | Direct file read |
| Code | `.py`, `.js`, `.ts`, `.tsx`, `.jsx`, `.html`, `.css`, `.yaml`, `.toml` | Direct file read |
| Images | `.png`, `.jpg`, `.jpeg`, `.tiff`, `.bmp` | `doctr-ocr` MCP (OCR text extraction) |

See `references/file-types.md` for reading strategies per type.

---

## Naming Convention

```
YYYY-MM-DD_DocumentType_ShortDescription[_Entity]_Hash4.ext
```

### Date Priority (highest to lowest)
1. Signature date found in document
2. Explicit "Date:" field in document
3. Document creation date in metadata
4. File system modification date

### Examples
```
2025-11-14_Contract_OfficeLeaseAgreement_AcmeCorp_a3f2.pdf
2026-01-10_Invoice_CloudHostingJanuary_AWS_b7e1.xlsx
2024-06-03_Letter_EmploymentOffer_JohnDoe_c9d4.docx
2026-01-15_Code_ReactAuthComponent_d8a1.tsx
2025-09-20_Spreadsheet_Q3BudgetForecast_e5b3.xlsx
2026-01-31_Report_AnnualSalesAnalysis_f2c7.pdf
```

See `references/naming-convention.md` for full rules.

---

## Dry-Run Output Format

Before any file operations, the system presents:

```
============================================================
INQUISITOR MODE - DRY RUN PREVIEW
============================================================
Target: F:/documents/unsorted (recursive)
Files found: 47
Duplicates detected: 3

RENAMES:
  scan_0042.pdf
    -> Contracts/2025-11-14_Contract_OfficeLeaseAgreement_a3f2.pdf
    Confidence: 0.92 | Dates: signature=2025-11-14

  IMG_20260110.jpg
    -> Invoices/2026-01-10_Invoice_CloudHosting_b7e1.jpg
    Confidence: 0.78 | Dates: doc_date=2026-01-10

  untitled(3).docx
    -> Review/untitled(3)_c9d4.docx
    Confidence: 0.45 | LOW CONFIDENCE - needs human review

DUPLICATES:
  report_v2.pdf == report_final.pdf (sha256 match)
    -> Keep: report_v2.pdf (newer modified date)
    -> Flag: report_final.pdf (moved to Duplicates/)

NEW FOLDERS:
  + Contracts/
  + Invoices/
  + Review/
  + Duplicates/
  + Code/
  + Correspondence/

============================================================
Approve? Execute with: "Inquisitor execute"
Modify plan with: "Inquisitor adjust [instructions]"
Cancel with: "Inquisitor cancel"
============================================================
```

---

## JSON Manifest Schema

Output: `.inquisitor/manifest/workspace-manifest.json`

```json
{
  "version": "1.0.0",
  "processed_at": "ISO timestamp",
  "source_directory": "path",
  "recursive": true,
  "total_files_scanned": 47,
  "total_files_processed": 44,
  "duplicates_found": 3,
  "low_confidence_files": 2,
  "categories_created": ["Contracts", "Invoices", "..."],
  "files": [
    {
      "original_name": "string",
      "original_path": "string (relative to source)",
      "new_name": "string",
      "new_path": "string (relative to source)",
      "document_type": "Contract|Invoice|Letter|Report|Spreadsheet|Code|Image|Other",
      "category": "Legal|Financial|HR|Technical|Correspondence|Other",
      "dates": {
        "primary_date": "YYYY-MM-DD",
        "date_source": "signature|document|metadata|filesystem",
        "signature_date": null,
        "effective_date": null,
        "expiration_date": null,
        "all_dates_found": []
      },
      "entities": {
        "parties": [],
        "people": [],
        "organizations": [],
        "amounts": [],
        "addresses": [],
        "emails": [],
        "phone_numbers": []
      },
      "summary": "1-2 sentence description",
      "tags": [],
      "language": "en",
      "confidence": 0.92,
      "relationships": [
        {"related_file": "other_file.pdf", "relation": "references|invoice_for|amendment_to|reply_to|version_of"}
      ],
      "file_hash": "sha256:...",
      "file_size_bytes": 245000,
      "original_metadata": {
        "created": "ISO",
        "modified": "ISO",
        "extension": ".pdf"
      },
      "processing": {
        "read_method": "ocr|docx_reader|excel_reader|direct_read",
        "read_duration_ms": 1200,
        "classification_model": "sonnet",
        "ocr_confidence": 0.88
      }
    }
  ],
  "duplicate_groups": [
    {
      "hash": "sha256:...",
      "files": ["path1", "path2"],
      "kept": "path1",
      "reason": "newer_modified_date"
    }
  ],
  "statistics": {
    "by_category": {"Legal": 12, "Financial": 8},
    "by_type": {"Contract": 7, "Invoice": 5},
    "by_year": {"2025": 20, "2026": 15},
    "avg_confidence": 0.87,
    "processing_time_seconds": 142
  }
}
```

See `references/manifest-schema.md` for full schema documentation.

---

## Incremental Processing

Track processed files to avoid re-processing:

```json
// .inquisitor/state/processed-files.json
{
  "files": {
    "sha256:abc123...": {
      "original_path": "docs/contract.pdf",
      "current_path": "Contracts/2025-11-14_Contract_Lease_a3f2.pdf",
      "processed_at": "ISO",
      "manifest_entry_id": 0
    }
  }
}
```

On re-run: compute hash of each file, skip if already in processed-files.json.

---

## Rollback System

```json
// .inquisitor/rollback/rollback.json
{
  "session_id": "uuid",
  "executed_at": "ISO",
  "operations": [
    {
      "type": "rename",
      "from": "scan_0042.pdf",
      "to": "Contracts/2025-11-14_Contract_OfficeLeaseAgreement_a3f2.pdf",
      "hash_before": "sha256:...",
      "hash_after": "sha256:..."
    },
    {
      "type": "mkdir",
      "path": "Contracts/"
    }
  ]
}
```

Rollback command: `Inquisitor rollback` — reverses all operations in reverse order.

---

## Confidence Routing

| Confidence | Action |
|------------|--------|
| >= 0.85 | Auto-classify and rename |
| 0.70 - 0.84 | Auto-classify, flag for review in dry-run |
| 0.50 - 0.69 | Place in `Review/` folder with original name preserved |
| < 0.50 | Place in `Uncategorized/` folder, minimal rename (date + hash only) |

---

## Duplicate Detection

1. Compute SHA-256 hash for every file during Scan phase
2. Group files by hash
3. For duplicates:
   - Keep the file with the most recent modification date
   - Move others to `Duplicates/` folder
   - Log in manifest `duplicate_groups`
4. Present in dry-run for user approval

---

## Relationship Linking

The `linker` agent examines extracted content to find cross-references:

| Relation Type | Detection Method |
|---------------|-----------------|
| `references` | File A mentions File B's title, number, or ID |
| `invoice_for` | Invoice references a contract number or PO |
| `amendment_to` | Amendment document references original contract |
| `reply_to` | Email/letter references another correspondence |
| `version_of` | Similar content with version indicators (v1, v2, draft, final) |
| `attachment_of` | File referenced as attachment in another document |

---

## Load Estimation and Memory Safety

Files are not batched by count — they're batched by **estimated token weight** to prevent worker context overflow:

| Weight Class | Estimated Tokens | Handling |
|-------------|-----------------|----------|
| `light` | < 5K | Full parallel batch (up to 10 workers) |
| `medium` | 5K - 20K | Full parallel batch |
| `heavy` | 20K - 50K | Reduced parallelism |
| `very_heavy` | 50K - 80K | Minimal parallelism (1-2 workers) |
| `oversized` | > 80K | **Chunked reading** — one file at a time, sequential |

**Chunked reading** for oversized files extracts head + tail + dates + entities + summary instead of full text. Classification accuracy is preserved because key metadata is always captured.

**Orchestrator budget**: Results are flushed to disk when accumulated context exceeds 400K tokens, keeping only 200-char summaries in memory.

**Calibration**: After each batch, predicted vs actual token counts are compared. Weight multipliers auto-adjust if estimates are consistently off.

See `references/tool-orchestration.md` for full implementation.

---

## Circuit Breakers

| Agent Type | Failure Threshold | Cooldown | Notes |
|------------|-------------------|----------|-------|
| `reader-worker` | 5 consecutive | 60s | OCR failures, corrupt files |
| `classifier` | 3 consecutive | 120s | Classification errors |
| `renamer` | 2 consecutive | 300s | File system errors are serious |
| `verifier` | 3 consecutive | 60s | Hash mismatches |

---

## Invocation

```
Inquisitor Mode                              # Start with current directory
Inquisitor Mode scan F:/path/to/directory    # Start with specific directory
Inquisitor execute                           # Execute approved dry-run plan
Inquisitor adjust [instructions]             # Modify plan before execution
Inquisitor cancel                            # Cancel current operation
Inquisitor rollback                          # Undo last execution
Inquisitor status                            # Show processing progress
```

**Skill Metadata:**
| Field | Value |
|-------|-------|
| Trigger | "Inquisitor Mode" or "Inquisitor Mode scan [path]" |
| Skip When | Single file rename, already organized directory |
| Dependencies | MCP tools: doctr-ocr, office (read_docx), excel |

---

## References

| Reference | Content |
|-----------|---------|
| `references/core-workflow.md` | RARV cycle adapted for document processing |
| `references/agent-types.md` | All agent types with capabilities |
| `references/task-queue.md` | Queue system for file processing |
| `references/tool-orchestration.md` | Efficiency tracking and parallel dispatch |
| `references/file-types.md` | Supported file types and reading strategies |
| `references/naming-convention.md` | Renaming rules and date extraction |
| `references/manifest-schema.md` | JSON manifest and relationship linking |
| `references/memory-system.md` | Episodic/semantic memory for document patterns |

---

**Version:** 1.0.0 | Adapted from Loki Mode v2.35.0 orchestration patterns
