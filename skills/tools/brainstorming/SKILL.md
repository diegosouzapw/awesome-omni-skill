---
name: brainstorming
description: Use when creating or developing anything, before writing code or implementation plans - refines rough ideas into fully-formed designs through structured Socratic questioning, alternative exploration, and incremental validation. Optimized for git worktree workflows and Claude CLI agent patterns.
---

# Brainstorming Ideas Into Designs

## Overview

Transform rough ideas into fully-formed designs through structured questioning and alternative exploration.

**Core principle:** Ask questions to understand, explore alternatives, present design incrementally for validation.

**Announce at start:** "I'm using the brainstorming skill to refine your idea into a design."

## Quick Reference

| Phase | Key Activities | Output |
|-------|---------------|--------|
| **1. Understanding** | Ask questions (one at a time), assess current state | Purpose, constraints, criteria |
| **2. Exploration** | Propose 2-3 approaches with trade-offs | Architecture options |
| **3. Design Presentation** | Present in 200-300 word sections | Complete design with validation |
| **4. Implementation Planning** | Choose implementation strategy | Worktree + plan OR direct implementation |

## The Process

Copy this checklist to track progress:

```
Brainstorming Progress:
- [ ] Phase 0: Current State Assessment (check working directory, git status)
- [ ] Phase 1: Understanding (purpose, constraints, criteria gathered)
- [ ] Phase 2: Exploration (2-3 approaches proposed and evaluated)
- [ ] Phase 3: Design Presentation (design validated in sections)
- [ ] Phase 4: Implementation Planning (strategy chosen and executed)
```

### Phase 0: Current State Assessment

**ALWAYS start by checking the current state:**

```bash
# Check what's in the working directory
pwd
ls -la

# If it's a git repo, check status
git status
git worktree list  # See if worktrees exist
```

**Determine context:**
- Are we in a git repository?
- Are there existing worktrees?
- What files/structure already exist?
- Is this a new project or existing codebase?

**Based on findings, inform your approach:**
- New project → Start fresh with questions
- Existing project → Ask about integration points
- Multiple worktrees → Consider which worktree or if new one needed
- Clean repo → Can proceed with standard workflow

### Phase 1: Understanding

**Ask ONE question at a time** to refine the idea:

**Key questions to gather:**
- **Purpose:** What problem does this solve? Who is it for?
- **Constraints:** Time, technical, resource limitations?
- **Success criteria:** What does "done" look like?
- **Integration:** Does this connect to existing code/systems?
- **Scale:** Is this a small feature or large system?

**Example questions:**
- "What's the main problem you're trying to solve?"
- "Are there any specific technologies or patterns you want to use?"
- "Does this need to integrate with any existing systems?"
- "What's your timeline for this?"
- "How will you know when this is successful?"

**For skill development specifically:**
- "What triggers this skill? What phrases should activate it?"
- "What external data sources does it need?"
- "Should it generate files? What format?"
- "Does it need to call other skills?"

### Phase 2: Exploration

**Propose 2-3 different approaches** with clear trade-offs:

**For each approach, provide:**
- **Core architecture:** High-level structure
- **Trade-offs:** Pros and cons
- **Complexity:** Simple/Moderate/Complex
- **Best for:** What use case this serves

**Example approaches for a web service:**

**Approach 1: Serverless Functions (AWS Lambda/Netlify)**
- Architecture: Event-driven functions, managed infrastructure
- Pros: Auto-scaling, pay-per-use, minimal ops
- Cons: Cold starts, vendor lock-in, debugging challenges
- Complexity: Moderate (infrastructure-as-code)
- Best for: Variable traffic, event-driven workflows

**Approach 2: Containerized API (Docker + Express)**
- Architecture: Monolithic container, single process
- Pros: Easy to develop/debug locally, portable, standard
- Cons: Manual scaling, requires container orchestration
- Complexity: Simple to start, moderate at scale
- Best for: Steady traffic, team familiar with containers

**Approach 3: Microservices (Kubernetes)**
- Architecture: Multiple services, service mesh
- Pros: Independent scaling, team autonomy, resilient
- Cons: High operational overhead, complex debugging
- Complexity: High (networking, orchestration, monitoring)
- Best for: Large teams, complex domains, high scale

**Ask:** "Which approach resonates with your needs and constraints?"

### Phase 3: Design Presentation

**Present design in 200-300 word sections:**

**Typical sections:**
1. **Architecture Overview:** High-level diagram (in words/ASCII)
2. **Core Components:** Main modules and their responsibilities
3. **Data Flow:** How information moves through the system
4. **External Integrations:** APIs, databases, services
5. **Error Handling:** How failures are managed
6. **Testing Strategy:** Unit, integration, E2E approach
7. **Deployment:** How it gets from dev to production

