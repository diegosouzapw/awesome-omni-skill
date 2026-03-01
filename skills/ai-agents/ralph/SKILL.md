---
name: ralph
description: "Agent-agnostic autonomous loop creator. Use when asked to 'use ralph', 'ralph this', 'reverse ralph', 'decompose a feature', or '/ralph decompose'. Forward mode implements features end-to-end; decompose mode breaks existing features into atomic user stories for reimplementation."
---

# Ralph - Agent-Agnostic Autonomous Loop Creator

Ralph creates autonomous coding loops that implement features by breaking them into small user stories and completing them one at a time. Each iteration spawns a fresh headless agent with clean context. Memory persists via git, `progress.txt`, and `prd.json`. Ralph is not tied to any single AI agent — the user chooses which agent and model powers each loop.

## Workflow

### Step 1: Understand the Feature

If the user tagged a markdown file via `@`, read it as the feature spec. Otherwise, ask the user to describe the feature.

Ask clarifying questions if needed:
- What problem does this solve?
- What are the key user actions?
- What's out of scope?
- How do we know it's done?

### Step 2: Configure the Loop

Ask the user four questions to configure the loop:

#### 2a. Which AI agent?

Present this list and ask the user to choose:

1. `claude` — Claude Code
2. `droid` — Factory Droid
3. `codex` — OpenAI Codex
4. `opencode` — OpenCode
5. `gemini` — Gemini CLI
6. `copilot` — GitHub Copilot
7. `cc-compatible` — Claude Code-compatible binary (user provides binary name, e.g., `zai`, `minimax`, `kimi`)
8. `custom` — Fully custom command (user provides entire command template with `$PROMPT_FILE` as placeholder)

**Headless command templates per agent** (used when generating the loop script):

```bash
# claude — Claude Code
claude -p "$(cat "$PROMPT_FILE")" --dangerously-skip-permissions --model $MODEL

# droid — Factory Droid
droid exec --skip-permissions-unsafe -f "$PROMPT_FILE" --output-format text -m $MODEL

# codex — OpenAI Codex
codex exec --yolo -m $MODEL "$(cat "$PROMPT_FILE")"

# opencode — OpenCode
opencode run --yolo -m $MODEL "$(cat "$PROMPT_FILE")"

# gemini — Gemini CLI
gemini -p "$(cat "$PROMPT_FILE")" --yolo -m $MODEL

# copilot — GitHub Copilot
copilot -p "$(cat "$PROMPT_FILE")" --yolo --model $MODEL

# cc-compatible — Same as Claude Code with user-provided binary name
$BINARY -p "$(cat "$PROMPT_FILE")" --dangerously-skip-permissions --model $MODEL

# custom — User provides entire command template
# User's template must include $PROMPT_FILE where the prompt file path should go
```

If the user selects `cc-compatible`, ask for the binary name (e.g., `zai`). If the user selects `custom`, ask for the full command template and instruct them to use `$PROMPT_FILE` where the prompt file path belongs.

#### 2b. Which model?

Ask the user for the model identifier (free text input). Examples:
- Claude Code: `opus`, `claude-opus-4-6`, `sonnet`, `claude-sonnet-4-5-20250929`
- Factory Droid: `claude-opus-4-6`, `o3`
- OpenAI Codex: `o3`, `o4-mini`
- OpenCode: `anthropic/claude-opus-4-6`, `openai/o3` (format: `provider/model`)
- Gemini CLI: `gemini-2.5-pro`, `gemini-2.0-flash`
- GitHub Copilot: `claude-sonnet-4-5`, `gpt-4o`

