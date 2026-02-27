---
name: skill-creation-best-practice
description: "Provides guidelines and best practices for creating and improving new skills for agents. (ref: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)"
---

# Skill creation best Practices

Learn how to create effective skills that agents can discover and use successfully.

---

Great skills are concise, well-structured, and tested in real usage. This guide provides practical authoring decisions to help agents discover and use skills effectively.

## Core Principles

### Conciseness Is Key

The context window is a shared resource. Your skill shares the context window with everything the agent needs to know, including:
- System prompt
- Conversation history
- Metadata for other skills
- The actual request

Not every token in the skill has a direct cost. At startup, only the metadata (name and description) for all skills is preloaded. The agent reads SKILL.md only when the skill becomes relevant and reads additional files only when needed. Still, conciseness inside SKILL.md matters. Once the agent loads it, every token competes with conversation history and other context.

**Default assumption**: The agent is already smart enough.

Add only context the agent does not already have. Challenge every piece of information:
- "Does the agent really need this explanation?"
- "Can I assume the agent already knows this?"
- "Is this paragraph worth its token cost?"

**Good example: concise** (about 50 tokens):
````markdown
## Extracting PDF Text

Use `pdfplumber` for text extraction:

```python
import pdfplumber

with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```
````

**Bad example: too verbose** (about 150 tokens):
```markdown
## Extracting PDF Text

PDF (Portable Document Format) files are a common file format that can include text, images, and other content. To extract text from a PDF, you need to use a library. There are many libraries available for PDF processing, but we recommend pdfplumber because it is easy to use and works for most cases. First, you need to install it using pip. Then you can use the following code...
```

The concise version assumes the agent already knows what a PDF is and how libraries work.

### Set the Right Degree of Freedom

Match the level of specificity to the task's fragility and variability.

**High freedom** (text-based guidance):

Use when:
- Multiple approaches are valid
- Decisions depend on context
- Heuristics guide the approach

Example:
```markdown
## Code Review Process

1. Analyze code structure and organization
2. Check for potential bugs and edge cases
3. Suggest improvements to readability and maintainability
4. Verify compliance with project conventions
```

**Medium freedom** (pseudocode or parameterized scripts):

Use when:
- Recommended patterns exist
- Some variation is acceptable
- Configuration affects behavior

Example:
````markdown
## Report Generation

Use this template and customize as needed:

```python
def generate_report(data, format="markdown", include_charts=True):
    # Process data
    # Generate output in the specified format
    # Optionally include visualizations
```
````

**Low freedom** (specific scripts, no or near-zero parameters):

Use when:
- Operations are fragile and error-prone
- Consistency matters
- A specific sequence must be followed

Example:
````markdown
## Database Migration

Run this script exactly as follows:

```bash
python scripts/migrate.py --verify --backup
```

Do not modify the command or add extra flags.
````

**Analogy**: Think of the agent as a robot exploring paths.
- **A narrow bridge with cliffs on both sides**: There is exactly one safe way forward. Provide explicit guardrails and precise instructions (low freedom). Example: a database migration that must be executed in a strict sequence.
- **An open field with no hazards**: Many paths can succeed. Give general direction and trust the agent to find the best route (high freedom). Example: a code review where the best approach depends on context.

### Test Across All Intended Models

Skills act as an add-on to a model, so effectiveness depends on the underlying model. Test the skill on every model you plan to use. If multiple models are in scope, aim for instructions that work well across all of them.

## Skill Structure

<Note>
**YAML frontmatter**: SKILL.md frontmatter requires two fields:

`name`:
- Max 64 characters
- Must contain only lowercase letters, numbers, and hyphens
- Must not include XML tags
- Must not include reserved words (as defined by the platform)

`description`:
- Must not be empty
- Max 1024 characters
- Must not include XML tags
- Must explain what the skill does and when it should be used

Refer to your platform specification for the complete skill structure.
</Note>

### Naming Conventions

Use a consistent naming pattern to make skills easy to reference and discuss. We recommend **gerunds** (verb + -ing) because they clearly describe the activity or function the skill provides.

Note that the `name` field must use only lowercase letters, numbers, and hyphens.

