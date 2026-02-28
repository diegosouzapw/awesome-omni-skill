---
name: fair-data-model-assessment
description: "Assess data models against FAIR principles using RDA-FDMM indicators. Use when: (1) Evaluating vendor-delivered data models for FAIR compliance, (2) Reviewing schemas, ontologies, or data dictionaries before integration, (3) Creating FAIR assessment reports for data governance reviews, (4) Preparing data model documentation for enterprise or regulatory standards, (5) Auditing existing data assets for FAIRness gaps. Covers 41 RDA indicators across Findable, Accessible, Interoperable, Reusable dimensions with maturity scoring (0-4 scale)."
---

# FAIR Data Model Assessment

Assess data models, schemas, and data dictionaries against FAIR principles using the RDA FAIR Data Maturity Model framework.

## Quick Reference

| Task | Approach |
|------|----------|
| Full assessment | Follow Assessment Workflow below |
| Quick check | Use Essential Indicators only (see references/rda-indicators.md) |
| Generate report | Run `scripts/generate_report.py` after assessment |
| Calculate scores | Run `scripts/score_calculator.py` with assessment JSON |

## Context: Data Models vs Published Datasets

The RDA-FDMM was designed for published datasets with DOIs. Internal data models require adapted assessment:

| RDA Focus | Data Model Adaptation |
|-----------|----------------------|
| DOI/PID resolution | Schema defines unique entity identifiers |
| Registry indexing | Model documented in enterprise catalog |
| HTTP retrieval | Schema accessible via standard formats |
| License metadata | Usage rights documented |

See `references/assessment-questions.md` for the full adapted question set.

## Assessment Workflow

### Step 1: Gather Artifacts

Collect all available documentation:
- Schema files (JSON Schema, XSD, DDL, etc.)
- Data dictionaries or field definitions
- Entity-relationship diagrams
- Metadata specifications
- Provenance documentation
- Usage/license documentation

### Step 2: Determine Scope

Not all 41 indicators apply to pre-publication data models. Classify the assessment:

**Pre-publication internal model**: Focus on indicators F2, F3, I1, I2, I3, R1, R1.2, R1.3
**Model with planned publication**: Include F1, F4, A1, A2, R1.1
**Published/registered model**: Full indicator set applies

### Step 3: Conduct Assessment

For each applicable indicator in `references/rda-indicators.md`:

1. Read the indicator definition and priority level
2. Answer the assessment questions in `references/assessment-questions.md`
3. Assign maturity level (0-4):
   - 0: Not applicable or not addressed
   - 1: Initial/ad-hoc implementation
   - 2: Basic/partial implementation
   - 3: Defined/consistent implementation
   - 4: Managed/optimized implementation

Record responses in this structure:
```json
{
  "model_name": "Vendor Data Model X",
  "assessment_date": "2025-01-06",
  "assessor": "Name",
  "scope": "pre-publication",
  "indicators": {
    "F1": { "maturity": 2, "notes": "UUIDs defined but not globally resolvable" },
    "F2": { "maturity": 3, "notes": "Rich metadata in data dictionary" }
  }
}
```

### Step 4: Calculate Scores

Run the score calculator:
```bash
python scripts/score_calculator.py assessment.json
```

This produces:
- Per-principle scores (F, A, I, R)
- Overall FAIRness percentage
- Priority-weighted score (Essential > Important > Useful)

### Step 5: Generate Report

```bash
python scripts/generate_report.py assessment.json --output report.md
```

The report includes:
- Executive summary with overall score
- Per-principle breakdown with findings
- Gap analysis highlighting low-maturity indicators
- Specific recommendations for improvement

## Interpreting Results

| Score Range | Interpretation |
|-------------|----------------|
| 80-100% | Excellent FAIR compliance |
| 60-79% | Good compliance, minor gaps |
| 40-59% | Moderate compliance, improvement needed |
| 20-39% | Significant gaps, prioritize remediation |
| 0-19% | Major FAIR deficiencies |

## Domain-Specific Standards

For life sciences data models, see `references/domain-standards.md` for:
- CDISC standards (CDASH, SDTM, ADaM)
- HL7 FHIR resources
- ISA framework
- Allotrope Foundation schemas

## Common Findings and Remediation

