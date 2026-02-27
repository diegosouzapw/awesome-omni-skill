---
name: kimmo-agent-friendly-score
description: Score developer tools and SaaS products for AI agent compatibility. Use when evaluating how well a devtool works with AI coding assistants, or when optimizing a product for the agent era.
---

# Agent-Friendly Score - DevTool Evaluation

Evaluate developer tools and SaaS products for compatibility with AI coding assistants (Cursor, Claude, GitHub Copilot).

## When to Use

- User asks "How agent-friendly is [tool]?"
- User wants to evaluate a devtool for AI compatibility
- User is building a devtool and wants to optimize for AI assistants
- User is comparing tools and AI compatibility matters

## The Agent Era Context

85% of developers now use AI tools regularly. When a developer asks Cursor to "add email functionality," the AI picks the service, writes the integration, and runs the install command.

**The New Funnel**:

- Traditional: Marketing → Landing → Docs → Trial → Conversion
- Agent: Problem → AI suggestion → `npm install` → Subscription

Tools that AI can easily work with get recommended. Tools it can't work with become invisible.

## Scoring Framework

### Category 1: SDK & API Design (30 points)

| Criterion                         | Points | How to Check                        |
| --------------------------------- | ------ | ----------------------------------- |
| SDK available for major languages | 10     | Check docs for JS, Python, Go, etc. |
| Consistent, predictable API       | 10     | Review API reference for patterns   |
| Complete TypeScript definitions   | 5      | Check npm package for .d.ts files   |
| Clear error messages              | 5      | Test error responses                |

**Scoring guide**:

- 25-30: Excellent - AI can generate correct code first try
- 15-24: Good - AI mostly succeeds, occasional fixes needed
- 0-14: Poor - AI struggles to generate working code

### Category 2: Documentation Quality (25 points)

| Criterion                      | Points | How to Check                   |
| ------------------------------ | ------ | ------------------------------ |
| Docs lead with working code    | 10     | First thing on quickstart page |
| Copy-paste examples work       | 5      | Try the first 3 examples       |
| Parseable structure (H1→H2→H3) | 5      | View page source/outline       |
| No login walls on docs         | 5      | Access docs without account    |

**Scoring guide**:

- 20-25: AI can extract and apply correctly
- 10-19: AI needs some interpretation
- 0-9: AI will likely hallucinate or fail

### Category 3: Training Data Presence (20 points)

| Criterion                    | Points | How to Check                   |
| ---------------------------- | ------ | ------------------------------ |
| GitHub repos using this tool | 8      | Search GitHub for imports      |
| Stack Overflow presence      | 6      | Search SO for [tool] questions |
| Tutorial/blog coverage       | 6      | Search "[tool] tutorial"       |

**Scoring guide**:

- 15-20: Strong training data signal
- 8-14: Moderate presence
- 0-7: AI may not know this tool well

### Category 4: MCP Integration (15 points)

| Criterion                  | Points | How to Check                    |
| -------------------------- | ------ | ------------------------------- |
| Official MCP server exists | 10     | Check mcp.so, official docs     |
| MCP server is maintained   | 3      | Recent commits, version updates |
| MCP server is discoverable | 2      | Listed on MCP.so or npm         |

**Scoring guide**:

- 12-15: Full agent workflow integration
- 5-11: Partial integration
- 0-4: Not in agent workflow

### Category 5: Time to Working (10 points)

| Criterion                       | Points | How to Check                       |
| ------------------------------- | ------ | ---------------------------------- |
| Install to "hello world" <5 min | 5      | Time yourself following quickstart |
| No complex onboarding           | 3      | Can start without account?         |
| Sensible defaults               | 2      | Works without config?              |

**Scoring guide**:

- 8-10: Instant productivity
- 4-7: Reasonable setup
- 0-3: Significant friction

## Evaluation Workflow

### Step 1: Identify the Tool

Get from user:

- Tool name and URL
- Category (email, auth, database, etc.)
- Main competitor to compare against

### Step 2: SDK Evaluation

Check official SDK:

```bash
# Check npm for TypeScript types
npm info [package] types

# Check for SDK in multiple languages
# Visit: github.com/[org] and look for SDK repos
```

