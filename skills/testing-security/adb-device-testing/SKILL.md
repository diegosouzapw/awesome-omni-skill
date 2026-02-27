---
name: adb-device-testing
description: Use when testing Android apps on ADB-connected devices/emulators - UI automation, screenshots, location spoofing, navigation, app management. Triggers on ADB, emulator, Android testing, location mock, UI test, screenshot walkthrough.
---

# ADB Device Testing

## Quick Reference

### Screenshots (Multimodal)
```bash
adb exec-out screencap -p > /tmp/screen.png
```
Then use Read tool to VIEW the image.

### Location Spoofing (Emulator Only)
```bash
# longitude, latitude
adb emu geo fix -74.006 40.7128  # NYC
adb emu geo fix 2.3522 48.8566   # Paris
```

IMPORTANT: for getting proper component location always use Element Discovery (uiautomator)

### Navigation
| Action | Command |
|--------|---------|
| Tap | `adb shell input tap <x> <y>` |
| Scroll up | `adb shell input swipe 500 1500 500 500 300` |
| Scroll down | `adb shell input swipe 500 500 500 1500 300` |
| Back | `adb shell input keyevent KEYCODE_BACK` |
| Home | `adb shell input keyevent KEYCODE_HOME` |
| Text | `adb shell input text "hello"` |
| Long press | `adb shell input swipe 500 500 500 500 1000` |

### Element Discovery (uiautomator)
```bash
adb shell uiautomator dump /sdcard/ui.xml
adb pull /sdcard/ui.xml /tmp/ui.xml
# Parse bounds="[x1,y1][x2,y2]" → tap center
```

### App Management
```bash
adb install -r app.apk
adb shell am start -n <package>/<activity>
adb shell am force-stop <package>
adb shell pm clear <package>
```

## Testing Workflow

Copy this checklist:

```
Progress:
- [ ] Verify device connected (adb devices)
- [ ] Setup: install app, clear state, set location
- [ ] Screenshot initial state
- [ ] Execute test actions (tap/swipe)
- [ ] Wait after each action (sleep 0.5-1)
- [ ] Screenshot and verify each step
- [ ] Report results with evidence
```

### Step-by-Step

**1. Verify Connection**
```bash
adb devices -l
```
No devices? Check USB debugging enabled, emulator running.

**2. Setup**
```bash
adb install -r /path/to/app.apk
adb shell pm clear <package>
adb emu geo fix <lon> <lat>  # emulator only
adb shell am start -n <package>/<activity>
sleep 2
```

**3. Test Loop**
```bash
# Screenshot
adb exec-out screencap -p > /tmp/screen_01.png
# View screenshot with Read tool to analyze UI
# Identify tap coordinates from UI or uiautomator dump
adb shell input tap <x> <y>
sleep 1
# Screenshot to verify
adb exec-out screencap -p > /tmp/screen_02.png
```

**4. Element Finding**
```bash
adb shell uiautomator dump /sdcard/ui.xml
adb pull /sdcard/ui.xml /tmp/ui.xml
# Grep for element: bounds="[100,200][300,400]"
# Tap center: (100+300)/2=200, (200+400)/2=300
adb shell input tap 200 300
```

## Critical Rules

1. **Always wait** after UI actions before screenshots (`sleep 0.5-1`)
2. **View screenshots** with Read tool - don't just capture
3. **Use absolute paths** for screenshot files
4. **Location spoofing = emulator only** - physical devices need mock apps
5. **Parse uiautomator XML** for precise element coordinates

## Advanced Features

**Device info:**
```bash
adb shell wm size              # resolution
adb shell getprop ro.product.model
```

**Logs:**
```bash
adb logcat -d | grep <package>
```

**Screen recording:**
```bash
adb shell screenrecord /sdcard/demo.mp4
# Ctrl+C to stop
adb pull /sdcard/demo.mp4 /tmp/
```

**For complex testing scenarios**: See [ADVANCED.md](ADVANCED.md)
