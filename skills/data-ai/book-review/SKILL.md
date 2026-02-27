---
name: book-review
description: Review any part of the book using parallel specialist agents
user-invocable: true
---

# Book Review

Review part of the book specified by `$ARGUMENTS`, using the project's book
review policies.

## Step 1: Figure Out What to Review

The user's `$ARGUMENTS` could be anything. Interpret them:

| Arguments | What to do |
|-----------|-----------|
| *(empty)* | Run `git diff -- book/src/` and `git diff --cached -- book/src/`. Review the uncommitted changes. If no changes, say so and stop. |
| A file path | Read that file. Review the whole chapter. |
| A file path with lines (`foo.md:50-80`) | Read that file. Focus the review on those lines (but read the full file for context). |
| Multiple file paths | Read all of them. Review each, and also look for consistency across them. |
| A section heading or topic name | Use Grep/Glob to find where that topic is discussed in `book/src/`. Read those sections. Review them. |
| A conceptual description ("how we explain X", "the transition between Y and Z") | Search the book to find the relevant passages. Read them with surrounding context. Review them as a unit. |
| A mix of the above | Handle each part, then review the union. |

If you're unsure what the user means, ask before launching reviewers.

## Step 2: Discover and Launch Reviewers

1. Use Glob to find all `.claude/book-review/*.md` files.
2. Read `.claude/book-review/standards.md` (master standards shared by all
   reviewers).
