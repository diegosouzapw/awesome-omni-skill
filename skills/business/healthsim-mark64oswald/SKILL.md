---
name: healthsim
description: "HealthSim generates realistic synthetic healthcare data for testing EMR systems, claims processing, pharmacy benefits, and analytics. Use for ANY request involving: (1) synthetic patients, clinical data, or medical records, (2) healthcare claims, billing, or adjudication, (3) pharmacy prescriptions, formularies, or drug utilization, (4) HL7v2, FHIR, X12, or NCPDP formatted output, (5) healthcare testing scenarios or sample data generation."
---

# HealthSim - Synthetic Healthcare Data Generation

## Overview

HealthSim generates realistic synthetic healthcare data through natural conversation. Rather than writing code or configuration files, describe what you need and Claude generates appropriate data.

**Products:**

| Product | Domain | What It Generates | Status |
|---------|--------|-------------------|--------|
| **PatientSim** | Clinical/EMR | Patients, encounters, diagnoses, procedures, labs, vitals, medications | Active |
| **MemberSim** | Payer/Claims | Members, professional claims, facility claims, payments, accumulators | Active |
| **RxMemberSim** | Pharmacy/PBM | Prescriptions, pharmacy claims, formularies, DUR alerts, prior auths | Active |
| **TrialSim** | Clinical Trials | Studies, sites, subjects, visits, adverse events, efficacy, CDISC output | Active |
| **PopulationSim** | Demographics/SDOH | Population profiles, cohort specifications, health disparities, SVI/ADI analysis | Active |
| **NetworkSim** | Provider Networks | Providers, facilities, pharmacies, networks, benefit structures | Active |

## Quick Start

### Generate Clinical Data

**Request:** "Generate a 65-year-old diabetic patient with hypertension"

Claude will produce a patient with:
- Demographics (age 65, realistic name/address)
- Diagnoses (E11.9 Type 2 diabetes, I10 hypertension)
- Medications (metformin, lisinopril)
- Labs (A1C, BMP with values in expected ranges)
- Comorbidities (likely hyperlipidemia, possible obesity)

### Generate Claims

**Request:** "Create a professional claim for an office visit"

Claude will produce:
- Claim header (provider NPI, member ID, service date)
- Service lines (CPT 99213/99214, charges)
- Diagnoses (ICD-10 codes)
- Adjudication (allowed, paid, patient responsibility)

### Generate Pharmacy Data

**Request:** "Generate a pharmacy claim that triggers a drug interaction alert"

Claude will produce:
- Prescription details (NDC, quantity, days supply)
- Pharmacy claim (BIN, PCN, cardholder ID)
- DUR alert (DD code, clinical significance, recommendation)
- Claim response (approved with warning or rejected)

## Scenario Skills

### PatientSim Scenarios

Load these for clinical data generation:

| Scenario | Use When | Key Elements |
|----------|----------|--------------|
| **ADT Workflow** | admission, discharge, transfer, patient movement | A01/A02/A03 events, bed management, census |
| **Diabetes Management** | diabetic, A1C, glucose, metformin, insulin | Disease progression, medication escalation, complications |
| **Heart Failure** | CHF, HFrEF, BNP, ejection fraction | NYHA classification, GDMT therapy, decompensation |
| **Chronic Kidney Disease** | CKD, eGFR, dialysis, nephrology | CKD staging, progression, comorbidities |
| **Sepsis/Acute Care** | sepsis, infection, ICU, critical | Sepsis criteria, antibiotic protocols, ICU stay |
| **Orders & Results** | lab order, radiology, ORM, ORU, results | Orders, specimens, lab panels, radiology reports |
| **ED Chest Pain** | chest pain, emergency, ACS, troponin | Risk stratification, HEART score, workup |
| **Elective Joint** | hip replacement, knee replacement, arthroplasty | Pre-op, surgery, recovery, PT |
| **Maternal Health** | pregnancy, prenatal, L&D, postpartum | Prenatal visits, GDM, preeclampsia, delivery |
| **Oncology** | cancer, tumor, chemotherapy, breast/lung/colorectal | Staging, treatment protocols, tumor markers |

See: [skills/patientsim/](skills/patientsim/) for detailed skills

### MemberSim Scenarios

Load these for claims and payer data:

| Scenario | Use When | Key Elements |
|----------|----------|--------------|
| **Plan & Benefits** | plan, benefit plan, HMO, PPO, HDHP | Plan types, cost sharing, pharmacy tiers |
| **Enrollment & Eligibility** | enrollment, eligibility, 834, 270/271 | Member add/term, coverage verification, QLE |
| **Professional Claims** | office visit, 837P, physician claim | E&M coding, place of service, adjudication |
| **Facility Claims** | hospital, inpatient, 837I, DRG | Revenue codes, DRG assignment, LOS |
| **Prior Authorization** | prior auth, pre-cert, authorization | Request/response workflow, approval criteria |
| **Accumulator Tracking** | deductible, OOP, accumulator | Year-to-date tracking, family vs individual |
| **Value-Based Care** | quality measures, VBC, HEDIS | Attribution, measure compliance, incentives |
| **Behavioral Health** | mental health, psychiatry, SUD, therapy | Psychotherapy, medication management, PHP/IOP |

See: [skills/membersim/](skills/membersim/) for detailed skills

### RxMemberSim Scenarios

Load these for pharmacy and PBM data:

| Scenario | Use When | Key Elements |
|----------|----------|--------------|
| **Retail Pharmacy** | prescription fill, retail, copay | New/refill, pricing, patient pay |
| **Specialty Pharmacy** | specialty drug, biologics, hub | Limited distribution, PA, patient support |
| **DUR Alerts** | drug interaction, DUR, therapeutic dup | Alert types, severity, override |
| **Formulary Management** | formulary, tier, coverage | Tier structure, PA requirements, alternatives |
| **Rx Enrollment** | rx enrollment, pharmacy member, BIN PCN | Pharmacy benefit eligibility, plan assignment |
| **Rx Prior Auth** | pharmacy PA, step therapy | Clinical criteria, approval workflow |
| **Rx Accumulators** | rx deductible, rx OOP, Part D phases | Pharmacy cost sharing tracking, TrOOP |
| **Manufacturer Programs** | copay card, patient assistance | Copay cards, PAPs, hub services |

See: [skills/rxmembersim/](skills/rxmembersim/) for detailed skills

### TrialSim Scenarios

Load these for clinical trial data generation:

| Scenario | Use When | Key Elements |
|----------|----------|--------------|
| **Clinical Trials Domain** | trial concepts, phases, CDISC | Phase definitions, regulatory, standards |
| **Recruitment & Enrollment** | screening, enrollment, consent | Screening funnel, I/E criteria, randomization |
| **Phase 1 Dose Escalation** | Phase 1, FIH, MTD, 3+3, BOIN, CRM | Dose escalation, DLT, PK sampling |
| **Phase 2 Proof-of-Concept** | Phase 2, Simon's, MCP-Mod | Futility stopping, dose-response |
| **Phase 3 Pivotal** | Phase 3, pivotal, registration trial | Multi-site, endpoints, safety monitoring |
| **Oncology Trials** | oncology trial, tumor endpoints | RECIST, survival endpoints, biomarkers |
| **Cardiovascular Trials** | CV outcomes, MACE | Cardiac events, biomarkers |
| **CNS Trials** | CNS, Alzheimer's, MS | Cognitive scales, imaging |
| **Cell & Gene Therapy** | CAR-T, gene therapy, CGT | Long-term follow-up, CRS, ICANS |
| **Dimensional Analytics** | trial analytics, star schema, dashboard | fact/dim tables, DuckDB, Databricks |

See: [skills/trialsim/](skills/trialsim/) for detailed skills

### PopulationSim Scenarios

Load these for population intelligence and cohort definition:

| Scenario | Use When | Key Elements |
|----------|----------|--------------|
| **Geographic Profile** | county profile, demographics for, MSA | County/tract/metro demographics, health indicators |
| **Health Patterns** | diabetes rate, prevalence, disparities | CDC PLACES measures, age-adjusted rates |
| **SDOH Analysis** | SVI, ADI, social vulnerability, deprivation | SVI themes, ADI rankings, barriers |
| **Cohort Definition** | define cohort, population segment | CohortSpecification for generation products |
| **Trial Support** | diversity planning, site selection, feasibility | FDA diversity, site ranking, enrollment projections |

**Key Differentiator**: PopulationSim analyzes real population data (Census, CDC) and outputs specifications, not synthetic records. These specs drive realistic generation in PatientSim, MemberSim, and TrialSim.

