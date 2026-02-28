---
name: adr-graph-easy-architect
description: ASCII architecture diagrams for ADRs via graph-easy. TRIGGERS - ADR diagram, architecture diagram, ASCII diagram.
---

# ADR Graph-Easy Architect

Create comprehensive ASCII architecture diagrams for Architecture Decision Records (ADRs) using graph-easy. Pure text output with automatic layout - no image rendering required.

## When to Use This Skill

- Writing new ADR that involves architectural changes
- ADR describes migration, integration, or system changes
- User asks for visual representation of a decision
- Existing ADR diagram needs review or update

## Preflight Check

Run these checks in order. Each layer depends on the previous.

### Layer 1: Package Manager

```bash
/usr/bin/env bash << 'SETUP_EOF'
# Detect OS and set package manager
case "$(uname -s)" in
  Darwin) PM="brew" ;;
  Linux)  PM="apt" ;;
  *)      echo "ERROR: Unsupported OS (require macOS or Linux)"; exit 1 ;;
esac
command -v $PM &>/dev/null || { echo "ERROR: $PM not installed"; exit 1; }
echo "✓ Package manager: $PM"
SETUP_EOF
```

### Layer 2: Perl + cpanminus (mise-first approach)

```bash
# Prefer mise for unified tool management
if command -v mise &>/dev/null; then
  # Install Perl via mise
  mise which perl &>/dev/null || mise install perl
  # Install cpanminus under mise perl
  mise exec perl -- cpanm --version &>/dev/null 2>&1 || {
    echo "Installing cpanminus under mise perl..."
    mise exec perl -- curl -L https://cpanmin.us | mise exec perl -- perl - App::cpanminus
  }
  echo "✓ cpanminus installed (via mise perl)"
else
  # Fallback: Install cpanminus via system package manager
  command -v cpanm &>/dev/null || {
    echo "Installing cpanminus via $PM..."
    case "$PM" in
      brew) brew install cpanminus ;;
      apt)  sudo apt install -y cpanminus ;;
    esac
  }
  echo "✓ cpanminus installed"
fi
```

### Layer 3: Graph::Easy Perl module

```bash
# Check if Graph::Easy is installed (mise-first)
if command -v mise &>/dev/null; then
  mise exec perl -- perl -MGraph::Easy -e1 2>/dev/null || {
    echo "Installing Graph::Easy via mise perl cpanm..."
    mise exec perl -- cpanm Graph::Easy
  }
  echo "✓ Graph::Easy installed (via mise perl)"
else
  perl -MGraph::Easy -e1 2>/dev/null || {
    echo "Installing Graph::Easy via cpanm..."
    cpanm Graph::Easy
  }
  echo "✓ Graph::Easy installed"
fi
```

### Layer 4: Verify graph-easy is in PATH

```bash
# Verify graph-easy is accessible and functional
command -v graph-easy &>/dev/null || {
  echo "ERROR: graph-easy not found in PATH"
  exit 1
}
# Test actual functionality (--version exits with code 2, unreliable)
echo "[Test] -> [OK]" | graph-easy &>/dev/null && echo "✓ graph-easy ready"
```

### All-in-One Preflight Script

```bash
/usr/bin/env bash << 'PREFLIGHT_EOF'
# Copy-paste this entire block to ensure graph-easy is ready (macOS + Linux)
# Prefers mise for unified cross-platform tool management

# Check for mise first (recommended)
if command -v mise &>/dev/null; then
  echo "Using mise for Perl management..."
  mise which perl &>/dev/null || mise install perl
  mise exec perl -- cpanm --version &>/dev/null 2>&1 || \
    mise exec perl -- curl -L https://cpanmin.us | mise exec perl -- perl - App::cpanminus
  mise exec perl -- perl -MGraph::Easy -e1 2>/dev/null || mise exec perl -- cpanm Graph::Easy
else
  # Fallback: system package manager
  echo "💡 Tip: Install mise for unified tool management: curl https://mise.run | sh"
  case "$(uname -s)" in
    Darwin) PM="brew" ;;
    Linux)  PM="apt" ;;
    *)      echo "ERROR: Unsupported OS"; exit 1 ;;
  esac
  command -v $PM &>/dev/null || { echo "ERROR: $PM not installed"; exit 1; }
  command -v cpanm &>/dev/null || { [ "$PM" = "apt" ] && sudo apt install -y cpanminus || brew install cpanminus; }
  perl -MGraph::Easy -e1 2>/dev/null || cpanm Graph::Easy
fi

# Verify graph-easy is in PATH and functional
command -v graph-easy &>/dev/null || {
  echo "ERROR: graph-easy not in PATH after installation"
  exit 1
}
# Test actual functionality (--version exits with code 2, unreliable)
echo "[Test] -> [OK]" | graph-easy &>/dev/null && echo "✓ graph-easy ready"
PREFLIGHT_EOF
```

