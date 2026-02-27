---
name: dev-swarm-stage-market-research
description: Conduct comprehensive market research and competitive analysis to validate the problem, understand the market landscape, and identify opportunities. Use when starting stage 01 (market-research) or when user asks about competitors or market analysis.
---

# Stage 01 - Market Research

Conduct comprehensive market research and competitive analysis to validate the problem, understand the market landscape, identify competitors, and discover market gaps and opportunities.

## When to Use This Skill

- User asks to start stage 01 (market-research)
- User wants to research competitors or market landscape
- User asks about market size, trends, or opportunities

## Your Roles in This Skill

See `dev-swarm/docs/general-dev-stage-rule.md` for role selection guidance.

## Role Communication

See `dev-swarm/docs/general-dev-stage-rule.md` for the required role announcement format.

## Pre-Stage Check

Before starting, verify previous stages:

1. Check if `00-init-ideas/` folder has content (not just `.gitkeep`)
2. If previous stage is empty and has no `SKIP.md`:
   - Ask user: "Stage 00 is not complete. Would you like to skip it or start from that stage first?"

## Instructions

### Step 1: Context Review

Read all files to understand the project:

- `ideas.md`
- `00-init-ideas/*.md` - All markdown files

### Step 2: Create Stage Proposal

**General Rules:** See `dev-swarm/docs/general-dev-stage-rule.md` → "Create Stage Proposal Rules" section.

If this stage is skipped (has SKIP.md), execute the next non-skipped stage's agent skill. Otherwise, create the file `01-market-research/README.md` with the following content:

#### 2.1 Stage Goal

Brief the goal in 2-3 paragraphs:
- What this stage aims to achieve
- Why market research is critical for product success
- How this builds upon previous stages
- What deliverables will be produced

#### 2.2 File Selection

Select files from these options based on project needs:

**Market Analysis:**
- `competitive-analysis.md` - In-depth analysis of direct and indirect competitors
- `similar-products.md` - Research on similar products in the market
- `market-size-opportunity.md` - TAM, SAM, SOM analysis

**Strategic Analysis:**
- `swot-analysis.md` - Strengths, weaknesses, opportunities, and threats analysis
- `market-trends.md` - Current industry trends and market direction
- `market-gaps.md` - Identified gaps in the market

**Business Intelligence:**
- `pricing-strategy.md` - Competitor pricing research and recommended pricing models
- `feature-comparison.md` - Feature comparison matrix

For each selected file, provide:
- Short description
- Why it's essential for this project
- Key information it should include

#### 2.3 Request User Approval

Ask user: "Please check the Stage Proposal in `01-market-research/README.md`. Update it directly or tell me how to update it."

### Step 3: Execute Stage Plan

Once user approves `01-market-research/README.md`:

#### 3.1 Create All Planned Files

Create each file listed in the approved README:

- **For `.md` files:** Write comprehensive content based on actual web research

**Quality Guidelines:**
- Use web search to gather real market data and competitor information
- Cite sources where possible for credibility
- Include quantitative data when available
- Analyze at least 3-5 direct competitors
- Identify specific, actionable market opportunities

#### 3.2 Request User Approval for Files

After creating all files:
- Provide a summary of what was created
- Highlight key market insights discovered
- Ask: "Please review the market research documents. You can update or delete files, or let me know how to modify them."

### Step 4: Finalize Stage

Once user approves all files:

#### 4.1 Documentation Finalization
- Sync `01-market-research/README.md` to remove any deleted files
- Ensure all files are complete and well-formatted
- Verify research findings are accurate and well-sourced

#### 4.2 Prepare for Next Stage
- Summarize key market insights for reference in later stages
- Identify top competitors to consider during persona development
- Note market gaps that represent strongest opportunities

#### 4.3 Announce Completion

Inform user:
- "Stage 01 (Market Research) is complete"
- Summary of deliverables created
- Key insights discovered
- "Ready to proceed to Stage 02 (Personas) when you are"

## Stage Completion Rules

See `dev-swarm/docs/general-dev-stage-rule.md` for stage completion, commit, and skip rules.

## Key Principles

- Base research on real data, not assumptions
- Cite sources for credibility
- Connect findings to product concept
- Support smooth transition to persona development