**Good naming examples (gerunds)**:
- `processing-pdfs`
- `analyzing-spreadsheets`
- `managing-databases`
- `testing-code`
- `writing-documentation`

**Acceptable alternatives**:
- Noun phrases: `pdf-processing`, `spreadsheet-analysis`
- Action-oriented: `process-pdfs`, `analyze-spreadsheets`

**Avoid**:
- Vague names: `helper`, `utils`, `tools`
- Overly broad: `documents`, `data`, `files`
- Reserved words: names containing platform-reserved words
- Inconsistent patterns within the skill collection

Consistent naming makes it easier to:
- Reference skills in docs and conversations
- Understand what a skill does at a glance
- Organize and search across multiple skills
- Maintain a professional, cohesive skill library

### Writing Effective Descriptions

The `description` field enables discovery and must include both what the skill does and when to use it.

<Warning>
**Always write in third person.** Descriptions are inserted into the system prompt, and mismatched perspective can cause discovery issues.

- **Good:** "Processes Excel files and generates reports."
- **Avoid:** "I can help process Excel files."
- **Avoid:** "You can use this to process Excel files."
</Warning>

**Be specific and include key terms.** Include both what the skill does and the trigger/context for when to use it.

Each skill has exactly one description field. Description is critical for skill selection. The agent uses it to choose the right skill from many available. The description must be detailed enough for the agent to know when to pick the skill, while the rest of SKILL.md provides implementation details.

Effective examples:

**PDF processing skill:**
```yaml
description: Extracts text and tables from PDF files, fills forms, and merges documents. Use when PDF files, forms, or document extraction are mentioned.
```

**Excel analysis skill:**
```yaml
description: Analyzes Excel spreadsheets, creates pivot tables, and generates charts. Use when analyzing Excel files, spreadsheets, tabular data, or .xlsx files.
```

**Git commit helper skill:**
```yaml
description: Analyzes git diffs and generates descriptive commit messages. Use when the user asks for help writing a commit message or reviewing staged changes.
```

Avoid vague descriptions like:

```yaml
description: Helps with documents
```
```yaml
description: Processes data
```
```yaml
description: Does things with files
```

### Progressive Disclosure Pattern

SKILL.md acts like a table of contents for an onboarding guide, pointing the agent to more detailed material. Refer to your platform overview for how progressive disclosure works.

**Practical guidance:**
- Keep the SKILL.md body under 500 lines for optimal performance
- Split content into separate files as you approach this limit
- Use the following patterns to organize instructions, code, and resources effectively

#### Visual Overview: Simple to Complex

A basic skill starts with only a SKILL.md file containing metadata and instructions.
As the skill grows, you can bundle additional content that the agent loads only when needed.
The full skill directory structure can look like this:

```
pdf/
├── SKILL.md              # Main instructions (loaded on trigger)
├── FORMS.md              # Form-filling guide (loaded as needed)
├── reference.md          # API reference (loaded as needed)
├── examples.md           # Usage examples (loaded as needed)
└── scripts/
    ├── analyze_form.py   # Utility script (execute, not load)
    ├── fill_form.py      # Form filling script
    └── validate.py       # Validation script
```

#### Pattern 1: High-Level Guide with References

````markdown
---
name: pdf-processing
description: Extracts text and tables from PDF files, fills forms, and merges documents. Use when PDF files, forms, or document extraction are mentioned.
---

# PDF Processing

## Quick Start

Extract text with `pdfplumber`:
```python
import pdfplumber
with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```

## Advanced Features

**Form filling**: See FORMS.md for the complete guide
**API reference**: See REFERENCE.md for all methods
**Examples**: See EXAMPLES.md for common patterns
````

The agent loads FORMS.md, REFERENCE.md, or EXAMPLES.md only when needed.

#### Pattern 2: Domain-Specific Organization

For skills spanning multiple domains, organize content by domain to avoid loading irrelevant context. If a user asks about sales metrics, the agent should read only sales-related schemas, not finance or marketing data. This reduces token usage and keeps context focused.

```
bigquery-skill/
├── SKILL.md (overview and navigation)
└── reference/
    ├── finance.md (revenue, billing metrics)
    ├── sales.md (opportunities, pipeline)
    ├── product.md (API usage, features)
    └── marketing.md (campaigns, attribution)
```

