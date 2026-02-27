---
name: systematic-debugging
description: Structured debugging methodology for investigating and resolving technical issues. Use this skill when debugging any technical problem, investigating errors, or troubleshooting unexpected behavior to ensure root cause analysis rather than symptom treatment.
---

# Systematic Debugging Process

## Overview

This skill provides a disciplined, phase-based approach to debugging that ensures root cause identification rather than symptom treatment. The methodology emphasizes systematic investigation, hypothesis testing, and minimal incremental fixes.

## When to Use This Skill

Use this skill whenever:
- Debugging a technical issue or error
- Investigating unexpected behavior
- Troubleshooting test failures
- Resolving performance problems
- Analyzing system failures

## Core Principle

**ALWAYS find the root cause of any issue. NEVER fix a symptom or add a workaround instead of finding the root cause, even if it appears faster or more expedient.**

## Four-Phase Debugging Framework

### Phase 1: Root Cause Investigation

**Perform BEFORE attempting any fixes.** This phase focuses on understanding the problem completely.

#### Read Error Messages Carefully
- Don't skip past errors or warnings
- Error messages often contain the exact solution
- Note stack traces, line numbers, and context
- Look for patterns in multiple error occurrences

#### Reproduce Consistently
- Ensure the issue can be reliably reproduced
- Identify the minimal steps to trigger the problem
- Note any conditions that affect reproducibility
- Document the exact environment and context

#### Check Recent Changes
- Review what changed that could have caused this
- Use `git diff` to examine recent modifications
- Check recent commits and their impact
- Consider dependency updates or configuration changes

### Phase 2: Pattern Analysis

This phase involves comparing working and non-working states to identify differences.

#### Find Working Examples
- Locate similar working code in the same codebase
- Identify what makes the working example successful
- Note the context and dependencies of working code

#### Compare Against References
- If implementing a pattern, read the reference implementation completely
- Don't skim documentation—read it thoroughly
- Understand the intended usage and requirements
- Identify any prerequisites or setup steps

#### Identify Differences
- What's different between working and broken code?
- Compare structure, dependencies, and configuration
- Look for missing imports, incorrect types, or wrong parameters
- Check for environmental differences

#### Understand Dependencies
- What other components does this code depend on?
- Are there configuration files or settings required?
- What initialization or setup is needed?
- Are there version compatibility issues?

### Phase 3: Hypothesis and Testing

This phase requires forming clear hypotheses and testing them methodically.

#### 1. Form Single Hypothesis
- State clearly what you think is the root cause
- Base the hypothesis on evidence from previous phases
- Be specific about the suspected problem
- Example: "The error occurs because X is undefined when Y runs"

#### 2. Test Minimally
- Make the smallest possible change to test the hypothesis
- Change only ONE thing at a time
- Avoid adding multiple fixes simultaneously
- Keep changes reversible and isolated

#### 3. Verify Before Continuing
- Did the test work? Did it fix the problem?
- If not, form a new hypothesis—don't add more fixes
- Understand why the hypothesis was wrong
- Learn from failed hypotheses to refine the investigation

#### 4. When You Don't Know
- Say "I don't understand X" rather than pretending to know
- Ask for clarification or additional information
- Admit knowledge gaps honestly
- Seek help when needed

### Phase 4: Implementation Rules

Once the root cause is confirmed, follow these rules during implementation:

#### Always Have a Failing Test Case
- Create the simplest possible test that reproduces the issue
- If there's no test framework, write a one-off test script
- The test should fail before the fix and pass after
- Keep the test case minimal and focused

#### Never Add Multiple Fixes at Once
- Fix one thing at a time
- Test after each change
- Understand the impact of each modification
- Avoid the temptation to "fix everything"

#### Never Claim to Implement a Pattern Without Reading It
- Read reference implementations completely first
- Understand the pattern thoroughly before implementing
- Don't guess at how something should work
- Follow documented patterns exactly

#### Always Test After Each Change
- Run tests immediately after each modification
- Verify the fix works as expected
- Check for regressions in other functionality
- Confirm the original error is resolved

#### If First Fix Doesn't Work, STOP and Re-analyze
- Don't pile on additional fixes
- Return to Phase 1 or Phase 2
- Form a new hypothesis based on new information
- Avoid the "shotgun debugging" anti-pattern

## Anti-Patterns to Avoid

- **Symptom Treatment**: Fixing what appears broken without understanding why
- **Shotgun Debugging**: Making multiple changes hoping something works
- **Assumption-Based Fixes**: Guessing at solutions without investigation
- **Workaround Mentality**: Adding code to bypass problems rather than fixing them
- **Impatient Debugging**: Rushing to fix without understanding the root cause
