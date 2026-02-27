---
name: simple-gemini
description: Collaborative documentation and test code writing workflow using zen mcp's clink to launch gemini CLI session in WSL (via 'gemini' command) where all writing operations are executed. Use this skill when the user requests "use gemini to write test files", "use gemini to write documentation", "generate related test files", "generate an explanatory document", or similar document/test writing tasks. The gemini CLI session acts as the specialist writer, working with the main Claude model for context gathering, outline approval, and final review. For test code, codex CLI (also launched via clink) validates quality after gemini completes writing.
---

# Gemini Documentation & Test Writer

## Overview

This skill provides a collaborative writing workflow where gemini CLI (launched via zen mcp's clink tool as a WSL command `gemini`) serves as a specialist writer for markdown documentation and test code. All writing operations are executed within the gemini CLI session environment, while the main Claude model handles context gathering, user interaction, and test execution. For test code, codex CLI (also launched via clink as WSL command `codex`) performs quality validation after gemini completes the initial writing.

**Technical Architecture:**
- **zen-mcp clink**: Acts as the bridge to launch CLI tools in WSL environment
- **gemini CLI session**: Opened via `gemini` command in WSL, where all document/test writing happens
- **codex CLI session**: Opened via `codex` command in WSL for code review tasks
- **Conversation context**: Maintained via `continuation_id` across CLI sessions

**Division of Responsibilities:**
- **Gemini CLI Session** (in WSL): Specialist writer for .md documents and test code, all writing executed inside this CLI environment
- **Main Claude Model**: Context gathering, CLI invocation orchestration, outline approval, test execution, final review
- **Codex CLI Session** (in WSL): Test code quality validation and correction, review executed inside this CLI environment
- **User**: Approval of outlines and final review (in Interactive Mode) or information recipient (in Automated Mode)

## When to Use This Skill

Trigger this skill when the user or main model requests:
- "Use gemini to write test files"
- "Use gemini to write documentation"
- "Generate related test files"
- "Generate an explanatory document"
- Commands from Claude or codex to write documentation
- Any request to create .md documentation or test code files

## Operation Mode (automation_mode - READ FROM SSOT)

automation_mode definition and constraints: See CLAUDE.md「📚 共享概念速查」

**This skill's role**: Skill Layer (read-only), read from context `[AUTOMATION_MODE: true/false]`
- `false` → Interactive: Requires user confirmation (outline, review, test corrections)
- `true` → Automated: Autonomous decisions, escalate only for critical issues, record to auto_log.md

### Coverage Target Management (READ ONLY - G9 Compliance)

coverage_target definition and constraints: See CLAUDE.md「📚 共享概念速查」

**This skill's role**: Skill Layer (read-only), read from context `[COVERAGE_TARGET: X%]`, use when generating/evaluating test code (default 85% if missing)

## Workflow Decision Tree

```
User Request
    │
    ├─→ Document Writing? ──→ Document Writing Workflow
    │
    └─→ Test Code Writing? ──→ Test Code Writing Workflow
```

## Document Writing Workflow

### Phase 1: Preparation & Context Gathering (Main Claude)

**Main Claude's Responsibility:**

1. **Understand the Documentation Need:**
   - What type of document? (README, PROJECTWIKI, ADR, CHANGELOG, technical spec, etc.)
   - What is the purpose and audience?
   - What scope should be covered?

2. **Gather Context:**
   - Read relevant code files if needed
   - Check existing documentation structure
   - Identify project standards from CLAUDE.md
   - **Context File Selection:**
     - **Interactive Mode (automation_mode = false)**: Ask user: "Do you need me to reference existing code/files for this document?"
     - **Automated Mode (automation_mode = true)**: Auto-analyze project structure and select relevant files, log decision to auto_log.md

3. **Identify Document Requirements:**
   - For CLAUDE.md mandated documents (PROJECTWIKI.md, CHANGELOG.md, ADRs):
     - Apply standards from `references/doc_templates/README.md` (on-demand template loading)
     - Follow CLAUDE.md Project Knowledge Base Content Structure and Generation Rules Unified Template
   - For other documents:
     - Determine appropriate structure
     - Identify key sections needed

### Phase 2: Outline Generation (Gemini CLI Session)

**Invoke Gemini CLI Session via `mcp__zen__clink`:**

```
Tool: mcp__zen__clink
Parameters:
- cli_name: "gemini"  # Launches 'gemini' command in WSL
- prompt: "Generate a detailed outline for [document type] covering [scope].
          Context: [provide all gathered context]
          Requirements: [standards from CLAUDE.md if applicable]
          Purpose: [document purpose]
          Audience: [target readers]"
- files: [list of relevant file paths for context - absolute paths]
- role: "default" (or "planner" for complex planning tasks)
- continuation_id: [if continuing from previous gemini CLI session]
```

**What Happens:**
1. zen-mcp clink opens a gemini CLI session in WSL
2. The prompt and files are passed into the gemini CLI environment
3. All outline generation work is executed inside the gemini CLI session
4. The gemini CLI session returns the completed outline
5. The session context is preserved via continuation_id for future calls

**Gemini CLI Session Output:** Detailed outline with:
- Main sections and subsections
- Key points to cover in each section
- Special considerations (diagrams, code examples, etc.)

### Phase 3: Outline Review & Approval (Main Claude + User)

**Main Claude's Responsibility:**

** automation_mode check**: `[AUTOMATION_MODE: false]` → Interactive / `true` → Automated

#### Interactive Mode (automation_mode=false, Default)

1. **Present Outline to User:**
   ```
   Gemini CLI has generated document outline:

   [Show outline]

   Do you approve this outline?
   - Yes: Continue writing
   - No: Please provide modification suggestions
   - Modify: [Specific modification suggestions]
   ```

2. **Wait for User Approval** - Do NOT proceed without confirmation

3. **Iterate if Needed:** If user requests changes, provide feedback to gemini and regenerate outline

#### Automated Mode (automation_mode=true)

1. **Main Claude Reviews Outline Autonomously (based on automation_mode=true):**
   - Check completeness: All required sections present?
   - Check structure: Follows template requirements?
   - Check scope: Covers all identified needs?
   - Check standards: Aligns with CLAUDE.md?

2. **Auto-Decision**: Meets standards → auto-approve + log; Else → retry (max 2×) + log, escalate if failed

3. **Present Decision to User (Information Only):**
   ```
    Outline auto-approved (automated mode)

   [Show outline summary]

   Approval reasons:
   - Structure complete
   - Meets template requirements
   - Covers all requirements

   Continuing to write complete document...
   ```

### Phase 4: Document Writing (Gemini CLI Session)

**After Outline Approval, Invoke Gemini CLI Session:**

```
Tool: mcp__zen__clink
Parameters:
- cli_name: "gemini"  # Reuses the same gemini CLI session in WSL
- prompt: "Write the complete [document type] based on this approved outline:
          [outline]

          Writing Guidelines:
          - Follow the outline structure exactly
          - For PROJECTWIKI.md/CHANGELOG.md/ADR: strictly follow templates in references/doc_templates/README.md (load specific templates as needed)
          - Use Mermaid diagrams where appropriate (```mermaid blocks)
          - Write in clear, professional Chinese (or English if specified)
          - Include code examples where helpful
          - Ensure consistency with CLAUDE.md standards

          Context: [all gathered context]
          Referenced files: [files to reference]"