````markdown SKILL.md
# BigQuery Data Analysis

## Available Datasets

**Finance**: revenue, ARR, billing -> see reference/finance.md
**Sales**: opportunities, pipeline, accounts -> see reference/sales.md
**Product**: API usage, features, adoption -> see reference/product.md
**Marketing**: campaigns, attribution, email -> see reference/marketing.md

## Quick Search

Use `grep` to find specific metrics:

```bash
grep -i "revenue" reference/finance.md
grep -i "pipeline" reference/sales.md
grep -i "api usage" reference/product.md
```
````

#### Pattern 3: Conditional Details

Show basic content and link to advanced details:

```markdown
# DOCX Processing

## Document Creation

Use docx-js for new documents. See DOCX-JS.md.

## Document Editing

For simple edits, modify XML directly.

**For tracked changes**: See REDLINING.md
**For OOXML details**: See OOXML.md
```

The agent reads REDLINING.md or OOXML.md only when the user needs those features.

### Avoid Deeply Nested References

Agents may partially read files referenced by other files. When nested references appear, agents may preview with commands like `head -100`, which can lead to incomplete information.

**Keep references one level deep from SKILL.md.** Link all reference files directly from SKILL.md so the agent reads the full file when needed.

**Bad example: too deep**:
```markdown
# SKILL.md
See advanced.md...

# advanced.md
See details.md...

# details.md
The real info is here...
```

**Good example: one level deep**:
```markdown
# SKILL.md

**Basic usage**: instructions in SKILL.md
**Advanced features**: see advanced.md
**API reference**: see reference.md
**Examples**: see examples.md
```

### Use a Table of Contents for Long Reference Files

For reference files longer than 100 lines, add a table of contents at the top. This helps agents see the full scope even when previewing.

**Example**:
```markdown
# API Reference

## Table of Contents
- Authentication and setup
- Core methods (create, read, update, delete)
- Advanced features (batch operations, webhooks)
- Error handling patterns
- Code examples

## Authentication and setup
...

## Core methods
...
```

Agents can then read the full file or jump to a specific section as needed.

For details on how this filesystem-based architecture enables progressive disclosure, see the Runtime Environment section below.

## Workflows and Feedback Loops

### Use Workflows for Complex Tasks

Break complex operations into clear, ordered steps. For especially complex workflows, provide a checklist the agent can copy into its response to track progress.

**Example 1: Research Synthesis Workflow** (no-code skill):

````markdown
## Research Synthesis Workflow

Copy this checklist to track progress:

```
Research progress:
- [ ] Step 1: Read all source documents
- [ ] Step 2: Identify major themes
- [ ] Step 3: Cross-check claims
- [ ] Step 4: Create a structured summary
- [ ] Step 5: Verify citations
```

**Step 1: Read all source documents**

Review each document in the `sources/` directory. Note the key arguments and supporting evidence.

**Step 2: Identify major themes**

Look for patterns across sources. Which themes recur? Where do sources agree or disagree?

**Step 3: Cross-check claims**

For each major claim, confirm it appears in the source material. Note the sources that support each point.

**Step 4: Create a structured summary**

Organize findings by theme. Include:
- Primary claims
- Supporting evidence from sources
- Contrasting views (if any)

**Step 5: Verify citations**

Ensure every claim references the correct source documents. If citations are incomplete, return to Step 3.
````

This example shows how workflows apply to analysis tasks without code. Checklists work well for complex, multi-step processes.

**Example 2: PDF Form Filling Workflow** (code-enabled skill):

````markdown
## PDF Form Filling Workflow

Copy this checklist and check items as you complete them:

```
Task progress:
- [ ] Step 1: Analyze the form (run analyze_form.py)
- [ ] Step 2: Create the field mapping (edit fields.json)
- [ ] Step 3: Validate the mapping (run validate_fields.py)
- [ ] Step 4: Fill the form (run fill_form.py)
- [ ] Step 5: Verify the output (run verify_output.py)
```

**Step 1: Analyze the form**

Run: `python scripts/analyze_form.py input.pdf`

