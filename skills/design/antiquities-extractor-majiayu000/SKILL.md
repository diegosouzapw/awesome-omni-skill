---
name: antiquities-extractor
description: Extract and structure data from documents about the illegal antiquities trade, including dealers, collectors, artifacts, locations, and relationships. Use when processing news reports, academic articles, legal documents, encyclopedia entries, or research materials pertaining to looted artifacts, antiquities trafficking, provenance research, or cultural heritage crimes. Returns structured JSON with entities (persons, organizations, artifacts, locations) and their relationships.
---

# Antiquities Trade Data Extractor

Extract structured intelligence from documents about the illegal antiquities trade.

## Overview

This skill helps extract and structure information from any document relating to the illicit antiquities trade—including dealers, collectors, looters, auction houses, museums, artifacts, and the networks connecting them. The output is machine-readable JSON suitable for database ingestion, network analysis, and provenance research.

## Core Entity Types

Extract four main entity types, each with a specific canonical ID format:

**PERSON**: Individuals involved in the trade
- Canonical ID: `firstname_lastname` (lowercase, underscores)
- Attributes: role, nationality, birth/death dates, known aliases, specialization
- Roles include: dealer, collector, looter, archaeologist, official, restorer, consultant, auction_house_official
- Specialization: Geographic, temporal, or typological focus (e.g., "South-East Asian antiquities", "Greek pottery", "Egyptian Middle Kingdom artifacts")

**ORGANIZATION**: Institutions and businesses
- Canonical ID: `abbreviated_name` (e.g., `j_paul_getty_museum`, `christie_s`)
- Add `entity_type`: museum, gallery, auction_house, law_enforcement, university, customs, government, restoration_lab
- Attributes: location, founded_date, known_involvement (if any), collection_focus
- Collection_focus: Primary areas of collecting or dealing (e.g., "Pre-Columbian art", "Asian antiquities", "Classical Mediterranean")

**ARTIFACT**: Cultural objects
- Canonical ID: `descriptive_identifier` (e.g., `euphronios_sarpedon_krater`, `egyptian_canopic_jar_louvre_e_14384`)
- Add `object_type`: pottery, sculpture, manuscript, coin, textile, tomb_goods, etc.
- Attributes: origin_location, estimated_date, condition, legal_status (looted, disputed, recovered, etc.), current_location, provenance_notes

**LOCATION**: Geographic places where artifacts were looted, found, traded, or stored
- Canonical ID: `place_name_geography` (e.g., `geneva_freeport`, `el_minya_egypt`, `tomb_kv62_valley_of_kings`)
- Add `location_type`: excavation_site, freeport, dealer_location, museum, auction_house, country, region
- Attributes: significance (why mentioned), archaeological_importance

## Relationship Types

Identify connections between entities with these relation types:

- `looted_from`: ARTIFACT ← PERSON/ORGANIZATION, or ARTIFACT ← LOCATION
- `sold_to`: PERSON/ORGANIZATION → PERSON/ORGANIZATION (with date and artifact details)
- `handled_by`: ARTIFACT → PERSON/ORGANIZATION (dealer, restorer, auctioneer, etc.)
- `located_in`: ARTIFACT/ORGANIZATION → LOCATION
- `employed_by`: PERSON → ORGANIZATION
- `collaborated_with`: PERSON → PERSON (dealers working together)
- `recovered_by`: ARTIFACT ← PERSON/ORGANIZATION (law enforcement, customs)
- `displayed_at`: ARTIFACT → ORGANIZATION (museum exhibition)
- `authenticated_by`: ARTIFACT → PERSON (expert who verified provenance)
- `repatriated_to`: ARTIFACT → ORGANIZATION/LOCATION (return to origin)
- `specialized_in`: PERSON/ORGANIZATION → SPECIALIZATION_AREA (geographic region, time period, or object type)

## Extraction Workflow

