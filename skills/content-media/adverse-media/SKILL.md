---
name: adverse-media
description: Search for negative news coverage, controversies, and reputational risks associated with individuals or companies across news sources and media databases
allowed-tools: ["Bash", "Read", "Write", "WebSearch"]
---

# Adverse Media Screening Skill

## Purpose
This skill searches for negative news, controversies, scandals, and reputational risks associated with entities across global news sources and media databases.

## When to Use This Skill
Activate this skill when the user:
- Requests adverse media or negative news screening
- Asks about controversies or scandals involving an entity
- Needs reputational risk assessment
- Wants to check media coverage of a person or company
- Uses keywords like: "adverse media", "negative news", "scandals", "controversies", "bad press"

## How to Use

### 1. Identify Search Target
- Extract entity name
- Note time period if specified
- Consider name variations and aliases

### 2. Run Adverse Media Search
```bash
cd /Users/superfunguy/wsp/scolo/backend
python -c "from src.tools import adverse_media; import json; result = adverse_media.check('ENTITY_NAME'); print(json.dumps(result, indent=2))"
```

### 3. Analyze Results
Categories of adverse media:
- **Financial Crime**: Fraud, money laundering, embezzlement
- **Corruption**: Bribery, kickbacks, political corruption
- **Legal Issues**: Lawsuits, regulatory violations, arrests
- **Ethical Concerns**: Environmental damage, labor violations
- **Reputational**: Scandals, controversies, negative publicity

## Examples

### Example: Check Company Reputation
**User**: "Any negative news about Wells Fargo?"
**Action**:
```bash
python -c "from src.tools import adverse_media; import json; result = adverse_media.check('Wells Fargo'); print(json.dumps(result, indent=2))"
```

## Important Notes
- Consider source credibility and bias
- Distinguish between allegations and confirmed facts
- Check publication dates for relevance
- Multiple sources strengthen findings