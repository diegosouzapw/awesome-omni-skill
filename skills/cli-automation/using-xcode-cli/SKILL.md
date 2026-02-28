---
name: using-xcode-cli
description: Builds and manages iOS/macOS apps using xcodebuild and xcrun simctl CLI tools. Use when working with Xcode projects, running apps in simulators, managing simulator instances, taking screenshots, capturing logs, running tests, or automating builds.
---

# Using Xcode CLI

## Overview

Native Xcode CLI tools (`xcodebuild` and `xcrun simctl`) provide full control over iOS/macOS builds and simulators without opening Xcode IDE. **Core principle:** Use CLI for automation, headless builds, and CI/CD—the same tools Xcode uses internally.

## When to Use

- Building iOS/macOS apps from command line
- Running apps in iOS Simulator programmatically
- Taking screenshots or recording video of simulator
- Running unit/UI tests with specific targeting
- Automating builds in CI/CD pipelines
- Managing simulator instances (boot, shutdown, erase)
- Simulating location, push notifications, or permissions
- Capturing app logs for debugging

**Symptoms that trigger this skill:**
- "Unable to find destination matching"
- "No scheme named X found"
- "xcodebuild: error:"
- Need to automate Xcode workflows
- Building without opening Xcode IDE

## When NOT to Use

- Editing code or project settings → Use Xcode IDE
- Managing Swift Package dependencies → Use `swift package` CLI
- Cross-platform builds → Use platform-specific tools
- Signing/provisioning profile management → Use Xcode or fastlane

## Quick Start

**Find available simulators:**
```bash
xcrun simctl list devices available
```

**Build for simulator:**
```bash
UDID=$(xcrun simctl list devices --json | jq -r '.devices | .[].[] | select(.name=="iPhone 16 Pro" and .isAvailable==true) | .udid' | head -1)
xcodebuild -workspace App.xcworkspace -scheme App -destination "platform=iOS Simulator,id=$UDID" -derivedDataPath /tmp/build build
```

**Install and launch:**
```bash
APP_PATH=$(find /tmp/build -name "*.app" -type d | head -1)
xcrun simctl install $UDID "$APP_PATH"
xcrun simctl launch --console $UDID com.bundle.identifier
```

**Take screenshot:**
```bash
xcrun simctl io $UDID screenshot /tmp/screenshot.png
```

## Quick Reference

| Task | Command |
|------|---------|
| List schemes | `xcodebuild -workspace App.xcworkspace -list` |
| List simulators | `xcrun simctl list devices available` |
| Get simulator UDID | `xcrun simctl list devices --json \| jq ...` |
| Boot simulator | `xcrun simctl boot $UDID` |
| Build for simulator | `xcodebuild ... -destination "platform=iOS Simulator,id=$UDID" build` |
| Install app | `xcrun simctl install $UDID /path/to/App.app` |
| Launch app | `xcrun simctl launch --console $UDID com.bundle.id` |
| Take screenshot | `xcrun simctl io $UDID screenshot /tmp/shot.png` |
| Run tests | `xcodebuild ... test` |
| Stream logs | `/usr/bin/log stream --predicate 'processImagePath CONTAINS "App"'` |

## Reference Documentation

| Reference | Contents |
|-----------|----------|
| [xcodebuild.md](reference/xcodebuild.md) | Project discovery, building, archiving, testing |
| [simctl.md](reference/simctl.md) | Device management, screenshots, video, location, permissions |
| [logging.md](reference/logging.md) | Log streaming and filtering predicates |
| [workflows.md](reference/workflows.md) | End-to-end automation scripts |

## Common Patterns

### Build + Run
1. Find simulator UDID via `simctl list devices --json`
2. Boot simulator with `simctl boot $UDID`
3. Build with `xcodebuild` using `-derivedDataPath`
4. Find .app bundle in derived data
5. Install with `simctl install`
6. Launch with `simctl launch --console`

### Run Tests
```bash
xcodebuild -workspace App.xcworkspace -scheme App \
  -destination "platform=iOS Simulator,id=$UDID" \
  -only-testing "AppTests/SpecificTest" test
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| "Unable to find destination" | Verify simulator exists with `simctl list devices`. Use exact name or UDID. |
| "No scheme found" | Run `xcodebuild -list` to see available schemes. Ensure you're using `-workspace` or `-project` flag. |
| "App not found after build" | Use `-derivedDataPath /tmp/build` and search there with `find`. |
| Simulator not responding | Try `xcrun simctl shutdown all` then boot fresh. |
| Build succeeds but app crashes | Check `xcrun simctl launch --console` for runtime errors. |
| Tests hang indefinitely | Add `-destination-timeout` flag. Ensure simulator is booted first. |
| Wrong simulator selected | Always use UDID from `simctl list devices --json`, not device name alone. |
| Stale build artifacts | Use `clean build` action or delete derived data directory. |
