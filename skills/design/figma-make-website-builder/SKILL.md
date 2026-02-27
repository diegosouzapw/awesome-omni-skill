---
name: figma-make-website-builder
description: Structured 9-phase workflow for building production-ready websites using Claude (architecture, logic, reasoning) paired with Figma Make (UI, interactions, deployment). Use when planning, designing, or building a website with Figma Make.
alwaysApply: false
metadata:
  author: codekit
  tags:
    - figma
    - website
    - design
    - workflow
    - production
---

# Figma Make Website Builder

A structured 9-phase workflow for building production-ready websites using Claude paired with Figma Make. Claude handles architecture, logic, content strategy, and QA. Figma Make handles pixel-perfect UI, interactions, and deployment.

## Quick Reference

| Phase | File | Role | Description |
|-------|------|------|-------------|
| 1 | [phase-1-architecture-strategy.md](references/phase-1-architecture-strategy.md) | Principal Architect | Site map, user journeys, component inventory, tech stack |
| 2 | [phase-2-design-system.md](references/phase-2-design-system.md) | Design Director | Color, typography, spacing, 30+ component specs |
| 3 | [phase-3-content-architecture.md](references/phase-3-content-architecture.md) | Conversion Copywriter | Headlines, CTAs, social proof, FAQ content |
| 4 | [phase-4-component-logic.md](references/phase-4-component-logic.md) | Frontend Architect | State machines, data flow, error handling, React structure |
| 5 | [phase-5-prompt-engineering.md](references/phase-5-prompt-engineering.md) | AI Prompt Engineer | Convert specs into 5 Figma Make prompts |
| 6 | [phase-6-animation-interaction.md](references/phase-6-animation-interaction.md) | Motion Designer | Load sequences, scroll behaviors, micro-interactions |
| 7 | [phase-7-responsive-strategy.md](references/phase-7-responsive-strategy.md) | Responsive Specialist | Breakpoints, layout transforms, content prioritization |
| 8 | [phase-8-data-integration.md](references/phase-8-data-integration.md) | Full-Stack Architect | Data models, APIs, auth, Supabase integration |
| 9 | [phase-9-qa-optimization.md](references/phase-9-qa-optimization.md) | QA Engineer | Performance, accessibility, SEO, security checklist |

## Workflow Overview

```
Phase 1-4 (Claude)     Phase 5 (Bridge)     Phase 6-8 (Claude)     Phase 9 (Review Loop)
┌─────────────────┐    ┌──────────────┐     ┌─────────────────┐    ┌──────────────────┐
│ 1. Architecture │    │ 5. Convert   │     │ 6. Animation    │    │ 9. QA Checklist  │
│ 2. Design System│───▶│    specs to  │────▶│ 7. Responsive   │───▶│    ↕ iterate     │
│ 3. Content      │    │ Figma Make   │     │ 8. Data Layer   │    │ Figma Make edits │
│ 4. Logic        │    │   prompts    │     │                 │    └──────────────────┘
└─────────────────┘    └──────────────┘     └─────────────────┘
```

- **Phases 1-4** and **6-8**: Done in Claude
- **Phase 5**: Bridge — converts Claude output into Figma Make prompts
- **Phase 9**: Review loop — iterate between Claude QA and Figma Make refinement
- **Target**: ~2 hours for a complete production website

## Problem → Phase Mapping

| Problem | Start With |
|---------|------------|
| Starting a new website from scratch | [Phase 1 — Architecture](references/phase-1-architecture-strategy.md) |
| Need a design system or brand tokens | [Phase 2 — Design System](references/phase-2-design-system.md) |
| Writing website copy and CTAs | [Phase 3 — Content](references/phase-3-content-architecture.md) |
| Building interactive components (forms, calculators) | [Phase 4 — Component Logic](references/phase-4-component-logic.md) |
| Ready to prompt Figma Make | [Phase 5 — Prompt Engineering](references/phase-5-prompt-engineering.md) |
| Adding animations and interactions | [Phase 6 — Animation](references/phase-6-animation-interaction.md) |
| Making a site responsive | [Phase 7 — Responsive](references/phase-7-responsive-strategy.md) |
| Connecting a database or API | [Phase 8 — Data Integration](references/phase-8-data-integration.md) |
| Reviewing and optimizing before launch | [Phase 9 — QA](references/phase-9-qa-optimization.md) |

## Usage

Run phases sequentially — each phase's output feeds the next. You can also jump to individual phases for specific tasks. When running the full workflow, provide the website type (portfolio, SaaS, e-commerce, etc.) and brand attributes (minimal, bold, luxury, playful) at Phase 1 to propagate context through all subsequent phases.