Test API consistency:

- Are endpoints predictable? (e.g., `/users`, `/users/:id`)
- Are responses consistent?
- Are errors structured?

### Step 3: Documentation Audit

Visit docs and check:

- [ ] First code example is within scroll view
- [ ] Examples include all necessary imports
- [ ] Examples actually work when copied
- [ ] Structure uses semantic headings
- [ ] No authentication required to view

### Step 4: Training Data Check

Search GitHub:

```
"import { X } from '[package]'" language:JavaScript
"from [package] import" language:Python
```

Search Stack Overflow:

```
[tool] is:question
```

### Step 5: MCP Check

Search for MCP server:

- https://mcp.so - search for tool name
- Official docs - search for "MCP" or "Model Context Protocol"
- GitHub - search "[tool] mcp server"

### Step 6: Time Test

Follow quickstart:

1. Start timer
2. Follow official quickstart exactly
3. Stop when first API call succeeds
4. Record time and friction points

## Output Template

```markdown
# Agent-Friendly Score: [Tool Name]

## Overall Score: [X]/100

| Category         | Score | Max |
| ---------------- | ----- | --- |
| SDK & API Design | X     | 30  |
| Documentation    | X     | 25  |
| Training Data    | X     | 20  |
| MCP Integration  | X     | 15  |
| Time to Working  | X     | 10  |

## Grade: [A/B/C/D/F]

- A (85-100): AI will recommend and integrate correctly
- B (70-84): AI will usually succeed
- C (55-69): AI needs help, may hallucinate
- D (40-54): Significant AI compatibility issues
- F (<40): AI will struggle or avoid

## Breakdown

### SDK & API Design ([X]/30)

**Strengths:**

- [What works well]

**Gaps:**

- [What's missing]

### Documentation ([X]/25)

**Strengths:**

- [What works well]

**Gaps:**

- [What's missing]

### Training Data Presence ([X]/20)

- GitHub repos found: [X]
- Stack Overflow questions: [X]
- Tutorial coverage: [High/Medium/Low]

### MCP Integration ([X]/15)

- MCP server: [Official/Community/None]
- Status: [Active/Stale/N/A]

### Time to Working ([X]/10)

- Quickstart time: [X minutes]
- Friction points: [list]

## Recommendations

### Quick Wins (High Impact, Low Effort)

1. [Recommendation]
2. [Recommendation]

### Strategic Improvements

1. [Recommendation]
2. [Recommendation]

## Competitor Comparison

| Metric          | [Tool] | [Competitor] |
| --------------- | ------ | ------------ |
| Agent Score     | X/100  | Y/100        |
| MCP Server      | Yes/No | Yes/No       |
| Time to Working | X min  | Y min        |

## Verdict

[One paragraph summary of whether this tool is positioned for the agent era]
```

## Benchmarks by Category

### Email APIs

| Tool     | Typical Score | Notes                       |
| -------- | ------------- | --------------------------- |
| Resend   | 85-90         | MCP, clean SDK, great docs  |
| Postmark | 80-85         | MCP, enterprise-ready       |
| SendGrid | 60-70         | No official MCP, legacy API |

### Authentication

| Tool          | Typical Score | Notes                 |
| ------------- | ------------- | --------------------- |
| Clerk         | 85-90         | MCP, great DX         |
| Auth0         | 75-80         | MCP, but complex      |
| Firebase Auth | 80-85         | MCP, Google ecosystem |

### Databases

| Tool        | Typical Score | Notes                 |
| ----------- | ------------- | --------------------- |
| Supabase    | 85-90         | MCP, hosted, great DX |
| Neon        | 85-90         | MCP, serverless       |
| PlanetScale | 75-80         | MCP (read-only)       |

Use these as calibration when scoring.

## Key Insight

The best technical product doesn't always win anymore. The most **AI-accessible** product wins. When an AI assistant can:

1. Understand your docs
2. Generate working code
3. Integrate via MCP

...you're in the conversation. When it can't, you're invisible to the fastest-growing developer segment.

---

By Kimmo Ihanus | [kimmoihanus.com](https://kimmoihanus.com)