- files: [relevant files - absolute paths]
- role: "default"
- continuation_id: [reuse from outline generation - maintains session context]
```

**What Happens:**
1. zen-mcp clink reconnects to the existing gemini CLI session using continuation_id
2. The session has context from the previous outline generation
3. The full document writing work is executed inside the gemini CLI session
4. The gemini CLI session returns the completed markdown document

**Gemini CLI Session Output:** Complete markdown document

### Phase 5: Review & Finalization (Main Claude + User)

**Main Claude's Responsibility:**

** automation_mode check**: `[AUTOMATION_MODE: false]` → Interactive / `true` → Automated

#### Interactive Mode (automation_mode=false, Default)

1. **Present Document:**
   ```
   Gemini CLI has completed document writing:

   Document type: [type]
   File path: [proposed path]

   [Show document content or summary]

   Please review this document:
   - Approve and save
   - 🔄 Needs modification: [Please specify modification content]
   - Regenerate
   ```

2. **Handle Feedback:**
   - If approved: Write the document to the file system
   - If modifications needed: Provide feedback to gemini for revision
   - If regeneration needed: Return to Phase 2

3. **Finalize:**
   - Save document to appropriate location
   - Update CHANGELOG.md if this is a significant documentation change
   - Confirm completion with user

#### Automated Mode (automation_mode=true)

1. **Main Claude Validates Document Autonomously (based on automation_mode=true):**
   - **For PROJECTWIKI/CHANGELOG/ADR**: Check against `references/doc_templates/README.md` (load specific templates for validation)
     - All required sections present?
     - Mermaid diagrams included?
     - Links are valid?
     - Consistent with CLAUDE.md standards?
   - **For other documents**: Check completeness, clarity, and consistency

2. **Auto-Decision**: Meets quality → auto-approve + save + update CHANGELOG + log; Else → retry (max 2×) + log, escalate if failed

3. **Present Final Result to User (Information Only):**
   ```
    Document auto-completed and saved (automated mode)

   Document type: [type]
   File path: [actual path]

   Quality checks:
   - Structure complete
   - Meets standards
   - Format correct
   - Links valid

   [Show document summary or key sections]

   CHANGELOG.md automatically updated
   ```

## Test Code Writing Workflow

### Phase 1: Preparation & Context Gathering (Main Claude)

**Main Claude's Responsibility:**

1. **Understand Testing Need:**
   - What code/module needs testing?
   - Test type: unit, integration, or E2E?
   - Testing framework: pytest, unittest, jest, etc.?
   - Coverage requirements: Read from context `[COVERAGE_TARGET: X%]` (default 85% if missing)

2. **Gather Context:**
   - Read the code to be tested
   - Identify key functions/classes/modules
   - Check existing test structure
   - Review testing standards from CLAUDE.md

3. **Identify Test Requirements:**
   - Key functionality to test
   - Edge cases and boundary conditions
   - Error handling scenarios
   - Performance considerations if applicable

### Phase 2: Test Code Generation (Gemini CLI Session)

**Invoke Gemini CLI Session via `mcp__zen__clink`:**

```
Tool: mcp__zen__clink
Parameters:
- cli_name: "gemini"  # Launches 'gemini' command in WSL
- prompt: "Generate comprehensive test code for [module/function].

          Code to Test:
          [code content or file references]

          Test Requirements:
          - Framework: [pytest/unittest/etc.]
          - Test types: [unit/integration/E2E]
          - Coverage target: ≥ {coverage_target from context, e.g., 85%}
          - Include: normal cases, edge cases, error handling, boundary conditions

          Standards:
          - Follow best practices from references/test_patterns.md
          - Clear test names and assertions
          - Proper setup/teardown
          - Mock external dependencies
          - Document complex test logic

          Context: [project structure, existing tests, conventions]"
