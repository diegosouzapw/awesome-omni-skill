---
name: init-project
description: Initialize a new ML project from this template
disable-model-invocation: true
allowed-tools: Bash(git *, python *, pip *, wandb *), Read, Edit, Write, Glob, Grep, AskUserQuestion
argument-hint: [project-name] [description]
---

# Initialize Project

Set up a new ML project from the template. User provides project name and description.

## Input

$ARGUMENTS

If project name or description not provided, ask the user.

## Workflow

### Phase 1: Gather Info

1. **Get project details** from user:
   - Project name (used for WandB project, directory context)
   - Short description (one sentence)
   - WandB entity (team or username)
   - WandB launch queue name
   - Python version preference (default: current system python3)
   - Environment variables / secrets needed (e.g., Kaggle token, HuggingFace token, custom API keys)

### Phase 2: Python Environment

2. **Create and activate virtual environment**
   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   ```

3. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

4. **Verify imports work**
   ```bash
   python -c "import wandb; import torch; print('All imports OK')"
   ```

### Phase 3: WandB Setup

5. **Check WandB login**
   ```bash
   wandb status
   ```
   - If not logged in, prompt user to run `wandb login`
   - Verify the entity is accessible

6. **Verify WandB Launch queue**
   ```bash
   wandb launch-agent --help
   ```
   - Ask user to confirm:
     - Queue name exists and is created in WandB UI
     - At least one agent is running or will be started on GPU environment
   - Test connectivity: `python -c "import wandb; api = wandb.Api(); print('WandB API OK')"`

### Phase 4: Environment Variables

7. **Write `.claude/settings.local.json`** with project secrets and env vars:
   - Ask user for any API keys / tokens they need (e.g., `KAGGLE_USERNAME`, `KAGGLE_KEY`, `HF_TOKEN`)
   - Write secrets only to `.claude/settings.local.json` (gitignored, local only):

     ```json
     {
       "env": {
         "WANDB_API_KEY": "<value>",
         "KAGGLE_USERNAME": "<value>",
         "KAGGLE_KEY": "<value>",
         "HF_TOKEN": "<value>"
       }
     }
     ```

   - Only include keys the user actually needs — omit unused ones
   - Verify `.claude/settings.local.json` is in `.gitignore`: `git check-ignore .claude/settings.local.json`
   - Non-secret project defaults go in `.claude/settings.json` (committed, shared):

     ```json
     {
       "env": {
         "WANDB_ENTITY": "<entity>",
         "WANDB_PROJECT": "<project-name>",
         "WANDB_QUEUE": "<queue-name>"
       }
     }
     ```

### Phase 5: CLAUDE.md — Project-Specific Notes

8. **Write CLAUDE.md** with project-specific content:

   ```markdown
   # CLAUDE.md

   ## Project: <project-name>

   <description>

   ## Key Features

   - **Git controlled**: Version-controlled experiment workflow
   - **WandB based**: Experiment tracking, launch, and evaluation
   - **Python programmed**: pip + requirements.txt
   - **Claude Code managed**: Human provides ideas, Claude Code runs the loop

   See `.claude/rules/` for detailed workflow rules.

   ## WandB Config

   - **Entity**: <entity>
   - **Project**: <project-name>
   - **Queue**: <queue-name>

   ## Current Baseline

   Initial baseline. No experiments run yet.

   ## Project-Specific Notes

   <!-- Add notes as the project evolves -->
   ```

### Phase 6: Baseline Documentation

9. **Create initial baseline doc** at `docs/baseline-history.md`:

   ```markdown
   # Baseline History

   ## Baseline 0 — Initial

   - **Date**: <today>
   - **Description**: Initial project setup from template
   - **Merged branches**: None (starting point)
   - **Discarded branches**: None
   - **Key metrics**: N/A (no training run yet)

   ---
   ```

### Phase 7: Git Cleanup

10. **Ensure clean git state**
   - Verify on `main` branch: `git branch --show-current`
   - Delete any stale local branches (except `main`): list with `git branch`, confirm with user before deleting
   - Delete any stale remote branches: `git branch -r` and confirm
   - Stage and commit all initialization files:
     - `CLAUDE.md`
     - `docs/baseline-history.md`
     - `.venv` should be in `.gitignore` (create/update if needed)

11. **Push to remote**
    - `git push origin main`

### Phase 8: Verify

12. **Run verification checklist** and print results:
    - [ ] Python venv active and dependencies installed
    - [ ] WandB logged in and API accessible
    - [ ] WandB entity and project confirmed
    - [ ] WandB launch queue name recorded
    - [ ] CLAUDE.md has project-specific config
    - [ ] docs/baseline-history.md exists
    - [ ] Git is on `main`, clean, no stale branches
    - [ ] `.gitignore` includes `.venv/` and `.claude/settings.local.json`
    - [ ] `.claude/settings.json` has non-secret project defaults
    - [ ] `.claude/settings.local.json` has secrets (not committed)
    - [ ] All changes committed and pushed
    - [ ] Next: run `/setup-automation` to set up GitHub labels, secrets, and automatic CI pipeline

13. **Print next steps**:
    - "Project initialized. Next:"
    - "1. Run `/setup-automation` to enable automatic result analysis (WandB → GitHub Actions → Claude Code)"
    - "2. Ensure a WandB Launch agent is running on your GPU environment"
    - "3. Use `/new-experiment [idea]` to create your first experiment"

## Rules

- Always ask for confirmation before deleting any git branches
- Never commit `.venv/` or `.claude/settings.local.json` — ensure `.gitignore` covers both
- If WandB login fails, do not proceed — guide user to fix it
- All WandB config (entity, project, queue) must be recorded in CLAUDE.md
