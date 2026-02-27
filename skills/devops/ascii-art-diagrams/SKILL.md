---
name: ascii-art-diagrams
description: MUST be loaded before creating any ASCII art or text-based diagrams in markdown files. Provides mandatory workflow for properly aligned diagrams. You MUST follow the PLAN, DRAW, VERIFY phases in order. Do NOT skip any phase. Do NOT take shortcuts. Follow the workflow precisely. Diagrams that do not follow this workflow will be rejected. Before proceedig with work, confirm that you understand the entire workflow and will follow it for every diagram you create. You are not to proceed unless you are going to follow the process exactly. You MUST confirm that each step is followed before and after every diagram you create.
license: MIT
compatibility: opencode
---

# ASCII Art Diagrams

This skill provides a **mandatory workflow** for creating properly aligned ASCII art diagrams.

You MUST follow the three phases: PLAN, DRAW, and VERIFY, in that order. Do NOT skip any phase.

Any diagrams that do not follow this workflow will be rejected.

Do NOT rush through the workflow trying to save time or get to the drawing phase faster. Speed is not important. Accuracy and adherence to the workflow is what matters.

Do NOT optimize or shortcut any step in the process.

Do NOT skip any step in the process.

Do NOT claim that you have followed the process if you have not fully completed each step correctly and accurately and have not confirmed that you have done so.

Do NOT deprioritize any steps by jumping directly to drawing and writing.

Do NOT present diagrams until you have fully completed the VERIFY phase.

Do NOT move on to the next diagram until you have fully completed the VERIFY phase for the current diagram and have confirmed that you have done so.

Do NOT start work without confirming you understand the entire workflow and these requirements.

Do NOT perform only superficial verification.

Do NOT abbreviate the verification steps.

You MUST use follow the process on every diagram.

You MUST confirm that you have followed each step before and after every diagram you create.

Refer to the REFERENCE.md document as needed but DO NOT skip any of the workflow steps. The REFERENCE.md is a resource to help you execute the workflow correctly, not a replacement for it.

You MAY use subagents to parallelize the work if it is helpful, but you MUST ensure that the workflow is followed for each diagram and that you have confirmed that it has been followed before proceeding to the next diagram or presenting any diagrams to the user.

You MAY skip steps ONLY if the user explicitly indicates you may do so, but always confirm that you understand which steps you are allowed to skip and which steps you are not allowed to skip before proceeding. When in doubt, do NOT skip any steps and ask for clarification.

---

# TOOLS

This skill includes reusable Python scripts in the `scripts/` directory. These scripts support the
PLAN, DRAW, and VERIFY phases. They do NOT replace the workflow — they help you execute it more
accurately.

## scripts/grid.py — Grid Builder (Phase 2: DRAW)

A library for constructing ASCII diagrams with precise 1-based column placement. Eliminates
off-by-one alignment errors that are common when drawing freehand.

**When to use:** STRONGLY RECOMMENDED for any diagram with multiple vertical elements, branching,
or boxes that must align across rows. Simple single-column diagrams (e.g., a linear flowchart)
may be drawn freehand if preferred.

**Usage:** Import as a library in an inline Python script. Write the script, run it, and capture
the output as your diagram content.

```python
import sys
sys.path.insert(0, '<skill_base_dir>/scripts')
from grid import Grid

g = Grid(width=80)

# Create a line, place elements at exact columns, emit
L = g.line()
g.put(L, 1, '+------+')       # place text at column 1
g.put(L, 10, 'open()')         # place label at column 10
g.put(L, 17, '+------+-----+') # place border at column 17
g.emit(L)                      # print the line (right-stripped)

# Other useful methods:
g.fill(L, 5, 15, '-')          # fill cols 5-15 with '-'
g.hline(L, 5, 15)              # draw +----...----+ from col 5 to 15
g.ruler()                       # print a column ruler for planning
s = g.build(L)                  # return string instead of printing
```

**Key principle:** Decide column positions in Phase 1 (PLAN), then use `g.put(L, col, text)` to
place every element at its exact planned position. This makes the column ruler from Phase 1
directly actionable.

## scripts/verify.py — Automated Verifier (Phase 3: VERIFY)

Automates the mechanical verification checks (Steps 1-4) of Phase 3. Step 5 (Final Read-Through)
remains manual.

**When to use:** MANDATORY. You MUST run this script on every diagram before presenting it.
If the script reports failures, you MUST fix them before proceeding. The script automates:

- Step 1: Unicode scan (banned characters)
- Step 2: Junction audit (every `|`/`^`/`v` adjacent to a horizontal line has matching `+`)
- Step 3: Box consistency (border widths, padding)
- Step 4: Arrow connectivity (no floating arrows)

**Usage:**

```bash
# Verify a diagram piped from grid.py:
python3 draw_diagram.py | python3 <skill_base_dir>/scripts/verify.py

# Verify a specific diagram from a markdown file by heading:
python3 <skill_base_dir>/scripts/verify.py --extract "State Machine" < file.md

# Verify the Nth fenced code block (1-based):
python3 <skill_base_dir>/scripts/verify.py --block 3 < file.md

# Show column positions of structural chars on a specific line:
python3 <skill_base_dir>/scripts/verify.py --columns 5 < diagram.txt

# Quiet mode (only print failures):
python3 <skill_base_dir>/scripts/verify.py --quiet < diagram.txt
```

**Exit codes:** 0 = all checks passed, 1 = one or more checks failed.

**After verify.py passes, you MUST still perform Step 5 (Final Read-Through) manually.**

