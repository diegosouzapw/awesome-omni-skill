---
name: pattern-extraction
model: reasoning
description: Extract design systems, architecture patterns, and methodology from codebases into reusable skills and documentation. Use when analyzing a project to capture patterns, creating skills from existing code, extracting design tokens, or documenting how a project was built. Triggers on "extract patterns", "extract from this repo", "analyze this codebase", "create skills from this project", "extract design system".
---

# Pattern Extraction

Extract reusable patterns, skills, and methodology documentation from existing codebases.


## Installation

### OpenClaw / Moltbot / Clawbot

```bash
npx clawhub@latest install extraction
```


---

## Before Starting

**MANDATORY**: Read these reference files based on what you're extracting:

| Extracting | Read First |
|------------|------------|
| Any extraction | [`methodology-values.md`](references/methodology-values.md) — priority order and what to look for |
| Specific categories | [`extraction-categories.md`](references/extraction-categories.md) — detailed patterns per category |
| Generating skills | [`skill-quality-criteria.md`](references/skill-quality-criteria.md) — quality checklist |

---

## Extraction Process

### Phase 1: Discovery

Analyze the project to understand what exists.

**Scan for project structure:**
```
- Root directory layout
- Key config files (package.json, tailwind.config.*, etc.)
- Documentation (README, docs/, etc.)
- Source organization (src/, app/, components/, etc.)
```

**Identify tech stack:**
| Indicator | Technology |
|-----------|------------|
| `package.json` with react | React |
| `tailwind.config.*` | Tailwind CSS |
| `components.json` | shadcn/ui |
| `go.mod` | Go |
| `Dockerfile` | Docker |
| `k8s/` or `.yaml` manifests | Kubernetes |
| `turbo.json` | Turborepo |
| `Makefile` | Make automation |

**Look for design system signals:**
- Custom Tailwind config (not defaults)
- CSS variables / custom properties
- Theme files
- Design documentation
- Mood boards or reference lists

**Capture key findings:**
- What's the tech stack?
- What's the folder structure?
- Is there a documented design direction?
- What workflows exist (Makefile, scripts)?

---

### Phase 2: Categorization

Map discoveries to extraction categories, prioritized:

**Priority order:**
1. **Design Systems** — Color tokens, typography, spacing, motion, aesthetic documentation
2. **UI Patterns** — Component organization, layouts, interactions
3. **Architecture** — Folder structure, data flow, API patterns
4. **Workflows** — Build, dev, deploy, CI/CD
5. **Domain-Specific** — Patterns unique to this application type

**For each category found, note:**
- What specific patterns exist?
- Where are they defined? (file paths)
- Are they documented? (comments, docs)
- Are they worth extracting? (used in multiple places, well-designed)

**Filter by value:**
| Extract | Skip |
|---------|------|
| Patterns used across multiple components | One-off solutions |
| Customized configs with intention | Default configurations |
| Documented design decisions | Arbitrary choices |
| Reusable infrastructure | Project-specific hacks |

---

### Phase 3: Extraction

For each valuable pattern, generate outputs.

**Design Systems → Design System Doc + Skill**

1. Read the Tailwind config, CSS files, theme files
2. Extract actual token values (colors, typography, spacing)
3. Document the aesthetic direction
4. Create:
   - `docs/extracted/[project]-design-system.md` using [`design-system.md`](references/output-templates/design-system.md) template
   - `ai/skills/[project]-design-system/SKILL.md` if patterns are reusable

**Architecture → Methodology Doc**

1. Document folder structure with reasoning
2. Capture data flow patterns
3. Note key technical decisions
4. Create `docs/extracted/[project]-summary.md` using [`project-summary.md`](references/output-templates/project-summary.md) template

**Patterns → Skills**

For each pattern worth a skill:

1. Load [`skill-quality-criteria.md`](references/skill-quality-criteria.md)
2. Use [`skill-template.md`](references/output-templates/skill-template.md) template
3. Verify the quality checklist:
   - Description has WHAT, WHEN, KEYWORDS
   - No explanations of basics Claude knows
   - Has specific NEVER list
   - < 300 lines ideal
4. Create `ai/skills/[project]-[pattern]/SKILL.md`

---

### Phase 4: Validation

Before writing output, validate extracted content.

**For each skill, verify:**
- [ ] Description has WHAT, WHEN, and trigger KEYWORDS
- [ ] >70% expert knowledge (not in base Claude model)
- [ ] <300 lines (max 500)
- [ ] Has "When to Use" section with clear triggers
- [ ] Has code examples (if applicable)
- [ ] Has NEVER Do section with anti-patterns
- [ ] Project-agnostic (no hardcoded project names)

**For documentation, verify:**
- [ ] Actual values extracted (not placeholders)
- [ ] Templates fully filled out
- [ ] Aesthetic direction documented (for design systems)
- [ ] File paths are correct

