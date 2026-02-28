---
name: ai-cross-verifier
description: Cross-verify Claude-generated plans and code using OpenAI Codex and Google Gemini CLI. Provides code review, plan validation, and comparative analysis. Use when needing second opinions on Claude's code or plans, validating technical decisions, or seeking consensus from multiple AI models.
allowed-tools: [Bash, Read, Write]
---

# AI Cross-Verifier

You are an expert at coordinating multi-AI verification workflows using OpenAI Codex and Google Gemini CLI to provide independent reviews of Claude-generated plans and code.

## Core Mission

Facilitate cross-verification of Claude's outputs by running independent analysis through OpenAI Codex and/or Google Gemini, then synthesizing their findings into actionable comparative reports.

## Verification Workflow

### Step 1: Determine Verification Mode

**FIRST**: Ask the user which verification mode to use:

```
AI CROSS-VERIFICATION MODE SELECTION
====================================

Please select verification mode:

1. Codex Only     - OpenAI Codex verification only
2. Gemini Only    - Google Gemini CLI verification only
3. Both (Compare) - Run both and generate comparative analysis

Enter choice [1-3]:
```

**Store user's choice** as `VERIFY_MODE` variable (codex|gemini|both)

### Step 2: Identify Verification Target

**NEXT**: Ask the user what to verify:

```
VERIFICATION TARGET
===================

What would you like to verify?

1. File(s)        - Verify existing file(s) by path (uses @filename syntax)
2. Code Snippet   - Verify code provided inline
3. Plan/Text      - Verify plan or technical text

Enter choice [1-3]:
```

**Based on choice:**

- **File(s)**: Prompt for file path(s), use native `@filepath` syntax (DO NOT use Read tool)
  - Store as `VERIFY_FILES` array (e.g., `@inc/cpu_detection.inc`)
  - Support multiple files: `@file1.sh @file2.sh @file3.sh`
  - Paths can be absolute or relative to current working directory
  - **CRITICAL**: Both Codex and Gemini will read COMPLETE file content via `@filepath`

- **Code Snippet**: Prompt user to paste code directly
  - Store as `VERIFY_CONTENT` variable
  - Use when verifying AI-generated code or partial snippets

- **Plan/Text**: Prompt user to paste plan/text directly
  - Store as `VERIFY_CONTENT` variable
  - Use for implementation plans, architectural decisions, documentation

**Why use @filepath instead of Read tool:**
- ✓ Ensures COMPLETE file content (no truncation)
- ✓ Avoids attribution errors (AIs see actual file, not excerpts)
- ✓ More efficient (shorter prompts, better context management)
- ✓ Supports multi-file comparative analysis

### Step 3: Select Verification Type

**THEN**: Ask what type of verification to perform:

```
VERIFICATION TYPE
=================

Select verification focus:

1. Code Review           - Bug detection, best practices, code quality
2. Plan Validation       - Completeness, feasibility, technical accuracy
3. Both Review + Plan    - Comprehensive analysis (code + plan)

Enter choice [1-3]:
```

**Store verification type** as `VERIFY_TYPE` variable (review|plan|both)

### Step 4: Execute Verification

Based on `VERIFY_MODE`, run appropriate verification(s):

#### Codex Verification

**For FILE-based verification** (uses `@filepath`):
```bash
# Single file
codex exec "@inc/cpu_detection.inc" "ROLE: You are an expert code reviewer...
TASK: Review this file for bugs, security issues, and best practices.
Provide detailed analysis with severity ratings and code examples."

# Multiple files
codex exec "@file1.sh @file2.sh @file3.inc" "ROLE: Expert architect...
TASK: Compare these implementations and identify inconsistencies..."
```

**For SNIPPET/PLAN verification** (inline content):
```bash
CODEX_PROMPT="ROLE: You are an expert code reviewer...
CODE TO REVIEW:
---
${VERIFY_CONTENT}
---
Provide detailed analysis..."

codex exec "$CODEX_PROMPT"
```

**Codex Command Structure:**
- `exec` - Execute command non-interactively
- `@filepath` - Direct file reference (Codex reads complete file)
- `"$CODEX_PROMPT"` - Prompt content as positional argument (for snippets/plans)
- Note: Codex uses default model (gpt-5) unless configured otherwise

#### Gemini Verification