This extracts form fields and locations and saves them to `fields.json`.

**Step 2: Create the field mapping**

Edit `fields.json` to add values for each field.

**Step 3: Validate the mapping**

Run: `python scripts/validate_fields.py fields.json`

Fix validation errors before continuing.

**Step 4: Fill the form**

Run: `python scripts/fill_form.py input.pdf fields.json output.pdf`

**Step 5: Verify the output**

Run: `python scripts/verify_output.py output.pdf`

If validation fails, return to Step 2.
````

Clear steps prevent the agent from skipping critical validation. Checklists help both the agent and you track progress through multi-step workflows.

### Implement Feedback Loops

**Common pattern**: Run validator -> Fix errors -> Repeat

This pattern significantly improves output quality.

**Example 1: Style guide compliance** (no-code skill):

```markdown
## Content Review Process

1. Create content following the guidelines in STYLE_GUIDE.md
2. Review against the checklist:
   - Check terminology consistency
   - Verify examples follow the standard format
   - Confirm all required sections exist
3. If issues are found:
   - Note each issue with a specific section reference
   - Fix the content
   - Review the checklist again
4. Proceed only when all requirements are met
5. Finalize and save the document
```

This shows a validation loop pattern using a reference document instead of a script. The "validator" is STYLE_GUIDE.md, and the agent verifies by reading and comparing.

**Example 2: Document editing process** (code-enabled skill):

```markdown
## Document Editing Process

1. Make edits in `word/document.xml`
2. **Validate immediately**: `python ooxml/scripts/validate.py unpacked_dir/`
3. If validation fails:
   - Read the error message carefully
   - Fix XML issues
   - Run validation again
4. **Proceed only when validation succeeds**
5. Rebuild: `python ooxml/scripts/pack.py unpacked_dir/ output.docx`
6. Test the output document
```

Validation loops catch errors early.

## Content Guidelines

### Avoid Time-Sensitive Information

Do not include information that will become outdated:

**Bad example: time-sensitive** (will be wrong):
```markdown
If you are doing this before August 2025, use the old API.
If you are doing this after August 2025, use the new API.
```

**Good example** (use an "old patterns" section):
```markdown
## Current Method

Use the v2 API endpoint: `api.example.com/v2/messages`

## Old Pattern

<details>
<summary>Legacy v1 API (deprecated in 2025-08)</summary>

The v1 API used: `api.example.com/v1/messages`

This endpoint is no longer supported.
</details>
```

The old-pattern section preserves historical context without cluttering the main content.

### Use Consistent Terminology

Choose one term and use it consistently across the skill:

**Good - consistent**:
- Always "API endpoint"
- Always "field"
- Always "extract"

**Bad - inconsistent**:
- Mixing "API endpoint", "URL", "API route", "path"
- Mixing "field", "box", "element", "control"
- Mixing "extract", "pull", "fetch", "retrieve"

Consistency helps the agent understand and follow instructions.

## Common Patterns

### Template Pattern

Provide output format templates. Match the strictness to your needs.

**For strict requirements** (API responses, data formats, etc.):

````markdown
## Report Structure

Always use this exact template:

```markdown
# [Analysis Title]

## Executive Summary
[One-paragraph summary of key findings]

## Key Findings
- Finding 1 with supporting data
- Finding 2 with supporting data
- Finding 3 with supporting data

## Recommendations
1. Specific, actionable recommendation
2. Specific, actionable recommendation
```
````

**For flexible guidance** (adaptation is useful):

````markdown
## Report Structure

This is a reasonable default, but use your best judgment based on the analysis:

```markdown
# [Analysis Title]

## Executive Summary
[Summary]

## Key Findings
[Adapt sections based on findings]

## Recommendations
[Customize to the specific context]
```

Adjust sections as needed for specific analysis types.
````

### Example Pattern

When output quality depends on examples, provide input-output pairs like a normal prompt:

````markdown
## Commit Message Format

Generate commit messages following these examples:

**Example 1:**
Input: Added user authentication with JWT tokens
Output:
```
feat(auth): implement JWT-based authentication

Add login endpoint and token validation middleware
```

