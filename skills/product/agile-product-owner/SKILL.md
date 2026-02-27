---

# === CORE IDENTITY ===
name: agile-product-owner
title: Agile Product Owner Skill Package
description: Agile product ownership toolkit for Senior Product Owner including INVEST-compliant user story generation, sprint planning, backlog management, velocity tracking, Example Mapping, Three Amigos, and Definition of Ready. Use for story writing, sprint planning, stakeholder communication, requirements discovery, and agile ceremonies.
domain: product
subdomain: product-management

# === WEBSITE DISPLAY ===
difficulty: intermediate
time-saved: "TODO: Quantify time savings"
frequency: "TODO: Estimate usage frequency"
use-cases:
  - Defining product roadmaps and feature prioritization
  - Writing user stories and acceptance criteria
  - Running Example Mapping and Three Amigos sessions
  - Enforcing Definition of Ready as a sprint entry gate
  - Stakeholder communication and alignment

# === RELATIONSHIPS ===
related-agents: []
related-skills:
  - product-team/workshop-facilitation
  - product-team/continuous-discovery
related-commands: []
orchestrated-by: []

# === TECHNICAL ===
dependencies:
  scripts: []
  references:
    - references/frameworks.md
    - references/templates.md
    - references/tools.md
    - references/example-mapping.md
    - references/three-amigos.md
    - references/definition-of-ready.md
    - references/story-splitting-patterns.md
    - references/epic-breakdown-patterns.md
    - references/epic-hypothesis-format.md
    - references/story-mapping-workshop.md
  assets: []
compatibility:
  python-version: 3.8+
  platforms: [macos, linux, windows]
tech-stack: [Python 3.8+, CLI, CSV processing, JSON export]

# === EXAMPLES ===
examples:
  -
    title: Example Usage
    input: "TODO: Add example input for agile-product-owner"
    output: "TODO: Add expected output"

# === ANALYTICS ===
stats:
  downloads: 0
  stars: 0
  rating: 0.0
  reviews: 0

# === VERSIONING ===
version: v1.1.0
author: Claude Skills Team
contributors: []
created: 2025-10-19
updated: 2026-02-16
license: MIT

# === DISCOVERABILITY ===
tags: [agile, owner, product, example-mapping, three-amigos, definition-of-ready, bdd, requirements-discovery]
featured: false
verified: true
---

# Agile Product Owner

## Overview

This skill provides a complete toolkit for agile product ownership -- from backlog refinement to sprint delivery. It combines Python automation tools for story generation with collaborative practices (Example Mapping, Three Amigos, Definition of Ready) that ensure stories are well-understood before development begins.

**Core Value:** Reduce mid-sprint scope discovery and rework by ensuring every story meets a hard Definition of Ready gate before entering the sprint.

**Target Audience:** Product Owners, Business Analysts, Scrum Masters, and cross-functional teams practicing agile delivery.

**Use Cases:**
- Breaking epics into deliverable, INVEST-compliant user stories
- Running Example Mapping sessions to discover requirements through concrete examples
- Facilitating Three Amigos collaboration across business, development, and testing
- Enforcing Definition of Ready as a non-negotiable sprint entry gate
- Sprint planning and capacity allocation


## Core Capabilities

- **Story Generation** - INVEST-compliant user story generator with acceptance criteria, estimation, and sprint planning
- **Requirements Discovery** - Example Mapping technique for collaborative specification through concrete examples
- **Cross-Functional Collaboration** - Three Amigos sessions bridging business, development, and testing perspectives
- **Quality Gates** - Definition of Ready checklist enforced as a hard gate before sprint entry
- **Agile Ceremonies** - Complete frameworks for planning, review, retrospective, and daily standup


## Key Workflows

### Workflow 1: Story Discovery and Readiness

**Time:** 25-minute timebox per story

**Steps:**
1. Select story from backlog (about to enter sprint)
2. Run Three Amigos session with business, dev, and testing representatives
3. Use Example Mapping to explore rules, examples, and questions
4. Resolve all red cards (questions) before development starts
5. Verify story against Definition of Ready checklist

**Expected Output:** Story with complete acceptance criteria, concrete examples, resolved questions, and DoR confirmation

**References:**
- [Example Mapping](references/example-mapping.md) -- card types, session flow, conversational patterns
- [Three Amigos](references/three-amigos.md) -- perspectives, anti-patterns, facilitation
- [Definition of Ready](references/definition-of-ready.md) -- hard gate criteria, enforcement

### Workflow 2: Epic Breakdown and Sprint Planning

