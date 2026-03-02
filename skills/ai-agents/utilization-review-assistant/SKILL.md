---
name: utilization-review-assistant
description: Support utilization review decisions by evaluating medical necessity, level-of-care appropriateness, and length-of-stay justification against InterQual and Milliman criteria. Use when performing concurrent or retrospective utilization reviews, preparing peer-to-peer appeals, or assessing admission and continued-stay criteria.

metadata:
  display_name: "Utilization Review Assistant"
  short_description: "Support medical necessity and level-of-care UR decisions"
  default_prompt: "Check my medical necessity and level of care for gaps risks and required fixes"
  version: "1.0.0"
  tags:
    - healthcare
  icon_path: "assets/icon.png"
---

# Utilization Review Assistant

## Overview

Assist utilization review (UR) nurses and physicians in evaluating the medical necessity and appropriateness of healthcare services. This skill applies standardized criteria (InterQual, Milliman Care Guidelines, CMS guidelines) to patient clinical data to assess admission appropriateness, level-of-care determinations, continued-stay justification, and discharge readiness — supporting both concurrent and retrospective review workflows.

## When to Use

- Evaluating whether an admission meets inpatient vs. observation criteria
- Justifying continued inpatient stay for payer review
- Preparing clinical summaries for peer-to-peer discussions
- Performing retrospective utilization review for denials management
- Assessing level-of-care appropriateness (ICU, step-down, med-surg, SNF)
- Supporting discharge planning with readiness assessment

## Required Inputs

| Input | Description | Format |
|-------|-------------|--------|
| Patient clinical data | Diagnoses, vitals, labs, treatments, clinical status | Structured object |
| Admission details | Date, source, admitting diagnosis, attending | Structured object |
| Current treatment plan | Active orders, medications, therapies | Structured list |
| Review type | Concurrent, retrospective, pre-service | Enum string |
| Payer information | Insurance type, plan specifics | Structured object |

## Methodology

### Step 1: Clinical Data Synthesis

Compile the clinical picture required for criteria application:

1. **Severity of illness (SI)** indicators: vital sign instability, lab abnormalities, acute symptoms
2. **Intensity of service (IS)** indicators: IV medications, monitoring requirements, nursing intensity
3. **Current clinical trajectory**: improving, stable, worsening
4. **Comorbidity burden**: conditions affecting recovery and resource needs

### Step 2: Criteria Application

Apply standardized utilization criteria systematically:

**InterQual Framework:**

Admission Criteria:
- Severity of Illness (must meet threshold)
- Intensity of Service (must match SI)

Continued Stay Criteria:
- Ongoing acute care needs
- Active treatment requiring inpatient level
- Clinical instability preventing safe discharge

Discharge Criteria:
- Clinical stability parameters met
- Safe discharge plan in place
- Follow-up arranged

**CMS Two-Midnight Rule:**
- Inpatient admission appropriate when physician expects patient to require hospital care spanning at least two midnights
- If less than 2 midnights expected, observation status typically appropriate
- Exceptions: certain procedures on the CMS Inpatient-Only List

### Step 3: Level-of-Care Assessment

Determine the appropriate care level:

| Level | Criteria Indicators |
|-------|-------------------|
| ICU | Mechanical ventilation, vasopressors, continuous monitoring, hemodynamic instability |
| Step-down/Telemetry | Cardiac monitoring, frequent assessments, IV drips (non-vasopressor) |
| Med-Surg Inpatient | IV medications, skilled nursing, diagnostic workup requiring observation |
| Observation | Expected less than 2 midnights, diagnostic uncertainty, short-course treatment |
| SNF/Rehab | Medically stable, requires daily skilled services, 3-midnight qualifying stay |
| Home with services | Stable, homebound, needs skilled nursing/therapy visits |

### Step 4: Justification Documentation

Build the clinical justification narrative:

1. **Medical necessity statement**: Why this level of care is required
2. **Alternative consideration**: Why lower-level care is insufficient
3. **Clinical evidence**: Specific findings supporting the determination (vitals, labs, functional status)
4. **Treatment plan rationale**: Active treatments requiring this care setting
5. **Discharge barriers**: What must change before step-down or discharge is safe

### Step 5: Determination and Recommendation

