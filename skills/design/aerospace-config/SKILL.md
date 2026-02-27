---
name: aerospace-config
description: Complete AeroSpace tiling window manager configuration assistant for macOS. Use for any AeroSpace configuration task including keybindings, workspace management, window rules, layouts (BSP, columns, rows, accordion), focus behavior, gaps, floating windows, app-specific settings, exec commands, or any window management customization. Also use for debugging and troubleshooting AeroSpace issues including log inspection, window state analysis, and diagnosing layout problems. This skill fetches fresh documentation for each request to ensure accurate, up-to-date guidance.
---

# AeroSpace Configuration Assistant

This skill provides comprehensive assistance for configuring AeroSpace, a tiling window manager for macOS. AeroSpace uses TOML for configuration and offers extensive customization for keyboard-driven window management, workspaces, layouts, and window behavior.

## Configuration Location

**Config File:** `/Users/alex/.config/aerospace/aerospace.toml`

Alternative locations: `~/.aerospace.toml`

## Core Principles

### ALWAYS Follow This Workflow

1. **Read Current Config**: Use Read tool to examine existing configuration
2. **Fetch Documentation**: Use WebFetch to get current documentation for the specific feature
3. **Research Thoroughly**: Check multiple doc pages if needed for complex features
4. **Implement Carefully**: Preserve existing config, validate TOML syntax
5. **Explain Clearly**: Document what changed and why
6. **Guide Testing**: Explain how to reload config (`aerospace reload-config`)

### Documentation Strategy

**Never embed documentation text.** Always use WebFetch to retrieve current information from the AeroSpace documentation.

AeroSpace documentation is comprehensive and continuously updated. The configuration system supports many options across window management, keybindings, and layouts.

## Full Configuration Capabilities

AeroSpace supports configuration across these comprehensive areas. For each request, fetch the relevant documentation:

### 1. Getting Started & Overview
- **Installation & Setup**: Initial configuration, first-time setup
  - https://nikitabobko.github.io/AeroSpace/guide
- **Default Config**: Understanding the default configuration
  - https://nikitabobko.github.io/AeroSpace/goodness#default-config

### 2. Keybindings & Modes
- **Key Bindings**: Main mode, custom modes, key syntax
  - https://nikitabobko.github.io/AeroSpace/guide#key-bindings
  - https://nikitabobko.github.io/AeroSpace/guide#binding-modes
- **Key Syntax**: Modifier keys, key names, key combinations
  - https://nikitabobko.github.io/AeroSpace/guide#key-syntax

### 3. Window Management
- **Tree Structure**: Understanding the window tree paradigm
  - https://nikitabobko.github.io/AeroSpace/guide#tree
- **Layouts**: BSP (Binary Space Partitioning), columns, rows, accordion, tiles
  - https://nikitabobko.github.io/AeroSpace/guide#layouts
- **Floating Windows**: Float rules, floating toggle
  - https://nikitabobko.github.io/AeroSpace/guide#floating-windows

### 4. Workspaces & Monitors
- **Workspaces**: Workspace configuration, assignment, switching
  - https://nikitabobko.github.io/AeroSpace/guide#workspaces
- **Multiple Monitors**: Monitor assignment, workspace-to-monitor mapping
  - https://nikitabobko.github.io/AeroSpace/guide#multiple-monitors

### 5. Application-Specific Rules
- **Window Detection Callbacks**: on-window-detected, app-specific behavior
  - https://nikitabobko.github.io/AeroSpace/guide#on-window-detected-callback
- **App Identifiers**: Bundle IDs, window titles for rules

### 6. Visual Appearance
- **Gaps**: Inner gaps, outer gaps, per-workspace gaps
  - https://nikitabobko.github.io/AeroSpace/guide#gaps

### 7. Commands Reference
- **All Commands**: Complete command reference for keybindings and exec
  - https://nikitabobko.github.io/AeroSpace/commands
- **Individual Commands**: (fetch specific command pages as needed)
  - https://nikitabobko.github.io/AeroSpace/commands#focus
  - https://nikitabobko.github.io/AeroSpace/commands#move
  - https://nikitabobko.github.io/AeroSpace/commands#resize
  - https://nikitabobko.github.io/AeroSpace/commands#workspace
  - https://nikitabobko.github.io/AeroSpace/commands#move-node-to-workspace
  - https://nikitabobko.github.io/AeroSpace/commands#layout
  - https://nikitabobko.github.io/AeroSpace/commands#split
  - https://nikitabobko.github.io/AeroSpace/commands#join-with
  - https://nikitabobko.github.io/AeroSpace/commands#flatten-workspace-tree
  - https://nikitabobko.github.io/AeroSpace/commands#exec-and-forget
  - https://nikitabobko.github.io/AeroSpace/commands#mode
  - https://nikitabobko.github.io/AeroSpace/commands#reload-config

### 8. Configuration Options
- **TOML Configuration**: Full config reference
  - https://nikitabobko.github.io/AeroSpace/guide#config-syntax
- **Config Options Reference**: All available configuration options
  - https://nikitabobko.github.io/AeroSpace/config-reference

### 9. Advanced Features
- **Exec Commands**: Running shell commands, exec-and-forget
  - https://nikitabobko.github.io/AeroSpace/commands#exec-and-forget
- **Focus Behavior**: Focus follows window, mouse handling
- **Normalization**: Container normalization options

### 10. Troubleshooting & Debugging
- **Troubleshooting Guide**: Common issues, debugging
  - https://nikitabobko.github.io/AeroSpace/guide#troubleshooting
- **Config Validation**: Checking config syntax
- **Logs & Debugging**: Finding logs, debug information

## Configuration File Structure

