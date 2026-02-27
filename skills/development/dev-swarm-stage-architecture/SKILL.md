---
name: dev-swarm-stage-architecture
description: Design the complete system architecture including components, data flow, infrastructure, database schema, and API design. Use when starting stage 07 (architecture) or when user asks about system design, tech stack, or database schema.
---

# Stage 07 - Architecture

Design the complete system architecture including components, data flow, infrastructure, and technical decisions that will guide all subsequent development work.

## When to Use This Skill

- User asks to start stage 07 (architecture)
- User wants to design system architecture or select tech stack
- User asks about database design, API architecture, or infrastructure

## Your Roles in This Skill

See `dev-swarm/docs/general-dev-stage-rule.md` for role selection guidance.

## Role Communication

See `dev-swarm/docs/general-dev-stage-rule.md` for the required role announcement format.

## Pre-Stage Check

Before starting, verify previous stages:

1. Check if `00-init-ideas/` through `06-ux/` folders have content (not just `.gitkeep`)
2. If any previous stage is empty and has no `SKIP.md`:
   - Ask user: "Stage {XX} is not complete. Would you like to skip it or start from that stage first?"

## Instructions

### Step 1: Context Review

Read all files to understand the project:

- `ideas.md`
- `00-init-ideas/*.md` through `06-ux/*.md` - All markdown files

### Step 2: Create Stage Proposal

**General Rules:** See `dev-swarm/docs/general-dev-stage-rule.md` → "Create Stage Proposal Rules" section.

If this stage is skipped (has SKIP.md), execute the next non-skipped stage's agent skill. Otherwise, create the file `07-architecture/README.md` with the following content:

#### 2.1 Stage Goal

Brief the goal in 2-3 paragraphs:
- What this stage aims to achieve (define system architecture, tech stack, data models, APIs)
- Why architecture design is critical before development begins
- How this builds upon previous stages (UX flows, PRD requirements, MVP scope, tech research findings)
- What deliverables will be produced

#### 2.2 File Selection

Select files from these options based on project needs:

**System Architecture:**
- `system-architecture.md` - Overall system architecture describing components and interactions
- `architecture-diagram/` - High-level system architecture diagrams
- `C4-component-diagram/` - C4 model diagrams: Context, Containers, Components
**Technology Stack:**
- `tech-stack.md` - Selected languages, frameworks, libraries, and tools
- `tech-stack-rationale.md` - Detailed reasoning for each technology choice

**Database Design:**
- `database-design.md` - Database schema design and data modeling approach
- `database-schema/` - ER diagrams showing tables and relationships

**API Architecture:**
- `api-design.md` - API architecture patterns, authentication, versioning strategy
- `api-endpoints.md` - Complete listing of API endpoints

**Infrastructure:**
- `infrastructure-design.md` - Overview of infrastructure components
- `infrastructure-diagram/` - Infrastructure topology diagrams
**Security & Scalability:**
- `security-architecture.md` - Security design including authentication and authorization
- `scalability-plan.md` - Scalability considerations and strategies

**Data & Integrations:**
- `data-flow-diagram/` - Data flow diagrams
- `integration-architecture.md` - Third-party service integrations

For each selected file, provide:
- Short description
- Why it's essential for this project
- Key information it should include

#### 2.3 Request User Approval

Ask user: "Please check the Stage Proposal in `07-architecture/README.md`. Update it directly or tell me how to update it."

### Step 3: Execute Stage Plan

Once user approves `07-architecture/README.md`:

#### 3.1 Create All Planned Files

Create each file listed in the approved README:

- **For `.md` files:** Write comprehensive content with clear sections and technical details
- **For diagram folders:** Follow `dev-swarm/docs/mermaid-diagram-guide.md` to create related diagrams files

**Quality Guidelines:**
- Base architecture decisions on PRD requirements, UX designs, and tech research findings
- Ensure tech stack choices align with team capabilities and validated assumptions
- Design for MVP scope first, with considerations for future scalability
- Include clear component boundaries and interfaces
- Document all major architectural decisions and trade-offs
- Reference any constraints or recommendations from tech research stage

#### 3.2 Request User Approval for Files

After creating all files:
- Provide a summary of what was created
- Highlight key architectural decisions and trade-offs
- Ask: "Please review the architecture design. You can update or delete files, or let me know how to modify them."

### Step 4: Finalize Stage

Once user approves all files:

#### 4.1 Documentation Finalization
- Sync `07-architecture/README.md` to remove any deleted files
- Ensure all files are complete and well-formatted
- Check that all diagrams render correctly

#### 4.2 Prepare for Next Stage
- Summarize key architectural decisions for reference in tech-specs stage
- Identify any technical risks or open questions

#### 4.3 Announce Completion

Inform user:
- "Stage 07 (Architecture) is complete"
- Summary of deliverables created
- Key architectural decisions made
- "Ready to proceed to Stage 08 (Tech Specs) when you are"

## Stage Completion Rules

See `dev-swarm/docs/general-dev-stage-rule.md` for stage completion, commit, and skip rules.

## Key Principles

- Document architectural decisions and trade-offs
- Design for MVP scope first
- Consider security and scalability from the start
- Support smooth transition to tech specs
