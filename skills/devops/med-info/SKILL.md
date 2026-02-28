---
name: med-info
description: Label-backed medication answers with citations and traceable IDs. RxCUI/NDC/set_id, key label sections, optional recalls/shortages/FAERS/interactions.
metadata: {"clawdbot": {"emoji": "💊", "os": ["darwin", "linux"], "requires": {"bins": ["python3"]}}}
---

# med-info

**Medication answers you can cite.**

`med-info` turns a drug name (or RxCUI, NDC, SPL set_id) into a **label-backed** summary with **traceable identifiers** and **source links**.

Use it when you want “show your work” medication information for notes, training, QA, internal docs, or agent workflows.

Not medical advice.

## What you get

- **Authoritative sources first**: FDA labeling via openFDA + DailyMed, identifiers via RxNorm/RxClass.
- **Citations + traceability**: RxCUI, NDC (product/package), SPL set_id, effective dates, and URLs.
- **The sections you actually need**: boxed warning, indications, dosing, contraindications, warnings, interactions, adverse reactions.
- **Optional safety context** (opt-in flags): recalls, shortages, FAERS aggregates, interactions, drug class, hazardous drug flag, REMS linkouts, Orange Book, Purple Book.
- **Automation friendly**: `--json` output for pipelines.

## Privacy

Do not include PHI. Query by drug name or identifiers only.

## Quickstart

```bash
cd {baseDir}
python3 scripts/med_info.py "Eliquis" --brief
```

Common workflows:

```bash
# Only the sections you care about
python3 scripts/med_info.py "Eliquis" --sections contraindications,drug_interactions --brief

# Find keyword hits in label text (fast way to answer "does the label mention X?")
python3 scripts/med_info.py "Eliquis" --find ritonavir --find CYP3A4 --find P-gp --find-max 8

# Deterministic lookups by identifier (best for reproducibility)
python3 scripts/med_info.py "70518-4370-0"   # NDC (package)
python3 scripts/med_info.py "70518-4370"     # NDC (product)
python3 scripts/med_info.py "05999192-ebc6-4198-bd1e-f46abbfb4f8a"  # SPL set_id
```

## Disambiguation (when there are multiple labels)

```bash
python3 scripts/med_info.py "metformin" --candidates
python3 scripts/med_info.py "metformin" --candidates --pick 2 --brief
python3 scripts/med_info.py "metformin" --set-id "05999192-ebc6-4198-bd1e-f46abbfb4f8a"
```

## Optional add-ons

```bash
# Pharmacist-friendly output bundle
python3 scripts/med_info.py "Eliquis" --pharmacist --brief

# Safety signals and operational context (opt-in)
python3 scripts/med_info.py "metformin" --recalls --brief
python3 scripts/med_info.py "amphetamine" --shortages --brief
python3 scripts/med_info.py "Eliquis" --faers --faers-max 10
python3 scripts/med_info.py "Eliquis" --interactions --interactions-max 20
python3 scripts/med_info.py "Eliquis" --rxclass
python3 scripts/med_info.py "cyclophosphamide" --hazardous
python3 scripts/med_info.py "isotretinoin" --rems

# Reference datasets
python3 scripts/med_info.py "adalimumab" --purplebook
python3 scripts/med_info.py "metformin" --orangebook

# Chemistry (best-effort)
python3 scripts/med_info.py "ibuprofen" --chem
```

## Output shaping

```bash
python3 scripts/med_info.py "ibuprofen" --json
python3 scripts/med_info.py "Eliquis" --brief --sections all
python3 scripts/med_info.py "Eliquis" --print-url --brief   # prints queried URLs (api_key redacted)
```

## Sources (high level)

- openFDA: drug labels, NDC directory, recalls/enforcement, shortages, FAERS
- RxNorm / RxClass (RxNav): normalization and drug classes
- DailyMed: SPL label history and media
- MedlinePlus Connect: patient-friendly summaries (links)
- Orange Book and Purple Book: best-effort context

## Safety notes

- For clinical decisions, verify against the **full official label**.
- Input is treated as untrusted, openFDA `search` strings are escaped to prevent query injection.

## Keys and rate limits

Works without any keys. Optionally:
- `OPENFDA_API_KEY`: increases openFDA rate limits for heavy usage.