## Recommended Draw-Verify Loop

For complex diagrams, use this pattern:

1. Complete Phase 1 (PLAN) — column positions, dimensions, ruler
2. Write a Python script using `grid.py` to construct the diagram
3. Run the script and pipe output to `verify.py`
4. If verify fails, fix the grid script and repeat from step 3
5. Once verify passes, perform Step 5 (manual read-through)
6. Write the verified output into the target file

Replace `<skill_base_dir>` with the actual base directory path of this skill (shown when the
skill is loaded).

---

# MANDATORY WORKFLOW

**You MUST follow these three phases in order. Do not skip any phase.**

---

## Phase 1: PLAN (Before Drawing)

**STOP. Do not draw anything until you complete this phase.**

### 1.1 Answer These Questions

Write out your answers before proceeding:

1. What is the maximum width allowed? (Default: 80 chars, max: 100 chars)
2. How many boxes/elements are needed?
3. What are the box labels? What is the longest label?
4. For nested diagrams: What is the total width of inner content + padding?

### 1.2 Calculate Dimensions

For each box:

- Box width = longest text + 2 (padding) + 2 (borders)
- Example: "Database" (8 chars) → 8 + 2 + 2 = 12 chars → `+----------+`

For containers holding multiple boxes:

- Inner width = (box_width × num_boxes) + (spacing × (num_boxes - 1)) + (padding × 2)
- Container must be wider than inner width

### 1.3 Create a Column Ruler and Mark Positions

**This step is mandatory for any diagram with branching or multiple vertical elements.**

```
         1111111111222222222233333333334444444444
1234567890123456789012345678901234567890123456789012345
```

Mark where each vertical element will be:

- "Center `|` at column 25"
- "Left branch `+` at column 15"
- "Right branch `+` at column 35"

### 1.4 Verify Fit

Before drawing, confirm:

- [ ] Total width fits within max width
- [ ] All vertical elements have assigned column positions
- [ ] Container (if any) is wide enough for contents + padding on all sides

**Only proceed to Phase 2 after completing all items above.**

If planning becomes too complicated, consider simplifying the diagram or breaking it into multiple
steps. The goal is to have a clear plan before you start drawing. However, _sometimes_ it might be
easier to start with a draft drawing to get a sense of the layout first, then iterate on it by
validating whether it meets the verification requirements, then adjusting the plan as needed. That
might help you get unstuck if you are having trouble planning the layout conceptually before
drawing anything.

---

## Phase 2: DRAW

**Recommended:** Use `scripts/grid.py` for this phase. See the TOOLS section above for details.

### 2.1 Character Whitelist

Use ONLY these characters:

- **Corners/intersections:** `+`
- **Horizontal lines:** `-`
- **Vertical lines:** `|`
- **Arrows:** `>` `<` `^` `v`
- **Diagonals (sparingly):** `/` `\`

**BANNED** (never use):

```
┌ ┐ └ ┘ ─ │ ├ ┤ ┬ ┴ ┼ ╔ ╗ ╚ ╝ ═ ║ ╭ ╮ ╰ ╯ ▶ ▼ ◀ ▲ ● ○ ◆ ◇
```

### 2.2 Draw to Your Planned Grid

- Place elements at the column positions you determined in Phase 1
- Keep the ruler visible while drawing (remove it later)
- If using `grid.py`, use `g.put(L, col, text)` with the exact column positions from your plan

### 2.3 The Junction Rule

**Every `|` entering or leaving a horizontal line needs a `+` at that EXACT column.**

```
WRONG:                    RIGHT:
    |                         |
--------                  ----+---
    |                         |
```

---

## Phase 3: VERIFY (Required - Do Not Skip)

**After drawing, you MUST complete ALL of these checks before presenting the diagram.**

**MANDATORY:** Run `scripts/verify.py` on your diagram output. This automates Steps 1-4 below.
If verify.py reports any failures, you MUST fix them before proceeding. See the TOOLS section
above for usage details. After verify.py passes, you MUST still complete Step 5 manually.

### Step 1: Unicode Scan

Scan for any banned characters:

- Rounded or styled corners (not `+`)
- Solid lines (not `-` or `|`)
- Fancy arrows (not `<`, `>`, `^`, `v`)

**If found → Return to Phase 2 and redraw.**

### Step 2: Junction Audit (Column Counting)

For EVERY `|` that connects to a horizontal line:

1. Count the exact column position of the `|`
2. Count the exact column position of the `+` on the horizontal line
3. Write out the verification:

```
Line 5: `|` at column 23
Line 6: `+` at column 23 ✓
```

**If ANY mismatch → Fix before proceeding. Do not present the diagram.**

### Step 3: Box Consistency Check

1. Identify groups of related boxes (same logical level)
2. Verify they have the same width
3. Verify adequate padding inside boxes (at least 1 space on each side of text)
4. Verify adequate padding between boxes and container edges

**If inconsistent → Standardize before proceeding.**

### Step 4: Arrow Connectivity Check

For each arrow (`v`, `^`, `<`, `>`):

1. Verify it touches a line or box edge (no floating arrows)
2. Verify the target has a `+` at the connection point
3. Verify that the arrows accurately represent the intended flow

**If floating → Fix before proceeding.**

### Step 5: Final Read-Through

Read the diagram top-to-bottom, left-to-right:

- Does the flow make sense?
- Are there any obvious visual glitches?
- Are there any obvious semantic errors in the diagram?
- Is there adequate whitespace/padding throughout?

**Only after ALL 5 steps pass should you present the diagram to the user.**