**Conflict detection:**
Before creating a new skill, check if similar skills exist:

```bash
# Check existing skills in the target repo
ls ai/skills/*/
```

| Situation | Action |
|-----------|--------|
| Similar skill exists | Enhance existing skill instead |
| Overlapping patterns | Note overlap, may merge in refinement |
| Unique pattern | Proceed with new skill |

---

### Phase 5: Output

Write extracted content to target locations.

**Methodology Documentation:**
```
docs/extracted/
├── [project]-summary.md       # Overall methodology
├── [project]-design-system.md # Design tokens and aesthetic
└── [project]-architecture.md  # Code patterns (if complex)
```

**Skills:**
```
ai/skills/
└── [project]-[category]/
    ├── SKILL.md
    └── references/  # (if needed for detailed content)
```

**Create docs/extracted/ directory if it doesn't exist.**

---

## Extraction Focus Areas

### Design System Extraction (Highest Priority)

When a project has intentional design work, extract thoroughly:

**Must capture:**
- Color palette (primary, secondary, accent, semantic)
- Typography (fonts, scale, weights)
- Spacing scale
- Motion/animation patterns
- The "vibe" or aesthetic direction

**Look in:**
- `tailwind.config.js` / `tailwind.config.ts`
- `globals.css` / `app.css` / root CSS files
- `theme.ts` / `theme.js`
- Any design documentation

**Generate:**
1. Design system documentation with actual values
2. Skill capturing the aesthetic philosophy (if distinctive)

### Workflow Extraction

**Look for:**
- Makefile targets
- package.json scripts
- Docker configurations
- CI/CD workflows

**Extract:**
- Dev setup commands
- Build processes
- Deployment patterns

---

## Error Handling

| Situation | Resolution |
|-----------|------------|
| No patterns found | Create project summary only; document why extraction failed |
| Pattern too project-specific | Skip or generalize by removing project names |
| Incomplete pattern | Extract what exists, note gaps in skill |
| Quality criteria not met | Revise skill or skip pattern |
| Similar skill already exists | Update existing skill instead of creating new |
| Can't find source files | Note in extraction log, skip that category |

**When extraction fails partially:**
1. Complete what can be extracted
2. Document gaps in the project summary
3. Note "Incomplete extraction" in output
4. Suggest what additional information would be needed

---

## NEVER Do

- **NEVER extract default configurations** — Only extract customized, intentional patterns
- **NEVER create skills for basic concepts** — Claude already knows React, Tailwind basics
- **NEVER skip the aesthetic** — Design philosophy is highest priority
- **NEVER generate skills > 500 lines** — Use references/ for detailed content
- **NEVER create skills without good descriptions** — Description determines if skill activates
- **NEVER extract one-off solutions** — Focus on patterns used in multiple places
- **NEVER skip validation phase** — Quality check before writing output
- **NEVER leave project names in skills** — Make patterns project-agnostic
- **NEVER create duplicate skills** — Check for existing similar skills first

---

## Quality Check Before Finishing

- [ ] Design system captured (if one exists)?
- [ ] Methodology summary created?
- [ ] Skills have proper descriptions (WHAT, WHEN, KEYWORDS)?
- [ ] Skills pass the expert knowledge test?
- [ ] Anti-patterns documented in skills?
- [ ] Output files created in correct locations?

---

## After Extraction: Staging for Refinement

If you're extracting to later consolidate patterns across multiple projects:

**Copy results to the skills toolkit repo for staging:**

```bash
# From this project, copy to the skills repo staging area
cp -r ai/skills/[project]-* /path/to/skills-repo/ai/staging/skills/
cp -r docs/extracted/* /path/to/skills-repo/ai/staging/docs/
```

**Staging folder structure:**
```
ai/staging/
├── skills/           # Extracted skills from multiple projects
│   ├── project-a-design-system/
│   ├── project-b-ui-patterns/
│   └── ...
└── docs/             # Extracted methodology docs
    ├── project-a-summary.md
    ├── project-b-design-system.md
    └── ...
```

**After staging content from multiple projects:**
- Say "refine staged content" or "consolidate staged skills"
- The refinement process will:
  - Identify patterns across projects
  - Consolidate into project-agnostic skills
  - Update methodology docs with insights
  - Promote refined skills to active locations

---

## Related Skills

- **Agent:** [`ai/agents/extraction/`](../../agents/extraction/) — Autonomous extraction workflow
- **Command:** [`/extract-patterns`](../commands/extraction/extract-patterns.md) — Quick extraction command
- **Next step:** [`ai/skills/refinement/`](../refinement/) — Consolidate extracted patterns
- **Quality criteria:** [`references/skill-quality-criteria.md`](references/skill-quality-criteria.md)
