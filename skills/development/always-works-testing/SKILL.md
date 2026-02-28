---
name: always-works-testing
description: Default testing standard for all implementation work - ensures code actually works through mandatory execution validation before confirming to user. Applies automatically whenever writing, modifying, debugging, or implementing any code (scripts, APIs, UI, configs, data operations, logic changes). This is the baseline expectation, not an optional extra - every implementation must be verified through actual execution, not assumed correct.
---

# Always Works™ Testing Philosophy

Distinguish between "should work" (theory) and "does work" (reality) through systematic verification.

## Core Principles

1. **Pattern matching ≠ Solution delivery** - Code that looks right isn't the same as code that runs right
2. **Solve problems, not write code** - The goal is working functionality, not elegant untested logic
3. **Untested code = Guess** - Without execution verification, it's speculation

## The 30-Second Reality Check

Before claiming something works, answer YES to ALL:

1. **Did I run/build the code?** - Not just read it, actually execute it
2. **Did I trigger the exact feature I changed?** - Not similar code, the specific modification
3. **Did I see the expected result with my own observation?** - Including GUI, terminal output, logs
4. **Did I check for error messages?** - Stderr, console errors, warning logs
5. **Would I bet $100 this works?** - Confidence based on observation, not assumption

If any answer is NO or UNCERTAIN → Test before confirming to user.

## Forbidden Phrases

Don't use these without actual verification:

- "This should work now"
- "I've fixed the issue" (especially on 2nd+ attempt)
- "Try it now" (without trying it yourself)
- "The logic is correct so..."

These phrases signal untested assumptions. Replace with observation-based confirmations only after testing.

## Test Requirements by Change Type

Execute appropriate verification for each change:

- **UI Changes**: Click the actual button/link/form in a browser
- **API Changes**: Make the actual API call with curl/Postman/code
- **Data Changes**: Query the actual database and verify results
- **Logic Changes**: Run the specific scenario with test inputs
- **Config Changes**: Restart the service and verify it loads correctly
- **Script Changes**: Execute the script with representative inputs

## Application in Claude's Context

When working with computer tools:

1. **After writing code**: Use `bash_tool` to execute and verify output
2. **For file changes**: Use `view` to confirm changes, then test functionality
3. **For scripts**: Run with sample inputs, check exit codes and output
4. **For syntax**: Don't just check syntax - run the code
5. **For web content**: If creating HTML/JS, verify it actually renders/executes

## The Embarrassment Test

Before responding to user: "If the user records trying this and it fails, will I feel embarrassed to see their face?"

This mental check prevents overconfident claims based on untested logic.

## Reality Economics

- Time saved skipping tests: **30 seconds**
- Time wasted when it fails: **30 minutes**  
- User trust lost: **Immeasurable**

When users report the same bug repeatedly, they're not thinking "the AI is trying hard" - they're thinking "why am I wasting time with an unreliable tool?"

## Mandatory Workflow

Apply this sequence for every implementation:

1. **Write/modify code**
2. **Run the 30-Second Reality Check** - Honest self-assessment
3. **Execute actual test** - Use available tools to verify
4. **Observe results** - Check stdout, stderr, GUI, logs, database
5. **Confirm to user ONLY after verification** - Base confidence on observation

## When Full Testing Isn't Possible

If you cannot perform complete verification (no access to prod environment, missing credentials, etc.):

- **Explicitly state the limitation** to the user
- **List what you verified** and what you couldn't
- **Don't imply full verification** when only partial testing occurred
- **Recommend what the user should test** before deploying

Example: "I've verified the syntax and logic structure, but I cannot test the actual API calls without credentials. You should test: [specific scenarios]"
