---
name: pbi-squire
description: Analyze, create, and modify Power BI projects with intelligent assistance. Diagnose calculation issues, create new measures and visuals, and deploy changes with validation. Uses Microsoft's Power BI Modeling MCP for live semantic model editing when available, with automatic fallback to file-based TMDL manipulation. Supports PBIP projects and Power BI Desktop.
---

# PBI Squire

## Overview

Complete Power BI development assistant that orchestrates specialized agents for DAX calculations, M code transformations, and report design. Handles semantic model changes (measures, tables, columns), data transformations (Power Query/M code), and report-side changes (PBIR visuals, layouts).

**Key Capabilities:**
- Diagnose and fix calculation issues in existing dashboards
- Create new measures, calculated columns, tables, and visuals
- Transform data with Power Query / M code
- Document existing dashboards in business-friendly language
- Deploy changes with validation gates
- Merge and compare Power BI projects

**Architecture:**
- Uses Power BI Modeling MCP for live model editing when available
- Falls back to file-based TMDL manipulation automatically
- Orchestrates specialist agents via Task Blackboard pattern
- Routes unclear requests through clarification flow

---

## Quick Start

Tell me what you need help with. I'll route to the appropriate workflow:

| You say... | I'll do... |
|------------|-----------|
| "Fix this measure" / "Something is broken" | **EVALUATE** - diagnose and plan fixes |
| "Create a YoY growth measure" | **CREATE_ARTIFACT** - design new DAX artifacts |
| "Filter this table in Power Query" | **DATA_PREP** - M code transformations |
| "Explain what this dashboard does" | **SUMMARIZE** - document in business language |
| "Apply the changes" | **IMPLEMENT** - execute the planned changes |
| "Compare these two projects" | **MERGE** - diff and merge projects |
| "Set up data anonymization" | **SETUP_ANONYMIZATION** - mask sensitive columns |
| "Help me with Power BI" | I'll ask clarifying questions first |

**Not sure what you need?** Just describe your situation - I'll ask clarifying questions to understand your goal before starting any workflow.

**Common clarification questions I might ask:**
- "Are you trying to **fix** something, **create** something new, or **understand** something?"
- "Is this about **DAX calculations** or **Power Query / M code**?"
- "What's the path to your Power BI project?"

---

## When to Use This Skill

**Trigger Keywords:**
- Power BI, PBIX, PBIP, DAX, M code, Power Query
- Semantic model, measure, calculated column, TMDL, PBIR
- Create dashboard, fix measure, add visual, deploy report
- ETL, transformation, query folding, partition, data source
- Plugin version, update, check for updates, what version

**Trigger Actions:**
- "Fix this measure" → EVALUATE workflow
- "Create a YoY growth measure" → CREATE_ARTIFACT workflow (code artifacts)
- "Create a new calculated column" → CREATE_ARTIFACT workflow
- "Add a new dashboard page" → CREATE_PAGE workflow (Developer)
- "Build a visual" / "Create a card" → CREATE_PAGE workflow (Pro - visuals require PBIR)
- "Filter this table in Power Query" → DATA_PREP workflow
- "Edit the M code for..." → DATA_PREP workflow
- "Merge these two tables" → DATA_PREP workflow
- "Apply the changes" → IMPLEMENT workflow
- "What does this dashboard do?" → SUMMARIZE workflow
- "Explain this metric" → SUMMARIZE workflow
- "Document this dashboard" → SUMMARIZE workflow
- "Merge these two projects" → MERGE workflow
- "Set up data anonymization" → SETUP_ANONYMIZATION workflow
- "Mask sensitive columns" → SETUP_ANONYMIZATION workflow
- "Configure data masking" → SETUP_ANONYMIZATION workflow
- "Set up design standards" → SETUP_DESIGN_STANDARDS (Developer)
- "Review dashboard for consistency" → QA_LOOP with design critique (Developer)
- "Check against design guidelines" → QA_LOOP with design critique (Developer)
- "Check for updates" → VERSION_CHECK workflow
- "What version am I running?" → VERSION_CHECK workflow
- "Is PBI Squire up to date?" → VERSION_CHECK workflow
- "Help me with Power BI" → Ask clarifying questions first

**File Patterns:**
- `*.pbip`, `*.pbix`, `*.tmdl`, `*.bim`
- `*/.SemanticModel/**`, `*/.Report/**`

---

## Pre-Workflow Checks

Before executing any workflow, perform these checks in order:

### Step -2: MCP Availability Check (IMPORTANT)

**Purpose**: Detect whether Power BI Modeling MCP is available for live validation.

**How to check:**
1. Look for MCP server configuration in the project or global Claude settings
2. Attempt a simple MCP call if configuration suggests MCP is available

**MCP Status affects workflow behavior:**

| MCP Status | Reading/Analysis | Writing/Editing | Validation |
|------------|------------------|-----------------|------------|
| Available | ✅ Full features | ✅ Full features | ✅ Live DAX validation |
| Not Available | ✅ Full features | ⚠️ Works (no validation) | ⚠️ Structural only |

**If MCP is NOT available and workflow involves writing:**

```
┌──────────────────────────────────────────────────────────────────────────┐
│  ⚠️ MCP NOT DETECTED - Limited Validation Mode                           │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Power BI Modeling MCP is not configured for this session.               │
│                                                                          │
│  What this means:                                                        │
│  • Reading and analyzing TMDL/PBIR files: ✅ Works normally              │
│  • Creating/editing DAX code: ⚠️ Works but WITHOUT live validation       │
│  • Syntax errors will only be caught when you open in Power BI Desktop   │
│                                                                          │
│  To enable live validation:                                              │
│  1. Install Power BI Modeling MCP (VS Code extension)                    │
│  2. Open Power BI Desktop with your model                                │
│  3. Restart Claude Code                                                  │
│                                                                          │
│  Options:                                                                │
│  [P] Proceed without validation (I'll verify in Power BI Desktop)        │
│  [C] Cancel and set up MCP first                                         │
└──────────────────────────────────────────────────────────────────────────┘
```

**For read-only workflows (SUMMARIZE, VERSION_CHECK):** Proceed without warning since MCP is not needed.

