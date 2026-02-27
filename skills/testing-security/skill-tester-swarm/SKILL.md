---
name: skill-tester-swarm
description: Automated QA system for Claude Code skills. Discovers skills from GitHub/SkillsMP, installs them, generates tests, executes with screenshot capture, evaluates quality with A-F grades, and publishes an interactive dashboard.
version: 1.0.0
author: niveshdandyan
triggers:
  - test skills
  - skill tester
  - test claude skills
  - skill quality
  - skill qa
  - evaluate skills
  - skill dashboard
---

# Skill Tester Swarm

> "I don't just use skills. I test them all and show you which ones actually work."

An 8-agent swarm that automatically tests Claude Code skills and generates visual reports with screenshots.

## What It Does

1. **Discovers** skills from GitHub and marketplaces
2. **Installs** each in isolation
3. **Generates** test prompts from triggers
4. **Executes** tests and captures output
5. **Screenshots** terminal and file results
6. **Scores** quality (A-F grades)
7. **Reports** with pass/fail matrix
8. **Publishes** interactive dashboard

## Quick Start

```bash
./scripts/orchestrator.sh run
```

## Output

- Screenshots of every test
- Quality scores and grades
- Interactive web dashboard
- JSON data for automation

## Agents

| Agent | Role |
|-------|------|
| 0 | Skill Discoverer |
| 1 | Skill Installer |
| 2 | Test Generator |
| 3 | Test Executor |
| 4 | Screenshot Capturer |
| 5 | Quality Evaluator |
| 6 | Report Generator |
| 7 | Dashboard Publisher |

See README.md for full documentation.