**After each section:** "Does this look right so far? Any concerns or questions?"

**Stay flexible:** If the person questions something, be ready to:
- Return to Phase 1 for more clarity
- Return to Phase 2 to explore alternatives
- Adjust the current design

### Phase 4: Implementation Planning

**Once design is approved, choose implementation strategy:**

**Option A: Git Worktree Workflow (Recommended for existing repos)**

When to use:
- Working in existing git repository
- Want isolated development environment
- Making significant changes
- May need to switch back to main branch

Steps:
1. **Check current worktrees:** `git worktree list`
2. **Propose worktree location:** 
   - Adjacent to main repo: `../repo-name-feature`
   - Or subdirectory: `/tmp/worktrees/feature-name`
3. **Safety check:** Ensure directory doesn't exist
4. **Create worktree:**
   ```bash
   git worktree add -b feature/name ../feature-workspace
   cd ../feature-workspace
   ```
5. **Create planning document** in worktree
6. **Begin implementation** with context intact

**Option B: Direct Implementation (For simple features)**

When to use:
- Small, contained changes
- New project without git yet
- Quick prototype or experiment
- Working in /home/claude scratch space

Steps:
1. **Create working directory** if needed
2. **Create planning document** as markdown
3. **Begin implementation**

**Option C: Skill Development Workflow**

When to use:
- Creating or modifying a skill
- Need to bundle scripts/assets/references

Steps:
1. **Create skill directory structure:**
   ```
   /home/claude/skill-name/
   ├── SKILL.md          # Instructions for Claude
   ├── scripts/          # Python/bash scripts
   ├── assets/           # Fonts, images, templates
   └── references/       # Documentation, examples
   ```
2. **Write SKILL.md** with comprehensive instructions
3. **Add scripts** if skill needs to generate files
4. **Bundle as .zip** for user to upload

**Ask:** "Ready to move forward with [chosen option]?"

## Git Worktree Best Practices

**Based on your parallel development patterns:**

### Creating Worktrees

**Location strategy:**
```bash
# Adjacent to main repo (recommended)
git worktree add ../repo-feature feature/name

# In centralized worktree directory
git worktree add /tmp/worktrees/feature-name feature/name

# For agent-based workflows
git worktree add ../repo-architect architect/design-phase
git worktree add ../repo-bugfix bugfix/issue-123
git worktree add ../repo-docs docs/api-update
```

### Worktree Lifecycle

**Check existing worktrees:**
```bash
git worktree list
```

**Clean up when done:**
```bash
# Remove worktree
git worktree remove ../repo-feature

# Or if directory already deleted
git worktree prune
```

### Syncing Worktrees

**Keep worktrees updated:**
```bash
cd ../repo-feature
git fetch origin
git rebase origin/main  # Or merge if preferred
```

### Agent-Role Worktrees

**For specialized development roles:**
- **Architect worktree:** Design documents, architecture decisions
- **Implementation worktree:** Feature development, new code
- **Bugfix worktree:** Issue resolution, hotfixes
- **Documentation worktree:** README updates, API docs
- **Testing worktree:** Test development, QA work

Each can work independently, merge back to main when ready.

## Question Patterns

### Clarifying Questions (Use Throughout)

**Open-ended** (for understanding):
- "What problem are you trying to solve?"
- "How will users interact with this?"
- "What happens when X fails?"

**Specific** (for constraints):
- "What's your deployment target? (AWS/GCP/Local/etc.)"
- "What's your preferred language/framework?"
- "Do you have any existing code this needs to integrate with?"

**Binary** (for quick decisions):
- "Should this be synchronous or asynchronous?"
- "Do you want automated tests included?"
- "Should I create a git worktree for this?"

### Validation Questions (Phase 3)

**After each design section:**
- "Does this align with your needs?"
- "Does this look right so far?"
- "Any concerns about this approach?"
- "Anything you'd change here?"

**Before moving to implementation:**
- "Ready to move forward with this design?"
- "Would you like me to create a worktree for implementation?"
- "Should I write up a detailed implementation plan?"

## Key Principles

| Principle | Application |
|-----------|-------------|
| **Check context first** | Always assess current state before starting |
| **One question at a time** | Don't overwhelm with multiple questions |
| **YAGNI ruthlessly** | Remove unnecessary features from designs |
| **Explore alternatives** | Always propose 2-3 approaches before settling |
| **Incremental validation** | Present design in sections, validate each |
| **Flexible progression** | Go backward when needed - not rigid phases |
| **Worktree-aware** | Know when to suggest isolated workspace |
| **Skill-conscious** | Recognize when task would benefit from a skill |