**Low F scores**: Add persistent identifiers, improve metadata richness, register in catalog
**Low A scores**: Document access protocols, ensure format longevity
**Low I scores**: Map to standard vocabularies, use formal schemas, add qualified references
**Low R scores**: Add license info, document provenance, align with community standards

---

## Interactive Assessment Mode

For guided assessments, Claude can interactively walk through each indicator, ask questions, and build the assessment JSON.

### Starting an Interactive Assessment

Say: **"Start an interactive FAIR assessment for [model name]"**

Claude will guide you through:
1. Collecting basic information (model name, assessor, scope)
2. Walking through applicable indicators based on scope
3. For each indicator:
   - Explaining the indicator purpose
   - Asking relevant assessment questions
   - Suggesting maturity level based on responses
   - Capturing notes/evidence
4. Generating complete assessment JSON
5. Calculating and displaying scores

### Scope-Based Indicator Sets

Not all 41 indicators apply to every context. Choose your scope:

**Pre-publication internal model** (8 indicator groups):
- Focus on: F2, F3, I1, I2, I3, R1, R1.2, R1.3
- Skip: External identifiers, catalog registration, access protocols

**Planned-publication model** (14 indicator groups):
- Add to above: F1, F4, A1, A1.1, A2, R1.1

**Published/registered model** (16 indicator groups, 41 total indicators):
- Full indicator set applies

### Interactive Question Flow

For each indicator group, Claude will:

1. **Explain** the indicator purpose and relevance
2. **Ask** the assessment questions from `references/assessment-questions.md`
3. **Suggest** a maturity level based on your answers:
   - All yes → Maturity 3-4
   - Some yes → Maturity 2
   - Awareness only → Maturity 1
   - None → Maturity 0
4. **Confirm** the score with you (you can override)
5. **Capture** notes and evidence

**Example for F2 (Rich Metadata):**

```
━━━ Assessing F2: Rich Metadata for Discovery ━━━

This indicator checks whether your model has sufficient
descriptive information for discovery by humans and machines.

Questions:
1. Does a data dictionary exist with field-level documentation? [y/n]
2. Are data types specified for all fields? [y/n]
3. Are constraints documented (nullable, length, format)? [y/n]
4. Are business definitions provided (not just technical names)? [y/n]
5. Are valid value sets/enumerations documented? [y/n]
6. Is the purpose/context of the model documented? [y/n]
7. Are relationships between entities documented? [y/n]

Based on your answers (5/7 yes), I suggest maturity level 3:
"Comprehensive data dictionary with business definitions"

Accept maturity 3? [Enter to accept, or type 0-4 to override]
Notes for this indicator: ___
```

### Assessment Commands

During interactive mode, you can use these commands:

| Command | Action |
|---------|--------|
| `skip` | Skip current indicator |
| `back` | Return to previous indicator |
| `summary` | Show current assessment progress |
| `save` | Save current progress to JSON file |
| `calculate` | Calculate scores with current responses |
| `help` | Show available commands |
| `quit` | Exit assessment (progress is lost unless saved) |

### Quick Assessment Mode

For rapid initial screening, ask for: **"Quick FAIR assessment for [model name]"**

This uses the 10-question quick checklist:

1. Does every entity/table have a unique identifier? (F1)
2. Is there a complete data dictionary? (F2)
3. Is the model registered in a searchable catalog? (F4)
4. Can the schema be accessed without proprietary tools? (A1)
5. Is the schema in a formal, machine-readable format? (I1)
6. Are fields mapped to standard vocabularies? (I2)
7. Are relationships explicitly documented? (I3)
8. Are all fields fully documented with business definitions? (R1)
9. Are usage rights/license documented? (R1.1)
10. Does the model align with domain community standards? (R1.3)

**Quick scoring**: 8-10 = Strong, 5-7 = Moderate, 0-4 = Significant gaps

### Validation

The skill includes JSON validation for assessment files:

```bash
# Validate an assessment file
python scripts/score_calculator.py assessment.json --validate-only

# Calculate scores with validation
python scripts/score_calculator.py assessment.json

# Skip validation (not recommended)
python scripts/score_calculator.py assessment.json --no-validate

# Treat warnings as errors
python scripts/score_calculator.py assessment.json --strict
```

### Calibration

For consistent scoring across assessors, see `references/calibration-guide.md`:

- Maturity level definitions with evidence requirements
- Scoring examples for each indicator
- Inter-rater reliability process
- Reference assessments for training