**Example 2:**
Input: Fixed a bug where dates display incorrectly in reports
Output:
```
fix(reports): correct date formatting in timezone conversion

Use UTC timestamps consistently across report generation
```

**Example 3:**
Input: Updated dependencies and refactored error handling
Output:
```
chore: update dependencies and refactor error handling

- Upgrade lodash to 4.17.21
- Standardize error response format across endpoints
```

Follow this style: type(scope): concise summary, then a detailed description.
````

Examples help the agent understand desired style and detail level more clearly than description alone.

### Conditional Workflow Pattern

Guide the agent through decision points:

```markdown
## Document Change Workflow

1. Decide the change type:

   **Creating new content?** -> Follow the "Creation Workflow" below
   **Editing existing content?** -> Follow the "Editing Workflow" below

2. Creation workflow:
   - Use the docx-js library
   - Build the document from scratch
   - Export to .docx format

3. Editing workflow:
   - Unpack the existing document
   - Edit XML directly
   - Validate after each change
   - Repack when finished
```

<Tip>
If a workflow grows large or complex, consider moving it to a separate file and instructing the agent to read the appropriate file based on the task.
</Tip>

## Evaluation and Iteration

### Build Evaluations First

**Create evaluations before writing extensive documentation.** This ensures the skill solves real problems, not imaginary ones.

**Evaluation-driven development:**
1. **Identify gaps**: Run the agent on representative tasks without the skill. Document specific failures or missing context.
2. **Create evaluations**: Build three scenarios that test those gaps.
3. **Establish a baseline**: Measure performance without the skill.
4. **Write minimal instructions**: Create only the content needed to address the gaps and pass evaluations.
5. **Iterate**: Run evaluations, compare to baseline, and improve.

This approach ensures you solve actual problems rather than anticipate requirements that may never occur.

**Evaluation structure**:
```json
{
  "skills": ["pdf-processing"],
  "query": "Extract all text from this PDF file and save it to output.txt",
  "files": ["test-files/document.pdf"],
  "expected_behavior": [
    "Successfully read the PDF using an appropriate PDF library or command-line tool",
    "Extract text content from every page without missing any",
    "Save the extracted text in a clear, readable format to a file named output.txt"
  ]
}
```

<Note>
This example shows a data-driven evaluation with a simple test rubric. There is currently no built-in way to run these evaluations. Users can build their own evaluation system. Evaluations are the source of truth for measuring skill effectiveness.
</Note>

### Use Agents to Iteratively Develop Skills

The most effective skill development process includes the agent itself. Work with one agent instance ("Agent A") to create a skill and use another instance ("Agent B") to test it. Agent A helps design and refine instructions, and Agent B tests them on real tasks. This works because the model understands both how to write effective agent instructions and what information agents need.

**Creating a new skill:**

1. **Complete the task without a skill**: Use Agent A and a normal prompt to solve the problem. As you work, you naturally provide context, explain settings, and share procedural knowledge. Notice the information you provide repeatedly.

2. **Identify reusable patterns**: After completing the task, identify the context you used that would help similar future tasks.

   **Example**: If you ran a BigQuery analysis, you may have provided table names, field definitions, filtering rules ("always exclude test accounts"), and common query patterns.

3. **Ask Agent A to create the skill**: "Create a skill that captures this BigQuery analysis pattern. Include table schemas, naming conventions, and rules for filtering test accounts."

   <Tip>
   The model understands skill format and structure natively. You do not need a special system prompt or a "skill creation" skill. Asking the agent to create a skill will produce properly structured SKILL.md content with frontmatter and body.
   </Tip>

4. **Review for conciseness**: Check whether Agent A added unnecessary explanation. Ask: "Remove the explanation of what win rate means since the agent already knows it."

5. **Improve information architecture**: Ask Agent A to reorganize content more effectively. For example: "Move the table schema to a separate reference file. We may add more tables later."

6. **Test on similar tasks**: Use Agent B (a new instance with the skill loaded) on relevant use cases. Observe whether Agent B finds the right information, applies rules correctly, and completes the task.

7. **Iterate based on observations**: If Agent B struggles or misses something, return to Agent A with specifics: "When the agent used this skill, it forgot to filter by Q4 dates. Do we need a section on date filtering patterns?"

