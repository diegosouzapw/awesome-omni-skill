---
name: remotion-visual-debug
description: Visual debugging for Remotion video projects. Use when working with Remotion compositions to verify animations, layouts, positioning, and timing by rendering frame samples. Essential for self-correcting visual issues that aren't apparent from code alone.
allowed-tools: Bash(node *), Read
metadata:
  tags: remotion, video, debugging, visual, frames, animation
---

# Remotion Visual Debugging

You cannot natively view video files, but you CAN read rendered JPEG frames. This skill enables visual debugging of Remotion projects by extracting frame samples and analyzing them.

## When to Use Visual Debugging

Use this skill when:
- **Layout issues**: Verifying element positioning, spacing, alignment
- **Animation timing**: Checking animation sequences, transitions, easing
- **Text rendering**: Verifying font rendering, text overflow, truncation
- **Responsive design**: Checking how content fits at different points
- **Color/styling**: Verifying gradients, shadows, opacity
- **Scene transitions**: Checking handoffs between scenes
- **Before delivery**: Final verification before rendering full video

## The Render Script

The skill bundles a universal Remotion frame extractor. Use the cross-platform wrapper which automatically sets up the project:

```bash
node ~/.claude/skills/remotion-visual-debug/scripts/render-frames.mjs [options]
```

The wrapper automatically copies `render-stills.mjs` to your project directory (if needed) and runs it. Works on Windows, macOS, and Linux.

### Quick Reference

```bash
# List available compositions
node ~/.claude/skills/remotion-visual-debug/scripts/render-frames.mjs --list

# Extract 9 frames across entire video (default)
node ~/.claude/skills/remotion-visual-debug/scripts/render-frames.mjs -n 9 --grid --labels

# Analyze specific animation range (frames 100-200, 12 samples)
node ~/.claude/skills/remotion-visual-debug/scripts/render-frames.mjs --from 100 --to 200 -n 12 --grid --labels

# Analyze by timestamp (5s to 10s)
node ~/.claude/skills/remotion-visual-debug/scripts/render-frames.mjs --from 5000 --to 10000 --ms -n 9 --grid --labels

# Specific frames only
node ~/.claude/skills/remotion-visual-debug/scripts/render-frames.mjs -f 0,30,60,90,120 --grid --labels
```

**Alias tip**: Add to your shell config for shorter commands:

```bash
# Bash/Zsh (Linux/macOS)
alias remotion-debug="node ~/.claude/skills/remotion-visual-debug/scripts/render-frames.mjs"

# PowerShell (Windows)
function remotion-debug { node ~/.claude/skills/remotion-visual-debug/scripts/render-frames.mjs $args }
```

### Output

- Individual frames: `out/stills/frame-XXXX.jpeg`
- Grid composite: `out/stills/grid.jpg` (when `--grid` used)

## Visual Debugging Workflow

### Step 1: Understand the Composition

```bash
node ~/.claude/skills/remotion-visual-debug/scripts/render-frames.mjs --list
```

This shows available compositions with dimensions, duration, and FPS.

### Step 2: Render Overview Grid

For initial overview, render 9-12 frames across the entire video:

```bash
node ~/.claude/skills/remotion-visual-debug/scripts/render-frames.mjs -n 9 --grid --labels
```

### Step 3: Read and Analyze the Grid

```
Read the file: out/stills/grid.jpg
```

The grid image shows frames with labels (frame number + timestamp). Analyze:
- Overall flow and pacing
- Scene transitions
- Layout consistency
- Any obvious visual issues

### Step 4: Deep Dive Problem Areas

If you spot issues in a specific time range, extract more frames from that area:

```bash
# If issue appears around frame 150, analyze frames 120-180 closely
node ~/.claude/skills/remotion-visual-debug/scripts/render-frames.mjs --from 120 --to 180 -n 12 --grid --labels
```

### Step 5: Verify Fixes

After making code changes, re-render the affected range to verify the fix.

## Script Options Reference

### Frame Selection (mutually exclusive)
| Option | Description | Example |
|--------|-------------|---------|
| `-f, --frames` | Specific frame numbers | `-f 0,30,60,90` |
| `-t, --timestamps` | Specific timestamps (ms) | `-t 0,1000,2000` |
| `-i, --interval` | Every N milliseconds | `-i 500` |
| `--from/--to -n` | Uniform distribution in range | `--from 0 --to 300 -n 9` |
| `--ms` | Interpret from/to as milliseconds | `--from 5000 --to 10000 --ms` |

### Grid Options
| Option | Description | Example |
|--------|-------------|---------|
| `--grid, -g` | Create grid composite | `--grid` (auto), `--grid 3` (3 cols), `--grid 3x2` |
| `--labels, -l` | Add frame/timestamp labels | `--labels` |
| `--gap` | Gap between images (px) | `--gap 8` |

### Output Options
| Option | Description | Example |
|--------|-------------|---------|
| `-s, --scale` | Scale factor (default 0.5) | `-s 1` (full res) |
| `-q, --quality` | JPEG quality 0-100 | `-q 90` |
| `-o, --output` | Output directory | `-o ./debug-frames` |

### Project Options
| Option | Description | Example |
|--------|-------------|---------|
| `-p, --project` | Project directory | `-p /path/to/project` |
| `-c, --composition` | Composition ID | `-c MyScene` |
| `-e, --entry` | Entry point file | `-e src/Video.tsx` |

## Common Debugging Scenarios

### Animation Not Smooth
Extract many frames from the animation range to see individual steps:
```bash
node ~/.claude/skills/remotion-visual-debug/scripts/render-frames.mjs --from 50 --to 80 -n 15 --grid --labels
```

### Element Positioning Wrong
Render at full scale for pixel-accurate analysis:
```bash
node ~/.claude/skills/remotion-visual-debug/scripts/render-frames.mjs -f 100 -s 1 -q 95
```

### Scene Transition Issue
Focus on the transition frames:
```bash
# If Scene 1 ends at frame 150, check 140-160
node ~/.claude/skills/remotion-visual-debug/scripts/render-frames.mjs --from 140 --to 160 -n 10 --grid --labels
```

### Text Rendering/Font Issues
Render specific frames at full resolution:
```bash
node ~/.claude/skills/remotion-visual-debug/scripts/render-frames.mjs -f 30,60,90 -s 1 --grid
```

## Dependencies

The script requires these packages (install in any Remotion project):
- `@remotion/renderer`
- `@remotion/bundler`
- `sharp`

```bash
npm install @remotion/renderer @remotion/bundler sharp
```

## Tips for Effective Visual Debugging

1. **Start with overview** - Always render a full-video grid first
2. **Use labels** - The `--labels` flag helps correlate frames with code
3. **Iterate quickly** - Bundle caching makes subsequent renders fast
4. **Check boundaries** - Scene transitions and animation start/end points
5. **Verify at scale** - Use `-s 1` for pixel-accurate checks when needed
6. **Compare before/after** - Render same frames before and after fixes
