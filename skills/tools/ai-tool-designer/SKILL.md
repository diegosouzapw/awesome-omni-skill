---
name: ai-tool-designer
description: Guide for designing effective tools for AI agents. Use when creating tools for custom agent systems or any AI tool interfaces. Provides principles for tool naming, input/output design, error handling, and evaluation methodologies that maximize agent effectiveness.
license: Complete terms in LICENSE.txt
---

# AI Agent Tool Designer

## Overview

This skill provides comprehensive guidance for designing tools that AI agents can use effectively. Whether building custom agent tools or any AI-accessible interfaces, these principles maximize agent success in accomplishing real-world tasks.

Note: Use the more specific mcp-builder skill if you want to create an MCP server.

The quality of a tool system is measured not by how comprehensively it implements features, but by how well it enables AI agents to accomplish realistic, complex tasks using only the tools provided.

---

## Agent-Centric Design Principles

Before implementing any tool system, understand these foundational principles for designing tools that AI agents can use effectively:

### 1. Build for Workflows, Not Just API Endpoints

**Principle:** Design thoughtful, high-impact workflow tools rather than simply wrapping existing API endpoints.

**Why it matters:** Agents need to accomplish complete tasks, not just make individual API calls. Tools that consolidate related operations reduce the number of steps agents must take and improve success rates.

**How to apply:**
- Consolidate related operations (e.g., `schedule_event` that both checks availability and creates the event)
- Focus on tools that enable complete tasks, not just individual API calls
- Consider what workflows agents actually need to accomplish, not just what the underlying API offers
- Ask: "What is the user trying to accomplish?" rather than "What does the API provide?"

**Examples:**
- ❌ Bad: Separate tools `check_calendar_availability`, `create_calendar_event`, `send_event_notification`
- ✅ Good: Single tool `schedule_event` with parameters for checking conflicts and sending notifications

### 2. Optimize for Limited Context

**Principle:** Agents have constrained context windows - make every token count.

**Why it matters:** When agents run out of context, they fail to complete tasks. Verbose tool outputs force agents to make difficult decisions about what information to keep or discard.

**How to apply:**
- Return high-signal information, not exhaustive data dumps
- Provide "concise" vs "detailed" response format options (default to concise)
- Default to human-readable identifiers over technical codes (names over IDs when possible)
- Consider the agent's context budget as a scarce resource
- Implement character limits and graceful truncation (typically 25,000 characters)
- Use pagination with reasonable defaults (20-50 items)

**Examples:**
- ❌ Bad: Return all 50 fields from user object including metadata, internal IDs, timestamps in multiple formats
- ✅ Good: Return name, email, role, and key status fields; offer `detailed=true` parameter for full data

### 3. Design Actionable Error Messages

**Principle:** Error messages should guide agents toward correct usage patterns, not just report failures.

**Why it matters:** Agents learn tool usage through feedback. Clear, educational errors help agents self-correct and succeed on retry.

**How to apply:**
- Suggest specific next steps in error messages
- Make errors educational, not just diagnostic
- Include examples of correct usage when parameters are invalid
- Guide agents toward solutions: "Try using filter='active_only' to reduce results"
- Avoid technical jargon; use natural language

**Examples:**
- ❌ Bad: "Error 400: Invalid request"
- ✅ Good: "The limit parameter must be between 1-100. You provided 500. Try using limit=50 and pagination with offset to retrieve more results."

### 4. Follow Natural Task Subdivisions

**Principle:** Tool names and organization should reflect how humans think about tasks, not just API structure.

**Why it matters:** Agents use tool names and descriptions to decide which tool to call. Natural naming improves tool discovery and reduces wrong tool selections.

**How to apply:**
- Tool names should reflect human mental models of tasks
- Group related tools with consistent prefixes for discoverability
- Design tools around natural workflows, not just API structure
- Use action-oriented naming: `search_users`, `create_project`, `send_message`
- Include service/system prefix to avoid conflicts: `slack_send_message` not just `send_message`

**Examples:**
- ❌ Bad: `api_endpoint_users_post`, `api_endpoint_users_get`, `api_endpoint_users_delete`
- ✅ Good: `create_user`, `search_users`, `delete_user`

### 5. Use Evaluation-Driven Development

**Principle:** Create realistic evaluation scenarios early and let agent feedback drive tool improvements.

**Why it matters:** Only by testing tools with actual agents can you discover usability issues. Prototype quickly and iterate based on real agent performance.

**How to apply:**
- Create 10+ complex, realistic questions agents should answer using your tools
- Test with actual AI agents attempting to solve these questions
- Observe where agents struggle, make mistakes, or run out of context
- Iterate on tool design based on agent feedback
- Measure success by agent task completion rate, not feature completeness

**Process:**
1. Build initial tools based on these principles
2. Create evaluation questions (see [Evaluation Guide](./references/evaluation_guide.md))
3. Test with agents
4. Identify failure patterns
5. Refine tools
6. Repeat

---

## Tool Design Framework

Follow this systematic framework when designing any tool for AI agents:

### Phase 1: Planning

**1. Identify Core Workflows**
- List the most valuable operations agents need to perform
- Prioritize tools that enable the most common and important use cases
- Consider which tools work together to enable complex workflows

**2. Design Input Schemas**
- Use strong validation (dry-validation for Ruby, JSON Schema)
- Include proper constraints (min/max length, regex patterns, ranges)
- Provide clear, descriptive field descriptions with examples
- Set sensible defaults to reduce required parameters

**3. Design Output Formats**
- Support multiple formats (JSON for programmatic, Markdown for human-readable)
- Define consistent response structures across similar tools
- Plan for large-scale usage (thousands of users/resources)
- Implement character limits and truncation strategies
- Include pagination metadata (`has_more`, `next_offset`, `total_count`)