## Common Scenarios

### Scenario 1: New Feature in Existing Repo

```
1. Phase 0: Check git status, current worktrees
2. Phase 1: Ask about feature purpose, integration points
3. Phase 2: Propose 2-3 implementation approaches
4. Phase 3: Present detailed design for chosen approach
5. Phase 4: Create worktree, write plan, begin implementation
```

### Scenario 2: New Project from Scratch

```
1. Phase 0: Confirm working in /home/claude or if git init needed
2. Phase 1: Deep-dive questions on requirements
3. Phase 2: Explore architectural patterns (monolith/microservices/etc)
4. Phase 3: Complete system design with all components
5. Phase 4: Create project structure, write plan
```

### Scenario 3: Creating a New Skill

```
1. Phase 0: Check /mnt/skills/user directory
2. Phase 1: Ask about trigger phrases, inputs, outputs
3. Phase 2: Explore skill architectures (simple/script-based/multi-step)
4. Phase 3: Design skill workflow, script needs, asset requirements
5. Phase 4: Create skill directory, write SKILL.md, bundle
```

### Scenario 4: Bug Fix or Refactor

```
1. Phase 0: Examine current code, understand issue
2. Phase 1: Clarify scope (just fix vs. refactor vs. redesign)
3. Phase 2: Explore fix approaches (minimal vs. comprehensive)
4. Phase 3: Present solution with test strategy
5. Phase 4: Worktree for bugfix, implement with tests
```

## Integration with Other Skills

**When brainstorming leads to skill development:**
- Use **skill-creator** skill for comprehensive skill design
- Bundle scripts, assets, references appropriately
- Write detailed SKILL.md following patterns

**When brainstorming leads to complex implementation:**
- Consider using **planning skills** for detailed task breakdown
- Create milestone-based plan in worktree
- Hand off to implementation phase with full context

**When brainstorming involves research:**
- Use **web_search** during exploration phase
- Research technologies, patterns, best practices
- Validate assumptions about libraries/frameworks

## Examples

### Example 1: API Service Design

**Phase 1 - Understanding:**
```
Q: "What will this API do?"
A: "Handle user authentication and profile management"

Q: "What's your expected scale?"
A: "Start with 1000 users, grow to 10k"

Q: "Any specific security requirements?"
A: "HIPAA compliance needed"
```

**Phase 2 - Exploration:**
```
Approach 1: OAuth2 with JWT tokens (industry standard, complex)
Approach 2: Session-based auth (simpler, server state required)
Approach 3: API keys only (simple, limited features)

[User selects Approach 1]
```

**Phase 3 - Design:**
```
Architecture: Express.js + PostgreSQL + Redis
- Auth endpoints: /login, /logout, /refresh
- Profile endpoints: /profile, /profile/update
- JWT tokens with refresh token rotation
- Redis for token blacklisting
...
```

### Example 2: Creating Research Skill

**Phase 1:**
```
Q: "What kind of research should this skill do?"
A: "Competitive analysis for product features"

Q: "What output format?"
A: "Markdown report with comparison table"
```

**Phase 2:**
```
Approach 1: Web search + structured parsing (automated)
Approach 2: Web search + manual analysis (flexible)
Approach 3: API integration (if competitors have APIs)
```

**Phase 3:**
```
Skill workflow:
1. Ask user for competitor list
2. Search for each competitor's features
3. Parse and structure findings
4. Generate comparison markdown
5. Create summary with recommendations
```

## Tips for Success

**Do:**
- ✅ Check current state before asking questions
- ✅ Ask one question at a time
- ✅ Present trade-offs clearly in alternatives
- ✅ Validate design incrementally
- ✅ Suggest worktrees for significant work
- ✅ Go backward if new info emerges
- ✅ Use web search to validate technical choices

**Don't:**
- ❌ Assume you know what user needs
- ❌ Present only one approach
- ❌ Dump entire design at once
- ❌ Force linear progression when backtracking is better
- ❌ Forget to check existing worktrees
- ❌ Skip validation questions
- ❌ Ignore git repository context

## Skill Announcement

**At the start of every brainstorming session, announce:**

"I'm using the brainstorming skill to refine your idea into a design. I'll start by checking the current project state, then ask questions to understand your needs, explore different approaches, and incrementally present a complete design for your validation."

This sets expectations and shows you're following a structured process.

---

**Remember:** This skill is about conversation, not interrogation. Stay flexible, be curious, and help your partner think through their ideas thoroughly before jumping into implementation.