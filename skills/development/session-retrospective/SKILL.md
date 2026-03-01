---
name: session-retrospective
description: Review the current session to extract learnings and propose improvements to CLAUDE.md files and skills. Run at the end of a session or when asked to reflect on what was learned. Routes project-specific learnings to the project CLAUDE.md, universal patterns to the global ~/.claude/CLAUDE.md, and repeated workflows to skill creation/modification proposals. Triggers include "retrospective", "session retro", "振り返って", "何を学んだ？", "セッションの学び", "what did we learn?", "improve from this session".
---

# Session Retrospective

Review the current session to extract learnings and propose improvements to CLAUDE.md files and skills, making Claude more autonomous with each session.

## Overview

This skill analyzes the conversation history to identify learnings that should be codified into:
- **Project CLAUDE.md** — Project-specific patterns, commands, and conventions
- **Global ~/.claude/CLAUDE.md** — Universal coding styles, Claude behaviors, and cross-project patterns
- **Skills** — Multi-step workflows worth automating (the most valuable output)

Skill proposals are the highest-value output of a retrospective. A single well-designed skill saves more future time than a dozen CLAUDE.md entries. Actively hunt for skill opportunities — do not default to CLAUDE.md when a skill would be more appropriate.

Unlike `/revise-claude-md` which focuses on missing context, this skill provides broader analysis including corrected approaches, skill opportunity detection, and skill improvement opportunities.

## Quick Start

```
/session-retrospective
```

No arguments needed. The skill analyzes the current session context automatically.

## Workflow

### Phase 1: Analyze Session

Gather full session data and reflect on it:

1. **Extract full transcript history** (Compact で失われた内容も含む完全な記録):
   ```bash
   extract-session-history.ts
   ```
   stdout に出力されたファイルパスを Read ツールで読み込む。

2. **Review extracted history** focusing on:
   - Tasks performed and their outcomes
   - Errors encountered and how they were resolved
   - Questions asked to the user (signals missing context)
   - User corrections to Claude's approach
   - Repeated patterns of work
   - Tool usage patterns (Tool Usage Summary セクション参照)
   - Compact boundaries (セッション内のフェーズ遷移を示す)

3. **Git activity** (if in a git repository):
   ```bash
   git log --oneline -20
   git diff --stat HEAD~5..HEAD
   ```

4. **Current context files**:
   - Read project CLAUDE.md (if exists)
   - Read global ~/.claude/CLAUDE.md
   - List existing skills in ~/.claude/skills/

### Phase 2: Categorize Learnings

Classify each learning into categories. See [references/learning-categories.md](references/learning-categories.md) for detailed definitions and examples.

**Summary of categories:**

1. **Missing Context** — Information Claude needed but did not have
   - Example: "This project uses pnpm, not npm"
   - Example: "Always run nix flake check before darwin-rebuild"

2. **Corrected Approaches** — User corrections to Claude's behavior
   - Example: "User prefers Japanese commit messages"
   - Example: "Use `gh` for GitHub URLs, not WebFetch"

3. **Repeated Workflows** — Multi-step procedures performed multiple times
   - Example: "Check CI → read logs → fix → push → wait (repeated 3 times)"
   - Example: "Every nix change: edit → nix flake check → darwin-rebuild switch"

4. **Tool/Library Knowledge** — Discoveries about specific tools or APIs
   - Example: "ast-grep requires stopBy: end for relational rules"
   - Example: "Chrome DevTools MCP snapshot should be taken before interaction"

5. **Preference Patterns** — User style or preference observations
   - Example: "User prefers concise output, dislikes verbose explanations"
   - Example: "User wants draft PRs with English descriptions"

### Phase 2.5: Skill Opportunity Scan

After categorizing learnings, perform a dedicated scan for skill opportunities across ALL categories — not just "Repeated Workflows". See [references/skill-opportunity-detection.md](references/skill-opportunity-detection.md) for the full detection framework.

**For every learning in every category**, apply these quick checks:

1. **The /invoke test**: "Would the user type `/skill-name` for this?"
2. **The orchestration test**: "Does this involve 3+ steps with tool chaining?"
3. **The teaching test**: "Did the user describe a multi-step process?"
4. **The cross-session test**: "Did the user signal this is a recurring task?"
5. **The ecosystem test**: "Does this resemble an existing skill's structure?"
6. **The systematization test**: "Did this session consume external knowledge AND encode it into a reusable tool? (Requires 2+ indicators from Signal 6 in skill-opportunity-detection.md)"

If ANY check passes, flag the learning as a skill candidate and carry it forward to Phase 3 routing with a skill proposal bias.

**Explicit requirement**: Consider at least one skill proposal per retrospective. If no learnings pass the checks above, document why in the results ("No skill opportunities detected because: [reason]"). This forces active evaluation rather than passive defaulting to CLAUDE.md.

**Scan existing skills for modification opportunities**:
```bash
ls ~/.claude/skills/
```
For each skill that was used or relevant to the session, check:
- Was the skill missing information that was discovered during the session?
- Did the workflow deviate from what the skill prescribed?
- Would a new reference file improve the skill?

#### Generalization Check

For each identified learning, also ask:
> "Is this specific to a narrow context (tool X, environment Y, one-off situation Z),
> or does it reflect a **broader methodological/behavioral principle**?"

