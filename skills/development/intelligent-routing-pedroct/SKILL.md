---
name: intelligent-routing
description: Route tasks to the right specialist agents and skills.
---

# Intelligent Routing

## When to use
At the start of a request to select agents and skills.

## Signals
- UI, CSS, layout, accessibility: frontend-specialist
- API, data, auth, business logic: backend-specialist
- CI/CD, build tooling, DX: platform-engineer
- Infra, deploy, environments: devops-engineer
- Integrations, webhooks, SDKs: integrations-engineer
- Data ingestion, ETL, pipelines: data-pipeline-engineer
- Bugs and stack traces: debugger
- Risky changes and secrets: security-auditor
- Tests and coverage: qa-engineer
- Scope and tradeoffs: product-manager
- Data modeling: data-architect

## Steps
1. Classify the request into 1-2 domains.
2. Select matching agent profiles from `agents/`.
3. Load relevant skills from `.agents/skills/`.
4. State which agents and skills are being applied.
