---
name: api-reference
description: PDF-RAG API reference. REQUIRED after any failed curl/jq to localhost:8000 (404, null, jq error). Also use when uncertain about endpoint path or response shape.
---

# PDF-RAG API Skill

**Use this skill when:** curl/jq fails, response shape is wrong, or you're unsure of the endpoint.

## Cheat Sheet

### Response Paths
| Endpoint | jq path |
|----------|---------|
| parties | `.resolved.canonical_parties[].canonical_name` |
| dates | `.resolved.governing_date` |
| land | `.resolved.canonical_tracts[].tract_identity_phrase` |
| doc_type | `.resolved.doc_category` |
| conveyance | `.resolved.conveys_fee` |
| documents list | `.items[]` |
| packages list | `.packages[]` |
| package docs | `.documents[]` |
| chain edges | `.chains["fee"].dag.edges[]` |
| estate graph nodes | `.dag.nodes[]` |
| estate graph summary | `.summary` |
| estate graph gaps | `.dag.gaps[]` |

### Stage Names
```
extract_parties, extract_dates, extract_land_description, extract_conveyance, extract_document_classification
markdown, segmentation, assembly, runsheet, runtime_linking
```

## Shell Command Pattern

**IMPORTANT:** Never use shell variable interpolation in curl commands. Always inline values directly:

```bash
# WRONG - variable interpolation can fail
DOC_ID="abc123" && curl -s "http://localhost:8000/api/documents/$DOC_ID/parties"

# RIGHT - inline the value directly
curl -s "http://localhost:8000/api/documents/abc123/parties"
```

## Gotchas & Common Mistakes

### Estate Graph (L3)
- **GET vs POST**: `GET` retrieves stored graph, `POST` rebuilds from L0-L2 data
- **estate_type is lowercase**: `"fee"`, `"mineral"`, `"leasehold"` (not uppercase)
- **Chain = nodes grouped by tract**: A "mineral chain" = all mineral nodes sharing same `tract_id`
- **Quick summary**: `.summary.mineral_estates` gives count
- **List nodes by type**: `.dag.nodes[] | select(.estate_type == "mineral")`
- **Group into chains**: `[.dag.nodes[] | select(.estate_type == "mineral")] | group_by(.tract_id)`
- **Count chains**: Above query `| length`

### Rebuild Estate Graph (Exploratory Mode 1.1)

**Endpoint**: `POST /api/packages/{package_id}/estate-graph`

**Request body** (JSON):
```json
{"version": "1.1"}
```

**Full curl command**:
```bash
curl -s -X POST "http://localhost:8000/api/packages/{package_id}/estate-graph" \
  -H "Content-Type: application/json" \
  -d '{"version": "1.1"}'
```

**Version options**:
- `"1.0"`: Exploratory Layers - POC 5-layer fuzzy matcher (L0-L4 gates)
- `"1.1"`: Exploratory Signals - EdgeScore-based matcher (includes comparison field)
- `"2.0"`: Operative deterministic matcher (exact matches only)

**Response paths**:
- Nodes: `.dag.nodes[]`
- Edges: `.dag.edges[]`
- Gaps: `.dag.gaps[]`
- Summary: `.summary`
- Exploratory outcomes (v1.1): `.exploratory_outcomes`
- Winner/runner-up comparison (v1.1): `.exploratory_outcomes[estate_id].comparison`

**Comparison data (v1.1)**:
```
.exploratory_outcomes[estate_id].comparison:
  .winner_id: string
  .runner_up_id: string | null
  .rank_delta: number
  .top_winner_reasons[]: {signal, winner_state, runner_up_state, differentiation}
  .top_runner_up_losses[]: {signal, issue}
  .suppression_analysis: {eligible, blocking_contradictions[]}
```

### Chain Analysis (deprecated - use Estate Graph)
- **Use the endpoint, not Package.meta**: Chain analysis results are in `GET /packages/{id}/chain-analysis`, not buried in `Package.meta.chain_analysis`
- **L2 before L3**: Run L2 clustering (`POST /packages/{id}/l2/parties`) BEFORE chain analysis for proper cross-document party matching
- **Chain types are dict keys**: Access via `.chains["fee"]`, `.chains["mineral"]`, `.chains["leasehold"]`

### Extraction Endpoints
- **Always use `.resolved`**: The clean data is in `.resolved`, raw LLM output is in `.extraction` (debug mode only)
- **Debug mode**: Add `?debug=true` to get raw extraction + normalized + extraction_context
- **Party IDs are document-scoped**: `party_id` is unique within a document, use L2 clusters for cross-doc identity

### List Endpoints
- **Documents list**: `.items[]` (paginated response)
- **Packages list**: `.packages[]` (NOT `.[]`)
- **Package documents**: `.documents[]` (nested in package detail)
- **Chain edges**: `.chains["fee"].dag.edges[]`

### Processing
- **Stages array**: POST /api/process takes `stages: ["extract_parties", "extract_dates", ...]`
- **Poll for completion**: GET /api/jobs/{job_id} until status is "completed" or "failed"
- **enrich_only=true**: Re-run L1 enrichment without re-running L0 extraction (saves LLM cost)

---

**Need full schema details?** Read `.claude/skills/api-reference/API_REFERENCE.md`