If the user says "default" or leaves it blank, omit the model flag entirely (use the agent's built-in default).

#### 2c. Loop name?

Suggest a name based on the feature in kebab-case (e.g., `add-task-priorities`). The user can accept or provide a different name. This becomes the filename: `.ralph/<loop-name>.sh`

#### 2d. Auto-push and create PR?

Ask the user: **"Should Ralph automatically push the branch and create a PR when the loop finishes?"**

- **Yes** — When the loop ends (all stories complete or max iterations reached), the script will push the branch to `origin` and create a pull request via `gh pr create`. The base branch (e.g., `main` or `master`) is auto-detected at generation time by checking which branch exists on the remote.
- **No** — The script only runs locally. All commits stay local. The user pushes and creates PRs themselves.

If the user says yes, detect the base branch now (at generation time) by checking:
1. Does `refs/remotes/origin/main` exist? → use `main`
2. Does `refs/remotes/origin/master` exist? → use `master`
3. Neither → default to `main`

Bake the resolved base branch directly into the generated script as `DEFAULT_BRANCH="main"` (or `"master"`).

### Step 3: Create prd.json

Generate a `prd.json` file in the project root:

```json
{
  "project": "[Project Name]",
  "branchName": "ralph/[feature-name-kebab-case]",
  "description": "[Feature description]",
  "userStories": [
    {
      "id": "US-001",
      "title": "[Story title]",
      "description": "As a [user], I want [feature] so that [benefit]",
      "acceptanceCriteria": [
        "Criterion 1",
        "Criterion 2",
        "Typecheck passes"
      ],
      "priority": 1,
      "passes": false,
      "notes": ""
    }
  ]
}
```

### Step 4: Generate and Run the Loop

1. Create `.ralph/` directory in the project root if it doesn't exist
2. Check `.gitignore` — add `.ralph/`, `.ralph-archive/`, and `.ralph-last-branch` if missing (create `.gitignore` if it doesn't exist)
3. Read `scripts/ralph.sh` (the reference template in this skill) to understand the loop structure
4. Generate `.ralph/<loop-name>.sh` using the reference template's structure, with the selected agent command baked in:
   - Use the exact command template from Step 2a, substituting the user's model from Step 2b
   - If no model was specified, omit the model flag from the command
   - Set `PROMPT_FILE="$SCRIPT_DIR/<loop-name>-prompt.md"` (co-located with the script)
   - Set `AGENT_BIN` to the selected executable (for `custom`, use the command's executable token)
   - Keep all existing logic: archive, branch tracking, progress init, completion detection, 2s sleep between iterations
   - If the user opted for auto-push+PR in Step 2d: include the `finalize()` function with `DEFAULT_BRANCH` set to the resolved base branch, and `AUTO_PUSH_PR="true"`. Otherwise set `AUTO_PUSH_PR="false"` and omit the `finalize()` function.
5. Copy `scripts/prompt.md` (from this skill) → `.ralph/<loop-name>-prompt.md`
6. Make the script executable: `chmod +x .ralph/<loop-name>.sh`
7. Tell the user: `Run with: .ralph/<loop-name>.sh [max_iterations]`

## Critical Rules for User Stories

### Size: Small but Substantive

Each story MUST be completable in ONE iteration. If you can't describe it in 2-3 sentences, it's too big. But each story must also involve meaningful work — if it's a single find-and-replace or a one-line edit, it's too small and should be combined with related work.

**Too small (combine with related work):**
- "Replace nvidia-smi with rocm-smi in one file" → combine into a broader documentation accuracy story
- "Add one missing env var to README" → combine with other doc gaps
- "Fix a typo in a config file" → combine with other config improvements
- Any story that a developer could complete in under 5 minutes

**Right-sized:**
- Add a database column and migration
- Add a UI component to an existing page
- Update a server action with new logic
- Add a filter dropdown to a list
- Fix a validation bug, add tests for the fix, and update docs
- Consolidate duplicated helper functions across multiple files

**Too big (split these):**
- "Build the entire dashboard" → schema, queries, UI components, filters
- "Add authentication" → schema, middleware, login UI, session handling

### Order: Dependencies First

1. Schema/database changes (migrations)
2. Server actions / backend logic
3. UI components that use the backend
4. Dashboard/summary views

### Acceptance Criteria: Verifiable

**Good:**
- "Add `status` column with default 'pending'"
- "Filter dropdown has options: All, Active, Completed"
- "Typecheck passes"

**Bad:**
- "Works correctly"
- "Good UX"

**Always include:**
- `"Typecheck passes"` on every story
- `"Verify in browser"` on UI stories

## Example

**User says:** "use ralph to add task priorities"

**Step 1:** Read the feature description, ask clarifying questions.

**Step 2:** Ask: Which agent? → `claude`. Which model? → `sonnet`. Loop name? → `add-task-priorities`. Auto-push+PR? → `yes`.

**Step 3:** Create prd.json:
```json
{
  "project": "TaskApp",
  "branchName": "ralph/task-priority",
  "description": "Add priority levels (high/medium/low) to tasks",
  "userStories": [
    {
      "id": "US-001",
      "title": "Add priority field to database",
      "description": "As a developer, I need to store task priority.",
      "acceptanceCriteria": [
        "Add priority column: 'high' | 'medium' | 'low' (default 'medium')",
        "Migration runs successfully",
        "Typecheck passes"
      ],
      "priority": 1,
      "passes": false,
      "notes": ""
    },
    {
      "id": "US-002",
      "title": "Display priority badge on task cards",
      "description": "As a user, I want to see priority at a glance.",
      "acceptanceCriteria": [
        "Colored badge: red=high, yellow=medium, gray=low",
        "Visible without hovering",
        "Typecheck passes",
        "Verify in browser"
      ],
      "priority": 2,
      "passes": false,
      "notes": ""
    }
  ]
}
```

**Step 4:** Generate `.ralph/add-task-priorities.sh` (with `claude -p "$(cat "$PROMPT_FILE")" --dangerously-skip-permissions --model sonnet` as the agent command, `AUTO_PUSH_PR="true"` and `DEFAULT_BRANCH="main"` since the user opted for auto-push+PR), copy prompt to `.ralph/add-task-priorities-prompt.md`, and add `.ralph/`, `.ralph-archive/`, and `.ralph-last-branch` to `.gitignore`.

> prd.json created with 2 user stories. Run `.ralph/add-task-priorities.sh` to start autonomous execution.

## How Ralph Executes

Each iteration, a fresh headless agent:
1. Reads `prd.json` and `progress.txt`
2. Picks highest priority story where `passes: false`
3. Implements it
4. Runs quality checks (typecheck, lint, test)
5. Commits if passing (or commits progress notes if stuck — see "If You Get Stuck" in prompt)
6. Updates `prd.json` to mark `passes: true` (or updates `notes` if blocked)
7. Appends learnings to `progress.txt`
8. Exits

Loop continues until all stories pass or max iterations hit.

If the user opted for auto-push+PR (Step 2d), then after the loop ends (all stories complete or max iterations reached):
1. The loop script pushes the branch to origin
2. Creates a PR via `gh pr create` from the ralph branch to the default branch
3. If `gh` CLI is not available or PR creation fails, prints manual instructions instead of aborting
4. Push or PR failures are handled gracefully — they never mask a successful loop run

If the user chose local-only, the script simply exits after the loop. All commits remain local.

## Files Reference

| File | Purpose |
|------|---------|
| `prd.json` | User stories with pass/fail status |
| `progress.txt` | Append-only learnings for future iterations |
| `.ralph/<name>.sh` | Generated loop script with agent command baked in |
| `.ralph/<name>-prompt.md` | Prompt file for the loop (copy of `scripts/prompt.md`) |

## Decompose Mode

Triggered by: `/ralph decompose <input>` or natural language like "reverse ralph this feature",
"decompose X into a replication plan", "break this feature down into user stories".

### What it does

Reverse Ralph takes any description of an existing feature and decomposes it — recursively
and completely — into an atomized `prd.json` that a forward Ralph loop can execute to
greenfield-reimplement the feature.

Decomposition is behavioral and functional: it captures what the feature does and how it
behaves from the outside, not how it is built internally. The forward loop handles all
implementation decisions.

### Inputs accepted

- URLs (fetched and spidered up to 2 hops of linked documentation)
- Local file paths (read directly)
- Natural language descriptions
- Any combination of the above

### Agent workflow

Follow the detailed instructions in `scripts/decompose-init-prompt.md` for steps 1–7 below.

1. **Gather inputs.** Read all provided sources. For URLs, fetch the page and follow
   documentation links up to 2 hops deep (same documentation domain only). Synthesize
   all gathered content into a capability surface: a structured description of all
   observable behaviors, states, inputs, outputs, configuration options, and integrations
   of the feature.

2. **Ask the loop name.** Ask the user what to name this decomposition run. Used for the
   generated script filename: `.ralph/decompose-<n>.sh`.

3. **Ask which execution agent** to use for the decomposition loop. Same agent matrix as
   forward Ralph: `claude`, `droid`, `codex`, `opencode`, `gemini`, `copilot`,
   `cc-compatible`, or `custom`.

4. **Ask which model** (optional — leave blank to use the CLI default).

5. **Seed `decomp.json`.** Generate the initial state file with top-level capability
   clusters extracted from the capability surface. Each cluster gets status `needs_split`.

6. **Generate `.ralph/decompose-<n>.sh`** from `scripts/decompose.sh`, substituting
   `__AGENT__`, `__MODEL__`, and `__LOOP_NAME__`. Copy `scripts/decompose-prompt.md` to
   `.ralph/decompose-<n>-prompt.md`.

7. **Instruct the user** to run:
   ```
   .ralph/decompose-<n>.sh [max_iterations]
   ```
   Default `max_iterations` is 50. The loop runs autonomously until all leaf nodes in
   `decomp.json` have status `atomic` (split parent nodes get status `split`), then
   emits `prd.json`.

### Known Limitations

- **Context window**: The full `decomp.json` is appended to each iteration's prompt.
  For very large feature decompositions (hundreds of nodes), this may approach agent
  context limits. If this happens, increase `max_iterations` and let the loop resume
  across runs.

### Decompose Files Reference

| File | Purpose |
|------|---------|
| `scripts/decompose-init-prompt.md` | Initialization prompt for the orchestrating agent (steps 1–7) |
| `decomp.json` | Decomposition state tree with nodes and status |
| `prd.json` | Final output — forward-Ralph-compatible flat story list |
| `.ralph/decompose-<n>.sh` | Generated decomposition loop script |
| `.ralph/decompose-<n>-prompt.md` | Prompt file for the decomposition loop |
