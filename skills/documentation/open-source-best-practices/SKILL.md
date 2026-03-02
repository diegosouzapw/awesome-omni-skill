---
name: open-source-best-practices
description: Validates and prepares a GitHub project for open source release by ensuring all essential documentation and legal foundations are in place. Uses Git History Cleaner to identify and remove secrets, credentials, and sensitive data before publication. Use when you want to release a project publicly or harden an existing public repo.
license: See LICENSE file in repository root
metadata:
  author: AndreaGriffiths11
  version: "1.0"
allowed-tools: file_reader, file_writer, github_api, license_selector, documentation_validator, git_history_analyzer
---

# Open Source Best Practices

This skill guides you through preparing your GitHub project for sustainable open source release.

## How to Use This Skill

1. **See the full workflow** in [AGENTS.md](AGENTS.md) - the complete phases and checklist
2. **Reference detailed guides** in [references/](references/) folder:
   - File requirements and structure
   - License selection decision tree
   - Security scanning and git history cleaning
   - Governance framework
   - Maintainer expectations
   - GitHub Sponsors setup
   - Template examples

## Quick Overview

The workflow has 8 phases (do them in order; Phase 1 isn't optional):

1. **Security First** - Clean your git history using Git History Cleaner
2. **Legal & Ownership** - Choose license, verify ownership, clarify admin rights
3. **Community Foundations** - Add Code of Conduct, governance, decision-making
4. **Documentation & Onboarding** - README, CONTRIBUTING, issue/PR templates
5. **Setup Files & Infrastructure** - .gitignore, CI/CD, protected branches
6. **Maintainer Expectations** - Define roles, SLAs, communication
7. **Security & Vulnerability Reporting** - SECURITY.md, vulnerability process
8. **Funding & Sustainability** - GitHub Sponsors (optional but recommended)

## Get Started

When a user asks about open sourcing their project, begin by asking:
- **"What does your project do?"** - Understand scope
- **"Who's the audience?"** - Know your users
- **"Is your git history clean?"** - Check for secrets first

Then guide them through the phases using the full [AGENTS.md](AGENTS.md) workflow.

## Key Resources

- [AGENTS.md](AGENTS.md) - Complete 8-phase workflow
- [references/file-checklist.md](references/file-checklist.md) - What files and why
- [references/license-selection.md](references/license-selection.md) - How to choose
- [references/security-practices.md](references/security-practices.md) - Clean git history
- [references/governance.md](references/governance.md) - Make decisions sustainably
- [references/maintainer-expectations.md](references/maintainer-expectations.md) - Healthy projects
- [references/sponsors-setup.md](references/sponsors-setup.md) - Enable funding
- [references/template-examples.md](references/template-examples.md) - Copy-paste templates
