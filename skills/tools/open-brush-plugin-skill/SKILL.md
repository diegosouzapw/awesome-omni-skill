---
name: open-brush-plugin-skill
description: Create and modify Lua plugins for Open Brush with full API documentation access. Use when the user wants to create Open Brush plugins, work with Lua scripts for Open Brush, or asks about Open Brush scripting API, Lua functions, or plugin development.
---

# Open Brush Lua Plugin Development

This skill helps you create and modify Lua plugins for Open Brush, a VR painting application. You have access to the complete Open Brush Lua API documentation.

## Quick Start

### Plugin File Structure

**CRITICAL: Plugin Naming Convention**
Plugins MUST be named with the correct prefix or Open Brush won't recognize them:
- **PointerScript**.PluginName.lua (e.g., PointerScript.Wobble.lua)
- **SymmetryScript**.PluginName.lua (e.g., SymmetryScript.ManyAround.lua)
- **ToolScript**.PluginName.lua (e.g., ToolScript.Circle.lua)
- **BackgroundScript**.PluginName.lua (e.g., BackgroundScript.Lines.lua)

A basic plugin structure:

```lua
-- Plugin metadata (in Settings table)
Settings = {
  description = "What this plugin does"
}

-- Optional parameters (exposed as UI widgets)
Parameters = {
  speed = {label = "Speed", type = "float", min = 1, max = 100, default = 50},
  count = {label = "Count", type = "int", min = 1, max = 10, default = 5},
  color = {label = "Color", type = "color", default = Color.red},
  enabled = {label = "Enabled", type = "toggle", default = true},
  name = {label = "Name", type = "text", default = "Untitled"},
  mode = {label = "Mode", type = "list", items = {"A", "B", "C"}, default = "A"},
}

-- Main function (required) - runs every frame
function Main()
  -- Your plugin logic here
  -- Return value determines plugin type (Transform, Path, PathList, or nothing)
end

-- Optional: runs once when plugin starts
function Start()
  -- Setup code
end

-- Optional: runs once when plugin ends
function End()
  -- Cleanup code
end
```

### Where to Find Information

