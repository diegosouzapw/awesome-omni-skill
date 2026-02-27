---
name: codex-reviewer
description: Use OpenAI's Codex CLI as an independent code reviewer to provide second opinions on code implementations, architectural decisions, code specifications, and pull requests. Trigger when users request code review, second opinion, independent review, architecture validation, or mention Codex review. Provides unbiased analysis using GPT-5-Codex model through the codex exec command for non-interactive reviews.
---

# Codex Independent Code Reviewer

This skill enables Claude to leverage OpenAI's Codex CLI as an independent third-party reviewer for code, architectural decisions, and specifications. Codex runs as a separate AI agent with GPT-5-Codex, providing an unbiased second opinion to improve code quality and catch issues that might be missed in single-reviewer scenarios.

## When to Use This Skill

Use this skill when:
- User explicitly requests a "second opinion" or "independent review" on code
- User asks to "review with Codex" or mentions using Codex for validation
- Complex architectural decisions need validation from an independent perspective
- Code review needs additional scrutiny for security, performance, or correctness
- Pull requests require thorough review before merging
- Specifications or design documents need technical validation
- User asks "what would another AI think about this code?"

## Prerequisites

Before using this skill, verify:
1. Codex CLI is installed on the system (`which codex`)
2. User is authenticated (ChatGPT account or API key)
3. The current directory is within a git repository or project structure
4. For file-specific reviews, the target files exist in the working directory

## Core Review Workflow

### Step 1: Prepare Review Context

Identify what needs review:
- **Code files**: Specific files or directories
- **Pull requests**: Changes between branches
- **Architecture**: Design documents or implementation patterns
- **Specifications**: Requirements or technical specs

### Step 2: Execute Codex Review

Use `codex exec` for non-interactive reviews. The command runs Codex in a separate, isolated session:

**Basic syntax:**
```bash
codex exec [flags] "review prompt"
```

**Key flags:**
- `--model gpt-5-codex`: Use the specialized coding model (recommended)
- `--sandbox read-only`: **Always use this for reviews** - provides read-only access (required for review-only workflow)
- `-a/--ask-for-approval <mode>`: Control when to ask for approval before actions (`never`, `on-request`, `on-failure`, `always`)
  - For reviews, use `-a never` or omit (defaults to appropriate mode based on sandbox)
- `--json`: Output structured JSON events for parsing (useful with `jq` or `--output-schema`)
- `--output-schema <path>`: Output structured summary as JSON matching provided schema
- `--full-auto`: Convenience flag that expands to `--sandbox workspace-write -a on-failure` - **Do not use for review-only workflows**
- `-o output.txt`: Write final message to file
- `--profile <name>`: Use saved configuration profile (useful for team defaults)
- `--cd <path>`: Change to specified directory before running (useful for scoping repos)
- `--skip-git-repo-check`: Skip git repository validation (useful for spec-only reviews)

**Input methods:**

For longer or reusable prompts, you can:
- **Use stdin**: `codex exec --model gpt-5-codex --sandbox read-only - < review-prompt.txt`
- **Use profiles**: `codex exec --profile review "Review src/auth.py"` (profiles can set model, sandbox, and other defaults)

### Step 3: Analyze Codex Feedback

Parse Codex's response for:
- Security vulnerabilities and bugs
- Code quality issues
- Performance concerns
- Architectural problems
- Best practice violations
- Alternative approaches

### Step 4: Present Synthesis

Provide the user with:
1. **Summary**: High-level findings from Codex
2. **Key Issues**: Critical problems identified
3. **Recommendations**: Actionable suggestions from both reviewers
4. **Divergent Opinions**: Where Claude and Codex differ (valuable insights)
5. **Consensus**: Where both agree (high confidence findings)

## Common Review Patterns

### Pattern 1: Code File Review

Review a specific implementation:

```bash
codex exec --model gpt-5-codex --sandbox read-only "Review the file src/auth/login.py for security issues, bugs, and code quality. Look for authentication vulnerabilities, injection risks, and edge cases."
```

### Pattern 2: Pull Request Review

Compare branches and review changes:

```bash
codex exec --model gpt-5-codex --sandbox read-only "First run 'git diff main...HEAD' to see all changes in the current branch. Then review those changes focusing on: 1) Breaking changes, 2) Performance implications, 3) Test coverage, 4) Security concerns. Provide detailed feedback on each modified file with specific line references."
```

For multi-repo or complex PR reviews:

```bash
codex exec --model gpt-5-codex --sandbox read-only --cd /path/to/repo "Run 'git status' and 'git diff main...HEAD' to understand the PR scope. Review all modified files for correctness, security issues, and adherence to project patterns."
```

### Pattern 3: Architecture Review

Validate design decisions:

```bash
codex exec --model gpt-5-codex --sandbox read-only "Review the architecture described in docs/ARCHITECTURE.md and the implementation in src/. Are there any inconsistencies? Does the implementation follow the intended design? Suggest improvements."
```

### Pattern 4: Specification Validation

Check if code matches spec:

```bash
codex exec --model gpt-5-codex --sandbox read-only "Compare the specification in SPEC.md with the implementation in src/api/. Does the code correctly implement all specified requirements? Are there any deviations or missing features?"
```

Review spec quality with code context:

```bash
codex exec --model gpt-5-codex --sandbox read-only --skip-git-repo-check "Review the specification in docs/FEATURE_SPEC.md. Then examine the existing codebase in src/ to understand current patterns, architecture, and constraints. Evaluate if the spec is: 1) Complete and clear, 2) Consistent with existing code patterns, 3) Technically feasible, 4) Missing any edge cases or requirements."
```

### Pattern 5: Focused Code Review

Target specific concerns:

```bash
codex exec --model gpt-5-codex --sandbox read-only "Review src/database/ focusing only on: 1) SQL injection vulnerabilities, 2) Connection pooling issues, 3) Transaction handling bugs. Ignore style issues."
```

### Pattern 6: Comparative Analysis

Get architectural alternatives:

```bash
codex exec --model gpt-5-codex --sandbox read-only "Review the current microservices architecture in the codebase. Suggest alternative approaches that might be more suitable. Consider: scalability, maintainability, and deployment complexity."
```

## Best Practices

### Prompting Codex Effectively

1. **Be specific**: Define exact scope and concerns
2. **Set context**: Mention language, framework, or domain
3. **List priorities**: What matters most (security, performance, etc.)
4. **Request format**: Ask for structured output if needed
5. **Avoid ambiguity**: Clear, actionable review requests

### Sandbox Safety

- **Always use `--sandbox read-only` for reviews** - This skill is for review feedback only, never implementation
- Never use `--full-auto` or `--sandbox workspace-write` - Codex should only read and analyze, not modify files
- The review workflow is: Codex provides feedback → Claude or user implements changes

### Handling Disagreements

When Claude and Codex disagree:
1. Present both perspectives clearly to the user
2. Explain the reasoning behind each view
3. Let the user make the final decision
4. Note that disagreements often highlight edge cases or trade-offs

### Review Scope Management

- For large codebases, review in focused segments
- Use multiple Codex calls for different aspects (security, performance, etc.)
- Combine Codex's specialized insights with Claude's contextual knowledge

## Advanced Usage

### JSON Output Mode

For programmatic processing:

```bash
codex exec --model gpt-5-codex --sandbox read-only --json "Review auth.py for vulnerabilities" > review.jsonl
```

Parse JSONL output for structured data:
- `turn.started`: Review begins
- `item.completed`: Contains reasoning and findings
- `agent_message`: Final review summary
- `turn.completed`: Includes token usage

**Post-process JSON output with jq:**

```bash
# Extract only the final agent message
codex exec --model gpt-5-codex --sandbox read-only --json "Review src/api/" | jq 'select(.type=="agent_message")'

# Get just the review text
codex exec --model gpt-5-codex --sandbox read-only --json "Review src/api/" | jq -r 'select(.type=="agent_message") | .message.content[0].text'
```

**Use output schema for structured summaries:**

```bash
# Create a schema file (review-schema.json)
cat > review-schema.json << 'EOF'
{
  "security_issues": ["string"],
  "performance_concerns": ["string"],
  "bugs": ["string"],
  "recommendations": ["string"]
}
EOF

# Get structured output matching the schema
codex exec --model gpt-5-codex --sandbox read-only --output-schema review-schema.json "Review auth.py and output findings in the specified format"
```

### Resuming Reviews

Continue a previous review session (model and sandbox are already set in the session):

```bash
# Resume the most recent session
codex exec resume --last "Now focus on the error handling in the code you just reviewed"

# Resume a specific session by ID
codex exec resume 019a1b6a-1b29-7153-8f3e-40678da51ec8 "Please elaborate on the security issues you mentioned"
```

**Session management for multi-turn reviews:**

When performing complex reviews that require back-and-forth discussion, capture the session ID from the initial review output. The session ID appears in the header output:

```
session id: 019a1b6a-1b29-7153-8f3e-40678da51ec8
```

Save this ID to resume the same conversation later. This is especially important for:
- Reviews with follow-up questions
- Multi-stage reviews (first security, then performance, etc.)
- Collaborative reviews where different team members need to continue the discussion

**When to use resume vs. new session:**
- **Use resume**: Multi-turn discussions, follow-up questions, iterative reviews
- **Use new session**: One-off checks, independent reviews of different code sections

### Custom Model Configuration

Use different models or reasoning levels:

```bash
codex exec --model gpt-5-codex --sandbox read-only --config reasoning_effort=high "Perform deep analysis of the cryptographic implementation"
```

## Integration with Claude's Review

Claude should:
1. **First** perform its own analysis of the code/architecture
2. **Then** invoke Codex for independent review
3. **Compare** findings and identify:
   - Agreements (high confidence issues)
   - Disagreements (need user judgment)
   - Unique insights from each reviewer
4. **Synthesize** into a comprehensive review for the user

## Limitations

- Codex reviews are non-interactive; plan questions in advance
- Codex operates independently and doesn't see Claude's conversation history
- Large files may need to be reviewed in segments
- Codex cannot access external APIs or network resources by default
- Review quality depends on prompt clarity and context provided

## Example Complete Workflow

```bash
# 1. Claude performs initial review
# (Claude analyzes the code internally)

# 2. Invoke Codex for second opinion with read-only sandbox
codex exec --model gpt-5-codex --sandbox read-only "Review src/payment/processor.py for:
1. Race conditions in transaction processing
2. Proper error handling and rollback
3. Security issues with payment data
4. Edge cases that could cause data loss
Provide specific line numbers and severity ratings."
# Note: Capture the session ID from output for potential follow-ups

# 3. If follow-up needed, resume the session
codex exec resume --last "Can you suggest specific fixes for the race conditions you identified?"

# 4. Compare findings
# (Claude compares its findings with Codex's output)

# 5. Present synthesized review to user
# (Claude creates unified report with both perspectives)
```

## Troubleshooting

**"codex: command not found"**
- Codex CLI is not installed. User needs to install it first.

**"Authentication required"**
- Run `codex` interactively first to authenticate
- Or set `CODEX_API_KEY` environment variable

**"Permission denied" errors**
- Check file permissions in the working directory
- Verify sandbox mode is appropriate for the task

**Codex review seems shallow**
- Improve prompt specificity
- Break large reviews into focused segments
- Use `--config reasoning_effort=high` for complex tasks
