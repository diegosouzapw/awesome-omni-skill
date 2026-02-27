---
name: scaffold-bulk-review-prototypes
description: Review all prototypes at once for cross-prototype consistency, coverage gaps, ADR follow-through, and scope discipline. Use for a full audit of all prototypes.
allowed-tools: Read, Grep, Glob
---

# Bulk Prototype Review

Review every registered prototype for completeness, quality, and cross-prototype consistency.

## Steps

### 1. Gather All Prototypes

1. **Read the prototypes index** at `scaffold/prototypes/_index.md`.
2. **Read every prototype file** in `scaffold/prototypes/` (Glob `scaffold/prototypes/PROTO-*.md`).
3. **Read the design doc** at `scaffold/design/design-doc.md`.
4. **Read all system designs** — Glob `scaffold/design/systems/SYS-*.md`.
5. **Read all engine docs** from `scaffold/engine/`.
6. **Read all ADRs** — Glob `scaffold/decisions/ADR-*.md`.
7. **Read known issues** at `scaffold/decisions/known-issues.md`.
8. **Read design debt** at `scaffold/decisions/design-debt.md`.
9. If no prototypes exist, report that and stop.

### 2. Per-Prototype Completeness

For each prototype, check pre-spike sections (all prototypes) and post-spike sections (Complete prototypes only):

**Pre-Spike:**

| Section | What "Complete" Means |
|---------|----------------------|
| Question | ONE specific, answerable question |
| Hypothesis | Expected answer with reasoning |
| Scope: Build | Numbered list of minimum steps |
| Scope: Skip | At least 2 explicit exclusions |
| Approach | Numbered execution steps |
| Related Documents | At least 1 valid scaffold doc reference |

**Post-Spike (if Status is Complete):**

| Section | What "Complete" Means |
|---------|----------------------|
| Answer | Direct answer to the Question |
| Evidence | Specific measurements or observations |
| Surprises | Findings listed or explicitly "None" |
| Design Impact | Table of affected docs with changes |
| ADRs Filed | Every Design Impact has a corresponding ADR |
| Disposition | Discarded / Archived / Absorbed with reason |

### 3. Per-Prototype Quality

For each prototype, check:
- **Question is specific and singular** — not compound or vague.
- **Scope is minimal** — Build scope is the minimum to answer the question.
- **Skip boundaries are explicit** — clear discipline boundary.
- **Right-sized** — achievable in one spike session.

### 4. Cross-Prototype Consistency

This is unique to bulk review — check relationships BETWEEN prototypes:

- **Question overlap.** Flag prototypes with substantially similar questions. Suggest merging or clarifying the distinction.
- **Scope overlap.** Flag prototypes with overlapping Build scopes — they may be doing redundant work.
- **Coverage gaps.** Cross-reference with systems, engine docs, slices, and known issues. Flag areas of significant uncertainty that have no prototype.
- **ADR follow-through.** For Complete prototypes: verify every Design Impact entry has a filed ADR. Flag missing ADRs.
- **Disposition consistency.** Flag Absorbed prototypes without a referenced TASK-###. Flag prototypes marked Complete but with empty post-spike sections.
- **Stale prototypes.** Flag Draft prototypes that reference documents which have since changed significantly or been deprecated.
- **Status consistency.** Check that index status matches file status for every prototype.

### 5. Registration Check

- Every prototype in `scaffold/prototypes/_index.md` must have a corresponding file.
- Every prototype file must be registered in `scaffold/prototypes/_index.md`.
- IDs, names, statuses, and dispositions must match between files and index.

## Output Format

```
## Bulk Prototype Review — X prototypes audited

### Overview
| ID | Name | Status | Pre-Spike | Post-Spike | Quality Issues | ADRs |
|----|------|--------|-----------|------------|----------------|------|
| PROTO-001 | ... | Draft | 5/6 | — | 1 issue | — |
| PROTO-002 | ... | Complete | 6/6 | 5/6 | 0 issues | 2 |

### Per-Prototype Details

#### PROTO-001 — [Name]
| Section | Status | Notes |
|---------|--------|-------|
| ... | ... | ... |

Quality Issues:
- [specific issues with quotes]

(repeat for each prototype)

### Cross-Prototype Consistency
- **Question overlap:** [OK / overlapping prototypes found]
- **Scope overlap:** [OK / redundant Build scopes found]
- **Coverage gaps:** [OK / areas without prototypes]
- **ADR follow-through:** [OK / X missing ADRs]
- **Disposition consistency:** [OK / issues found]
- **Stale prototypes:** [OK / X stale prototypes]

### Registration
[Any mismatches between index and files]

### Recommendations (prioritized)
1. [Most impactful fix across all prototypes]
2. [Second priority]
3. ...
```

## Rules

- This skill is read-only. Do not modify any files.
- **Cross-prototype checks are the main value** of bulk review over individual `/scaffold-review-prototype` calls. Emphasize relationship issues.
- Be specific. Quote problematic text when flagging issues.
- **Coverage gaps are high-value findings.** Identifying what SHOULD be prototyped but isn't is as important as auditing existing prototypes.
- If all prototypes are well-written, say so. Don't manufacture issues.
- Prioritize recommendations by blast radius — issues that affect multiple prototypes or the pipeline rank higher.