**Time:** 2-4 hours for planning; 1 hour weekly for refinement

**Steps:**
1. Define epic with scope and personas
2. Generate stories: `python scripts/user_story_generator.py epic.json`
3. Review INVEST criteria compliance
4. Run Three Amigos on each story
5. Estimate with planning poker
6. Pull stories into sprint until capacity reached

**Expected Output:** Sprint backlog of ready, estimated, well-understood stories


Complete toolkit for agile product ownership - from backlog refinement to sprint delivery. This skill provides Python tools for story generation, comprehensive frameworks for agile ceremonies, and battle-tested templates for user stories and epics.

**What This Skill Provides:**
- INVEST-compliant user story generator with sprint planning
- Complete agile ceremony frameworks (planning, review, retro)
- Velocity tracking and capacity planning methods
- Story estimation and backlog refinement processes
- Example Mapping, Three Amigos, and Definition of Ready practices

**Best For:**
- Breaking epics into deliverable user stories
- Sprint planning and capacity allocation
- Running effective agile ceremonies
- Tracking team velocity and burndown
- Discovering requirements collaboratively before development

## Quick Start

### Generate User Stories
```bash
# From sample epic
python scripts/user_story_generator.py

# From custom epic
python scripts/user_story_generator.py epic.json

# Sprint planning (30 points)
python scripts/user_story_generator.py --sprint --capacity 30
```

### Story Quality Check
Use INVEST criteria to validate stories:
- **I**ndependent: Can complete without dependencies
- **N**egotiable: Flexible on implementation
- **V**aluable: Delivers user/business value
- **E**stimable: Team can size it
- **S**mall: Fits in one sprint
- **T**estable: Clear success criteria

See [frameworks.md](references/frameworks.md) for complete INVEST guidelines.

## Core Workflows

### 1. Epic Breakdown Process

**Steps:**
1. Define epic with scope and personas
2. Generate stories: `python scripts/user_story_generator.py epic.json`
3. Review INVEST criteria compliance
4. Refine acceptance criteria
5. Prioritize and estimate with team

**Epic JSON Format:**
```json
{
  "name": "User Dashboard",
  "description": "Create dashboard for users",
  "personas": ["end_user", "power_user"],
  "scope": [
    "View key metrics and KPIs",
    "Customize dashboard layout",
    "Export dashboard data"
  ],
  "technical_requirements": [
    "Implement caching for performance"
  ]
}
```

**Detailed Process:** See [frameworks.md](references/frameworks.md) for INVEST criteria and backlog refinement.

**Templates:** See [templates.md](references/templates.md) for epic and user story templates.

### 2. Sprint Planning Process

**Part 1: What Will We Build? (2 hours)**
1. Set sprint goal aligned with quarterly OKRs
2. Calculate team capacity (velocity x sprint days)
3. Pull stories from backlog until capacity reached
4. Identify dependencies and risks

**Part 2: How Will We Build It? (2 hours)**
1. Break stories into technical tasks
2. Estimate task hours
3. Assign initial owners
4. Commit to sprint scope

**Sprint Planning Tool:**
```bash
python scripts/user_story_generator.py epic.json --sprint --capacity 30
```

**Detailed Framework:** See [frameworks.md](references/frameworks.md) for complete sprint planning guide.

**Templates:** See [templates.md](references/templates.md) for sprint planning agenda.

### 3. Backlog Refinement Process

**Weekly Activity (1 hour mid-sprint):**
1. Review upcoming epics
2. Break into user stories
3. Write acceptance criteria
4. Estimate with planning poker
5. Ensure 2-3 sprints of refined backlog

**Estimation Scale:**
- 1-2 points: Simple (2-4 hours)
- 3-5 points: Moderate (1-3 days)
- 8 points: Complex (3-5 days)
- 13+ points: Too large - needs breakdown

**Detailed Process:** See [frameworks.md](references/frameworks.md) for refinement and estimation methods.

**Templates:** See [templates.md](references/templates.md) for story templates and splitting techniques.

## Python Tools

### user_story_generator.py
INVEST-compliant story generator with sprint planning.

**Key Features:**
- Epic breakdown into user stories
- Automatic acceptance criteria generation
- Story point estimation
- INVEST criteria validation
- Sprint capacity planning
- Multiple output formats (text, JSON, CSV)

