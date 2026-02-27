---
name: cpm:review
description: Adversarial review of epic docs and stories. Agents from the party roster examine planning artifacts through their professional lens, challenging assumptions, spotting gaps, and flagging risks. Triggers on "/cpm:review".
---

# Adversarial Review

Run a critical review of an epic doc or a specific story using the party agent roster. Each persona examines the artifact through their professional lens — challenging assumptions, spotting gaps, and flagging risks. Produces a structured review document with severity-tagged findings and an optional autofix that generates remediation tasks.

## Input

Check for input in this order:

1. If `$ARGUMENTS` references a file path (e.g. `docs/epics/01-epic-auth.md`), use that as the epic doc. If it also includes a story number (e.g. `docs/epics/01-epic-auth.md 2`), note that as the target story for a story-level review.
2. If `$ARGUMENTS` is a review file path from a previous `/cpm:review` run (e.g. `docs/reviews/01-review-auth.md`), read its `**Source**:` field to resolve the original epic doc. Offer to re-review or continue from the previous findings.
3. If no path given, look for the most recent `docs/epics/*-epic-*.md` file and ask the user to confirm.
4. If no epic docs exist, tell the user there's nothing to review and stop.

After resolving the epic doc, read it with the Read tool. Then ask the user about review scope:

- **Review the entire epic** — All stories and tasks will be examined
- **Review a specific story** — User selects which story to focus on

For story-level review, present the list of stories from the epic doc and let the user choose. If a story number was provided in `$ARGUMENTS`, skip the selection and use that story directly.

## Roster Loading

Load the agent roster at session start. Check for a project-level override first:

1. **Project override**: Read `docs/agents/roster.yaml` in the current project directory. If it exists, use it as the complete roster (no merging with defaults).
2. **Plugin default**: If no project override exists, read the plugin's `agents/roster.yaml` (located in the same plugin directory as this skill, at `../../agents/roster.yaml` relative to this file).

If neither file can be found, proceed without agent personas — fall back to a single-perspective review and note the limitation to the user.

After loading, briefly introduce the review team. Don't list all agents — just confirm the roster is loaded and how many agents are available:

```
Review roster loaded: {N} agents available. I'll select the most relevant reviewers based on the content.
```

## Library Check

After roster loading and before starting the review, check the project library for reference documents:

1. **Glob** `docs/library/*.md`. If no files found or directory doesn't exist, skip silently.
2. **Read front-matter** of each file found (the YAML block between `---` delimiters, typically the first ~10 lines). Filter to documents whose `scope` array includes `review` or `all`.
3. **Report to user**: "Found {N} library documents relevant to this review: {titles}. Agents will reference these during their review." If none match the scope filter, skip silently.
4. **Deep-read selectively** during the review step when an agent's review would benefit from referencing library content — e.g. an architect referencing architecture docs when reviewing structural decisions, or a developer citing coding standards when reviewing implementation tasks.

**Graceful degradation**: If any library document has malformed or missing front-matter, fall back to using the filename as context. Never block the review due to a malformed library document.

**Compaction resilience**: Include library scan results (files found, scope matches) in the progress file so post-compaction continuation doesn't re-scan.

### Template Hint (Startup)

After the Library Check and before Agent Selection, display:

> Output format is fixed (used by downstream skills). Run `/cpm:templates preview review` to see the format.

## Agent Selection

Select agents dynamically based on the review scope and content. The goal is to match reviewer expertise to what's being reviewed.

### Selection Rules

- **Story-level review**: Pick **2-3 agents** whose expertise is most relevant to the story's content.
- **Epic-level review**: Pick **3-4 agents** covering the broadest range of concerns across all stories.

### Selection Criteria

For each story or epic, assess the content and match agents by relevance:

- **Technical implementation** (API endpoints, data models, algorithms) → Developer, Architect
- **User-facing features** (UI flows, forms, notifications) → UX Designer, Product Manager
- **Infrastructure and deployment** (CI/CD, environments, scaling) → DevOps Engineer, Architect
- **Data handling and security** (auth, encryption, PII) → QA Engineer, Architect, Developer
- **Process and scope** (story sizing, dependency chains, delivery risk) → Scrum Master, Product Manager
- **Documentation and naming** (terminology, clarity, consistency) → Technical Writer

Always include at least one agent who will challenge the *business value* of the work (PM or Scrum Master) and at least one who will challenge the *technical approach* (Developer or Architect). This ensures every review has both a "should we?" and a "can we?" perspective.

