---
name: understando
description: Pre-commit quiz that tests your understanding of code changes before allowing commits. Use when committing code through Claude Code to ensure engineers understand what they're committing. Requires hook installation - see repository README for setup.
license: MIT
allowed-tools: Bash, Read, Glob, Grep, AskUserQuestion
---

# Understando - Pre-Commit Code Understanding Quiz

You are running a quiz to verify the user understands the code changes they are about to commit.

> **Setup Required**: This skill works best with the understando-gate hook installed. See the [repository](https://github.com/claytonkim/understando) for hook installation instructions.

## Quiz Flow

### Step 1: Load Configuration

Read `.understando.json` from the project root. If not found, use defaults:
- `default_questions`: 5
- `default_level`: "medium"
- `pass_threshold`: 0.8

### Step 2: Get the Diff

Run `git diff --staged` to get the changes being committed. If no staged changes, inform the user and exit.

### Step 3: Ask Difficulty Level

Prompt the user:
```
Select quiz difficulty:
1. Easy - Surface-level understanding
2. Medium - Moderate depth, some context required
3. Hard - Deep understanding, edge cases, architectural implications

Enter choice (1/2/3) or press Enter for default:
```

### Step 4: Generate Questions

Based on the diff and difficulty level, generate questions that test understanding:

**Easy questions** test:
- What a function/component does
- What files were modified
- Basic "what changed" comprehension

**Medium questions** test:
- Why a change was made (intent)
- How the change affects related code
- What the expected behavior is

**Hard questions** test:
- Edge cases the change might introduce
- Architectural implications
- Potential bugs or issues
- How this integrates with the broader system

**Question format mix:**
- ~60% multiple choice (4 options, one correct)
- ~40% free-text (short answer)

Present questions one at a time. For multiple choice, show options labeled A-D.

### Step 5: Judge Answers

**Multiple choice:** Exact match (A/B/C/D)

**Free-text:** Evaluate if the answer demonstrates understanding. Be lenient - accept answers that show comprehension even if not perfectly worded. Look for:
- Key concepts mentioned
- Correct understanding of the change
- Reasonable explanation

### Step 6: Calculate Results

After all questions:
- Calculate percentage correct
- If >= pass_threshold: PASS
- If < pass_threshold: FAIL

### Step 7: Show Results

**If PASSED:**
```
Quiz Complete! You scored X/Y (Z%)

Proceeding with commit...
```
Create a marker file `.understando-passed` in the project root, then execute the original commit command. The hook will see this marker and allow the commit through.

**If FAILED:**
```
Quiz Complete. You scored X/Y (Z%)

You need 80% to pass. Here's what you missed:

Question N: [question]
Your answer: [their answer]
Correct answer: [correct answer or explanation]
Why: [brief explanation]

[Repeat for each wrong answer]

Would you like to try again with new questions? (yes/skip)
```

- If "yes" or "y": Generate NEW questions (not the same ones), go back to Step 4
- If "skip": Show shame message, create `.understando-passed` marker, then execute the commit

### Shame Message

When user skips:
```
Skipping quiz...

Remember: Understanding your code changes helps catch bugs early
and makes you a better engineer. Consider reviewing the diff before
your next commit.

Proceeding with commit anyway...
```

## Important Notes

1. **Generate diverse questions** - Don't repeat similar questions across retries
2. **Be fair but thorough** - Questions should be answerable from the diff, not require external knowledge
3. **Keep it interactive** - Wait for user input after each question
4. **Create marker file** - Before committing, create `.understando-passed` file so the hook allows the commit through. The hook will automatically delete this file after the commit.

## Example Questions by Difficulty

**Easy:**
- "What function was modified in `src/utils.js`?" (MCQ)
- "How many files were changed in this commit?" (MCQ)
- "What is the new return type of the `calculate` function?" (Free-text)

**Medium:**
- "Why was the error handling added to `fetchUser`?" (Free-text)
- "Which component will be affected by this props change?" (MCQ)
- "What problem does the new validation logic solve?" (Free-text)

**Hard:**
- "What edge case could cause the new `parseInput` function to fail?" (Free-text)
- "How does this change affect the application's state management?" (Free-text)
- "Which of these scenarios would NOT be handled by the new error boundary?" (MCQ)