Produce the UR determination:

Determination Options:
- APPROVED: Meets criteria for requested level of care
- APPROVED WITH MODIFICATION: Meets criteria at a different level
- PENDING CLINICAL: Additional information needed for determination
- REFERRED TO PHYSICIAN REVIEWER: Does not clearly meet criteria, requires MD review
- NOT MEETING CRITERIA: Does not meet medical necessity for this level

## Output Specification

The output includes:

**review_metadata**: review_type, review_date, reviewer, payer

**patient_summary**: admitting_diagnosis (description, icd10), admission_date, current_los, current_level

**clinical_indicators**: severity_of_illness indicators with values and threshold status, intensity_of_service items with frequency and inpatient requirement, clinical_trajectory (improving/stable/worsening)

**criteria_evaluation**: criteria_set used (InterQual/Milliman/CMS), admission_criteria_met, continued_stay_criteria_met, discharge_criteria_met, level_of_care_appropriate, recommended_level, supporting_evidence

**determination**: decision, rationale narrative, next_review_date, estimated_discharge, discharge_barriers with resolution plans

**peer_to_peer_prep**: key_talking_points, anticipated_payer_concerns, supporting_clinical_evidence

## Analysis Framework

### Medical Necessity Evaluation Matrix

| Factor | Meets Inpatient | Observation | Outpatient |
|--------|----------------|-------------|------------|
| Vital instability | Persistent abnormality | Transient, improving | Stable |
| IV medication need | Continuous or frequent | Short-course (under 24h) | Oral equivalent available |
| Monitoring | Continuous/q1-2h | q4-6h | Self-monitoring |
| Procedure recovery | High-risk, anesthesia | Minor, moderate sedation | Office-based |
| Fall/safety risk | High, requires supervision | Moderate | Low |
| Functional status | Cannot perform ADLs | Limited ADLs | Independent |

### Length-of-Stay Benchmarking

Compare actual LOS against geometric mean LOS (GMLOS) for the MS-DRG:
- **Below GMLOS**: Efficient, ensure not premature discharge
- **At GMLOS**: Expected, document ongoing medical necessity
- **Above GMLOS**: Requires strong justification; identify and document barriers

## Examples

**Input**: 72-year-old admitted for community-acquired pneumonia, day 3. On IV ceftriaxone/azithromycin. O2 4L NC, SpO2 94%. Temp 99.1F (down from 102F on admission). WBC 11.2 (down from 18.5).

**UR Assessment**:
- SI: Meets — supplemental O2 requirement, resolving leukocytosis
- IS: Meets — IV antibiotics, supplemental oxygen
- Trajectory: Improving (temp, WBC trending down)
- Continued stay: APPROVED — still requiring supplemental O2 and IV antibiotics; transition to oral not yet safe (O2 dependent)
- Estimated step-down: Day 4-5 when O2 weaned and tolerating oral antibiotics
- Next review: Day 4

## Guidelines

1. **Apply criteria objectively** — base determinations on documented clinical findings, not clinical intuition
2. **Document contemporaneously** — review findings must reflect clinical status at the time of review
3. **Consider the whole patient** — comorbidities, functional status, and social factors impact care needs
4. **Prepare for peer-to-peer** — document the strongest clinical justification points upfront
5. **Track avoidable days** — identify days where care could have been provided at a lower level

## Validation Checklist

- [ ] Clinical data is current (within 24 hours for concurrent review)
- [ ] SI and IS indicators are specific and measurable (not vague)
- [ ] Criteria set used is specified and appropriate for the payer
- [ ] Level-of-care recommendation is justified with specific clinical rationale
- [ ] Discharge barriers are identified with resolution plans and timelines
- [ ] Peer-to-peer talking points address likely payer objections
- [ ] Review complies with UMCRA (Utilization Management Certification and Review Act) if applicable

## HIPAA Compliance Notes

- UR data shared with payers must comply with minimum necessary and TPO (Treatment, Payment, Operations) permitted uses
- Peer-to-peer discussions should be documented but limited to relevant clinical information
- UR determinations stored in the medical record are subject to patient access rights
- Third-party UR vendors must operate under a BAA
- Restrict UR system access to authorized UR staff, physicians, and care management personnel