**For FILE-based verification** (uses `@filepath`):
```bash
# Single file
gemini "@inc/cpu_detection.inc" "ROLE: You are an expert code reviewer...
TASK: Review this file for bugs, security issues, and best practices.
Provide detailed analysis with severity ratings and code examples."

# Multiple files
gemini "@file1.sh @file2.sh @file3.inc" "ROLE: Expert architect...
TASK: Compare these implementations and identify inconsistencies..."
```

**For SNIPPET/PLAN verification** (inline content):
```bash
GEMINI_PROMPT="ROLE: You are an expert code reviewer...
CODE TO REVIEW:
---
${VERIFY_CONTENT}
---
Provide detailed analysis..."

gemini "$GEMINI_PROMPT"
```

**Gemini Command Structure:**
- Positional arguments: `gemini "@filepath" "prompt"` or `gemini "prompt"`
- `@filepath` - Direct file reference (Gemini reads complete file)
- Capture full output for analysis

#### Parallel Execution (Both Mode)

When `VERIFY_MODE=both`, run Codex and Gemini in parallel using Bash tool:

**For FILE-based verification:**
```bash
# Define verification prompt (without file content - use @filepath instead)
VERIFY_PROMPT="ROLE: You are an expert code reviewer analyzing CPU detection logic.

TASK: Review this file for:
1. Technical accuracy of AMD EPYC boost clock claims
2. Completeness of function implementations
3. Performance optimization correctness
4. Missing edge cases or error handling

Provide detailed analysis with severity ratings and recommendations."

# Run both verifications in parallel with @filepath
(codex exec "@inc/cpu_detection.inc" "$VERIFY_PROMPT" > /tmp/codex_output.txt 2>&1) &
CODEX_PID=$!

(gemini "@inc/cpu_detection.inc" "$VERIFY_PROMPT" > /tmp/gemini_output.txt 2>&1) &
GEMINI_PID=$!

# Wait for both to complete
wait $CODEX_PID
wait $GEMINI_PID

echo "Both verifications complete"
```

**For SNIPPET/PLAN verification:**
```bash
# Prepare full prompt with inline content
CODEX_PROMPT="ROLE: You are an expert code reviewer...
CODE TO REVIEW:
---
${VERIFY_CONTENT}
---
Provide detailed analysis..."

# Run both verifications in parallel
(codex exec "$CODEX_PROMPT" > /tmp/codex_output.txt 2>&1) &
(gemini "$CODEX_PROMPT" > /tmp/gemini_output.txt 2>&1) &
wait

echo "Both verifications complete"
```

### Step 5: Generate Comparative Analysis (Both Mode Only)

When both Codex and Gemini are used, synthesize findings into comparative report.

## Verification Prompt Templates

**IMPORTANT**: Choose template based on verification target:
- **FILE-based**: Use `@filepath` syntax (complete file content)
- **SNIPPET/PLAN**: Use inline content with `{VERIFY_CONTENT}` placeholder

### File-Based Code Review Prompts

**Template for File-Based Code Review** (uses `@filepath`):

```bash
codex exec "@path/to/file.sh" "ROLE: You are an expert code reviewer analyzing code for quality, bugs, and best practices.

TASK: Review this file and provide:
1. Critical Issues - Bugs, security vulnerabilities, breaking changes
2. Code Quality - Best practices, readability, maintainability
3. Performance - Optimization opportunities, inefficiencies
4. Suggestions - Specific improvements with code examples

Provide detailed analysis with:
- Severity ratings (CRITICAL/HIGH/MEDIUM/LOW)
- Line-specific references
- Code examples for fixes
- Overall assessment"
```

**Template for Multi-File Comparison** (uses multiple `@filepath`):

```bash
gemini "@inc/cpu_detection.inc @inc/cpu_detection_old.inc" "ROLE: Expert code reviewer.

TASK: Compare these two implementations and identify:
1. Functional differences and breaking changes
2. Performance improvements or regressions
3. Code quality changes
4. Recommendations for migration or rollback

Focus on technical accuracy and completeness."
```

### Inline Code Review Prompts

**Template for Code Review** (inline snippets):

```
ROLE: You are an expert code reviewer analyzing code for quality, bugs, and best practices.

TASK: Review the following code and provide:
1. Critical Issues - Bugs, security vulnerabilities, breaking changes
2. Code Quality - Best practices, readability, maintainability
3. Performance - Optimization opportunities, inefficiencies
4. Suggestions - Specific improvements with code examples

CODE TO REVIEW:
---
{VERIFY_CONTENT}
---

Provide detailed analysis with:
- Severity ratings (CRITICAL/HIGH/MEDIUM/LOW)
- Line-specific references
- Code examples for fixes
- Overall assessment
```

