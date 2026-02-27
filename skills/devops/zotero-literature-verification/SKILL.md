---
name: zotero-literature-verification
description: Complete workflow for verifying academic literature citations using Zotero MCP with full PDF reading and token management
---

# Zotero Literature Verification

Complete workflow for verifying academic literature citations using Zotero MCP with **100% word-by-word PDF reading** and **token budget management**.

## Quick Start

When invoked with `/zotero-literature-verification`, guide the user through:

1. **Token Budget Estimation** - Calculate required tokens based on page count
2. **PDF Extraction** - Extract complete PDF text using PyMuPDF
3. **Sequential Reading** - Read every word from first to last page in chunks
4. **Citation Verification** - Verify citations with line numbers and exact quotes
5. **Reference Generation** - Generate ACM-formatted reference list from Zotero

## Zero-Tolerance Protocol ⚠️

**CRITICAL**: When verifying citations, you MUST:
- ✅ Extract COMPLETE PDF text to /tmp
- ✅ Read EVERY word from first page to last page
- ✅ Record line numbers for all verified citations
- ✅ NEVER use keyword search as substitute for complete reading
- ✅ Monitor token usage after each paper

**FORBIDDEN**:
- ❌ Reading only Abstract and Conclusion
- ❌ Using grep/search without full text reading
- ❌ Skipping middle sections of papers
- ❌ Assuming content without reading

## Token Budget Estimation

**Formula**: `Pages × 700 + 1000 = Estimated Tokens`

| Papers | Pages | Est. Tokens | Safe? |
|--------|-------|-------------|-------|
| 1-2 | 10-20 | ~20k | ✅ Safe |
| 3-4 | 30-40 | ~50k | ✅ Safe |
| 5-6 | 50-90 | ~90k | ✅ Safe |
| 7+ | 100+ | ~140k+ | ⚠️ Split sessions |

**Token Zones**:
- **Safe**: < 100k used (100k+ remaining)
- **Caution**: 100-150k used (50-100k remaining)
- **Danger**: > 150k used (< 50k remaining) → Pause and wait for user

## Workflow

### Step 1: Extract PDF

```python
import fitz
doc = fitz.open('/Users/Zhuanz/Zotero/storage/XXXXXXXX/Paper.pdf')
text = '\n'.join([p.get_text() for p in doc])
with open('/tmp/paper_full.txt', 'w') as f:
    f.write(text)
print(f"✅ {doc.page_count} pages, {len(text)} chars")
```

### Step 2: Read Sequentially

**Read in chunks of 250-300 lines**:

```python
# Chunk 1: Lines 0-250 (Title, Abstract, Introduction)
Read("/tmp/paper_full.txt", offset=0, limit=250)

# Chunk 2: Lines 250-500 (Methods, early Results)
Read("/tmp/paper_full.txt", offset=250, limit=250)

# Chunk 3: Lines 500-750 (Results, Discussion)
Read("/tmp/paper_full.txt", offset=500, limit=250)

# Chunk 4: Lines 750-1000 (Conclusion, References)
Read("/tmp/paper_full.txt", offset=750, limit=250)
```

### Step 3: Verify Citations

After complete reading, locate exact quotes:

```bash
# Find exact line number
grep -n "exact quoted phrase" /tmp/paper_full.txt

# Read context (±20 lines)
Read("/tmp/paper_full.txt", offset=436, limit=40)
```

### Step 4: Get Metadata from Zotero

Use Zotero MCP tools:
- `mcp__zotero__zotero_get_item_metadata` - Get complete metadata
- `mcp__zotero__zotero_search_items` - Search by author/year/title
- `mcp__zotero__zotero_get_item_fulltext` - Get PDF text

### Step 5: Generate Report

```markdown
## Verification Report

| Paper | Pages | Status | Issues |
|-------|-------|--------|--------|
| Author 2025 | 15 | ✅ | Corrected: false attribution |
| Author 2024 | 19 | ✅ | None - accurate |

## References (ACM Format)

[Author Year] FirstName LastName, FirstName LastName. Year.
Title of Paper. In Proceedings of CONF (CONF 'YY), Vol. 19.
AAAI Press, Pages. DOI: https://doi.org/XX.XXXX/XXXXXXX
```

## Token Management

**Monitor after each paper**:
```python
if tokens_remaining < 50000:
    print("⚠️ WARNING: Less than 50k tokens remaining")
    print("Recommend: Save progress and continue in new session")
```

**Real Performance** (from 2026-02-05):
- 6 papers, 91 pages, 369,478 characters
- Token used: 113,000 / 200,000 (56.5%)
- Time: ~2 hours
- Result: 100% accurate verification

## Emergency Procedures

### Token Budget Exhausted (< 30k remaining)

```bash
# Save progress
cat > /tmp/verification_progress.txt << EOF
Completed: Paper1, Paper2, Paper3
Current: Paper4 (line 500/1200)
Pending: Paper5, Paper6
Token used: 150,000
EOF

# Report to user and STOP
```

## Quality Checklist

Before claiming "verification complete":
- [ ] Read complete text (first page → last page)
- [ ] Located References section (proves completeness)
- [ ] Recorded line numbers for all citations
- [ ] Verified numerical data (N=X, p<0.05)
- [ ] Checked author's evaluation words
- [ ] Collected complete metadata from Zotero
- [ ] Generated ACM-formatted reference list
- [ ] Stayed within token budget

## Dependencies

```bash
# Install PyMuPDF
pip install PyMuPDF

# Ensure Zotero is running with local API enabled
# Settings → Advanced → "Allow other applications to communicate with Zotero"
```

## Documentation

- `instructions.md` - Complete workflow details
- `QUICK_REFERENCE.md` - Quick reference card
- `example_workflow.py` - Working example
- `README.md` - Project overview

## Version History

- **v2.0.0** (2026-02-05): Added token management, 6-paper verification workflow
- **v1.0.0** (2026-02-02): Initial release
