---
name: fix-repo
description: Improve a specific repo's corpus conformance by fixing its top diverging cops in parallel using worktree-isolated teammates.
allowed-tools: Bash(*), Read, Write, Edit, Grep, Glob, Task, TeamCreate, TaskCreate, TaskUpdate, TaskList, TaskGet, SendMessage
---

# Fix Repo — Repo-Targeted Conformance Improvement

This skill targets a specific corpus repo (e.g., rails, discourse, mastodon) and fixes
the cops that contribute the most FP/FN for that repo. Unlike `/fix-cops` (globally worst
cops) or `/fix-department` (all cops in a gem), this focuses on improving a specific repo's
match rate.

If you edit code yourself (without dispatching teammates), do that work in a dedicated
git worktree by default. Only skip this when the user explicitly asks to use the current tree.

## Workflow

### Phase 0: Assess (you do this)

1. **If no repo was specified**, show the repo list and let the user pick:
   ```bash
   python3 scripts/investigate-repo.py --list $ARGUMENTS
   ```

2. **Once a repo is chosen**, run the repo investigation and **paste its full output
   verbatim to the user** (the table IS the primary output — do not summarize or skip it):
   ```bash
   python3 scripts/investigate-repo.py <repo-name> $ARGUMENTS
   ```
   The script automatically excludes cops fixed since the last corpus oracle run
   by scanning git commit messages, so the output only shows cops that still need work.

3. Show the user the top diverging cops and confirm the target repo.

### Phase 1: Full Table & Recommendations (you do this)

1. **Show the complete table** of ALL diverging cops (use `--limit 0`) ordered by FP+FN
   descending. This is the full picture of what needs to happen to reach 100%:
   ```bash
   python3 scripts/investigate-repo.py <repo-name> --limit 0 $ARGUMENTS
   ```
   Paste the full output verbatim to the user.

2. **Check global FP/FN** for the top candidates (use `--repos-only | head -5` for each).
   This helps prioritize cops where the target repo is a significant contributor.

3. **Recommend up to 4 cops** to fix in this batch. Prioritize by impact (highest FP+FN
   first), but use judgment about difficulty. Present your recommendations with global
   context:
   ```
   My recommendations for this batch (4 cops):
   1. Style/OptionHash — 91 FP on rails (1,819 FP globally), likely <hypothesis>
   2. Layout/IndentationWidth — 0 FP, 5,696 FN on rails (Xk FN globally), likely <hypothesis>
   3. ...

   Want me to proceed with these, or would you like to swap any out?
   ```

   Don't skip any cop category — Layout alignment cops, low-match-rate cops, and FN-heavy
   cops are all fair game. The goal is 100% conformance, so everything needs to be tackled
   eventually.

**Wait for user confirmation before proceeding.** The user may want to pick different cops
or adjust the batch size.

Once confirmed, investigate each selected cop's FP/FN pattern in depth:
```bash
python3 scripts/investigate-cop.py Department/CopName --context --fp-only --limit 10
python3 scripts/investigate-cop.py Department/CopName --context --fn-only --limit 10
```

**Run the delta reducer** on up to 3 examples per cop (mix of FP and FN) to get minimal reproductions:
```bash
python3 scripts/reduce-mismatch.py Department/CopName repo_id filepath:line            # FP
python3 scripts/reduce-mismatch.py Department/CopName repo_id filepath:line --type fn   # FN
```
Pick examples from different repos when possible. The reduced files (typically 5–20 lines) go
to `/tmp/nitrocop-reduce/` — read them and include them in the teammate prompt.

Summarize: cop name, repo-specific FP/FN, global FP/FN, minimal repro(s), root cause hypothesis.

### Phase 2: Dispatch (you do this)

1. Create a team:
   ```
   TeamCreate(team_name="fix-repo", description="Improve <repo-name> conformance")
   ```

2. Create tasks for each cop fix.

3. Spawn one teammate per cop using the Task tool. **Critical settings:**
   - `isolation: "worktree"` — each teammate gets its own git worktree
   - `subagent_type: "general-purpose"` — needs full edit/bash access
   - `team_name: "fix-repo"`
   - `mode: "bypassPermissions"` — teammates need to run cargo test etc.

4. Each teammate prompt MUST include:
   - The exact cop name (e.g., `Lint/ConstantResolution`)
   - The FP/FN counts (both repo-specific and global) and root cause hypothesis
   - **The minimal repro(s) from the delta reducer** — paste the reduced Ruby source directly
   - Whether to focus on FP fixes, FN fixes, or both
   - A reminder that parallel-agent activity often leaves unrelated local modifications; those files are off-limits
   - The teammate workflow (Phase 3 below) — paste the full instructions

