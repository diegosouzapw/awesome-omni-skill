---
name: ae-mcp
description: After Effects automation through Model Context Protocol. Use when users request creating, modifying, or animating content in Adobe After Effects, including compositions, layers, keyframes, effects, expressions, text animations, lower thirds, title cards, logo reveals, or any motion graphics tasks. Trigger on phrases like "create in After Effects", "animate this", "make a motion graphic", "add keyframes", "apply effects", or when users mention AE-specific concepts like compositions, precomps, expressions, or time remapping.
---

# After Effects MCP Skill

## Overview

This skill enables automation of Adobe After Effects through MCP tools. It covers project management, composition creation, layer manipulation, animation, effects, and expressions.

## Critical Property Path Formats

After Effects properties require exact naming. Common patterns:

**Transform properties:**
- Position: `"Position"`
- Scale: `"Scale"`
- Rotation: `"Rotation"` or `"Z Rotation"`
- Opacity: `"Opacity"`
- Anchor Point: `"Anchor Point"`

**Text properties:**
- Source Text: `"Source Text"`
- For text animators: Use relative paths like `"Opacity"` within animator context

**Shape layer properties:**
- Position within shape group: `"Contents.Shape 1.Transform.Position"`
- Scale within shape group: `"Contents.Shape 1.Transform.Scale"`

**Effect properties:**
- Effect parameter: `"Effects.Gaussian Blur.Blurriness"`

## Value Formats by Property Type

Different properties require specific value formats:

**Position**: `{"x": 960, "y": 540}` or `[960, 540]`
**Scale**: `[100, 100]` (percentages)
**Rotation**: `45` (degrees)
**Opacity**: `100` (percentage, 0-100)
**Color (RGB 0-1 range)**: `{"r": 1.0, "g": 0.5, "b": 0.0}`
**Source Text**: String value

## Common Workflows

### 1. Creating a Basic Animated Scene

```
1. Create project (optional name)
2. Create composition with specs (width, height, duration, frameRate)
3. Add layers (solid, shape, text, AV)
4. Set initial properties (position, scale, opacity)
5. Add keyframes at different times
6. Apply easing for smooth motion
7. Save project
```

### 2. Text Animation Pattern

```
1. Add text layer with content
2. Set text properties (fontSize, color, position, justification)
3. Use create_text_animator for preset animations:
   - typewriter
   - fadeInChars
   - scaleInChars
   - slideInChars
   - randomize
   - wave
4. Or manually add keyframes to text properties
```

### 3. Effect Application Pattern

```
1. Apply effect to layer using apply_effect
2. Modify effect properties using modify_effect_properties
3. Or use apply_effect_template for preset effects:
   - gaussianBlur, directionalBlur, glassBlur
   - curves, colorBalance, brightnessContrast, vibrance
   - glow, dropShadow, vignette
   - cinematicLook, vhsRetro, neonGlow, filmGrain
   - chromaticAberration, duotone
```

### 4. Expression-Based Animation

```
1. Set expression on property using set_expression
2. Common expression templates available via apply_expression_template:
   - wiggle, wiggleSmooth, wiggleFadeIn, wiggleFadeOut
   - loopCycle, loopPingpong, loopOffset, loopContinue
   - time, clock, countdown, frameNumber
   - matchPosition, offsetPosition, inverseRotation
   - followPath, bounce, inertia, overshoot, springy
3. Link properties between layers using link_properties
```

## Key Tool Behaviors and Gotchas

### Keyframe Setting

**set_keyframe** - Basic keyframe setting:
- Use for simple keyframes without easing
- Value format must match property type
- Time in seconds (not frames)

**set_keyframe_advanced** - With easing control:
- inType/outType: `LINEAR`, `BEZIER`, `HOLD`
- inEase/outEase: `{speed: number, influence: number}`
- Use for smooth, professional motion

**apply_easy_ease** - Quick easing application:
- Apply to existing keyframes
- Type: `IN`, `OUT`, `BOTH`
- Can target specific keyframe by index or all keyframes

### Layer References

Layers can be identified by:
- `layerName`: Preferred for clarity
- `layerIndex`: 1-based (1 = top layer)

When modifying layers, always verify layer exists first via list_layers or get_layer_info.

### Composition Management

**Creating compositions:**
- Specify complete dimensions and timing
- backgroundColor uses 0-1 RGB values
- frameRate commonly: 24, 30, 60