- **If broader principle exists**: Document the **general principle** as the primary entry.
  Include the specific instance as a concrete example, not as the title/framing.
  Route to `~/.claude/CLAUDE.md` with the general principle.

- **If genuinely narrow**: Keep specific, but flag it:
  "Does this narrow symptom suggest a general principle that was missed?"

**Anti-pattern to avoid (from real session):**
- ✗ Extracted: "non-interactive environment debugging tip" (env-specific framing)
- ✓ Should extract: "investigation plans must include direct observation means" (general principle)
  with "non-interactive env" as one concrete example

**Artifact Pipeline Check**: Beyond generalizing facts, also ask:
> "Did this session produce a reusable artifact? If so, is the *artifact-creation pipeline* itself generalizable across domains?"

If yes, recommend the appropriate mechanism using this routing:

```
Is the methodology a repeatable workflow the user would invoke by name?
  YES → Skill (standard). Use skill-creator to build it.
  NO → Is it background knowledge that should always be available?
    YES → Skill (user-invocable: false, Reference content).
    NO → Does it need constrained tool access (read-only analysis)?
      YES → Custom Agent (agents/*.md). Use context: fork + agent: from skills.
      NO → Does it augment an existing skill's evaluation criteria?
        YES → Reference file in existing skill's references/.
        NO → Skill (standard) as default. Most flexible option.
```

Default to Skill (standard) when uncertain — it matches the user's preference for skill-based workflows.

**Example:**
- Artifact: Evaluation agent derived from Anthropic PDF (specific instance)
- Pipeline: "Methodology document → extract criteria → create evaluation tool" (generalizable)
- Recommended: Skill (standard) — e.g., `/review-accessibility [url]`
- Alternative: Custom Agent if read-only constraint is needed

### Phase 3: Route Proposals

Determine where each learning belongs using routing logic. See [references/routing-logic.md](references/routing-logic.md) for the full decision tree.

**Quick routing summary:**

- **Project-specific** (contains project paths, build commands, framework-specific details)
  → Project CLAUDE.md

- **Universal/cross-project** (coding style, general tool usage, Claude's behavior)
  → ~/.claude/CLAUDE.md

- **Skill opportunity** (detected via Phase 2.5 scan — complex workflow, user-taught
  process, cross-session repetition signal, or tool orchestration pattern)
  → New skill or skill modification proposal

- **Tool-specific knowledge for existing skill**
  → Skill modification proposal

### Phase 4: Draft Proposals

For each routed learning, draft a concrete proposal:

**CLAUDE.md additions:**
- Format: One line per concept (consistent with `/revise-claude-md`)
- Show exact placement (after which section heading)
- Use diff format for clarity
- Example:
  ```diff
  ## Development Commands
  + nix flake check - Validate Nix syntax before applying changes
  ```

**New skill proposals** (use template from [references/skill-opportunity-detection.md](references/skill-opportunity-detection.md)):
- Proposed name and invocation example
- When to use (trigger scenario)
- **Why-not-CLAUDE.md justification** (mandatory — why this needs to be a skill)
- Workflow outline (numbered steps with tools used)
- Parameters (what would be parameterized)
- Similar existing skill (for calibration)
- Estimated complexity (Simple / Medium / Complex)

A skill proposal without a why-not-CLAUDE.md justification is incomplete.

**Skill modification proposals:**
- Show before/after diff of changes
- Explain rationale for the modification
- Example:
  ```diff
  ## Quick Start

  - curl -s https://api.example.com/health
  + curl -sf https://api.example.com/health - Add -f flag to fail on HTTP errors
  ```

### Phase 5: Present and Apply

Present all proposals grouped by target:

```
## Session Retrospective Results

### Project CLAUDE.md Proposals (N items)
[numbered proposals with diffs]

### Global CLAUDE.md Proposals (N items)
[numbered proposals with diffs]

### Skill Proposals (N items)
[new skills and modifications]

---
Which proposals would you like to apply?
(all / specific numbers like 1,3,5 / none)
```

**After user approval:**
- Apply CLAUDE.md changes with Edit tool
- Create new skills (use skill-creator toolchain or direct file creation)
- Apply skill modifications with Edit tool
- Report what was applied

## Guidelines

### What NOT to Propose

- Information Claude already knows (well-known concepts, standard library usage)
- Temporary or one-off decisions that don't generalize
- Information already present in the target CLAUDE.md or skill
- Overly specific instructions that reduce flexibility

### Proposal Quality

- Each proposal should be actionable (specific text to add, not vague suggestions)
- Each proposal should justify its token cost (high signal-to-noise ratio)
- Prefer additions that prevent future mistakes over documenting facts
- Keep proposals concise — one line per concept for CLAUDE.md entries

### Respecting Existing Content

- Read existing CLAUDE.md files before proposing additions
- Do not duplicate existing entries
- Match the style and language of the existing file (Japanese or English)
- Place additions in the appropriate section (do not append everything at the end)

### Language Matching

When proposing CLAUDE.md additions:
- Detect the primary language of the target file
- Write proposals in that language
- For mixed-language files, match the language of the relevant section

## Related Skills

- **/revise-claude-md** — Focused CLAUDE.md updates based on missing context (narrower scope)
- **skill-improver** — Evaluates and improves existing skill quality
- **skill-creator** — Creates new skills from scratch
- **skill-tester** — Validates skill behavior after modifications