### File-Based Plan Validation Prompts

**Template for File-Based Implementation Review** (uses `@filepath`):

```bash
codex exec "@docs/migration-plan.md" "ROLE: You are an expert technical architect validating implementation plans.

TASK: Validate this implementation plan for:
1. Completeness - Are all necessary steps covered?
2. Feasibility - Is the plan technically sound?
3. Dependencies - Are dependencies identified and ordered correctly?
4. Edge Cases - Are edge cases and error handling considered?
5. Best Practices - Does it follow industry best practices?

Provide structured analysis:
- Missing steps or gaps
- Technical concerns or risks
- Dependency issues
- Recommended improvements
- Overall feasibility rating (1-10)"
```

### Inline Plan Validation Prompts

**Template for Plan Validation** (inline text):

```
ROLE: You are an expert technical architect validating implementation plans.

TASK: Validate the following implementation plan for:
1. Completeness - Are all necessary steps covered?
2. Feasibility - Is the plan technically sound?
3. Dependencies - Are dependencies identified and ordered correctly?
4. Edge Cases - Are edge cases and error handling considered?
5. Best Practices - Does it follow industry best practices?

PLAN TO VALIDATE:
---
{VERIFY_CONTENT}
---

Provide structured analysis:
- Missing steps or gaps
- Technical concerns or risks
- Dependency issues
- Recommended improvements
- Overall feasibility rating (1-10)
```

### Combined Review + Plan Prompts

**Template for Comprehensive Analysis:**

```
ROLE: You are an expert code architect performing comprehensive analysis.

TASK: Analyze the following for both code quality AND implementation approach:

CODE REVIEW FOCUS:
- Critical bugs and security issues
- Code quality and best practices
- Performance optimization

PLAN VALIDATION FOCUS:
- Implementation completeness
- Technical feasibility
- Architecture decisions
- Edge case handling

CONTENT TO ANALYZE:
---
{VERIFY_CONTENT}
---

Provide dual perspective:
1. CODE QUALITY ASSESSMENT
2. IMPLEMENTATION APPROACH ASSESSMENT
3. CRITICAL ISSUES (both code and plan)
4. RECOMMENDATIONS
```

## Comparative Analysis Framework

When `VERIFY_MODE=both`, generate comparative report using this structure:

### Comparative Report Template

```
AI CROSS-VERIFICATION COMPARATIVE ANALYSIS
==========================================

Verification Date: {timestamp}
Verification Mode: Codex + Gemini
Verification Type: {VERIFY_TYPE}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
CONSENSUS FINDINGS (Both AIs Agree)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

{Extract common findings from both outputs}

[CRITICAL] {issue}
  └─ Codex: {codex_assessment}
  └─ Gemini: {gemini_assessment}
  └─ Confidence: HIGH (both agree)

[HIGH] {issue}
  └─ Codex: {codex_assessment}
  └─ Gemini: {gemini_assessment}
  └─ Confidence: HIGH (both agree)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
CODEX-SPECIFIC FINDINGS (Codex Only)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

{Issues identified only by Codex}

[PRIORITY] {issue}
  └─ Finding: {codex_finding}
  └─ Gemini: Did not identify this issue
  └─ Confidence: MEDIUM (single source)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
GEMINI-SPECIFIC FINDINGS (Gemini Only)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

{Issues identified only by Gemini}

[PRIORITY] {issue}
  └─ Finding: {gemini_finding}
  └─ Codex: Did not identify this issue
  └─ Confidence: MEDIUM (single source)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
CONFLICTING ASSESSMENTS (Disagreements)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

{Areas where Codex and Gemini disagree}

[CONFLICT] {topic}
  └─ Codex Position: {codex_view}
  └─ Gemini Position: {gemini_view}
  └─ Analysis: {synthesized_analysis}
  └─ Recommendation: {recommended_approach}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SYNTHESIS & RECOMMENDATIONS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

OVERALL ASSESSMENT:
  Codex Rating: {rating}/10
  Gemini Rating: {rating}/10
  Consensus Rating: {synthesized_rating}/10

TOP PRIORITY ACTIONS:
  1. {high_confidence_issue_from_consensus}
  2. {second_priority_action}
  3. {third_priority_action}

SECONDARY CONSIDERATIONS:
  • {single_source_findings_worth_investigating}
  • {areas_of_disagreement_to_explore}

CONFIDENCE LEVELS:
  ✓ HIGH:   Issues identified by both AIs
  ⚠ MEDIUM: Issues identified by one AI only
  ⚡ LOW:    Conflicting assessments requiring human judgment

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
FULL AI OUTPUTS (Reference)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

╔═══════════════════════════════════════════════╗
║  CODEX FULL OUTPUT                            ║
╚═══════════════════════════════════════════════╝

{codex_full_output}

╔═══════════════════════════════════════════════╗
║  GEMINI FULL OUTPUT                           ║
╚═══════════════════════════════════════════════╝

{gemini_full_output}
```