See: [skills/populationsim/](skills/populationsim/) for detailed skills

### NetworkSim Scenarios

Load these for provider network knowledge and entity generation:

| Scenario | Use When | Key Elements |
|----------|----------|---------------|
| **Network Types** | HMO, PPO, EPO, POS, HDHP | Network definitions, cost/flexibility tradeoffs |
| **Plan Structures** | deductible, copay, coinsurance, OOP | Benefit design, cost sharing, accumulators |
| **Pharmacy Benefits** | tier structure, formulary, PBM | Tier design, formulary types, pharmacy networks |
| **PBM Operations** | BIN, PCN, claims processing, rebates | Claim flow, adjudication, manufacturer rebates |
| **Utilization Management** | prior auth, step therapy, QL | PA process, step requirements, quantity limits |
| **Specialty Pharmacy** | specialty drugs, hub model, REMS | Limited distribution, specialty services |
| **Network Adequacy** | access standards, time distance | Time/distance, provider ratios, ECPs |
| **Provider Generation** | generate provider, NPI, physician | Synthetic providers with taxonomy, credentials |
| **Facility Generation** | generate hospital, facility, CCN | Synthetic facilities with beds, services |
| **Pharmacy Generation** | generate pharmacy, NCPDP | Synthetic pharmacies with type, chain |

See: [skills/networksim/](skills/networksim/) for detailed skills

### Generative Framework

Load these for batch generation and specification-driven data creation:

| Scenario | Use When | Key Elements |
|----------|----------|--------------|
| **Profile Builder** | batch generation, cohort profile, population spec | Demographics, conditions, coverage definitions |
| **Journey Builder** | temporal patterns, event sequences, care pathways | Timelines, triggers, branching |
| **Quick Generate** | simple single-entity generation | Fast single patient/member/claim |
| **Distributions** | customize statistical patterns | Age, cost, utilization distributions |
| **Templates** | start from common patterns | Pre-built profiles and journeys |

**Key Differentiator**: The Generative Framework enables **specification-driven generation** at scale. Build a profile, define a journey, then execute to generate hundreds or thousands of correlated entities across all products.

See: [skills/generation/](skills/generation/) for detailed skills

### Common Skills

Cross-product infrastructure skills:

| Skill | Use When | Key Elements |
|-------|----------|--------------|
| **State Management** | save scenario, load scenario, list scenarios | DuckDB persistence, scenario naming |
| **Identity Correlation** | link patient, find member, cross-product | SSN correlation, entity linking |
| **DuckDB Skill** | query database, SQL, schema | Direct database operations |

See: [skills/common/](skills/common/) for detailed skills

## Output Formats

### Default: JSON

By default, Claude outputs data as JSON objects that match the canonical data model.

### Healthcare Standards

Request specific formats:

| Format | Request Phrases | Use Case |
|--------|-----------------|----------|
| **FHIR R4** | "as FHIR", "FHIR bundle", "FHIR resources" | Interoperability, modern APIs |
| **C-CDA** | "as C-CDA", "as CCD", "discharge summary", "referral note" | Clinical documents, HIE |
| **HL7v2** | "as HL7", "ADT message", "HL7v2" | Legacy EMR integration |
| **X12 834** | "as 834", "X12 enrollment", "enrollment file" | Benefit enrollment |
| **X12 270/271** | "as 270", "eligibility inquiry", "eligibility check" | Eligibility verification |
| **X12 837** | "as 837", "X12 claim", "EDI format" | Claims submission |
| **X12 835** | "as 835", "remittance", "ERA" | Payment posting |
| **NCPDP D.0** | "as NCPDP", "pharmacy claim format" | Pharmacy transactions |
| **CDISC SDTM** | "as SDTM", "SDTM domains" | Clinical trial regulatory submission |
| **CDISC ADaM** | "as ADaM", "analysis datasets" | Clinical trial statistical analysis |

See: [formats/](formats/) for transformation skills

### Analytics Formats

| Format | Request Phrases | Use Case |
|--------|-----------------|----------|
| **Dimensional (DuckDB)** | "star schema for DuckDB", "dimensional model", "for analytics" | Local BI development |
| **Dimensional (Databricks)** | "star schema for Databricks", "load to Databricks", "Unity Catalog" | Enterprise analytics |