**Iterating an existing skill:**

The same layered pattern applies when improving a skill. Alternate between:
- **Collaborating with Agent A** (the expert helping improve the skill)
- **Testing with Agent B** (the agent doing real work using the skill)
- **Observing Agent B's behavior** and bringing insights back to Agent A

1. **Use the skill in real workflows**: Provide a real task to Agent B (skill loaded), not just a test scenario.

2. **Observe Agent B's behavior**: Note where it struggles, succeeds, or makes unexpected choices.

   **Observation example**: "When I asked Agent B for a regional sales report, it wrote a query but forgot to filter test accounts, even though the skill mentions the rule."

3. **Return to Agent A for improvement**: Share the current SKILL.md and the observation. "Agent B forgot to filter test accounts. The skill mentions filtering, but it may not be prominent enough."

4. **Review Agent A's proposal**: Agent A might suggest rephrasing rules to be more prominent (e.g., "MUST filter" instead of "always filter"), or reorganizing the workflow section.

5. **Apply changes and test**: Update the skill with Agent A's improvements and retest with Agent B on similar requests.

6. **Repeat based on usage**: Continue this observe-improve-test cycle as new scenarios arise. Each iteration improves the skill based on observed behavior rather than assumptions.

**Collecting team feedback:**

1. Share the skill with teammates and observe usage
2. Ask: Does the skill activate when expected? Are the instructions clear? What is missing?
3. Incorporate feedback to address blind spots in your own usage patterns

**Why this works**: Agent A understands agent needs, you provide domain expertise, Agent B reveals gaps through real use, and iterative improvement is based on observed behavior rather than assumptions.

### Observe How Agents Navigate the Skill

As you iterate, pay attention to how agents actually use the skill. Watch for:

- **Unexpected exploration paths**: Does the agent read files in an unexpected order? This may indicate the structure is less intuitive than you thought.
- **Missing connections**: Does the agent fail to follow references to important files? Links may need to be more explicit or prominent.
- **Over-reliance on specific sections**: If the agent keeps reading the same file, that content might belong in the main SKILL.md instead.
- **Ignored content**: If the agent never accesses bundled files, they may be unnecessary or insufficiently signposted in the main instructions.

Iterate based on these observations, not assumptions. The skill metadata (`name` and `description`) is especially important. Agents use these fields to decide whether to trigger the skill based on the current task. Make sure they clearly explain what the skill does and when it should be used.

## Anti-Patterns to Avoid

### Avoid Windows-Style Paths

Always use forward slashes in file paths, even on Windows:

- ✓ **Good**: `scripts/helper.py`, `reference/guide.md`
- ✗ **Avoid**: `scripts\helper.py`, `reference\guide.md`

Unix-style paths work across all platforms, while Windows-style paths can cause errors on Unix systems.

### Avoid Offering Too Many Options

Do not present multiple approaches unless necessary:

````markdown
**Bad example: too many choices** (confusing):
"You can use pypdf or pdfplumber or PyMuPDF or pdf2image or ..."

**Good example: provide a default** (with an escape hatch):
"Use pdfplumber for text extraction:
```python
import pdfplumber
```

If you need OCR for scanned PDFs, use pdf2image and pytesseract instead."
````

## Advanced: Skills with Executable Code

The following sections focus on skills that include executable scripts. If your skill only uses Markdown instructions, skip to the Effective Skill Checklist.

### Solve, Do Not Dodge

When writing scripts for a skill, handle error conditions instead of leaving them to the agent.

**Good example: handle errors explicitly**:
```python
def process_file(path):
    """Process a file and create it if it does not exist."""
    try:
        with open(path) as f:
            return f.read()
    except FileNotFoundError:
        # Create file with default content instead of failing
        print(f"File {path} not found, creating a default")
        with open(path, 'w') as f:
            f.write('')
        return ''
    except PermissionError:
        # Offer a fallback instead of failing
        print(f"Cannot access {path}, using default")
        return ''
```

**Bad example: leave it to the agent**:
```python
def process_file(path):
    # Fail and let the agent figure it out
    return open(path).read()
```