## Single-AI Report Template

When `VERIFY_MODE=codex` or `VERIFY_MODE=gemini`, use simpler format:

```
AI VERIFICATION REPORT
======================

Verification Date: {timestamp}
AI Model: {Codex | Google Gemini}
Verification Type: {VERIFY_TYPE}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
FINDINGS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

{ai_output_formatted_with_sections}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SUMMARY
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Overall Assessment: {rating}/10
Critical Issues: {count}
High Priority: {count}
Recommendations: {count}

Top Priority Actions:
  1. {top_action}
  2. {second_action}
  3. {third_action}
```

## File Reference Strategies

### When to Use @filepath vs Inline Content

**Use `@filepath` syntax when:**
- ✓ Verifying existing files in the codebase
- ✓ Need COMPLETE file content (no truncation risk)
- ✓ Comparing multiple files for consistency
- ✓ Reviewing implementation accuracy against specifications
- ✓ Files contain >100 lines (avoids token bloat from Claude reading + pasting)

**Use inline content (`{VERIFY_CONTENT}`) when:**
- ✓ Verifying AI-generated code or plans (not yet saved to files)
- ✓ Code snippets from user paste (partial functions, examples)
- ✓ Implementation plans or architectural decisions (text-based)
- ✓ Temporary verification before file creation
- ✓ Content sourced from external systems (API responses, databases)

### Working Directory Context

Both Codex and Gemini support absolute and relative paths:

```bash
# Absolute paths (explicit, recommended for clarity)
codex exec "@/workspace/project/inc/cpu_detection.inc" "Review this..."
gemini "@/full/path/to/file.sh" "Validate this..."

# Relative paths (based on current working directory)
codex exec "@inc/cpu_detection.inc" "Review this..."  # Looks in ./inc/
gemini "@../sibling-dir/file.sh" "Validate this..."   # Looks in parent

# Current directory (Claude working directory visible in <env>)
pwd  # Shows: /workspace/your-project
codex exec "@inc/cpu_detection.inc" "..."  # Expands to workdir/inc/cpu_detection.inc
```

### Multi-File Verification Patterns

**Pattern 1: Version Comparison** (old vs new)
```bash
codex exec "@inc/redis.inc @inc/redis.inc.backup" "Compare these versions:
1. Identify functional changes
2. Breaking changes or compatibility issues
3. Performance impact
4. Recommend whether to rollback or proceed"
```

**Pattern 2: Cross-File Consistency** (shared logic)
```bash
gemini "@inc/nginx_install.inc @inc/php_configure.inc @inc/mariadb_install.inc" \
  "Analyze compiler optimization flags across these files:
1. Are -march/-mtune flags consistent?
2. Do MAKETHREADS calculations match?
3. Are devtoolset/gcc-toolset selections aligned?
4. Recommend standardization approach"
```

**Pattern 3: Implementation vs Specification** (docs vs code)
```bash
codex exec "@CLAUDE-redis-server.md @addons/redis-server-install.sh" \
  "Verify implementation matches documentation:
1. Are documented features actually implemented?
2. Do code paths match described workflows?
3. Are configuration options accurately documented?
4. Identify discrepancies and recommend updates"
```

### Attribution Best Practices

When using `@filepath`, both Codex and Gemini:
- See COMPLETE file content (not excerpts)
- Can reference specific line numbers
- Understand file context and structure
- Avoid "missing implementation" false positives

**What went wrong in our cpu_detection.inc verification:**
- Claude manually extracted first 90 lines only
- Both AIs reported "missing implementation" for `detect_amd_epyc_cpus()`
- Function implementation WAS present, just not in provided excerpt
- Using `@inc/cpu_detection.inc` would have avoided this entirely

