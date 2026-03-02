---
name: architecture-documentation-creator
description: Create comprehensive technical documentation for code systems including data flow diagrams, architecture overviews, algorithm documentation, cheat sheets, and multi-file documentation sets. Use when documenting pipelines, algorithms, system architecture, data flow, multi-stage processes, similarity algorithms, or creating developer onboarding materials. Covers Mermaid diagrams, progressive disclosure, critical patterns, JSON schemas, Pydantic models, and print-friendly reference materials.
---

# Architecture Documentation Creator

## Purpose

This skill provides a structured approach to creating comprehensive technical documentation for complex code systems, including data flow diagrams, algorithm documentation, architecture overviews, and quick reference materials. Based on proven patterns from successful documentation projects.

## When to Use This Skill

Use this skill when you need to:
- Document multi-stage pipelines or data flow systems
- Create architecture documentation for complex systems
- Document algorithms with multiple phases or layers
- Create onboarding materials for new developers
- Build comprehensive documentation sets (README + detailed docs + cheat sheets)
- Document systems that bridge multiple languages or technologies
- Create visual data flow diagrams with Mermaid
- Document critical implementation patterns
- Build troubleshooting guides for complex systems

**Trigger Keywords**: document architecture, create documentation, data flow diagram, document pipeline, document algorithm, architecture overview, technical documentation, developer documentation, onboarding docs, cheat sheet, mermaid diagram, document system

## Documentation Structure

### The Three-File Pattern

For comprehensive system documentation, create three complementary files:

**1. README.md** (Navigation Hub)
- Overview of the system
- Quick reference guide
- Links to detailed documentation
- Getting started section
- Critical patterns at-a-glance
- **Target length**: 400-500 lines

**2. Detailed Technical Documentation** (Deep Dive)
- Complete component breakdown
- Data flow diagrams (Mermaid)
- JSON schemas and data models
- Stage-by-stage or component-by-component analysis
- Performance characteristics
- Error handling
- **Target length**: 800-1,200 lines

**3. CHEAT-SHEET.md** (Quick Reference)
- One-page print-friendly reference
- Critical patterns with ✅/❌ examples
- Quick reference tables
- Common commands
- Troubleshooting guide
- File location references
- **Target length**: 200-300 lines

### File Naming Conventions

```
docs/architecture/
├── README.md                          # Start here
├── {system-name}-data-flow.md        # Detailed pipeline/flow docs
├── {algorithm-name}.md               # Algorithm deep dives
├── CHEAT-SHEET.md                    # Print-friendly reference
└── diagrams/                         # Optional: separate diagram files
```

## Creating Data Flow Diagrams

### Mermaid Diagram Best Practices

**1. Use Appropriate Diagram Types**:
- **Flowchart**: Sequential processes, decision trees
- **Sequence Diagram**: Component interactions over time
- **State Diagram**: State machines, lifecycle flows
- **Graph**: Data flow, dependencies

**2. Progressive Detail Pattern**:
```markdown
## High-Level Overview
[Simple 5-10 node diagram showing major components]

## Component Breakdown
[Detailed diagrams for each component/stage]

## Complete Flow
[Comprehensive end-to-end diagram]
```

**3. Mermaid Flowchart Example**:
```markdown
\`\`\`mermaid
flowchart TD
    A[Stage 1: Input] --> B[Stage 2: Process]
    B --> C{Decision}
    C -->|Yes| D[Stage 3a: Path A]
    C -->|No| E[Stage 3b: Path B]
    D --> F[Stage 4: Output]
    E --> F
\`\`\`
```

**4. Labeling Best Practices**:
- Use clear, concise labels
- Include data format in transitions (e.g., "JSON via stdin")
- Show error paths with different colors/styles
- Add notes for complex logic

### Data Flow Documentation Pattern

For each stage/component document:
1. **Purpose**: What this stage does
2. **Input Format**: JSON schema, examples
3. **Processing**: Key logic and algorithms
4. **Output Format**: JSON schema, examples
5. **Dependencies**: What it requires
6. **Performance**: Typical processing time
7. **Error Handling**: How failures are handled