See: [formats/dimensional-analytics.md](formats/dimensional-analytics.md) for star schema details

### Export Formats

| Format | Request Phrases |
|--------|-----------------|
| **CSV** | "as CSV", "save to CSV", "spreadsheet" |
| **Parquet** | "as Parquet", "for analytics" |
| **SQL INSERT** | "as SQL", "INSERT statements" |

## Generation Parameters

### Demographics

| Parameter | Default | Options |
|-----------|---------|---------|
| **age_range** | 18-90 | Any range, e.g., "pediatric (0-17)", "senior (65+)" |
| **gender** | weighted (49% M, 51% F) | "male", "female", specific distribution |
| **count** | 1 | Any number, batches for large counts |

### Clinical (PatientSim)

| Parameter | Options |
|-----------|---------|
| **conditions** | diabetes, heart failure, CKD, hypertension, COPD, etc. |
| **severity** | mild, moderate, severe, well-controlled, poorly-controlled |
| **complications** | with/without specific complications |

### Claims (MemberSim)

| Parameter | Options |
|-----------|---------|
| **claim_type** | professional, institutional, dental |
| **claim_status** | paid, denied, pending, partial |
| **network_status** | in-network, out-of-network |

### Pharmacy (RxMemberSim)

| Parameter | Options |
|-----------|---------|
| **fill_type** | new, refill |
| **drug_type** | generic, brand, specialty |
| **dur_alerts** | none, warning, reject |

## Reproducibility

For consistent results across sessions:

**Request:** "Generate 10 patients using seed 42"

Claude will:
1. Use seed 42 for all random selections
2. Generate identical output if same parameters used
3. Note the seed in output for reference

## Validation

Claude automatically validates generated data for:

- **Structural**: Required fields, data types, formats
- **Temporal**: Date ordering (discharge after admission, etc.)
- **Referential**: Foreign key relationships
- **Clinical**: Age-appropriate conditions, gender-appropriate conditions
- **Business**: Valid code combinations, realistic pricing

Request explicit validation: "Validate this patient data"

## Reference Data

For code lookups and documentation:

| Reference | Description |
|-----------|-------------|
| [Code Systems](references/code-systems.md) | ICD-10, CPT, HCPCS, LOINC, NDC, RxNorm |
| [Terminology](references/terminology.md) | Healthcare terminology and abbreviations |
| [Clinical Rules](references/clinical-rules.md) | Clinical business rules and guidelines |
| [Validation Rules](references/validation-rules.md) | All validation rules and constraints |
| [HL7v2 Segments](references/hl7v2-segments.md) | HL7v2 segment definitions (MSH, PID, OBR, OBX, etc.) |

## Format Transformations

Transform generated data to healthcare standards:

### Healthcare Standards

| Format | Skill | Use Case |
|--------|-------|----------|
| FHIR R4 | [formats/fhir-r4.md](formats/fhir-r4.md) | Modern interoperability, REST APIs |
| C-CDA | [formats/ccda-format.md](formats/ccda-format.md) | CCD, Discharge Summary, clinical documents |
| HL7v2 ADT | [formats/hl7v2-adt.md](formats/hl7v2-adt.md) | Admit/Discharge/Transfer messages |
| HL7v2 ORM | [formats/hl7v2-orm.md](formats/hl7v2-orm.md) | Order messages (lab, rad, meds) |
| HL7v2 ORU | [formats/hl7v2-oru.md](formats/hl7v2-oru.md) | Results/observation messages |
| X12 837 | [formats/x12-837.md](formats/x12-837.md) | Claims submission (P/I) |
| X12 835 | [formats/x12-835.md](formats/x12-835.md) | Remittance/ERA |
| NCPDP D.0 | [formats/ncpdp-d0.md](formats/ncpdp-d0.md) | Pharmacy transactions |

### Export Formats

| Format | Skill | Use Case |
|--------|-------|----------|
| CSV | [formats/csv.md](formats/csv.md) | Spreadsheets, data analysis, bulk export |
| SQL | [formats/sql.md](formats/sql.md) | Database INSERT statements, data loading |
| Dimensional | [formats/dimensional-analytics.md](formats/dimensional-analytics.md) | Star schema for DuckDB, Databricks, BI tools |

## Integration & Testing