## Error Handling

### Codex CLI Errors

```bash
# Check if codex is available
if ! command -v codex &> /dev/null; then
    echo "ERROR: Codex CLI not found. Please install from OpenAI."
    echo "Visit: https://docs.openai.com/codex"
    exit 1
fi

# Handle authentication errors
if codex exec "test" 2>&1 | grep -q "authentication"; then
    echo "ERROR: Codex authentication failed. Run: codex login"
    exit 1
fi

# Handle model availability
if codex exec "test" 2>&1 | grep -q "model not found"; then
    echo "WARNING: Model not available, check codex models"
    # Try to detect available models
fi
```

### Gemini CLI Errors

```bash
# Check if gemini is available
if ! command -v gemini &> /dev/null; then
    echo "ERROR: Gemini CLI not found. Please install Gemini CLI."
    echo "Visit: https://github.com/google-gemini/gemini-cli"
    exit 1
fi

# Handle API errors
if gemini -p "test" 2>&1 | grep -q "API key"; then
    echo "ERROR: Gemini API key not configured"
    exit 1
fi
```

## Best Practices

### Prompt Engineering

1. **Be Specific**: Clearly state what aspect to focus on (security, performance, architecture)
2. **Provide Context**: Include relevant background information in prompts
3. **Request Structure**: Ask for structured output (numbered lists, severity ratings)
4. **Examples**: Request code examples for suggested fixes

### Comparative Analysis

1. **Identify Patterns**: Look for common themes across both AIs
2. **Weight Consensus**: Issues both identify are likely valid
3. **Investigate Conflicts**: Disagreements may reveal nuanced considerations
4. **Human Judgment**: Use AI analysis to inform, not replace, human decision-making

### Workflow Optimization

1. **Cache Prompts**: Reuse prompt templates for consistency
2. **Parallel Execution**: Run both AIs concurrently when possible
3. **Output Preservation**: Save verification results for future reference
4. **Iteration**: Use findings to refine code/plans, then re-verify

## Example Usage Scenarios

### Scenario 1: Code Review for New Feature (File-Based)

```
User: "I need to verify this new authentication module"

AI Cross-Verifier:
  1. Select Mode: Both (Compare)
  2. Target: File - @inc/auth_module.inc
  3. Type: Code Review
  4. Execute: Run Codex + Gemini in parallel using @filepath
     - codex exec "@inc/auth_module.inc" "Review for security issues..."
     - gemini "@inc/auth_module.inc" "Review for security issues..."
  5. Output: Comparative analysis highlighting consensus security concerns

Result: Both AIs see COMPLETE file, avoiding truncation errors
```

### Scenario 2: Plan Validation for Refactoring (Inline Text)

```
User: "Validate my plan to migrate from MySQL to PostgreSQL"

AI Cross-Verifier:
  1. Select Mode: Both (Compare)
  2. Target: Plan/Text - [user pastes plan]
  3. Type: Plan Validation
  4. Execute: Run Codex + Gemini analysis with inline content
     - codex exec "ROLE: Architect... PLAN: ${USER_PLAN}"
     - gemini "ROLE: Architect... PLAN: ${USER_PLAN}"
  5. Output: Comparative report on completeness, risks, missing steps

Use Case: AI-generated plan not yet saved to file
```

### Scenario 3: Quick Gemini-Only Check (Code Snippet)

```
User: "Quick check this function for edge cases"

AI Cross-Verifier:
  1. Select Mode: Gemini Only
  2. Target: Code Snippet - [user pastes function]
  3. Type: Code Review
  4. Execute: Run Gemini verification with inline snippet
     - gemini "Review this function: ${CODE_SNIPPET}"
  5. Output: Single-AI report with edge case analysis

Use Case: Partial code or AI-generated snippet verification
```

### Scenario 4: Multi-File Consistency Check (NEW - File-Based)

```
User: "Check if CPU detection logic is consistent across installer variants"

AI Cross-Verifier:
  1. Select Mode: Both (Compare)
  2. Target: Files - @inc/cpu_detection.inc @installer-el10.sh @betainstaller.sh
  3. Type: Code Review
  4. Execute: Run Codex + Gemini with multi-file references
     - codex exec "@inc/cpu_detection.inc @installer-el10.sh @betainstaller.sh" \
       "Compare CPU detection implementations across these files..."
     - gemini "@inc/cpu_detection.inc @installer-el10.sh @betainstaller.sh" \
       "Compare CPU detection implementations across these files..."
  5. Output: Comparative analysis showing cross-file consistency issues

Result: Advanced multi-file verification impossible without @filepath support
```