AeroSpace configs use TOML format:

```toml
# Start AeroSpace at login
start-at-login = true

# Gaps between windows
[gaps]
inner.horizontal = 10
inner.vertical = 10
outer.left = 10
outer.bottom = 10
outer.top = 10
outer.right = 10

# Main mode keybindings
[mode.main.binding]
alt-h = 'focus left'
alt-j = 'focus down'
alt-k = 'focus up'
alt-l = 'focus right'

# Workspace keybindings
alt-1 = 'workspace 1'
alt-2 = 'workspace 2'

# Move windows to workspaces
alt-shift-1 = 'move-node-to-workspace 1'
alt-shift-2 = 'move-node-to-workspace 2'

# Layout commands
alt-slash = 'layout tiles horizontal vertical'
alt-comma = 'layout accordion horizontal vertical'

# Window detection callbacks
[[on-window-detected]]
if.app-id = 'com.apple.finder'
run = 'layout floating'
```

## Research Process for Each Request

When user requests any configuration change:

1. **Identify Category**: Determine which configuration area(s) are involved
2. **Fetch Core Docs**: Get main documentation page for that category
3. **Fetch Specifics**: If needed, get specific command reference or examples
4. **Check Commands**: If keybinding involves commands, fetch command documentation
5. **Verify Syntax**: Check TOML syntax and available options
6. **Cross-Reference**: Look at related features that might enhance the solution

## Implementation Guidelines

### Code Quality
- Use clear section organization and comments
- Follow TOML best practices
- Validate syntax before suggesting
- Use consistent indentation

### Config Organization
- Group related settings together (gaps, keybindings, rules)
- Add section comments for clarity
- Preserve user's existing organizational structure
- Keep keybindings logically grouped by function

### Error Prevention
- Check for config file existence before editing
- Validate that TOML syntax is correct
- Ensure keybinding commands exist
- Verify app bundle IDs for window rules

### User Guidance
- Explain what each change does
- Link to relevant documentation pages
- Suggest how to test changes (`aerospace reload-config`)
- Offer related enhancements when appropriate
- Mention keyboard shortcuts in clear format (alt-shift-1)

## Common Tasks Quick Reference

### Testing Changes
```bash
aerospace reload-config
```

### Finding App Bundle IDs

**Method 1: Using osascript (preferred for known app names)**
```bash
osascript -e 'id of app "AppName"'
```
Example:
```bash
osascript -e 'id of app "Finder"'
# Returns: com.apple.finder
```

**Method 2: List all running apps**
```bash
aerospace list-apps
```

**Method 3: Using mdls on the app bundle**
```bash
mdls -name kMDItemCFBundleIdentifier /Applications/AppName.app
```

### Listing Windows

**List all windows:**
```bash
aerospace list-windows --all
```

**List windows in specific workspace:**
```bash
aerospace list-windows --workspace TER
```

**List windows as JSON (for detailed analysis):**
```bash
aerospace list-windows --all --json
```

**Filter windows by app (example for Ghostty):**
```bash
aerospace list-windows --all --json | python3 -c "import json,sys; data=json.load(sys.stdin); filtered=[w for w in data if 'AppName' in w.get('app-name','')]; print(json.dumps(filtered, indent=2))"
```

**List active workspaces:**
```bash
aerospace list-workspaces --monitor all --empty no
```

## Debugging & Log Inspection

When troubleshooting issues (window resizing, layout problems, unexpected behavior), use these debugging techniques:

### Check if AeroSpace is Running
```bash
pgrep -l AeroSpace && echo "AeroSpace is running" || echo "AeroSpace is NOT running"
```

### View System Logs

AeroSpace logs to the macOS unified logging system. Use the `log` command to retrieve logs:

**Get recent AeroSpace logs (last 30 minutes):**
```bash
/usr/bin/log show --predicate 'processImagePath CONTAINS "AeroSpace"' --last 30m --info --debug | tail -200
```

**Get logs for specific time period:**
```bash
/usr/bin/log show --predicate 'processImagePath CONTAINS "AeroSpace"' --last 10m --style compact
```

**Stream live logs (useful during testing):**
```bash
/usr/bin/log stream --predicate 'processImagePath CONTAINS "AeroSpace"' --style compact
```

Note: Use `/usr/bin/log` with full path to avoid shell interpretation issues.

### Debug Windows Session

To debug a specific problematic window:

1. Start debug session:
```bash
aerospace debug-windows
```

2. Focus the problematic window

3. Run the command again to get results:
```bash
aerospace debug-windows
```

### Common Debugging Scenarios

**Window not floating/tiling as expected:**
1. List windows to see current state: `aerospace list-windows --workspace WORKSPACE_NAME --json`
2. Check if on-window-detected rule is correct
3. Note: Rules only apply to NEW windows - existing windows keep their state

**Layout issues after config change:**
1. Reload config: `aerospace reload-config`
2. Close and reopen the problematic app for new rules to apply
3. Check logs for any errors

**App windows behaving unexpectedly:**
1. Verify app bundle ID: `osascript -e 'id of app "AppName"'`
2. List all windows from that app to see how AeroSpace perceives them
3. Some apps (like Ghostty) report tabs as separate windows - this is a known macOS limitation

### Debug Mode
```bash
aerospace enable-debug-mode
```

## Remember

- **Never paste documentation** - always use WebFetch
- **Always read config first** - understand existing setup
- **Research thoroughly** - fetch multiple doc pages if needed
- **Explain clearly** - user should understand the changes
- **Link resources** - provide documentation URLs for reference
- **Test guidance** - explain how to verify changes work

AeroSpace is actively developed. When in doubt, fetch the documentation.