- files: [code files to test + existing test examples - absolute paths]
- role: "default"
- continuation_id: [if continuing from previous gemini CLI session]
```

**What Happens:**
1. zen-mcp clink opens a gemini CLI session in WSL
2. Source code and test examples are passed into the gemini CLI environment
3. All test code generation work is executed inside the gemini CLI session
4. The gemini CLI session returns the completed test code file(s)

**Gemini CLI Session Output:** Complete test code file(s)

### Phase 3: Test Code Validation (Codex)

**Main Claude invokes Codex via `mcp__zen__codereview`:**

```
Tool: mcp__zen__codereview
Parameters:
- step: "Review the test code generated by gemini for quality, completeness, and adherence to testing standards.

        Focus Areas:
        - Test coverage adequacy
        - Assertion completeness
        - Edge case handling
        - Code quality (readability, maintainability)
        - Framework best practices
        - Mock/fixture usage
        - Error handling in tests"
- step_number: 1
- total_steps: 2-3
- next_step_required: true
- findings: ""
- relevant_files: [absolute paths to generated test files]
- review_type: "full"
- model: "codex"
- review_validation_type: "external"
- confidence: "exploring"
- files_checked: [test file paths]
```

**Codex Output:** Review findings with identified issues (if any)

### Phase 4: Test Code Correction (If Needed - Codex CLI)

**If Codex CLI Identifies Issues:**

** automation_mode check**: `[AUTOMATION_MODE: false]` → Interactive / `true` → Automated

#### Interactive Mode (automation_mode=false, Default)

Main Claude presents findings to user:
```
Codex CLI check found the following issues:

[Critical] Test file A:line B - Missing boundary condition tests
[Medium] Test file A:line C - Assertions not specific enough
...

Do you approve Codex CLI auto-correcting these issues?
- Yes: Continue corrections
- No: Manual modification
```

After approval, codex CLI applies corrections (following codex-code-reviewer workflow).

#### Automated Mode (automation_mode=true)

Main Claude reviews issues and decides autonomously (based on automation_mode=true):

1. **Evaluate Issue Severity:**
   - Critical/High severity: Always fix
   - Medium severity: Fix if straightforward
   - Low severity: Fix if no risk

2. **Auto-Decision**: Fixable + low risk → auto-approve + log; Complex/high-risk → escalate + log

3. **Present Decision to User (Information Only):**
   ```
    Test code issues auto-corrected (automated mode)
   [automation_mode=true set by router]

   Corrected issues:
   - [Critical] Added boundary condition tests
   - [Medium] Enhanced assertion descriptions
   - [Low] Optimized test naming

   Decision basis: automation_mode=true, all issues safely fixable
   Recorded to auto_log.md

   Continuing to run tests...
   ```

### Phase 5: Test Execution & Review (Main Claude + User)

**Main Claude's Responsibility:**

** automation_mode check**: `[AUTOMATION_MODE: false]` → Interactive / `true` → Automated

1. **Execute Tests:**
   - Run the test suite using appropriate commands
   - Capture output and results
   - Analyze failures if any

#### Interactive Mode (automation_mode=false, Default)

2. **Present Results to User:**
   ```
   Gemini CLI has completed test code writing, Codex CLI has verified quality.

   Test files: [file paths]
   Coverage: [percentage]

   Test run results:

    Passed: X tests
    Failed: Y tests
     Skipped: Z tests

   [Detailed results]

   Are you satisfied? Need adjustments?
   - Approve and save
   - 🔄 Needs adjustment
   - Regenerate
   ```

3. **Iterate if Needed:**
   - If tests fail due to test code issues: Provide feedback to gemini/codex for correction
   - If tests reveal bugs in source code: Handle separately (not this skill's responsibility)

#### Automated Mode (automation_mode=true)

2. **Main Claude Evaluates Test Results Autonomously (based on automation_mode=true):**
   - **Success Criteria:**
     - All tests pass (or only expected skips)
     - Coverage ≥ coverage_target (read from context via `[COVERAGE_TARGET: X%]`, default 85%)
     - No critical failures

3. **Auto-Decision**: Pass + coverage ≥ target → auto-save + log; Fail → analyze: fixable → retry (max 2×), source bugs → report + save, persistent → escalate

4. **Present Final Result to User (Information Only):**
   ```
    Tests auto-completed (automated mode)

   Test files: [file paths]

   Test run results:
   - Passed: X tests (100%)
   - Coverage: 78%

   Quality checks:
   - All tests passed
   - Coverage met target (≥ coverage_target, e.g. 85%)
   - Test structure clear
   - Assertions complete

   Test files saved
   ```

## Collaboration Guidelines

### Main Claude's Role

- **Context Provider**: Gather all necessary context for gemini
- **User Interface**: Handle all user interactions and approvals
- **Executor**: Run tests, validate results
- **Quality Gate**: Final review before delivery
- **Feedback Loop**: Provide structured feedback to gemini/codex for iterations

### Gemini CLI Session's Role (in WSL)

- **Specialist Writer**: Focus purely on high-quality writing within the CLI session environment
- **Standard Adherent**: Strictly follow templates and standards provided in the prompt
- **Context Consumer**: Access and use files passed into the CLI session environment
- **Session-based Processing**: All writing work happens inside the WSL CLI session
- **Responsive**: Accept feedback and iterate within the same session via continuation_id
- **Independent Execution**: Operates independently in WSL, not relying on Main Claude's capabilities

### Codex's Role

- **Test Validator**: Ensure test code quality
- **Best Practice Enforcer**: Apply testing standards
- **Corrector**: Fix identified issues
- **Quality Reporter**: Provide clear findings to main Claude

## Document Type Standards

### CLAUDE.md Mandated Documents

For these documents, **strictly follow** templates in `references/doc_templates/README.md` (on-demand template loading):

1. **PROJECTWIKI.md**
   - Follow Project Knowledge Base Content Structure and Generation Rules Unified Template
   - Include all 12 required sections
   - Use Mermaid diagrams
   - Ensure traceability links
   - Template: `projectwiki_template.md`

2. **CHANGELOG.md**
   - Follow Keep a Changelog format
   - Semantic Versioning
   - Link to commits and PRs
   - Template: `changelog_template.md`

3. **ADR (Architecture Decision Records)**
   - Use MADR template
   - Format: `YYYYMMDD-title.md`
   - Include context, alternatives, decision, consequences

4. **plan.md**
   - Detailed implementation plan
   - Checkable task list
   - Review section for summary

See `references/doc_templates/README.md` for template index, then load specific templates as needed.

### Other Documents

For general documentation:
- Follow project conventions
- Use clear structure
- Include examples where helpful
- Maintain consistency with existing docs

## Test Code Standards

Follow patterns from `references/test_patterns.md`:

1. **Test Organization:**
   - One test file per source file (convention: `test_<module>.py` or `<module>_test.py`)
   - Group related tests in classes
   - Clear, descriptive test names

2. **Coverage Requirements:**
   - Target: coverage_target from context (default 85%, minimum 70%)
   - Test all public APIs
   - Cover edge cases and error conditions

3. **Best Practices:**
   - AAA pattern (Arrange-Act-Assert)
   - Use fixtures for setup/teardown
   - Mock external dependencies
   - Avoid test interdependencies
   - Clear assertions with helpful messages

## Tool Parameters Reference

### mcp__zen__clink (Launch CLI Tools in WSL)

**Purpose:** Bridge tool to launch and interact with CLI tools (gemini, codex) in WSL environment.

**How It Works:**
1. clink launches the specified CLI tool as a command in WSL (e.g., `gemini` command)
2. Opens an interactive CLI session for that tool
3. Passes prompt and files into the CLI session environment
4. All processing happens inside the CLI tool's session
5. Returns the CLI session's output to Main Claude
6. Maintains session context via continuation_id for multi-turn conversations

**Key parameters for launching Gemini CLI:**
- `cli_name`: "gemini" (required - launches `gemini` command in WSL)
- `prompt`: The writing request with full context and requirements (required)
- `files`: List of relevant files for context - absolute paths (optional)
  - These files are accessible within the gemini CLI session
- `images`: List of image paths for visual context - absolute paths (optional)
- `role`: "default", "codereviewer", or "planner" (optional, default: "default")
  - Determines the gemini CLI's behavior/persona within the session
- `continuation_id`: Reuse to maintain the same gemini CLI session across multiple calls (optional)
  - Enables multi-turn conversations within the same CLI environment
  - Example: outline generation → full document writing uses same session

**Available Roles for Gemini CLI Session:**
- `default`: General-purpose writing within CLI
- `planner`: Planning and outlining tasks within CLI
- `codereviewer`: Code-related documentation within CLI

**Example Flow:**
```
Call 1 with continuation_id=None:
  → Launches: gemini command in WSL
  → Session: New gemini CLI session starts
  → Returns: Outline + continuation_id="abc123"