Configuration parameters must be justified and documented. If the correct value is unknown, how will the agent decide?

**Good example: self-documenting**:
```python
# HTTP requests typically complete within 30 seconds
# A longer timeout accounts for slow connections
REQUEST_TIMEOUT = 30

# Three retries balance reliability and speed
# Most transient failures resolve by the second retry
MAX_RETRIES = 3
```

**Bad example: magic numbers**:
```python
TIMEOUT = 47  # Why 47?
RETRIES = 5   # Why 5?
```

### Provide Utility Scripts

Even if the agent could write scripts, prebuilt scripts have advantages:

**Advantages of utility scripts**:
- More reliable than generated code
- Save tokens (no need to include code in context)
- Save time (no code generation required)
- Ensure consistency across usage

Executable scripts can work alongside instruction files. Instruction files (forms.md) can reference scripts, and the agent can execute them without loading their content into context.

**Important distinction**: Be explicit about what the agent should do.
- **Execute the script** (most common): "Run analyze_form.py to extract fields"
- **Read as a reference** (for complex logic): "See analyze_form.py for the field extraction algorithm"

For most utility scripts, execution is more reliable and efficient, so prefer running them. For details on script execution, see the Runtime Environment section below.

**Example**:
````markdown
## Utility Scripts

**analyze_form.py**: Extracts all form fields from a PDF

```bash
python scripts/analyze_form.py input.pdf > fields.json
```

Output format:
```json
{
  "field_name": {"type": "text", "x": 100, "y": 200},
  "signature": {"type": "sig", "x": 150, "y": 500}
}
```

**validate_boxes.py**: Checks for overlapping bounding boxes

```bash
python scripts/validate_boxes.py fields.json
# Returns: "OK" or a list of conflicts
```

**fill_form.py**: Applies field values to the PDF

```bash
python scripts/fill_form.py input.pdf fields.json output.pdf
```
````

### Use Visual Analysis

If inputs can be rendered as images, have the agent analyze them:

````markdown
## Form Layout Analysis

1. Convert the PDF to images:
   ```bash
   python scripts/pdf_to_images.py form.pdf
   ```

2. Analyze each page image to identify form fields
3. The agent can visually verify field locations and types
````

<Note>
This example requires creating a `pdf_to_images.py` script.
</Note>

Vision capabilities help understand layout and structure.

### Create Verifiable Intermediate Outputs

When agents handle complex, open-ended tasks, mistakes are possible. The "plan-validate-execute" pattern catches errors early by having the agent create a structured plan, validate it with a script, and then execute it.

**Example**: Imagine asking an agent to update 50 PDF form fields based on a spreadsheet. Without validation, it may reference non-existent fields, create conflicting values, miss required fields, or apply updates incorrectly.

**Solution**: Use the workflow pattern above, but add an intermediate `changes.json` file that is validated before applying changes. The workflow becomes: analyze -> **create plan file** -> **validate plan** -> execute -> verify.

**Why this works:**
- **Catch errors early**: Validation finds problems before changes are applied
- **Machine-verifiable**: Scripts provide objective checks
- **Reversible planning**: Agents can iterate on the plan without touching the original files
- **Clear debugging**: Error messages point to specific issues

**Use for**: Batch operations, destructive changes, complex validation rules, high-risk operations.

**Implementation tip**: Use specific error messages like "Field 'signature_date' not found. Available fields: customer_name, order_total, signature_date_signed" to help the agent fix issues.

### Package Dependencies

Skills run in code execution environments with platform-specific limitations:

- **Host environment**: Can install packages from npm and PyPI, and pull from GitHub repos
- **Limited API environment**: No network access and no runtime package installation

List required packages in SKILL.md and verify they are available in the code execution tools for your environment.

### Runtime Environment

Skills run in a code execution environment with filesystem access, command execution, and code execution. See your platform overview for a conceptual description of this architecture.

**How this affects authoring:**

**How agents access skills:**

1. **Metadata preloading**: At startup, all skill names and descriptions from YAML frontmatter are loaded into the system prompt
2. **On-demand file reads**: Agents use read tools to access SKILL.md and other files as needed
3. **Efficient script execution**: Utility scripts can be run via commands without loading full content into context; only script output consumes tokens
4. **No context penalty for large files**: Reference files, data, or documents consume context tokens only when actually read