---

## Part 1: DSL Syntax

### Basic Elements

```
# Nodes (square brackets)
[Node Name]

# Edges (arrows)
[A] -> [B]

# Labeled edges
[A] -- label --> [B]

# Bidirectional
[A] <-> [B]

# Chain
[A] -> [B] -> [C]
```

### Groups (Containers)

```
# Named group with dashed border
( Group Name:
  [Node A]
  [Node B]
)

# Nested connections
( Before:
  [Old System]
)
( After:
  [New System]
)
[Before] -> [After]
```

### Node Labels

```
# Custom label (different from ID)
[db] { label: "PostgreSQL Database"; }

# ASCII markers for visual distinction INSIDE boxes
# (emojis break box alignment - use ASCII markers instead)
[deleted] { label: "[x] Old Component"; }
[added] { label: "[+] New Component"; }
[warning] { label: "[!] Deprecated"; }
[success] { label: "[OK] Passed"; }
```

**Character rules for nodes:**

- Graphical emojis (🚀 💡 ✅ ❌) - NEVER (double-width breaks box alignment)
- Unicode symbols (✓ ✗ ⚠ → ←) - OK (single-width, safe)
- ASCII markers ([x] [+] [!] :) ) - ALWAYS safe (monospace)

Use `graph { label: "..."; }` for graphical emojis in title/legend.