Call 2 with continuation_id="abc123":
  → Reconnects: To existing gemini CLI session "abc123"
  → Session: Same gemini CLI with previous context
  → Returns: Full document + same continuation_id
```

### mcp__zen__codereview (Codex CLI)

Key parameters for test validation:
- `model`: "codex"
- `review_type`: "full"
- `review_validation_type`: "external"
- `relevant_files`: Absolute paths to test files
- Follow codex-code-reviewer skill standards

## Error Handling

### If Gemini CLI Fails
- Main Claude: Notify user of the CLI invocation failure
- Offer to retry with adjusted parameters or write manually
- Check if continuation_id needs to be reset
- Provide more specific instructions and context
- Consider breaking down into smaller tasks

### If Codex Validation Fails
- Main Claude: Present issues to user
- Offer manual correction or codex automatic correction
- Maximum 3 correction iterations to prevent loops

### If User Rejects Output
- Gather specific feedback
- Provide detailed feedback to gemini CLI via new clink invocation
- Maintain continuation_id for context continuity
- Regenerate with improvements

## Resources

### references/doc_templates/README.md

Document templates index and on-demand loading guide. Load this reference when:
- Writing PROJECTWIKI.md, CHANGELOG.md, ADR, or plan.md
- Need to verify format compliance

Then load specific template files as needed:
- `projectwiki_template.md` - PROJECTWIKI.md 标准模板
- `changelog_template.md` - CHANGELOG.md 标准模板
- `readme_template.md`, `api_doc_template.md`, etc.

### references/test_patterns.md

Testing best practices and code patterns. Load when:
- Writing test code
- User asks about testing standards
- Need test structure examples

## Example Usage Scenarios

### Example 1: Writing ADR (Interactive Mode)

**User**: "Use gemini to write documentation, I need to record the database selection decision ADR"

**Main Claude**:
```
OK, I'll coordinate Gemini CLI to write the ADR document for you.