### Named Agent Format

Each agent's review contribution uses the format:

```
{icon} **{displayName}**: {review content}
```

This matches the party mode format for consistency across CPM skills.

## Process

**State tracking**: Before starting Step 1, create the progress file (see State Management below). Each step below ends with a mandatory progress file update — do not skip it. After saving the final review file, delete the progress file.

### Step 1: Analyse the Artifact

Read the target artifact (full epic or specific story) and build a structured understanding:

- **For epic-level review**: Parse all `##` story headings, their acceptance criteria, tasks, dependencies, and status. Note the epic's overall structure — how many stories, dependency chains, completion state.
- **For story-level review**: Parse the target `##` story heading, its acceptance criteria, all `###` tasks, and any `**Blocked by**` dependencies. Also note the story's position within the broader epic for context.

**Spec discovery**: If the epic doc has a `**Source spec**:` field, read the referenced spec. This enables spec compliance review — checking whether the epic's stories cover the spec's requirements. If no spec exists, skip silently.

**ADR discovery**: **Glob** `docs/architecture/[0-9]*-adr-*.md`. If ADRs exist, read them. This enables ADR compliance review — checking whether stories respect architectural decisions. If no ADRs exist, skip silently.

Present a brief summary to the user: what's being reviewed, how many stories/tasks, current status. If a spec or ADRs were found, note that they'll be used as review context.

**Update progress file now** — write the full `.cpm-progress-{session_id}.md` with Step 1 summary before continuing.

### Step 2: Conduct Adversarial Review

This is the core of the skill. Each selected agent examines the artifact through their professional lens and produces findings.

#### Review Dimensions

Agents should look for issues across these concern types:

- **Unclear Requirements** — Acceptance criteria that are vague, ambiguous, or untestable. Stories where the definition of done is subjective.
- **Missing Acceptance Criteria** — Important outcomes that aren't captured. Edge cases, error states, or integration points that should have criteria but don't.
- **Hidden Complexity** — Tasks that look simple but have non-obvious implementation challenges. Underestimated work that will cause delays.
- **Architectural Risks** — Structural decisions that could cause problems at scale, create tight coupling, or conflict with existing patterns in the codebase.
- **Testability Concerns** — Stories or tasks that would be difficult to verify. Acceptance criteria that can't be automatically checked.
- **Scope Creep** — Stories that try to do too much, tasks that go beyond their parent story's acceptance criteria, or work that belongs in a different epic.
- **Dependency Risks** — Missing or incorrect `**Blocked by**` declarations. Circular dependencies. Stories that should be sequenced but aren't.
- **Spec Compliance** — Stories or acceptance criteria that don't align with the spec's requirements. Must-have requirements that aren't covered by any story. Only applicable when a source spec was discovered in Step 1.
- **ADR Compliance** — Stories or tasks that contradict or ignore architectural decisions from existing ADRs. Implementation approaches that conflict with the rationale or constraints documented in ADRs. Only applicable when ADRs were discovered in Step 1.
- **Missing Test Coverage** — Stories with acceptance criteria tagged `[unit]`, `[integration]`, or `[feature]` that lack a corresponding testing task. Stories where criteria warrant automated testing but have no test approach tags. Only applicable when the source spec has a testing strategy or when stories carry test approach tags.

Not every agent reviews every dimension. Each agent focuses on what they'd naturally notice given their role:

- **Product Manager**: Unclear requirements, scope creep, missing user value, spec compliance
- **Architect**: Architectural risks, hidden complexity, dependency risks, ADR compliance
- **Developer**: Hidden complexity, testability concerns, missing acceptance criteria, ADR compliance, missing test coverage
- **UX Designer**: Unclear requirements from a user perspective, missing edge cases in user flows
- **QA Engineer**: Testability concerns, missing acceptance criteria, missing error states, missing test coverage
- **DevOps Engineer**: Architectural risks (deployment, scaling), dependency risks
- **Technical Writer**: Unclear requirements (naming, terminology), scope creep (documentation gaps)
- **Scrum Master**: Scope creep, dependency risks, story sizing concerns

#### Severity Classification

Each finding must be tagged with exactly one severity level:

- **Critical** — Blocks execution. The story or task cannot be implemented correctly as written. Examples: contradictory acceptance criteria, impossible dependency chain, missing essential requirement.
- **Warning** — Likely to cause problems. Implementation can proceed but will probably hit issues. Examples: vague acceptance criteria that different developers would interpret differently, hidden complexity that makes time estimates unreliable.
- **Suggestion** — Improvement opportunity. Not blocking and may not cause problems, but addressing it would improve quality. Examples: acceptance criteria that could be more specific, a task that could be split for clarity.

#### Review Execution

For each selected agent, in turn:

1. Review the artifact through the agent's professional lens.
2. Produce 2-5 findings (not a comprehensive audit — focus on the most impactful observations).
3. Tag each finding with a concern type and severity.
4. Reference specific stories, tasks, or acceptance criteria by number.
5. If library documents were loaded, reference relevant standards or constraints where they apply.

Format each finding as:

```
{icon} **{displayName}** [{severity}]: {finding}
→ {story/task reference}: {specific issue and why it matters}
```

After all agents have reviewed, present the findings to the user grouped by concern type (not by agent). Within each concern type, order by severity (critical first, then warning, then suggestion).

**Update progress file now** — write the full `.cpm-progress-{session_id}.md` with Step 2 summary (concern types found, finding counts by severity) before continuing.

### Step 3: Write Review File

Save the review to `docs/reviews/{nn}-review-{slug}.md`. Create the `docs/reviews/` directory if it doesn't exist.

- `{nn}` is a zero-padded auto-incrementing number. Use the Glob tool to list existing `docs/reviews/[0-9]*-review-*.md` files, find the highest number, and increment by 1. If none exist, start at `01`.
- `{slug}` is derived from the epic doc name (e.g. epic doc `01-epic-auth.md` produces review slug `auth`). For story-level reviews, append the story number: `auth-s2`.

Format:

```markdown
# Review: {Epic or Story Title}

**Date**: {today's date}
**Source**: {path to epic doc}
**Scope**: {Epic | Story N: {Story Title}}
**Agents**: {comma-separated list of agent displayNames who participated}
**Findings**: {total count} ({critical count} critical, {warning count} warnings, {suggestion count} suggestions)

## Summary

{2-3 sentence overview of the review — overall assessment, most significant concerns, and whether the artifact is ready for execution or needs attention.}

## Findings

### Unclear Requirements
{findings in this category, ordered by severity}

- **[{severity}]** {icon} **{agent}**: {finding}
  → {story/task reference}: {specific issue}

### Missing Acceptance Criteria
{findings in this category}

### Hidden Complexity
{findings in this category}

### Architectural Risks
{findings in this category}

### Testability Concerns
{findings in this category}

### Scope Creep
{findings in this category}

### Dependency Risks
{findings in this category}

### Spec Compliance
{findings in this category — only if a source spec was discovered}

### ADR Compliance
{findings in this category — only if ADRs were discovered}

### Missing Test Coverage
{findings in this category — only if test approach tags or testing strategy present}
```

Only include concern type sections that have findings. If a category has no findings, omit the heading entirely.

Step 4 may append a `## Remediation` section to this file — see Autofix below.

Tell the user the review file path after saving.

**Update progress file now** — write the full `.cpm-progress-{session_id}.md` with Step 3 summary before continuing.

### Step 4: Autofix

After writing the review file, offer the user the option to generate remediation tasks from the findings. This step is optional — the user can skip it entirely.

#### Decision Gate

If the review produced **no critical or warning findings**, skip autofix entirely — there's nothing actionable to fix. Inform the user: "No critical or warning findings — skipping autofix." Use the Edit tool to append a `## Remediation` section to the review file:

```markdown

## Remediation

No critical or warning findings — autofix not applicable.
```

If critical or warning findings exist, present the autofix option using AskUserQuestion:

- **Generate fix tasks** — Create remediation items from critical and warning findings
- **Skip autofix** — Proceed to pipeline handoff without generating tasks

If the user chooses **Skip autofix**, use the Edit tool to append a `## Remediation` section to the review file before proceeding to Step 5:

```markdown

## Remediation

Autofix offered but declined. {N} critical and {N} warning findings remain unaddressed.
```

Suggestions are informational and never generate fix tasks, regardless of the user's choice.

#### Adaptive Path Selection

When the user opts into autofix, determine the target based on the epic's status:

1. **Read the epic doc's top-level `**Status**:` field.**
2. **If `Pending` or `In Progress`** (active epic): Use the **epic amendment path** — append a remediation story to the epic doc.
3. **If `Complete`** or if the epic doc doesn't exist: Use the **standalone task path** — create Claude Code tasks directly.

