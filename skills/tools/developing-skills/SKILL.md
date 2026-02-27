---
name: developing-skills
description: "MUST be loaded before working with any Skill. Covers creating, building, reviewing, assessing, checking, auditing, evaluating, updating, modifying, and improving skills. Invoke PROACTIVELY before writing or changing any SKILL.md file. Provides structure, workflows, and validation for skill development. Supports both personal skills and standalone distributable skills (GitHub repos). (user)"
---

<essential_principles>
Skills are modular, filesystem-based capabilities that provide domain expertise on demand.

**1. Name and Description Are CRITICAL** — The name and description are the ONLY way agents discover and decide to use a skill. If these are wrong, the skill will never be invoked. Treat them as the most important lines in the entire skill. See references/skill-structure.md for detailed guidance.

**2. Skills Are Prompts** — All prompting best practices apply. Be clear, be direct, use XML structure. Assume Claude is smart - only add context Claude doesn't have.

**3. SKILL.md Is Always Loaded** — When invoked, the agent reads SKILL.md. Use this guarantee:
- Essential principles go in SKILL.md (can't be skipped)
- Workflow-specific content goes in workflows/
- Reusable knowledge goes in references/

**4. Router Pattern for Complex Skills**
```
skill-name/
├── SKILL.md              # Router + principles
├── workflows/            # Step-by-step procedures (FOLLOW)
├── references/           # Domain knowledge (READ)
├── templates/            # Output structures (COPY + FILL)
└── scripts/              # Reusable code (EXECUTE)
```
SKILL.md routes to workflow → workflow specifies which references to read.

**6. Progressive Disclosure** — SKILL.md under 500 lines. Split detailed content into reference files. Load only what's essential make common activities fast and detailed aspects of the skill discoverable.

**7. Challenge Every Token** — Context window is shared. Before adding content, ask: "Does the agent already know this?" If in doubt, leave it out.

**8. Skills Can Be Standalone Repos** — Skills can live in personal skills directories or as standalone GitHub repos (distributable). Detect context and confirm intent.
</essential_principles>

<intake>
What would you like to do?

1. Create new skill
2. Audit/modify existing skill
3. Add component (workflow/reference/template/script)
4. Get guidance

**Ensure this is clear before proceeding.**
</intake>

<routing>
| Response | Next Action | Workflow |
|----------|-------------|----------|
| 1, "create", "new", "build", "develop" | Ask: "Task-execution skill or domain expertise skill?" | Route to appropriate develop workflow |
| 2, "audit", "modify", "existing" | Ask: "Path to skill?" | Route to appropriate workflow |
| 3, "add", "component" | Ask: "Add what? (workflow/reference/template/script)" | workflows/add-{type}.md |
| 4, "guidance", "help" | General guidance | workflows/get-guidance.md |

**Progressive disclosure for option 1 (develop):**
- If user selects "Task-execution skill" → workflows/develop-new-skill.md
- If user selects "Domain expertise skill" → workflows/develop-domain-expertise-skill.md

**Progressive disclosure for option 3 (add component):**
- If user specifies workflow → workflows/add-workflow.md
- If user specifies reference → workflows/add-reference.md
- If user specifies template → workflows/add-template.md
- If user specifies script → workflows/add-script.md

**Intent-based routing (if user provides clear intent without selecting menu):**
- "audit this skill", "check skill", "review" → workflows/audit-skill.md
- "verify content", "check if current" → workflows/verify-skill.md
- "develop domain expertise", "exhaustive knowledge base" → workflows/develop-domain-expertise-skill.md
- "develop skill for X", "build new skill", "create skill" → workflows/develop-new-skill.md
- "add workflow", "add reference", etc. → workflows/add-{type}.md
- "upgrade to router" → workflows/upgrade-to-router.md

**After reading the workflow, follow it exactly.**
</routing>

<escalation_triggers>
Stop and ask the user when:
- Skill scope is unclear after initial questions (don't guess at requirements)
- Multiple valid architectural approaches exist (simple vs. router pattern)
- External API is involved but documentation is outdated or conflicting
- Existing skill has unusual structure that may be intentional
- Changes would affect more than 10 files in an existing skill
</escalation_triggers>

<quick_reference>
**Simple skill (single file):**
```yaml
---
name: skill-name
description: "What it does and when to use it."
---

<objective>What this skill does</objective>
<quick_start>Immediate actionable guidance</quick_start>
<process>Step-by-step procedure</process>
<success_criteria>How to know it worked</success_criteria>
```

**Complex skill (router pattern):**
```
SKILL.md:
  <essential_principles> - Always applies
  <intake> - Question to ask
  <routing> - Maps answers to workflows

workflows/:
  <required_reading> - Which refs to load
  <process> - Steps
  <success_criteria> - Done when...

references/:
  Domain knowledge, patterns, examples

templates/:
  Output structures Claude copies and fills
  (plans, specs, configs, documents)

scripts/:
  Executable code Claude runs as-is
  (deploy, setup, API calls, data processing)
```
</quick_reference>

<reference_index>
All in `references/`:

**Structure:** recommended-structure.md, skill-structure.md
**Principles:** core-principles.md, be-clear-and-direct.md, use-xml-tags.md
**Patterns:** common-patterns.md, workflows-and-validation.md
**Assets:** using-templates.md, using-scripts.md
**Quality:** skill-checklist.md, evaluation-driven-development.md
**Advanced:** executable-code.md, api-security.md, iteration-and-testing.md
</reference_index>

<workflows_index>
All in `workflows/`:

| Workflow | Purpose |
|----------|---------|
| develop-new-skill.md | Build a skill from scratch |
| develop-domain-expertise-skill.md | Build exhaustive domain knowledge base for build/ |
| audit-skill.md | Analyze skill against best practices |
| verify-skill.md | Check if content is still accurate |
| add-workflow.md | Add a workflow to existing skill |
| add-reference.md | Add a reference to existing skill |
| add-template.md | Add a template to existing skill |
| add-script.md | Add a script to existing skill |
| upgrade-to-router.md | Convert simple skill to router pattern |
| get-guidance.md | Help decide what kind of skill to build |
</workflows_index>

<yaml_requirements>
**⚠️ THE MOST IMPORTANT PART OF ANY SKILL ⚠️**

The name and description determine whether an agent will EVER use your skill. Get these wrong and the skill is useless — it will sit unused while the agent struggles without it.

```yaml
---
name: skill-name          # lowercase-with-hyphens, matches directory
description: "..."        # What it does AND when to use it (address agent as you) - MUST be quoted
---
```

**Description MUST include:**
1. What the skill does (capabilities)
2. **ALL activities covered** — If the skill handles creating, reviewing, updating, and auditing, say so explicitly. Users phrase requests differently ("assess", "check", "audit", "review", "modify", "update", "create", "build"). If any should trigger your skill, include them.
3. When to use it (trigger conditions)
4. Action words like "MUST be loaded before..." or "Use PROACTIVELY when..." for skills that should auto-invoke

**Description MUST NOT include:**
- Implementation details (file structure, tools used, architectural patterns, internal mechanisms)
- How the skill works internally — the description is for matching user intent, not explaining internals
- The description is a **matching surface**, not documentation. Every word should help the agent decide "does this skill apply to what the user asked?" If a word doesn't help with that decision, remove it.

**Critical insight:** If an agent doesn't invoke your skill, it's almost always because the description didn't match how the user phrased their request. Test against multiple phrasings.

**Read references/skill-structure.md for comprehensive guidance on getting this right.**

**Naming conventions (gerund form required):**
- Use gerund (verb ending in -ing): `developing-*`, `processing-*`, `managing-*`, `setting-up-*`
- Use plural nouns: `developing-skills`, `processing-images`, `managing-ads`
- Avoid: vague names (`helper`, `utils`), reserved words (`anthropic-*`, `claude-*`)
</yaml_requirements>

<success_criteria>
A well-structured skill:
- Has valid YAML frontmatter
- Uses pure XML structure (no markdown headings in body)
- Has essential principles inline in SKILL.md
- Routes directly to appropriate workflows based on user intent
- Keeps SKILL.md under 500 lines
- Asks minimal clarifying questions only when truly needed
</success_criteria>