First let me understand some information:
1. What is the background of the database selection?
2. What database options were considered?
3. Which database was ultimately chosen?
4. What were the main decision factors?

[After collecting information...]

Now let Gemini CLI generate ADR outline...
[Call mcp__zen__clink with cli_name="gemini"]

Gemini CLI generated outline as follows:
[Display outline]

Approve?
```

**User**: "Yes"

**Main Claude**:
```
[Call mcp__zen__clink for complete writing, using continuation_id]

Gemini CLI has completed ADR writing:

File: docs/adr/20251020-select-postgresql.md

[Display content]

Approve to save?
```

### Example 2: Writing Test Code (Interactive Mode)

**User**: "Use gemini to write test file, test src/features.py"

**Main Claude**:
```
OK, let me first analyze the code in src/features.py...

[Read code]

Found key functions that need testing:
- create_features()
- validate_data()
- transform_features()

Test framework: pytest
Target coverage: ≥ {coverage_target from context, e.g., 85%}

Now let Gemini CLI write test code...
[Call mcp__zen__clink with cli_name="gemini"]

Test code has been generated. Now let Codex CLI verify quality...
[Call mcp__zen__codereview with model="codex"]

Codex CLI check results:
✅ Complete test coverage
✅ Clear assertions
⚠️ Suggest adding boundary condition tests

Do you approve Codex CLI to automatically add boundary condition tests?
```

**User**: "Yes"

**Main Claude**:
```
[Codex CLI corrections applied]

Test code has been improved. Now running tests...
[Executing tests]

Test results:
✅ All tests passed
Coverage: 78%

Test file saved: tests/test_features.py
```

### Example 3: Writing ADR (Automated Mode)

**User**: "Use gemini to write documentation, record the database selection decision ADR. Automated execution, no confirmation needed"

**Router**: Sets `automation_mode=true` (detects "Automated execution, no confirmation needed")

**Main Claude** (reads automation_mode=true from context):
```
[automation_mode=true] Automation mode detected, will autonomously approve and complete the document.

First, let me understand some information:
1. What is the background of the database selection?
2. What database options were considered?
3. What database was finally selected?
4. What were the main decision factors?

[After collecting information...]

Generating ADR outline...
[Calling mcp__zen__clink with cli_name="gemini"]

✅ Outline automatically approved (automation mode)

Outline summary:
1. Background and Problem Statement
2. Decision Drivers
3. Considered Options (PostgreSQL, MongoDB, MySQL)
4. Decision Outcome
5. Implementation Plan
6. Validation Method