**For write workflows (EVALUATE, CREATE_ARTIFACT, IMPLEMENT):** Show the warning above and let user decide.

#### MCP Mode Announcement (Required)

After determining MCP status, **always** display the operating mode to the user:

```
Operating Mode: LIVE MODE (MCP connected)
  → DAX validation, data sampling, and live queries available

Operating Mode: FILE MODE (no MCP)
  → Reading/writing TMDL files directly, no live validation
```

This ensures the user knows what capabilities are available for the session. Display this BEFORE proceeding to any workflow.

### Step -1: Project Setup Check (AUTOMATIC)

**Purpose**: Auto-configure the project on first skill invocation. No manual bootstrap needed for Analyst Edition.

**Check for setup indicators in the current project directory:**

1. Look for `.claude/pbi-squire.json`
2. Look for `CLAUDE.md` with plugin reference

**If BOTH exist → Continue to Step 0**

**If EITHER missing → Inline Setup Flow**

---

#### Inline Setup Flow (Both Editions)

When configuration is missing, automatically guide the user through setup inline:

##### 1. Detect Power BI Projects

Search the current working directory for Power BI projects:

```bash
# Find .pbip files within 2 levels
find . -maxdepth 2 -name "*.pbip" -type f 2>/dev/null
```

##### 2. Determine Configuration Scope

Based on the number of projects found:

| Projects Found | Mode | Action |
|----------------|------|--------|
| 0 | N/A | Ask user for project path |
| 1 | `single` | Auto-configure for that project |
| 2+ | `shared` | Ask if shared repository |

**For 0 projects found:**
```
┌─────────────────────────────────────────────────────────────────────┐
│  FIRST-TIME SETUP                                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  No Power BI projects (.pbip) found in the current directory.       │
│                                                                     │
│  Please provide the path to your Power BI project folder:           │
│                                                                     │
│  Example: C:\Projects\SalesReport                                   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

**For 1 project found (auto-configure as single):**
```
┌─────────────────────────────────────────────────────────────────────┐
│  FIRST-TIME SETUP                                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Found Power BI project:                                            │
│  • SalesReport/SalesReport.pbip                                     │
│                                                                     │
│  Configuring PBI Squire for this project...                         │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

**For 2+ projects found (ask about shared mode):**
```
┌─────────────────────────────────────────────────────────────────────┐
│  FIRST-TIME SETUP                                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Found 3 Power BI projects in this folder:                          │
│  • SalesReport/SalesReport.pbip                                     │
│  • SalesOverview/SalesOverview.pbip                                 │
│  • SalesKPIs/SalesKPIs.pbip                                         │
│                                                                     │
│  Configure as a shared repository where all these projects use      │
│  the same PBI Squire settings?                                      │
│                                                                     │
│  [Y] Yes - shared settings for all (Recommended)                    │
│  [N] No - configure each project separately                         │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

##### 3. Prompt for Data Sensitivity (BLOCKING)

**This step MUST complete before creating config files. Do NOT auto-select a default. Wait for the user's explicit answer.**

```
┌─────────────────────────────────────────────────────────────────────┐
│  DATA SENSITIVITY                                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Should PBI Squire require data anonymization before querying       │
│  data via MCP?                                                      │
│                                                                     │
│  Select YES if:                                                     │
│  • Data contains PII, financial, or healthcare information          │
│  • Your organization prohibits using real data with AI models       │
│  • Compliance policies require data masking                         │
│                                                                     │
│  [Y] Yes - require anonymization setup before data queries          │
│  [N] No - proceed without masking requirements                      │
│                                                                     │
│  ⚠️ You must choose one — there is no default.                      │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

##### 4. Create Configuration Files

Create the following files inline (no bootstrap script needed):

**`.claude/pbi-squire.json`** - Skill configuration:
```json
{
  "projectPath": "<detected-or-provided-path>",
  "dataSensitiveMode": <value-from-step-3>,
  "mode": "single",
  "projectOverrides": {}
}
```

> **CRITICAL:** The `dataSensitiveMode` value MUST reflect the user's answer from Step 3. Never default to `false`. If Step 3 has not been completed, do NOT create this file.

For shared repositories:
```json
{
  "projectPath": "<current-directory>",
  "dataSensitiveMode": <value-from-step-3>,
  "mode": "shared",
  "projectOverrides": {}
}
```