1. **Read thoroughly** for context about each entity before extracting
2. **Identify entities** by their mention in the text (not just what's explicitly stated)
3. **Create canonical IDs** using the format guidelines—consistency is critical for later linking
4. **Collect mentions** (all text variants: "Medici", "Giacomo Medici", "the dealer Medici")
5. **Record attributes** from context clues, dates, roles, descriptions
6. **Find relationships** by looking for verbs and actions: sold, looted, handled, recovered, authenticated, employed
7. **Include dates** in relationship attributes whenever available
8. **Note uncertainty** in attributes or mention uncertain relationships only if strongly indicated

## Key Extraction Principles

- **Canonicalize consistently**: Use the same canonical_id every time a person/organization is mentioned, even if the text uses different forms
- **Be precise with roles**: A person may have multiple roles (dealer AND collector); include both if supported by the text
- **Capture specializations**: When text indicates a person or organization's area of focus, extract it as `specialization` or `collection_focus`. Look for patterns like:
  - "X collected [geographic/temporal/type] antiquities"
  - "dealer specializing in [area]"
  - "museum's collection of [area]"
  - "expert in [area]"
- **Include context**: Relationship attributes should capture what, when, and how (e.g., `artifact: "Euphronios krater"`, `date: "1967"`, `context: "sold at Sotheby's London"`)
- **Preserve ambiguity**: If a relationship is implied but not explicit, note it only if the text strongly suggests it
- **Track locations**: Every artifact should have provenance history (looted from, sold in, recovered from)
- **Capture aliases**: Some dealers used multiple names; list known aliases in the `mentions` array


## Specialization Extraction Patterns

Look for these language patterns when extracting areas of interest:

**Geographic focus:**
- "collected [region] antiquities" → South-East Asian, Egyptian, Roman, etc.
- "dealer in [region] art"
- "specialized in objects from [location]"

**Temporal focus:**
- "specialist in [period] artifacts" → Bronze Age, Tang Dynasty, Classical Period
- "[era] expert"

**Typological focus:**
- "focused on [object type]" → pottery, sculpture, coins, textiles
- "dealt primarily in [category]"

**Multiple specializations:**
- Extract all mentioned areas as an array
- Prioritize primary specialization if explicitly stated

## Reference Materials

See `references/extraction-schema.md` for complete JSON schema specification and examples.

See `references/entity-patterns.md` for common language patterns and clues that signal different entity types and relationships.

See `references/known-figures.md` for reference profiles of historically significant dealers and collectors (for disambiguation and attribute enrichment).

## Output Format

Always return valid JSON with this structure:

```json
{
  "entities": [
    {
      "canonical_id": "string",
      "type": "PERSON|ORGANIZATION|ARTIFACT|LOCATION",
      "full_name": "string (required for PERSON, optional for others)",
      "mentions": ["array", "of", "text", "mentions"],
      "attributes": {
        "role": "string or array (PERSON)",
        "nationality": "string (PERSON)",
        "birth_date": "YYYY (PERSON)",
        "death_date": "YYYY (PERSON)",
        "known_aliases": ["array (PERSON)"],
        "specialization": "string or array of geographic, temporal, or typological focus (PERSON)",
        "entity_type": "museum|gallery|auction_house|law_enforcement|university|customs|government|restoration_lab (ORGANIZATION)",
        "location": "string (ORGANIZATION)",
        "founded_date": "YYYY (ORGANIZATION)",
        "known_involvement": "string (ORGANIZATION)",
        "collection_focus": "string or array of primary collecting areas (ORGANIZATION)",
        "object_type": "pottery|sculpture|manuscript|coin|textile|tomb_goods|etc (ARTIFACT)",
        "origin_location": "string (ARTIFACT)",
        "estimated_date": "string (ARTIFACT)",
        "condition": "string (ARTIFACT)",
        "legal_status": "looted|disputed|recovered|legitimate|unknown (ARTIFACT)",
        "current_location": "string (ARTIFACT)",
        "provenance_notes": "string (ARTIFACT)",
        "location_type": "excavation_site|freeport|dealer_location|museum|auction_house|country|region (LOCATION)",
        "country": "string (LOCATION)",
        "significance": "string (LOCATION)",
        "archaeological_importance": "string (LOCATION)"
      }
    }
  ],
  "relationships": [
    {
      "source_id": "canonical_id",
      "target_id": "canonical_id",
      "relation_type": "looted_from|sold_to|handled_by|located_in|employed_by|collaborated_with|recovered_by|displayed_at|authenticated_by|repatriated_to|specialized_in",
      "attributes": {
        "date": "YYYY or YYYY-MM-DD if known",
        "artifact": "artifact name or canonical_id",
        "context": "brief description if needed",
        "source_text": "optional direct quote or page reference"
      }
    }
  ],
  "metadata": {
    "source_document": "title or reference",
    "extraction_notes": "any ambiguities, missing information, or uncertainties"
  }
}
```

## Validation Guidelines

When extracting specializations, ensure they are:
- **Specific enough to be meaningful** - not just "antiquities" but "Egyptian New Kingdom antiquities"
- **Consistent in terminology** across entities - standardize geographic and temporal terms
- **Supported by explicit or strongly implied text evidence** - don't infer specialization without textual basis
- **Formatted as arrays** when multiple specializations exist
- **Distinguished from roles** - "dealer" is a role, "Greek pottery dealer" has role AND specialization

Always validate that:
- All `source_id` and `target_id` values in relationships correspond to actual entities in the entities array
- Use underscores, not spaces
- Specializations use consistent terminology (create a controlled vocabulary as you extract)
- Multiple specializations are captured when a person/organization worked across areas
