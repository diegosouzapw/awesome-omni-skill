---
name: skill-create-flow
version: 0.1.0
description: Create new high-quality agent skills with a standalone, repeatable workflow (no dependency on other skills). Use when you want to go from a vague skill idea → narrow scope → extract expert frameworks → write SKILL.md + examples/evals/index/changelog artifacts → self-validate with test prompts.
---

# Skill Create Flow

Build a new skill using a repeatable, high-signal workflow that separates **methodology discovery** from **pack**aging**—without requiring any other skills to be installed or invoked.

## Best for

This workflow excels at creating **procedural agent skills** where the value comes from following a structured methodology rather than raw knowledge retrieval.

**Ideal skill types:**
- **Methodology-based**: Debugging, code reviews, writing specs, systematic problem-solving
- **Decision-heavy**: Architecture decisions, UX design, business strategy, negotiation
- **Multi-step workflows**: Defined sequence of operations with clear deliverables
- **Quality-focused**: Where "excellent vs mediocre" output is distinguishable

**Less ideal for:**
- Pure reference/lookup skills (use knowledge retrieval instead)
- Simple one-shot commands (use direct tool invocation instead)
- Skills requiring external dependencies (this flow produces standalone skills)

## Scope & Outputs

You are producing a *new skill* folder (with its own `SKILL.md` and optional bundled resources). This flow helps you:

- Narrow the intent until “excellent vs mediocre” is judgeable.
- Extract expert frameworks (when needed).
- Generate a production-ready, testable skill spec and artifacts.

**Target artifacts (recommended)**
- `SKILL.md`
- `examples/` (1 tiny example)
- `tests/` (3–5 eval prompts or scenarios)
- `index_entry.json` (if you maintain a skill index)
- `CHANGELOG.md` (optional but helpful for iteration)

## Decision Rules (Pick the Track)

### Track A — Non-technical / judgment-heavy (default if unsure)

Examples: writing, sales, hiring, product decisions, strategy, negotiation.

Use the full flow below (steps 1–6).

### Track B — Technical / objective correctness dominates

Examples: code generation patterns, CLI automation, infrastructure scripts, data pipelines.

Usually shorten Step 2 (“expert frameworks”) and spend more time on Step 5 (“validation”).

## Workflow (Standalone)

### Step 1 — Lock the intent (narrowing)

Converge from broad → specific using this 5-layer funnel:
1) **Domain**: what domain is the skill for?
2) **Context (5W1H)**: who/what/where/when/why/how constraints.
3) **Comparative choice**: pick the closest of 2–3 similar scenarios.
4) **Boundaries (via negativa)**: confirm what is explicitly excluded.
5) **Concrete anchor**: one real recent case with inputs + desired output.

**Deliverable from Step 1**
- A 5–10 line “skill brief”:
  - target user + context
  - inputs the user will provide
  - outputs the skill must produce
  - what’s explicitly out of scope
  - quality bar (“excellent output looks like…”)

### Step 2 — Extract expert frameworks (masters-level)

Produce:
- 1–3 core frameworks/checklists experts use **in this exact scenario**.
- Common failure modes + counterexamples.
- Minimal decision rules (when to do A vs B; when to stop; when to ask for more info).

**Stop condition**
- You can explain the method in a way that is: specific, testable, and not generic advice.

### Step 3 — Draft the trigger contract (frontmatter)

Write a frontmatter `description` that triggers reliably:
- Include 3–6 concrete “when to use” phrases users will actually type.
- Include exclusions if you keep getting false triggers.
- Avoid “does everything”.

**Template**
```yaml
---
name: <skill-name>
description: <What it does>. Use when <trigger 1>, <trigger 2>, or <trigger 3>. Avoid when <non-goal>.
---
```

### Step 4 — Write `SKILL.md` (lean + procedural)

Keep `SKILL.md` short and procedural:
- Decision rules first.
- Output contract (what files/sections are produced) is explicit.
- Put long templates into `references/` if they exceed ~100 lines.

Minimum recommended sections:
- Scope & boundaries
- Inputs (what the user must provide)
- Procedure (step-by-step)
- Decision rules & failure handling
- Output contract
- Test prompts (what you will validate with)

### Step 5 — Create supporting artifacts (examples/evals/index/changelog)

**A) Tiny example (`examples/<name>-example.txt`)**
- ≤30 lines; show the most common invocation and expected output shape.

**B) Evals / test prompts (`tests/evals_<name>.yaml`)**
Include 3–5 scenarios:
- happy path
- missing input
- edge case
- out-of-scope
- safety/constraint case (if relevant)

Minimal template:
```yaml
cases:
  - name: happy_path
    prompt: |
      <a realistic user request that should trigger the skill>
    expect:
      - <must-have output property 1>
      - <must-have output property 2>
```

**C) Index entry (`index_entry.json`)** (only if you keep an index)
```json
{
  "slug": "<skill-name>",
  "name": "<Human title>",
  "summary": "<=160 chars>",
  "entry": "skills/<skill-name>/SKILL.md"
}
```

**D) `CHANGELOG.md`** (optional)
Use it if you expect iteration; otherwise skip.

### Step 6 — Validate via simulations

Dry-run the new skill against your test prompts:
- Does it trigger when it should?
- Does it avoid false triggers?
- Does it produce the promised output contract?

If it fails, iterate:
- tighten frontmatter `description`
- move long content into references
- add/adjust decision rules

## Prompt Templates

### Template 1 — Step 1 (narrowing)

“I want a skill for: <broad goal>. Help me narrow it using this 5-layer funnel: domain → 5W1H → pick closest scenario → boundaries → one real recent case. Ask only the minimum questions (1–3 at a time).”

### Template 2 — Step 2 (expert frameworks)

“For the exact scenario we narrowed to, give me 1–3 expert-level frameworks/checklists, decision rules, and common failure modes. Keep it specific and testable (no generic advice).”

### Template 3 — Step 4/5 (write artifacts)

“Using the skill brief + frameworks, draft:
1) SKILL.md (lean + procedural),
2) 1 tiny example,
3) 3–5 eval prompts,
4) optional index entry + changelog.
Keep templates short; push long templates into references/.”

## Notes (Keep It Lean)

- Prefer **decision rules** over long explanation.
- Prefer **one tiny example** over many medium ones.
- If you feel tempted to add a long “how skills work” section, don’t—keep the flow operational.