#### Epic Amendment Path (Active Epics)

When the epic has pending work, append a remediation story to the epic doc. This keeps fix tasks within the planning artifact so `cpm:do` picks them up naturally.

1. **Determine the next story number**: Read the epic doc, find the highest `**Story**: {N}` number, and increment by 1.

2. **Build the remediation story**: Create a new `##` story section from the critical and warning findings. Each finding becomes a task within the story.

   Format:

   ```markdown
   ## Address review findings
   **Story**: {next N}
   **Status**: Pending
   **Blocked by**: —

   **Acceptance Criteria**:
   - Each critical and warning finding from the review has been addressed
   - Changes do not break existing acceptance criteria on other stories

   ### Fix: {finding summary}
   **Task**: {N.1}
   **Description**: [{severity}] {full finding text with story/task reference}
   **Status**: Pending

   ### Fix: {finding summary}
   **Task**: {N.2}
   **Description**: [{severity}] {full finding text with story/task reference}
   **Status**: Pending

   ---
   ```

3. **Append using Edit tool**: Use the Edit tool to append the new story section to the end of the epic doc (before any `## Lessons` section if one exists). **Never use Write** — Edit preserves the existing content.

4. **Report**: Tell the user what was added — story number, task count, and which findings were converted.

5. **Update review file**: Use the Edit tool to append a `## Remediation` section to the review file:

   ```markdown

   ## Remediation

   **Path**: Epic amendment
   **Target**: {epic doc path}
   **Story**: {story number} — Address review findings

   | # | Finding | Severity | Task |
   |---|---------|----------|------|
   | 1 | {finding summary} | {severity} | {N.1} |
   | 2 | {finding summary} | {severity} | {N.2} |
   ```

   Include one row per finding that generated a task. The Task column references the task number within the remediation story.

#### Standalone Task Path (Complete or Missing Epics)

When the epic is complete or no epic doc exists, create Claude Code tasks directly. This avoids reopening a completed planning artifact.

1. **Build tasks from findings**: For each critical and warning finding, create a Claude Code task:

   ```
   TaskCreate:
     subject: "Fix: {finding summary}"
     description: "[{severity}] {full finding text}\n\nSource review: {review file path}\nOriginal artifact: {epic doc path}\n{story/task reference}"
     activeForm: "Fixing: {finding summary}"
   ```

2. **Set dependencies**: If findings have a logical order (e.g. a critical finding should be addressed before a related warning), use TaskUpdate with `addBlockedBy` to sequence them. Otherwise, leave all tasks independent.

3. **Report**: Tell the user how many tasks were created and list them briefly.

4. **Update review file**: Use the Edit tool to append a `## Remediation` section to the review file:

   ```markdown

   ## Remediation

   **Path**: Standalone tasks (epic complete)
   **Tasks created**: {count}

   | # | Finding | Severity | Task |
   |---|---------|----------|------|
   | 1 | {finding summary} | {severity} | {task subject} |
   | 2 | {finding summary} | {severity} | {task subject} |
   ```

   Include one row per finding that generated a task. The Task column uses the Claude Code task subject.

**Update progress file now** — write the full `.cpm-progress-{session_id}.md` with Step 4 summary before continuing.

### Step 5: Pipeline Handoff

After autofix (or after skipping it), offer the user options for what to do next. The options adapt based on the epic's status.

#### Pre-Execution Epics (Status: Pending or In Progress)

Use AskUserQuestion with these options:

- **Continue to /cpm:pivot** — Use the review findings to amend the epic doc or its upstream spec. Pass both the review file path and the epic doc path as context.
- **Continue to /cpm:do** — Proceed to task execution. Pass the epic doc path as context. The review findings serve as informational background.
- **Just exit** — End the session, no handoff.

#### Post-Execution Epics (Status: Complete)

Use AskUserQuestion with these options:

- **Continue to /cpm:retro** — Feed review findings into a retrospective. Pass the review file path and epic doc path as context.
- **Continue to /cpm:pivot** — Use the review findings to amend the epic doc or upstream artifacts. Pass both paths as context.
- **Continue to /cpm:discover** — Start a new planning cycle informed by the review. Pass the review file path as context.
- **Continue to /cpm:spec** — Build requirements informed by the review. Pass the review file path as context.
- **Just exit** — End the session, no handoff.