## Documenting Algorithms

### Algorithm Documentation Template

```markdown
## Algorithm Name

### Overview
[1-2 paragraph high-level explanation]

### Architecture
[Describe phases, layers, or steps]

### Phase-by-Phase Breakdown

#### Phase 1: [Name]
**Purpose**: [What this phase does]
**Input**: [What it receives]
**Output**: [What it produces]
**Key Logic**: [Important details]

[Repeat for each phase]

### Implementation Examples
[4+ concrete examples showing edge cases]

### Performance Characteristics
| Metric | Value | Notes |
|--------|-------|-------|

### Accuracy Metrics
[If applicable: precision, recall, F1, etc.]

### Common Pitfalls
[✅/❌ patterns showing correct vs incorrect usage]
```

### Critical Pattern Documentation

Always document critical patterns with:
- **Why it matters** explanation
- **✅ CORRECT** code example
- **❌ WRONG** code example
- File location reference (file.py:line-range)

Example:
```markdown
### Pattern: Extract Before Normalize

**Why it matters**: Normalization removes formatting that contains semantic information. Extracting features first preserves original meaning.

\`\`\`python
# ✅ CORRECT: Extract features BEFORE normalization
features = extract_semantic_features(code)    # Phase 1
normalized = normalize_code(code)             # Phase 2
penalty = calculate_penalty(features)         # Phase 3

# ❌ WRONG: Normalizing first destroys semantic info
normalized = normalize_code(code)
features = extract_semantic_features(normalized)  # Too late!
\`\`\`

**Location**: `lib/algorithm.py:45-67`
```

## Creating Cheat Sheets

### Cheat Sheet Structure

A print-friendly one-page reference should include:

**1. Header**:
```markdown
# System Name - Quick Reference Cheat Sheet

**Version**: 1.0 | **Last Updated**: YYYY-MM-DD | **Print This Page**
```

**2. Visual Overview**:
- ASCII diagram of system architecture
- Component relationship diagram

**3. Critical Patterns** (⚠️ Section):
- Top 5-7 patterns that must be followed
- ✅/❌ code comparisons
- File location references

**4. Quick Reference Tables**:
- Commands and their usage
- Configuration options
- Data models (condensed)
- File locations

**5. Troubleshooting Quick Reference**:
| Issue | Cause | Solution |
|-------|-------|----------|

**6. Key Metrics** (if applicable):
| Metric | Value | Meaning |

### Table Best Practices

Use tables for:
- Penalty/multiplier systems
- Configuration options
- Component locations
- Command references
- Performance benchmarks
- Accuracy metrics

Keep tables concise (3-5 columns max for printability).

## Data Model Documentation

### Documenting JSON Schemas

For each data structure, provide:

**1. Schema Definition**:
```markdown
### DataStructureName
\`\`\`json
{
  "field_name": "type",        // Description
  "required_field": "string",  // What it contains
  "optional_field?": "number"  // When it's used
}
\`\`\`
```

**2. Field Descriptions Table**:
| Field | Type | Required | Description |
|-------|------|----------|-------------|

**3. Example**:
```json
{
  "field_name": "example_value",
  "required_field": "actual data",
  "optional_field": 42
}
```

### Documenting Pydantic Models

```markdown
### ModelName (Pydantic)

**Definition**:
\`\`\`python
class ModelName(BaseModel):
    field_name: str
    count: int = 0
    tags: List[str] = []
\`\`\`

**Fields**:
- `field_name` (str): Description
- `count` (int): Description (default: 0)
- `tags` (List[str]): Description (default: empty list)

**Example**:
\`\`\`python
model = ModelName(
    field_name="example",
    count=5,
    tags=["tag1", "tag2"]
)
\`\`\`
```

## Performance Documentation

### Benchmark Table Pattern