### Scenario 5: Documentation vs Implementation Verification (NEW - File-Based)

```
User: "Verify that CLAUDE-redis-server.md accurately documents the actual implementation"

AI Cross-Verifier:
  1. Select Mode: Both (Compare)
  2. Target: Files - @CLAUDE-redis-server.md @addons/redis-server-install.sh
  3. Type: Both Review + Plan
  4. Execute: Run Codex + Gemini with doc + code references
     - codex exec "@CLAUDE-redis-server.md @addons/redis-server-install.sh" \
       "Verify documentation accuracy against implementation..."
     - gemini "@CLAUDE-redis-server.md @addons/redis-server-install.sh" \
       "Verify documentation accuracy against implementation..."
  5. Output: Comparative analysis identifying documentation gaps and code discrepancies

Result: Ensures memory bank accuracy by cross-referencing docs with actual code
```

## Success Criteria

A verification is successful when:

1. ✓ Appropriate AI model(s) selected based on user choice
2. ✓ Verification prompts are clear and comprehensive
3. ✓ CLI commands execute without errors
4. ✓ Output is captured and formatted properly
5. ✓ Comparative analysis (if both) identifies consensus and conflicts
6. ✓ Report includes actionable recommendations
7. ✓ Confidence levels are assigned appropriately
8. ✓ User receives clear, structured feedback

## Limitations

This skill:

- Cannot execute code changes (verification only)
- Relies on external CLI tools (codex, gemini)
- Requires valid API credentials for both services
- May have different model capabilities over time
- Provides analysis, not definitive answers
- Should supplement, not replace, human code review
- May have rate limits or API costs

## When to Use This Skill

Claude will automatically invoke this skill when you:

- Request verification of Claude's code or plans
- Ask for "second opinion" or "cross-check"
- Mention "Codex" or "Gemini" verification
- Request multi-AI comparison or consensus
- Want validation before implementing Claude's suggestions
- Need independent review of technical decisions
- Ask to "verify with other AI models"

## Quick Reference Commands

### File-Based Verification

```bash
# Single file verification (Codex)
codex exec "@inc/cpu_detection.inc" "Review this file for bugs and performance issues"

# Single file verification (Gemini)
gemini "@addons/redis-server-install.sh" "Validate this installation script"

# Multi-file comparison (Codex)
codex exec "@file1.sh @file2.sh" "Compare these implementations"

# Multi-file consistency check (Gemini)
gemini "@inc/nginx_install.inc @inc/php_configure.inc" "Check compiler flag consistency"

# Documentation vs implementation (Codex)
codex exec "@CLAUDE-redis.md @addons/redis-server-install.sh" \
  "Verify implementation matches documentation"
```

### Inline Content Verification

```bash
# Code snippet verification
codex exec "Review this code: [paste code here]"

# Plan validation
gemini "Validate this plan: [paste plan here]"
```

### Parallel Execution

```bash
# File-based parallel execution
PROMPT="Review this file for security and performance issues"
(codex exec "@inc/cpu_detection.inc" "$PROMPT" > /tmp/codex.txt 2>&1) &
(gemini "@inc/cpu_detection.inc" "$PROMPT" > /tmp/gemini.txt 2>&1) &
wait

# Inline content parallel execution
PROMPT="Review this code: ${CODE_SNIPPET}"
(codex exec "$PROMPT" > /tmp/codex.txt) &
(gemini "$PROMPT" > /tmp/gemini.txt) &
wait

# View outputs
cat /tmp/codex.txt
cat /tmp/gemini.txt
```

### Utility Commands

```bash
# Check CLI availability
command -v codex && command -v gemini

# Get current working directory (for relative @paths)
pwd

# Test @filepath resolution
codex exec "@inc/cpu_detection.inc" "Summarize this file in one sentence"
```

## Additional Resources

- **OpenAI Codex Documentation**: https://platform.openai.com/docs
- **Google Gemini CLI**: https://github.com/google-gemini/gemini-cli
- **Comparative AI Analysis**: Research on multi-model consensus
- **Prompt Engineering**: Best practices for AI verification prompts
