---
name: awareness-analyzer
description: Diagnose audience awareness level and market sophistication using Eugene Schwartz's Breakthrough Advertising framework
allowed-tools:
  - Read
  - WebFetch
  - Task
invocation: user
---

# Awareness Analyzer

Use this skill to diagnose where a target audience sits on the problem awareness spectrum and market sophistication scale. This determines what type of copy will resonate.

## When to Use

- Before writing any marketing copy
- When analyzing why existing copy isn't converting
- When entering a new market or targeting a new audience
- When competitors are winning with different messaging

## Input Required

1. **Product/Service Description**: What you're selling and what problem it solves
2. **Target Audience**: Who you're trying to reach (developers, founders, enterprises, etc.)
3. **Existing Copy** (optional): Current marketing materials to analyze

## Analysis Process

### Step 1: Determine Problem Awareness Level

Ask these diagnostic questions:

| Level | Diagnostic Question | If YES |
|-------|---------------------|--------|
| Unaware | Does the audience not realize they have a problem? | Lead with symptoms |
| Problem Aware | Do they feel the pain but not know solutions exist? | Sell the solution outcome |
| Solution Aware | Do they know solutions exist but not your product? | Sell your unique mechanism |
| Product Aware | Do they know your product but haven't bought? | Sell proof and offers |
| Most Aware | Are they ready to buy, just need the right trigger? | Sell price/urgency |

### Step 2: Determine Market Sophistication

| Stage | Market Signal | Strategy |
|-------|---------------|----------|
| 1 | Few competitors, novel category | Simple direct claim |
| 2 | Growing competition | Expand claim (faster, better, more) |
| 3 | Crowded market | Unique mechanism required |
| 4 | Claims exhausted | Escalate with proof |
| 5 | Market fatigue | Identity over features |

### Step 3: Generate Recommendations

Based on the diagnosis, provide:

1. **Awareness Level**: Which of the 5 levels the audience is at
2. **Sophistication Stage**: Which of the 5 stages the market is at
3. **Messaging Strategy**: Recommended approach for copy
4. **Headlines for Each Level**: Example headlines that would work at each awareness level
5. **Red Flags**: What NOT to do based on the diagnosis

## Output Format

```markdown
## Awareness Diagnosis

### Audience: [Target Audience]
### Product: [Product Name]

### Problem Awareness Level: [Level]
**Evidence**: [Why you determined this level]
**Implication**: [What this means for copy]

### Market Sophistication Stage: [Stage]
**Evidence**: [Market signals observed]
**Implication**: [What this means for differentiation]

### Recommended Messaging Strategy
[Specific recommendations based on diagnosis]

### Sample Headlines by Awareness Level

**For Unaware Audience:**
- [Symptom-focused headline]

**For Problem Aware Audience:**
- [Solution-focused headline]

**For Solution Aware Audience:**
- [Mechanism-focused headline]

**For Product Aware Audience:**
- [Proof-focused headline]

**For Most Aware Audience:**
- [Offer-focused headline]

### Red Flags to Avoid
- [What not to do based on this diagnosis]
```

## Common Mistakes

1. **Pitching mechanisms to unaware audiences**: Don't mention "AI agents" or "RAG" to people who don't know they have a problem
2. **Being generic in sophisticated markets**: "Chat with AI" doesn't work anymore in Stage 4+ markets
3. **Selling features to identity-driven buyers**: Stage 5 markets need tribal belonging, not feature lists
4. **Assuming your audience is Most Aware**: Most audiences are Problem Aware or Solution Aware

## Example Analysis

**Product**: AI coding assistant
**Audience**: Enterprise developers

**Diagnosis**:
- Problem Awareness: **Solution Aware** - They know AI coding tools exist, have tried GitHub Copilot
- Market Sophistication: **Stage 4** - Many competitors, claims exhausted

**Strategy**: Can't just say "AI that writes code" - need unique mechanism + proof
- Lead with: "The only AI that understands your entire codebase context"
- Support with: Enterprise logos, security certifications, integration proof