### Phase 3: Teammate Workflow (paste this into each teammate's prompt)

```
You are fixing false positives/negatives in a single nitrocop cop to improve corpus
conformance for a target repo. Follow the CLAUDE.md rules strictly.

**NEVER use git stash or git stash pop.** You are in an isolated git worktree — just commit directly.
Parallel-agent activity is common. If you see unrelated modified files, do not edit/revert them.

## Steps

1. **Read the cop source** at `src/cop/<dept>/<cop_name>.rs`
   Read the vendor RuboCop spec at `vendor/rubocop*/spec/rubocop/cop/<dept>/<cop_name>_spec.rb`
   **Check for existing investigation comments** (marked with "Known false positives" or
   "reverted") — these document previously attempted fixes that regressed on corpus
   validation. Do NOT repeat the same approach. Either find a different root cause or
   extend the prior approach to avoid its documented failure mode.

2. **Understand the FP/FN pattern** from the examples provided in your prompt.
   If needed, read the actual source files from `vendor/corpus/<repo_id>/<path>` to see more context.

3. **Add test cases (TDD)**:
   - For FP fixes: add the false-positive pattern to `tests/fixtures/cops/<dept>/<cop_name>/no_offense.rb`
   - For FN fixes: add the missed detection to `tests/fixtures/cops/<dept>/<cop_name>/offense.rb`
   - Run `cargo test --release -p nitrocop --lib -- <cop_name_snake>` to verify the test FAILS

4. **Fix the cop implementation** in `src/cop/<dept>/<cop_name>.rs`

5. **Verify**:
   - `cargo test --release -p nitrocop --lib -- <cop_name_snake>` — all tests pass
   - `cargo fmt`
   - `cargo clippy --release -- -D warnings`

6. **Commit your fix**:
   ```bash
   git add src/cop/<dept>/<cop_name>.rs tests/fixtures/cops/<dept>/<cop_name>/
   # Add any other changed files
   git commit -m "Fix <Department/CopName> false positives/negatives

   <one-line description of what was wrong>

   Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>"
   ```
   Stage only files for this cop fix. Do not include unrelated modified files.

7. **Report back** via SendMessage with:
   - What the root cause was
   - What you changed
   - Whether tests pass
   - The commit SHA
   - If you could NOT fix it: explain why and whether it should be deferred
```

### Phase 4: Collect Results (you do this)

1. Wait for all teammates to report back.

2. For each completed fix:
   - Note the worktree branch name from the Task result
   - Cherry-pick or merge the commit into your working branch

3. Run full verification:
   ```bash
   cargo fmt
   cargo clippy --release -- -D warnings
   cargo test --release
   ```

4. Verify each fixed cop against the corpus:
   ```bash
   python3 scripts/check-cop.py Department/CopName --verbose --rerun
   ```
   **Corpus validation is the acceptance gate** — unit tests passing is necessary but
   NOT sufficient.

5. **Handle regressions**: if a fix increases FP count (even if unit tests pass), revert
   the code change but **add a detailed investigation comment** to the cop source file
   documenting what was tried, why it regressed, and what a correct fix would need. Use:
   ```rust
   /// ## Known false positives (N FP in corpus as of YYYY-MM-DD)
   ///
   /// An attempt was made to ... (commit XXXXXXXX, reverted). The approach: ...
   /// This fixed the target FPs but introduced N NEW false positives (X→Y FP).
   /// Root cause of regression: ...
   /// A correct fix needs to: ...
   ```

6. Re-run the repo investigation to show updated status:
   ```bash
   python3 scripts/investigate-repo.py <repo-name>
   ```
   Note: This still reads the original corpus data. Per-cop verification via check-cop.py
   gives the ground truth for fixed cops.

6. Report to the user:
   - Which cops were fixed (with FP/FN counts)
   - Estimated impact on the target repo's match rate
   - Which cops couldn't be fixed (and why)
   - Summary of changes ready for commit/PR

## Arguments

- `/fix-repo` — show repo list, let user pick
- `/fix-repo rails` — fix top diverging cops for the rails repo
- `/fix-repo discourse` — fix top diverging cops for discourse
- `/fix-repo rails --fp-only` — only fix FP-producing cops for rails
- `/fix-repo --input /path/to/corpus-results.json rails` — use local corpus file