3. For EACH policy file **except `standards.md`**, launch a `general-purpose`
   Task agent (use model `sonnet`). Give each agent a prompt constructed from
   this template — adapt it based on what you're reviewing:

   > You are reviewing part of "The Ragu Book," a technical book about a
   > recursive proof system (proof-carrying data framework).
   >
   > Read these files:
   > - `.claude/book-review/standards.md` (master standards that always apply)
   > - `.claude/book-review/{focus}.md` (your specific review policy)
   >
   > **What to review:** {describe the specific content — file paths, line
   > ranges, the topic, whatever the user specified. If reviewing a diff,
   > include the diff text and list the full file paths for context.}
   >
   > {If the scope is a diff or a narrow range, add: "Focus on the specified
   > content. Flag issues in surrounding text ONLY if the specified content
   > introduced an inconsistency with it."}
   >
   > {If the scope spans multiple files or a concept, add: "Pay attention to
   > consistency across the passages. Flag contradictions, redundancies, or
   > gaps in the explanation as it spans these locations."}
   >
   > If the policy references other files (like `book/macros.txt` or
   > `book/src/appendix/terminology.md`), read those too.
   >
   > For each finding:
   > - **Location**: file path and quoted text (or line number)
   > - **Issue**: what's wrong, specifically
   > - **Suggestion**: a concrete rewrite, not just "improve this"
   > - **Severity**: `must-fix` (clearly wrong/confusing) or `suggestion`
   >
   > Stay within your policy's scope. Be specific. If you find no real issues,
   > say so — don't manufacture problems.

   Additionally, read `book/FIXME.md` and extract the `###`-level headings
   under `## Deferred Issues`. Include them in each reviewer's prompt as a
   bullet list with the instruction:
   > The following issues are already tracked and deferred. Do NOT raise
   > findings that duplicate or overlap with these items. If your finding
   > is essentially the same issue, omit it.
   >
   > {bullet list of ### headings from book/FIXME.md}
   >
   > However, if while reviewing you notice that a deferred issue appears to
   > have been **resolved** by existing content or is now easily resolvable
   > given changes in the surrounding text, mention it as a separate
   > observation (not a finding). Format:
   > - **Resolvable FIXME**: "{FIXME heading}" — {brief explanation of why
   >   it appears resolved or how to resolve it now}

   Launch ALL agents in parallel (multiple Task calls in one message).

## Step 3: Run Automated Link Checks

In parallel with the reviewer agents (launched in Step 2), run the automated
QA scripts to detect broken links and dead pages:

1. Run `python3 qa/book/broken_links.py` — finds broken internal links and
   invalid anchors (requires `mdbook` to be installed).
2. Run `python3 qa/book/dead_pages.py` — finds markdown files not listed in
   `SUMMARY.md`.
3. Run `python3 qa/book/line_width.py` — finds lines exceeding the
   80-character width limit (excludes code blocks, math blocks, and other
   exempt content).

If either script fails to run (e.g., `mdbook` is not installed), note the
failure in your output but do not block the review. Include any findings from
these scripts alongside the reviewer agents' results in the synthesis step.

## Step 4: Synthesize

Once all agents return, organize findings by location (earliest in the book
first). If multiple reviewers flagged overlapping concerns, merge them. Present:

- **Must-fix** issues first
- **Suggestions** second
- For each finding, note which reviewer(s) identified it

If the review scope was cross-file or conceptual, add a short section at the end
about cross-cutting observations (consistency, narrative coherence, etc.).

Before presenting findings, read `book/FIXME.md` and remove any finding that
substantially overlaps with an already-deferred issue (match against the `###`
headings). If you remove findings this way, do not mention them at all.

If any reviewer flagged a resolvable FIXME, collect them in a **Potentially
Resolvable FIXMEs** section after the main findings. The user can then use
`/book-fixme` to address them.

## Step 5: Validate Proposed Changes

After synthesizing findings, but before presenting them to the user:

1. Collect all proposed changes (the "suggestion" field from each finding) into
   a single numbered list — the **proposed plan**.
2. For EACH policy file (same set as Step 2 — every `.claude/book-review/*.md`
   except `standards.md`), launch a `general-purpose` Task agent (model
   `sonnet`) with this prompt:

   > You are validating a set of proposed book edits against review policies.
   >
   > Read these files:
   > - `.claude/book-review/standards.md` (master standards)
   > - `.claude/book-review/{focus}.md` (your policy)
   >
   > Here is the proposed plan of changes:
   > {numbered list of proposed changes with locations and suggested rewrites}
   >
   > For each proposed change, check whether applying it would **introduce** a
   > violation of any rule in your policy or the master standards. Only flag
   > real conflicts — do not restate rules that are already satisfied.
   >
   > For each conflict found:
   > - **Change #**: which proposed change
   > - **Rule violated**: quote the relevant policy text
   > - **Conflict**: explain specifically how the suggestion violates the rule
   > - **Resolution**: suggest how to fix the suggestion to comply
   >
   > If no proposed changes conflict with your policy, say so.

   Launch ALL agents in parallel.

3. Merge validation feedback into the findings. For each conflict:
   - If the validator provides a compliant alternative, replace the original
     suggestion with the corrected version.
   - If the conflict has no clear resolution, annotate the finding with the
     conflict so the user can decide during triage.

4. If any suggestions were corrected, briefly note it in the synthesis output
   (e.g., "Grammar validator caught that suggestion #3 would violate the
   lowercase-technical-terms rule; corrected.").

## Step 6: Triage

After presenting findings, use AskUserQuestion to let the user decide the
disposition of each finding (or group of closely related findings). For each,
offer:

- **Fix** — apply the suggested fix now
- **Skip** — drop this finding; do not record it anywhere
- **Defer** — record this finding in `book/FIXME.md` for future attention

If there are many findings, you may batch them into logical groups and ask
about each group rather than each individual finding.

## Step 7: Apply Fixes

For each finding the user chose to **fix**:

1. Read the target file to get the current content.
2. Apply the suggested change using the Edit tool. If the finding is a group
   (e.g., "add slugs to these 15 headings"), apply all changes in the group.
3. After applying all fixes, run any relevant QA scripts to verify the changes
   don't introduce new issues:
   - If formatting changes were made: `python3 qa/book/line_width.py`
   - If links or headings were changed: `python3 qa/book/broken_links.py`
4. Briefly report what was changed and any QA results.

## Step 8: Record Deferred Issues

For each finding the user chose to **defer**:

1. Read `book/FIXME.md`.
2. Check whether a substantially similar issue is already listed. First scan
   the `###` headings for a potential match; if a heading looks similar, read
   the paragraph beneath it to confirm it describes the same issue. Only skip
   if both the heading and description match — do not treat a heading match
   alone as a duplicate.
3. If the issue is new, append it as a new `###`-level subsection under
   `## Deferred Issues`, following the existing format: a short descriptive
   heading, then a concise paragraph explaining the issue and its location.
4. Write the updated file.

After recording, confirm what was deferred and remind the user they can use
`/book-refine` to turn feedback into policy improvements.