**4. Plan Error Handling**
- Design clear, actionable, agent-friendly error messages
- Handle authentication and authorization errors gracefully
- Consider rate limiting and timeout scenarios
- Provide guidance on how to proceed after errors

### Phase 2: Implementation

**Tool Naming Conventions:**
- Use snake_case: `search_users`, `create_project`
- Include service prefix: `github_create_issue`, `slack_send_message`
- Be action-oriented: start with verbs (get, list, search, create, update, delete)
- Be specific: avoid generic names that could conflict

**Tool Descriptions:**
Write comprehensive descriptions that include:
- One-line summary of what the tool does
- Detailed explanation of purpose and functionality
- When to use this tool (and when NOT to use it)
- Parameter descriptions with examples
- Return value schema
- Error handling guidance

**Tool Annotations** (if supported by your system):
- `readOnlyHint: true` for read-only operations
- `destructiveHint: false` for non-destructive operations
- `idempotentHint: true` if repeated calls have same effect
- `openWorldHint: true` if interacting with external systems

### Phase 3: Refinement

**Code Quality Checklist:**
- ✅ No duplicated code between tools (DRY principle)
- ✅ Shared logic extracted into reusable functions
- ✅ Similar operations return similar formats (consistency)
- ✅ All external calls have error handling
- ✅ Full type coverage (type hints, TypeScript types)
- ✅ Every tool has comprehensive documentation

**Testing:**
- Test with valid and invalid inputs
- Test error handling paths
- Test with real AI agents using evaluation questions
- Test pagination and large result sets
- Test character limits and truncation

---

## Response Format Guidelines

All tools that return data should support multiple formats for flexibility:

### JSON Format (`response_format="json"`)
**Purpose:** Machine-readable structured data for programmatic processing

**Best practices:**
- Include all available fields and metadata
- Use consistent field names and types
- Suitable for when agents need to process data further
- Return IDs alongside names for precision

**Example:**
```json
{
  "users": [
    {
      "id": "U123456",
      "name": "John Doe",
      "email": "john@example.com",
      "role": "developer",
      "active": true
    }
  ],
  "total": 150,
  "count": 20,
  "has_more": true,
  "next_offset": 20
}
```

### Markdown Format (`response_format="markdown"`, typically default)
**Purpose:** Human-readable formatted text for user presentation

**Best practices:**
- Use headers, lists, and formatting for clarity
- Convert timestamps to readable format ("2024-01-15 10:30 UTC" vs epoch)
- Show display names with IDs in parentheses ("@john.doe (U123456)")
- Omit verbose metadata (show one profile image URL, not all sizes)
- Group related information logically
- Use when presenting information to end users

**Example:**
```markdown
## Users (20 of 150)

- **John Doe** (@john.doe)
  - Email: john@example.com
  - Role: Developer
  - Status: Active

- **Jane Smith** (@jane.smith)
  - Email: jane@example.com
  - Role: Designer
  - Status: Active

*Showing 20 results. Use offset=20 to see more.*
```

---

## Pagination Best Practices

For tools that list resources:

**Implementation requirements:**
- Always respect the `limit` parameter (never load all results when limit specified)
- Implement offset-based or cursor-based pagination
- Return pagination metadata: `has_more`, `next_offset`/`next_cursor`, `total_count`
- Never load all results into memory for large datasets
- Default to reasonable limits (20-50 items typical)

**Response structure:**
```json
{
  "items": [...],
  "total": 150,
  "count": 20,
  "offset": 0,
  "has_more": true,
  "next_offset": 20
}
```

**Clear guidance in responses:**
Include instructions for getting more data:
- "Showing 20 of 150 results. Use offset=20 to see the next page."
- "Results truncated. Add filters to narrow the search."

---

## Character Limits and Truncation

To prevent overwhelming context windows:

**Implementation:**
- Define CHARACTER_LIMIT constant (typically 25,000 characters)
- Check response size before returning
- Truncate gracefully with clear indicators
- Provide guidance on how to filter/paginate for complete results

**Example handling:**
```ruby
CHARACTER_LIMIT = 25_000

if result.length > CHARACTER_LIMIT
  truncated_data = data[0...[1, data.length / 2].max]
  response[:truncated] = true
  response[:truncation_message] =
    "Response truncated from #{data.length} to #{truncated_data.length} items. " \
    "Use 'offset' parameter or add filters like status='active' to see more."
end
```

---

## Input Validation Best Practices

**Security and usability:**
- Validate all parameters against schema before processing
- Sanitize file paths to prevent directory traversal
- Validate URLs and external identifiers
- Check parameter sizes and ranges
- Prevent command injection in system calls
- Return clear validation errors with examples of correct format

**Schema design:**
- Use strong validation (dry-validation, JSON Schema)
- Include constraints (minLength, maxLength, pattern, minimum, maximum)
- Provide detailed field descriptions with examples
- Mark required vs optional parameters clearly
- Set sensible defaults where possible

---

## Resources

This skill includes reference documentation for deeper exploration:

### references/tool_design_patterns.md
Comprehensive patterns and anti-patterns for common tool design scenarios with detailed examples.

### references/evaluation_guide.md
Complete methodology for creating evaluation questions that test tool effectiveness with AI agents, including how to run evaluations and interpret results.

---

## Further Reading

For detailed examples and advanced patterns:
- [Tool Design Patterns](./references/tool_design_patterns.md) - Comprehensive patterns and examples
- [Evaluation Guide](./references/evaluation_guide.md) - Testing methodology and evaluation creation