| Guide | Description |
|-------|-------------|
| [Integration Guide](docs/integration-guide.md) | Cross-skill data flows and consistency |
| [Testing Patterns](docs/testing-patterns.md) | Validation and test patterns |

## Examples

### Example 1: Simple Patient

**Request:** "Generate a patient"

**Output:**
```json
{
  "mrn": "MRN00000001",
  "name": {
    "given_name": "John",
    "family_name": "Smith"
  },
  "birth_date": "1975-03-15",
  "gender": "M",
  "address": {
    "street_address": "123 Main Street",
    "city": "Springfield",
    "state": "IL",
    "postal_code": "62701"
  }
}
```

### Example 2: Clinical Scenario

**Request:** "Generate a 58-year-old female with poorly controlled Type 2 diabetes"

**Output:**
```json
{
  "patient": {
    "mrn": "MRN00000002",
    "name": { "given_name": "Maria", "family_name": "Garcia" },
    "birth_date": "1967-08-22",
    "gender": "F"
  },
  "diagnoses": [
    { "code": "E11.65", "description": "Type 2 diabetes with hyperglycemia" },
    { "code": "I10", "description": "Essential hypertension" },
    { "code": "E78.5", "description": "Hyperlipidemia" }
  ],
  "medications": [
    { "name": "Metformin", "dose": "1000 mg", "frequency": "BID" },
    { "name": "Glipizide", "dose": "10 mg", "frequency": "BID" },
    { "name": "Lisinopril", "dose": "20 mg", "frequency": "QD" }
  ],
  "labs": [
    { "test": "HbA1c", "value": "9.8", "unit": "%", "flag": "H" },
    { "test": "Glucose", "value": "245", "unit": "mg/dL", "flag": "H" },
    { "test": "Creatinine", "value": "1.2", "unit": "mg/dL" }
  ]
}
```

### Example 3: Professional Claim

**Request:** "Generate a paid professional claim for an office visit"

**Output:**
```json
{
  "claim": {
    "claim_id": "CLM20250115000001",
    "claim_type": "PROFESSIONAL",
    "member_id": "MEM001234",
    "provider_npi": "1234567890",
    "service_date": "2025-01-15",
    "place_of_service": "11",
    "principal_diagnosis": "E11.9",
    "claim_lines": [
      {
        "line_number": 1,
        "procedure_code": "99214",
        "charge_amount": 175.00,
        "units": 1
      }
    ]
  },
  "adjudication": {
    "status": "paid",
    "allowed_amount": 125.00,
    "paid_amount": 100.00,
    "deductible": 0.00,
    "copay": 25.00,
    "coinsurance": 0.00
  }
}
```

### Example 4: Pharmacy Claim with DUR

**Request:** "Generate a pharmacy claim that gets rejected for early refill"

**Output:**
```json
{
  "claim": {
    "claim_id": "RX20250115000001",
    "transaction_code": "B1",
    "ndc": "00071015523",
    "drug_name": "Atorvastatin 20mg",
    "quantity": 30,
    "days_supply": 30,
    "service_date": "2025-01-15"
  },
  "response": {
    "status": "rejected",
    "reject_code": "79",
    "reject_message": "Refill Too Soon",
    "dur_alert": {
      "type": "ER",
      "message": "Refill 12 days early (before 80% used)",
      "previous_fill_date": "2024-12-27",
      "days_early": 12
    }
  }
}
```

## Tips

1. **Be specific**: "diabetic patient with A1C of 9.5" beats "sick patient"
2. **Request format early**: "Generate as FHIR..." rather than converting after
3. **Use seeds**: For reproducible test data across sessions
4. **Batch large requests**: "Generate 100 in batches of 20"
5. **Validate sensitive data**: Request validation for production-like scenarios

## Disclaimer

HealthSim generates **synthetic test data only**. It is not a clinical decision support system and does not provide medical advice, diagnosis recommendations, or treatment guidance.

**Intended uses:**

- Software development and testing
- System integration validation
- Training and educational demonstrations
- Performance and load testing

**Not intended for:**

- Clinical decision support
- Medical advice or treatment recommendations
- Actual patient care
- Processing real PHI

The clinical patterns, medication regimens, and lab values reflect general healthcare conventions suitable for test data. They do not account for individual patient circumstances or the full complexity of clinical practice.