**`.claude/settings.json`** - Permissions (if doesn't exist):
```json
{
  "permissions": {
    "allow": [
      "Read",
      "Glob",
      "Grep",
      "Edit",
      "Write",
      "Bash(python *)",
      "Bash(git *)"
    ]
  }
}
```

**`CLAUDE.md`** - Add plugin reference (append if exists, create if not):
```markdown
## PBI Squire Plugin

This project uses the PBI Squire plugin for DAX/M code analysis.

Power BI projects are located at: `<project-path>`
```

**`.claude/tools/pbi-squire/version.txt`** - Version tracking:
- Read version from plugin's `tools/developer/version.txt`
- Write to local project

##### 5. Developer Edition Additional Check

After inline setup, check if this is Developer Edition:

1. Check if `skills/pbi-squire/developer-features.md` exists in the plugin directory
   - Exists → **Developer Edition**
   - Missing → **Analyst Edition** (continue normally)

2. **If Developer Edition**, check for Python tools:
   ```bash
   test -f ".claude/tools/pbi-squire/tmdl_format_validator.py" && echo "TOOLS_AVAILABLE" || echo "TOOLS_MISSING"
   ```

3. **If Python tools missing**, show informational message but **DO NOT block**:
   ```
   ┌─────────────────────────────────────────────────────────────────────┐
   │  ⚠️ PYTHON TOOLS NOT INSTALLED                                      │
   ├─────────────────────────────────────────────────────────────────────┤
   │                                                                     │
   │  Developer Edition detected, but Python analysis tools are not     │
   │  installed. Using Claude-native fallback (slightly slower).        │
   │                                                                     │
   │  For faster performance, run bootstrap to install Python tools:    │
   │                                                                     │
   │  Windows:                                                           │
   │  & "$HOME\.claude\plugins\custom\pbi-squire\tools\bootstrap.ps1"   │
   │                                                                     │
   │  macOS/Linux:                                                       │
   │  bash "$HOME/.claude/plugins/custom/pbi-squire/tools/bootstrap.sh" │
   │                                                                     │
   │  Continuing with Claude-native validation...                        │
   │                                                                     │
   └─────────────────────────────────────────────────────────────────────┘
   ```

4. **Continue with workflow** - do not require user to re-run command

---

**Key Principle**: Setup happens inline on first invocation. The user never needs to run a separate bootstrap step for Analyst Edition. Developer Edition can use bootstrap for Python tools, but works without them via fallback.

---

**Why Setup Matters**: Without configuration:
- CLAUDE.md won't reference the plugin properly
- Skill configuration won't exist
- Auto-approve permissions won't be set for Power BI files

### Step 0: Read Skill Configuration

Check for `.claude/pbi-squire.json` in the project directory:

```json
{
  "projectPath": "C:/Projects/SalesReport",
  "dataSensitiveMode": true
}
```

**`projectPath`:**
- If set → Use this as the default project location (skip project selection prompt)
- If null → Ask user for project path
- Can be a folder (will search for .pbip) or specific .pbip file

**`dataSensitiveMode`:**
- If `true` → Enforce anonymization check before any MCP data queries (Step 2 is required)
- If `false` → Proceed without anonymization checks (Step 2 can be skipped)

### Step 0.5: PBIX File Detection (CRITICAL - First Response)

**Purpose**: Immediately detect when user references a `.pbix` file and recommend conversion BEFORE any analysis begins.

**Check the user's provided path:**

1. If the path ends with `.pbix` (case-insensitive), this is a binary PBIX file
2. **STOP** - Do not proceed with any workflow

**Extract project name:** Remove `.pbix` extension from filename (e.g., `SalesReport.pbix` → `SalesReport`)

**Display this message IMMEDIATELY as the first response:**

```
╔═══════════════════════════════════════════════════════════════════════════╗
║  📦 PBIX FILE DETECTED - CONVERSION REQUIRED                              ║
╠═══════════════════════════════════════════════════════════════════════════╣
║                                                                           ║
║  You've pointed to a .pbix file:                                          ║
║  <user-provided-path>                                                     ║
║                                                                           ║
║  PBIX is a compressed binary format that significantly limits analysis.   ║
║  To get full visibility into your DAX, M code, and visuals, you need     ║
║  to convert to the Power BI Project (.pbip) format.                       ║
║                                                                           ║
╚═══════════════════════════════════════════════════════════════════════════╝

┌─────────────────────────────────────────────────────────────────────────┐
│  ⚠️  IMPORTANT: Each PBIP project needs its own folder!                  │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  When you save a PBIP project, Power BI creates MULTIPLE files:         │
│                                                                         │
│    <project-name>/                    ← Container folder (you create)   │
│    ├── <project-name>.pbip            ← Project file                    │
│    ├── <project-name>.SemanticModel/  ← Data model folder               │
│    └── <project-name>.Report/         ← Report folder                   │
│                                                                         │
│  Without a container folder, these files mix with other projects!       │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘

Would you like me to create the project folder for you?

  [Y] Yes, create: <configured-projects-folder>/<project-name>/
      Then I'll guide you through the Save As steps.

  [N] No, I'll handle it myself
      (Make sure to create a folder first, then Save As PBIP inside it)

  [E] Explain extraction options (DAX-only, limited analysis)
```

**Variable Substitutions:**
- `<user-provided-path>`: The exact path the user provided
- `<configured-projects-folder>`: Read from `.claude/pbi-squire.json` → `projectPath`
  - If `projectPath` is null: Use "a short path like C:\PBI or D:\Projects"
- `<project-name>`: Extract from the .pbix filename (without extension)

**If user selects [Y] (Create folder for me):**

1. Create the folder using Bash:
   ```bash
   mkdir -p "<configured-projects-folder>/<project-name>"
   ```

2. Display success message with Save As instructions:
   ```
   ✅ Folder created: <configured-projects-folder>/<project-name>/

   Now complete the conversion in Power BI Desktop:
   ─────────────────────────────────────────────────
   1. Open Power BI Desktop
   2. File → Open → Select: <user-provided-pbix-path>
   3. File → Save As → Power BI Project (.pbip)
   4. Navigate to: <configured-projects-folder>/<project-name>/
   5. Click Save (use the default filename)

   When you're done, let me know and I'll analyze the project at:
   <configured-projects-folder>/<project-name>
   ```

3. Wait for user confirmation, then continue to Step 1 with the new path

**If user selects [N] (I'll handle it myself):**
- Remind them to create a containing folder first
- Display the recommended path structure
- Wait for them to provide the new PBIP folder path
- When they return with a path, continue to Step 1

**If user selects [E] (Explain extraction options):**
- Explain pbi-tools extraction option (DAX-only)
- Emphasize this is limited analysis without M code visibility
- Still recommend PBIP conversion for full analysis

**Why Each Project Needs Its Own Folder:**
- PBIP "Save As" creates 3+ items at the save location (not inside a single file)
- Without a container folder, multiple projects' files intermix
- This makes it impossible to identify which files belong to which project
- A clean folder structure enables working with multiple projects over time

**Why This Step is First:**
- Users often have .pbix files on their Desktop or Downloads folder
- The first response should guide them to the proper workflow
- Offering to create the folder removes friction from the conversion process
- This prevents wasted time analyzing a format with limited capabilities

### Step 1: Project Selection (Multi-Project Handling)

**Read config to determine mode:**
- Load `.claude/pbi-squire.json`
- Check `mode` field: `"single"` or `"shared"`

---

#### Single Mode

When `mode: "single"`:
- Use `projectPath` directly
- If it's a folder, search for .pbip files
- If 1 project found → use it automatically
- If 0 projects found → report error and suggest checking the path

---

#### Shared Repository Mode

When `mode: "shared"`:

1. **List all .pbip files in `projectPath`**
2. **Check for project-specific overrides in `projectOverrides`**
3. **Ask user which project to work with THIS session**

```
┌─────────────────────────────────────────────────────────────────────┐
│  📁 Repository: C:/Projects/Analytics                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Found 4 Power BI projects:                                         │
│                                                                     │
│  [1] HR-Dashboard        ⚠️ data masking required                   │
│  [2] Sales-Report                                                   │
│  [3] Marketing-Overview                                             │
│  [4] Finance-Summary                                                │
│                                                                     │
│  Which project? (1-4 or project name):                              │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

4. **Apply override settings for selected project**:
   - Check if `projectOverrides["<project-name>"]` exists
   - If exists, merge those settings with base config
   - Example: `dataSensitiveMode` may be overridden per-project

**Example config with overrides:**
```json
{
  "projectPath": "C:/Projects/Analytics",
  "dataSensitiveMode": false,
  "mode": "shared",
  "projectOverrides": {
    "HR-Dashboard": {
      "dataSensitiveMode": true
    }
  }
}
```

When "HR-Dashboard" is selected, `dataSensitiveMode` becomes `true` for that session.

---

#### Fallback Behavior

When the user provides a folder path directly (not from config):

1. Search for `*.pbip` files in the target folder
2. **If 1 project found**: Use it automatically
3. **If multiple projects found**: Ask user to select:
   ```
   Found N Power BI projects:
   1. ProjectA/ProjectA.pbip
   2. ProjectB/ProjectB.pbip

   Which project would you like to work with?
   ```
4. **If 0 projects found**: Report error and suggest checking the path

### Step 2: Anonymization Check (Before Data Queries)

**After selecting a specific project**, check for anonymization configuration:

1. Look for `.anonymization/config.json` **inside the project folder**
   - Correct: `ProjectA/.anonymization/config.json`
   - Wrong: `ParentFolder/.anonymization/config.json`

2. **If config exists**: Verify `DataMode` parameter is set appropriately before sampling data

3. **If config does NOT exist** and workflow will query data:
   ```
   ⚠️ This project doesn't have anonymization configured.
   MCP queries may expose sensitive data.

   Options:
   1. Set up anonymization first (/setup-data-anonymization --project "<path>")
   2. Proceed anyway (data is not sensitive)
   3. Work in file-only mode (no data queries)
   ```

### Safe Operations (No Anonymization Check Needed)

These operations don't expose data and can proceed without anonymization:
- `dax_query_operations.validate()` — syntax check only
- `measure_operations.list()` — metadata only
- `table_operations.list()` — metadata only
- File-based TMDL reads/edits

---

## Workflows

### EVALUATE (Diagnose/Fix)

**Use when:** User reports something is broken, incorrect, or not working as expected.

**Process:**
1. Connect to project (MCP or file-based)
2. Clarify the problem through interactive Q&A
3. Investigate existing code and data
4. Plan changes with proposed fixes
5. Verify approach before implementation

**Output:** `findings.md` with diagnosed issues and proposed fixes

**Next step:** "implement the changes" to apply fixes

---

### CREATE_ARTIFACT (New Measure/Column/Table)

**Use when:** User wants to add new **code-based artifacts** (DAX measures, calculated columns, tables).

> **Note:** For visuals, use CREATE_PAGE workflow. Visuals require layout coordinates, field bindings, and PBIR file generation which this workflow doesn't handle.

**Process:**
1. Connect to project and analyze schema
2. Decompose request into discrete code artifacts
3. Specify each artifact through interactive Q&A
4. Discover existing patterns for consistency
5. Generate DAX/M code with validation

**Output:** `findings.md` with new artifact specifications (DAX/M code)

**Next step:** "implement the changes" to create artifacts

**Can also be called in embedded mode** by CREATE_PAGE workflow to generate measures as part of page creation.

---

### CREATE_PAGE (New Dashboard Page) ⭐ Pro

**Use when:** User wants a complete new report page with multiple visuals.

> **Pro Feature:** This workflow requires the Developer Edition. Core users can create individual measures with CREATE_ARTIFACT, then manually add visuals in Power BI Desktop.

**Process:**
1. Validate project has .Report folder (PBIR required)
2. Analyze the business question being answered
3. Decompose into required measures and visuals
4. Specify measures (delegates to CREATE_ARTIFACT in embedded mode)
5. Design layout using 8px grid system
6. Design interactions (cross-filtering, drill-through)
7. Generate PBIR files

**Output:** `findings.md` with measures + visual specifications + PBIR files

**Next step:** "implement the changes" to create the page

---

### DATA_PREP (M Code / Power Query)

**Use when:** User wants to modify data transformations, Power Query logic, table filtering, column operations, or any ETL changes.

**Trigger phrases:**
- "Filter this table to only include..."
- "Add a column that calculates..."
- "Merge these two tables"
- "Change the data source"
- "Optimize this Power Query"
- "Edit the M code for..."

**Process:**
1. **Analyze patterns** - Discover naming conventions, existing transformation styles in the project
2. **Design transformation** - Present simplest approach first; show alternatives with pros/cons if relevant
3. **Check query folding** - Validate performance impact; warn if proposed changes break folding
4. **Apply M code changes** - Safe TMDL partition editing with backups
5. **Validate syntax** - TMDL format validation

**Key considerations:**
- Always check query folding impact before applying (see `references/query_folding_guide.md`)
- Follow existing project naming patterns
- Present alternatives when complexity trade-offs exist:
  ```
  Option 1 (Recommended): Filter in M code
    ✓ Simple and straightforward
    ✓ Maintains query folding
    ✗ Loads all data then filters

  Option 2: Filter at source (SQL WHERE)
    ✓ Better performance for large datasets
    ✗ More complex to maintain
  ```

**Query folding rules (quick reference):**
- Preserves folding: SelectRows, SelectColumns, RenameColumns, Sort, Group, indexed joins
- Breaks folding: Custom columns with M functions, text operations, pivot/unpivot

**Output:** Updated TMDL partition files + validation results

**References:**
- `references/query_folding_guide.md` - Complete folding rules
- `references/common_transformations.md` - M code pattern library
- `references/m_best_practices.md` - Style guide
- `references/tmdl_partition_structure.md` - TMDL formatting

---

### IMPLEMENT (Apply Changes)

**Use when:** User has reviewed findings and wants to apply proposed changes.

**Prerequisites:** Must have `findings.md` from previous workflow

**Process:**
1. Validate findings file exists with proposed changes
2. Apply code changes (MCP transactions or file writes)
3. Validate DAX syntax
4. Apply visual changes (PBIR)
5. Validate PBIR structure
6. Deploy (optional, if requested)
7. Test (optional, if URL available)

**Output:** Updated project files + validation results

---

### SUMMARIZE (Document Existing)

**Use when:** User wants to understand what an existing dashboard does, explain metrics to stakeholders, or create documentation.

**Trigger phrases:**
- "Tell me what this dashboard is doing"
- "Explain how this metric is calculated"
- "Document this report for the business team"
- "What does this page show?"
- "Help me understand this dashboard"

**Process:**
1. **Validate project** - Ensure .Report folder exists (required for visual analysis)
2. **Extract structure** - Parse pages, visuals, filters, interactions from PBIR
3. **Extract measures** - Parse DAX definitions and dependencies from TMDL
4. **Translate to business language** - Apply translation guidelines
5. **Generate report** - Structured markdown documentation

**Translation principles:**
- Focus on "what" and "why", not "how"
- Use business terminology (not DAX function names)
- Describe visual purpose, not just type
- Include just enough technical detail for credibility

**Example translations:**
| Technical | Business |
|-----------|----------|
| `CALCULATE(SUM(...), ALL(...))` | Total ignoring current filters |
| `SAMEPERIODLASTYEAR` | Same period last year |
| Line chart with Date on X-axis | Line chart tracking trends over time |

**Output:** `dashboard_analysis.md` with:
- **Executive Summary** - What the dashboard does, who it's for
- **Page-by-Page Analysis** - Each page's purpose and visuals
- **Metrics Glossary** - Business-friendly measure definitions with dependencies
- **Filter & Interaction Guide** - How users navigate and filter

**Output example structure:**
```markdown
# Dashboard Analysis: Sales Performance

## Executive Summary
This dashboard provides sales leadership with visibility into revenue
performance, regional comparisons, and year-over-year growth trends.

## Pages

### 1. Executive Summary
**Purpose:** High-level KPIs for executive stakeholders

**Visuals:**
- **Total Revenue Card** - Current total revenue ($2.4M)
- **YoY Growth Card** - Year-over-year change (+12.5%)

**Filters:** Year, Quarter

### 2. Regional Breakdown
[...]

## Metrics Glossary

### Total Sales
- **Definition:** Sum of all invoice amounts excluding returns
- **Dependencies:** Sales Amount column
- **Logic:** Filters out cancelled orders

### YoY Growth %
- **Definition:** Percentage change vs same period last year
- **Dependencies:** Total Sales, Prior Year Sales
```

**References:**
- `references/translation-guidelines.md` - Technical to business language
- `references/visual_types.md` - How to describe each visual type
- `references/dax_common_patterns.md` - Common DAX pattern translations
- `references/interaction_patterns.md` - Interaction behavior descriptions
- `assets/analysis_report_template.md` - Full report template

---

### MERGE (Compare/Merge Projects)

**Use when:** User wants to compare or combine two Power BI projects.

**Process:**
1. Connect to both projects
2. Extract schemas from both
3. Compare and identify differences
4. Explain semantic meaning of differences
5. Present decisions for each conflict
6. Apply merge with user approval

**Output:** Merged project + merge report

---

### VERSION_CHECK (Plugin Status)

**Use when:** User asks about plugin version, updates, tier (Pro/Free), or installation status.

**Trigger phrases:**
- "Check for updates"
- "What version am I running?"
- "Is PBI Squire up to date?"
- "Am I on Pro or Free?"
- "Update the plugin"

**Process:**

1. **Read Plugin Metadata**
   - Load `.claude-plugin/plugin.json` from plugin installation directory
   - Extract: `version`, `repository`
   - Detect edition: Check if `skills/pbi-squire/developer-features.md` exists (Developer) or not (Analyst)

2. **Read Project Version (if bootstrapped)**
   - Check `.claude/tools/pbi-squire/version.txt` in current project
   - Compare with plugin version

3. **Report Status**
   ```
   ┌──────────────────────────────────────────────────┐
   │ PBI Squire Status                          │
   ├──────────────────────────────────────────────────┤
   │ Plugin (global):                                 │
   │   Version:  1.3.0 (Developer Edition)                  │
   │   Location: ~/.claude/plugins/custom/powerbi-*  │
   ├──────────────────────────────────────────────────┤
   │ Project (local):                                 │
   │   Version:  1.2.0  ← Update available            │
   │   Location: .claude/tools/pbi-squire/       │
   ├──────────────────────────────────────────────────┤
   │ Update Instructions:                             │
   │   Plugin:  cd ~/.claude/plugins/custom/power*    │
   │            git pull                              │
   │   Project: Run bootstrap.ps1 from project dir   │
   └──────────────────────────────────────────────────┘
   ```

4. **If Update Requested**
   - Provide step-by-step update commands
   - Warn about any breaking changes (if known)

**Output:** Version status report with update instructions if needed

**Plugin metadata location:** `$HOME/.claude/plugins/custom/pbi-squire/.claude-plugin/plugin.json`

**See also:** `references/update-info.md` for detailed update procedures

---

### SETUP_ANONYMIZATION (Data Masking)

**Use when:** User wants to set up data anonymization to protect sensitive information before using MCP queries.

**Trigger phrases:**
- "Set up data anonymization"
- "Mask sensitive columns"
- "Configure data masking"
- "Hide PII in my data"
- "Anonymize customer data"

**Process:**
1. **Scan for sensitive columns** - Parse TMDL files for table/column definitions
2. **Match against patterns** - Detect names, emails, SSN, phones, addresses, amounts
3. **Confirm with user** - Present findings grouped by confidence (HIGH/MEDIUM/LOW)
4. **Generate M code** - Create DataMode parameter and masking transformations
5. **Apply changes** - Edit partition TMDL files with user approval
6. **Configure** - Write `.anonymization/config.json` and update skill config

**What gets created:**
- `DataMode` parameter (toggle between "Real" and "Anonymized")
- Conditional M code transformations in table partitions
- Configuration file for tracking masked columns

**Masking strategies:**

| Strategy | Example | Use For |
|----------|---------|---------|
| Sequential numbering | "Customer 1, Customer 2" | Names |
| Fake domain | "user1@example.com" | Emails |
| Partial mask | "XXX-XX-1234" | SSN, Tax ID |
| Fake prefix | "(555) 555-1234" | Phone numbers |
| Scale factor | Original * 0.8-1.2 | Financial amounts |
| Date offset | Original +/- 30 days | Birth dates |
| Placeholder | "[Content redacted]" | Free text |

**Output:**
- Updated TMDL files with masking logic
- `.anonymization/config.json` with configuration
- Updated `.claude/pbi-squire.json` with `dataSensitiveMode: true`

**Invokes:** `powerbi-anonymization-setup` agent

**References:**
- `references/anonymization-patterns.md` - Detection patterns and M code templates

**After setup:**
To test, open Power BI Desktop → Transform Data → Manage Parameters → Change DataMode to "Anonymized".

---

## Subagent Architecture

This skill uses Claude Code's **leaf subagent pattern** for context isolation and parallel execution. The **main thread** (skill layer) orchestrates workflows and spawns specialized subagents directly.

### Main Thread Orchestration (CRITICAL)

**IMPORTANT**: Claude Code subagents cannot spawn other subagents. Therefore:
- The **main thread** (skill layer) manages multi-phase workflows
- The **main thread** spawns leaf subagents directly via `Task()`
- Subagents perform their work and return results; they do NOT spawn other subagents

### Workflow Execution Pattern

**Execute workflows in the MAIN THREAD (do NOT delegate to an orchestrator subagent):**

1. **Load the appropriate workflow file** based on trigger:
   - "Fix this measure" → Load `workflows/evaluate-pbi-project-file.md`
   - "Create a YoY growth measure" → Load `workflows/create-pbi-artifact-spec.md`
   - "Apply the changes" → Load `workflows/implement-deploy-test-pbi-project-file.md`
   - "Merge these two projects" → Load `workflows/merge-powerbi-projects.md`

2. **Execute phases directly in main thread**:
   - Create scratchpad with `findings.md`
   - Spawn investigation subagents (parallel): `Task(powerbi-code-locator)`, `Task(powerbi-visual-locator)`
   - Check quality gates (main thread reads findings.md)
   - Spawn planning subagent: `Task(powerbi-dashboard-update-planner)`
   - If planner wrote specialist specs → Spawn specialists: `Task(powerbi-dax-specialist)`, `Task(powerbi-mcode-specialist)`
   - Spawn validation subagents (parallel): `Task(powerbi-dax-review-agent)`, `Task(powerbi-pbir-validator)`
   - Report completion

3. **Subagents communicate via findings.md**:
   - Subagents read/write to designated sections
   - Main thread coordinates and checks quality gates
   - No subagent-to-subagent communication

**Reference:** See `references/orchestration-pattern.md` for detailed routing logic and quality gates

### Specialist Agents

The orchestrator delegates to specialized agents based on artifact type:

#### DAX Specialist (`powerbi-dax-specialist`)
**Handles:** Measures, calculated columns, calculation groups, KPIs

**Expertise:**
- Time Intelligence (SAMEPERIODLASTYEAR, DATEADD, etc.)
- Filter Context (CALCULATE, FILTER, ALL, KEEPFILTERS)
- Performance patterns (DIVIDE, iterator vs aggregator)
- Relationship-aware calculations (RELATED, USERELATIONSHIP)

#### M-Code Specialist (`powerbi-mcode-specialist`)
**Handles:** Partitions (table M queries), named expressions, ETL

**Expertise:**
- ETL patterns (Table.TransformColumns, Table.AddColumn)
- Query folding optimization
- Privacy levels
- Data type enforcement
- Error handling (try/otherwise)

### Investigation Agents

| Agent | Purpose | Output Section |
|-------|---------|----------------|
| `powerbi-code-locator` | Find DAX/M/TMDL code | Section 1.A |
| `powerbi-visual-locator` | Find PBIR visuals | Section 1.B |
| `powerbi-data-context-agent` | Query live data via XMLA | Section 1.C |
| `powerbi-pattern-discovery` | Find similar artifacts | Section 1.D |

### Planning Agents

| Agent | Purpose | Output Section |
|-------|---------|----------------|
| `powerbi-dashboard-update-planner` | Design calculation & visual changes | Section 2 |
| `powerbi-artifact-decomposer` | Break complex requests into artifacts | Section 1.0 |
| `powerbi-data-understanding-agent` | Build specifications via Q&A | Section 1.2 |

### Validation Agents

| Agent | Purpose | Output Section |
|-------|---------|----------------|
| `powerbi-dax-review-agent` | Validate DAX syntax & best practices | Section 2.5 |
| `powerbi-pbir-validator` | Validate PBIR visual.json structure | Section 2.6 |
| `power-bi-verification` | Generate test cases & impact analysis | Section 3 |

### Execution Agents

| Agent | Purpose | Output Section |
|-------|---------|----------------|
| `powerbi-code-implementer-apply` | Apply TMDL changes | Section 4.A |
| `powerbi-visual-implementer-apply` | Apply PBIR changes | Section 4.B |

### Configuration Agents

| Agent | Purpose | Output |
|-------|---------|--------|
| `powerbi-anonymization-setup` | Set up data masking for sensitive columns | .anonymization/config.json |

### Pro-Only Agents

These agents are available only in the Developer Edition:

| Agent | Purpose |
|-------|---------|
| `powerbi-playwright-tester` | Browser automation testing |
| `powerbi-ux-reviewer` | Design critique from screenshots |
| `powerbi-qa-inspector` | DOM inspection for deployment errors |

### Agent Directory Structure

All agents are **leaf subagents** (they do not spawn other subagents):

```
agents/
├── core/                    # Core agents (all editions)
│   ├── powerbi-code-locator.md      # Investigation
│   ├── powerbi-visual-locator.md    # Investigation
│   ├── powerbi-dashboard-update-planner.md  # Planning (writes specs)
│   ├── powerbi-dax-specialist.md    # Code generation
│   ├── powerbi-mcode-specialist.md  # Code generation
│   ├── powerbi-dax-review-agent.md  # Validation
│   ├── powerbi-pbir-validator.md    # Validation
│   └── ... (additional leaf agents)
│
└── pro/                     # Pro agents (Developer Edition only)
    ├── powerbi-playwright-tester.md
    ├── powerbi-ux-reviewer.md
    └── powerbi-qa-inspector.md
```

**Note**: Orchestration logic is in `references/orchestration-pattern.md` and executed by the main thread, not a subagent.

---

## Connection Modes

### Power BI Desktop Mode (Default)
- Connects to running Power BI Desktop instance
- Uses Windows Integrated Authentication
- Full MCP capabilities

### File-Only Mode (Fallback)
- Works directly with TMDL/PBIR files
- No live validation
- Reduced capabilities (warned per operation)

---

## Session State

The skill maintains session state in `.claude/state.json`:
- Active tasks and their status
- Model schema cache
- Resource locks (for file-based mode)
- Connection status

Tasks use the **Task Blackboard pattern** where specialists read/write to designated sections of `findings.md`.

---

## Project Setup (Automatic or Bootstrap)

The skill auto-configures on first invocation. No separate bootstrap step required for Analyst Edition.

**Analyst Edition:** Auto-configures inline (no Python required)
**Developer Edition:** Auto-configures inline + optional bootstrap for Python tools

### Automatic Setup (Default)

When you first invoke the skill in a new project, it will:
1. Detect Power BI projects in your folder
2. Ask about shared repository mode (if multiple projects found)
3. Ask about data sensitivity settings
4. Create configuration files inline
5. Continue with your request

See "Step -1: Project Setup Check" for the detailed flow.

### Bootstrap Process (Optional)

**Use bootstrap for:**
- Developer Edition Python tools (faster performance)
- CI/CD automation (explicit setup without prompts)
- Force refresh of configuration

```powershell
# Windows
& "$HOME\.claude\plugins\custom\pbi-squire\tools\bootstrap.ps1"

# macOS/Linux
bash "$HOME/.claude/plugins/custom/pbi-squire/tools/bootstrap.sh"
```

**What gets created (Analyst Edition):**
```
YourProject/
├── CLAUDE.md                        ← Project instructions
├── .claude/
│   ├── pbi-squire.json         ← Skill configuration
│   ├── settings.json                ← Permissions
│   ├── tasks/                       ← Task findings files
│   ├── tools/
│   │   └── pbi-squire/
│   │       └── version.txt          ← Version tracking only
│   └── helpers/
│       └── pbi-squire/
│           └── pbi-url-filter-encoder.md
└── YourProject.pbip
```

**Additional files for Developer Edition:**
```
YourProject/
├── .claude/
│   ├── tools/
│   │   └── pbi-squire/         ← Python tools (Pro only)
│   │       ├── token_analyzer.py
│   │       ├── tmdl_format_validator.py
│   │       └── ... (13 Python scripts)
│   └── powerbi-design-standards.md  ← Design template (Pro only)
└── ...
```

### Version Tracking

The bootstrap script tracks versions to manage updates:

| Command | Purpose |
|---------|---------|
| `bootstrap.ps1` | Install/update tools if needed |
| `bootstrap.ps1 -CheckOnly` | Check if update available (exit code 1 = update needed) |
| `bootstrap.ps1 -Force` | Force reinstall even if current |

### Updating the Plugin

To update this skill to the latest version:

**Step 1: Pull latest plugin code**
```powershell
cd "$HOME\.claude\plugins\custom\pbi-squire"
git pull
```

**Step 2: Update project tools (run from your project directory)**
```powershell
cd "C:\path\to\your\project"
& "$HOME\.claude\plugins\custom\pbi-squire\tools\bootstrap.ps1"
```

**Quick check if update is available:**
```powershell
& "$HOME\.claude\plugins\custom\pbi-squire\tools\bootstrap.ps1" -CheckOnly
# Exit code 1 = update available, 0 = current
```

**Plugin location:** `$HOME\.claude\plugins\custom\pbi-squire\`

### What Gets Overwritten on Update

| File | Behavior | Safe to Customize? |
|------|----------|-------------------|
| `CLAUDE.md` | Prompts before appending | ✅ Yes |
| `.claude/settings.json` | Skips if exists | ✅ Yes |
| `.claude/pbi-squire.json` | Skips if exists | ✅ Yes |
| `.claude/tasks/*` | Not touched | ✅ Yes |
| `.claude/tools/pbi-squire/*` | Overwritten | ❌ No (plugin-managed) |
| `.claude/helpers/pbi-squire/*` | Overwritten | ❌ No (plugin-managed) |
| `.claude/tools/*.py` (root) | Not touched | ✅ Yes (your scripts) |
| `.claude/helpers/*` (root) | Not touched | ✅ Yes (your files) |

**Important:** Only the `pbi-squire/` subfolders are plugin-managed. Your own scripts in `.claude/tools/` or `.claude/helpers/` are safe.

To customize behavior, edit:
- `.claude/settings.json` - Permissions and Claude Code settings
- `.claude/pbi-squire.json` - Skill configuration
- `CLAUDE.md` - Project-specific instructions

---

## Validation Gates

All changes pass through validation before completion:

| Validator | What it Checks | Blocking |
|-----------|----------------|----------|
| DAX Review | Syntax, semantics, best practices | Yes (errors) |
| PBIR Validator | JSON structure, config integrity | Yes (errors) |
| MCP Validation | Live syntax check (MCP mode only) | Yes (errors) |

Warnings are reported but don't block implementation.

---

## Tool-First Fallback Pattern (Core vs Pro)

The plugin uses a **tool-first fallback pattern** to provide optimal performance for Pro users while ensuring Core users can still accomplish all tasks.

### How It Works

| Check | Developer Edition | Analyst Edition |
|-------|-------------|--------------|
| Python tools available? | Yes (via bootstrap) | No |
| Execution speed | Fast (milliseconds) | Slower (uses LLM) |
| Token cost | Lower | Higher |
| Functionality | Same | Same |

### When Agents Use Tools

Agents check for tool availability before executing:

```bash
# Example check
test -f ".claude/tools/tmdl_format_validator.py" && echo "TOOL_AVAILABLE" || echo "TOOL_NOT_AVAILABLE"
```

- **If available**: Use Python tool for faster, deterministic execution
- **If not available**: Use Claude-native approach (Read, Edit, reference docs)

### Tool Mapping

| Task | Pro Tool | Core Fallback |
|------|----------|---------------|
| TMDL validation | `tmdl_format_validator.py` | Claude validates against `tmdl_partition_structure.md` |
| Visual editing | `pbir_visual_editor.py` | Claude uses Edit tool on visual.json |
| M code editing | `m_partition_editor.py` | Claude uses Edit tool with tab handling |
| Pattern analysis | `m_pattern_analyzer.py` | Claude reads and analyzes TMDL |
| Anonymization | `sensitive_column_detector.py` | Claude uses `anonymization-patterns.md` |

See `references/tool-fallback-pattern.md` for complete documentation.

---

## Tracing & Observability

The skill provides structured trace output to show workflow progress, agent invocations, and MCP tool usage.

### Trace Output Format

**Workflow Start:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🚀 WORKFLOW: [workflow-name]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Phase Markers:**
```
📋 PHASE [N]: [Phase Name]
   └─ [Description of what's happening]
```

**Agent Invocations:**
```
   └─ 🤖 [AGENT] [agent-name]
   └─    Starting: [brief description]
   └─ 🤖 [AGENT] [agent-name] complete
   └─    Result: [summary]
```

**MCP Tool Calls:**
```
   └─ 🔌 [MCP] [tool-name]
   └─    [parameters or context]
   └─    ✅ Success: [result summary]
   └─    ❌ Error: [error message]
```

**Workflow Complete:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ WORKFLOW COMPLETE: [workflow-name]
   └─ Output: [file path or summary]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Icon Reference

| Icon | Meaning |
|------|---------|
| 🚀 | Workflow start |
| 📋 | Phase/step |
| 🤖 | Agent invocation |
| 🔌 | MCP tool call |
| 📂 | File operation |
| 🔍 | Search/discovery/validation |
| ✏️ | Edit/write |
| ✅ | Success |
| ❌ | Error/failure |
| ⚠️ | Warning |

See `resources/tracing-conventions.md` for complete tracing documentation.

---

## References

This skill includes reference documentation:

### `assets/findings_template.md`
Template for Task Blackboard used by all workflows.

### Visual Templates (Public Repository)
PBIR visual templates with `{{PLACEHOLDER}}` syntax are maintained in a public repository:

**GitHub:** https://github.com/cn-dataworks/pbir-visuals/visual-templates/

Template types include:
- Cards, line charts, bar charts, column charts
- Tables, matrices, pie charts, scatter charts
- Azure maps (gradient, bubble)
- Slicers (date range, dropdown, multi-select)
- Static images

Templates are fetched via `WebFetch` from raw.githubusercontent.com.
See `assets/visual-templates/README.md` for usage and contribution instructions.

### `references/`
- `getting-started.md` - Onboarding guide with data masking workflow
- `glossary.md` - Technical terms explained
- `troubleshooting-faq.md` - Common issues and solutions
- `update-info.md` - Version management, update procedures, edition detection

---

## Example Prompts

| Goal | Example Prompt | Workflow |
|------|----------------|----------|
| Fix a problem | "Help me fix the YoY calculation in my Sales dashboard" | EVALUATE |
| Create DAX measure | "Create a margin percentage measure" | CREATE_ARTIFACT |
| Create calculated column | "Add a full name column combining first and last" | CREATE_ARTIFACT |
| Create a visual | "Add a sales KPI card with YoY comparison" | CREATE_PAGE (Developer) |
| Create a page | "Build a regional performance dashboard page" | CREATE_PAGE (Developer) |
| Transform data | "Filter the Customers table to active only" | DATA_PREP |
| Edit M code | "Add a calculated column in Power Query" | DATA_PREP |
| Apply changes | "Implement the changes from findings.md" | IMPLEMENT |
| Understand dashboard | "Summarize this dashboard and explain what it does" | SUMMARIZE |
| Document metrics | "Explain how the Total Sales metric works" | SUMMARIZE |
| Compare projects | "Merge my dev and prod projects" | MERGE |
| Check version | "What version of PBI Squire am I running?" | VERSION_CHECK |
| Anonymize data | "Set up data anonymization for this project" | SETUP_ANONYMIZATION |
| Mask PII | "Hide sensitive columns like names and emails" | SETUP_ANONYMIZATION |
| Design standards | "Set up design guidelines for my dashboard" | SETUP_DESIGN_STANDARDS (Developer) |
| Design review | "Review my dashboard against design guidelines" | QA_LOOP (Developer) |

---

## Pro Features

> **Note:** The following features require the Pro version of this plugin.
> If `developer-features.md` exists in this skill folder, those additional capabilities are available.

**Pro capabilities include:**
- **Design Standards** - Customizable design guidelines for consistent dashboard styling
- **Design Critique** - AI-powered review of dashboards against design standards
- **Template Harvesting** - Extract reusable visual templates from existing dashboards
- **UX Dashboard Review** - Expert analysis of published dashboards using Playwright
- **QA Loop** - Automated deploy → inspect → fix cycle for runtime error detection

**QA Loop prerequisites:**
The QA Loop (`/qa-loop-pbi-dashboard`) focuses on **runtime and deployment errors** (grey boxes, crashes, broken visuals), not syntax validation. Code must already be validated before running the QA loop, either:
- By the user manually
- Through the `/implement-deploy-test-pbi-project-file` workflow (which validates TMDL/PBIR syntax)

**Design Standards workflow:**
1. Bootstrap creates `.claude/powerbi-design-standards.md` in your project
2. Customize with your organization's brand colors, typography, and rules
3. Run `/qa-loop-pbi-dashboard --design-critique` to validate against standards
4. Agent scores design (1-5) and provides specific fix recommendations

See `developer-features.md` for full Pro documentation (Pro version only).
