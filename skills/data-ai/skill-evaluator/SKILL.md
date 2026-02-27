---
name: skill-evaluator
description: Evaluate Claude Code skills against best practices including size, structure, examples, and prompt engineering quality. Provides comprehensive analysis with actionable suggestions.
---

# Claude Code Skill Evaluator

Systematically evaluate Claude Code skills for quality, compliance with best practices, and optimization opportunities. Provides detailed assessment with actionable suggestions for improvement.

## Table of Contents

- [Instructions](#instructions)
  - [1. Find Skill](#1-find-skill)
  - [2. Read the Skill File](#2-read-the-skill-file)
  - [3. Analyze Against Best Practices](#3-analyze-against-best-practices)
    - [Dimension 1: Size & Length](#dimension-1-size--length)
    - [Dimension 2: Scope Definition](#dimension-2-scope-definition)
    - [Dimension 3: Description Quality](#dimension-3-description-quality)
    - [Dimension 4: Structure & Organization](#dimension-4-structure--organization)
    - [Dimension 5: Examples](#dimension-5-examples)
    - [Dimension 6: Anti-Pattern Detection](#dimension-6-anti-pattern-detection)
    - [Dimension 7: Prompt Engineering Quality](#dimension-7-prompt-engineering-quality)
    - [Dimension 8: Completeness](#dimension-8-completeness)
  - [4. Generate Comprehensive Evaluation Report](#4-generate-comprehensive-evaluation-report)
  - [5. Deliver Report to User](#5-deliver-report-to-user)
- [Important Guidelines](#important-guidelines)
- [Requirements](#requirements)
- [Context & Standards](#context--standards)

## Instructions

### 1. Find Skill

Identify the skill passed in the directory passed to you or find all in the user's `~/.claude/skills/` directory. For each directory (excluding hidden files), verify it contains a `SKILL.md` file.

Present the user with:
- List of available skills
- Ask which skill to evaluate (or accept skill name as input)

### 2. Read the Skill File

Once a skill is selected, read its `SKILL.md` file and extract:
- Frontmatter metadata (name, description)
- Total line count
- Word count
- Character count
- Structure and sections

### 3. Analyze Against Best Practices

Evaluate the skill across **8 dimensions**:

#### Dimension 1: Size & Length
**Guidelines:**
- Body: Under 500 lines (hard maximum)
- Name: Maximum 64 characters
- Description: Maximum 1024 characters (200 char summary preferred)
- Table of Contents: Include if over 100 lines

**Assessment:**
- Count total lines in SKILL.md body
- Flag if over 500 lines
- Compliment if well-sized (ideal: 100-300 lines for medium skills)
- Check if TOC exists (expected for 100+ line skills)

#### Dimension 2: Scope Definition
**Guidelines:**
- Narrow focus (one skill = one capability)
- Clear boundary of what the skill does and doesn't do
- No scope creep (e.g., "document processing" → "PDF form filling")

**Assessment:**
- Does the description clearly state what the skill does?
- Are there multiple conflicting capabilities within one skill?
- Is the boundary clear to a new user?

#### Dimension 3: Description Quality
**Guidelines:**
- Third-person voice (avoid "I can" or "you can")
- Include both WHAT and WHEN TO USE
- Specific, searchable terminology
- 200 character summary ideal

**Assessment:**
- Voice and tone appropriate?
- Discovery terms clear? (Would users search for these terms?)
- Is "when to use" explained?

#### Dimension 4: Structure & Organization
**Guidelines:**
- Clear section hierarchy (headings, subsections)
- Logical flow (progressive disclosure)
- Step-by-step instructions preferred for workflows
- Rules/constraints clearly stated

**Assessment:**
- Is structure logical?
- Can a user easily navigate?
- Are instructions sequential or scattered?

#### Dimension 5: Examples
**Guidelines:**
- Quality over quantity
- Typical: 2-3 examples for basic skills, more for format-heavy
- Concrete (not abstract)
- Show patterns and edge cases

**Assessment:**
- How many examples? (count them)
- Are examples concrete and realistic?
- Do they demonstrate key patterns?
- Are there enough to show variations?

#### Dimension 6: Anti-Pattern Detection
**Red flags (check for these):**
- ❌ Windows-style paths (should use forward slashes)
- ❌ Magic numbers without justification
- ❌ Vague terminology (inconsistent synonyms)
- ❌ Time-sensitive instructions (date-dependent)
- ❌ Deeply nested file references (over 2 levels)
- ❌ Vague descriptions (missing WHAT or WHEN)
- ❌ Scope creep (trying to do too much)
- ❌ No error handling or validation steps
- ❌ No user feedback loops (for complex workflows)
- ❌ Multiple conflicting approaches for same task

**Assessment:**
- Count violations
- Severity of each violation
- Impact on usability

#### Dimension 7: Prompt Engineering Quality
**Guidelines:**
- Imperative language (verb-first instructions)
- Explicit rules with clear boundaries
- Validation loops where appropriate (especially for destructive ops)
- Clear error handling
- Assumes user is intelligent (don't over-explain)

**Assessment:**
- Is language imperative?
- Are there validation steps?
- How clear are the rules?
- Is error handling explicit?

#### Dimension 8: Completeness
**Guidelines:**
- Requirements listed (what's needed to use the skill)
- Edge cases acknowledged
- Limitations stated where relevant

**Assessment:**
- Are prerequisites clear?
- Are limitations or edge cases mentioned?
- Is scope of responsibility clear?

### 4. Generate Comprehensive Evaluation Report

Create a detailed evaluation report with these components:

1. **Executive Summary**: 1-2 paragraphs covering overall assessment, key strengths, and critical issues

2. **Metrics**: Present line count, word count, character count, and guideline compliance assessment

3. **Dimensional Analysis**: For each of the 8 dimensions:
   - Status indicator (✓ Pass / ⚠ Warning / ❌ Fail)
   - 1-2 sentence assessment explaining the rating

4. **Detected Issues**: Organize by severity:
   - Critical Issues (must fix) - any ❌ Fail items with explanation
   - Warnings (should address) - any ⚠ Warning items with explanation
   - Observations (minor items worth noting)

5. **Comparative Analysis**: Compare the skill against official skills repository patterns with examples and rationale

6. **Actionable Suggestions**: Numbered list of specific improvements, prioritized by impact:
   - High Priority (do this first)
   - Medium Priority (nice to have)
   - Low Priority (optional refinements)

   Each suggestion should include concrete rationale, not vague guidance.

7. **Overall Assessment**:
   - Professional verdict on production-readiness
   - Clear recommendation (Keep as-is / Minor tweaks / Significant refactor / Major restructure)

### 5. Deliver Report to User

Present the complete evaluation report to the user in a clear, formatted structure. Ensure:
- Status indicators are visible (✓ Pass / ⚠ Warning / ❌ Fail)
- Actionable suggestions are specific (not vague)
- Rationale is explained for each issue
- Prioritization is clear

## Important Guidelines

- **Be brutally honest**: Point out real issues, don't sugarcoat
- **Specific over vague**: "The examples don't show error handling" not "examples could be better"
- **Professional tone**: Constructive criticism, not harsh
- **Evidence-based**: Reference specific lines or patterns from the skill
- **Proportional feedback**: Don't over-critique minor issues
- **Future-focused**: Suggest improvements, not judgment

## Requirements

- User has installed skills in `~/.claude/skills/`
- Target skill has a valid `SKILL.md` file with frontmatter
- User accepts the detailed, honest evaluation

## Context & Standards

This evaluator uses best practices from:
- Official Anthropic Claude Code Skills documentation
- Analysis of official skills repository patterns
- Professional technical writing standards
- Prompt engineering best practices for LLM interactions

All assessments are comparative to **official guidelines**, not arbitrary standards.