```markdown
### Performance Benchmarks

| Operation | Small (<100) | Medium (100-1k) | Large (1k+) |
|-----------|--------------|-----------------|-------------|
| Scan | 50ms | 500ms | 5s |
| Process | 100ms | 1s | 10s |
| Total | 150ms | 1.5s | 15s |

**Bottlenecks**:
1. [Component name] - [Why it's slow]
2. [Component name] - [Why it's slow]

**Optimization Strategies**:
- Strategy 1: [Description]
- Strategy 2: [Description]
```

## Troubleshooting Guide Pattern

Create troubleshooting sections with:

**1. Table Format** (for cheat sheets):
```markdown
| Issue | Cause | Solution |
|-------|-------|----------|
| Error message | Why it happens | How to fix |
```

**2. Detailed Format** (for full docs):
```markdown
### Issue: [Problem Description]

**Symptoms**:
- Observable behavior 1
- Observable behavior 2

**Root Cause**:
[Explanation of why this happens]

**Solution**:
1. Step 1
2. Step 2
3. Step 3

**Verification**:
[How to confirm it's fixed]

**Related**: See [Component Name] documentation
```

## Cross-Referencing Strategy

### Internal References

Link related sections within documentation:
```markdown
See [Component Interactions](#component-interactions) for details.
For algorithm specifics, see [similarity-algorithm.md](similarity-algorithm.md).
```

### File Location References

Always include file:line references:
```markdown
**Location**: `lib/extractor.py:45-67`
**See**: `config/settings.json:12-15`
```

### Navigation Aids

In README.md, provide clear navigation:
```markdown
## Documentation Structure

- **[README.md](README.md)** - Start here
- **[pipeline-data-flow.md](pipeline-data-flow.md)** - Pipeline details
- **[algorithm.md](algorithm.md)** - Algorithm deep dive
- **[CHEAT-SHEET.md](CHEAT-SHEET.md)** - Quick reference
```

## Code Example Guidelines

### Example Best Practices

1. **Show Complete Context**: Include imports, setup
2. **Add Comments**: Explain non-obvious parts
3. **Show Output**: Include expected results
4. **Use Real Data**: Avoid "foo", "bar" when possible
5. **Highlight Key Lines**: Use comments to draw attention

### Example Template

```markdown
### Example: [What This Demonstrates]

\`\`\`python
# Setup
from module import Class

# The key pattern being demonstrated
result = Class.method(
    param1="value",  # ← This parameter is critical
    param2=42
)

# Expected output
# {'status': 'success', 'count': 42}
\`\`\`

**Explanation**:
[What's happening and why it matters]
```

## Documentation Metadata

### Version Tracking

Add to every documentation file:
```markdown
**Version**: 1.0
**Last Updated**: 2025-11-17
**Author**: [Name or "Auto-generated"]
**Related**: [Links to related docs]
```

### Change Log (Optional)

For living documentation:
```markdown
## Change Log

### 2025-11-17 - v1.0
- Initial documentation creation
- Added data flow diagrams
- Created cheat sheet

### 2025-11-18 - v1.1
- Updated algorithm section
- Fixed typos in examples
```

## Quality Checklist

Before finalizing documentation, verify:

### Content Quality
- [ ] Overview explains what the system does
- [ ] All stages/components documented
- [ ] Data flow is clear with diagrams
- [ ] Critical patterns highlighted with ✅/❌
- [ ] Code examples are tested and accurate
- [ ] File references include line numbers
- [ ] Troubleshooting guide is comprehensive

### Structure Quality
- [ ] Clear hierarchy with headers
- [ ] Table of contents for files >100 lines
- [ ] Cross-references work correctly
- [ ] Navigation is intuitive
- [ ] Progressive disclosure used appropriately

### Technical Quality
- [ ] JSON schemas are valid
- [ ] Code examples are syntax-correct
- [ ] Mermaid diagrams render properly
- [ ] Performance numbers are realistic
- [ ] File paths are accurate

### Usability Quality
- [ ] New developers can find what they need
- [ ] Cheat sheet fits on one printed page
- [ ] Search-friendly (good keywords in headers)
- [ ] Examples cover common use cases
- [ ] Troubleshooting covers real issues

## File Organization

### Recommended Directory Structure

