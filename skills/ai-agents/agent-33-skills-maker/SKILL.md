---
name: agent-33-skills-maker
description: Create Claude Desktop skills from descriptions, SOPs, or existing agents. Three modes - create from scratch, convert SOPs to skills, or migrate agents to skill format. Outputs ready-to-deploy .zip packages.
---

# Agent-33: Skills Maker

## Core Principle
**Single agent, three input types, one output format.** Convert any workflow description, SOP document, or existing agent into a Claude Desktop skill package ready for deployment.

## When to Activate

Activate this agent when user:
- Wants to create a new skill for Claude Desktop
- Has an SOP that should become a skill
- Wants to migrate an existing agent to skill format
- Says "create a skill", "convert to skill", "migrate agent"
- Asks about skill generation or deployment

## Three Modes of Operation

### Mode 1: Create from Scratch
**Input:** Description of what the skill should do
**Output:** Complete skill package (.zip)

**Usage:**
```bash
./run_agent_33.py create \
  --name "skill-name" \
  --description "What it does" \
  --mcps "notion,gmail" \
  --examples "Example 1,Example 2"
```

### Mode 2: Convert SOP to Skill
**Input:** Standard Operating Procedure document
**Output:** Skill that automates the SOP workflow

**Usage:**
```bash
./run_agent_33.py sop \
  --input "path/to/sop.md" \
  --name "workflow-skill"
```

### Mode 3: Migrate Agent to Skill
**Input:** Existing agent (Sub-Agent or simple agent)
**Output:** Skill with same functionality

**Usage:**
```bash
./run_agent_33.py migrate \
  --agent 1 \
  --name "master-log-skill"
```

## Output Structure

All modes generate the same structure:
```
output/skill-name/
├── SKILL.md           # Complete skill definition
├── README.md          # Installation instructions
├── examples/          # Optional examples
└── skill-name.zip     # Ready for Claude Desktop import
```

## Quality Standards

All generated skills include:
- ✅ Proper frontmatter (name, description)
- ✅ Clear activation triggers
- ✅ MCP integration (if applicable)
- ✅ Success criteria checklist
- ✅ Examples and guidelines
- ✅ Professional README.md

## MCP Integration

Agent-33 automatically detects and includes MCP dependencies:

**Supported MCPs:**
- notion / notionApi
- gmail / server-gmail-autoauth-mcp
- playwright
- filesystem
- supabase
- wordpress-mcp
- elementor
- firecrawl-mcp
- fetch
- context7
- sequential-thinking
- applescript-control-mac

## Workflow Example

**User:** "Create a skill for Supabase query assistance"

**Agent-33 executes:**
1. Parse request → detect need for supabase MCP
2. Generate SKILL.md with:
   - Activation keywords: "supabase", "query", "database"
   - Guidelines for SQL best practices
   - MCP integration section
   - Success checklist
3. Generate README.md with installation steps
4. Package as .zip
5. Output: `output/supabase-query-helper/supabase-query-helper.zip`

**Result:** Ready to import into Claude Desktop

## Template Reference

Agent-33 uses proven skill patterns from:
- curv-design-system
- dashboard-auto-generation
- formula-botanica-slides

All generated skills follow the same tested structure.

## Success Checklist

When Agent-33 completes successfully:
- ✅ Skill directory created in `output/`
- ✅ SKILL.md has valid frontmatter
- ✅ README.md has installation instructions
- ✅ .zip package created and validated
- ✅ MCP dependencies detected and documented
- ✅ Activation keywords are specific and relevant

## Deployment Steps

After generation:
1. Locate `.zip` file in `output/skill-name/`
2. Open Claude Desktop → Settings → Skills
3. Click "Add Skill"
4. Upload the .zip file
5. Test activation with relevant keywords

## Technical Details

**Location:** `/Users/dannymcmillan/My Drive/Claude-Code-Projects/Sub-Agents/Agent-33-Skills-Maker/`

**Core Modules:**
- `skill_generator.py` - Core skill creation logic
- `package_builder.py` - .zip packaging
- `sop_parser.py` - Parse SOP documents
- `agent_parser.py` - Parse existing agents

**Validation:**
- Checks for required frontmatter
- Validates SKILL.md structure
- Ensures README.md exists
- Verifies .zip package integrity

## Common Use Cases

1. **SOP Automation:** Convert written procedures into executable skills
2. **Agent Migration:** Move 32 existing agents to Claude Desktop
3. **MCP Management:** Create skills that orchestrate multiple MCPs
4. **Workflow Packaging:** Bundle complex workflows as reusable skills

## Error Handling

**If skill generation fails:**
- Check input file exists (for SOP/migrate modes)
- Ensure name is kebab-case (lowercase-with-hyphens)
- Verify MCP names are valid
- Review terminal output for specific errors

**If .zip import fails in Claude Desktop:**
- **"Zip file contains path with invalid characters"** → macOS Icon files included (fixed in v1.1.0+)
  - Solution: Delete Icon files from skill folder before packaging
  - Or run: `find skill-folder -name "Icon*" -delete`
- Validate frontmatter in SKILL.md
- Ensure name and description are present
- Check .zip contains SKILL.md and README.md
- Verify no hidden files (.DS_Store, .git) in package

## Next Steps

After using Agent-33:
1. Review generated SKILL.md
2. Test skill in Claude Desktop
3. Refine activation keywords if needed
4. Deploy to production

## Version

- **Agent:** Agent-33-Skills-Maker
- **Version:** 1.1.0
- **Created:** 2025-10-20
- **Updated:** 2025-10-23 (Fixed macOS Icon\r file exclusion)
- **Compatibility:** Claude Desktop skills format
