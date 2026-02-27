---
name: skill-builder
description: Expert guidance for creating, writing, building, and refining Claude Code Skills. Use when working with SKILL.md files, authoring new skills, improving existing skills, or understanding skill structure, progressive disclosure, workflows, validation patterns, and XML formatting.
allowed-tools:
  - Read
  - Write
  - Glob
  - Bash
  - AskUserQuestion
---

<objective>
Agent Skills are modular, filesystem-based capabilities that provide domain-specific expertise through progressive disclosure. This Skill teaches you how to create effective Skills that Claude can discover and use successfully.

Skills are organized prompts that get loaded on-demand. All prompting best practices apply, with an emphasis on pure XML structure for consistent parsing and efficient token usage.
</objective>

<quick_start>
<workflow>
1. **Gather requirements**: Ask targeted questions to understand scope, complexity, and specific needs
2. **Detect project context**: Determine if skill should be project-specific or global
3. **Research if needed**: For external APIs/standards, offer to fetch current documentation
4. **Create directory and SKILL.md**:
   - Directory name: Follow verb-noun convention: `create-*`, `manage-*`, `setup-*`, `generate-*`
   - YAML frontmatter: `name` (matches directory), `description` (third person, triggers)
5. **Write concise instructions**: Use pure XML structure, appropriate tags for complexity level
6. **Apply progressive disclosure**: Keep SKILL.md under 500 lines, split details to reference files
7. **Validate structure**: Run automated checks (YAML, line count, required tags, naming)
8. **Test with real scenario**: Invoke skill to verify guidance is clear and actionable
9. **Create slash command**: Lightweight wrapper at correct location (project or global)

See [references/skill-structure.md](references/skill-structure.md) for complete details.
</workflow>

<example_skill>
```yaml
---
name: process-pdfs
description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
---

<objective>
Extract text and tables from PDF files, fill forms, and merge documents using Python libraries.
</objective>

<quick_start>
Extract text with pdfplumber:

```python
import pdfplumber
with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```
</quick_start>

<advanced_features>
**Form filling**: See [forms.md](forms.md)
**API reference**: See [reference.md](reference.md)
</advanced_features>
```
</example_skill>
</quick_start>

<xml_structure>
<required_tags>
All skills must have these tags:

- **`<objective>`** - What the skill does and why it matters
- **`<quick_start>`** - Immediate, actionable guidance
- **`<success_criteria>`** or **`<when_successful>`** - How to know it worked
</required_tags>

<conditional_tags>
Based on skill complexity, add these tags as needed:

- **`<context>`** - Background/situational information
- **`<workflow>` or `<process>`** - Step-by-step procedures
- **`<advanced_features>`** - Deep-dive topics (progressive disclosure)
- **`<validation>`** - How to verify outputs
- **`<examples>`** - Multi-shot learning
- **`<anti_patterns>`** - Common mistakes to avoid
- **`<security_checklist>`** - Non-negotiable security patterns
- **`<testing>`** - Testing workflows
- **`<common_patterns>`** - Code examples and recipes
- **`<reference_guides>` or `<detailed_references>`** - Links to reference files
</conditional_tags>