- **File paths matter**: Agents navigate skill directories like a filesystem. Use forward slashes (`reference/guide.md`) and never backslashes
- **Use descriptive filenames**: Names should indicate content, e.g., `form_validation_rules.md` instead of `doc2.md`
- **Organize for discovery**: Structure directories by domain or function
  - Good: `reference/finance.md`, `reference/sales.md`
  - Bad: `docs/file1.md`, `docs/file2.md`
- **Bundle comprehensive resources**: Include full API docs, extensive examples, large datasets. They cost no context tokens until accessed
- **Prefer scripts for deterministic operations**: Create `validate_form.py` rather than asking the agent to generate code
- **Make execution intent explicit**:
  - "Run analyze_form.py to extract fields" (execute)
  - "See analyze_form.py for the extraction algorithm" (read as reference)
- **Test file access patterns**: Use real requests to ensure the agent can navigate the directory structure

**Example:**

```
bigquery-skill/
├── SKILL.md (overview, references files)
└── reference/
    ├── finance.md (revenue metrics)
    ├── sales.md (pipeline data)
    └── product.md (usage analysis)
```

If a user asks about revenue, the agent reads SKILL.md, sees the reference to `reference/finance.md`, and reads only that file. The sales.md and product.md files remain on disk and consume zero context tokens until needed. This filesystem-based model enables progressive disclosure by letting the agent selectively load exactly what each task requires.

For full technical architecture details, refer to your platform overview.

### MCP Tool References

If a skill uses MCP (Model Context Protocol) tools, always use fully qualified tool names to avoid "tool not found" errors.

**Format**: `ServerName:tool_name`

**Examples**:
```markdown
Use the BigQuery:bigquery_schema tool to fetch table schemas.
Use the GitHub:create_issue tool to create an issue.
```

Where:
- `BigQuery` and `GitHub` are MCP server names
- `bigquery_schema` and `create_issue` are tool names within those servers

Without the server prefix, agents may not be able to locate the tool, especially when multiple MCP servers are available.

### Do Not Assume Tools Are Installed

Do not assume packages are available:

````markdown
**Bad example: assume installation**:
"Process files using the pdf library."

**Good example: explicit dependencies**:
"Install the required package: `pip install pypdf`

Then use it:
```python
from pypdf import PdfReader
reader = PdfReader("file.pdf")
```"
````

## Technical Notes

### YAML Frontmatter Requirements

SKILL.md frontmatter requires `name` and `description` with specific validation rules:
- `name`: max 64 characters, lowercase/number/hyphen only, no XML tags, no reserved words
- `description`: max 1024 characters, non-empty, no XML tags

Refer to your platform skill overview for full structure details.

### Token Budget

Keep the SKILL.md body under 500 lines for optimal performance. If content exceeds this, split it into separate files using the progressive disclosure pattern above. For architecture details, refer to your platform overview.

## Effective Skill Checklist

Before sharing a skill, verify the following:

### Core Quality
- [ ] Description is specific and includes key terms
- [ ] Description includes what the skill does and when to use it
- [ ] SKILL.md body is under 500 lines
- [ ] Additional details live in separate files (if needed)
- [ ] No time-sensitive information (or it is in an "old patterns" section)
- [ ] Terminology is consistent throughout
- [ ] Examples are concrete, not abstract
- [ ] File references are one level deep
- [ ] Progressive disclosure is used appropriately
- [ ] Workflow steps are clear

### Code and Scripts
- [ ] Scripts solve the problem instead of deferring to the agent
- [ ] Error handling is explicit and helpful
- [ ] No "voodoo constants" (all values are justified)
- [ ] Required packages are listed and verified as available
- [ ] Scripts have clear documentation
- [ ] No Windows-style paths (all forward slashes)
- [ ] Validation/confirmation steps for critical operations
- [ ] Feedback loops for quality-critical tasks

### Testing
- [ ] At least three evaluations are created
- [ ] Tested on all intended models
- [ ] Tested in real usage scenarios
- [ ] Team feedback incorporated (if applicable)