**IMPORTANT: All documentation files are located inside this skill's directory.**
The skill directory is typically at: `~/.claude/skills/open-brush-plugin-skill/` (or `C:\Users\USERNAME\.claude\skills\open-brush-plugin-skill\` on Windows)

**All paths below are relative to the skill directory, NOT the user's project directory.**

**When creating a new plugin:**
1. Read `references/instructions.md` for critical API syntax rules and plugin structure
2. Check `references/examples/` directory for similar working code (PointerScript.*, SymmetryScript.*, ToolScript.*, BackgroundScript.*)
3. Read `references/guides/example-plugins/` for explanations of what example plugins do
4. Read `references/guides/writing-plugins/` for step-by-step tutorials on each plugin type

**When you need API details:**
1. Check `references/lua-modules/__autocomplete.lua` for complete list of available classes/methods/properties
2. Read specific files in `references/api-docs/` directory: app.md, brush.md, vector3.md, path.md, etc.

**External reference** (for context only): https://icosa.gitbook.io/open-brush-plugin-scripting-docs

## Instructions for AI Agents

### Critical API Rules

**MOST IMPORTANT**: Only use APIs listed in `references/lua-modules/__autocomplete.lua`. NEVER invent methods or properties. If unsure, say "I'm unsure - let me check the API documentation" and read `__autocomplete.lua`. Favor the Open Brush API over standard Lua library functions.

**Core Syntax Rules**:
- Constructors: `ClassName:New(...)` (capital N, always use colon)
- Properties: `object.property` (dot notation)
- Methods: `object:method()` (colon notation)
- API classes start with capital letters (e.g., `Vector3`, `Transform`, `Path`)
- Methods do NOT return self - no method chaining
- Use `Math` (capital M) library, not lua's `math`

**Plugin Structure**:
All plugins define:
- `Main()` - called every frame (required)
- `Start()` - called when plugin begins (optional)
- `End()` - called when plugin ends (optional)
- `Settings` table - plugin metadata (optional)
- `Parameters` table - UI sliders for user input (optional)

```lua
Settings = {
  description = "Plugin description",
  space = "pointer" -- or "canvas", "world", etc.
}

Parameters = {
  speed = {label = "Speed", type = "float", min = 1, max = 100, default = 50},
  count = {label = "Count", type = "int", min = 1, max = 10, default = 5},
  color = {label = "Color", type = "color", default = Color.red},
  enabled = {label = "Enabled", type = "toggle", default = true},
  name = {label = "Name", type = "text", default = "Untitled"},
  mode = {label = "Mode", type = "list", items = {"A", "B", "C"}, default = "A"},
}
-- Access as: Parameters.speed, Parameters.count, etc.
```

**Four Plugin Types**:

1. **Pointer Plugin** - Returns single `Transform`
   - Modifies the user's primary brush pointer position while user draws

2. **Symmetry Plugin** - Returns `Path` or list of `Transform`
   - Controls multiple brush pointers while the user draws
   - Each transform generates one additional stroke
   - Important to understand its special use of coordinate spaces (especially the symmetry widget)
   - Can be combined with Pointer plugins
   - Overrides and replaces mirror or multimirror symmetry modes

3. **Tool Plugin** - Returns `Path` or `PathList`
   - Generates a complete stroke or strokes in one action based on the users actions
   - Return the stroke data as Path or PathList in Main() - do not use Path:Draw() etc directly
   - Typically triggered on button press/release
   - Active mirror, multimirror or symmetry plugin modes are automatically applied to the output

4. **Background Plugin** - Returns nothing
   - Runs autonomously every frame
   - Draws strokes using explicit `Draw()` methods

**Important Constraints**:
- Brush type cannot change during a stroke - only between strokes
- Brush size cannot change during a stroke but modifying pressure within a stroke often affects size
- Brush color cannot change during a stroke but color overrides can be applied within a stroke.
- Understand coordinate spaces - default varies by plugin type (check Settings.space)
- Transform scale component affects stroke width/thickness

**For complete details, examples, and edge cases, read `references/instructions.md`**

### Common Gotchas

1. **Coordinate Spaces**: By default, Pointer/Tool plugins use `space="pointer"` (relative to brush hand) while Symmetry plugins use the symmetry widget as origin. Override with `Settings.space="canvas"` or `Settings.space="pointer"`.

2. **Path Smoothing**: Open Brush smooths paths for hand-drawn strokes. For geometric shapes, add extra points with `Path:SubdivideSegments(n)` to prevent rounding.

3. **Multiple Active Plugins**: You can run multiple Background plugins simultaneously, but only one of each other type (Pointer/Symmetry/Tool). You can combine the effects of Pointer and Symmetry. Or Symmetry and Tool.

### Plugin Development Workflow

When helping users with Open Brush Lua plugins:

1. **Check example plugins for similar functionality** - Consult both `references/examples/` (actual code) and `references/guides/example-plugins/` (explanations). The examples demonstrate working patterns.
2. **Verify API calls** - Check `references/lua-modules/__autocomplete.lua` before using any API methods or properties
3. **Ask clarifying questions** about what the plugin should do before writing code
4. **Provide complete, working examples** that users can copy and test
5. **Consider performance** - Warn if operations might be slow (e.g., processing thousands of strokes every frame)

## Example Plugin Templates

### Simple Stroke Counter (Background Plugin)
```lua
Settings = {
  description = "Counts total strokes in sketch"
}

local hasCountedOnce = false

function Main()
  if not hasCountedOnce then
    local sketch = App.sketch()
    local strokes = sketch:strokes()
    local count = strokes:count()
    print("Total strokes: " .. count)
    hasCountedOnce = true
  end
end
```

### Position Logger (Background Plugin)
```lua
Settings = {
  description = "Logs user position every 2 seconds"
}

local timer = Timer:New()

function Start()
  timer:Start()
end

function Main()
  if timer:Elapsed() > 2.0 then
    local pos = User.position()
    print(string.format("User at: %.2f, %.2f, %.2f", pos.x, pos.y, pos.z))
    timer:Reset()
  end
end
```

## Additional Resources

- **API Documentation Repository**: https://github.com/icosa-foundation/open-brush-plugin-scripting-docs
- **Main Documentation Repository**: https://github.com/icosa-foundation/open-brush-docs
- **Example Plugins**: Browse the `LuaScriptExamples` directory in the API docs repo for real-world plugin examples
