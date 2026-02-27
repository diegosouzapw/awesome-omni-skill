---
name: mobile-testing
description: "Executes automated tests on mobile apps via MCP. Use when testing iOS/Android apps, verifying UI states, automating interactions, or performing end-to-end validation. Not for web testing, API validation, or desktop applications."
metadata:
  mcp-server: mobile-mcp
  version: 1.0.0
  author: PharmaScan
  category: testing
---

# Mobile Testing Skill

## Overview

This skill provides structured workflows for testing mobile applications using the mobile-mcp server. All operations require device selection as the first step.

## Prerequisites

BEFORE any mobile interaction, you MUST:

1. **Select a device**: Call `mcp__mobile-mcp__mobile_list_available_devices`
2. **Store the device ID**: Use the returned identifier in all subsequent calls
3. **Verify connectivity**: Confirm device responds to `mcp__mobile-mcp__mobile_get_screen_size`

## Core Workflows

### Workflow: Test Mobile Application

--# Step 1: Select Device

Call `mcp__mobile-mcp__mobile_list_available_devices` to get available devices.

If multiple devices returned:
- Ask user to select one
- Store the selected device ID in a variable
- Proceed with that device ID

If only one device:
- Use it directly
- Optionally inform the user which device was selected

--# Step 2: Verify Device State

Call `mcp__mobile-mcp__mobile_get_screen_size` to confirm device is responsive.

Example response includes screen dimensions for coordinate calculations.

--# Step 3: Prepare Application State

For testing an app:

**If app needs to be installed first:**
```dart
mcp__mobile-mcp__mobile_install_app(
  device: deviceId,
  path: "/path/to/app.ipa"  // iOS
  // or
  path: "/path/to/app.apk"  // Android
)
```

**If app needs to be launched:**
```dart
mcp__mobile-mcp__mobile_launch_app(
  device: deviceId,
  packageName: "com.example.app"
)
```

**If app is already running:**
```dart
mcp__mobile-mcp__mobile_terminate_app(
  device: deviceId,
  packageName: "com.example.app"
)
// Then re-launch for clean state
```

--# Step 4: Perform Interactions

**To click an element:**
```dart
1. Call mcp__mobile-mcp__mobile_list_elements_on_screen(device: deviceId)
2. Identify target element coordinates from response
3. Call mcp__mobile-mcp__mobile_click_on_screen_at_coordinates(device: deviceId, x: x, y: y)
```

**To type text:**
```dart
mcp__mobile-mcp__mobile_type_keys(
  device: deviceId,
  text: "input text",
  submit: true  // or false if not submitting
)
```

**To swipe/navigate:**
```dart
mcp__mobile-mcp__mobile_swipe_on_screen(
  device: deviceId,
  direction: "up" | "down" | "left" | "right",
  distance: 400  // optional, uses defaults if omitted
)
```

**To verify state:**
```dart
mcp__mobile-mcp__mobile_take_screenshot(device: deviceId)
// or
mcp__mobile-mcp__mobile_list_elements_on_screen(device: deviceId)
```

--# Step 5: Verify Results

CRITICAL: ALWAYS verify outcomes after interactions.

1. **Take screenshot**: `mcp__mobile-mcp__mobile_take_screenshot` to capture current state
2. **Check elements**: `mcp__mobile-mcp__mobile_list_elements_on_screen` to verify expected elements exist
3. **Compare against expectations**: Report results to user

## Interaction Patterns

### Click Pattern

**ALWAYS** follow this sequence:

1. `mcp__mobile-mcp__mobile_list_elements_on_screen` → get coordinates
2. `mcp__mobile-mcp__mobile_click_on_screen_at_coordinates` → perform click
3. `mcp__mobile-mcp__mobile_take_screenshot` → verify change

**NEVER** click without first identifying the element.

### Swipe Pattern

**When swiping:**
- `direction` must be: `up`, `down`, `left`, or `right`
- `distance` defaults: 400px (iOS) or 30% screen (Android)
- Specify distance when precise control needed

**After swiping:**
- Wait for animations (500ms implicit)
- Take screenshot to verify navigation occurred

### Type Pattern

**Before typing:**
- Verify input field is focused (list elements to check)

**When typing:**
- Use `submit: true` to send Enter key
- Use `submit: false` for partial input

**Note:** Typing requires focused element. If typing fails, click the element first.

### Orientation Pattern

**To change orientation:**
```dart
mcp__mobile-mcp__mobile_set_orientation(
  device: deviceId,
  orientation: "portrait" | "landscape"
)
```

**When orientation matters:**
- Set BEFORE interactions that depend on layout
- Verify with `mcp__mobile-mcp__mobile_get_orientation` before critical operations

## Verification Checklists

### Before Any Interaction

