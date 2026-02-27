---
name: testing-skills-activation
description: Use when creating or refining Claude Code skills to validate that skill descriptions trigger correctly - provides systematic testing methodology for skill activation patterns using test cases and automated evaluation
---

# Testing Skills Activation

## Overview

This skill provides a systematic methodology for testing and iterating on Claude Code skill descriptions to ensure they activate at the right times. It helps prevent both false positives (activating when they shouldn't) and false negatives (not activating when they should).

## When to Use This Skill

Use this skill when:
- Creating a new Claude Code skill and want to validate its description
- Refining an existing skill's description to improve activation accuracy
- A skill is activating too often (false positives) or not often enough (false negatives)
- You want to measure the effectiveness of skill description changes

## The Process

### Step 1: Review Best Practices

Before testing, review what makes effective skill descriptions.

**Read the best practices reference:**
See `best-practices-reference.md` in this skill directory for comprehensive guidelines on writing effective skill descriptions.

**Key principles:**
- Description field is THE primary discovery mechanism
- Lead with strongest triggers (technology names, file types, activities)
- Include specific examples (library names, framework names)
- State both what IS and what is NOT in scope
- Use user language, not technical jargon
- Target 90%+ accuracy

### Step 2: Generate Test Cases

Create a diverse set of test scenarios in `test-cases.json` within the skill directory being tested:

```json
[
  {
    "id": 1,
    "user_message": "create a new hook to fetch data at /api/users",
    "project_context": "React project using react-query v5, TypeScript",
    "expected_activation": true,
    "rationale": "User mentions creating a hook in a react-query project - clear library usage"
  },
  {
    "id": 2,
    "user_message": "iterate through this list and extract property x",
    "project_context": "FastAPI Python backend project",
    "expected_activation": false,
    "rationale": "Pure algorithmic task, no library-specific functionality"
  }
]
```

**Test case coverage:**
- **True positives:** Scenarios where skill SHOULD activate
- **True negatives:** Scenarios where skill should NOT activate
- **Edge cases:** Ambiguous scenarios that test boundaries
- **Diverse technologies:** Cover different libraries/frameworks the skill targets

**Aim for 15-25 test cases** with roughly 60% positive, 40% negative.

### Step 3: Run Baseline Tests

Use the `run-tests.sh` script from this skill directory:

```bash
cd /path/to/skill-being-tested
/path/to/testing-skills-activation/run-tests.sh
```

The script will:
1. Run tests in batches of 5 using `claude -p`
2. Auto-detect skill invocation intention in responses
3. Ask for confirmation (y/n/override)
4. Generate results JSON and markdown report in `/tmp/claude/`
5. Calculate accuracy metrics

**What counts as "activation":**
- Claude mentions the skill by name
- Claude says it would "invoke" or "use" the skill
- Claude directly uses MCP tools the skill wraps (if applicable)

### Step 4: Analyze Results

Review the generated report in `/tmp/claude/test-report-TIMESTAMP.md`:

**Key metrics to examine:**
- **Accuracy:** Overall percentage (target: 90%+)
- **False Positives:** Skill activated when it shouldn't (too broad)
- **False Negatives:** Skill didn't activate when it should (missing triggers)
- **True Positives/Negatives:** Correct activations and non-activations

**Pattern analysis:**
- Group failures by type (library, activity, context)
- Identify competing skills (other skills activating instead)
- Look for missing keywords in failed cases
- Check if description is too broad or too narrow

### Step 5: Iterate on Description

Based on failure patterns, refine the skill description:

**For False Positives (too broad):**
- Add explicit negative examples: "Do NOT use for..."
- Tighten scope with qualifiers: "third-party libraries" not just "libraries"
- Be more specific about trigger conditions

**For False Negatives (too narrow):**
- Add specific library/framework names mentioned in failures
- Lead with most important trigger
- Include synonym patterns users might use
- Add activity verbs: "implementing", "debugging", "configuring"

**Common improvements:**
```
# Before (vague)
"Use when working with libraries"

# After (specific)
"Use when working with third-party libraries (react-query, FastAPI, Django)
for implementing features or debugging library behavior. NOT for language
built-ins (dict, Array) or pure algorithms."
```

### Step 6: Re-test and Measure

After updating the description:
1. Re-run the same test cases
2. Compare new accuracy with baseline
3. Check if failures shifted (e.g., fixed FN but created FP)
4. Iterate again if accuracy < 90%

**Iteration cycle:**
- Baseline: Document starting accuracy
- Iteration 1: First refinement + re-test
- Iteration 2: Second refinement + re-test
- Continue until 90%+ or diminishing returns

## The Testing Script

Located at: `superpowers/skills/testing-skills-activation/run-tests.sh`

**Usage:**
```bash
# From the skill directory being tested
/path/to/testing-skills-activation/run-tests.sh

# Or with absolute path
cd /home/user/my-skill
/home/user/testing-skills-activation/run-tests.sh
```

**What it does:**
1. Looks for `test-cases.json` in current directory (skill being tested)
2. Runs tests in batches of 5 to reduce waiting
3. Auto-detects skill invocation with pattern matching
4. Saves results to `/tmp/claude/test-results-TIMESTAMP.json`
5. Generates report to `/tmp/claude/test-report-TIMESTAMP.md`
6. Outputs summary with accuracy percentage

**Auto-detection patterns:**
- Skill name mention: "using-live-documentation"
- Invocation intention: "invoke.*skill-name", "use.*skill-name"
- MCP tool usage: "mcp__context7" (configurable per skill)

**Manual override:**
- `y` = detection was correct
- `n` = flip the detection
- `override` = ignore detection, manually specify

## Test Case Design Guidelines

### Good Test Cases

**Specific and realistic:**
```json
{
  "user_message": "implement optimistic updates for the todo mutations",
  "project_context": "React with react-query v5, creating a todo app",
  "expected_activation": true,
  "rationale": "Optimistic updates is a react-query specific concept"
}
```

**Clear boundary testing:**
```json
{
  "user_message": "configure the logger to write errors to a file",
  "project_context": "Python application using standard logging module",
  "expected_activation": false,
  "rationale": "Standard library, not third-party"
}
```

### Poor Test Cases

**Too vague:**
```json
{
  "user_message": "help me with my code",
  "project_context": "Some project",
  "expected_activation": true
}
```

**Ambiguous expectation:**
```json
{
  "user_message": "fix the error",
  "project_context": "React app",
  "expected_activation": true,
  "rationale": "Could be library issue or logic bug - unclear"
}
```

## Skill Competition Issues

Sometimes skills don't activate because other skills take priority:

**Example: Documentation skill vs. Debugging skill**
- User: "the useQuery hook is returning stale data"
- Expected: documentation skill (library-specific behavior)
- Actual: systematic-debugging skill (bug keyword triggered)

**Solutions:**
1. Make descriptions more specific about priority
2. Add "especially when..." clauses to handle overlap
3. Document known competition in skill's internal notes
4. Accept some competition as unavoidable (90% is the target, not 100%)

## Common Pitfalls

### 1. Testing with Wrong Mindset

**Wrong:** "Did the skill auto-activate?"
- Can't test auto-activation with `claude -p` (no skill system)

**Right:** "Would Claude decide to invoke this skill?"
- Tests intention recognition, which correlates with discovery

### 2. Too Few Test Cases

**Wrong:** 5 test cases
- Not enough coverage to identify patterns

**Right:** 15-25 test cases
- Diverse scenarios reveal edge cases

### 3. Only Testing Positive Cases

**Wrong:** All tests expect activation
- Can't detect if description is too broad

**Right:** Mix of positive (60%) and negative (40%)
- Validates both activation AND non-activation

### 4. Ignoring Context in Prompts

**Wrong:** Just the user message
- Claude asks for clarification instead of acting

**Right:** Full context with "assume you know what user refers to"
- Claude focuses on next action, not information gathering

## Integration with Other Workflows

### With Skill Creation

1. Draft initial skill description
2. Generate test cases
3. Run baseline tests
4. Iterate description
5. Document final accuracy in skill's internal notes

### With Skill Updates

1. Note the change being made
2. Run tests before and after
3. Ensure accuracy doesn't decrease
4. Update test cases if skill scope changed

### With Best Practices Research

After researching new best practices:
1. Apply learnings to skill descriptions
2. Re-test existing skills
3. Update test cases to cover new patterns
4. Document new patterns in best practices doc

## Success Criteria

**Good enough (70-80%):**
- Skill activates for most intended scenarios
- Few false positives
- Acceptable for internal/personal use

**Production quality (90%+):**
- High precision and recall
- Minimal competition with other skills
- Ready for sharing/publishing

**Excellent (95%+):**
- Nearly perfect activation
- Rare false positives/negatives
- Well-scoped and well-documented

## Files and Locations

**Skill structure:**
```
skill-being-tested/
  ├── SKILL.md                    # Skill definition with description
  └── test-cases.json             # Test scenarios (create this)

testing-skills-activation/
  ├── SKILL.md                    # This documentation
  ├── best-practices-reference.md # Best practices for skill descriptions
  ├── run-tests.sh                # Test runner script
  └── README.md                   # Quick reference guide

/tmp/claude/
  ├── test-results-TIMESTAMP.json # Test results (auto-generated)
  └── test-report-TIMESTAMP.md    # Analysis report (auto-generated)
```

## Quick Start

```bash
# 1. Create test cases in skill directory
cd /path/to/my-skill
cat > test-cases.json << 'EOF'
[
  {
    "id": 1,
    "user_message": "your test message",
    "project_context": "project context",
    "expected_activation": true,
    "rationale": "why this should/shouldn't activate"
  }
]
EOF

# 2. Run tests
/path/to/testing-skills-activation/run-tests.sh

# 3. View report
cat /tmp/claude/test-report-*.md | tail -100

# 4. Iterate on SKILL.md description field

# 5. Re-run tests
/path/to/testing-skills-activation/run-tests.sh

# 6. Compare results and iterate until 90%+ accuracy
```

## Example: using-live-documentation Iteration

**Baseline (Before):**
- Description: "Use when implementing features, debugging, or answering questions about code, specially if it involves libraries/frameworks..."
- Accuracy: 50% (11/22)
- Issues: Too broad, buried the lead, competing with explore-first pattern

**Iteration 1 (After):**
- Description: "Use when working with third-party libraries or frameworks (react-query, FastAPI, pydantic, Django, Express, pandas, Next.js, NestJS, Celery, pytest, Pinia, etc.) for implementing features, debugging library-specific behavior, or answering API questions - fetches current documentation ensuring accurate implementation. Do NOT use for language built-ins (Python dict/list, JavaScript Array), standard library (fs, json, os.path), or pure algorithms."
- Accuracy: 73% (16/22)
- Improvement: +23 percentage points
- Remaining issues: Competition with other skills (systematic-debugging, requesting-code-review, using-beads)

**Key changes that worked:**
- Led with "third-party libraries or frameworks"
- Added specific library names for pattern matching
- Explicit negative examples
- Removed implementation details from description

## Summary

Testing skill activation is a systematic process:
1. Research best practices
2. Generate diverse test cases (15-25 cases, 60/40 split)
3. Run baseline tests with the script
4. Analyze failure patterns
5. Iterate on description
6. Re-test and measure improvement
7. Target 90%+ accuracy

The `run-tests.sh` script automates the testing, the `test-cases.json` defines scenarios, and the reports guide iteration. This process ensures skills activate reliably in production use.