**Modifying compositions:**
- Can change any parameter after creation
- Changes affect all nested instances

### Shape Layer Limitations

Shape layers have complex property hierarchies:
- Must navigate through Contents group
- Transform properties nested within shape groups
- Some properties may be hidden initially

When shape operations fail with "property is hidden" error, the layer structure may need manual adjustment or different approach.

### Pre-built Templates

**Lower thirds** (create_lower_third):
- Styles: modern, corporate, news, minimal, social
- Position: bottomLeft, bottomRight, bottomCenter
- Automatically creates text and animated background

**Title cards** (create_title_card):
- Styles: cinematic, documentary, social, minimal
- Full-screen animated title sequences

**Transitions** (create_transition):
- Types: wipe_left, wipe_right, wipe_up, wipe_down, dissolve, push, slide, zoom
- Duration and easing customizable

**Logo reveals** (create_logo_reveal):
- Requires imported logo footage
- Styles: fade, scale, slide, spin, glitch, particle

### Expression Best Practices

1. **Use templates when possible**: apply_expression_template provides tested, working expressions
2. **Property references**: Use `thisLayer.transform.position` not just `position`
3. **Time-based**: Use `time` variable for continuous animation
4. **Value references**: Use `value` to reference current property value

### Common Errors and Solutions

**"Property not found"**
- Verify exact property name spelling
- Check if property path includes parent groups
- Use get_layer_info to see available properties

**"Value is not an array"**
- Check value format matches property type
- Scale requires array: `[100, 100]`
- Position can use object or array

**"Property is hidden"**
- Shape layer properties may be hidden initially
- Try different approach or manual layer setup
- Some properties only accessible after layer creation

**"File couldn't be opened for writing"**
- Provide full absolute path for save operations
- Mac: `/Users/username/path/to/file.aep`
- Windows: `C:/Users/username/path/to/file.aep`

## Project Organization Best Practices

1. **Layer naming**: Use descriptive names for easy reference
2. **Precomposing**: Group related layers into precomps for organization
3. **Markers**: Add markers for timing references and notes
4. **Work area**: Set work area to focus rendering on specific sections
5. **Project structure**: Organize footage into folders using organize_project_items

## Animation Timing Guidelines

**Duration by content type:**
- UI animations: 0.3-0.5 seconds
- Text reveals: 0.5-1.5 seconds
- Logo animations: 1-3 seconds
- Full title cards: 3-5 seconds
- Transitions: 0.5-1.5 seconds

**Easing recommendations:**
- Natural motion: Easy Ease (both)
- Bouncy entrance: Ease Out with overshoot
- Snappy exit: Ease In
- Continuous loop: Linear

## Common Use Cases

### Social Media Graphics
1. Create 1080x1920 (portrait) or 1920x1080 (landscape) comp
2. Add branded colors and text
3. Use quick animations (0.3-0.8s)
4. Apply modern, bold styles

### Corporate Videos
1. Create 1920x1080 comp at 30fps
2. Use corporate/professional templates
3. Longer, smoother animations (1-2s)
4. Professional color grading effects

### Motion Graphics Loops
1. Set looping expressions on properties
2. Use modulo operations for seamless loops
3. Match first and last frames
4. Time remap for speed variations

### Explainer Animations
1. Build scene sequentially
2. Use text animators for callouts
3. Coordinate timing with markers
4. Parent layers for grouped motion

## Performance Considerations

- Complex expressions can slow preview
- Disable expressions during editing if needed
- Use proxies for heavy footage
- Reduce composition complexity when possible
- Pre-render complex sections

## Integration Tips

When working with imported footage:
1. Use import_footage for single files
2. Use import_folder for batches
3. Replace footage items with replace_footage
4. Find missing footage with find_missing_footage
5. Collect files before sharing: collect_files

## Quick Reference: Most Used Functions

**Essential:**
- create_project, save_project
- create_composition
- add_text_layer, add_shape_layer, add_solid_layer
- set_keyframe, apply_easy_ease
- save_project

**Animation:**
- set_keyframe_advanced
- apply_expression_template
- create_text_animator

**Effects:**
- apply_effect_template
- modify_effect_properties

**Templates:**
- create_lower_third
- create_title_card
- create_transition

**Organization:**
- list_compositions, list_layers
- get_composition_info, get_layer_info
- precompose_layers