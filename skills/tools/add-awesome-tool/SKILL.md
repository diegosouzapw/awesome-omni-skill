---
name: add-awesome-tool
description: This skill should be used when analyzing a link to an AI tool and adding it to the awesome-ai-tools readme with proper categorization
---

# Add Awesome Tool

Analyze a URL (or multiple URLs) to an AI tool, extract relevant information, and add it to the awesome-ai-tools data/tools.json file with proper categorization. The readme.md is automatically generated from the JSON data.

## Workflow Overview

This repository uses a **JSON-first approach**:
- **data/tools.json** - Source of truth for all tools
- **scripts/generate-readme.js** - Generates readme.md from tools.json
- **readme.md** - Auto-generated, do not edit directly

## Usage

When the user provides a link to an AI tool, use this skill to:

1. Fetch and analyze the webpage content
2. Extract key information (name, description, features, pricing, etc.)
3. Determine the appropriate category and subcategory
4. Add the tool entry to tools.json in the correct category
5. Regenerate readme.md from the updated JSON
6. Update the "Last Updated" date

The skill supports both single links and multiple links in one request.

## Workflow

To add a tool:

1. **Analyze the URL**: Use WebFetch to extract information from the tool's website
2. **Determine category**: Identify the correct category/subcategory from data/tools.json structure
3. **Create tool entry**: Format as JSON object with all relevant fields
4. **Update tools.json**: Add the tool to the appropriate category in data/tools.json
5. **Regenerate readme**: Run `node scripts/generate-readme.js` to update readme.md
6. **Update metadata**: Update lastUpdated field if needed

## Bundled Resources

### scripts/

**analyze_and_add.py**: Helper script (DEPRECATED - Use WebFetch + JSON editing instead):
- Fetches webpage content from provided URLs
- Extracts tool information (name, description, features, pricing)
- Outputs structured data for manual addition to tools.json

For the JSON workflow, use Claude's built-in tools:
- **WebFetch**: Extract information from tool websites
- **Read/Edit**: Modify data/tools.json directly
- **Bash**: Run `node scripts/generate-readme.js` to regenerate readme

### references/

**categories.md**: Complete list of categories from tools.json with descriptions to help with categorization

## Implementation

When the user provides a link (or says something like "add this tool"):

1. **Analyze the URL**:
   - Use WebFetch to extract information from the tool's website
   - Extract: name, type, website, repository, documentation, installation, key features, pricing, etc.

2. **Identify category**:
   - Read `data/tools.json` to see available categories
   - Determine correct category/subcategory based on tool type
   - Refer to `references/categories.md` for guidance

3. **Format as JSON**:
   - Create a tool object with relevant fields
   - Follow the structure of existing entries in tools.json
   - Only include fields that have actual values

4. **Update tools.json**:
   - Use Edit tool to add the new tool entry to the appropriate category
   - Insert alphabetically within the category if possible

5. **Regenerate readme**:
   ```bash
   node scripts/generate-readme.js
   ```

6. **Update metadata** in tools.json if needed (lastUpdated field)

## Categories

The readme contains these main categories:
- AI Inference Providers (with subcategories)
- MCP Providers
- CLI Tools
- Cloud-Based Agentic Coding Services
- VS Code Extensions
- JetBrains IDE Tools
- Full IDE Tools
- Code Review & Security Tools
- Testing & QA Tools
- API Testing Tools
- Documentation & Code Explanation
- Database & SQL Tools
- Local Model Infrastructure
- AI/ML Libraries & Frameworks
- Browser Extensions
- Search & Research Tools
- Other Tools & Infrastructure

## Entry Format

Each tool entry in tools.json follows this structure:

```json
{
  "name": "Tool Name",
  "type": "Brief description",
  "developer": "Company/Organization (if different from tool name)",
  "website": "https://example.com",
  "repository": "https://github.com/... (if open source)",
  "documentation": "https://docs.example.com (if available)",
  "installation": "installation command",
  "models": "Supported models",
  "keyFeatures": [
    "Feature 1",
    "Feature 2",
    "Feature 3"
  ],
  "pricing": "Pricing model (if applicable)",
  "worksWith": [
    "Compatible tools/platforms",
    "Integration options",
    "Use cases"
  ]
}
```

Common field names (use camelCase):
- name, type, developer, stakeholder, website, repository, documentation
- installation, models, keyFeatures, pricing, specialFeatures
- status, release, formerName, rebranding
- worksWith (array)

## Examples

**User**: "Add this tool: https://github.com/example/awesome-ai-cli"

**Assistant**:
1. Uses WebFetch to analyze the URL
2. Extracts: "Awesome AI CLI - A command-line tool for..."
3. Determines: "CLI Tools" → "Full Agentic Project-Level CLIs" category
4. Creates JSON object with all extracted fields
5. Edits tools.json to add the entry in correct category
6. Runs `node scripts/generate-readme.js` to update readme
7. Shows the user the generated entry

**User**: "Add these: https://tool1.com https://tool2.com"

**Assistant**: Processes both URLs, adds both to tools.json, then regenerates readme once

## Limitations

- Webpage content must be accessible (no paywalls or login requirements)
- Works best with official tool websites that have clear documentation
- May need manual adjustment for tools that fit multiple categories
- Cannot automatically determine "Works with:" compatibility without additional context
- JSON editing requires careful attention to syntax and structure