<critical_rule>
**Remove ALL markdown headings (#, ##, ###) from skill body content.** Replace with semantic XML tags. Keep markdown formatting WITHIN content (bold, italic, lists, code blocks, links).
</critical_rule>
</xml_structure>

<intelligence_rules>
<simple_skills>
Single domain, straightforward tasks: Use required tags only.

Example: Text extraction, file format conversion, simple API calls
</simple_skills>

<medium_skills>
Multiple patterns, some complexity: Use required tags + workflow/examples as needed.

Example: Document processing with multiple steps, API integration with configuration
</medium_skills>

<complex_skills>
Multiple domains, security concerns, API integrations: Use required tags + conditional tags as appropriate.

Example: Payment processing, authentication systems, multi-step workflows with validation
</complex_skills>

<principle>
Don't over-engineer simple skills. Don't under-specify complex skills. Match tag selection to actual complexity and user needs.
</principle>
</intelligence_rules>

<generation_protocol>
<step_0>
<description>
**Adaptive Requirements Gathering**: Before building, gather requirements through intelligent questioning that infers obvious details and asks about genuine gaps.
</description>

<critical_first_action>
**BEFORE doing anything else**, check if context was provided.

IF no context provided (user just invoked the skill without describing what to build):
→ **IMMEDIATELY use AskUserQuestion** with these exact options:

1. **Create a new skill** - Build a skill from scratch
2. **Update an existing skill** - Modify or improve an existing skill
3. **Get guidance on skill design** - Help me think through what kind of skill I need

**DO NOT** ask "what would you like to build?" in plain text. **USE the AskUserQuestion tool.**

Routing after selection:
- "Create new" → proceed to adaptive intake below
- "Update existing" → enumerate existing skills as numbered list (see below), then gather requirements for changes
- "Guidance" → help user clarify needs before building

<update_existing_flow>
When "Update existing" is selected:

1. **List all skills in chat as numbered list** (DO NOT use AskUserQuestion - there may be many skills):
   - Glob for `~/skills/*/SKILL.md`
   - Present as numbered list in chat:
     ```
     Available skills:
     1. create-agent-skills
     2. generate-natal-chart
     3. manage-stripe
     ...
     ```
   - Ask: "Which skill would you like to update? (enter number)"

2. After user enters number, read that skill's SKILL.md
3. Ask what they want to change/improve using AskUserQuestion or direct question
4. Proceed with modifications
</update_existing_flow>

IF context was provided (user said "build a skill for X"):
→ Skip this gate. Proceed directly to adaptive intake.
</critical_first_action>

<adaptive_intake>
<analyze_description>
Parse the initial description:
- What's explicitly stated (operations, services, outputs)
- What can be inferred without asking (skill type, complexity, obvious patterns)
- What's genuinely unclear or ambiguous (scope boundaries, edge cases, specific behaviors)

Do NOT ask about things that are obvious from context.
</analyze_description>

<generate_questions>
Use AskUserQuestion to ask 2-4 domain-specific questions based on actual gaps.

Question generation guidance:
- **Scope questions**: "What specific operations?" not "What should it do?"
- **Complexity questions**: "Should this handle [specific edge case]?" based on domain knowledge
- **Output questions**: "What should the user see/get when successful?"
- **Boundary questions**: "Should this also handle [related thing] or stay focused?"

Avoid:
- Questions answerable from the initial description
- Generic questions that apply to all skills
- Yes/no questions when options would be more helpful
- Obvious questions like "what should it be called?" when the name is clear

Each question option should include a description explaining the implications of that choice.
</generate_questions>

<decision_gate>
After receiving answers, present decision gate using AskUserQuestion:

Question: "Ready to proceed with building, or would you like me to ask more questions?"

Options:
1. **Proceed to building** - I have enough context to build the skill
2. **Ask more questions** - There are more details to clarify
3. **Let me add details** - I want to provide additional context

If "Ask more questions" selected → loop back to generate_questions with refined focus
If "Let me add details" → receive additional context, then re-evaluate
If "Proceed" → continue to research_trigger, then step_1
</decision_gate>

<research_trigger>
<detection_patterns>
Detect if the skill involves ANY of the following:

**External APIs or web services:**
- Keywords: "API", "endpoint", "REST", "GraphQL", "webhook", "HTTP"
- Service names: "Stripe", "AWS", "Firebase", "OpenAI", "Anthropic", etc.

**Standard vocabularies and ontologies:**
- Keywords: "schema.org", "vocabulary", "ontology", "JSON-LD", "RDF", "microdata"
- Examples: Schema.org types, Dublin Core, FOAF

**Protocol specifications:**
- Keywords: "specification", "standard", "protocol"
- Examples: "HTTP", "WebSocket", "MQTT", "OAuth", "SAML"

**Third-party libraries or frameworks:**
- Keywords: "library", "package", "framework", "npm", "pip", "cargo"
- Examples: "React", "Django", "pandas", "TensorFlow"

**Data format standards:**
- Keywords: "format", "parser", "serialization"
- Examples: "CSV", "Parquet", "Protocol Buffers", "Avro", "XML"

Detection method: Check user's initial description and all collected requirements for these keywords or patterns.
</detection_patterns>

<research_prompt>
When detected, use AskUserQuestion:

"This skill involves [detected technology/standard]. Would you like me to research current [technology] documentation and patterns before building?"

Options:
1. **Yes, research first** - Fetch 2024-2025 documentation for accurate, up-to-date implementation
2. **No, proceed with general patterns** - Use knowledge cutoff data (January 2025)
3. **I'll provide the documentation** - User will supply relevant documentation or links

If option 1 selected:
- For web APIs: Use WebSearch for "[technology] API documentation 2024 2025"
- For libraries: Use Context7 MCP if available, otherwise WebSearch
- For standards/vocabularies: Use WebSearch for "[standard] specification latest"
- Focus on: current versions, recent changes, migration guides, common patterns
- Summarize findings in internal notes for use in skill generation

If option 3 selected:
- Wait for user to provide documentation
- Read provided links or files
- Summarize key information for skill generation
</research_prompt>

<research_findings_usage>
Incorporate research findings into:
- **Step 1** (Domain analysis): Update with current patterns and best practices
- **Step 4** (Content writing): Use current syntax, API endpoints, method signatures
- **Step 5** (Reference files): Include links to up-to-date documentation
- **Examples and code blocks**: Ensure all code examples use current versions

Note findings in skill generation process:
```
Research findings for [technology]:
- Current version: [version]
- Key changes from knowledge cutoff: [summary]
- Recommended patterns: [list]
- Documentation links: [urls]
```
</research_findings_usage>
</research_trigger>
</adaptive_intake>
</step_0>

<step_0_5>
<description>**Project Context Detection**: Determine if skill should be project-specific or global.</description>

<detection_logic>
Check for project indicators in current working directory or parent directories:

1. **Primary indicators** (strong project context):
   - `CLAUDE.md` file exists
   - `.claude/` directory exists
   - Git repository root (`.git/` directory)

2. **Secondary indicators** (language-specific projects):
   - `package.json` (Node.js)
   - `pyproject.toml` or `setup.py` (Python)
   - `Cargo.toml` (Rust)
   - `pom.xml` or `build.gradle` (Java)
   - `go.mod` (Go)

3. **Path determination**:
   ```bash
   # Use Bash to check project context:
   if [ -f "CLAUDE.md" ] || [ -d ".claude" ]; then
     PROJECT_CONTEXT=true
     PROJECT_ROOT=$(pwd)
   elif git rev-parse --git-dir > /dev/null 2>&1; then
     PROJECT_CONTEXT=true
     PROJECT_ROOT=$(git rev-parse --show-toplevel)
   else
     PROJECT_CONTEXT=false
   fi
   ```

4. **Set paths based on context**:
   - If `PROJECT_CONTEXT=true`:
     - `SKILLS_PATH="$PROJECT_ROOT/skills/"`
     - `COMMANDS_PATH="$PROJECT_ROOT/commands/"`
   - If `PROJECT_CONTEXT=false`:
     - `SKILLS_PATH="$HOME/skills/"`
     - `COMMANDS_PATH="$HOME/commands/"`
</detection_logic>

<user_confirmation>
If project context detected, use AskUserQuestion:

"Detected project context at: [project_root]

Where should this skill be created?"

Options:
1. **Project-specific** (skills/) - Tracked in git, shared with team, specific to this codebase
2. **Global** (~/skills/) - Personal use across all projects, general-purpose skill

Provide recommendation based on skill purpose:
- If skill mentions project-specific terms (file paths, module names, codebase concepts) → Recommend project-specific
- If skill is general-purpose (works with any codebase) → Recommend global
- If user explicitly mentioned "for this project" → Recommend project-specific
- Default recommendation: Project-specific (safer, can be moved to global later)
</user_confirmation>

<path_usage>
Use the determined paths for all file operations:

- **Step 0.5**: Create directory at `$SKILLS_PATH/[skill-name]/`
- **Step 3-5**: Write SKILL.md to `$SKILLS_PATH/[skill-name]/SKILL.md`
- **Step 5**: Create references at `$SKILLS_PATH/[skill-name]/references/`
- **Step 8**: Create slash command at `$COMMANDS_PATH/[skill-name].md`
- **Step 8.5** (if applicable): Create README at `$SKILLS_PATH/[skill-name]/README.md`

All subsequent file operations MUST use these paths, not hardcoded paths.
</path_usage>

<no_project_context>
If NO project context detected:
- Skip user confirmation
- Use global paths: `~/skills/` and `~/commands/`
- Inform user: "Creating global skill at ~/skills/[skill-name]/"
</no_project_context>
</step_0_5>

<step_1>
**Analyze the domain**: Understand what the skill needs to teach and its complexity level. Incorporate gathered requirements and any research findings from step_0.
</step_1>

<step_2>
**Select XML tags**: Choose required tags + conditional tags based on intelligence rules.
</step_2>

<step_3>
**Write YAML frontmatter**: Validate name (matches directory, verb-noun convention) and description (third person, includes triggers).
</step_3>

<step_4>
**Structure content in pure XML**: No markdown headings in body. Use semantic XML tags for all sections.
</step_4>

<step_5>
**Apply progressive disclosure**: Keep SKILL.md under 500 lines. Split detailed content into reference files.
</step_5>

<step_6>
<description>**Validate Structure**: Run automated checks to ensure skill meets all requirements.</description>

<validation_execution>
Execute the following validation checks and report results:

**1. File Structure Checks:**
```bash
# Check SKILL.md exists
[ -f "$SKILLS_PATH/$SKILL_NAME/SKILL.md" ] && echo "✅ SKILL.md exists" || echo "❌ SKILL.md missing"

# Check directory name matches YAML name
YAML_NAME=$(head -20 "$SKILLS_PATH/$SKILL_NAME/SKILL.md" | grep "^name:" | cut -d: -f2 | tr -d ' ')
[ "$YAML_NAME" = "$SKILL_NAME" ] && echo "✅ Name matches directory" || echo "⚠️  Name mismatch"
```

**2. YAML Frontmatter Validation:**
```bash
# Extract and validate YAML
head -20 "$SKILLS_PATH/$SKILL_NAME/SKILL.md" | python3 -c "
import yaml, sys
try:
    doc = yaml.safe_load(sys.stdin)
    print('✅ YAML syntax valid')

    # Check required fields
    if 'name' in doc:
        print(f'✅ Name field present: {doc[\"name\"]}')
        if len(doc['name']) <= 64:
            print(f'✅ Name length OK: {len(doc[\"name\"])} chars')
        else:
            print(f'⚠️  Name too long: {len(doc[\"name\"])} chars (max 64)')
    else:
        print('❌ Name field missing')

    if 'description' in doc:
        print(f'✅ Description field present: {len(doc[\"description\"])} chars')
        if len(doc['description']) <= 1024:
            print(f'✅ Description length OK')
        else:
            print(f'⚠️  Description too long: {len(doc[\"description\"])} chars (max 1024)')
    else:
        print('❌ Description field missing')

except yaml.YAMLError as e:
    print(f'❌ YAML parse error: {e}')
"
```

**3. Line Count Check:**
```bash
LINE_COUNT=$(wc -l < "$SKILLS_PATH/$SKILL_NAME/SKILL.md")
echo "SKILL.md line count: $LINE_COUNT"
if [ $LINE_COUNT -lt 500 ]; then
    echo "✅ Line count under 500 limit ($LINE_COUNT lines, $(( (500 - LINE_COUNT) )) remaining)"
else
    echo "⚠️  Line count exceeds 500 limit ($LINE_COUNT lines, $(( (LINE_COUNT - 500) )) over)"
fi
```

**4. XML Structure Checks:**
```bash
# Check for markdown headings
HEADING_COUNT=$(grep -c '^#' "$SKILLS_PATH/$SKILL_NAME/SKILL.md" || echo 0)
if [ $HEADING_COUNT -eq 0 ]; then
    echo "✅ No markdown headings found"
else
    echo "⚠️  Found $HEADING_COUNT markdown headings - should use XML tags instead"
    grep -n '^#' "$SKILLS_PATH/$SKILL_NAME/SKILL.md"
fi

# Check required tags
for TAG in "objective" "quick_start" "success_criteria"; do
    if grep -q "<$TAG>" "$SKILLS_PATH/$SKILL_NAME/SKILL.md"; then
        echo "✅ Required tag <$TAG> present"
    else
        if [ "$TAG" = "success_criteria" ]; then
            if grep -q "<when_successful>" "$SKILLS_PATH/$SKILL_NAME/SKILL.md"; then
                echo "✅ Alternative tag <when_successful> present"
            else
                echo "❌ Missing required tag: <$TAG> or <when_successful>"
            fi
        else
            echo "❌ Missing required tag: <$TAG>"
        fi
    fi
done

# List all XML tags found
echo "XML tags found:"
grep -oE '<[a-z_]+>' "$SKILLS_PATH/$SKILL_NAME/SKILL.md" | sort | uniq -c | sort -rn
```

**5. Progressive Disclosure Check:**
```bash
if [ $LINE_COUNT -gt 300 ]; then
    if [ -d "$SKILLS_PATH/$SKILL_NAME/references" ]; then
        REF_COUNT=$(find "$SKILLS_PATH/$SKILL_NAME/references" -name "*.md" | wc -l)
        echo "✅ Progressive disclosure: $REF_COUNT reference files created"
    else
        echo "⚠️  SKILL.md is $LINE_COUNT lines but no reference files found - consider splitting"
    fi
fi
```

**6. Reference Link Validation:**
```bash
# Check all reference links exist
grep -oE '\[.*\]\([^)]+\.md\)' "$SKILLS_PATH/$SKILL_NAME/SKILL.md" | while read link; do
    FILE=$(echo "$link" | sed 's/.*(\(.*\))/\1/')
    if [ -f "$SKILLS_PATH/$SKILL_NAME/$FILE" ]; then
        echo "✅ Reference link valid: $FILE"
    else
        echo "⚠️  Broken reference link: $FILE"
    fi
done
```

**7. Naming Convention Check:**
```bash
# Check verb-noun pattern
SKILL_FIRST_WORD=$(echo "$SKILL_NAME" | cut -d- -f1)
COMMON_VERBS="create manage setup generate analyze process coordinate build handle configure deploy execute extract transform validate parse render compile"

if echo "$COMMON_VERBS" | grep -qw "$SKILL_FIRST_WORD"; then
    echo "✅ Naming convention: '$SKILL_FIRST_WORD' follows verb-noun pattern"
else
    echo "⚠️  Naming: '$SKILL_FIRST_WORD' is not a common verb - verify it's action-oriented"
fi
```
</validation_execution>

<validation_report>
After running checks, present a consolidated validation report:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Skill Validation Report: [skill-name]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

File Structure:     [✅/❌] [details]
YAML Frontmatter:   [✅/❌] [details]
Line Count:         [✅/⚠️]  [X lines / 500 limit]
Required Tags:      [✅/❌] [3/3 present]
No MD Headings:     [✅/⚠️]  [X found]
XML Structure:      [✅/❌] [tags properly nested]
Progressive Disc:   [✅/⚠️]  [X ref files]
Reference Links:    [✅/⚠️]  [all valid / X broken]
Naming Convention:  [✅/⚠️]  [verb-noun pattern]
Slash Command:      [✅/❌] [exists at correct path]

Overall Status: [PASS ✅ / NEEDS FIXES ⚠️  / FAIL ❌]

[If issues found, list them with suggested fixes]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
</validation_report>

<failure_handling>
If any critical validations fail (❌):

1. **Stop immediately** - Do not proceed to Step 7
2. **Report failures** clearly with specific line numbers or details
3. **Attempt automatic fixes** where possible:
   - Remove markdown headings → convert to XML tags
   - Fix YAML syntax errors if obvious
   - Rename files to match conventions
4. **If cannot auto-fix**, ask user:
   ```
   Validation failed. How should I proceed?

   Options:
   1. Let me try to fix automatically
   2. Show me the issues and I'll fix manually
   3. Proceed anyway (not recommended)
   ```
5. **Re-run validation** after fixes applied
6. **Only proceed to Step 7** when all critical checks pass

**Warning-level issues** (⚠️) can proceed but should be noted:
- Line count 400-500: "Approaching limit, future edits may need reference split"
- Uncommon verb in name: "Ensure name is clear and action-oriented"
- No reference files when >300 lines: "Consider splitting for better progressive disclosure"
</failure_handling>

<quick_validation_mode>
For simple skills or rapid iteration, offer quick validation:

"Run full validation checks or quick validation?"

Quick validation only checks:
- YAML frontmatter valid
- Required tags present
- No markdown headings
- Line count < 500

Full validation runs all 7 check categories above.
</quick_validation_mode>
</step_6>

<step_7>
<description>**Test with Real Usage**: Verify the skill works as expected through actual invocation.</description>

<test_offer>
After skill creation and validation, use AskUserQuestion:

"The [skill-name] skill has been created and validated. How would you like to proceed?"

Options:
1. **Test it now** - I'll provide a sample scenario to test the skill guidance
2. **Skip testing** - Test it later during actual usage
3. **You provide test scenario** - I'll use your scenario to test the skill

Recommendation: Option 1 (testing now catches issues before delivery)
</test_offer>

<sample_scenario_generation>
If user selects "Test it now", generate a realistic test scenario based on skill purpose:

**For different skill types:**

- **API integration skills** (e.g., manage-stripe):
  ```
  Scenario: You need to create a new subscription for a customer.
  Let's invoke the skill and see what guidance it provides.
  ```

- **Code processing skills** (e.g., coordinate-subagents):
  ```
  Scenario: You need to find all files handling user authentication.
  Let's use the skill to craft an efficient subagent prompt.
  ```

- **Data transformation skills** (e.g., process-csv):
  ```
  Scenario: You have a CSV file with 10,000 rows and need to filter
  and transform specific columns. Let's test the skill's guidance.
  ```

- **Setup/configuration skills** (e.g., setup-testing):
  ```
  Scenario: You need to add testing to a new TypeScript project.
  Let's see what the skill recommends.
  ```

Generate scenario that exercises the skill's primary workflow.
</sample_scenario_generation>

<skill_invocation>
After scenario generation:

1. **Invoke the skill** using the Skill tool:
   ```
   Skill: [skill-name]
   Context: [generated scenario]
   ```

2. **Observe the response**:
   - Is the guidance clear and actionable?
   - Are the steps in logical order?
   - Are examples helpful and realistic?
   - Is any critical information missing?
   - Are there any confusing or ambiguous instructions?

3. **Document observations**:
   ```
   Test Observations:
   ✅ Clear: [what worked well]
   ⚠️  Unclear: [what was confusing]
   ❌ Missing: [what information was needed but not provided]
   💡 Suggestions: [potential improvements]
   ```
</skill_invocation>

<iteration_prompt>
After testing and observation, ask user:

"Based on the test, the skill guidance seems [assessment]. Would you like to iterate?"

Options:
1. **Yes, make improvements** - I'll update based on testing observations
2. **No, it's good enough** - Proceed to Step 8 (slash command creation)
3. **Show me the issues first** - Let me review before deciding

If option 1 selected:
- List specific improvements to make
- Make edits to SKILL.md or reference files
- Re-run validation (Step 6)
- Offer to test again with same or different scenario

If option 2 or 3 selected:
- Proceed to Step 8
</iteration_prompt>

<user_provided_scenario>
If user selected "You provide test scenario":

1. **Wait for user scenario**: "Please describe the scenario you'd like to test with."

2. **Acknowledge scenario**: "I'll test the skill with: [user scenario]"

3. **Invoke skill** with user's scenario

4. **Follow observation and iteration process** as above
</user_provided_scenario>

<skip_testing>
If user selected "Skip testing":

1. **Note in output**: "Testing skipped - recommend testing during first actual use"

2. **Provide testing reminder**:
   ```
   When you first use this skill, observe:
   - Is the guidance immediately actionable?
   - Are there any unclear instructions?
   - Is any critical information missing?

   If issues found, you can improve the skill by editing:
   $SKILLS_PATH/[skill-name]/SKILL.md
   ```

3. **Proceed to Step 8**
</skip_testing>

<testing_benefits>
Benefits of testing before delivery:
- **Catches unclear instructions** before user encounters them
- **Validates that examples are realistic** and actually helpful
- **Ensures workflow steps are logical** and in correct order
- **Identifies missing information** that domain experts might assume
- **Improves skill quality** immediately, not after user frustration
- **Builds confidence** that skill will work when needed

Testing adds 2-5 minutes but can save hours of confusion later.
</testing_benefits>

<common_testing_findings>
Issues frequently discovered during testing:

1. **Missing context**: Skill assumes knowledge user doesn't have
   - Fix: Add `<context>` section with background

2. **Steps out of order**: Workflow jumps around
   - Fix: Reorder steps to follow actual execution sequence

3. **Examples too abstract**: Code examples don't match real use cases
   - Fix: Use more realistic, specific examples

4. **Missing error handling**: Doesn't address what to do when things fail
   - Fix: Add `<troubleshooting>` or error handling guidance

5. **Terminology mismatch**: Skill uses different terms than user expects
   - Fix: Add `<terminology>` section or adjust language

6. **Too verbose or too terse**: Wrong level of detail
   - Fix: Adjust based on complexity (intelligence rules)
</common_testing_findings>
</step_7>

<step_8>
**Create slash command wrapper**: Create a lightweight slash command that invokes the skill.

Location: `$COMMANDS_PATH/{skill-name}.md` (determined in Step 0.5)

Template:
```yaml
---
description: {Brief description of what the skill does}
argument-hint: [{argument description}]
allowed-tools: Skill({skill-name})
---

<objective>
Delegate {task} to the {skill-name} skill for: $ARGUMENTS

This routes to specialized skill containing patterns, best practices, and workflows.
</objective>

<process>
1. Use Skill tool to invoke {skill-name} skill
2. Pass user's request: $ARGUMENTS
3. Let skill handle workflow
</process>

<success_criteria>
- Skill successfully invoked
- Arguments passed correctly to skill
</success_criteria>
```

The slash command's only job is routing—all expertise lives in the skill.
</step_8>
</generation_protocol>

<yaml_requirements>
<required_fields>
```yaml
---
name: skill-name-here
description: What it does and when to use it (third person, specific triggers)
---
```
</required_fields>

<validation_rules>
See [references/skill-structure.md](references/skill-structure.md) for complete validation rules and naming conventions.
</validation_rules>
</yaml_requirements>

<when_to_use>
<create_skills_for>
- Reusable patterns across multiple tasks
- Domain knowledge that doesn't change frequently
- Complex workflows that benefit from structured guidance
- Reference materials (schemas, APIs, libraries)
- Validation scripts and quality checks
</create_skills_for>

<use_prompts_for>
One-off tasks that won't be reused
</use_prompts_for>

<use_slash_commands_for>
Explicit user-triggered workflows that run with fresh context
</use_slash_commands_for>
</when_to_use>

<reference_guides>
For deeper topics, see reference files:

**Core principles**: [references/core-principles.md](references/core-principles.md)
- XML structure (consistency, parseability, Claude performance)
- Conciseness (context window is shared)
- Degrees of freedom (matching specificity to task fragility)
- Model testing (Haiku vs Sonnet vs Opus)

**Skill structure**: [references/skill-structure.md](references/skill-structure.md)
- XML structure requirements
- Naming conventions
- Writing effective descriptions
- Progressive disclosure patterns
- File organization

**Workflows and validation**: [references/workflows-and-validation.md](references/workflows-and-validation.md)
- Complex workflows with checklists
- Feedback loops (validate → fix → repeat)
- Plan-validate-execute pattern

**Common patterns**: [references/common-patterns.md](references/common-patterns.md)
- Template patterns
- Examples patterns
- Consistent terminology
- Anti-patterns to avoid

**Executable code**: [references/executable-code.md](references/executable-code.md)
- When to use utility scripts
- Error handling in scripts
- Package dependencies
- MCP tool references

**API security**: [references/api-security.md](references/api-security.md)
- Preventing credentials from appearing in chat
- Using the secure API wrapper
- Adding new services and operations
- Credential storage patterns

**Iteration and testing**: [references/iteration-and-testing.md](references/iteration-and-testing.md)
- Evaluation-driven development
- Claude A/B development pattern
- Observing how Claude navigates Skills
- XML structure validation during testing

**Prompting fundamentals**:
- [references/clear-and-direct.md](references/clear-and-direct.md)
- [references/use-xml-tags.md](references/use-xml-tags.md)
</reference_guides>

<success_criteria>
A well-structured skill has:

- Valid YAML frontmatter with descriptive name and comprehensive description
- Pure XML structure with no markdown headings in body
- Required tags: objective, quick_start, success_criteria
- Conditional tags appropriate to complexity level
- Progressive disclosure (SKILL.md < 500 lines, details in reference files)
- Clear, concise instructions that assume Claude is smart
- Real-world testing and iteration based on observed behavior
- Lightweight slash command wrapper for discoverability
</success_criteria>