Approval reasons:
- Complete structure (complies with MADR template)
- Covers all requirements
- Meets standards

Continuing to write complete document...
[Calling mcp__zen__clink with continuation_id]

✅ Document automatically completed and saved (automation mode)

Document type: ADR
File path: docs/adr/20251020-select-postgresql.md

Quality check:
- Complete structure (all MADR sections)
- Complies with CLAUDE.md standards
- Correct format
- No broken links

Document summary:
- Decision: Selected PostgreSQL
- Rationale: Excellent performance, complete features, team has experience
- Impact: Query performance improved 3-5x, requires 1-2 weeks migration

CHANGELOG.md automatically updated

ADR writing completed!
```

### Example 4: Writing Test Code (Automated Mode)

**User**: "Use gemini to write test file, test src/features.py, complete automatically"

**Router**: Sets `automation_mode=true` (detects "complete automatically")

**Main Claude** (reads automation_mode=true from context):
```
[automation_mode=true] Automation mode detected, will autonomously complete test code writing.

Analyzing code...
[Reading code]

Key functions to be tested identified:
- create_features()
- validate_data()
- transform_features()

Test framework: pytest
Target coverage: ≥ {coverage_target from context, e.g., 85%}

Generating test code...
[Calling mcp__zen__clink with cli_name="gemini"]

Conducting quality verification...
[Calling mcp__zen__codereview with model="codex"]

Codex CLI found 3 improvement points:
- [Medium] Suggest adding boundary condition tests
- [Low] Optimize test naming
- [Low] Add explanatory comments

✅ Test code issues automatically corrected (automation mode)

Corrected issues:
- [Medium] Added boundary condition tests (empty list, None, extreme values)
- [Low] Optimized test naming (more descriptive)
- [Low] Added comments for complex logic

Running tests...
[Executing pytest --cov=src tests/]

✅ Testing automatically completed (automation mode)

Test file: tests/test_features.py

Test execution results:
- Passed: 15 tests (100%)
- Coverage: 82%

Quality check:
- All tests passed
- Coverage meets target (≥ coverage_target, e.g., 85%)
- Clear test structure (AAA pattern)
- Complete assertions (with error messages)
- Correct Mock usage

Detailed results:
- test_create_features_normal:
- test_create_features_empty_input:
- test_create_features_none_input:
- test_validate_data_valid:
- test_validate_data_invalid:
- test_transform_features_basic:
- ... (Total 15 tests)

Test file saved, test code writing completed!
```

## Notes

- **CLI Session Architecture**: zen-mcp clink launches actual CLI tools (`gemini` and `codex` commands) in WSL environment
- **Execution Location**: All writing operations happen inside the gemini CLI session in WSL, not in the main Claude model
- **Context Preservation**: continuation_id maintains the same CLI session across multiple calls, enabling multi-turn conversations within the CLI environment
- Gemini CLI session excels at understanding context and producing well-structured writing
- Main Claude acts as orchestrator and bridge, not the actual writer

** CRITICAL - automation_mode & auto_log Management:**
- automation_mode: See CLAUDE.md「📚 共享概念速查」and G11 Three-Layer Architecture
- This skill: Skill Layer (read-only), follows router-set automation_mode, logs all auto-decisions

**CRITICAL - auto_log.md Generation (auto_log - READ FROM SSOT):**

auto_log mechanism and template: See CLAUDE.md「📚 共享概念速查」and `skills/shared/auto_log_template.md`

**This skill's role**: Generate complete auto_log.md from router-collected fragments (when automation_mode=true and task complete)

- Codex CLI session (also in WSL) ensures test code meets engineering standards
- This workflow separates concerns:
  - **Context gathering**: Main Claude
  - **Writing**: Gemini CLI session in WSL
  - **Validation**: Codex CLI session in WSL
  - **Execution & Review**: Main Claude
  - **Mode management**: main-router (sets automation_mode)
- The clink tool provides seamless integration with external CLI tools while maintaining conversation context
- **WSL Integration**: All CLI tools run in WSL, ensuring compatibility with Linux-based tools and commands
