---
name: architecture-docs
description: Use this skill when creating, updating, or maintaining ARCHITECTURE.md files, when users ask about "my architecture documentation" or "architecture", when generating presentations/slides/PowerPoint from architecture documentation, when generating diagrams from architecture documentation, when validating/checking/auditing architecture (including BIAN alignment, META layers, standards compliance), or when answering questions about documented components, data structures, integrations, security, performance, deployment, technology stack, or architectural decisions
---

# Architecture Documentation Skill

This skill provides comprehensive guidelines for creating and maintaining ARCHITECTURE.md files using the standardized template from ARCHITECTURE_DOCUMENTATION_GUIDE.md.

## When This Skill is Invoked

Automatically activate when:
- User asks to create architecture documentation
- User asks to update or edit ARCHITECTURE.md
- User mentions documenting system architecture
- User requests architecture review, audit, or analysis (triggers Design Drivers calculation prompt)
- User explicitly asks to "calculate design drivers" or "update design drivers"
- User asks about architecture documentation structure or best practices
- User edits Section 1 Executive Summary Key Metrics (triggers metric consistency check)
- User requests metric consistency check, verify metrics, or audit metrics
- **User asks informational questions about the documented architecture** (if ARCHITECTURE.md exists)
  - "What is our [authentication/scaling/data flow/etc.] approach?"
  - "How does [component/system/integration] work?"
  - "What technologies do we use for [purpose]?"
  - "Tell me about the architecture of [system]"
- **User asks to generate, create, or add diagrams to architecture documentation** (triggers Workflow 9)
  - "Generate my architecture diagrams"
  - "Create Mermaid diagrams from ARCHITECTURE.md"
  - "Add diagrams to my architecture"

### Query Pattern Triggers

This skill automatically activates when users ask questions about documented architecture, including:

**Reference Patterns**:
- "According to my architecture documentation..."
- "Based on the architecture..."
- "What does the architecture use/require/implement for..."
- "My architecture documentation shows/says..."
- "The architecture specifies/defines..."
- "Check/Validate/Verify the architecture [aspect]..."
- "Audit the [architecture component/pattern]..."

**Technical Query Keywords**:
- **Components**: "components", "services", "modules", "microservices", "systems"
- **Data**: "data structures", "data flow", "database", "schema", "models", "entities"
- **Integration**: "APIs", "integrations", "external systems", "endpoints", "interfaces"
- **Security**: "authentication", "authorization", "encryption", "security", "compliance"
- **Performance**: "scaling", "performance", "SLA", "capacity", "throughput", "latency"
- **Deployment**: "deployment", "cloud provider", "infrastructure", "environments", "regions"
- **Technology**: "tech stack", "languages", "frameworks", "tools", "libraries", "versions"
- **Decisions**: "why choose", "decision", "trade-offs", "alternatives", "ADR", "rationale"
- **Validation**: "check", "validate", "verify", "audit", "alignment", "BIAN", "META", "service domain", "layer", "standards", "compliance check"

**Multi-section Queries**:
- Questions requiring synthesis across multiple sections
- Cross-cutting concerns (e.g., "How does authentication work with external systems?")
- Implementation details spanning components, data, and deployment

---

## 🎯 AUTOMATIC WORKFLOW DETECTION

**IMPORTANT**: Immediately upon skill invocation, analyze the user's request to detect their intent.

### Detection Logic

Check the user's original message (before `/architecture-docs` was invoked) for these patterns:

#### Workflow 8: Presentation Generation
**Triggers:**
- Keywords: "generate", "create", "make" + "presentation", "slides", "PowerPoint", "pptx", "deck", "markdown presentation"
- Examples: "generate presentations", "create slides", "make PowerPoint", "generate presentation MD"
- Stakeholder mentions: "business presentation", "architecture slides", "compliance deck"
- Language: "presentación en español", "slides in English"

**Action when detected:**
1. Confirm: "I'll help you generate an architecture presentation Markdown file for Claude PowerPoint."
2. Jump directly to **Workflow 8, Step 1** (Stakeholder Type Selection)
3. Do NOT ask which workflow - proceed automatically

#### Workflow 9: Diagram Generation
**Triggers:**
- Keywords: "generate", "create", "add", "update", "make" + "diagram", "diagrams", "Mermaid diagram", "architecture diagram"
- Examples: "generate my architecture diagrams", "create diagrams from ARCHITECTURE.md", "add diagrams to my architecture"
- Section-specific: "generate diagrams for Section 4", "create data flow diagrams"
- Format mentions: "Mermaid diagrams", "visual diagrams", "architecture diagrams"

**Action when detected:**
1. Confirm: "I'll help you generate architecture diagrams."
2. Jump directly to **Workflow 9, Step 1** (Diagram Type Selection)
3. Do NOT ask which workflow - proceed automatically

#### Other Workflows
If the user's request matches other documented workflows (1-7, 9-10), follow their respective trigger patterns.

### If No Pattern Matches

If the user's request doesn't match any workflow triggers:
1. Acknowledge the skill invocation
2. Ask which workflow they want to use
3. Provide brief description of available workflows

---

## File Naming Convention

**IMPORTANT**: All architecture documents MUST be named **ARCHITECTURE.md**

- When creating a new architecture document, always use the filename: `ARCHITECTURE.md`
- When updating existing architecture, work with the file: `ARCHITECTURE.md`
- Do NOT use alternative names like: architecture.md, ARCH.md, system-architecture.md, etc.
- Each project/system should have ONE primary ARCHITECTURE.md file

### Location
- Place ARCHITECTURE.md in the root of the project or in a `/docs` directory
- For multi-project repositories, each project subdirectory gets its own ARCHITECTURE.md

## Working with ARCHITECTURE.md - Context Optimization

**IMPORTANT**: When working with ARCHITECTURE.md, optimize for context by loading sections incrementally, NOT the entire document at once.

### Context-Efficient Workflow

1. **Initial Assessment**
   - Read ONLY lines 1-50 of ARCHITECTURE.md to locate the **Document Index**
   - The Document Index provides exact line ranges for each section
   - Use the index to plan which sections need work
   - Identify which specific section(s) need work

2. **Section-Based Editing**
   - Use the Document Index to find exact line ranges for your target section
   - Use the Read tool with `offset` and `limit` parameters to load ONLY the relevant section
   - **Add context buffer**: Read ±10-20 lines beyond the section boundaries
   - Work on one section at a time
   - Example: Index shows Section 5 is lines 601-850, read lines 590-860 (with 10-line buffer)

3. **Incremental Updates**
   - Make edits to individual sections using the Edit tool
   - Avoid reading the full document unless absolutely necessary
   - For multi-section updates, work sequentially section by section
   - Update the Document Index line ranges if section size changes significantly

4. **Verification**
   - After edits, read ONLY the modified section to verify changes
   - Use Grep to search for specific content without loading full file
   - Update "Index Last Updated" date in the Document Index after major changes

### Using the Document Index

**Every ARCHITECTURE.md should include a Document Index** (before Section 1) with exact line ranges.

**Index Format Example:**
```markdown
## Document Index

**Quick Navigation:**
- [Section 1: Executive Summary](#1-executive-summary) → Lines 1-80
- [Section 2: System Overview](#2-system-overview) → Lines 81-150
- [Section 3: Architecture Principles](#3-architecture-principles) → Lines 151-350
- [Section 4: Architecture Layers](#4-architecture-layers) → Lines 351-600
- [Section 5: Component Details](#5-component-details) → Lines 601-850
- [Section 6: Data Flow Patterns](#6-data-flow-patterns) → Lines 851-1000
- [Section 7: Integration Points](#7-integration-points) → Lines 1001-1150
- [Section 8: Technology Stack](#8-technology-stack) → Lines 1151-1300
- [Section 9: Security Architecture](#9-security-architecture) → Lines 1301-1550
- [Section 10: Scalability & Performance](#10-scalability--performance) → Lines 1551-1750
- [Section 11: Operational Considerations](#11-operational-considerations) → Lines 1751-1950
- [Section 12: Architecture Decision Records (ADRs)](#12-architecture-decision-records-adrs) → Lines 1951-end

**Index Last Updated:** YYYY-MM-DD
```

**How to Use the Index:**

1. **Find the Index**: Read lines 1-50 to locate the Document Index
2. **Identify Target Section**: Check the index for exact line ranges
3. **Load with Context Buffer**: Read target section ± 10-20 lines for context preservation
4. **Example**:
   ```
   # To edit Section 5 (Component Details):
   # Index shows: Lines 601-850
   # Load with 10-line buffer:
   Read(file_path="ARCHITECTURE.md", offset=590, limit=270)
   # This reads lines 590-860
   ```

**Context Buffer Guidelines:**

| Edit Type | Buffer Size | Use Case |
|-----------|-------------|----------|
| **Minimal** | ±5-10 lines | Small edits (single paragraph, config value) |
| **Standard** | ±10-20 lines | Section edits (rewriting subsection, adding components) |
| **Extended** | ±20-50 lines | Cross-section edits (changes referencing adjacent sections) |

**Maintaining the Index:**

After significant edits:
1. Use `grep -n "^## [0-9]" ARCHITECTURE.md` to find actual section line numbers
2. Update the Document Index with current line ranges
3. Update "Index Last Updated" date
4. Verify index accuracy periodically (quarterly reviews)

**Note**: The Document Index is part of the ARCHITECTURE_DOCUMENTATION_GUIDE.md template and should be included in all new ARCHITECTURE.md files.

---

## Architecture Type Selection Workflow

**PURPOSE**: When creating a new ARCHITECTURE.md, prompt the user to select their architecture type. This determines the structure and content of Section 4 (Meta Architecture) and Section 5 (Component Details).

### When to Trigger

**Activate this workflow when:**
- ✅ User asks to create a NEW ARCHITECTURE.md document
- ✅ User explicitly requests to "change architecture type" or "select architecture type"
- ✅ User is updating an existing ARCHITECTURE.md and mentions changing from one architecture type to another

**Skip this workflow when:**
- ❌ Editing an existing ARCHITECTURE.md (type already selected)
- ❌ User is only updating specific sections unrelated to architecture type
- ❌ Document type is already clear from context

### Available Architecture Types

