---
name: agent-selection
description: Systematic framework for selecting the optimal specialized agent for any task. Use when delegating to subagents via the Task tool to ensure the most appropriate specialist is chosen based on framework, domain, task type, and complexity. Applies decision tree logic to match tasks with agent expertise.
---

# Agent Selection

## Overview

Select the optimal specialized agent for any task using systematic decision-making criteria. This skill provides a structured approach to matching tasks with agent expertise across frameworks, domains, project types, and quality concerns.

## When to Use This Skill

Invoke this skill before using the Task tool to delegate work to a specialized agent. Specifically use when:

- Delegating complex or multi-step tasks that match specialized agent capabilities
- Framework-specific work (React, Vue, Laravel, Node.js, Swift, etc.)
- Domain-specific tasks (database optimization, security audits, API design, graphics)
- Quality assurance tasks (code review, testing strategy, architecture review, performance profiling)
- Project-type specific work (MCP servers, Figma plugins, VS Code extensions, game development)
- Multi-domain tasks requiring orchestration across multiple specialists
- Exploring unfamiliar codebases or investigating complex bugs

## Agent Selection Process

Follow this systematic process to select the appropriate agent:

### Step 1: Analyze Task Characteristics

Identify the key characteristics of the task:

1. **Framework dependency**: Does the task require specific framework knowledge?
   - Frontend: React 18+ (hooks, context, Suspense, lazy loading)
   - TypeScript: strict typing, type guards, generics, zero 'any'
   - Build: Vite (HMR, code-splitting, optimization)
   - Testing: Vitest + @testing-library/react, Playwright e2e

2. **Domain specialization**: Does the task fall into a specific domain?
   - Security (audits, vulnerability assessment, OWASP compliance)
   - Performance (profiling, optimization, bundle analysis)
   - Documentation (API docs, guides, architecture docs, READMEs)
   - Content (marketing copy, communication, user-facing content)

3. **Project type**: Is this a specific project type?
   - Animation design (planning, storyboards, motion strategy)
   - Animation implementation (Framer Motion, CSS animations)
   - UI polish (transitions, visual refinements, micro-interactions)
   - Animation catalog/registry system

4. **Quality concern**: Is this a quality/architecture task?
   - Code review (after writing/modifying code)
   - Testing strategy and implementation (TDD, coverage, meaningful tests)
   - Architecture enforcement and refactoring (patterns, SOLID principles)
   - Performance optimization (React profiling, bundle size, lighthouse)

5. **Investigation type**: Is this exploratory or diagnostic?
   - Codebase exploration (finding files/patterns in animation registry)
   - Bug investigation (root cause analysis, reproduction)
   - Context management (token optimization, focused queries)
   - Planning complex implementations

### Step 2: Apply Selection Logic

Use this decision tree to select the agent:

**Framework-Specific Tasks** → Select framework agent
- React 18+ work (hooks, context, Suspense) → `react-specialist`
- TypeScript strict typing, zero 'any' → `typescript-guardian`

**Domain-Specific Tasks** → Select domain specialist
- Security audits, OWASP, vulnerabilities → `security-auditor`
- Documentation (API docs, guides, READMEs) → `technical-writer`
- Performance optimization, profiling → `performance-profiler`

**Project-Type Tasks** → Select project specialist
- Animation design (planning, storyboards) → `animation-designer`
- Animation implementation (Framer Motion) → `animation-developer`
- UI polish (smooth transitions, visual refinements) → `ui-polish-specialist`
- Content strategy (marketing, communication) → `content-marketer`

**Quality & Architecture Tasks** → Select quality agent
- Code review (after writing/modifying code) → `code-reviewer`
- Test strategy (Vitest, Playwright, TDD) → `testing-architect`
- Architecture review (patterns, refactoring) → `architecture-guardian`
- Performance issues (profiling, optimization) → `performance-profiler`

**Investigation Tasks** → Select investigation agent
- Bug investigation, root cause analysis → `debugger`
- Context management (token optimization) → `context-manager`

**No Clear Match** → Use general delegation
- Ambiguous tasks → Auto-delegation (main agent handles directly)

### Step 3: Verify Selection

Before delegating, verify the selection makes sense:

1. **Expertise match**: Does the agent's expertise align with task requirements?
2. **Scope appropriateness**: Is the task complex enough to warrant delegation?
3. **Context availability**: Can the task be completed independently by the agent?
4. **Parallel execution**: Can multiple agents work in parallel on independent subtasks?

### Step 4: Delegate with Clear Context

When using the Task tool:
   - Call the subagent and provide all the context that they need to do the task at highest quality in the context of the whole codebase.
   - Never send a one-liner prompt to a subagent! Always provide them with all context they need to do the job. Do not let them start from scratch.
   - Always include references to required documentation:
     - `docs/architecture.md`
     - `docs/api.md`
     - `docs/testing.md`
     - include other or more documentation references in the agent prompt if needed for the task
   - **NEVER** just handover the task you have been given to an agent without passing on the work you have already done! Do not waste token on letting agents repeat the research or work you have already done!
   - Be conscious of token usage! Do not duplicate work in the agent that you or another agent have already done!

## Quick Reference Patterns

Common task patterns and their agent matches:

**"Review this React component for performance"**
→ `performance-profiler` (not react-specialist, because focus is performance)

**"Build a new animation component with Framer Motion"**
→ `animation-developer`

**"Design the motion strategy for modal transitions"**
→ `animation-designer`

**"Audit this codebase for XSS vulnerabilities"**
→ `security-auditor`

**"Eliminate 'any' types from this component"**
→ `typescript-guardian`

**"After writing new animation code, review for quality"**
→ `code-reviewer` (proactive quality check)

**"Implement test coverage for the animation registry"**
→ `testing-architect`

**"Find all modal animation components"**
→ `Explore` with thoroughness="medium"

**"Investigate why the animation replay button fails"**
→ `debugger`

**"Refactor animation metadata to follow co-located pattern"**
→ `architecture-guardian`

**"Polish the animation card hover states"**
→ `ui-polish-specialist`

**"Write marketing copy for animation showcase"**
→ `content-marketer`

**"Optimize token usage when analyzing large registry"**
→ `context-manager`

## Edge Cases and Special Considerations

**Multiple viable agents**: When a task could fit multiple agents (e.g., animation component with TypeScript errors could use animation-developer OR typescript-guardian):
- Prioritize the primary concern (types → typescript-guardian, animation logic → animation-developer)
- For animation work: animation-designer (planning) → animation-developer (implementation)

**Proactive quality agents**: Some agents should be invoked proactively without user request:
- `code-reviewer` → After writing/modifying significant code
- `testing-architect` → When adding new functionality (ensure 100% coverage)
- `typescript-guardian` → When seeing 'any' types or type errors