**Example: Emoji breaks alignment (DON'T DO THIS)**

```
# BAD - emoji inside node
[rocket] { label: "🚀 Launch"; }
```

Renders broken:

```
┌────────────┐
│ 🚀 Launch  │   <-- box edge misaligned due to double-width emoji
└────────────┘
```

**Example: ASCII marker preserves alignment (DO THIS)**

```
# GOOD - ASCII marker inside node
[rocket] { label: "[>] Launch"; }
```

Renders correctly:

```
┌────────────┐
│ [>] Launch │
└────────────┘
```

**Example: Emoji safe in graph title (OK)**

```
# OK - emoji in graph label (outside boxes)
graph { label: "🚀 Deployment Pipeline"; flow: east; }
[Build] -> [Test] -> [Deploy]
```

Renders correctly (emoji in title, not in boxes):

```
        🚀 Deployment Pipeline

┌───────┐     ┌──────┐     ┌────────┐
│ Build │ --> │ Test │ --> │ Deploy │
└───────┘     └──────┘     └────────┘
```

### Flow Direction (MANDATORY: Always specify)

```
# MANDATORY: Always specify flow direction explicitly
graph { flow: south; }   # Top-to-bottom (architecture, decisions)
graph { flow: east; }    # Left-to-right (pipelines, sequences)
```

Never rely on default flow - explicit is clearer.

### Graph Title and Legend (Outside Boxes - Emojis Safe Here)

Emojis break alignment INSIDE boxes but are SAFE in graph titles/legends.

**Emoji Selection Guide** - Choose emoji that matches diagram purpose:

| Diagram Type             | Emoji | Example Title                |
| ------------------------ | ----- | ---------------------------- |
| Migration/Change         | 🔄    | `"🔄 Database Migration"`    |
| Deployment/Release       | 🚀    | `"🚀 Deployment Pipeline"`   |
| Data Flow                | 📊    | `"📊 Data Ingestion Flow"`   |
| Security/Auth            | 🔐    | `"🔐 Authentication Flow"`   |
| Error/Failure            | ⚠️    | `"⚠️ Error Handling"`        |
| Decision/Branch          | 🔀    | `"🔀 Routing Decision"`      |
| Architecture             | 🏗️    | `"🏗️ System Architecture"`   |
| Network/API              | 🌐    | `"🌐 API Integration"`       |
| Storage/Database         | 💾    | `"💾 Storage Layer"`         |
| Monitoring/Observability | 📡    | `"📡 Monitoring Stack"`      |
| Hook/Event               | 🪝    | `"🪝 Hook Flow"`             |
| Before/After comparison  | ⏮️/⏭️ | `"⏮️ Before"` / `"⏭️ After"` |

```
# Title with semantic emoji
graph { label: "🚀 Deployment Pipeline"; flow: east; }

# Title with legend (multiline using \n)
graph { label: "🪝 Hook Flow\n──────────\n✓ Allow  ✗ Deny  ⚠ Warn"; flow: south; }
```

**Rendered:**

```
Hook Flow
 ──────────
✓ Allow ✗ Deny ⚠ Warn

   ╭───────╮
   │ Start │
   ╰───────╯
```

**Rule**: Emojis ONLY in `graph { label: "..."; }` - NEVER inside `[ node ]`

### Node Styling (Best Practices)

```
# Rounded corners for start/end nodes
[ Start ] { shape: rounded; }
[ End ] { shape: rounded; }

# Double border for emphasis
[ Critical Step ] { border: double; }

# Bold border for important nodes
[ Key Decision ] { border: bold; }

# Dotted border for optional/skippable
[ Optional ] { border: dotted; }

# Multiline labels with \n
[ Hook Input\n(stdin JSON) ]
```

**Rendered examples:**

```
╭─────────╮              ┌─────────┐
│ Rounded │              │ Default │
╰─────────╯              └─────────┘

╔═════════╗              ┏━━━━━━━━━┓
║ Double  ║              ┃  Bold   ┃
╚═════════╝              ┗━━━━━━━━━┛
```

> **Note:** Dotted borders (`{ border: dotted; }`) use `⋮` characters that render inconsistently on GitHub. Use sparingly.

### Edge Styles

```
[ A ] -> [ B ]      # Solid arrow (default)
[ A ] ..> [ B ]     # Dotted arrow
[ A ] ==> [ B ]     # Bold/double arrow
[ A ] - -> [ B ]    # Dashed arrow
[ A ] -- label --> [ B ]  # Labeled edge
```

---

## Part 2: Common Diagram Patterns

### Migration (Before → After)

```
graph { flow: south; }
[Before] -- migrate --> [After]
```

### Multi-Component System

```
graph { flow: south; }
[A] -> [B] -> [C]
[B] -> [D]
```

### Pipeline (Left-to-Right)

```
graph { flow: east; }
[Input] -> [Process] -> [Output]
```

### Decision with Options

```
graph { flow: south; }
[Decision] -> [Option A]
[Decision] -> [Option B]
```

### Grouped Components

```
( Group:
  [Component 1]
  [Component 2]
)
[External] -> [Component 1]
```

### Bidirectional Flow

```
[Client] <-> [Server]
[Server] -> [Database]
```

---

## Part 3: Rendering

### Command (MANDATORY: Always use boxart)

```bash
# MANDATORY: Always use --as=boxart for clean output
graph-easy --as=boxart << 'EOF'
graph { flow: south; }
[A] -> [B] -> [C]
EOF
```

**Never use** `--as=ascii` - it produces ugly `+--+` boxes instead of clean `┌──┐` lines.

### Output Modes

| Mode     | Command       | Usage                                |
| -------- | ------------- | ------------------------------------ |
| `boxart` | `--as=boxart` | MANDATORY - clean Unicode lines      |
| `ascii`  | `--as=ascii`  | NEVER USE - ugly output, legacy only |

### Validation Workflow

```bash
# 1. Write DSL to heredoc
# 2. Render with boxart
graph-easy --as=boxart << 'EOF'
[Your] -> [Diagram] -> [Here]
EOF

# 3. Review output
# 4. Iterate if needed
# 5. Copy final ASCII to ADR
```

---

## Part 4: Embedding in ADR

### Markdown Format (MANDATORY: Always Include Source)

**CRITICAL**: Every rendered diagram MUST be followed by a collapsible `<details>` block containing the graph-easy source code. This is non-negotiable for:

- **Reproducibility**: Future maintainers can regenerate the diagram
- **Editability**: Source can be modified and re-rendered
- **Auditability**: Changes to diagrams are trackable in git diffs

````markdown
## Architecture

```
┌──────────┐     ┌──────────┐     ┌──────────┐
│  Before  │ ──> │  After   │ ──> │ Database │
└──────────┘     └──────────┘     └──────────┘
```

<details>
<summary>graph-easy source</summary>

```
graph { flow: east; }
[Before] -> [After] -> [Database]
```

</details>
````

**The `<details>` block is MANDATORY** - never embed a diagram without its source.

### GFM Collapsible Section Syntax

GitHub Flavored Markdown supports HTML `<details>` and `<summary>` tags for collapsible sections. Key syntax rules:

**Structure:**

```html
<details>
  <summary>Click to expand</summary>

  <!-- BLANK LINE REQUIRED HERE -->
  Content goes here (Markdown supported)
  <!-- BLANK LINE REQUIRED HERE -->
</details>
```

**Critical rules:**

1. **Blank lines required** - Must have empty line after `<summary>` and before `</details>` for Markdown to render
2. **No indentation** - `<details>` and `<summary>` must be at column 0 (no leading spaces)
3. **Summary is clickable label** - Text in `<summary>` appears as the collapsed header
4. **Markdown inside works** - Code blocks, headers, lists all render correctly inside

**Optional: Default expanded:**

```html
<details open>
  <summary>Expanded by default</summary>

  Content visible on page load
</details>
```

**Common mistake (Markdown won't render):**

```html
<details>
  <summary>Broken</summary>
  No blank line - this won't render as Markdown!
</details>
```

**References:**

- [GitHub Docs: Collapsed sections](https://docs.github.com/en/get-started/writing-on-github/working-with-advanced-formatting/organizing-information-with-collapsed-sections)
- [GFM details/summary gist](https://gist.github.com/scmx/eca72d44afee0113ceb0349dd54a84a2)

### File Organization

No separate asset files needed - diagram is inline in the markdown.

### Regeneration

If ADR changes, regenerate by running the source through graph-easy again:

```bash
# Extract source from <details> block, pipe through graph-easy
graph-easy --as=boxart << 'EOF'
# paste source here
EOF
```

---

## Reference: Monospace-Safe Symbols

**Avoid emojis** - they have variable width and break box alignment on GitHub.

### Status Markers

| Meaning            | Marker |
| ------------------ | ------ |
| Added/New          | `[+]`  |
| Removed/Deleted    | `[x]`  |
| Changed/Updated    | `[*]`  |
| Warning/Deprecated | `[!]`  |
| Deferred/Pending   | `[~]`  |
| Current/Active     | `[>]`  |
| Optional           | `[?]`  |
| Locked/Fixed       | `[=]`  |

### Box Drawing (U+2500-257F)

```
─ │ ┌ ┐ └ ┘ ├ ┤ ┬ ┴ ┼   (light)
═ ║ ╔ ╗ ╚ ╝ ╠ ╣ ╦ ╩ ╬   (double)
```

### Arrows & Pointers

```
→ ← ↑ ↓              (arrows)
∨ ∧                  (logic - graph-easy uses these)
< > ^ v              (ASCII arrows)
```

### Shapes & Bullets

```
• ○ ●                (bullets)
□ ■                  (squares)
◇ ◆                  (diamonds)
```

### Math & Logic

```
× ÷ ± ≠ ≤ ≥ ∞       (math)
∧ ∨ ¬                (logic)
```

## Reference: Common Patterns

```
# Vertical flow (architecture)
graph { flow: south; }

# Horizontal flow (pipeline)
graph { flow: east; }

# Labeled edge
[A] -- label text --> [B]

# Group with border
( Group Name:
  [Node A]
  [Node B]
)

# Custom node label
[id] { label: "Display Name"; }
```

---

## Graph Label (MANDATORY: EVERY diagram MUST have emoji + title)

**WARNING**: This is the most commonly forgotten requirement. Diagrams without labels are invalid.

### Correct Example

```
graph { label: "🔄 Database Migration"; flow: south; }
[Old DB] -> [New DB]
```

### Anti-Pattern (INVALID - DO NOT DO THIS)

```
graph { flow: south; }
[Old DB] -> [New DB]
```

**Why this is wrong**: Missing `label:` with emoji. The preflight validator will **BLOCK** any ADR containing diagrams without `graph { label: "emoji ..."; }`.

---

## Mandatory Checklist (Before Rendering)

### Graph-Level (MUST have)

- [ ] **`graph { label: "🚀 Title"; }`** - semantic emoji + title (MOST FORGOTTEN - check first!)
- [ ] `graph { flow: south; }` or `graph { flow: east; }` - explicit direction
- [ ] Command uses `--as=boxart` - NEVER `--as=ascii`

### Embedding (MUST have - non-negotiable)

- [ ] **`<details>` block with source** - EVERY diagram MUST have collapsible source code block
- [ ] Format: rendered diagram in ` ``` ` block, followed immediately by `<details><summary>graph-easy source</summary>` with source in ` ``` ` block
- [ ] Never commit a diagram without its reproducible source

### Node Styling (Visual hierarchy)

- [ ] Start/end nodes: `{ shape: rounded; }` - entry/exit points
- [ ] Critical/important nodes: `{ border: double; }` or `{ border: bold; }`
- [ ] Optional/skippable nodes: `{ border: dotted; }`
- [ ] Default nodes: no styling (standard `┌──┐` border)
- [ ] Long labels use `\n` for multiline - max ~15 chars per line

### Edge Styling (Semantic meaning)

- [ ] Main/happy path: `->` solid arrow
- [ ] Conditional/alternate: `..>` dotted arrow
- [ ] Emphasized/critical: `==>` bold arrow
- [ ] Edge labels are SHORT (1-3 words): `-- YES -->`, `-- error -->`

### Character Safety (Alignment)

- [ ] NO graphical emojis inside nodes (🚀 💡 ✅ ❌ break alignment)
- [ ] Unicode symbols OK inside nodes (✓ ✗ ⚠ are single-width)
- [ ] ASCII markers ALWAYS safe ([x] [+] [!] [OK])
- [ ] Graphical emojis ONLY in `graph { label: "..."; }` title

### Structure (Organization)

- [ ] Groups `( Name: ... )` used for logical clustering when 4+ related nodes
- [ ] Node IDs short, labels descriptive: `[db] { label: "PostgreSQL"; }`
- [ ] No more than 7-10 nodes per diagram (split if larger)

## Success Criteria

### Correctness

1. **Parses without error** - graph-easy accepts the DSL
2. **Renders cleanly** - no misaligned boxes or broken lines
3. **Matches content** - all key elements from description represented
4. **Source preserved (MANDATORY)** - EVERY diagram MUST have `<details>` block with graph-easy DSL source immediately after the rendered output

### Aesthetics

1. **Uses boxart** - clean Unicode lines `┌──┐`, not ASCII `+--+`
2. **Visual hierarchy** - start/end rounded, important bold/double, optional dotted
3. **Consistent styling** - same border style = same semantic meaning throughout
4. **Readable labels** - multiline with `\n`, no truncation
5. **Clear flow** - direction matches natural reading (top-down or left-right)

### Comprehensiveness

1. **Semantic emoji in title** - emoji consciously chosen to match diagram purpose (see Emoji Selection Guide)
2. **Legend if needed** - multiline title with `\n` for complex diagrams
3. **Edge semantics** - solid=normal, dotted=conditional, bold=critical
4. **Logical grouping** - related nodes in `( Group: ... )` containers

## Troubleshooting

| Issue               | Cause                    | Solution                                      |
| ------------------- | ------------------------ | --------------------------------------------- |
| `command not found` | graph-easy not installed | Run preflight check                           |
| Misaligned boxes    | Used `--as=ascii`        | Always use `--as=boxart`                      |
| Box border broken   | Graphical emoji in node  | Remove 🚀💡, use ✓✗ or [x][+]                 |
| Nodes overlap       | Too complex              | Split into multiple diagrams (max 7-10 nodes) |
| Edge labels cut off | Label too long           | Shorten to 1-3 words                          |
| No title showing    | Wrong syntax             | Use `graph { label: "Title"; flow: south; }`  |
| Weird layout        | No flow direction        | Add `graph { flow: south; }` or `flow: east`  |
| Parse error         | Special chars in node    | Escape or simplify node names                 |

## Resources

- [Graph::Easy on CPAN](https://metacpan.org/dist/Graph-Easy)
- [Graph::Easy Manual](http://bloodgate.com/perl/graph/manual/)
- [Graph::Easy GitHub](https://github.com/ironcamel/Graph-Easy)