**Usage:**
```bash
# Basic story generation
python3 scripts/user_story_generator.py epic.json

# Sprint planning
python3 scripts/user_story_generator.py --sprint --capacity 30

# JSON output for Jira import
python3 scripts/user_story_generator.py -o json -f stories.json

# CSV for spreadsheet import
python3 scripts/user_story_generator.py -o csv -f backlog.csv

# Verbose mode (show all stories)
python3 scripts/user_story_generator.py -v
```

**Story Point Estimation:**
- 1 pt: Simple, basic, view (2-4 hours)
- 3 pts: Create, edit, update (1 day)
- 5 pts: Moderate complexity (2-3 days)
- 8 pts: Complex, integrate (3-5 days)
- 13 pts: Redesign, refactor (1-2 weeks, needs breakdown)

**Complete Documentation:** See [tools.md](references/tools.md) for full usage guide, input formats, and integration patterns.

## Reference Documentation

### Frameworks ([frameworks.md](references/frameworks.md))
Comprehensive agile methodologies:
- INVEST Criteria: Detailed guidelines for each criterion
- Sprint Planning: Two-part planning framework
- Sprint Review & Retrospective: Complete ceremony guides
- Daily Standup: Three questions format and anti-patterns
- Backlog Refinement: Estimation and prioritization
- Velocity Tracking: Calculation, trends, burndown charts
- Release Planning: Capacity and timeline forecasting

### Templates ([templates.md](references/templates.md))
Ready-to-use templates:
- User Story Templates: Standard, feature, technical, bug fix formats
- Epic Templates: Complete epic structure with examples
- Sprint Ceremony Agendas: Planning, review, retrospective formats
- Story Splitting Techniques: By workflow, business rules, data entry
- Best Practices: Writing effective stories and avoiding pitfalls

### Tools ([tools.md](references/tools.md))
Python tool documentation:
- user_story_generator.py: Complete usage guide
- Input Formats: Epic JSON structure
- Output Formats: Text, JSON, CSV examples
- Command-Line Options: All flags and parameters
- Integration Patterns: Jira, Linear, ProductBoard workflows
- Troubleshooting: Common issues and solutions
- Best Practices: DO/DON'T guidelines

### Consolidated References

The following references were consolidated from BDD requirements discovery methodology:

| Reference | Content | When to Load |
|-----------|---------|--------------|
| [Example Mapping](references/example-mapping.md) | Four card types, session flow, conversational patterns, output artifacts, anti-patterns | Story about to enter sprint; team uncertain about scope; cross-functional clarification needed |
| [Three Amigos](references/three-amigos.md) | Three perspectives (business/dev/test), session structure, cadence, confirmation bias defense, anti-patterns | Backlog refinement; before sprint planning; after significant scope change |
| [Definition of Ready](references/definition-of-ready.md) | Hard gate criteria, requirements completeness (functional/NFR/business rules), quality review dimensions, enforcement without bureaucracy | Verifying story readiness before sprint entry; establishing team standards |

### Additional references (story decomposition and workshops)

The following references extend story decomposition and workshop execution patterns:

| Reference | Content | When to Load |
|-----------|---------|--------------|
| [Story Splitting Patterns](references/story-splitting-patterns.md) | SPIDR and practical splitting patterns with before/after examples | Stories are too large or carry hidden coupling |
| [Epic Breakdown Patterns](references/epic-breakdown-patterns.md) | Epic decomposition, dependency mapping, and vertical slicing heuristics | Planning a delivery sequence from a broad initiative |
| [Epic Hypothesis Format](references/epic-hypothesis-format.md) | Hypothesis-driven epic framing with measurable outcomes | Defining epics that need testable business outcomes |
| [Story Mapping Workshop](references/story-mapping-workshop.md) | Story mapping method plus facilitation protocol for release slicing | Running collaborative story mapping sessions with cross-functional teams |

## Integration Points

This toolkit integrates with:
- **Project Management:** Jira, Linear, Azure DevOps, Asana
- **Collaboration:** Confluence, Notion, Miro
- **Version Control:** GitHub Issues, GitLab Issues
- **Roadmapping:** ProductBoard, Aha!, Roadmunk

See [tools.md](references/tools.md) for detailed integration workflows.

## Quick Commands

```bash
# Generate stories from sample
python scripts/user_story_generator.py

# Generate from custom epic
python scripts/user_story_generator.py epic.json

# Sprint planning (30 points)
python scripts/user_story_generator.py --sprint --capacity 30

# Export as JSON
python scripts/user_story_generator.py -o json -f stories.json

# Export as CSV
python scripts/user_story_generator.py -o csv -f backlog.csv

# Verbose output (all stories)
python scripts/user_story_generator.py -v
```
