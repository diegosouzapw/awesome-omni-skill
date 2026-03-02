---
name: ask-the-oracle
description: Submit complex code questions to OpenAI GPT-5 Pro for deep analysis when you have 20 minutes. Use when the user asks for architectural analysis, comprehensive code review, debugging complex issues, or requests expert analysis of their codebase that requires extended reasoning. Automatically handles file selection, code packing with Repomix, cost estimation, long-running API requests, and result presentation.
allowed-tools: Read, Write, Grep, Glob, Bash, AskUserQuestion
---

# Ask the Oracle

Inspired by Andrej Karpathy's approach of consulting GPT-5 Pro as an "Oracle" for complex code questions that require 20+ minutes of deep reasoning.

## When to Use This Skill

Use this skill when the user:
- Asks for **architectural analysis or design review**
- Needs **debugging help for complex, non-obvious issues**
- Requests **comprehensive code review** across multiple files
- Wants **expert analysis** beyond my immediate capabilities
- Says things like "deep dive", "expert analysis", "comprehensive review", "consult the Oracle"
- Is willing to wait 10-20 minutes for high-quality results
- Has a question that requires reasoning across large amounts of code

## When NOT to Use This Skill

Skip this skill when:
- The question is simple and I can answer it directly
- The user needs an immediate response
- The task is about editing or generating code (not analysis)
- The question is conversational or doesn't require codebase context

## Instructions

### Phase 1: Understand the Question

1. **Capture the user's question**
   - What specifically do they want to know?
   - What problem are they trying to solve?

2. **Determine scope**
   - Which files/directories are relevant?
   - Is this about architecture, bugs, performance, security, or best practices?

### Phase 2: Select Files

1. **Use Glob to identify relevant files**
   - Search for patterns matching the question scope
   - Examples:
     - `**/*.{js,ts}` for JavaScript/TypeScript projects
     - `src/**/*.py` for Python source code
     - `**/*.{go,mod}` for Go projects

2. **Ask user to confirm or refine file selection** using AskUserQuestion
   - Present the file count and pattern
   - Allow them to add/remove patterns
   - Example: "I found 45 JavaScript files in src/. Should I include tests/ as well?"

3. **Show estimated token count**
   - Use Bash to run: `cd <project-root> && node .claude/skills/ask-the-oracle/scripts/oracle.js <patterns> -- "test" 2>&1 | grep "tokens"`
   - This will show the token estimate from Repomix

### Phase 3: Formulate the Question

1. **Ask clarifying questions** using AskUserQuestion if needed:
   - "What specific aspect are you investigating?" (architecture/bugs/performance/etc.)
   - "What is the expected behavior or outcome?"
   - "Have you tried any debugging steps already?"

2. **Format the question clearly**
   - Start with context: "This is a [type] project with [key characteristics]"
   - State the specific question
   - Add any constraints or focus areas
   - Example:
     ```
     This is a Node.js Express API with TypeScript. The codebase handles user authentication
     and data processing. I'm experiencing memory leaks that occur after several hours of
     operation, particularly related to request handlers. Please analyze the code and:
     1. Identify potential memory leak sources
     2. Explain why the async refactor may have triggered this
     3. Suggest specific fixes with code examples
     ```

### Phase 4: Estimate Cost and Confirm

1. **Calculate estimated cost**
   - Repomix will show token count when packing
   - GPT-5 Pro pricing: $15/M input, $120/M output
   - Typical costs: $2-10 depending on codebase size

2. **Check against limits** (from .oraclerc)
   - If cost > $5, inform user and ask for confirmation
   - If cost > configured limit, don't proceed without explicit approval

3. **Confirm with user** using AskUserQuestion:
   - Show estimated cost
   - Confirm they want to proceed
   - Remind them it may take 20 minutes

### Phase 5: Submit to Oracle

1. **Run the oracle script** using Bash:
   ```bash
   cd <project-root> && node .claude/skills/ask-the-oracle/scripts/oracle.js <patterns> -- "<question>"
   ```

   Example:
   ```bash
   cd /Users/robgruhl/Projects/my-app && \
   node .claude/skills/ask-the-oracle/scripts/oracle.js \
   "src/**/*.ts" "lib/**/*.ts" -- \
   "Analyze this codebase for memory leaks. Focus on async handlers and event listeners."
   ```