1. **Microservices Architecture (Recommended)** - Cloud-native distributed systems with independent services
2. **META Architecture** - 6-layer enterprise model (Channels → UX → Business Scenarios → Business → Domain → Core) with [BIAN V12.0](https://bian.org/servicelandscape-12-0-0/views/view_51891.html) alignment for Layer 5 Domain service domains
3. **3-Tier Architecture** - Classic web application pattern (Presentation → Application/Business Logic → Data)
4. **N-Layer Architecture** - Customizable patterns (DDD, Clean Architecture, Hexagonal)
5. **BIAN Architecture** - 5-layer BIAN-compliant model (Channels → BIAN Business Scenarios → BIAN Business Capabilities → BIAN Service Domains → Core Systems) with full [BIAN V12.0](https://bian.org/servicelandscape-12-0-0/views/view_51891.html) compliance across layers 2-4

**BIAN Standard for META**: BIAN V12.0 is the default and recommended version for META architecture. Use the [BIAN Service Landscape V12.0](https://bian.org/servicelandscape-12-0-0/views/view_51891.html) to identify and define service domains for Layer 5 (Domain).

### Workflow Steps

#### Step 1: Present Architecture Type Options

When creating a new ARCHITECTURE.md, present the user with architecture type selection:

```markdown
📐 **Architecture Type Selection**

Before creating your ARCHITECTURE.md, please select the architecture type that best describes your system:

**1. Microservices Architecture (Recommended)** (Cloud-Native Distributed)
   - Best for: Cloud-native systems, independently deployable services, modern applications
   - Components: API Gateway → Services → Data Stores → Event Bus
   - Complexity: High
   - Team Size: Large (10+)
   - Why Recommended: Industry standard for scalable, resilient modern applications

**2. META Architecture** (6-Layer Enterprise)
   - Best for: Large enterprise systems, financial services, complex integrations
   - Layers: Channels → UX → Business Scenarios → Integration → Domain → Core
   - Complexity: Very High
   - Team Size: Large (10+)

**3. 3-Tier Architecture** (Classic Web Application)
   - Best for: Web apps, REST APIs, standard CRUD systems
   - Tiers: Presentation → Application/Business Logic → Data
   - Complexity: Low
   - Team Size: Small-Medium (2-8)

**4. N-Layer Architecture** (Customizable Patterns)
   - Best for: DDD, Clean Architecture, Hexagonal Architecture
   - Patterns: 4-Layer DDD, 5-Layer Extended, Clean Architecture, Hexagonal
   - Complexity: Medium-High
   - Team Size: Medium (4-10)

**5. BIAN Architecture** (5-Layer BIAN-Compliant)
   - Best for: Banking systems requiring BIAN V12.0 certification, full BIAN compliance
   - Layers: Channels → BIAN Business Scenarios → BIAN Business Capabilities → BIAN Service Domains → Core Systems
   - Complexity: Very High
   - Team Size: Large (10+)
   - BIAN Compliance: Full BIAN V12.0 across layers 2-4

For detailed comparison and decision guidance, refer to: `templates/ARCHITECTURE_TYPE_SELECTOR.md`

**Which architecture type best describes your system? (1-5, or type name)**

Note: Option 1 (Microservices) is recommended for most modern cloud-native applications.
```

#### Step 2: Capture User Selection

Wait for user response (1, 2, 3, 4, or 5) or architecture type name.

**Valid inputs:**
- Numeric: `1`, `2`, `3`, `4`, `5`
- Type names: `Microservices`, `META`, `3-Tier`, `N-Layer`, `BIAN`
- Variations: `microservices`, `meta`, `three-tier`, `n-layer`, `bian`

**If user is unsure:**
- Offer to load `templates/ARCHITECTURE_TYPE_SELECTOR.md` decision guide
- Provide quick decision tree questions
- Default to **Microservices (Recommended)** for modern cloud-native systems
- Default to **META** if user has enterprise requirements and regulatory compliance needs
- Default to **3-Tier** if user wants maximum simplicity and minimal operational complexity

#### Step 3: Load Type-Specific Templates

Based on user selection, load the appropriate templates:

| Selection | Section 4 Template | Section 5 Template |
|-----------|-------------------|-------------------|
| Microservices (1) | `templates/SECTION_4_MICROSERVICES.md` | `templates/SECTION_5_MICROSERVICES.md` |
| META (2) | `templates/SECTION_4_META.md` | `templates/SECTION_5_META.md` |
| 3-Tier (3) | `templates/SECTION_4_3TIER.md` | `templates/SECTION_5_3TIER.md` |
| N-Layer (4) | `templates/SECTION_4_NLAYER_PATTERNS.md` | *(Use generic component template)* |
| BIAN (5) | `templates/SECTION_4_BIAN.md` | `templates/SECTION_5_BIAN.md` |

**Loading Process:**
1. Read the appropriate Section 4 template file
2. Read the appropriate Section 5 template file (if exists)
3. Extract template content (excluding metadata comments)
4. Prepare for insertion into ARCHITECTURE.md at correct section boundaries

#### Step 4: Add Architecture Type Metadata

When creating the ARCHITECTURE.md, add an HTML comment metadata tag at the beginning of Section 4 to track the selected architecture type:

```html
<!-- ARCHITECTURE_TYPE: META -->
```

**Valid metadata values:**
- `<!-- ARCHITECTURE_TYPE: META -->`
- `<!-- ARCHITECTURE_TYPE: 3-TIER -->`
- `<!-- ARCHITECTURE_TYPE: MICROSERVICES -->`
- `<!-- ARCHITECTURE_TYPE: N-LAYER -->`
- `<!-- ARCHITECTURE_TYPE: BIAN -->`

**Purpose:**
- Enables type detection for future edits
- Used by validation rules
- Used by Design Drivers calculation
- Used by architecture compliance skill

#### Step 5: Create ARCHITECTURE.md with Type-Specific Content

Create the complete ARCHITECTURE.md using:
- Standard sections 1-3 (same for all types)
- Type-specific Section 4 (from loaded template)
- Type-specific Section 5 (from loaded template)
- Standard sections 6-12 (same for all types)

**Include:**
- Document Index (placeholder line ranges)
- Architecture type metadata comment in Section 4
- All template content properly formatted
- Placeholder values for customization

#### Step 6: Prompt for ADR Generation (Optional)

**Trigger**: Immediately after Step 5 completes successfully (ARCHITECTURE.md created)

**Objective**: Offer to generate ADR files from Section 12 table

---

##### Step 6.0: Present ADR Generation Prompt

After ARCHITECTURE.md is successfully created, display the following prompt:

```
✅ ARCHITECTURE.md created successfully!

═══════════════════════════════════════════════════════════
📋 Architecture Decision Records (ADRs) Setup
═══════════════════════════════════════════════════════════

Section 12 of your ARCHITECTURE.md includes an ADR table with placeholder
entries. I can automatically generate these ADR files using the standard
ADR template.

Would you like me to generate the ADR files now?

Options:
1. [Yes - Generate ADRs] - Create all ADR files listed in Section 12
2. [Preview First] - Show me which ADRs will be created
3. [No Thanks] - I'll create them manually later
4. [Learn More] - Tell me about ADRs and the template

Recommended: Option 1 (Generate ADRs) - Saves time and ensures consistency
```

**Wait for user response**: `1`, `2`, `3`, `4`, or keywords like "yes", "preview", "no", "learn more"

---

##### Step 6.1: Handle User Selection

**If user selects Option 1 (Yes - Generate ADRs)**:
- Proceed to Step 6.2 (Locate Section 12)

**If user selects Option 2 (Preview First)**:
- Proceed to Step 6.2 (Locate Section 12)
- After Step 6.3 (Extract ADR List), show preview
- Re-prompt: "Proceed with generation? (yes/no)"
- If yes: Continue to Step 6.4
- If no: Skip to Step 6.8 (Complete)

**If user selects Option 3 (No Thanks)**:
- Skip to Step 6.8 (Complete with skip message)

**If user selects Option 4 (Learn More)**:
- Show ADR information:

```
Architecture Decision Records (ADRs) document important architectural choices,
providing context for future team members and explaining the "why" behind
technical decisions.

**What ADRs Document**:
- Technology selection rationale
- Architecture pattern choices
- Trade-offs and alternatives considered
- Consequences of decisions

**ADR Template Structure**:
- Context: Problem statement and requirements
- Decision: What was decided
- Rationale: Why this choice was made
- Consequences: Positive/negative outcomes
- Alternatives: Options that were considered but rejected

**Template Location**: skills/architecture-docs/adr/ADR-000-template.md
**Guide**: skills/architecture-docs/ADR_GUIDE.md

Would you like me to generate the ADR files now? (yes/no)
```

- Re-prompt with Options 1-3
- Wait for new response

---

##### Step 6.2: Locate Section 12

**Method 1: Use Document Index** (preferred)

```bash
# Read Document Index to find Section 12 line range
grep -A 20 "^## Document Index" ARCHITECTURE.md | grep "Section 12"
```

**Expected output**: `- [Section 12: Architecture Decision Records (ADRs)](#12-architecture-decision-records-adrs) → Lines 1750-1800`

**Parse line range**: Extract start line `1750` and end line `1800`

**Method 2: Fallback - Find by Header**

If Document Index method fails:

```bash
# Find Section 12 header line number
grep -n "^## 12\. Architecture Decision Records" ARCHITECTURE.md
```

**Expected output**: `1750:## 12. Architecture Decision Records (ADRs)`

**Parse line number**: Extract `1750`

**Read until**: Next section header or end of file (typically 50-100 lines)

**Error Handling**: If Section 12 not found:

```
⚠️  Section 12: Architecture Decision Records not found in ARCHITECTURE.md

This is unusual - all ARCHITECTURE.md files should include Section 12.

Would you like me to:
1. [Add Section 12] - Add the section with empty ADR table
2. [Skip ADR Generation] - Continue without generating ADRs
3. [Manual Review] - Let me check the document structure first

Recommended: Option 3 (Manual Review)
```

---

##### Step 6.3: Extract ADR List from Section 12

**Read Section 12 content**:

```bash
# Using Read tool with line range from Step 6.2
Read(file_path="ARCHITECTURE.md", offset=1750, limit=50)
```

**Parse ADR table rows**:

```bash
# Extract ADR table rows using grep
grep -E "^\| \[ADR-" ARCHITECTURE.md
```

**Expected output** (example):
```
| [ADR-001](adr/ADR-001.md) | [Title] | Accepted | YYYY-MM-DD | High/Medium/Low |
| [ADR-002](adr/ADR-002-database.md) | Database Choice | Proposed | 2024-01-20 | High |
```

**Parse each row**:

For each line matching the pattern `^\| \[ADR-`:
1. Extract ADR number: `001`, `002`, etc. (from `[ADR-001]`)
2. Extract file path from link: `adr/ADR-001.md` (from `(adr/ADR-001.md)`)
3. Extract slug (if present): `` (empty) or `database` (from `ADR-002-database.md`)
4. Extract title: Column 2 (e.g., `[Title]` or `Database Choice`)
5. Extract status: Column 3 (e.g., `Accepted`, `Proposed`)
6. Extract date: Column 4 (e.g., `YYYY-MM-DD` or `2024-01-20`)
7. Extract impact: Column 5 (e.g., `High/Medium/Low` or `High`)

**Regex Pattern** for parsing:
```regex
^\| \[ADR-(\d{3})\]\(adr\/ADR-\d{3}(-[a-z0-9-]+)?\.md\) \| (.+?) \| (.+?) \| (.+?) \| (.+?) \|
```

**Capture Groups**:
- Group 1: ADR number (e.g., `001`)
- Group 2: Optional slug (e.g., `-database` or empty)
- Group 3: Title
- Group 4: Status
- Group 5: Date
- Group 6: Impact

**Store in list**:
```python
# Example structure (pseudocode)
adrs = [
    {
        "number": "001",
        "slug": "",  # Empty means generate from title
        "title": "[Title]",
        "status": "Accepted",
        "date": "YYYY-MM-DD",
        "impact": "High/Medium/Low",
        "file_path": "adr/ADR-001.md"
    },
    {
        "number": "002",
        "slug": "database",
        "title": "Database Choice",
        "status": "Proposed",
        "date": "2024-01-20",
        "impact": "High",
        "file_path": "adr/ADR-002-database.md"
    }
]
```

**Error Handling**: If no ADRs found (empty table):

```
ℹ️  Section 12 ADR table is empty or contains only placeholder entries.

No ADRs found to generate. This is normal for newly created ARCHITECTURE.md files.

Would you like me to:
1. [Skip ADR Generation] - Continue without creating ADR files
2. [Learn About ADRs] - Understand when to create ADRs
3. [Add Sample ADR] - Create one example ADR to get started

Recommended: Option 1 (Skip) - Create ADRs as architectural decisions are made
```

**Validation**:
- Check for duplicate ADR numbers (same number appears multiple times)
- Check for malformed rows (wrong column count)

**If duplicates found**:
```
⚠️  Duplicate ADR numbers detected in Section 12 table

Duplicates found: ADR-002 (appears 2 times)

This will cause file overwrites. Please fix the table before generating ADRs.

Would you like me to:
1. [Show Duplicates] - Display the conflicting rows
2. [Auto-Renumber] - Automatically renumber ADRs sequentially
3. [Skip ADR Generation] - Fix manually and regenerate later

Recommended: Option 1 (Show Duplicates)
```

---

##### Step 6.3a: Preview ADRs (if user selected Option 2)

If user selected "Preview First" in Step 6.1:

**Display ADR list**:
```
Found {count} ADRs in Section 12 table:

1. ADR-001: [Title] (Status: Accepted, Impact: High/Medium/Low)
   → Will create: adr/ADR-001-untitled.md
   Note: Placeholder title - you can customize later

2. ADR-002: Database Choice (Status: Proposed, Impact: High)
   → Will create: adr/ADR-002-database.md

Proceed with generation? (yes/no)
```

**Wait for response**:
- If `yes` or `1`: Continue to Step 6.4
- If `no` or `2` or `skip`: Skip to Step 6.8

---

##### Step 6.4: Prepare ADR Generation

**Get ARCHITECTURE.md directory**:

```bash
# Extract directory path from ARCHITECTURE.md location
# If ARCHITECTURE.md is in current directory, use "."
# Otherwise, extract directory from file path
ARCH_DIR="."
```

**Define ADR directory path**:
```bash
ADR_DIR="${ARCH_DIR}/adr"
# Results in: "./adr" relative to ARCHITECTURE.md
```

**Create ADR directory if doesn't exist**:
```bash
mkdir -p adr
```

**Error Handling**: If directory creation fails:

```
❌ Error: Unable to create adr/ directory

Reason: [Error message from mkdir]

Possible causes:
- Insufficient file permissions
- Path does not exist
- Disk space full

Would you like me to:
1. [Retry with Different Path] - Try creating in user home directory
2. [Skip ADR Generation] - Continue without creating ADR files
3. [Manual Instructions] - Show me how to create manually

Recommended: Option 3 (Manual Instructions)
```

**Success message**:
```
Generating ADR files in ./adr/
```

---

##### Step 6.5: Load ADR Template

**Template path**: Resolve dynamically — `$PLUGIN_DIR/skills/architecture-docs/adr/ADR-000-template.md`

**Resolve plugin directory** (run once per session if not already known):
```bash
PLUGIN_DIR=$(find "$HOME" -maxdepth 10 -type d -name "solutions-architect-skills" ! -path "*/node_modules/*" 2>/dev/null | head -1)
echo "$PLUGIN_DIR"
```

**Load template** using the resolved path:
```
Read(file_path="$PLUGIN_DIR/skills/architecture-docs/adr/ADR-000-template.md")
```
(Replace `$PLUGIN_DIR` with the actual path returned by the Bash command above)

**Store template content** in memory for reuse across all ADR files

---

##### Step 6.6: Generate Each ADR File

For each ADR in the `adrs` list from Step 6.3:

**Step 6.6a: Generate File Path**

```python
# Determine slug (pseudocode)
if adr["slug"] == "":
    # Generate slug from title using Step 6.6b rules
    slug = generate_slug(adr["title"])
else:
    # Use slug from file path in table
    slug = adr["slug"]

# Construct file path
file_name = f"ADR-{adr['number']}-{slug}.md"
file_path = f"adr/{file_name}"
```

**Step 6.6b: Generate Slug from Title** (if needed)

**Slug Generation Rules**:
1. Convert title to lowercase
2. Replace spaces with hyphens
3. Remove special characters: `:?/*<>|"` (keep alphanumeric and hyphens)
4. Remove consecutive hyphens (replace `--` with `-`)
5. Trim leading/trailing hyphens
6. Limit to 50 characters
7. If title is `[Title]` or placeholder, use slug `untitled`

**Examples**:
- `Technology Stack Selection` → `technology-stack-selection`
- `REST vs. gRPC: Which API?` → `rest-vs-grpc-which-api`
- `PostgreSQL Database Choice` → `postgresql-database-choice`
- `[Title]` → `untitled`

**Step 6.6c: Check for Existing File**

```bash
# Check if file already exists
if [ -f "adr/ADR-${number}-*.md" ]; then
    # File exists - trigger conflict handling
fi
```

**If exists**:
```
⚠️  ADR file conflict detected

Existing file: adr/ADR-001-old-title.md
New file: adr/ADR-001-technology-stack.md

Would you like me to:
1. [Skip This ADR] - Don't overwrite, keep existing file
2. [Rename New ADR] - Create as ADR-{next_available} instead
3. [Overwrite] - Replace existing file (destructive)
4. [Review Existing] - Show me what's in the existing file first

Recommended: Option 4 (Review Existing)
```

**Conflict Resolution**:
- **Never auto-overwrite** existing ADRs
- Always require user confirmation
- Suggest next available ADR number if renaming

**Step 6.6d: Populate Template**

```python
# Copy template (pseudocode)
adr_content = template_content

# Replace placeholders
adr_content = adr_content.replace("ADR-XXX", f"ADR-{adr['number']}")
adr_content = adr_content.replace("[Short Decision Title]", adr["title"])

# Replace status line (keep full line format from template)
status_line_old = "**Status**: Proposed | Accepted | Deprecated | Superseded by ADR-XXX"
status_line_new = f"**Status**: {adr['status']}"
adr_content = adr_content.replace(status_line_old, status_line_new)

# Replace date
if adr["date"] == "YYYY-MM-DD" or adr["date"] == "[Date]":
    # Use current date
    current_date = datetime.now().strftime("%Y-%m-%d")
    adr_content = adr_content.replace("**Date**: YYYY-MM-DD", f"**Date**: {current_date}")
else:
    # Use date from table
    adr_content = adr_content.replace("**Date**: YYYY-MM-DD", f"**Date**: {adr['date']}")

# Replace authors (default to Architecture Team)
adr_content = adr_content.replace("**Authors**: [Author names or team name]", "**Authors**: Architecture Team")

# Replace related ADRs (empty by default)
adr_content = adr_content.replace("**Related**: [Links to related ADRs, e.g., ADR-001, ADR-005]", "**Related**: []")
```

**Special handling for placeholder titles**:
```python
# If title is placeholder, add TODO note
if adr["title"] in ["[Title]", "[title]", "Title"]:
    # Add note about placeholder
    note = "\n> **TODO**: This ADR was auto-generated with a placeholder title. Please update the title, filename, and content with your actual architectural decision.\n"
    # Insert after first header (after "# ADR-XXX: [Title]" line)
    lines = adr_content.split('\n')
    for i, line in enumerate(lines):
        if line.startswith('# ADR-'):
            lines.insert(i + 1, note)
            break
    adr_content = '\n'.join(lines)
```

**Step 6.6e: Write ADR File**

```bash
# Write the file using Write tool
Write(file_path=file_path, content=adr_content)
```

**Error Handling**: If write fails:
- Log error message
- Mark ADR as "failed" in tracking
- Continue with next ADR (don't abort entire batch)

**Step 6.6f: Report Progress**

```
✅ Created: adr/ADR-001-technology-stack.md
```

**Repeat** Steps 6.6a-6.6f for all ADRs in list

---

##### Step 6.7: Display Summary Report

After all ADRs processed:

**Count results**:
```python
total_adrs = len(adrs)
created_adrs = count(created successfully)
skipped_adrs = count(skipped due to conflicts)
failed_adrs = count(failed to write)
```

**Display summary**:
```
═══════════════════════════════════════════════════════════
✅ ADR Generation Complete
═══════════════════════════════════════════════════════════

Successfully generated {created_adrs} of {total_adrs} ADR files in ./adr/

Created ADRs:
- adr/ADR-001-technology-stack.md
- adr/ADR-002-database-choice.md
- adr/ADR-003-api-protocol.md

{If skipped_adrs > 0:}
Skipped ADRs:
- ADR-004 (file already exists: adr/ADR-004-old-title.md)

{If failed_adrs > 0:}
Failed ADRs:
- ADR-005 (write error: permission denied)

Next steps:
1. Review and customize each ADR file
2. Fill in Context, Decision, and Rationale sections
3. Update placeholder titles (marked with TODO)
4. Link related ADRs in the "Related" field
5. Update status to "Accepted" once reviewed and approved

ADR Template Guide: skills/architecture-docs/ADR_GUIDE.md
```

---

##### Step 6.8: Complete Workflow

**If user skipped ADR generation** (Option 3 in Step 6.1):

```
ℹ️  Skipped ADR generation - you can create ADR files manually later

To create ADRs manually:
1. Update Section 12 table in ARCHITECTURE.md with your ADRs
2. Copy template: cp skills/architecture-docs/adr/ADR-000-template.md adr/ADR-001-your-title.md
3. Customize the ADR content

For guidance, see: skills/architecture-docs/ADR_GUIDE.md
```

**Return to main workflow**: Architecture Type Selection Workflow completes successfully

---

**End of Step 6**

### Detecting Existing Architecture Type

When editing an existing ARCHITECTURE.md, detect the architecture type:

**Detection Method 1: Metadata Comment**
```bash
grep -n "<!-- ARCHITECTURE_TYPE:" ARCHITECTURE.md
```

If found, extract the type from the comment.

**Detection Method 2: Section 4 Header Analysis**

If no metadata comment, infer from Section 4 headers:

```bash
# Check for BIAN indicators (check FIRST - most specific)
grep -E "(Layer 2: BIAN Business Scenarios|Layer 4: BIAN Service Domains)" ARCHITECTURE.md

# Check for META indicators
grep -E "(Layer 1: Channels|Layer 5: Domain|Layer 6: Core)" ARCHITECTURE.md

# Check for 3-Tier indicators
grep -E "(Tier 1: Presentation|Tier 3: Data)" ARCHITECTURE.md

# Check for Microservices indicators
grep -E "(API Gateway|Service Mesh|Microservices Catalog)" ARCHITECTURE.md

# Check for N-Layer indicators
grep -E "(Clean Architecture|Hexagonal|Ports & Adapters)" ARCHITECTURE.md
```

**Inference Rules (in order of specificity):**
- Contains "Layer 2: BIAN Business Scenarios" OR "Layer 4: BIAN Service Domains" → **BIAN**
- Contains "Layer 1: Channels" AND "Layer 5: Domain" AND "Layer 6: Core" → **META**
- Contains "Tier 1: Presentation" OR "Tier 3: Data" → **3-Tier**
- Contains "API Gateway" AND "Service Mesh" → **Microservices**
- Contains "Clean Architecture" OR "Hexagonal" → **N-Layer**
- Cannot determine → Ask user or default to **Microservices (Recommended)**

### Changing Architecture Type (Existing Document)

If user requests to change architecture type of an existing ARCHITECTURE.md:

**Warning Steps:**
1. **Detect current type** using detection methods above
2. **Warn user** about potential data loss:
   ```
   ⚠️  **Architecture Type Change Warning**

   Current type: [DETECTED_TYPE]
   Requested type: [NEW_TYPE]

   Changing architecture type will:
   - Replace Section 4 (Meta Architecture) with new structure
   - Replace Section 5 (Component Details) with new organization
   - Require manual component remapping

   **Recommendation**: Review and backup current Sections 4 & 5 before proceeding.

   Continue with architecture type change? (yes/no)
   ```

3. **If user confirms:**
   - Load new type templates
   - Replace Section 4 content
   - Replace Section 5 content
   - Update metadata comment
   - Update Document Index
   - Report changes to user

4. **If user declines:**
   - Cancel operation
   - Suggest manual editing approach

### Type-Specific Validation

After selecting or detecting architecture type, apply type-specific validation rules:

**BIAN Architecture:**
- ✅ Must have all 5 layers (Channels, BIAN Business Scenarios, BIAN Business Capabilities, BIAN Service Domains, Core Systems)
- ✅ Layer 2 must map to BIAN Business Areas (5 areas)
- ✅ Layer 3 must map to BIAN Business Domains (30+ domains)
- ✅ Layer 4 must implement BIAN Service Domains from [BIAN V12.0](https://bian.org/servicelandscape-12-0-0/views/view_51891.html)
- ✅ All Layer 4 service domain names validated against official [BIAN Service Landscape V12.0](https://bian.org/servicelandscape-12-0-0/views/view_51891.html)
- ✅ Layer 4 components must include complete BIAN metadata (Official Name, BIAN ID, Version, Business Domain, Business Area, URL)
- ✅ Control records documented per BIAN specification for all Layer 4 service domains
- ✅ All mandatory BIAN service operations implemented (Initiate, Update, Retrieve, Control)
- ✅ Behavior qualifiers documented per BIAN spec
- ✅ Functional patterns documented per BIAN spec (Managed Object, Tracked Object, etc.)
- ✅ Full BIAN V12.0 compliance level documented
- ✅ BIAN hierarchy traceability: Service Domain → Business Domain → Business Area
- ✅ Layers documented in correct order

**META Architecture:**
- ✅ Must have all 6 layers (Channels, UX, Business Scenarios, Business, Domain, Core)
- ✅ Layer 5 must include BIAN alignment section
- ✅ Verify BIAN V12.0 is documented as the default version
- ✅ Validate BIAN service domain **names (Capabilities)** against the official [BIAN Service Landscape V12.0](https://bian.org/servicelandscape-12-0-0/views/view_51891.html)
- ✅ Ensure BIAN IDs (SD-XXX) are used for internal document tracking only
- ✅ Validate Layer 5 (Domain) components include BIAN alignment subsection with official URLs
- ✅ Layers documented in correct order

**3-Tier Architecture:**
- ✅ Must have all 3 tiers (Presentation, Application, Data)
- ✅ No direct database access from Presentation tier
- ✅ Application tier should be stateless

**Microservices Architecture:**
- ✅ Must document API Gateway and Service Mesh (or justify omission)
- ✅ Database-per-service pattern followed
- ✅ Event bus and topics documented
- ✅ Circuit breakers configured

**N-Layer Architecture:**
- ✅ Must specify which pattern (4-Layer, 5-Layer, Clean, Hexagonal)
- ✅ Dependency direction documented
- ✅ Core/domain layer is framework-free (if applicable)

**For detailed validation rules**, see: `VALIDATIONS.md` § Type-Aware Validation

### Best Practices

**DO:**
- ✅ Always prompt for architecture type when creating new ARCHITECTURE.md
- ✅ Add metadata comment to track architecture type
- ✅ Load appropriate templates based on selection
- ✅ Warn before changing architecture type of existing document
- ✅ Apply type-specific validation rules

**DON'T:**
- ❌ Assume architecture type without asking
- ❌ Mix templates from different architecture types
- ❌ Change architecture type without user confirmation
- ❌ Skip metadata comment (makes future edits harder)

### Example Workflow

**User:** "Create architecture documentation for my microservices system"

**Assistant:**
1. Detects new ARCHITECTURE.md creation
2. Presents architecture type options (1-4)
3. User selects "3" (Microservices)
4. Loads `templates/SECTION_4_MICROSERVICES.md` and `templates/SECTION_5_MICROSERVICES.md`
5. Creates ARCHITECTURE.md with:
   - Standard Sections 1-3
   - Microservices Section 4 (API Gateway, Service Mesh, Services, Event Bus)
   - Microservices Section 5 (Service catalog format)
   - Standard Sections 6-12
   - Metadata comment: `<!-- ARCHITECTURE_TYPE: MICROSERVICES -->`
6. Reports completion with architecture type confirmation

---

## Automatic Index Updates

**CRITICAL**: After ANY edit that significantly changes section line numbers (>10 lines), automatically update the Document Index.

### When to Update

**Update if:**
- ✅ Added/removed content shifting section boundaries (>10 lines)
- ✅ Modified section headers or structure
- ✅ User requests: "update the index"

**Skip if:**
- ❌ Minor edits (<10 lines)
- ❌ Only metadata changes
- ❌ Typo fixes

### Workflow Overview

**Quick Steps:**
1. **Detect**: Run `grep -n "^## [0-9]" ARCHITECTURE.md` to find section boundaries
2. **Calculate**: Parse output to determine line ranges (Section_Start to Next_Section_Start - 1)
3. **Update**: Edit Document Index (typically lines 5-21) with new ranges
4. **Timestamp**: Update "Index Last Updated" to current date
5. **Report**: Inform user which sections changed

**Example:**
```bash
# After adding content to Section 8, it grew from 906-980 to 912-996
grep -n "^## [0-9]" ARCHITECTURE.md
# Parse: Section 8 now at line 912, Section 9 at 998
# Calculate: Section 8 = 912-997, Section 9 = 998-1243
# Update index and report to user
```

### Detailed Algorithm

For complete line range calculation algorithm, step-by-step examples, verification checklist, and edge cases:
→ **METRIC_CALCULATIONS.md** § Automatic Index Updates

### Best Practices

**DO:**
- ✅ Update after significant edits
- ✅ Use grep for accuracy (don't guess)
- ✅ Update timestamp
- ✅ Report changes to user

**DON'T:**
- ❌ Update for tiny changes
- ❌ Skip timestamp update
- ❌ Change index format

---

## Metric Consistency Detection & Management

**CRITICAL**: The Executive Summary (Section 1, Key Metrics subsection around lines 31-38) contains performance metrics that serve as the **Source of Truth** for system capacity, throughput, latency, and availability targets. These metrics are often duplicated across multiple sections (especially Section 6: Data Flow, Section 7: Integration Points, and Section 10: Scalability & Performance).

**Problem**: When metrics are updated in the Executive Summary, duplicates in other sections can become stale, creating inconsistencies.

**Solution**: This skill provides automatic metric consistency detection and review-based synchronization.

### When to Trigger Metric Audit

**Automatic Trigger:**
- ✅ After editing Section 1 Executive Summary Key Metrics (typically lines 31-38)
- ✅ When user updates any metric values in the Key Metrics subsection
- ✅ After Edit tool completes on lines in Section 1 that contain metrics (TPS, latency, SLA, etc.)

**Manual Trigger:**
- User explicitly requests: "Check metric consistency", "Verify metrics", "Audit metrics", "Sync metrics"
- User updates Section 10 (Scalability & Performance) and asks to verify consistency with Exec Summary
- During quarterly documentation review or audit

**Do NOT Trigger:**
- ❌ Minor edits outside Key Metrics subsection (e.g., updating document version or date)
- ❌ Edits to narrative text that don't change numeric metric values
- ❌ Changes to other sections that don't involve performance metrics

### Metric Audit Workflow

**Step 1: Extract Executive Summary Metrics**

Read ONLY lines 30-40 (Key Metrics section) using context-efficient approach:
```bash
Read(file_path="ARCHITECTURE.md", offset=30, limit=10)
```

Parse and extract all metrics with their values using these patterns:

| Metric Type | Pattern | Example Match |
|-------------|---------|---------------|
| **Average Read TPS** | `Average\s+Read\s+TPS:\s*(\d{1,3}(?:,\d{3})*)\s*transactions/second` | "Average Read TPS: 1,500 transactions/second" |
| **Peak Read TPS** | `Peak\s+Read\s+TPS:\s*(\d{1,3}(?:,\d{3})*)\s*transactions/second` | "Peak Read TPS: 3,000 transactions/second" |
| **Average Processing TPS** | `Average\s+Processing\s+TPS:\s*(\d{1,3}(?:,\d{3})*)\s*transactions/second` | "Average Processing TPS: 450 transactions/second" |
| **Peak Processing TPS** | `Peak\s+Processing\s+TPS:\s*(\d{1,3}(?:,\d{3})*)\s*transactions/second` | "Peak Processing TPS: 1,000 transactions/second" |
| **Average Write TPS** | `Average\s+Write\s+TPS:\s*(\d{1,3}(?:,\d{3})*)\s*transactions/second` | "Average Write TPS: 300 transactions/second" |
| **Peak Write TPS** | `Peak\s+Write\s+TPS:\s*(\d{1,3}(?:,\d{3})*)\s*transactions/second` | "Peak Write TPS: 800 transactions/second" |
| **Measurement Period** | `Measurement\s+Period:\s*(.+)` | "Measurement Period: Average over last 30 days" |
| **Percentile Latency** | `p(\d{2})\s*<\s*(\d+)ms` | "p95 < 100ms", "p99 < 200ms" |
| **Availability SLA** | `(\d{2,3}\.\d+)\s*%` | "99.99%", "99.9%" |
| **Concurrent Jobs** | `(\d{1,3}(?:,\d{3})*)\+?\s*concurrent` | "10,000+ concurrent", "5000 concurrent" |
| **Jobs per Hour** | `\[(\d{1,3}(?:,\d{3})*)\s*jobs?/hour\]` | "[1,620,000 jobs/hour]" |

Build a metrics registry from parsed data:
```
METRICS_REGISTRY = [
  {name: "Read TPS", value: 1500, unit: "TPS", category: "Read", stat_type: "Average", line: 32, measurement_period: "Average over last 30 days in production"},
  {name: "Read TPS", value: 3000, unit: "TPS", category: "Read", stat_type: "Peak", line: 33, measurement_period: "Peak observed during Black Friday 2024"},
  {name: "Processing TPS", value: 450, unit: "TPS", category: "Processing", stat_type: "Average", line: 35, measurement_period: "Average over last quarter"},
  {name: "Processing TPS", value: 1000, unit: "TPS", category: "Processing", stat_type: "Peak", line: 36, measurement_period: "Peak during end-of-month batch processing"},
  {name: "Write TPS", value: 300, unit: "TPS", category: "Write", stat_type: "Average", line: 38, measurement_period: "Average over last month"},
  {name: "Write TPS", value: 800, unit: "TPS", category: "Write", stat_type: "Peak", line: 39, measurement_period: "Peak during data migration events"},
  {name: "System Availability", value: 99.99, unit: "%", line: 42},
  {name: "Latency p95", value: 100, unit: "ms", line: 43},
  {name: "Latency p99", value: 200, unit: "ms", line: 43},
  {name: "Concurrent Jobs", value: 10000, unit: "jobs", line: 44}
]
```

**Step 2: Scan Document for Metric References**

Use Grep to find all occurrences of each metric value (context-efficient, no full file load):

```bash
# For each metric in registry, search with appropriate pattern (new standardized TPS format)
Grep(pattern="Average\s+Read\s+TPS:\s*1,?500", path="ARCHITECTURE.md", output_mode="files_with_matches")
Grep(pattern="Peak\s+Read\s+TPS:\s*3,?000", path="ARCHITECTURE.md", output_mode="files_with_matches")
Grep(pattern="Average\s+Processing\s+TPS:\s*450", path="ARCHITECTURE.md", output_mode="files_with_matches")
Grep(pattern="Peak\s+Processing\s+TPS:\s*1,?000", path="ARCHITECTURE.md", output_mode="files_with_matches")
Grep(pattern="Average\s+Write\s+TPS:\s*300", path="ARCHITECTURE.md", output_mode="files_with_matches")
Grep(pattern="Peak\s+Write\s+TPS:\s*800", path="ARCHITECTURE.md", output_mode="files_with_matches")
Grep(pattern="99\.99%", path="ARCHITECTURE.md", output_mode="files_with_matches")
Grep(pattern="p95.*100ms", path="ARCHITECTURE.md", output_mode="files_with_matches")
Grep(pattern="p99.*200ms", path="ARCHITECTURE.md", output_mode="files_with_matches")
Grep(pattern="10,?000.*concurrent", path="ARCHITECTURE.md", output_mode="files_with_matches")
```

**Note**: Use `-n` flag with grep bash command if you need line numbers, or use Grep tool with `output_mode="content"`.

**Step 3: Load Context for Each Match**

For each grep result, read ±5 lines for context to determine if it's a duplicate or coincidental match:
```bash
# If match found at line 787
Read(file_path="ARCHITECTURE.md", offset=782, limit=10)
# Reads lines 782-792 for context
```

**Step 4: Classify Findings**

Categorize each finding into one of four categories:

- **✓ Exact Match**: Same value, same metric concept, already consistent with Executive Summary
- **⚠️ Mismatch**: Same metric concept but different value (ACTION REQUIRED - user must decide)
- **ℹ️ Derived Value**: Related but transformed metric (e.g., 450 TPS = 1,620,000 jobs/hour via 450 × 3600)
- **? Ambiguous**: Number matches but context is unclear (flag for manual review)

**Classification Logic**:
1. Check if line is in Section 1 (skip, that's the source of truth)
2. Read context (±5 lines) to understand what the metric represents
3. Compare metric concept (is this the same "Job Creation Capacity" or different metric?)
4. Determine relationship (exact duplicate, derived value, or unrelated)

**Step 5: Generate Consistency Report**

Present findings in structured, user-friendly format:

```markdown
═══════════════════════════════════════════════════════════
   METRIC CONSISTENCY AUDIT REPORT
═══════════════════════════════════════════════════════════
Source of Truth: Section 1 (Executive Summary) Lines 31-38
Total Metrics Audited: 10
═══════════════════════════════════════════════════════════

## ✓ Exact Matches (Consistent)

| Metric | Exec Value | Location | Context | Status |
|--------|-----------|----------|---------|--------|
| Job Latency (p95) | < 100ms | Line 787 | Section 6: Data Flow Performance | ✓ Match |
| Job Latency (p99) | < 200ms | Line 787 | Section 6: Data Flow Performance | ✓ Match |
| Job Latency (p95) | < 100ms | Line 1345 | Section 10: Performance Table | ✓ Match |
| Job Latency (p99) | < 200ms | Line 1345 | Section 10: Performance Table | ✓ Match |

## ⚠️ Mismatches Found (ACTION REQUIRED)

| Metric | Exec Value | Location | Current Value | Difference |
|--------|-----------|----------|---------------|------------|
| Peak Job Creation | 1,000 TPS | Line 1367 | 500 TPS | -500 TPS |

## ℹ️ Derived Values (Informational)

| Metric | Exec Value | Location | Derived Value | Relationship |
|--------|-----------|----------|---------------|--------------|
| Job Creation | 450 TPS | Line 32 | 1,620,000 jobs/hour | 450 × 3600 |

═══════════════════════════════════════════════════════════
SUMMARY: ✓ 4 Matches | ⚠️ 1 Mismatch | ℹ️ 1 Derived
═══════════════════════════════════════════════════════════
```

**Step 6: Await User Decision**

If mismatches found, present options to user:
```
I found 1 metric mismatch. What would you like to do?

1. [Review Details] - Show detailed context for the mismatch
2. [Update Exec Summary] - Update Section 1 to match line 1367 (500 TPS)
3. [Update Document] - Update line 1367 to match Exec Summary (1,000 TPS)
4. [Preview Changes] - Show exactly what will change before applying
5. [Manual Review] - I'll review and decide later
6. [Ignore] - Keep as-is (intentional difference)
```

**Step 7: Apply Approved Changes**

If user approves updates:
1. Show preview of all changes (exact before/after for each location)
2. Confirm user approval
3. Use Edit tool to update each location sequentially
4. Update Document Index if line numbers shift significantly (>10 lines)
5. Report completion with summary of all changes made

### Common Metric Locations

**Known Duplicate Locations** (for typical architecture documents):

| Metric | Executive Summary | Common Duplicate Locations |
|--------|------------------|---------------------------|
| Job Creation Capacity (450 TPS) | Line ~32 | Section 10 Throughput Table (~line 1367) |
| Job Execution Capacity (500 TPS) | Line ~33 | Section 10 Throughput Table (~line 1368), Section 6 Data Flow (~line 787) |
| Peak Capacity (1,000/2,000 TPS) | Line ~34 | Section 10 Throughput Table (~lines 1367-1368) |
| Initial Load (150/350 TPS) | Line ~35 | Section 10 Capacity Planning (~lines 1378-1379) |
| System Availability (99.99%) | Line ~36 | Section 2 Use Case metrics (~line 115), Section 7 Integration SLA Table (~lines 829-832) |
| Latency Targets (p95/p99) | Line ~37 | Section 6 Data Flow Performance (~line 787), Section 10 Performance Table (~line 1345) |
| Concurrent Jobs (10,000+) | Line ~38 | Less commonly duplicated (usually only in Exec Summary) |

**Note**: Line numbers are approximate and vary by document. Use grep to find exact locations.

### Metric Update Best Practices

**DO:**
- ✅ Always scan full document after Exec Summary metric changes
- ✅ Present findings before making any changes (review-only first)
- ✅ Explain context for each mismatch (which section, what table/paragraph)
- ✅ Offer batch update option for multiple occurrences of same metric
- ✅ Show preview with exact before/after text for transparency
- ✅ Update Document Index if edits shift section boundaries

**DON'T:**
- ❌ Auto-update without user approval (always review-only first)
- ❌ Change metrics in unrelated sections without confirming intent
- ❌ Ignore ambiguous matches (always flag for review)
- ❌ Forget to check derived values (e.g., jobs/hour conversions from TPS)
- ❌ Skip context loading (always read ±5 lines to understand match)

### Error Handling & Edge Cases

**1. Multiple Conflicting Values**

Scenario: Same metric has different values in multiple sections (not just Exec Summary vs. one location)

Example:
- Exec Summary (Line 32): 450 TPS
- Section 6 (Line 787): 500 TPS
- Section 10 (Line 1367): 400 TPS

**Handling:**
```
⚠️ CONFLICT DETECTED: Job Execution Capacity has 3 different values

| Location | Value | Context |
|----------|-------|---------|
| Line 32 (Exec Summary) | 450 TPS | Design Capacity |
| Line 787 (Section 6) | 500 TPS | Data Flow Performance |
| Line 1367 (Section 10) | 400 TPS | Throughput Table |

Which value is correct?
1. Use Exec Summary (450 TPS) as source of truth
2. Use most common value (if clear majority)
3. Manual review - I need to investigate further

Please clarify which value should be used across all locations.
```

**2. Derived Value Calculation Errors**

Scenario: Derived value doesn't match expected conversion

Example:
- Exec Summary: 450 TPS
- Expected derived: 1,620,000 jobs/hour (450 × 3600)
- Found in doc: 1,650,000 jobs/hour

**Handling:**
```
ℹ️ Derived Value Mismatch Detected:

Metric: Job Creation Capacity
Base Value: 450 TPS (Line 32)
Expected Derived: 1,620,000 jobs/hour (450 × 3600)
Actual Found: 1,650,000 jobs/hour (Line 32)
Difference: +30,000 jobs/hour

Possible causes:
- Rounding or overhead factor applied
- Different calculation method
- Stale value from previous metric update

Would you like to:
1. Update to calculated value (1,620,000)
2. Keep current value (1,650,000) - might be intentional overhead
3. Investigate further
```

**3. Ambiguous Metric Context**

Scenario: Number matches but unclear if it's the same metric

Example:
- Exec Summary: Peak Capacity 1,000 TPS
- Found at line 1367: "System limit 1,000 TPS (theoretical max)"
- Found at line 1372: "Target throughput 1,000 TPS (steady state)"
- Found at line 1389: "1000 TPS limit"

**Handling:**
```
? AMBIGUOUS MATCH: Value "1,000 TPS" found in multiple contexts

Exec Summary Definition (Line 34):
"Peak Capacity: 1,000 TPS job creation, 2,000 TPS job execution (system limits)"

Found Matches:
1. Line 1367: "System limit 1,000 TPS"
   Context: Throughput table - appears to be same metric ✓

2. Line 1372: "1000 TPS (theoretical max)"
   Context: Throughput calculation - likely same metric ✓

3. Line 1389: "1000 TPS limit"
   Context: Capacity planning - likely same metric ✓

All 3 appear to reference Peak Capacity. Treat as exact matches? [Yes/No/Review Each]
```

**4. No Matches Found**

Scenario: Metric exists in Exec Summary but never referenced elsewhere

Example:
- Exec Summary: "Concurrent Jobs: Support for 10,000+ concurrent scheduled jobs"
- Grep search: No matches found in rest of document

**Handling:**
```
ℹ️ INFORMATIONAL: Metric appears only in Executive Summary

Metric: Concurrent Jobs (10,000+)
Location: Line 38 (Section 1)
Document Coverage: Not referenced in other sections

This is not necessarily an error - some high-level metrics may only appear in Exec Summary.

Recommendation:
- If this metric is important to architecture, consider adding to Section 10 (Scalability & Performance)
- If it's a summary-only metric, no action needed

Would you like to:
1. Add this metric to Section 10
2. Keep as-is (Exec Summary only)
3. Ignore this finding
```

**5. Too Many Grep Results**

Scenario: Common number matches hundreds of unrelated contexts

Example: Searching for "100" also matches "100 pods", "section 100", "version 1.00"

**Handling:**
- Use context-aware grep patterns to reduce false positives:
  - Bad: `grep -n "100" ARCHITECTURE.md` (too broad)
  - Good: `grep -n "p95.*100ms" ARCHITECTURE.md` (context-aware)
  - Better: `grep -n "p95\s*<\s*100ms" ARCHITECTURE.md` (exact pattern)

- If still >20 matches after context-aware patterns:
  1. Load only first 10 matches
  2. Present summary: "Found 47 matches for '100' - showing first 10 most relevant"
  3. Ask user if they want to see all or refine search pattern

### Example: Complete Metric Update Workflow

**Scenario:** User updates Job Execution Capacity from 500 TPS to 700 TPS

**Step-by-Step Execution:**

1. **User Action**:
   ```
   User: "Update the job execution capacity to 700 TPS in the Executive Summary"
   ```

2. **Claude Applies Edit**:
   ```
   [Uses Edit tool to change line 33: "500 TPS" → "700 TPS"]
   ✓ Updated line 33: Design Capacity - Job Execution: 700 TPS

   Now checking for metric consistency across the document...
   ```

3. **Automatic Audit Triggered**:
   ```
   [Metric audit automatically triggered because Section 1 Key Metrics was edited]

   Scanning document for references to old value (500 TPS)...
   [Runs: Grep(pattern="500\s*TPS", path="ARCHITECTURE.md")]
   [Found at: Line 787, Line 1368]
   ```

4. **Load Context for Each Match**:
   ```
   [Reads context around each match]
   Read(file_path="ARCHITECTURE.md", offset=782, limit=10)  # Lines 782-792 for Line 787
   Read(file_path="ARCHITECTURE.md", offset=1363, limit=10) # Lines 1363-1373 for Line 1368
   ```

5. **Present Findings**:
   ```
   ═══════════════════════════════════════════════════════════
   METRIC CONSISTENCY AUDIT REPORT
   ═══════════════════════════════════════════════════════════
   Triggered by: Executive Summary edit (Line 33)
   Metric Changed: Job Execution Capacity (500 TPS → 700 TPS)
   ═══════════════════════════════════════════════════════════

   ## ⚠️ Outdated References Found

   | Location | Section | Current Value | Should Be | Context |
   |----------|---------|---------------|-----------|---------|
   | Line 787 | Section 6: Data Flow Patterns | 500 TPS capacity | 700 TPS | "Performance: p50 = 40ms, p95 = 100ms, p99 = 200ms \| 500 TPS capacity" |
   | Line 1368 | Section 10: Scalability & Performance | 500 | 700 | Throughput table: "Job Execution (TPS) \| 500 \| 1000 \| 2000" |

   ═══════════════════════════════════════════════════════════
   SUMMARY: Found 2 locations with outdated metric value
   ═══════════════════════════════════════════════════════════

   Would you like me to update both locations to 700 TPS?

   Options:
   1. [Yes - Update All] - Apply changes to both locations
   2. [Preview Changes] - Show me exactly what will change
   3. [Manual Selection] - Let me choose individually
   4. [No Thanks] - I'll handle this manually
   ```

6. **User Requests Preview**:
   ```
   User: "Preview changes first"
   ```

7. **Show Preview**:
   ```
   Here's a preview of the changes I'll make:

   ════════════════════════════════════════════════════════════
   CHANGE 1 of 2: Line 787 (Section 6: Data Flow Patterns)
   ════════════════════════════════════════════════════════════

   Current:
   **Performance**: p50 = 40ms, p95 = 100ms, p99 = 200ms | 500 TPS capacity

   Will become:
   **Performance**: p50 = 40ms, p95 = 100ms, p99 = 200ms | 700 TPS capacity

   ════════════════════════════════════════════════════════════
   CHANGE 2 of 2: Line 1368 (Section 10: Throughput Table)
   ════════════════════════════════════════════════════════════

   Current:
   | Job Execution (TPS) | 500 | 1000 | 2000 | Downstream domain services |

   Will become:
   | Job Execution (TPS) | 700 | 1000 | 2000 | Downstream domain services |

   ════════════════════════════════════════════════════════════

   Proceed with both changes? [Yes/No/Choose Individually]
   ```

8. **User Approves**:
   ```
   User: "Yes"
   ```

9. **Apply Changes**:
   ```
   Applying metric consistency updates...

   [Uses Edit tool on line 787]
   ✓ Updated line 787: "500 TPS capacity" → "700 TPS capacity"

   [Uses Edit tool on line 1368]
   ✓ Updated line 1368: Throughput table "500" → "700"

   Checking if Document Index needs update...
   [Runs: grep -n "^## [0-9]" ARCHITECTURE.md to check section positions]
   [Determines: No significant line shifts, index still valid]

   ✓ Document Index is still accurate (no update needed)
   ```

10. **Completion Report**:
    ```
    ═══════════════════════════════════════════════════════════
    METRIC CONSISTENCY UPDATE COMPLETE
    ═══════════════════════════════════════════════════════════

    Summary of Changes:
    ✓ Executive Summary (Line 33): 500 TPS → 700 TPS
    ✓ Section 6 Data Flow (Line 787): 500 TPS → 700 TPS
    ✓ Section 10 Throughput Table (Line 1368): 500 → 700

    Total Locations Updated: 3
    Document Index: No update needed
    All references to Job Execution Capacity are now consistent.

    ═══════════════════════════════════════════════════════════
    ```

---

## Design Drivers Impact Metrics Calculation

**PURPOSE**: Automatically calculate and maintain Design Drivers impact metrics (Value Delivery, Scale, Impacts) based on data from existing architecture sections. Design Drivers provide a quantifiable assessment of the architecture's business value, customer reach, and implementation complexity.

### When to Trigger Design Drivers Calculation

**Automatic Prompt:**
- ✅ User requests "architecture review" or "audit" (prompt to calculate)
- ✅ User explicitly asks to "calculate design drivers" or "update design drivers"
- ✅ After significant architecture changes during review process

**Manual Trigger:**
- User explicitly requests: "Calculate design drivers", "Update Section 2.2.1", "Assess design impact"
- During quarterly documentation review or architecture validation
- When preparing architecture presentations or business justifications

**Do NOT Trigger:**
- ❌ Minor edits unrelated to architecture (typos, formatting, dates)
- ❌ Changes to sections not used in design drivers calculation (Sections 3, 4, 9, 11, 12)
- ❌ User explicitly declines when prompted during review

### Design Drivers Framework

The Design Drivers subsection (Section 2.2.1) measures three impact dimensions:

#### 1. Value Delivery
**Definition**: Effectiveness of change in customer experience
**Threshold**: >50% = High Impact, ≤50% = Low Impact
**Data Source**: Section 1 Executive Summary - Business Value bullets
**Extraction Pattern**: `(\d{1,3})%\s*(reduction|improvement|increase|efficiency|optimization|cost\s*savings?|faster|time\s*savings?)`

**Calculation Logic**:
1. Read Section 1 Executive Summary (lines 25-55)
2. Find all percentage metrics in Business Value subsection
3. Extract maximum percentage value
4. If ANY metric > 50% → HIGH Impact
5. If ALL metrics ≤ 50% → LOW Impact
6. Generate justification citing specific percentage and line number

**Example**:
- Input: "70% cost reduction" (Section 1, line 52)
- Output: HIGH Impact - "System delivers 70% cost reduction (Section 1, line 52)"

#### 2. Scale
**Definition**: Estimated number of customers/transactions impacted
**Threshold**: >100K = High Impact, ≤100K = Low Impact
**Data Source**: Section 2.3 Use Cases - Success Metrics
**Extraction Pattern**: `(\d{1,3}(?:,\d{3})*)\+?\s*(?:per\s*day|daily|customers?|users?|transactions?|reminders?|payments?|jobs?)`

**Calculation Logic**:
1. Read Section 2.3 Use Cases (approximate lines 100-165)
2. Find all volume metrics in Success Metrics subsections
3. Extract maximum customer/transaction count
4. If MAX count > 100,000 → HIGH Impact
5. If MAX count ≤ 100,000 → LOW Impact
6. Generate justification citing specific volume and line number

**Example**:
- Input: "500,000+ reminders per day" (Section 2.3, line 141)
- Output: HIGH Impact - "System impacts 500,000 customers/day (Section 2.3, line 141)"

#### 3. Impacts
**Definition**: Implementation complexity (configuration, development, deployment)
**Threshold**: >5 impacts = High Impact, ≤5 impacts = Low Impact
**Data Sources**: Section 5 Component Architecture + Section 8 Technology Stack
**Extraction Method**: Count subsection headers + table rows

**Calculation Logic**:
1. Read Section 5 Component Details (approximate lines 456-675)
2. Count component subsection headers matching pattern: `^###\s+\d+\.\d+`
3. Read Section 8 Technology Stack (approximate lines 912-998)
4. Count technology table rows (excluding headers and separators)
5. Total = component_count + technology_count
6. If total > 5 → HIGH Impact
7. If total ≤ 5 → LOW Impact
8. Generate justification with breakdown

**Example**:
- Input: 5 components in Section 5, 3 technologies in Section 8
- Output: HIGH Impact - "System requires 8 components/technologies (Section 5: 5, Section 8: 3)"

### 6-Phase Calculation Workflow

#### Phase 1: Detection & User Prompt

**Trigger Detection**:
When user request contains keywords:
- "architecture review", "audit", "assess architecture"
- "calculate design drivers", "update design drivers"
- "design impact", "impact metrics"

**User Prompt**:
```
═══════════════════════════════════════════════════════════
Would you like me to calculate Design Drivers impact metrics?
═══════════════════════════════════════════════════════════

Design Drivers provide quantifiable assessment across 3 dimensions:
• Value Delivery: Customer experience effectiveness (from Section 1)
• Scale: Customer/transaction volume impacted (from Section 2.3)
• Impacts: Implementation complexity (from Sections 5 & 8)

This will analyze existing architecture data and update Section 2.2.1.

[Calculate Now] [Skip] [Learn More]
```

If user selects "Learn More", explain:
```
Design Drivers help you:
✓ Quantify business value of architecture decisions
✓ Justify complexity based on scale and impact
✓ Communicate architecture priorities to stakeholders
✓ Validate alignment between architecture and business goals

The calculation is automatic - metrics are extracted from:
• Section 1: Business value percentages
• Section 2.3: Customer/transaction volumes
• Section 5: Component count
• Section 8: Technology count

Results are presented for your review before updating the document.
```

#### Phase 2: Context-Efficient Data Loading

**Sequential Loading Strategy** (minimize context usage):

```bash
# Load only required sections, NOT full document
# Total: ~400-500 lines vs 2000+ full document (75% reduction)

# Step 1: Load Document Index
Read(file_path="ARCHITECTURE.md", offset=1, limit=50)
# Parse index to get exact line ranges for Sections 1, 2, 5, 8

# Step 2: Load Section 1 Executive Summary
Read(file_path="ARCHITECTURE.md", offset=<section1_start-5>, limit=<section1_length+10>)
# Extract Business Value bullets, look for percentage metrics

# Step 3: Load Section 2.3 Use Cases
Read(file_path="ARCHITECTURE.md", offset=<section2.3_start-5>, limit=<section2.3_length+10>)
# Extract Success Metrics from all use cases, look for volume counts

# Step 4: Load Section 5 Component Details
Read(file_path="ARCHITECTURE.md", offset=<section5_start-5>, limit=<section5_length+10>)
# Count subsection headers (### X.X Component Name)

# Step 5: Load Section 8 Technology Stack
Read(file_path="ARCHITECTURE.md", offset=<section8_start-5>, limit=<section8_length+10>)
# Count technology table rows
```

**Context Buffer Guidelines**:
- Add ±5-10 lines buffer to each section for context preservation
- Load sections sequentially (one at a time), not simultaneously
- Extract metrics immediately after loading each section
- Clear temporary data before loading next section

#### Phase 3: Metric Extraction & Calculation

**Algorithm 1: Value Delivery Extraction**

```python
# Pseudo-code for Value Delivery calculation
def extract_value_delivery(section1_text, section1_start_line):
    # Find Business Value subsection
    business_value_match = re.search(r'\*\*Business Value:\*\*(.+?)(?=\n\n|\n\*\*|$)', section1_text, re.DOTALL)

    if not business_value_match:
        return {
            'impact': 'LOW',
            'justification': 'No quantifiable business value metrics found in Section 1',
            'percentage': None,
            'line': None
        }

    business_value_section = business_value_match.group(1)

    # Extract all percentage metrics
    percentage_pattern = r'(\d{1,3})%\s*(reduction|improvement|increase|efficiency|optimization|cost\s*savings?|faster|time\s*savings?)'
    matches = re.finditer(percentage_pattern, business_value_section, re.IGNORECASE)

    percentages = []
    for match in matches:
        percentage = int(match.group(1))
        context = match.group(2)
        # Calculate approximate line number within section
        line_offset = section1_text[:match.start()].count('\n')
        line_number = section1_start_line + line_offset
        percentages.append({
            'value': percentage,
            'context': context,
            'line': line_number
        })

    if not percentages:
        return {
            'impact': 'LOW',
            'justification': 'No percentage-based metrics found in Business Value',
            'percentage': None,
            'line': None
        }

    # Find maximum percentage
    max_metric = max(percentages, key=lambda x: x['value'])

    # Apply threshold
    impact = 'HIGH' if max_metric['value'] > 50 else 'LOW'

    return {
        'impact': impact,
        'justification': f"System delivers {max_metric['value']}% {max_metric['context']} (Section 1, line {max_metric['line']})",
        'percentage': max_metric['value'],
        'line': max_metric['line']
    }
```

**Algorithm 2: Scale Extraction**

```python
# Pseudo-code for Scale calculation
def extract_scale(section2_3_text, section2_3_start_line):
    # Extract all volume metrics from Success Metrics subsections
    volume_pattern = r'(\d{1,3}(?:,\d{3})*)\+?\s*(?:per\s*day|daily|customers?|users?|transactions?|reminders?|payments?|jobs?)'
    matches = re.finditer(volume_pattern, section2_3_text, re.IGNORECASE)

    volumes = []
    for match in matches:
        # Parse number (remove commas)
        volume_str = match.group(1).replace(',', '')
        volume = int(volume_str)
        context = match.group(0)
        # Calculate line number
        line_offset = section2_3_text[:match.start()].count('\n')
        line_number = section2_3_start_line + line_offset
        volumes.append({
            'value': volume,
            'context': context,
            'line': line_number
        })

    if not volumes:
        return {
            'impact': 'LOW',
            'justification': 'No volume metrics found in Use Cases success metrics',
            'volume': None,
            'line': None
        }

    # Find maximum volume
    max_volume = max(volumes, key=lambda x: x['value'])

    # Apply threshold
    impact = 'HIGH' if max_volume['value'] > 100000 else 'LOW'

    # Format volume with commas for readability
    volume_formatted = f"{max_volume['value']:,}"

    return {
        'impact': impact,
        'justification': f"System impacts {volume_formatted} customers/transactions per day (Section 2.3, line {max_volume['line']})",
        'volume': max_volume['value'],
        'line': max_volume['line']
    }
```

**Algorithm 3: Impacts Calculation**

```python
# Pseudo-code for Impacts calculation
def calculate_impacts(section5_text, section8_text):
    # Count component subsection headers in Section 5
    component_pattern = r'^###\s+\d+\.\d+\s+.+$'
    component_matches = re.findall(component_pattern, section5_text, re.MULTILINE)
    component_count = len(component_matches)

    # Count technology table rows in Section 8
    # Exclude header rows (contains | --- |) and empty rows
    tech_table_pattern = r'^\|[^|]+\|[^|]+\|.+\|$'
    separator_pattern = r'^\|\s*-+\s*\|'

    tech_rows = []
    for line in section8_text.split('\n'):
        if re.match(tech_table_pattern, line) and not re.match(separator_pattern, line):
            tech_rows.append(line)

    # First row is typically header, so subtract 1
    # Count unique tables (Languages, Frameworks, Databases, etc.)
    # Approximate: count all rows and subtract ~4-6 header rows
    technology_count = max(0, len(tech_rows) - 5)  # Conservative estimate

    # Total impacts
    total_impacts = component_count + technology_count

    # Apply threshold
    impact = 'HIGH' if total_impacts > 5 else 'LOW'

    return {
        'impact': impact,
        'justification': f"System requires {total_impacts} components/technologies (Section 5: {component_count}, Section 8: {technology_count})",
        'component_count': component_count,
        'technology_count': technology_count,
        'total': total_impacts
    }
```

#### Phase 4: Generate Calculation Report & User Review

**Report Format**:
```
═══════════════════════════════════════════════════════════
  DESIGN DRIVERS IMPACT METRICS CALCULATION
═══════════════════════════════════════════════════════════
Calculation Date: YYYY-MM-DD
Data Sources: Sections 1, 2.3, 5, 8
═══════════════════════════════════════════════════════════

## Value Delivery: [HIGH / LOW]
**Threshold**: >50% = High Impact
**Assessment**: [Percentage]% [context] exceeds threshold
**Justification**: System delivers [X]% [context] (Section 1, line [N])
**Source Data**: [Extracted percentage metric from Business Value]

## Scale: [HIGH / LOW]
**Threshold**: >100K = High Impact
**Assessment**: [Volume] [unit] exceeds threshold
**Justification**: System impacts [X] customers/day (Section 2.3, line [N])
**Source Data**: [Extracted volume metric from Success Metrics]

## Impacts: [HIGH / LOW]
**Threshold**: >5 = High Impact
**Assessment**: [Total] components/technologies exceeds threshold
**Justification**: System requires [Total] components/technologies (Section 5: [C], Section 8: [T])
**Breakdown**:
  - Components (Section 5): [Component count]
  - Technologies (Section 8): [Technology count]

═══════════════════════════════════════════════════════════
SUMMARY: [3/3 HIGH | 2/3 HIGH | 1/3 HIGH | 0/3 HIGH]
═══════════════════════════════════════════════════════════

Next Steps:
1. [Update Section 2.2.1] - Apply these metrics to Design Drivers subsection
2. [Preview Changes] - See exact updates before applying
3. [Manual Override] - Adjust values or classifications manually
4. [Export Report] - Save this report for documentation
5. [Skip Update] - Review only, don't modify ARCHITECTURE.md

Please select an option:
```

**Interactive Review Options**:

1. **Update Section 2.2.1**: Proceed with applying metrics to Design Drivers subsection
2. **Preview Changes**: Show exact before/after of Section 2.2.1
3. **Manual Override**: User can adjust thresholds, classifications, or justifications
4. **Export Report**: Save calculation report to separate file
5. **Skip Update**: Review-only mode, don't modify document

#### Phase 5: Apply Changes to ARCHITECTURE.md

**Insertion Logic**:

1. **Check if Section 2.2.1 exists**:
   ```bash
   Grep(pattern="### Design Drivers", path="ARCHITECTURE.md")
   # or
   Grep(pattern="### 2\.2\.1", path="ARCHITECTURE.md")
   ```

2. **If Section 2.2.1 DOES NOT exist** (first-time insertion):
   - Find end of Section 2.2 (Solution Overview)
   - Insert new Design Drivers subsection after Section 2.2
   - Update section numbering if needed (2.3 Use Cases remains 2.3)
   - Update Document Index (Section 2 line range expands)

3. **If Section 2.2.1 DOES exist** (update existing):
   - Read Section 2.2.1 content
   - Update only the impact assessments and justifications
   - Preserve structure and formatting
   - Update "Last Calculated" date
   - Update "Calculation Method" to "Automatic"

**Section 2.2.1 Template** (for insertion):
```markdown
### 2.2.1 Design Drivers

This architecture is driven by the following key factors:

#### Value Delivery
**Description**: Effectiveness of change in customer experience
- **Threshold**: >50% = High Impact, ≤50% = Low Impact
- **Current Assessment**: [HIGH / LOW] Impact
- **Justification**: [Extracted justification with line reference]

#### Scale
**Description**: Estimated number of customers impacted
- **Threshold**: >100K = High, ≤100K = Low
- **Current Assessment**: [HIGH / LOW] Impact
- **Justification**: [Extracted justification with line reference]

#### Impacts
**Description**: Implementation complexity across configuration, development, and applications
- **Threshold**: >5 = High, ≤5 = Low
- **Current Assessment**: [HIGH / LOW] Impact
- **Justification**: [Extracted justification with component/technology counts]

**Last Calculated**: [YYYY-MM-DD]
**Calculation Method**: Automatic
```

**Update Workflow**:

```bash
# Step 1: Find insertion point
Read(file_path="ARCHITECTURE.md", offset=82, limit=20)  # Section 2.2 Solution Overview
# Identify line number where Section 2.2 ends (before 2.3 Use Cases)

# Step 2: Insert or Update Section 2.2.1
# If new insertion:
Edit(file_path="ARCHITECTURE.md",
     old_string="[End of Section 2.2 content]\n\n### 2.3 Primary Use Cases",
     new_string="[End of Section 2.2 content]\n\n### 2.2.1 Design Drivers\n\n[Full template with calculated values]\n\n### 2.3 Primary Use Cases")

# If updating existing:
Edit(file_path="ARCHITECTURE.md",
     old_string="**Current Assessment**: [old value]\n- **Justification**: [old justification]",
     new_string="**Current Assessment**: [new value]\n- **Justification**: [new justification]")

# Step 3: Update "Last Calculated" date
Edit(file_path="ARCHITECTURE.md",
     old_string="**Last Calculated**: YYYY-MM-DD",
     new_string="**Last Calculated**: 2025-01-26")  # Use current date

# Step 4: Update Document Index (if significant line shift)
# Run grep to detect new section positions
grep -n "^## [0-9]" ARCHITECTURE.md
# Update index line ranges if Section 2 grew by >10 lines
```

#### Phase 6: Completion Report & Verification

**Completion Report Format**:
```
═══════════════════════════════════════════════════════════
  DESIGN DRIVERS UPDATE COMPLETE
═══════════════════════════════════════════════════════════

✓ Section 2.2.1 Design Drivers updated
✓ All three metrics calculated and applied
✓ Document Index updated (Section 2: Lines X-Y)
✓ Last Calculated: 2025-01-26

Summary of Metrics:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Value Delivery:  HIGH   (70% cost reduction)
Scale:           HIGH   (500,000 customers/day)
Impacts:         HIGH   (8 components/technologies)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Overall Design Impact: 3/3 HIGH

Data Sources:
• Section 1 (Executive Summary): Business value metrics
• Section 2.3 (Use Cases): Success metrics and volumes
• Section 5 (Components): Component architecture details
• Section 8 (Technology Stack): Technology inventory

Next Steps:
• Review Section 2.2.1 for accuracy
• Use these metrics in architecture presentations
• Recalculate after significant architecture changes
• Include in quarterly architecture reviews

═══════════════════════════════════════════════════════════
```

**Verification Steps**:
1. Read Section 2.2.1 to confirm updates applied correctly
2. Verify Document Index reflects new line ranges
3. Check "Last Calculated" date is current
4. Confirm all three metrics (Value Delivery, Scale, Impacts) are present
5. Validate justifications include source line numbers

### Edge Cases & Error Handling

#### Edge Case 1: Missing Source Data

**Scenario**: No percentage metrics found in Section 1 Business Value

**Handling**:
```
⚠️ MISSING DATA: Value Delivery

No percentage-based metrics found in Section 1 Business Value.

Recommendations:
1. Add quantifiable business value metrics to Section 1
2. Provide manual override for Value Delivery assessment
3. Default to LOW impact with note: "No quantifiable metrics found"

Would you like to:
[Add Metrics to Section 1] [Manual Override] [Default to LOW]
```

**Default Behavior**:
- Value Delivery: LOW Impact with justification "No quantifiable business value metrics found in Section 1"
- Scale: LOW Impact with justification "No volume metrics found in Use Cases"
- Impacts: Calculate based on available data (Section 5 or 8, whichever exists)

#### Edge Case 2: Threshold Edge Cases (Exactly 50%, 100K, or 5)

**Scenario**: Metric value exactly equals threshold

**Handling**:
```
ℹ️ THRESHOLD EDGE CASE: Scale

Volume metric: Exactly 100,000 customers/day (Section 2.3, line 141)
Threshold rule: >100K = HIGH (not ≥)

Assessment: LOW Impact (100K is NOT > 100K)
Note: Consider if this is close enough to merit HIGH classification

Would you like to:
[Use LOW (strict threshold)] [Override to HIGH] [Review Context]
```

**Default Rule**:
- Thresholds use strict > operator (not ≥)
- 50% exactly → LOW
- 100,000 exactly → LOW
- 5 exactly → LOW
- Flag these cases for user review with note in justification

#### Edge Case 3: Conflicting Metrics (Multiple Percentages)

**Scenario**: Multiple different percentage metrics in Business Value

**Example**:
- 70% cost reduction
- 45% time savings
- 60% efficiency improvement

**Handling**:
```
ℹ️ MULTIPLE METRICS: Value Delivery

Found 3 percentage metrics in Business Value:
• 70% cost reduction (Line 52)
• 45% time savings (Line 53)
• 60% efficiency improvement (Line 54)

Using maximum value approach:
Selected: 70% cost reduction → HIGH Impact

Alternative: Could use average (58.3%) → still HIGH
Alternative: Could list all metrics in justification

Proceed with maximum value (70%)? [Yes] [Use Average] [Custom]
```

**Default Behavior**: Use maximum percentage value

#### Edge Case 4: Non-Standard Document Structure

**Scenario**: Document Index missing or section numbers non-standard

**Handling**:
1. **Fallback Detection**: Use grep to find sections by header text
   ```bash
   grep -n "^## .*Executive Summary" ARCHITECTURE.md
   grep -n "^### .*Primary Use Cases" ARCHITECTURE.md
   grep -n "^## .*Component" ARCHITECTURE.md
   grep -n "^## .*Technology Stack" ARCHITECTURE.md
   ```

2. **Prompt User**:
   ```
   ⚠️ NON-STANDARD STRUCTURE

   Document Index not found or sections not in expected locations.

   Would you like me to:
   1. Search for sections by header text (slower but works)
   2. Update document to standard structure first
   3. Manually specify section line ranges

   Recommended: Option 2 (standardize structure)
   ```

3. **Graceful Degradation**: Calculate what's possible, note what's missing

#### Edge Case 5: Section 2.2.1 Already Exists with Manual Overrides

**Scenario**: User previously set manual override values in Section 2.2.1

**Detection**:
Check if "Calculation Method: Manual Override" exists in Section 2.2.1

**Handling**:
```
⚠️ MANUAL OVERRIDE DETECTED

Section 2.2.1 was previously updated with manual overrides.

Current Values (Manual):
• Value Delivery: HIGH (manually set)
• Scale: LOW (manually set)
• Impacts: HIGH (manually set)

New Calculated Values (Automatic):
• Value Delivery: HIGH (70% cost reduction)
• Scale: HIGH (500,000 customers/day)  ← DIFFERS
• Impacts: HIGH (8 components/technologies)

Differences found:
⚠️ Scale changed from LOW → HIGH (new data: 500,000/day)

Would you like to:
[Accept New Values] [Keep Manual Overrides] [Review Changes] [Merge (keep manual where unchanged)]
```

**Default Behavior**: Present comparison, require user approval before overwriting manual overrides

### Manual Override Workflow

**User can manually override any metric classification:**

```
Manual Override: Value Delivery
────────────────────────────────────────────────────────────
Calculated: HIGH (70% cost reduction)

Override Options:
1. [Keep Calculated] - Use automatic calculation (HIGH)
2. [Change to LOW] - Override to LOW impact
3. [Custom Percentage] - Specify different percentage
4. [Custom Justification] - Keep HIGH but change justification

If overriding, provide reason (for documentation):
[Text input: "Business value is delayed until Year 2"]

Method will be marked as: "Manual Override (2025-01-26)"
```

**When Manual Override Applied**:
- Update "Calculation Method" to "Manual Override"
- Add note in justification: "(Manual override: [reason])"
- Subsequent automatic calculations will prompt before overwriting

### Best Practices for Design Drivers

**DO:**
- ✅ Calculate after major architecture changes (new components, technologies)
- ✅ Recalculate quarterly during architecture reviews
- ✅ Use metrics in stakeholder presentations and justifications
- ✅ Document assumptions in justifications (which metric was used, why)
- ✅ Update source sections (1, 2.3, 5, 8) before calculating for accuracy
- ✅ Review calculation report carefully before applying

**DON'T:**
- ❌ Auto-calculate on every minor documentation update
- ❌ Override thresholds without documentation
- ❌ Ignore edge cases (flag for review, don't silently default)
- ❌ Skip user review (always present findings before updating)
- ❌ Forget to update "Last Calculated" date
- ❌ Calculate with stale data (ensure source sections are current)

### Integration with Existing Workflows

**Metric Consistency Workflow Integration**:
- Design Drivers calculation runs AFTER metric consistency check
- If Section 1 metrics are updated, first sync duplicates, then calculate drivers
- Ensures Design Drivers use most current metric values

**Document Index Integration**:
- Design Drivers insertion may shift Section 2.3 line numbers
- Automatically trigger Document Index update if Section 2 grows >10 lines
- Update index before final completion report

**Architecture Review Integration**:
- When user requests "review", "audit", or "validate" for ARCHITECTURE.md, follow the **Two-Phase Review & Audit Workflow**
- Workflow is documented in: **REVIEW_AUDIT_WORKFLOW.md**
- Phase 1 (Form Validation) must pass before Phase 2 (Content Improvement) runs
- Workflow provides prioritized feedback: HIGH → MEDIUM → LOW criticality

**Two-Phase Workflow Overview**:
  ```
  Phase 1: Form Validation (Blocking)
  ☐ Section structure compliance (12 required sections)
  ☐ Architecture type validation (META/3-TIER/MICROSERVICES/N-LAYER)
  ☐ Architecture Principles enforcement (9 required principles)
  ☐ Markdown formatting validation
  ☐ Metric consistency check (Section 1 vs rest of document)

  If Phase 1 passes → Proceed to Phase 2
  If Phase 1 fails → User must fix ❌ issues before Phase 2 runs

  Phase 2: Content Improvement (Prioritized)
  ☐ HIGH: Missing implementation details, dishonest trade-offs, incomplete components
  ☐ MEDIUM: Weak justifications, outdated references, missing cross-references
  ☐ LOW: Formatting suggestions, clarity improvements, redundancy reduction

  For complete workflow details, see: REVIEW_AUDIT_WORKFLOW.md
  ```

### Example: Complete Design Drivers Calculation

**Scenario**: User requests architecture review for Task Scheduling System

**Step-by-Step Execution**:

1. **User Request**:
   ```
   User: "Review the architecture documentation"
   ```

2. **Detection & Prompt**:
   ```
   [Architecture review trigger detected]

   As part of the architecture review, would you like me to calculate Design Drivers impact metrics?

   Design Drivers assess:
   • Value Delivery: Business value effectiveness
   • Scale: Customer/transaction volume
   • Impacts: Implementation complexity

   [Calculate Now] [Skip] [Learn More]
   ```

3. **User Approves**:
   ```
   User: "Calculate now"
   ```

4. **Context-Efficient Data Loading**:
   ```
   Loading architecture data for Design Drivers calculation...

   ✓ Document Index loaded (lines 1-21)
   ✓ Section 1 Executive Summary loaded (lines 25-54)
   ✓ Section 2.3 Use Cases loaded (lines 99-162)
   ✓ Section 5 Component Details loaded (lines 456-675)
   ✓ Section 8 Technology Stack loaded (lines 912-998)

   Total lines loaded: 436 / 2,000 (78% reduction)
   ```

5. **Metric Extraction**:
   ```
   Extracting metrics from architecture sections...

   Value Delivery:
   ✓ Found: 70% cost reduction (Section 1, line 52)
   ✓ Found: 60% time savings (Section 1, line 53)
   ✓ Maximum: 70% → HIGH Impact (exceeds 50% threshold)

   Scale:
   ✓ Found: 500,000+ reminders per day (Section 2.3, line 141)
   ✓ Found: 10,000+ concurrent jobs (Section 2.3, line 38)
   ✓ Maximum: 500,000 → HIGH Impact (exceeds 100K threshold)

   Impacts:
   ✓ Components (Section 5): 5 subsections found
   ✓ Technologies (Section 8): 3 table rows found
   ✓ Total: 8 → HIGH Impact (exceeds 5 threshold)
   ```

6. **Present Findings**:
   ```
   ═══════════════════════════════════════════════════════════
     DESIGN DRIVERS IMPACT METRICS CALCULATION
   ═══════════════════════════════════════════════════════════
   Calculation Date: 2025-01-26
   Data Sources: Sections 1, 2.3, 5, 8
   ═══════════════════════════════════════════════════════════

   ## Value Delivery: HIGH
   **Threshold**: >50% = High Impact
   **Assessment**: 70% exceeds threshold
   **Justification**: System delivers 70% cost reduction (Section 1, line 52)

   ## Scale: HIGH
   **Threshold**: >100K = High Impact
   **Assessment**: 500,000 exceeds threshold
   **Justification**: System impacts 500,000 customers/day (Section 2.3, line 141)

   ## Impacts: HIGH
   **Threshold**: >5 = High Impact
   **Assessment**: 8 exceeds threshold
   **Justification**: System requires 8 components/technologies (Section 5: 5, Section 8: 3)

   ═══════════════════════════════════════════════════════════
   SUMMARY: 3/3 HIGH - Strong business justification for architecture complexity
   ═══════════════════════════════════════════════════════════

   [Update Section 2.2.1] [Preview Changes] [Manual Override] [Skip]
   ```

7. **User Selects Update**:
   ```
   User: "Update Section 2.2.1"
   ```

8. **Apply Changes**:
   ```
   Updating Section 2.2.1 Design Drivers...

   [Checking if section exists]
   ✓ Section 2.2.1 not found - will insert new section

   [Finding insertion point]
   ✓ Section 2.2 ends at line 97
   ✓ Section 2.3 starts at line 99
   ✓ Insertion point: line 98

   [Inserting Design Drivers subsection]
   ✓ Inserted 22 lines at line 98
   ✓ Section 2.3 renumbered (now starts at line 121)

   [Updating Document Index]
   ✓ Section 2 range updated: Lines 56-154 → Lines 56-176
   ✓ Sections 2.3-12 ranges shifted by +22 lines
   ✓ Index Last Updated: 2025-01-26
   ```

9. **Completion Report**:
   ```
   ═══════════════════════════════════════════════════════════
     DESIGN DRIVERS UPDATE COMPLETE
   ═══════════════════════════════════════════════════════════

   ✓ Section 2.2.1 Design Drivers inserted at line 98
   ✓ All three metrics calculated and applied
   ✓ Document Index updated (Section 2: Lines 56-176)
   ✓ Last Calculated: 2025-01-26

   Summary of Metrics:
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   Value Delivery:  HIGH   (70% cost reduction)
   Scale:           HIGH   (500,000 customers/day)
   Impacts:         HIGH   (8 components/technologies)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   Overall Design Impact: 3/3 HIGH

   This indicates strong business justification for:
   • Architecture complexity (8 components/technologies)
   • Investment in scalability (500K customers/day)
   • Development resources (70% cost reduction value)

   ═══════════════════════════════════════════════════════════
   ```

### Performance & Context Optimization

**Context Usage Comparison**:

| Approach | Lines Loaded | Percentage |
|----------|-------------|------------|
| Full Document Load | 2,000+ | 100% |
| Design Drivers (Context-Efficient) | ~400-500 | 20-25% |
| **Savings** | **~1,500-1,600** | **75-80%** |

**Sequential Loading Benefits**:
- Load one section at a time (not all simultaneously)
- Extract metrics immediately after loading
- Clear section data before loading next
- Minimize context window pressure
- Faster execution with large documents

**Optimization Strategies**:
- Use Document Index for exact line ranges (no guessing)
- Load only ±5-10 line buffers (minimal context)
- Use Grep for initial detection (no loading required)
- Cache Document Index in memory (reuse across phases)
- Batch similar operations (all regex extractions together)

---

### Best Practices

✅ **DO**:
- Load specific sections using `Read` with offset/limit
- Use `Grep` to search for keywords without loading full file
- Work incrementally on one section at a time
- Read the table of contents or first 50 lines to orient yourself

❌ **DON'T**:
- Read the entire ARCHITECTURE.md file unless creating it from scratch
- Load multiple sections simultaneously unless they're small
- Use full document reads for simple section updates
- Assume section locations without checking first

### Example Workflows

**Task 1**: Update the Technology Stack section

1. Read lines 1-50 to confirm document structure
2. Use Grep to find "## Section 8: Technology Stack"
3. Read ONLY Section 8 (estimated lines 1001-1150)
4. Make edits using Edit tool
5. Read ONLY the updated section to verify

**Task 2**: Update Executive Summary metrics and ensure consistency

1. User edits Section 1 Key Metrics (lines 31-38)
2. Skill detects metric change automatically
3. Scan document using Grep for all metric values
4. Load context for each match (Read with ±5 line buffer)
5. Generate consistency report with findings
6. Present findings to user for review
7. If user approves, batch update all locations
8. Update Document Index if needed (>10 lines shifted)
9. Report completion summary

These approaches keep context usage minimal while maintaining full editing capability.

## Documentation Structure

This project uses **ARCHITECTURE_DOCUMENTATION_GUIDE.md** as the authoritative guide for creating and maintaining architecture documentation. This guide ensures consistency across all architecture documents.

## When to Use the Guide

### 1. Creating New Architecture Documents
When starting a new system architecture document (ARCHITECTURE.md):
- Follow the 12-section structure defined in ARCHITECTURE_DOCUMENTATION_GUIDE.md
- Use the templates provided for each section
- Maintain detailed coverage for: Layers, Components, Integration Points, Tech Stack, Security, and Performance
- Keep Executive Summary, System Overview, and Principles concise
- Reference ADR_GUIDE.md for architectural decision documentation

### 2. Documenting Existing Projects
When documenting an existing system:
- Use ARCHITECTURE_DOCUMENTATION_GUIDE.md as the structure template
- Focus on capturing current state accurately (not ideal state)
- Prioritize Sections 4-10 (core architecture sections)
- Document actual technologies, versions, and configurations in use
- Create ADRs for past decisions if context is available

### 3. Maintaining Architecture Documentation
When updating existing architecture docs:
- Follow the same section structure from ARCHITECTURE_DOCUMENTATION_GUIDE.md
- Keep core sections (Layers, Components, Security, Performance) detailed and current
- Update version numbers and technology stack as system evolves
- Add new ADRs for architectural changes
- Review quarterly for accuracy

## Key Principles

1. **Use Templates**: Follow section templates from ARCHITECTURE_DOCUMENTATION_GUIDE.md
2. **Stay Concise**: Keep Sections 1-3, 6, 12 brief; detail in Sections 4-5, 7-11
3. **Separate ADRs**: Use ADR_GUIDE.md for decision records, link from main doc
4. **Maintain Accuracy**: Document actual state, include version numbers
5. **Context Efficiency**: Balance detail with readability for Claude Code context

## Architecture Principles Enforcement (Section 3)

**CRITICAL**: Section 3 must include **9 required principles** in exact order, each with Description, Implementation, and Trade-offs.

### Required Principles

1. Separation of Concerns
2. High Availability
3. Scalability First
4. Security by Design
5. Observability
6. Resilience
7. Simplicity
8. Cloud-Native
9. Open Standards
10. Decouple Through Events (OPTIONAL - apply where async provides clear decoupling benefits)

### Quick Validation

✅ All 9 core principles present in order
✅ Each has Description + Implementation + Trade-offs
✅ Implementation is system-specific (not generic placeholders)
✅ Trade-offs honestly assess costs
✅ Decouple Through Events (#10) only where async benefits temporal decoupling

### Detailed Rules

For complete principle definitions, required structure, validation checklist, common mistakes, and updating workflows:
→ **VALIDATIONS.md** § Architecture Principles Enforcement
→ **ARCHITECTURE_DOCUMENTATION_GUIDE.md** § Section 3: Architecture Principles

## Required Section Names (Strict Enforcement)

**CRITICAL**: All ARCHITECTURE.md documents MUST use these **12 exact section names** in this **exact order**.

### Standard Section Names

1. Executive Summary
2. System Overview
3. Architecture Principles
4. Architecture Layers
5. Component Details
6. Data Flow Patterns (OPTIONAL - omit for simple systems, renumber 7→6, 8→7, etc.)
7. Integration Points
8. Technology Stack
9. Security Architecture
10. Scalability & Performance
11. Operational Considerations
12. Architecture Decision Records (ADRs)

### Format Requirement

Use exact format: `## [NUMBER]. [SECTION NAME]`

**Examples**:
- ✅ `## 1. Executive Summary`
- ✅ `## 12. Architecture Decision Records (ADRs)`
- ❌ `## Executive Summary` (missing number)
- ❌ `## 12. ADR References` (wrong name)

### Verification Command

```bash
grep -n "^## [0-9]" ARCHITECTURE.md
```

### Detailed Rules

For complete section name list, common mistakes, optional sections, renumbering workflow, and validation checklist:
→ **VALIDATIONS.md** § Section Name Enforcement
→ **ARCHITECTURE_DOCUMENTATION_GUIDE.md** § Document Structure Overview

---

## Workflow 7: Informational Query (Answer Questions About Architecture)

### When to Use

This workflow is activated when users ask questions about the architecture documented in ARCHITECTURE.md:
- Understanding how the system works
- Learning about specific architectural decisions
- Querying technology choices, patterns, or principles
- Understanding data flows, integrations, or components

### Automatic Invocation Conditions

This workflow is **automatically invoked** when:
1. User asks an architectural question (authentication, scaling, components, tech stack, etc.)
2. **AND** an ARCHITECTURE.md file exists in the current project

If ARCHITECTURE.md does not exist, this workflow is NOT invoked, allowing general architectural guidance.

**Detection Logic**:
```python
# Check if ARCHITECTURE.md exists before answering architectural questions
architecture_file_exists = file_exists("ARCHITECTURE.md")

if architecture_file_exists:
    # Invoke Workflow 7: Use ARCHITECTURE.md as source of truth
    load_architecture_and_answer()
else:
    # Don't invoke skill - provide general guidance
    pass
```

### Prerequisites

- ARCHITECTURE.md file exists in the project
- Document has valid structure (12 sections)
- Document Index is present (lines 5-21 typically)

### Process

#### Step 1: Detect Query Intent

Classify the user's question to determine which section(s) to reference:

- **Authentication/Security** → Section 9 (Security Considerations)
- **Data flow/pipelines** → Section 6 (Data Flow)
- **Components/services** → Section 5 (Component Details)
- **Technology stack** → Section 8 (Technology Stack)
- **Integrations** → Section 7 (Integration Points)
- **Scaling/performance** → Section 10 (Scalability & Performance)
- **Deployment** → Section 4 (Architecture Layers/Deployment)
- **Patterns/principles** → Section 3 (Architecture Principles)
- **Problem/solution** → Section 2 (System Overview)
- **Operations** → Section 11 (Operational Considerations)
- **Decisions/trade-offs** → Section 12 (Architecture Decision Records)
- **General overview** → Section 1 (Executive Summary)

**See QUERY_SECTION_MAPPING.md for detailed query-to-section mapping guide.**

#### Step 2: Load Document Index

Use context-efficient loading to get the Document Index:

```python
# Load Document Index (lines 1-50)
index_content = Read(file_path="ARCHITECTURE.md", offset=1, limit=50)
doc_index = parse_document_index(index_content)
```

**Expected Index Format**:
```markdown
## Document Index

**Quick Navigation:**
- [Section 1: Executive Summary](#1-executive-summary) → Lines 1-80
- [Section 2: System Overview](#2-system-overview) → Lines 81-150
...
- [Section 12: Architecture Decision Records](#12-architecture-decision-records-adrs) → Lines 1951-end

**Index Last Updated:** YYYY-MM-DD
```

#### Step 3: Load Relevant Section(s)

Using index-based loading for efficiency:

```python
# Get section range from Document Index
section_range = doc_index[section_number]

# Calculate load parameters
buffer = 20  # Standard buffer for context
offset = section_range["start"] - buffer - 1  # -1 for zero-indexing
limit = (section_range["end"] - section_range["start"]) + (2 * buffer)

# Load section
section_content = Read(file_path="ARCHITECTURE.md", offset=offset, limit=limit)
```

**Example**:
```python
# User asks: "What technologies do we use for the backend?"
# Detected section: Section 8 (Technology Stack)

# Index shows: Section 8 → Lines 912-998
buffer = 20
offset = 912 - 20 - 1 = 891
limit = (998 - 912) + (2 * 20) = 86 + 40 = 126

# Load: Read(file_path="ARCHITECTURE.md", offset=891, limit=126)
# This reads lines 891-1017 (Section 8 with 20-line buffers)
```

**Multi-Section Queries**:

For questions requiring multiple sections:

```python
# User asks: "How does authentication work with external systems?"
# Detected sections: Section 9 (Security) + Section 7 (Integrations)

# Load Section 9 first
section_9_content = Read(file_path="ARCHITECTURE.md", offset=section_9_start-20-1, limit=section_9_length+40)

# Load Section 7 second
section_7_content = Read(file_path="ARCHITECTURE.md", offset=section_7_start-20-1, limit=section_7_length+40)

# Synthesize answer from both sections
```

#### Step 4: Extract Relevant Information

- Search for keywords from user query within the loaded section
- Identify specific subsections that address the question
- Calculate line numbers for citations

**Line Number Calculation**:
```python
# Calculate absolute line numbers for citations
subsection_start = section_range["start"] + relative_offset
subsection_end = subsection_start + content_length

# Example: Found answer in Section 9.2 Authentication
# Section 9 starts at line 1301
# Subsection 9.2 is at relative offset 42 lines into the section
# Absolute line: 1301 + 42 = 1343
```

#### Step 5: Respond with Citations

**Standard Format**: Answer first, then footer citation

**Template**:
```
[Answer content based on ARCHITECTURE.md]

**Source**: ARCHITECTURE.md Section N (Section Name), lines X-Y
```

**Single-Section Answer Example**:
```
User: "How do we handle authentication?"

Response:
The system uses OAuth 2.0 for authentication with JWT tokens for API access.
Multi-factor authentication (MFA) is required for administrative users, and
session tokens expire after 24 hours of inactivity.

**Source**: ARCHITECTURE.md Section 9 (Security Considerations → Authentication &
Authorization), lines 1090-1114
```

**Multi-Section Answer Example**:
```
User: "What's the backend tech stack and how is it deployed?"

Response:
The backend is built with Java 17 and Spring Boot 3.2, using PostgreSQL 15 for the
primary database and Redis 7.0 for caching. The system is deployed on AWS using
containerized services via ECS with auto-scaling enabled for high availability.

**Sources**:
- ARCHITECTURE.md Section 8 (Technology Stack → Backend), lines 901-925
- ARCHITECTURE.md Section 8 (Technology Stack → Data Tier), lines 960-970
- ARCHITECTURE.md Section 4 (Architecture Layers → Cloud Infrastructure), lines 350-375
```

#### Step 6: Handle Missing Information

If the question cannot be answered from ARCHITECTURE.md:

**Response Format**:
1. Provide general architectural guidance from knowledge base
2. Clearly tag that it's NOT from the documented architecture
3. Recommend documenting it in the appropriate section

**Example**:
```
User: "What's our disaster recovery RTO?"

Response:
Based on the 99.99% availability SLA documented in Section 10, a typical RTO would be
4 hours for Tier 1 systems. Common industry practice for this availability tier includes:
- RTO: 4 hours
- RPO: 1 hour
- Quarterly DR testing

⚠️ **Recommendation**: This information is not currently documented in ARCHITECTURE.md.
Consider adding disaster recovery procedures to Section 11.4 (Operational Considerations →
Disaster Recovery) including:
- Recovery Time Objective (RTO)
- Recovery Point Objective (RPO)
- Backup strategies
- Failover procedures
- DR testing schedule

Would you like me to help document these DR procedures?

**Source**: General architectural guidance (not documented in ARCHITECTURE.md)
```

**Missing Information Template**:
```
[General architectural answer based on industry best practices]

⚠️ **Recommendation**: This information is not documented in ARCHITECTURE.md.
Consider adding to Section N.M ([Section Name → Subsection Name]).

Would you like me to help document this?

**Source**: General architectural guidance (not documented in ARCHITECTURE.md)
```

### Context Efficiency Guidelines

**Buffer Sizes**:
- Standard queries: ±20 lines
- Narrow queries (specific config): ±10 lines
- Cross-section queries: ±20 lines per section

**Loading Strategy**:
- Load Document Index once, cache in memory
- Load only relevant sections, not full document
- Use sequential loading (one section at a time) for multi-section queries
- Minimize total lines loaded (aim for <300 lines per query)

**Performance Comparison**:

| Approach | Lines Loaded | Context Usage |
|----------|-------------|---------------|
| Full Document Load | 2,000+ | 100% |
| Workflow 7 (Single Section) | ~100-150 | 5-7.5% |
| Workflow 7 (Multi-Section) | ~200-300 | 10-15% |
| **Savings** | **~1,700-1,800** | **85-90%** |

### Best Practices

**DO:**
- ✅ Always check if ARCHITECTURE.md exists before invoking this workflow
- ✅ Use Document Index for exact section ranges
- ✅ Load only relevant sections with minimal buffers
- ✅ Include line number citations in all responses
- ✅ Explicitly state when information is not documented
- ✅ Offer to help update documentation when gaps are found
- ✅ Use footer citations (answer first, source last)

**DON'T:**
- ❌ Load full document for simple queries
- ❌ Provide answers without citations when ARCHITECTURE.md exists
- ❌ Guess or assume content without reading the document
- ❌ Mix documented architecture with general guidance (clearly separate)
- ❌ Skip buffer context (always load ±10-20 lines)

### Example Workflows

#### Example 1: Question Fully Answered (Single Section)

**User**: "What databases do we use?"

**Process**:
1. Detect query intent → Section 8 (Technology Stack)
2. Load Document Index → Section 8: Lines 912-998
3. Load Section 8 with buffer → Read(offset=891, limit=126)
4. Extract database information → PostgreSQL 15, Redis 7.0, Elasticsearch 8.x
5. Calculate line numbers → Lines 960-970

**Response**:
```
The system uses three database technologies:

**Relational Database**: PostgreSQL 15 for primary transactional data
**Caching**: Redis 7.0 for session management and query caching
**Search**: Elasticsearch 8.x for full-text search and analytics

**Source**: ARCHITECTURE.md Section 8 (Technology Stack → Data Tier), lines 960-970
```

#### Example 2: Information Missing (With Recommendation)

**User**: "What's our backup retention policy?"

**Process**:
1. Detect query intent → Section 11 (Operational Considerations)
2. Load Document Index → Section 11: Lines 1751-1950
3. Load Section 11 with buffer → Read(offset=1730, limit=239)
4. Search for backup/retention information → Not found
5. Provide general guidance + recommendation

**Response**:
```
Based on industry best practices for systems with 99.99% availability, typical backup
retention policies include:
- Daily backups: Retained for 30 days
- Weekly backups: Retained for 90 days
- Monthly backups: Retained for 1 year
- Annual backups: Retained for 7 years (compliance)

⚠️ **Recommendation**: This information is not currently documented in ARCHITECTURE.md.
Consider adding backup retention policies to Section 11.3 (Operational Considerations →
Backup & Recovery) including:
- Backup frequency (full vs. incremental)
- Retention periods by backup type
- Storage location and redundancy
- Recovery testing schedule

Would you like me to help document the backup retention policy?

**Source**: General architectural guidance (not documented in ARCHITECTURE.md)
```

#### Example 3: Multi-Section Query

**User**: "How does user authentication integrate with our external systems?"

**Process**:
1. Detect query intent → Section 9 (Security) + Section 7 (Integrations)
2. Load Document Index
3. Load Section 9 → Read(offset=1280, limit=269)
4. Load Section 7 → Read(offset=980, limit=189)
5. Synthesize answer from both sections
6. Calculate line numbers for both citations

**Response**:
```
User authentication uses OAuth 2.0 with JWT tokens, requiring MFA for administrative
users. External system integrations leverage this authentication framework through
API Gateway integration.

For external systems, authentication tokens are validated at the API Gateway layer
before requests are forwarded to downstream services. Integration with Salesforce
uses OAuth 2.0 client credentials flow, while the payment gateway integration uses
mutual TLS certificates for secure communication.

**Sources**:
- ARCHITECTURE.md Section 9 (Security Considerations → Authentication), lines 1090-1114
- ARCHITECTURE.md Section 7 (Integration Points → External Systems), lines 1025-1048
```

### Query-to-Section Mapping Reference

For detailed query-to-section mapping, including:
- Common question patterns for each section
- Example queries and expected section targets
- Multi-section query scenarios
- Edge cases and ambiguous queries

**See: QUERY_SECTION_MAPPING.md**

### Example Conversations

#### Example 1: Informational Query - Single Section

**User**: "What technologies do we use for the backend?"

**Skill Response**:

*Step 1: Detect query intent → Section 8 (Technology Stack)*
*Step 2: Load Document Index (lines 1-50)*
*Step 3: Load Section 8 with buffer (lines 891-1017)*
*Step 4: Extract backend technologies*

According to ARCHITECTURE.md Section 8 (Technology Stack):

**Backend Stack**:
- Language: Java 17 (LTS)
- Framework: Spring Boot 3.2
- Build Tool: Maven 3.9
- Testing: JUnit 5, Mockito, Spring Test
- Code Quality: SonarQube

**Data Tier**:
- Relational Database: PostgreSQL 15
- Caching: Redis 7.0
- Search: Elasticsearch 8.x

**Source**: ARCHITECTURE.md Section 8 (Technology Stack → Backend & Data Tier), lines 901-970

---

#### Example 2: Informational Query - Information Missing

**User**: "What's our disaster recovery plan?"

**Skill Response**:

*Step 1: Detect query intent → Section 11 (Operational Considerations)*
*Step 2: Load Document Index*
*Step 3: Load Section 11 (lines 1730-1969)*
*Step 4: Search for disaster recovery information → Not found*
*Step 5: Provide general guidance + recommendation*

Based on the 99.99% availability SLA documented in Section 10, typical disaster recovery procedures for this tier include:

- **RTO (Recovery Time Objective)**: 4 hours
- **RPO (Recovery Point Objective)**: 1 hour
- **Backup frequency**: Continuous replication + daily snapshots
- **Failover**: Automated multi-region failover
- **Testing**: Quarterly DR drills

⚠️ **Recommendation**: This information is not currently documented in ARCHITECTURE.md.

Consider adding disaster recovery procedures to Section 11.4 (Operational Considerations → Disaster Recovery) including:
- Recovery Time Objective (RTO)
- Recovery Point Objective (RPO)
- Backup strategies and retention
- Failover procedures
- DR testing schedule

Would you like me to help document these DR procedures?

**Source**: General architectural guidance (not documented in ARCHITECTURE.md)

---

#### Example 3: Informational Query - Multi-Section

**User**: "How does user authentication integrate with our external systems?"

**Skill Response**:

*Step 1: Detect query intent → Section 9 (Security) + Section 7 (Integrations)*
*Step 2: Load Document Index*
*Step 3: Load Section 9 (lines 1280-1569)*
*Step 4: Load Section 7 (lines 980-1169)*
*Step 5: Synthesize answer from both sections*

User authentication uses OAuth 2.0 with JWT tokens for API access. Multi-factor authentication (MFA) is required for administrative users, and session tokens expire after 24 hours of inactivity.

For external system integrations:

**API Gateway Integration**: Authentication tokens are validated at the API Gateway layer before requests are forwarded to downstream services. The gateway enforces token expiration and refresh policies.

**Salesforce Integration**: Uses OAuth 2.0 client credentials flow for service-to-service authentication. Tokens are cached for 30 minutes to reduce authentication overhead.

**Payment Gateway Integration**: Uses mutual TLS certificates for secure communication. Certificate rotation is automated and occurs every 90 days.

All external integrations log authentication events to the central security monitoring system for audit and threat detection.

**Sources**:
- ARCHITECTURE.md Section 9 (Security Architecture → Authentication & Authorization), lines 1090-1114
- ARCHITECTURE.md Section 7 (Integration Points → External Systems), lines 1025-1075

---

#### Example 4: Standard Workflow - Creating Architecture Documentation

**User**: "Create architecture documentation for my microservices system"

**Skill Response**:

*Detects new ARCHITECTURE.md creation → Invokes Architecture Type Selection Workflow*

I'll help you create architecture documentation for your microservices system.

📐 **Architecture Type Selection**

Before creating your ARCHITECTURE.md, please select the architecture type that best describes your system:

**1. META Architecture** (6-Layer Enterprise)
   - Best for: Large enterprise systems, financial services, complex integrations
   - Layers: Channels → UX → Business Scenarios → Integration → Domain → Core
   - Complexity: Very High
   - Team Size: Large (10+)

**2. 3-Tier Architecture** (Classic Web Application)
   - Best for: Web apps, REST APIs, standard CRUD systems
   - Tiers: Presentation → Application/Business Logic → Data
   - Complexity: Low
   - Team Size: Small-Medium (2-8)

**3. Microservices Architecture** (Cloud-Native Distributed)
   - Best for: Cloud-native systems, independently deployable services
   - Components: API Gateway → Services → Data Stores → Event Bus
   - Complexity: High
   - Team Size: Large (10+)

**4. N-Layer Architecture** (Customizable Patterns)
   - Best for: DDD, Clean Architecture, Hexagonal Architecture
   - Patterns: 4-Layer DDD, 5-Layer Extended, Clean Architecture, Hexagonal
   - Complexity: Medium-High
   - Team Size: Medium (4-10)

**Which architecture type best describes your system? (1-4)**

*User selects: "3" (Microservices)*

Perfect! I'll create your ARCHITECTURE.md using the Microservices Architecture template.

*Loads templates/SECTION_4_MICROSERVICES.md and templates/SECTION_5_MICROSERVICES.md*
*Creates ARCHITECTURE.md with:*
- *Standard Sections 1-3*
- *Microservices Section 4 (API Gateway, Service Mesh, Services, Event Bus)*
- *Microservices Section 5 (Service catalog format)*
- *Standard Sections 6-12*
- *Metadata comment: `<!-- ARCHITECTURE_TYPE: MICROSERVICES -->`*

✓ Created ARCHITECTURE.md with Microservices Architecture structure

The document includes:
- Document Index (lines 5-21) for context-efficient navigation
- Section 4: Microservices architecture layers (API Gateway, Service Mesh, Services, Event Bus)
- Section 5: Service catalog template (ready for your services)
- Architecture type metadata for future validation

Next steps:
1. Fill in Section 1 (Executive Summary) with your system overview
2. Document your microservices in Section 5
3. Add your technology stack to Section 8
4. Define integration points in Section 7

Would you like help filling in any specific section?

---

#### Example 5: Metric Consistency Workflow

**User**: "Update the job execution capacity to 700 TPS in the Executive Summary"

**Skill Response**:

*Uses Edit tool to change line 33: "500 TPS" → "700 TPS"*

✓ Updated line 33: Design Capacity - Job Execution: 700 TPS

*Automatic audit triggered because Section 1 Key Metrics was edited*

Now checking for metric consistency across the document...

*Scans document for references to old value (500 TPS)*
*Found at: Line 787, Line 1368*
*Loads context around each match*

═══════════════════════════════════════════════════════════
   METRIC CONSISTENCY AUDIT REPORT
═══════════════════════════════════════════════════════════
Triggered by: Executive Summary edit (Line 33)
Metric Changed: Job Execution Capacity (500 TPS → 700 TPS)
═══════════════════════════════════════════════════════════

## ⚠️ Outdated References Found

| Location | Section | Current Value | Should Be | Context |
|----------|---------|---------------|-----------|---------|
| Line 787 | Section 6: Data Flow Patterns | 500 TPS capacity | 700 TPS | "Performance: p50 = 40ms, p95 = 100ms, p99 = 200ms \| 500 TPS capacity" |
| Line 1368 | Section 10: Scalability & Performance | 500 | 700 | Throughput table: "Job Execution (TPS) \| 500 \| 1000 \| 2000" |

═══════════════════════════════════════════════════════════
SUMMARY: Found 2 locations with outdated metric value
═══════════════════════════════════════════════════════════

Would you like me to update both locations to 700 TPS?

Options:
1. [Yes - Update All] - Apply changes to both locations
2. [Preview Changes] - Show me exactly what will change
3. [Manual Selection] - Let me choose individually
4. [No Thanks] - I'll handle this manually

*User selects: "Preview changes first"*

Here's a preview of the changes I'll make:

════════════════════════════════════════════════════════════
CHANGE 1 of 2: Line 787 (Section 6: Data Flow Patterns)
════════════════════════════════════════════════════════════

Current:
**Performance**: p50 = 40ms, p95 = 100ms, p99 = 200ms | 500 TPS capacity

Will become:
**Performance**: p50 = 40ms, p95 = 100ms, p99 = 200ms | 700 TPS capacity

════════════════════════════════════════════════════════════
CHANGE 2 of 2: Line 1368 (Section 10: Throughput Table)
════════════════════════════════════════════════════════════

Current:
| Job Execution (TPS) | 500 | 1000 | 2000 | Downstream domain services |

Will become:
| Job Execution (TPS) | 700 | 1000 | 2000 | Downstream domain services |

════════════════════════════════════════════════════════════

Proceed with both changes? [Yes/No/Choose Individually]

*User approves: "Yes"*

Applying metric consistency updates...

✓ Updated line 787: "500 TPS capacity" → "700 TPS capacity"
✓ Updated line 1368: Throughput table "500" → "700"
✓ Document Index is still accurate (no update needed)

═══════════════════════════════════════════════════════════
   METRIC CONSISTENCY UPDATE COMPLETE
═══════════════════════════════════════════════════════════

Summary of Changes:
✓ Executive Summary (Line 33): 500 TPS → 700 TPS
✓ Section 6 Data Flow (Line 787): 500 TPS → 700 TPS
✓ Section 10 Throughput Table (Line 1368): 500 → 700

Total Locations Updated: 3
Document Index: No update needed
All references to Job Execution Capacity are now consistent.

═══════════════════════════════════════════════════════════

---

## Workflow 8: Presentation Generation (Generate Architecture Presentations)

**⚡ Auto-triggers**: This workflow automatically activates when presentation-related keywords are detected (see Automatic Workflow Detection section above).

### When to Use

This workflow is activated when users request presentation generation from ARCHITECTURE.md:
- "Generate architecture presentation"
- "Create PowerPoint from architecture"
- "Generate presentation for [stakeholder]"
- "Create slides for architecture review"
- "Generate presentation markdown file"
- User references: "business presentation", "compliance review deck", "architecture overview slides"

### Output Format

This workflow generates a **Markdown file** (`.md`) structured for **Claude PowerPoint** to convert into a final PowerPoint presentation. Claude directly writes the slide content using extracted ARCHITECTURE.md data — no external scripts or runtimes required.

**Markdown Structure for Claude PowerPoint:**
```markdown
# [Presentation Title]
## [Subtitle — Stakeholder type | Date]

---

## [Slide Title]

Content bullets, tables, or code blocks

---

## SECCIÓN X: [Section Divider Title]

---

## [Next Slide Title]
...
```

**Slide type conventions:**
- **Standard slide**: `## Title` + bullet list
- **Section divider**: `## SECCIÓN N: Title` with no body content (full-color visual break)
- **Metrics slide**: `## Title` + markdown table with 3 metric columns
- **Comparison slide**: `## Title` + two-column bullet list (left/right separated by `**VS**` or `| Left | Right |` table)
- **Process slide**: `## Title` + numbered list (sequential steps)
- **Quote slide**: `## Title` + blockquote `>` syntax
- **Call to action**: `## Title` + bold contact info lines

### Activation Triggers

**Automatic Invocation:**
- User asks to "generate presentation", "create slides", "make PowerPoint", "create pptx", "generate presentation md"
- User specifies stakeholder: "business stakeholders", "architecture team", "compliance review"
- User specifies language: "presentación en español", "presentation in English"

**Manual Invocation:**
- User explicitly: "Run presentation generation workflow"
- User references: `/workflow presentation`

### Prerequisites

- **ARCHITECTURE.md file exists** in the project
- Document has **valid 12-section structure**
- **Document Index is present** (lines 1-50 typically)

### Step-by-Step Process

#### Step 1: Detect Request & Present Stakeholder Options

When presentation generation is requested, present the following options:

```
📊 **Architecture Presentation Generation**

I'll generate a PowerPoint presentation from your ARCHITECTURE.md file.

**Step 1: Select Stakeholder Type**

Who is the target audience for this presentation?

1. **Business Stakeholders** - Focus on value, ROI, business metrics, use cases
2. **Architecture Team** - Technical details, components, patterns, decisions

**Please select: 1 or 2**
```

**Stakeholder Focus Areas**:

| Stakeholder | Focus | Primary Sections | Slide Count |
|-------------|-------|------------------|-------------|
| Business | Business value, ROI, metrics, use cases | 1, 2, 10, 11 | ~7 slides |
| Architecture | Technical design, components, patterns | 3, 4, 5, 6, 7, 8, 12 | ~9 slides |

#### Step 2: Language Selection

After stakeholder selection:

```
**Step 2: Select Language**

Which language should the presentation use?

1. **English (EN)** - Default
2. **Spanish (ES)** - Español

**Please select: 1 or 2** (or type: en/es)
```

**Translation Scope**:
- ✅ **Translate**: Slide titles, section headers, labels, UI elements
- ❌ **Do NOT translate**: System names, component names, technologies, metrics, code snippets

#### Step 3: Confirmation

```
**Step 3: Confirmation**

I'll generate a presentation Markdown file with these settings:
- Stakeholder: [Business/Architecture/Compliance]
- Language: [English/Spanish]
- Slides: ~7-9 slides (~10 minutes)
- Output: /presentations/ARCHITECTURE_[Type]_[Lang].md (ready for Claude PowerPoint)

Proceed with generation? [Yes/No]
```

#### Step 4: Load Document Index

**Process**:
1. Read lines 1-50 of ARCHITECTURE.md
2. Parse Document Index to extract section line ranges
3. Validate 12-section structure
4. Extract system name from first heading

**Fallback**: If Document Index not found, use default line ranges with warning

#### Step 5: Load Required Sections Incrementally

**Context-Efficient Loading**:
- Load only sections required for the selected stakeholder
- Use ±10 line buffer around each section
- Minimize total lines loaded (50-80% reduction vs. full document)

**Sections by Stakeholder**:
- **Business**: Sections 1, 2, 10, 11 (~400 lines, 80% reduction)
- **Architecture**: Sections 3, 4, 5, 6, 7, 8, 12 (~1000 lines, 50% reduction)
- **Compliance**: Sections 7, 8, 9, 10, 11, 12 (~700 lines, 65% reduction)

#### Step 6: Generate Summaries Using LLM (NEW - Recommended)

**IMPORTANT**: Use LLM capabilities to generate slide summaries instead of regex extraction.

**Process**:
1. Load slide template for stakeholder type
2. For each content slide in template:
   - Identify `data_sources` from slide config (section numbers)
   - Read those sections from ARCHITECTURE.md
   - Generate summary using LLM based on slide type + stakeholder
   - Format as 3-6 bullet points
3. Save all summaries to JSON file

**Summarization Prompts by Slide Type**:

```
Executive Summary (Slide 3):
"Summarize the key metrics and system purpose in 3-4 bullet points for [stakeholder] stakeholders. Focus on quantifiable metrics and business value. Language: [language]"

Architecture Principles (Slide 4):
"List the top 5 architecture principles from this content. For each, provide the principle name and a 1-sentence description. Language: [language]"

Components (Slide 6):
"Summarize the main system components in 4-5 bullet points. Focus on component responsibilities and interactions. Language: [language]"

Technology Stack (Slide 7):
"List the key technologies used, organized by category (Backend, Frontend, Data, Infrastructure). Keep it concise. Language: [language]"

Data Flow (Slide 8):
"Describe the main data flow patterns in 3-4 bullet points. Focus on how data moves through the system. Language: [language]"

Integration Points (Slide 9):
"List the main external integrations in 4-5 bullet points. Include system names and integration methods. Language: [language]"

Security Architecture (Slide 10):
"Summarize security controls in 4-5 bullet points. Cover authentication, encryption, and access controls. Language: [language]"

ADRs (Slide 11):
"List the top 5 most important architecture decisions in format: ADR-### : [Decision title]. Language: [language]"
```

**Output Format (JSON)**:
```json
{
  "3": ["Metric bullet 1", "Metric bullet 2", "Metric bullet 3"],
  "4": ["Principle 1", "Principle 2", "Principle 3"],
  "5": ["Layer description 1", "Layer description 2"],
  ...
}
```

**Save to**: `/tmp/presentation_summaries_{stakeholder}_{language}.json`

**Fallback**: If LLM summarization produces incomplete content, re-read the relevant ARCHITECTURE.md sections and retry the summarization step.

#### Step 7: Load Language Translations

**Process**:
1. Load appropriate language JSON file (language_en.json or language_es.json)
2. Initialize LanguageManager with translations
3. Prepare slide titles, labels, and UI strings in target language

**Translation Files Location**:
- `/skills/architecture-docs/presentation/language_en.json`
- `/skills/architecture-docs/presentation/language_es.json`

#### Step 8: Generate Markdown File for Claude PowerPoint

**Claude directly writes the Markdown file** — no external scripts or runtime required.

**Generation Process**:
1. Use the LLM-generated summaries from Step 6 as slide content
2. Apply the slide template structure for the selected stakeholder (see Slide Templates section)
3. Map each slide type to its Markdown convention:

| Slide Type | Markdown Convention |
|------------|-------------------|
| Title | `# System Name\n## Subtitle — Stakeholder \| Date` |
| Agenda | `## Agenda` + numbered list of sections |
| Section Divider | `## SECCIÓN N: Title` (no body) |
| Standard content | `## Title` + bullet list (3-6 points) |
| Metrics | `## Title` + 3-column table `\| Metric \| Value \| Label \|` |
| Comparison | `## Title` + `\| Left Column \| Right Column \|` table |
| Process/Steps | `## Title` + numbered list (sequential steps) |
| Quote | `## Title` + `> blockquote text` |
| Call to Action | `## Title` + bold contact lines |
| Summary | `## Resumen / Summary` + key takeaway bullets |

4. Separate every slide with `---`
5. Apply selected language (ES/EN) to all titles, labels, and section headers
6. Keep system names, component names, technologies, and metric values in their original language

**Content Writing Rules**:
- Write concise, presentation-ready language (not documentation prose)
- Each content slide: 3-6 bullet points maximum
- Bullet points: one idea per line, ≤15 words
- Metrics table: exactly 3 columns, values extracted from ARCHITECTURE.md
- Comparison table: left = first concept, right = second concept, max 5 rows each

#### Step 9: Save Markdown File

**Output Path**: `/presentations/ARCHITECTURE_{Type}_{Lang}.md`

**Examples**:
- `/presentations/ARCHITECTURE_Business_EN.md`
- `/presentations/ARCHITECTURE_Architecture_ES.md`
- `/presentations/ARCHITECTURE_Compliance_EN.md`

**Process**:
1. Ensure `/presentations/` directory exists (create if needed)
2. Write the complete Markdown content to file using the Write tool
3. Verify file creation and display the path to the user

#### Step 10: Display Success Summary

**Success Message Template**:

```
═══════════════════════════════════════════════════════════
   PRESENTATION MARKDOWN GENERATION COMPLETE
═══════════════════════════════════════════════════════════

✓ Successfully generated presentation Markdown file!
📁 Output: /presentations/ARCHITECTURE_Business_EN.md
📊 Slides: 10 slides (~15 minute presentation)
🎯 Stakeholder: Business
🌐 Language: English

Data Sources Used:
- Section 1: Executive Summary (lines 25-65)
- Section 2: System Overview (lines 66-120)
- Section 10: Scalability & Performance (lines 1551-1750)
- Section 11: Operational Considerations (lines 1751-1950)

Next Steps:
1. Open the generated .md file
2. Use Claude PowerPoint to convert it to a .pptx file
3. Add company branding/logos in PowerPoint as needed

═══════════════════════════════════════════════════════════
```

### Invocation

This workflow is fully executed by Claude — no command-line tools or runtimes are required. Claude reads the relevant sections from ARCHITECTURE.md, generates the slide content using LLM summarization, and writes the resulting Markdown file directly using the Write tool.

**Skill invocation examples:**
```
/skill architecture-docs
> "Generate architecture presentation for business stakeholders in Spanish"

/skill architecture-docs
> "Create compliance presentation markdown"

/skill architecture-docs
> "Generate presentation for architecture team in English"
```

**Output file naming:**
- `/presentations/ARCHITECTURE_Business_ES.md`
- `/presentations/ARCHITECTURE_Architecture_EN.md`
- `/presentations/ARCHITECTURE_Compliance_ES.md`

**To convert to PowerPoint**: Open the generated `.md` file and use Claude PowerPoint to produce the final `.pptx`.

### Slide Templates by Stakeholder

#### Business Stakeholder Template (7 slides)

1. **Title Slide** - System name + "Architecture Overview for Business Stakeholders"
2. **Executive Summary** - Key metrics, business value, availability commitment (Section 1)
3. **Problem & Solution** - Problem statement + solution overview (Sections 2.1, 2.2)
4. **Key Use Cases & Target Users** - Top use cases, personas (Section 2.3)
5. **Performance & Availability** - SLA commitments, uptime, response time (Section 10)
6. **Architecture Principles** - Top 5 guiding principles (Section 3)
7. **Summary & Next Steps** - Key takeaways, contact, action items

#### Architecture Stakeholder Template (9 slides)

1. **Title Slide** - System name + "Technical Architecture Deep Dive"
2. **Executive Summary & Principles** - System purpose, top 5 principles (Sections 1, 3)
3. **Architecture Layers & Components** - Layer overview + key components (Sections 4, 5)
4. **Technology Stack** - Backend, frontend, data, infrastructure (Section 8)
5. **Data Flow & Integration Points** - Data patterns + external APIs (Sections 6, 7)
6. **Security Architecture** - Auth, encryption, access controls (Section 9)
7. **Key Architecture Decisions** - Top 5 ADRs (Section 12)
8. **Performance & Operational Model** - SLAs, monitoring, incident response (Sections 10, 11)
9. **Summary & Q&A** - Key patterns, open questions, next steps

#### Compliance Stakeholder Template

> ⚠️ **Coming soon** — Compliance presentation template is being reworked. Not available for generation at this time. If selected, inform the user and offer Business or Architecture instead.

### Error Handling

**Missing ARCHITECTURE.md**:
```
❌ Error: ARCHITECTURE.md not found at: [path]

Presentation generation requires a valid ARCHITECTURE.md file.
Please create one using the architecture-docs skill first.

To create ARCHITECTURE.md:
1. Use the architecture-docs skill
2. Select architecture type (Microservices, META, 3-Tier, BIAN, N-Layer)
3. Complete all 12 sections
4. Ensure Document Index is present
```

**Invalid Document Index**:
```
⚠️ Warning: Document Index not found or incomplete.
Using default line ranges. Presentation may have incomplete data.

Recommendation: Update Document Index in ARCHITECTURE.md
(See Workflow 4: Automatic Index Updates)
```

**Missing Sections**:
```
⚠️ Warning: Section [X] not found in ARCHITECTURE.md.
Slides requiring this section will show: "[Not documented in Section X]"

Affected slides:
- Slide 5: Use Cases (requires Section 2.3)
- Slide 8: Operational Support (requires Section 11)

Recommendation: Complete missing sections in ARCHITECTURE.md before regenerating.
```

**Empty Subsections**:
- Placeholder slides added: "[Not specified in ARCHITECTURE.md Section X.Y]"
- User notified of missing data in success summary
- Recommendation to complete documentation

### Context Efficiency

**Context Usage Comparison**:

| Approach | Lines Loaded | Context Usage | Savings |
|----------|-------------|---------------|---------|
| Full Document Load | 2,000-3,000 | 100% | - |
| Business Presentation | ~400 | 15-20% | 80-85% |
| Architecture Presentation | ~1,000 | 40-50% | 50-60% |
| Compliance Presentation | ~700 | 25-35% | 65-75% |

**Efficiency Techniques**:
- Document Index-based section location
- Incremental section loading with ±10 line buffers
- Load only stakeholder-relevant sections
- Reuse loaded sections across multiple extractions

### Integration with Other Workflows

**Typical Workflow Sequence**:
1. **Workflow 1**: Create ARCHITECTURE.md (select architecture type)
2. **Workflow 2**: Generate ADRs from Section 12
3. **Workflow 4**: Update Document Index after edits
4. **Workflow 5**: Verify metric consistency
5. **→ Workflow 8**: Generate presentations for stakeholders ✓

**Presentation Regeneration**:
- Regenerate presentations after ARCHITECTURE.md updates
- No need to recreate ARCHITECTURE.md, just regenerate presentations
- Presentations reflect latest documented architecture
- Version presentations by adding date suffix if needed

### Best Practices

**DO:**
- ✅ Generate presentations AFTER completing ARCHITECTURE.md
- ✅ Verify Document Index is up-to-date before generation
- ✅ Select stakeholder type based on presentation audience
- ✅ Regenerate presentations when architecture changes
- ✅ Review generated slides for accuracy
- ✅ Customize branding/logos after generation

**DON'T:**
- ❌ Generate presentations from incomplete ARCHITECTURE.md
- ❌ Manually edit ARCHITECTURE.md after presentation generation (regenerate instead)
- ❌ Assume all sections will be present (handle missing data gracefully)
- ❌ Translate technical content (system names, metrics remain as-is)

### Example Workflow Execution

**User**: "Generate architecture presentation for business stakeholders in Spanish"

**Skill Response**:

*Step 1: Detect request → Workflow 8 (Presentation Generation)*
*Step 2: Parse request → Stakeholder: Business, Language: Spanish*
*Step 3: Confirm settings*

📊 **Architecture Presentation Generation**

I'll generate a presentation Markdown file with these settings:
- Stakeholder: Business
- Language: Spanish
- Slides: ~10 slides (~15 minutes)
- Output: /presentations/ARCHITECTURE_Business_ES.md (ready for Claude PowerPoint)

Proceeding with generation...

═══════════════════════════════════════════════════════════
   ARCHITECTURE PRESENTATION GENERATION
═══════════════════════════════════════════════════════════
Stakeholder: Business
Language: ES
Output: /presentations/ARCHITECTURE_Business_ES.md
═══════════════════════════════════════════════════════════

Step 1/6: Loading ARCHITECTURE.md Document Index...
  ✓ Loaded index for 12 sections

Step 2/6: Loading required sections (1, 2, 10, 11)...
  ✓ Loaded 4 sections (387 lines)
  ✓ Context efficiency: 387/2,453 lines = 15.8%

Step 3/6: Generating slide summaries (LLM)...
  ✓ Generated summaries for 10 slides in Spanish

Step 4/6: Building Markdown slide structure...
  - Slide 1: Título ✓
  - Slide 2: Agenda ✓
  - Slide 3: Resumen Ejecutivo ✓
  - Slide 4: Problema y Solución ✓
  - Slide 5: Casos de Uso ✓
  - Slide 6: Valor de Negocio ✓
  - Slide 7: Disponibilidad del Sistema ✓
  - Slide 8: Soporte Operacional ✓
  - Slide 9: Principios de Arquitectura ✓
  - Slide 10: Resumen y Preguntas ✓

Step 5/6: Applying Spanish translations to titles and labels...
  ✓ All UI strings translated

Step 6/6: Writing Markdown file...
  ✓ Saved to /presentations/ARCHITECTURE_Business_ES.md

═══════════════════════════════════════════════════════════
   PRESENTATION MARKDOWN GENERATION COMPLETE
═══════════════════════════════════════════════════════════

✓ Successfully generated presentation Markdown file!
📁 Output: /presentations/ARCHITECTURE_Business_ES.md
📊 Slides: 10 slides (~15 minute presentation)
🎯 Stakeholder: Business
🌐 Language: Spanish

Data Sources Used:
- Section 1: Executive Summary (lines 25-87)
- Section 2: System Overview (lines 88-201)
- Section 10: Scalability & Performance (lines 1678-1802)
- Section 11: Operational Considerations (lines 1803-2015)

Próximos Pasos:
1. Abrir el archivo /presentations/ARCHITECTURE_Business_ES.md
2. Usar Claude PowerPoint para convertirlo a .pptx
3. Agregar branding corporativo / logos en PowerPoint si es necesario

═══════════════════════════════════════════════════════════

**Your presentation Markdown is ready!** Open `/presentations/ARCHITECTURE_Business_ES.md` and use **Claude PowerPoint** to convert it to a `.pptx` file.

---

## Workflow 9: Diagram Generation (Generate Architecture Diagrams)

**⚡ Auto-triggers**: This workflow automatically activates when diagram-related keywords are detected (see Automatic Workflow Detection section above).

### When to Use

This workflow is activated when users request diagram generation from ARCHITECTURE.md:
- "Generate architecture diagrams"
- "Create Mermaid diagrams from architecture"
- "Generate diagrams for my architecture"
- "Add diagrams to ARCHITECTURE.md"
- User references: "visual diagrams", "data flow diagrams", "component diagrams"

### Activation Triggers

**Automatic Invocation:**
- User asks to "generate diagrams", "create diagrams", "add diagrams", "make diagrams"
- User specifies: "Mermaid diagrams", "architecture diagrams", "visual diagrams"
- User references sections: "generate diagrams for Section 4"

**Manual Invocation:**
- User explicitly: "Run diagram generation workflow"
- User references: `/workflow diagrams`

### Prerequisites

- **ARCHITECTURE.md file exists** in the project
- Document has **valid 12-section structure** (or at least core sections 1-4)
- **Document Index is present** (lines 1-50 typically)
- **MERMAID_DIAGRAMS_GUIDE.md** available for templates (optional but recommended)

### Step-by-Step Process

#### Step 1: Detect Request & Present Diagram Type Options

When diagram generation is requested, present the following options:

```
📊 **Architecture Diagram Generation**

I'll generate Mermaid architecture diagrams for your ARCHITECTURE.md file.

**Step 1: Select Diagram Type**

What diagrams would you like to generate?

**1. High-Level Architecture Only (Recommended)** - Single comprehensive diagram
   - Shows Presentation → Application → Data layers (3-tier)
   - Or shows all 6 META layers for META architectures
   - Includes all major components and data flows

**2. Default Set** - 3 core diagrams
   - High-Level System Architecture (3-tier overview)
   - Infrastructure & Deployment (Azure AKS setup)
   - Component Interactions (Frontend ↔ Backend ↔ Database)

**Please select: 1 or 2**
```

**High-Level Architecture Diagram Details**:
- Single comprehensive Mermaid flowchart showing all tiers/layers
- Includes component relationships and data flows
- Color-coded by tier for easy visualization
- Suitable for executive presentations and documentation

**Default Set Diagram Details**:
1. **High-Level System Architecture** → Section 4.1 (Architecture Layers)
2. **Infrastructure & Deployment** → Section 9.2 (Deployment Architecture)
3. **High Availability & Failover** → Section 8.3 (Availability & Reliability)

#### Step 2: Target Location Selection

After diagram type selection:

```
**Step 2: Select Target Location**

Where should the diagrams be placed?

1. **Inline in ARCHITECTURE.md** - Embed diagrams directly in relevant sections (Recommended)
2. **Separate Section 4 Subsection** - Create dedicated "### Architecture Diagrams" subsection
3. **Both** - Inline for key diagrams + comprehensive set in Section 4

**Please select: 1, 2, or 3**
```

**Recommended Placements**:
- **High-Level Architecture**: Section 4.1 or Section 4 introduction
- **Data Flow Diagrams**: Section 5.3 (Data Flow Patterns)
- **Infrastructure Diagram**: Section 9.2 (Deployment Architecture)
- **HA Diagram**: Section 8.3 (Availability & Reliability)
- **Performance Diagram**: Section 8.1 (Performance Requirements)

#### Step 3: Confirmation

```
**Step 3: Confirmation**

I'll generate diagrams with these settings:
- Diagram Type: [High-Level Only (1)/Default Set (3)]
- Target Location: [Inline/Section 4 Subsection/Both]
- Total Diagrams: 1-6 diagrams (typically 3 for Default Set)
- Output: Mermaid code blocks embedded in ARCHITECTURE.md

Proceed with generation? [Yes/No]
```

#### Step 4: Load Document & Identify Architecture Type

**Process**:
1. Read lines 1-50 of ARCHITECTURE.md (Document Index)
2. Parse Document Index to extract section line ranges
3. Read Section 1 (Executive Summary) to identify system type
4. Determine architecture pattern (3-Tier, Microservices, META, N-Layer)
5. Read Section 4 (Architecture Layers) to understand component structure

**Architecture Type Detection**:
- **3-Tier**: Frontend → Backend → Database layers
- **Microservices**: Multiple independent services with API Gateway
- **META (6-Layer)**: Channels → UX → Business Scenarios → Service Orchestration → Atomic Services → Data
- **N-Layer**: Custom layered architecture (N > 3)

#### Step 5: Load Required Sections Incrementally

**Context-Efficient Loading**:
- Load only sections required for the selected diagram type
- Use ±10 line buffer around each section
- Minimize total lines loaded (similar to Workflow 8 approach)

**Sections by Diagram Type**:
- **High-Level Architecture Only (1 diagram)**: Section 4 (~150-300 lines, 85-90% reduction vs. full document)
- **Default Set (3 diagrams)**: Sections 4, 8, 9 (~500-800 lines, 60-75% reduction vs. full document)
- **Infrastructure Only**: Section 9 (~150-250 lines)
- **HA/Performance**: Section 8 (~200-350 lines)
- **Extended Set (6 diagrams)**: Sections 4, 5, 6, 8, 9 (~800-1200 lines, 40-60% reduction vs. full document)

#### Step 6: Generate Diagrams Using MERMAID_DIAGRAMS_GUIDE Templates

**Process**:
1. Load MERMAID_DIAGRAMS_GUIDE.md for reference templates
2. For each diagram to generate:
   - Identify appropriate Mermaid diagram type (graph TB, sequenceDiagram, etc.)
   - Extract key components, data flows, or infrastructure elements from sections
   - Apply color scheme and styling from templates
   - Generate complete Mermaid code block
3. Format with proper markdown code block syntax

**Mermaid Diagram Types**:
- **High-Level Architecture**: `graph TB` (top-to-bottom flowchart)
- **Data Flow**: `sequenceDiagram` (participant interactions)
- **Infrastructure**: `graph TB` with subgraphs (nested component groups)
- **HA/Performance**: `graph TB` or `graph LR` (left-to-right)

**Color Scheme** (from MERMAID_DIAGRAMS_GUIDE):
```
classDef presentation fill:#4A90E2,stroke:#2E5C8A,stroke-width:3px,color:#fff
classDef application fill:#F5A623,stroke:#B8791A,stroke-width:3px,color:#fff
classDef data fill:#7ED321,stroke:#5A9B18,stroke-width:3px,color:#fff
classDef cache fill:#BD10E0,stroke:#8A0CA3,stroke-width:3px,color:#fff
classDef azure fill:#0078D4,stroke:#005A9E,stroke-width:2px,color:#fff
```

#### Step 7: Insert Diagrams into ARCHITECTURE.md

**Process**:
1. For inline placement:
   - Read target section (e.g., Section 4.1)
   - Find appropriate insertion point (after section header or before subsection)
   - Use Edit tool to insert diagram with proper heading

2. For Section 4 subsection:
   - Create new `### 4.X Architecture Diagrams` subsection
   - Insert all diagrams with individual headings

3. Maintain proper markdown structure:
   ```markdown
   #### Diagram: High-Level System Architecture

   ```mermaid
   graph TB
       %% Diagram content here
   ```

   **Description**: This diagram shows...
   ```

#### Step 8: Verification & User Notification

**Verification**:
- ✅ All diagrams inserted successfully
- ✅ Mermaid syntax is valid (no unclosed code blocks)
- ✅ Proper headings and descriptions added
- ✅ Document structure maintained

**Notification**:
```
✅ **Diagram Generation Complete**

Generated diagrams (Default Set):
- High-Level System Architecture (Section 4.1)
- Infrastructure & Deployment (Section 9.2)
- High Availability & Failover (Section 8.3)

**Location**: ARCHITECTURE.md (inline in respective sections)

**Rendering**:
- GitHub/GitLab: Diagrams will render automatically
- VS Code: Install "Markdown Preview Mermaid Support" extension
- Online: Use https://mermaid.live/ to preview

**Next Steps**:
- Review diagrams in your markdown viewer
- Verify all components are correctly represented
- Request updates if any changes needed
```

### Error Handling

**Common Issues**:

| Issue | Solution |
|-------|----------|
| ARCHITECTURE.md not found | Prompt user to create ARCHITECTURE.md first (Workflow 1) |
| Document Index missing | Generate index first or use default line ranges with warning |
| Section not found | Skip that diagram type or prompt user to add section |
| Invalid Mermaid syntax | Validate syntax using mermaid-cli or fallback to simpler diagram |
| Insertion point conflict | Ask user for preferred location |

### Integration with Other Workflows

**Workflow Dependencies**:
- **Workflow 1 (Create ARCHITECTURE.md)**: Must run first if document doesn't exist
- **Workflow 8 (Presentation Generation)**: Can use generated diagrams in presentations
- **Workflow 7 (Informational Query)**: Can answer questions about generated diagrams

**Complementary Actions**:
- After diagram generation, suggest running Workflow 8 to create presentation
- Recommend adding diagram descriptions to ADRs (Workflow 2) if design decisions are visualized

---

## Optional Enhancements

- Data Flow Patterns (for complex flows)
- **Architecture Diagrams**: Create visual representations using Mermaid (recommended)
  - Section 4 templates include Mermaid examples for each architecture type
  - See MERMAID_DIAGRAMS_GUIDE.md for complete instructions
  - Supported diagram types: Layer diagrams, service topology, data flow, sequence diagrams
  - Benefits: Interactive, version-control friendly, renders in GitHub/GitLab
  - Alternative formats: C4 Model, PlantUML also supported
- Cost breakdown tables
- Disaster recovery procedures
- Runbooks (common operations and incident response procedures)

## Reference Documents

This skill includes the following asset files:

- **ARCHITECTURE_DOCUMENTATION_GUIDE.md**: Primary template and structure guide (located in skill assets)
- **ADR_GUIDE.md**: Architectural Decision Record format and guidelines (located in skill assets)

These reference documents are bundled with this skill and will be available when the skill is active.

**Note**: ARCHITECTURE_DOCUMENTATION_GUIDE.md is optimized for Claude Code context management. Load it when creating or significantly updating architecture docs.