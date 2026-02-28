---
name: Onboarding Guide Generator
description: Generate onboarding guides for new team members based on project structure
category: documentation
version: 1.0.0
triggers:
  - new-team-member
  - readme-changes
  - onboarding-request
globs: "**/docs/**,**/README.md,**/CONTRIBUTING.md"
---

# Onboarding Guide Generator Skill

Generate onboarding guides for new team members based on project structure.

## Trigger Conditions
- New team member event
- README or contributing guide changes significantly
- User invokes with "generate onboarding guide" or "new developer setup"

## Input Contract
- **Required:** Project repository structure
- **Required:** Development environment requirements
- **Optional:** Team-specific context, role (frontend, backend, fullstack)

## Output Contract
- Step-by-step onboarding guide
- Environment setup checklist
- Key architecture and codebase orientation
- Estimated time-to-productivity

## Tool Permissions
- **Read:** All project files, docs, configs, README, contributing guide
- **Write:** Onboarding guides in `docs/onboarding/`
- **Search:** Project structure and key files

## Execution Steps
1. Analyze project structure and technology stack
2. Identify environment prerequisites (languages, tools, services)
3. Map the development workflow (build, test, deploy)
4. Identify key documentation and architecture resources
5. Create step-by-step setup guide with verification steps
6. Estimate time-to-productivity
7. Generate checklist format for tracking

## Success Criteria
- Setup guide is complete with no undocumented steps
- Each step has a verification command
- Time-to-productivity estimated
- Key resources linked

## Escalation Rules
- Escalate if setup requires undocumented manual steps
- Escalate if prerequisites are not available or documented

## Example Invocations

**Input:** "Generate an onboarding guide for a new backend developer"

**Output:** Onboarding guide: 12 steps, estimated 3 hours. Prerequisites: Go 1.22+, Docker, PostgreSQL. Steps: 1) Clone repo, 2) Install Go tools, 3) Start Docker services, 4) Run migrations, 5) Run tests, 6) Start dev server... Each step has verification (e.g., "go test ./... should show 0 failures"). Architecture overview and key ADRs linked.