2. **The script will**:
   - Pack code with Repomix
   - Show cost estimate
   - Submit to GPT-5 Pro (background mode)
   - Poll every 3 seconds for completion
   - Display progress updates
   - Save response to history

3. **Monitor the output**
   - The script will show status updates
   - Wait for completion (don't timeout Claude Code!)
   - Typical time: 10-20 minutes

### Phase 6: Present Results

1. **The oracle script will display**:
   - Provider and model used
   - Time elapsed
   - Cost breakdown (input/reasoning/output tokens)
   - Full response from GPT-5 Pro

2. **Read the history file** using Read tool:
   - Location: `.claude/oracle-history/oracle-<timestamp>.json`
   - Contains full response and metadata

3. **Present to user**:
   - Summarize key findings
   - Highlight actionable recommendations
   - Show cost and time metrics
   - Ask if they have follow-up questions

### Error Handling

**If .oraclerc not found:**
- Inform user they need to set up configuration
- Point them to .oraclerc.example
- Explain they need an OpenAI API key

**If API key invalid:**
- Check if OPENAI_API_KEY environment variable is set
- Suggest checking their API key configuration
- Remind them they need GPT-5 Pro access

**If cost limit exceeded:**
- Show the estimated cost
- Ask if they want to proceed anyway
- Suggest ways to reduce cost (fewer files, compression)

**If timeout:**
- The script has a 25-minute timeout
- If it times out, the request might still be processing
- Response ID is saved, can be retrieved later

**If request fails:**
- Show the error message from OpenAI
- Suggest checking API status
- Offer to retry

## Important Notes

- **Cost**: Typically $2-10 per request depending on codebase size
- **Time**: Expect 10-20 minutes for responses (GPT-5 Pro uses extended reasoning)
- **API Access**: Requires OpenAI API key with GPT-5 Pro access
- **Privacy**: Code is sent to OpenAI (retained for 30 days per their policy)
- **History**: All consultations are saved to `.claude/oracle-history/`
- **Limits**: Configurable cost limits in .oraclerc prevent accidental overspending

## Example Usage

### Example 1: Memory Leak Analysis

**User**: "I have a memory leak in my Node.js service. Can you do a deep analysis?"

**Claude Code**:
1. Uses Glob to find: `**/*.js`, `src/**/*.ts`
2. Asks: "I found 42 files. Should I include tests?"
3. Asks: "What symptoms are you seeing?" → "Gradual memory growth, crashes at 2GB"
4. Asks: "When did this start?" → "After async refactor"
5. Shows estimate: ~$3.20 (45K tokens input)
6. Confirms: "Proceed?"
7. Runs oracle script
8. Waits 18 minutes
9. Presents detailed analysis with fixes

### Example 2: Architecture Review

**User**: "Review the architecture of my API"

**Claude Code**:
1. Uses Glob: `src/api/**/*.ts`, `src/models/**/*.ts`
2. Estimates: 80K tokens (~$5.40)
3. Asks confirmation (exceeds $5 threshold)
4. User confirms
5. Runs oracle script
6. Waits 22 minutes
7. Presents architecture analysis with recommendations

## Configuration

Users must have `.oraclerc` in project root:

```json
{
  "defaultProvider": "openai",
  "providers": {
    "openai": {
      "apiKey": "$OPENAI_API_KEY",
      "model": "gpt-5-pro",
      "enabled": true
    }
  },
  "limits": {
    "maxCostPerRequest": 10.00,
    "warnCostThreshold": 5.00
  }
}
```

## Tips for Best Results

1. **Be specific** in questions - the Oracle works best with clear, focused questions
2. **Select relevant files only** - more code = higher cost and longer processing
3. **Provide context** - explain what you've tried and what you're looking for
4. **Use for complex issues** - save simple questions for me directly
5. **Follow up** - you can ask clarifying questions after getting the response

---

**Remember**: This skill makes external API calls that cost money and take time. Always confirm with the user before submitting, and set clear expectations about the 20-minute wait time.
