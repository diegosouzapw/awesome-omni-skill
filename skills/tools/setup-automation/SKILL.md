---
name: setup-automation
description: Set up GitHub Actions + WandB Automation for automatic result analysis
disable-model-invocation: true
allowed-tools: Bash(gh *, git *, wandb *), Read, Edit, Write, AskUserQuestion
argument-hint: (no arguments)
---

# Setup Automation

Set up the automatic analysis pipeline: WandB Automation → GitHub Actions → Claude Code. This enables automatic result analysis and PR posting when WandB runs finish.

## Prerequisites

- `/init-project` must have been run first (WandB and GitHub are configured)
- User must have a GitHub PAT with `repo` scope ready
- User must have an Anthropic API key ready

## Workflow

### Phase 1: Gather Credentials

1. **Ask for credentials** (these will be stored as GitHub Secrets, NOT in the repo):
   - Anthropic API key (`ANTHROPIC_API_KEY`)
   - GitHub PAT with `repo` scope (for WandB webhook → GitHub dispatch)
   - Confirm WandB API key is available (should already be in `.claude/settings.local.json`)

### Phase 2: GitHub Repository Configuration

2. **Get repo info**
   ```bash
   gh repo view --json owner,name,url
   ```

3. **Set GitHub Secrets**
   ```bash
   gh secret set ANTHROPIC_API_KEY
   gh secret set WANDB_API_KEY
   ```
   Claude cannot set these programmatically (they require interactive input or piped values). Guide the user to set them:
   - Option A: `echo "<value>" | gh secret set ANTHROPIC_API_KEY`
   - Option B: Set via GitHub UI → Settings → Secrets and Variables → Actions

4. **Set GitHub Variables** (non-secret, read from `.claude/settings.json`):
   ```bash
   gh variable set WANDB_ENTITY --body "<entity>"
   gh variable set WANDB_PROJECT --body "<project>"
   ```

5. **Create GitHub labels** for experiment tracking:
   ```bash
   # Workflow state labels
   gh label create "experiment:in-progress" --color "FFA500" --description "Experiment branch is being developed" --force
   gh label create "experiment:done" --color "6E6E6E" --description "Experiment cycle completed" --force

   # Verdict labels
   gh label create "experiment:winner" --color "00AA00" --description "Experiment outperformed baseline" --force
   gh label create "experiment:loser" --color "DD0000" --description "Experiment did not outperform baseline" --force

   # Run state labels (set by check-results post-pr)
   gh label create "experiment:finished" --color "0E8A16" --description "WandB run finished successfully" --force
   gh label create "experiment:failed" --color "B60205" --description "WandB run failed" --force
   gh label create "experiment:crashed" --color "B60205" --description "WandB run crashed" --force
   gh label create "experiment:running" --color "1D76DB" --description "WandB run is in progress" --force
   gh label create "experiment:killed" --color "FBCA04" --description "WandB run was killed" --force

   # Baseline labels (used by rollback-baseline)
   gh label create "regression" --color "D93F0B" --description "Baseline regression detected" --force
   gh label create "baseline" --color "5319E7" --description "Related to baseline management" --force
   ```

### Phase 3: Verify GitHub Actions Workflow

6. **Check workflow file exists**
   ```bash
   cat .github/workflows/wandb-run-finished.yml
   ```
   - If missing, warn the user and suggest re-initializing from the template

7. **Verify workflow is enabled**
   ```bash
   gh workflow list
   ```
   - If the workflow is disabled, enable it: `gh workflow enable wandb-run-finished.yml`

### Phase 4: WandB Automation Setup

8. **Guide user through WandB Automation creation**

   WandB Automations must be configured in the WandB UI. Print the following instructions:

   ```
   === WandB Automation Setup ===

   1. Go to: https://wandb.ai/<entity>/<project>/automations
   2. Click "Create automation"
   3. Configure:
      - Name: "Auto-analyze on run finish"
      - Event type: "Run state changes"
      - Trigger on: finished, failed, crashed
      - Action: "Webhook"
      - URL: https://api.github.com/repos/<owner>/<repo>/dispatches
      - Method: POST
      - Headers:
        Authorization: Bearer <YOUR_GITHUB_PAT>
        Accept: application/vnd.github+json
      - Body: (copy from misc/wandb_webhook_payload.json)
   4. Save the automation
   ```

   Ask user to confirm when done.

### Phase 5: Test the Pipeline

9. **Test GitHub dispatch** (dry run):
   ```bash
   gh api repos/<owner>/<repo>/dispatches \
     -f event_type=wandb_run_finished \
     -f 'client_payload[branch]=test' \
     -f 'client_payload[run_id]=test-run' \
     -f 'client_payload[run_state]=finished'
   ```

10. **Verify the workflow ran**:
    ```bash
    gh run list --workflow=wandb-run-finished.yml --limit 1
    ```
    - If it shows a run triggered by `repository_dispatch`, the pipeline works
    - The test run will fail (no real branch/run) — that's expected, it just proves the trigger works

### Phase 6: Summary

11. **Print verification checklist**:
    - [ ] `ANTHROPIC_API_KEY` set in GitHub Secrets
    - [ ] `WANDB_API_KEY` set in GitHub Secrets
    - [ ] `WANDB_ENTITY` set in GitHub Variables
    - [ ] `WANDB_PROJECT` set in GitHub Variables
    - [ ] All GitHub labels created (workflow, verdict, run state, baseline)
    - [ ] `.github/workflows/wandb-run-finished.yml` exists and is enabled
    - [ ] WandB Automation created with webhook to GitHub dispatch
    - [ ] Test dispatch triggered the workflow

12. **Print how it works**:
    ```
    Pipeline: WandB run finishes → WandB Automation fires webhook →
    GitHub Actions receives dispatch → Claude Code analyzes results →
    Results posted to PR automatically
    ```

## Rules

- Never store secrets in the repo — only in GitHub Secrets or `.claude/settings.local.json`
- The GitHub PAT for the webhook belongs to the user — Claude should never ask for it in plaintext. Guide the user to paste it directly into WandB UI.
- If any step fails, explain what went wrong and how to fix it manually
- This skill is idempotent — safe to re-run if setup was partial
