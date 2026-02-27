---
name: touchdesigner-api-lookup
description: Query local TouchDesigner API documentation and class references. Use this skill when the user asks about specific TouchDesigner operators, Python classes, parameters, or methods.
version: 1.3.0
compatibility: Requires Python 3.
allowed-tools: Shell
---

# TouchDesigner API Lookup

This skill allows you to query a local database of TouchDesigner documentation. 
**ALWAYS use the provided Python scripts** to query information. **DO NOT** read the JSON files in `references/` directly.

## Available Scripts

### 1. List Class Members & Methods (`list_class.py`)
Use this to get an overview of a class, including inherited members/methods.
Useful when the user asks: "What parameters does Noise POP have?", "What methods are in OP?", "List properties of BaseCOMP".

**Usage:**
```bash
python scripts/list_class.py <ClassName>
```
*Note: You can use the Operator name (e.g., `NoisePOP`) or Class name (`NoisePOP_Class`).*

### 2. Get Member/Method Details (`get_detail.py`)
Use this to get detailed information (summary, type, arguments, return values) about a specific member or method.
Useful when the user asks: "How do I use the save method?", "What is the type of the 'seed' parameter?", "Arguments for cook method".

**Usage:**
```bash
python scripts/get_detail.py <ClassName> <MemberOrMethodName>
```

### 3. Update/Add Documentation (`scrape_docs.py`)
Use this to fetch the latest documentation from the TouchDesigner Wiki. Can scrape a single Operator or perform a batch update from the Category page.

**Single Operator Usage:**
```bash
python scripts/scrape_docs.py <OperatorName>
```
*Example: `python scripts/scrape_docs.py NoisePOP`*

**Batch Update Usage (Sync Category):**
To scrape all Python classes listed on the Category:Python_Reference page:
```bash
python scripts/scrape_docs.py --category --limit 0
```
*Use `--limit N` to test with only the first N classes.*
*This runs synchronously to avoid overloading the server.*

## Examples

**User:** "What parameters does the Noise POP have?"
**Agent:**
1. List members to find relevant ones:
   ```bash
   python scripts/list_class.py NoisePOP
   ```
2. The output shows `seed`. If the user wants details on `seed`:
   ```bash
   python scripts/get_detail.py NoisePOP seed
   ```

**User:** "How do I save a POP?"
**Agent:**
1. Check available methods:
   ```bash
   python scripts/list_class.py POP
   ```
2. See `save` method. Get details:
   ```bash
   python scripts/get_detail.py POP save
   ```

## Directory Structure
```
touchdesigner-api-lookup/
├── SKILL.md              # This file
├── scripts/
│   ├── list_class.py     # Lists inheritance, members, and methods
│   ├── get_detail.py     # Gets detailed info for a specific member/method
│   └── scrape_docs.py    # Scrapes TD Wiki to generate reference JSONs
└── references/           # JSON Data (DO NOT READ DIRECTLY)
```