#### Handoff Execution

If the user chooses a pipeline skill, pass the relevant file paths as the input context for that skill. The review file path becomes the `$ARGUMENTS` equivalent — the next skill should treat it as its starting context.

Delete the progress file after handoff or exit.

## State Management

Maintain `docs/plans/.cpm-progress-{session_id}.md` throughout the session for compaction resilience. This allows seamless continuation if context compaction fires mid-conversation.

**Path resolution**: All paths in this skill are relative to the current Claude Code session's working directory. When calling Write, Glob, Read, or any file tool, construct the absolute path by prepending the session's primary working directory. Never write to a different project's directory or reuse paths from other sessions.

**Session ID**: The `{session_id}` in the filename comes from `CPM_SESSION_ID` — a unique identifier for the current Claude Code session, injected into context by the CPM hooks on startup and after compaction. Use this value verbatim when constructing the progress file path. If `CPM_SESSION_ID` is not present in context (e.g. hooks not installed), fall back to `.cpm-progress.md` (no session suffix) for backwards compatibility.

**Resume adoption**: When a session is resumed (`--resume`) or context is cleared (`/clear`), `CPM_SESSION_ID` changes to a new value while the old progress file remains on disk. The hooks inject all existing progress files into context — if one matches this skill's `**Skill**:` field but has a different session ID in its filename, adopt it:
1. Read the old file's contents (already visible in context from hook injection).
2. Write a new file at `docs/plans/.cpm-progress-{current_session_id}.md` with the same contents.
3. After the Write confirms success, delete the old file: `rm docs/plans/.cpm-progress-{old_session_id}.md`.
Do not attempt adoption if `CPM_SESSION_ID` is absent from context — the fallback path handles that case.

**Create** the file before starting Step 1 (ensure `docs/plans/` exists). **Update** it after each step completes. **Delete** it only after the final review file has been saved and any autofix/handoff is complete — never before. If compaction fires between deletion and a pending write, all session state is lost.

Use the Write tool to write the full file each time (not Edit — the file is replaced wholesale). Format:

```markdown
# CPM Session State

**Skill**: cpm:review
**Step**: {N} of 5 — {Step Name}
**Output target**: docs/reviews/{nn}-review-{slug}.md
**Input source**: {path to epic doc}
**Review scope**: {Epic | Story N}
**Agents selected**: {comma-separated agent displayNames}

## Completed Steps

### Step 1: Analyse the Artifact
{Summary — what was reviewed, story/task counts, status}

### Step 2: Conduct Adversarial Review
{Summary — finding counts by severity and concern type, key themes}

### Step 3: Write Review File
{Summary — file path written, finding totals}

### Step 4: Autofix
{Summary — path chosen (epic amendment or standalone tasks), items generated, or skipped}

### Step 5: Pipeline Handoff
{Summary — user's choice, context passed}

{...include only completed steps...}

## Next Action
{What to do next}
```

The "Completed Steps" section grows as steps complete. Each summary should capture the key findings and decisions in enough detail for seamless continuation.

## Graceful Degradation

- **No roster found**: Fall back to single-perspective review without agent personas. Note the limitation to the user.
- **No library docs**: Skip silently — review proceeds without external reference documents.
- **Malformed epic doc**: If the epic doc can't be parsed (missing story headings, no acceptance criteria), report what was found and review what's available. Don't block on format issues.
- **Empty review**: If no findings are produced (unlikely but possible), write a review file noting the artifact passed review with no concerns, and skip autofix.

## Guidelines

- **Adversarial, not hostile.** The review should challenge assumptions and find real issues, not manufacture criticism. If something is well-designed, say so. Not every story needs 5 findings.
- **Specific over vague.** "Story 2's acceptance criteria are unclear" is useless. "Story 2, criterion 3 says 'handles errors gracefully' — what does gracefully mean? Which errors? This will be interpreted differently by different implementers" is actionable.
- **Severity matters.** Don't inflate severity to make findings seem more important. A genuinely critical finding (blocks execution) is rare. Most findings will be warnings or suggestions. That's fine.
- **Reference the artifact.** Every finding should point to a specific story, task, or acceptance criterion. Reviewers who can't point to what they're criticising aren't being helpful.
- **Match depth to scope.** A single-story review should take 2-3 agents and produce 3-8 findings. A full epic review should take 3-4 agents and produce 5-15 findings. Don't over-review small artifacts or under-review large ones.