- [ ] Device selected and device ID stored
- [ ] App is in expected state
- [ ] Coordinates verified (for clicks)

### After Any Interaction

- [ ] Screenshot captured
- [ ] Expected elements present
- [ ] Unexpected elements absent
- [ ] State change confirmed

### Before Test Completion

- [ ] All screenshots saved or reviewed
- [ ] Results logged
- [ ] App returned to clean state (optional)
- [ ] User notified of outcome

## Common Issues

--# Element Not Found

**Symptom**: `mcp__mobile-mcp__mobile_list_elements_on_screen` returns empty or unexpected elements.

**Diagnosis**:
1. Is the correct app launched?
2. Is the app on the expected screen?
3. Are there loading animations blocking the view?

**Recovery**:
```dart
// 1. Take screenshot to debug
mcp__mobile-mcp__mobile_take_screenshot(device: deviceId)

// 2. Try swiping to navigate
mcp__mobile-mcp__mobile_swipe_on_screen(device: deviceId, direction: "up")

// 3. Re-launch app for clean state
mcp__mobile-mcp__mobile_terminate_app(device: deviceId, packageName: package)
mcp__mobile-mcp__mobile_launch_app(device: deviceId, packageName: package)
```

--# Click Fails

**Symptom**: Click action completes but nothing happens.

**Diagnosis**:
1. Wrong coordinates (element moved)?
2. Overlapping elements?
3. Element disabled or not interactive?

**Recovery**:
```dart
// 1. Re-list elements (coordinates may have changed)
mcp__mobile-mcp__mobile_list_elements_on_screen(device: deviceId)

// 2. Try double-tap
mcp__mobile-mcp__mobile_double_tap_on_screen(device: deviceId, x: x, y: y)

// 3. Try long-press
mcp__mobile-mcp__mobile_long_press_on_screen_at_coordinates(device: deviceId, x: x, y: y)
```

--# Typing Fails

**Symptom**: Text not appearing or error on type.

**Diagnosis**:
1. No element focused?
2. Wrong input field?
3. Special characters not supported?

**Recovery**:
```dart
// 1. Click to focus first
mcp__mobile-mcp__mobile_click_on_screen_at_coordinates(device: deviceId, x: x, y: y)

// 2. Type without submit
mcp__mobile-mcp__mobile_type_keys(device: deviceId, text: "text", submit: false)

// 3. Submit separately if needed
mcp__mobile-mcp__mobile_press_button(device: deviceId, button: "ENTER")
```

--# App Crashes

**Symptom**: App becomes unresponsive after interaction.

**Recovery**:
```dart
// 1. Terminate and re-launch
mcp__mobile-mcp__mobile_terminate_app(device: deviceId, packageName: package)
mcp__mobile-mcp__mobile_launch_app(device: deviceId, packageName: package)

// 2. If persists, take screenshot for debugging
mcp__mobile-mcp__mobile_take_screenshot(device: deviceId)

// 3. Report to user with device state
```

## Device Management

### Listing Devices

```dart
mcp__mobile-mcp__mobile_list_available_devices()
// Returns: array of {id, name, type, ...}
```

### Selecting a Device

**For single device:** Use directly.

**For multiple devices:**
```dart
Ask user: "Which device should I use? [1] iPhone 15, [2] Android Emulator"
Store selection and use that deviceId
```

### Device Info

```dart
mcp__mobile-mcp__mobile_get_screen_size(device: deviceId)
// Returns: {width, height}
mcp__mobile-mcp__mobile_get_orientation(device: deviceId)
// Returns: {orientation: "portrait" | "landscape"}
```

## App Management Reference

| Operation | Tool | Parameters |
|:----------|:-----|:-----------|
| List apps | `mcp__mobile-mcp__mobile_list_apps` | device |
| Launch app | `mcp__mobile-mcp__mobile_launch_app` | device, packageName |
| Terminate app | `mcp__mobile-mcp__mobile_terminate_app` | device, packageName |
| Install app | `mcp__mobile-mcp__mobile_install_app` | device, path |
| Uninstall app | `mcp__mobile-mcp__mobile_uninstall_app` | device, bundle_id |

## Best Practices

**DO:**
- ALWAYS select device first
- VERIFY state before and after actions
- USE screenshots for verification
- HANDLE errors with recovery attempts
- REPORT results clearly

**DON'T:**
- NEVER click without listing elements first
- DON'T assume coordinates are stable
- DON'T skip verification steps
- DON'T leave app in crashed state
- DON'T proceed after failures without diagnosis

## Notes

- Coordinate system: top-left is (0, 0)
- All tools require `device` parameter
- Screen sizes vary by device model
- Wait times are implicit in tool calls
- Screenshot tool does not save automatically; use `mobile_save_screenshot` to persist