```
docs/
├── architecture/
│   ├── README.md              # Navigation hub
│   ├── {system}-overview.md   # High-level architecture
│   ├── {system}-data-flow.md  # Pipeline/data flow
│   ├── {algorithm}.md         # Algorithm details
│   ├── CHEAT-SHEET.md         # Quick reference
│   └── diagrams/              # Optional: separate diagrams
│       ├── overview.mmd
│       └── data-flow.mmd
├── api/                       # API documentation
├── guides/                    # How-to guides
└── reference/                 # Reference materials
```

## Progressive Disclosure Pattern

Follow Anthropic's progressive disclosure pattern:

**1. Start Simple** (README.md):
- What it does (2-3 sentences)
- Key concepts (bullet list)
- How to get started (3-5 steps)
- Links to detailed docs

**2. Add Detail** (Detailed docs):
- Complete component breakdown
- Full data flow diagrams
- All configuration options
- Performance characteristics

**3. Provide Reference** (Cheat sheet):
- Print-friendly one-pager
- Critical patterns only
- Quick lookup tables
- Troubleshooting guide

## Common Pitfalls to Avoid

### ❌ Anti-Patterns

1. **Too Much Detail Too Soon**: Don't put everything in README
2. **Missing Visuals**: Text-only documentation is hard to scan
3. **No Examples**: Abstract explanations without code
4. **Stale References**: File paths that don't exist
5. **No Troubleshooting**: Doesn't help with real problems
6. **Missing "Why"**: Only shows "what" and "how", not "why"
7. **No Cheat Sheet**: Developers have to search every time
8. **Inconsistent Structure**: Each doc uses different format

### ✅ Best Practices

1. **Visual First**: Start with a diagram
2. **Progressive Disclosure**: README → Detailed → Reference
3. **Show, Don't Tell**: Code examples for everything
4. **Stay Current**: Reference actual file:line locations
5. **Solve Real Problems**: Document actual troubleshooting
6. **Explain Rationale**: Always include "why it matters"
7. **One-Page Reference**: Create printable cheat sheet
8. **Consistent Templates**: Use same structure across docs

## Template: Complete Documentation Set

See [TEMPLATES.md](TEMPLATES.md) for ready-to-use templates for:
- README.md structure
- Detailed documentation structure
- Cheat sheet structure
- Algorithm documentation
- Data flow documentation
- Troubleshooting guide
- API reference

## Examples

### Example 1: Multi-Stage Pipeline

**Context**: 7-stage code consolidation pipeline bridging JavaScript and Python

**Documentation Created**:
1. **README.md** (448 lines) - Architecture overview, critical patterns, navigation
2. **pipeline-data-flow.md** (1,191 lines) - Complete stage-by-stage breakdown with 8 Mermaid diagrams
3. **similarity-algorithm.md** (857 lines) - Algorithm deep dive with examples
4. **CHEAT-SHEET.md** (250 lines) - One-page print reference

**Key Features**:
- Visual data flow for all 7 stages
- JSON schemas for inter-stage communication
- Critical pattern documentation (✅/❌ examples)
- Performance benchmarks and bottleneck analysis
- Troubleshooting guide for common issues

### Example 2: Algorithm Documentation

**Context**: Two-phase similarity algorithm with penalty system

**Documentation Approach**:
1. High-level overview (what it does)
2. Architecture diagram (3 phases)
3. Phase-by-phase breakdown
4. 4 complete examples (identical, HTTP mismatch, operator mismatch, multiple penalties)
5. Penalty multiplier table
6. Common pitfalls with ✅/❌
7. Performance and accuracy metrics

## Related Skills

- **session-report**: For documenting work sessions
- **backend-dev-guidelines**: For backend architecture patterns
- **frontend-dev-guidelines**: For frontend architecture patterns

## References

- Anthropic Best Practices: Progressive disclosure, 500-line rule
- Mermaid Documentation: https://mermaid.js.org/
- Markdown Guide: https://www.markdownguide.org/

---

**Skill Status**: COMPLETE ✅
**Version**: 1.0
**Last Updated**: 2025-11-17
