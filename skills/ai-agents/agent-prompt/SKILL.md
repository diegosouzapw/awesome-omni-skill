---
name: agent-prompt
description: Write effective system prompts for TD AI agents. Covers role definition, constraint specification, output formatting, tool usage instructions, and prompt structure patterns.
---

# Writing Effective Agent Prompts

Guidelines for crafting system prompts that produce reliable, consistent agent behavior.

## Prompt Structure

```markdown
# Role Definition
[Who the agent is and what it does]

# Constraints
[What the agent MUST and MUST NOT do]

# Process/Workflow
[Step-by-step instructions]

# Output Format
[Expected response structure]

# Context
[Environment info, available tools]
```

## Key Principles

### 1. Define Role Clearly

```markdown
You are a customer support specialist for [Company].
Your role is to help customers resolve billing issues
and answer questions about their accounts.
```

Not: "You are a helpful assistant."

### 2. Use Explicit Constraints

**DO this:**
```markdown
=== CRITICAL CONSTRAINTS ===
You are STRICTLY PROHIBITED from:
- Modifying customer payment methods without verification
- Sharing account details with third parties
- Making promises about refunds over $100

You MUST:
- Verify customer identity before account changes
- Log all interactions in the CRM
```

**NOT this:**
```markdown
Be careful with customer data.
```

### 3. Provide Step-by-Step Process

```markdown
## Your Process

1. **Verify Identity**: Ask for account email and last 4 digits of phone
2. **Understand Issue**: Let customer explain before offering solutions
3. **Check History**: Review past interactions in CRM
4. **Propose Solution**: Offer specific resolution with timeline
5. **Confirm**: Summarize actions taken and next steps
```

### 4. Specify Output Format

```markdown
## Required Output

End your response with:

### Summary
- Issue: [Brief description]
- Resolution: [Action taken]
- Follow-up: [Next steps if any]

Format all dates as YYYY-MM-DD.
Use bullet points for lists of 3+ items.
```

### 5. Handle Edge Cases

```markdown
## Special Cases

If customer is angry:
- Acknowledge frustration first
- Do not argue or justify
- Escalate if they mention legal action

If request is outside scope:
- Politely explain limitations
- Provide alternative resources
- Offer to transfer to appropriate team
```

### 6. Tool Usage Instructions

```markdown
## Available Tools

You have access to these tools:
- `search_kb`: Search knowledge base for product info
- `lookup_customer`: Get customer account details
- `create_ticket`: Create support ticket

ALWAYS use `lookup_customer` before discussing account details.
NEVER make up information - use `search_kb` when unsure.
```

## Common Patterns

### Read-Only Agent

```markdown
=== READ-ONLY MODE ===
You can ONLY read and analyze. You CANNOT:
- Create, modify, or delete any records
- Execute transactions
- Change system state

Your role is EXCLUSIVELY to provide information and recommendations.
```

### Multi-Step Workflow Agent

```markdown
## Workflow

1. **Gather**: Collect all required information
2. **Validate**: Check inputs against rules
3. **Process**: Execute the main task
4. **Verify**: Confirm results are correct
5. **Report**: Summarize what was done

Do NOT skip steps. If any step fails, stop and report the issue.
```

### Expert Agent

```markdown
You are a [domain] expert with deep knowledge of:
- [Specific area 1]
- [Specific area 2]
- [Specific area 3]

When answering:
- Cite specific sources or documentation
- Provide concrete examples
- Acknowledge uncertainty when present
```

## Anti-Patterns to Avoid

| Bad | Good |
|-----|------|
| "Be helpful" | "Answer questions about X using the knowledge base" |
| "Don't be rude" | "Use professional, neutral tone" |
| "Handle errors appropriately" | "If error occurs, log it and notify user with error code" |
| "Use good judgment" | "If amount > $100, require manager approval" |
| Long paragraphs | Bulleted lists and numbered steps |

## Variables in Prompts

Use `{{variable}}` for dynamic content:

```markdown
You are assisting {{customer_name}} (ID: {{customer_id}}).
Their current plan is {{plan_type}}.

Focus on issues related to their plan tier.
```

## Testing Prompts

Before deploying, test with:
1. **Happy path**: Normal expected inputs
2. **Edge cases**: Empty inputs, special characters
3. **Adversarial**: Attempts to break constraints
4. **Out of scope**: Requests outside agent's domain

## prompt.md Example

```markdown
You are a SQL query assistant for Treasure Data.

## Your Role
Help users write and optimize Trino SQL queries for TD.

## Constraints
- NEVER execute queries that modify data (INSERT, UPDATE, DELETE)
- ALWAYS include time filters using TD_INTERVAL or TD_TIME_RANGE
- Do NOT guess table schemas - use `describe_table` tool first

## Process
1. Understand the user's data question
2. Check available tables with `list_tables`
3. Get schema with `describe_table`
4. Write query with appropriate time filters
5. Explain the query logic

## Output Format
Provide queries in code blocks with explanation:

```sql
SELECT ...
```

**Explanation**: [Why this query answers the question]
```

## Related Skills

- **agent** - Agent configuration and deployment
- **tdx-basic** - CLI operations
