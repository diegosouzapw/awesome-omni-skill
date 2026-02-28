---
name: advanced-oscal-validator
description: Perform comprehensive OSCAL validation using community-inspired patterns including JSON schema validation, business rule validation, cross-reference checking, and best practices from IBM Trestle, oscal-pydantic, and Lula. Use for thorough document quality assurance.
---

# Advanced OSCAL Validator Skill

Perform comprehensive OSCAL document validation using advanced patterns inspired by community tools including IBM Trestle, oscal-pydantic, and Defense Unicorns' Lula.

## When to Use This Skill

Use this skill when you need to:
- Perform thorough validation beyond basic structure
- Validate against NIST OSCAL JSON schemas
- Check business rules and best practices
- Validate cross-references and links
- Ensure FedRAMP-specific requirements are met

---

## ⛔ Authoritative Data Requirement

Validation checks **user-provided documents** against structural rules.

### What This Skill Does (Safe)
- Validates OSCAL structure and syntax
- Checks UUID formats and references
- Verifies required fields are present
- Confirms cross-references resolve
- Applies business rule logic to YOUR document

### What Requires Authoritative Sources
| Validation Type | Requires |
|-----------------|----------|
| Baseline completeness | The baseline profile being validated against |
| Control reference validation | The catalog that controls reference |
| FedRAMP-specific rules | FedRAMP baseline |

### For Baseline Validation
```
To validate SSP completeness against a baseline, I need both:
1. Your SSP document (provided)
2. The baseline profile it should meet (e.g., FedRAMP Moderate)

I cannot determine if controls are missing without the authoritative baseline.
```

---

## Validation Levels

| Level | Description | Checks |
|-------|-------------|--------|
| Schema | JSON schema compliance | Structure, types, required fields |
| Semantic | Business logic | UUIDs, references, dates |
| Quality | Best practices | Completeness, clarity |
| Framework | FedRAMP/NIST specific | Baseline compliance |

## Advanced Validation Categories

### Schema Validation
Validate against official NIST OSCAL JSON schemas:
- Catalog schema
- Profile schema
- SSP schema
- Component definition schema
- Assessment schemas

### UUID Validation
- Format: RFC 4122 compliant
- Uniqueness: No duplicates within document
- References: All UUID refs resolve

### Cross-Reference Validation
- Control references exist in imported catalogs
- Party references resolve within document
- Component references are valid
- Resource links are accessible

### Business Rule Validation

| Rule | Description |
|------|-------------|
| BIZ-001 | SSP must import a profile |
| BIZ-002 | All baseline controls must be addressed |
| BIZ-003 | Implementation status required for each control |
| BIZ-004 | Responsible parties must be defined |
| BIZ-005 | System characteristics must be complete |

### FedRAMP-Specific Validation
- All required control families present
- POA&M references valid
- Required attachments present
- Naming conventions followed

## Validation Report Structure

```
ADVANCED VALIDATION REPORT
==========================
Document: ssp.json
Type: System Security Plan
Schema Version: 1.2.0
Validation Date: 2024-01-15

SUMMARY
-------
Schema Valid: ✅ Yes
Semantically Valid: ⚠️ Warnings
Quality Score: 85/100

SCHEMA VALIDATION
-----------------
Status: PASS
- Structure: Valid
- Required Fields: All present
- Data Types: Correct

UUID VALIDATION
---------------
Total UUIDs: 245
Unique: 245 ✅
Invalid Format: 0 ✅
Orphaned References: 2 ⚠️
  - #uuid-abc123 not found
  - #uuid-def456 not found

CROSS-REFERENCE VALIDATION
--------------------------
Control References: 320/325 valid
  Missing: AC-1(1), CM-7(1), SI-4(2), ...
  
Party References: 12/12 valid ✅
Component References: 45/45 valid ✅

BUSINESS RULES
--------------
✅ BIZ-001: Profile imported
⚠️ BIZ-002: 5 controls not addressed
✅ BIZ-003: All have implementation status
✅ BIZ-004: Responsible parties defined
⚠️ BIZ-005: System boundary incomplete

QUALITY CHECKS
--------------
- Implementation narratives: 95% complete
- Evidence references: 80% complete
- Parameter values: 100% set
- Remarks clarity: Good

RECOMMENDATIONS
---------------
1. Add missing control implementations
2. Resolve orphaned UUID references
3. Complete system boundary description
```

## How to Perform Advanced Validation

### Step 1: Schema Validation
1. Identify document type from root element
2. Fetch appropriate NIST schema
3. Validate document against schema
4. Collect all schema violations

### Step 2: UUID Analysis
1. Extract all UUIDs from document
2. Validate format (8-4-4-4-12 hex)
3. Check for duplicates
4. Build reference graph
5. Find orphaned references

### Step 3: Cross-Reference Check
1. Extract all internal references (#uuid-...)
2. Extract all control-id references
3. Resolve each reference
4. Report unresolved references

### Step 4: Business Rule Evaluation
Apply business rules based on document type:

**For SSP:**
- Verify profile import exists
- Check all baseline controls addressed
- Validate implementation statements present
- Confirm responsible parties assigned

**For Component Definition:**
- Verify component has title
- Check control implementations reference valid controls
- Validate capability descriptions

### Step 5: Quality Assessment
Score based on:
- Completeness of narratives
- Presence of evidence references
- Parameter value coverage
- Clarity and specificity

## Validation Patterns from Community

### From IBM Trestle
- Workspace-based validation
- Model assembly validation
- Profile resolution checking

### From oscal-pydantic
- Type-safe validation
- Field-level constraints
- Nested object validation

### From Lula
- Control validation automation
- Policy-as-code patterns
- Continuous validation

## Common Validation Issues

| Issue | Severity | Fix |
|-------|----------|-----|
| Missing metadata.title | ERROR | Add title |
| Invalid UUID format | ERROR | Regenerate UUID |
| Orphaned reference | WARNING | Update or remove |
| Missing implementation | WARNING | Add narrative |
| Empty remarks | INFO | Add context |

## Example Usage

When asked "Thoroughly validate this SSP":

1. Parse the SSP document
2. Validate against OSCAL SSP schema
3. Check all UUIDs for format and uniqueness
4. Resolve all cross-references
5. Apply SSP business rules
6. Score quality metrics
7. Generate comprehensive validation report
8. Provide prioritized fix recommendations
