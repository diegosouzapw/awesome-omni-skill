---
name: swedish-medications
description: Look up Swedish medication information from FASS (Farmaceutiska Specialiteter i Sverige). Use when users ask about medications, drugs, läkemedel, dosages, side effects (biverkningar), interactions, or need to understand prescriptions in Sweden. Covers all medications approved for use in Sweden.
---

# Swedish Medications Skill

Look up information about medications approved in Sweden using FASS (the official Swedish pharmaceutical database).

## Quick Start

CLI lookup:
```bash
fass-lookup paracetamol
fass-lookup Alvedon
fass-lookup "alvedon 500mg"
```

Node.js usage:
```javascript
const { lookupMedication, findMedication, searchMedications, getDatabaseStats } = require('swedish-medications');

// Formatted markdown output
console.log(lookupMedication('Alvedon'));

// Raw data
const med = findMedication('ibuprofen');
console.log(med.dose);

// Multi-result search
console.log(searchMedications('insulin', 5));

// Database stats
console.log(getDatabaseStats());
```

Direct script (dev/test):
```bash
node scripts/fass_lookup.js "paracetamol"
```

## Capabilities

- **Search medications** by name (brand or generic/substance)
- **Multi-result search** for category queries ("show me insulin medications")
- **Get detailed info**: dosage, side effects, interactions, contraindications
- **Swedish health context**: ATC codes, prescription status, regional recommendations
- **FASS links** for official information

## Usage Patterns

### Basic Lookup
When a user asks "What is Alvedon?" or "Tell me about paracetamol":
1. Run the lookup script with the medication name
2. Present key info: what it's for, dosage, common side effects
3. Include the FASS link for official information

### Interaction Check
When a user asks "Can I take X with Y?":
1. Look up both medications
2. Check the interactions section
3. Recommend consulting healthcare provider for complex cases

### Dosage Questions
When a user asks about dosing:
1. Look up the medication
2. Present standard adult dosage
3. Note: Always recommend following prescribed dosage or consulting pharmacist

### Category Search
When a user asks "What ADHD medications are available?" or "Search insulin medications":
1. Use multi-result search
2. Return a short list of matches
3. Offer to expand any item

## API Reference

### `lookupMedication(query: string): string`
Returns formatted markdown with medication info and FASS link.

### `findMedication(query: string): object | null`
Returns raw medication data object. Checks curated list first, then full database.

### `searchMedications(query: string, limit?: number): array`
Returns multiple matching medications (new in v2.0).

### `getDatabaseStats(): object`
Returns `{ curated: 23, full: 9064, substances: 1353 }`.

### `getFassUrl(query: string): string`
Returns the FASS.se search URL for a query.

## Important Notes

- This skill provides **information only**, not medical advice
- Always recommend consulting healthcare professionals for medical decisions
- Swedish medications may have different names than international equivalents
- Some medications require prescription (receptbelagt) in Sweden
 - Check FASS.se for the most complete official details

## Data Sources

- **FASS.se** - Official Swedish pharmaceutical information
- **Läkemedelsverket** - Swedish Medical Products Agency
- **1177.se** - Swedish healthcare guide

## References

See `references/common-medications.md` for quick reference on frequently asked medications.
See `references/atc-codes.md` for understanding medication classification.
