---
name: blueprintkit-plugin
description: BlueprintKit Plugin - Complete planning framework automatically including all 14 planning sections (.claude/skills/blueprintkit/planning/0-Master-Index.md through .claude/skills/blueprintkit/planning/13-Lessons-Learned-Continuous-Improvement.md) plus all 9 Claude Skills (tech-stack-selector, architecture-decisions, code-standards-enforcer, ci-cd-pipeline-builder, agile-executor, project-risk-identifier, automation-orchestrator, webapp-testing, web-artifacts-builder). When installed, all files are immediately available.
---

# BlueprintKit Plugin

This plugin automatically includes the complete BlueprintKit framework: all 14 planning sections plus all 9 specialized Claude Skills. When installed, all planning templates and execution skills are immediately available in a single package.

## Overview

The Project Planning Starter Pack Plugin provides a unified interface to a complete planning framework with 14 comprehensive planning sections plus nine specialized skills that cover the complete project lifecycle from planning through deployment and continuous improvement.

## Complete Planning Framework

This plugin includes 14 comprehensive planning sections:

1. **Master Index** - Complete navigation guide and overview
2. **Executive Summary** - Vision, problem statement, expected outcomes
3. **Objectives & Success Metrics** - Quantified success criteria and KPIs
4. **Scope Definition** - What's in/out, constraints, assumptions
5. **System Architecture & Design** - Technical blueprint and architecture decisions
6. **Technical Execution Workflow** - Complete technical implementation guide
7. **Project Phases & Timeline** - Phases, milestones, and timeline
8. **Resource Planning** - Team structure, skills, budget allocation
9. **Risk Management** - Risk identification, mitigation, contingency planning
10. **Execution Strategy** - Daily execution, ceremonies, quality assurance
11. **Monitoring & Reporting** - Metrics tracking and status reporting
12. **ROI & Value Realization** - Financial projections and value tracking
13. **Governance & Decision-Making** - Decision authority and escalation procedures
14. **Lessons Learned & Continuous Improvement** - Learning capture and process improvement

## Bundled Skills

This plugin also includes nine specialized execution skills:

1. **tech-stack-selector** - Technology decision framework with structured recommendations
2. **architecture-decisions** - Architecture Decision Records (ADRs) documentation
3. **code-standards-enforcer** - Comprehensive code review checklists and quality standards
4. **ci-cd-pipeline-builder** - GitHub Actions workflow templates and automation
5. **agile-executor** - Sprint planning, retrospectives, and Agile ceremony facilitation
6. **project-risk-identifier** - Risk assessment frameworks and mitigation strategies
7. **automation-orchestrator** - Script orchestration for setup, validation, and deployment
8. **webapp-testing** - Playwright-based web application testing toolkit
9. **web-artifacts-builder** - React + TypeScript artifact creation with shadcn/ui

## Installation

### For Claude Code Users

1. Clone or copy this repository to your local machine
2. Ensure the `.claude-plugin/` directory is in your project root
3. Claude Code will automatically detect and load the plugin

### For Skills.sh Users

Install BlueprintKit from skills.sh:

```bash
npx skills add JustineDevs/blueprintkit
```

This automatically installs:
- All 14 planning sections from `.claude/skills/blueprintkit/planning/` directory
- All 9 Claude Skills from `.claude/skills/blueprintkit/` directory
- Complete framework ready to use immediately

## Usage

### Individual Skill Access

Each skill in `.claude/skills/blueprintkit/` can be accessed individually. Skills auto-activate based on natural language queries:

- "What tech stack should we use?" → tech-stack-selector
- "Create an ADR" → architecture-decisions
- "Code review checklist" → code-standards-enforcer
- "Setup CI/CD" → ci-cd-pipeline-builder
- "Plan sprint" → agile-executor
- "Identify risks" → project-risk-identifier
- "Set up Claude skills" → automation-orchestrator
- "Test web application" → webapp-testing
- "Build web artifact" → web-artifacts-builder

### Plugin-Level Usage

The plugin provides unified access to all skills. Ask Claude:

- "Help me plan this project" - Activates multiple relevant skills
- "Set up project automation" - Uses automation-orchestrator and related skills
- "Review project quality" - Combines code-standards-enforcer with risk management

## Automatic File Access

When BlueprintKit is installed, the following files are automatically available:

### Planning Sections (All 14 Files)
- `.claude/skills/blueprintkit/planning/0-Master-Index.md` - Complete navigation guide
- `.claude/skills/blueprintkit/planning/1-Executive-Summary.md` - Vision and problem statement
- `.claude/skills/blueprintkit/planning/2-Objectives-Success-Metrics.md` - Success criteria and KPIs
- `.claude/skills/blueprintkit/planning/3-Scope-Definition.md` - Project boundaries
- `.claude/skills/blueprintkit/planning/4-System-Architecture-Design.md` - Technical blueprint
- `.claude/skills/blueprintkit/planning/5-Technical-Execution-Workflow.md` - Implementation guide
- `.claude/skills/blueprintkit/planning/6-Project-Phases-Timeline.md` - Phases and milestones
- `.claude/skills/blueprintkit/planning/7-Resource-Planning.md` - Team and budget
- `.claude/skills/blueprintkit/planning/8-Risk-Management.md` - Risk identification and mitigation
- `.claude/skills/blueprintkit/planning/9-Execution-Strategy.md` - Daily execution and ceremonies
- `.claude/skills/blueprintkit/planning/10-Monitoring-Reporting.md` - Metrics and reporting
- `.claude/skills/blueprintkit/planning/11-ROI-Value-Realization.md` - Financial projections
- `.claude/skills/blueprintkit/planning/12-Governance-Decision-Making.md` - Decision authority
- `.claude/skills/blueprintkit/planning/13-Lessons-Learned-Continuous-Improvement.md` - Learning capture

### Claude Skills (All 9 Skills)
- `.claude/skills/blueprintkit/tech-stack-selector/SKILL-INTERNAL.md` - Technology decisions
- `.claude/skills/blueprintkit/architecture-decisions/SKILL-INTERNAL.md` - ADR documentation
- `.claude/skills/blueprintkit/code-standards-enforcer/SKILL-INTERNAL.md` - Code quality
- `.claude/skills/blueprintkit/ci-cd-pipeline-builder/SKILL-INTERNAL.md` - CI/CD automation
- `.claude/skills/blueprintkit/agile-executor/SKILL-INTERNAL.md` - Agile ceremonies
- `.claude/skills/blueprintkit/project-risk-identifier/SKILL-INTERNAL.md` - Risk assessment
- `.claude/skills/blueprintkit/automation-orchestrator/SKILL-INTERNAL.md` - Script orchestration
- `.claude/skills/blueprintkit/webapp-testing/SKILL-INTERNAL.md` - Web testing toolkit
- `.claude/skills/blueprintkit/web-artifacts-builder/SKILL-INTERNAL.md` - Artifact creation

Each skill directory contains:
- `SKILL-INTERNAL.md` - Internal skill definition (not detected by skills.sh, only accessible via blueprintkit)
- `references/` - Detailed documentation and templates
- `scripts/` - Utility scripts for automation

## Integration with Project Planning

This plugin integrates with the Project Planning Starter Pack's comprehensive planning framework:

- **Planning Sections** (`.claude/skills/blueprintkit/planning/`) - 14 planning templates
- **Technical Documentation** (`docs/reference/`) - Architecture and technical summaries
- **Configuration Files** - CI/CD, linting, testing configurations
- **Ready-to-Build Structure** - Organized application code directories

## Examples

### Example 1: Complete Project Setup

**User**: "Help me set up a new TypeScript project with CI/CD"

**Plugin Response**:
1. Activates tech-stack-selector for technology recommendations
2. Activates ci-cd-pipeline-builder for GitHub Actions setup
3. Activates code-standards-enforcer for quality gates
4. Activates automation-orchestrator for script setup
5. Provides integrated workflow combining all skills

### Example 2: Architecture Documentation

**User**: "Document our database choice and set up testing"

**Plugin Response**:
1. Activates architecture-decisions for ADR creation
2. Activates webapp-testing for test setup
3. Links to planning sections for context
4. Provides integrated documentation workflow

## Configuration

The plugin is configured via `.claude-plugin/plugin.mdc` which defines:
- Activation rules
- File patterns to monitor
- Related documentation references
- Plugin metadata

## Contributing

To contribute to this plugin:

1. Review individual skill definitions in `.claude/skills/blueprintkit/`
2. Follow the skill creation guidelines in `skills-reference/template/SKILL.md`
3. Update this plugin's SKILL.md when adding new skills
4. Test plugin loading in Claude Code
5. Submit improvements via pull request

## Related Documentation

- [Main README](../README.md) - Project overview
- [Claude Skills README](../.claude/README.md) - Skills documentation
- [Planning Framework](../.claude/skills/blueprintkit/planning/) - Complete planning templates
- [Technical Summary](../docs/reference/TECHNICAL-SUMMARY.md) - Technical reference

## License

MIT License - See [LICENSE](../LICENSE) file for details.

