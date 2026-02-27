---
name: status-determination
description: Evaluates hospital admission packets to determine if inpatient or observation status is clinically supported. Uses a two-step workflow with clinical review checkpoint. Analyzes ADT, ORU (labs/vitals), and MDM clinical notes against Medicare two-midnight rule and severity criteria. Use when reviewing admission status decisions, inpatient vs observation classification, utilization review, or two-midnight compliance audits.
---

# Status Determination Skill

Screens inpatient/observation status decisions *after* the hospital has coded the patient but *before* claim submission—catching likely denials based on ICD-10 validation, documentation gaps, medical necessity alignment, and Two-Midnight Rule compliance.

**Target Users:** Hospital case managers, utilization review nurses, admitting physicians

## Architecture

This skill uses a **2-step workflow** with a clinical review checkpoint:

```
Step 1: Intake & Assessment
  ↓ (parse data, apply rubric, generate preliminary verdict)
  ↓ writes: waypoints/assessment.json

[User Checkpoint: Review & Confirm]

Step 2: Decision & Letter
  ↓ (capture reviewer input, generate report)
  ↓ writes: waypoints/decision.json
  ↓ writes: outputs/status-validation-report.md
```

## Quick Start

1. Provide paths to the admission packet (ADT, labs, vitals, MDM note)
2. Run Step 1: Intake & Assessment
3. Review the checkpoint summary
4. Confirm agreement or provide override reasoning
5. Receive the Status Validation Report

---

## Execution Flow

### Startup: Check for Existing Assessment

Before starting, check if `waypoints/assessment.json` exists:

```
IF waypoints/assessment.json exists AND status = "assessment_complete":
  → Ask: "Found existing assessment for [MRN]. Resume from Step 2? (Y/N)"
  → If Y: Skip to Step 2
  → If N: Start fresh from Step 1

ELSE:
  → Start from Step 1
```

---

### Step 1: Intake & Assessment

**Execute:** Read and follow `references/01-intake-assessment.md`

**Input:** 4 data files (ADT, ORU labs, ORU vitals, MDM note)

**Process:**
1. Parse admission data from all sources
2. Validate ICD-10 codes using MCP tools
3. Assess Severity of Illness against rubric thresholds
4. Assess Intensity of Services requirements
5. Check Two-Midnight Rule compliance (Medicare)
6. Identify documentation gaps
7. Generate preliminary verdict (SUPPORTED / AT RISK / LIKELY DENIAL)

**Output:** `waypoints/assessment.json`

**After Step 1:** Display summary and ask:
"Ready to proceed to Step 2: Clinical Review & Letter Generation? (Y/N)"

---

### Step 2: Decision & Letter

**Execute:** Read and follow `references/02-decision-letter.md`

**Input:** `waypoints/assessment.json`

**Process:**
1. Display clinical review checkpoint with assessment summary
2. Ask reviewer: "Do you agree with this assessment? (Agree/Disagree)"
3. If Disagree: Capture reasoning, flag for escalation
4. Generate Status Validation Report using `assets/letter-template.md`

**Output:**
- `waypoints/decision.json`
- `outputs/status-validation-report.md`

---

## Verdict Definitions

| Verdict | Meaning | Denial Risk | Action |
|---------|---------|-------------|--------|
| SUPPORTED | Evidence justifies status decision | LOW | Proceed to submission |
| AT RISK | Borderline — documentation gaps exist | MEDIUM | Strengthen documentation |
| LIKELY DENIAL | Evidence does not support status | HIGH | Change status or escalate |

---

## Reference Files

| File | Purpose |
|------|---------|
| [rubric.md](rubric.md) | Clinical criteria thresholds for labs, vitals, MDM |
| [tools.md](tools.md) | MCP tool integration patterns (ICD-10, CMS Coverage) |
| [references/01-intake-assessment.md](references/01-intake-assessment.md) | Step 1 detailed workflow |
| [references/02-decision-letter.md](references/02-decision-letter.md) | Step 2 detailed workflow |
| [assets/letter-template.md](assets/letter-template.md) | Report template with placeholders |

---

## Tool Dependencies

This skill integrates with:
- `ICD10_Codes` - Diagnosis code validation, hierarchy, specificity checks
- `CMS_Coverage` - Medicare coverage policy lookup (optional)

---

## Directory Structure

```
daisy-skill-1/
├── SKILL.md                     ← You are here (orchestrator)
├── rubric.md                    ← Clinical criteria thresholds
├── tools.md                     ← MCP tool integration
├── references/
│   ├── 01-intake-assessment.md  ← Step 1 logic
│   └── 02-decision-letter.md    ← Step 2 logic
├── waypoints/
│   ├── assessment.json          ← Step 1 output (resume state)
│   └── decision.json            ← Step 2 output
├── outputs/
│   └── status-validation-report.md  ← Final report
├── assets/
│   └── letter-template.md       ← Report template
├── sample-data/
│   ├── adt-a01-admission.json
│   ├── oru-r01-labs.json
│   ├── oru-vitals.json
│   └── mdm-t02-ed-note.json
└── examples/
    └── case-001-sepsis-cap.md   ← Reference output
```

---

## Sample Usage

```
User: Run status determination on the sample data in sample-data/

Claude:
1. Loads sample-data/adt-a01-admission.json, oru-r01-labs.json,
   oru-vitals.json, mdm-t02-ed-note.json
2. Executes Step 1 (01-intake-assessment.md)
3. Writes waypoints/assessment.json
4. Displays checkpoint summary
5. User confirms agreement
6. Executes Step 2 (02-decision-letter.md)
7. Writes outputs/status-validation-report.md
8. Displays completion with verdict and action items
```
