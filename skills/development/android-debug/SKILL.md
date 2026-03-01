---
name: android-debug
description: Debug Android Flutter apps including runtime errors, build issues, device connection problems, and performance issues. Use when troubleshooting Android-specific problems, crashes, or deployment issues.
---

# Android App Debugging Skill

Expert guidance for debugging Android Flutter applications, covering runtime errors, build issues, device connectivity, and performance problems.

## When to Use This Skill

- App crashes or runtime errors on Android
- Build failures in Android-specific code
- Device/emulator connection issues
- Performance problems (lag, memory, battery)
- Platform channel issues
- Native Android integration bugs
- APK/App Bundle generation problems

## Debugging Workflow

### 1. Identify the Problem Type

**Runtime Errors**: App crashes, exceptions, unexpected behavior
**Build Errors**: Gradle failures, dependency conflicts, compilation errors
**Device Issues**: Cannot connect, not detecting device
**Performance Issues**: Slow UI, memory leaks, battery drain

### 2. Gather Information

```bash
# Check Flutter doctor
flutter doctor -v

# List connected devices
flutter devices

# Run with verbose logging
flutter run --verbose

# Check Android logcat
flutter logs
# Or directly: adb logcat
```

### 3. Common Debug Commands

#### Device Connection
```bash
# List connected devices
adb devices

# Restart adb server
adb kill-server
adb start-server

# Connect to device over network
adb connect <device-ip>:5555

# Check device info
adb shell getprop ro.build.version.release
```

#### App Debugging
```bash
# Install debug APK
flutter install

# Run with debug logging
flutter run -d <device-id> --verbose

# Hot reload
r (in running flutter session)

# Hot restart
R (in running flutter session)

# View performance overlay
P (in running flutter session)
```

#### Log Analysis
```bash
# Filter logs by app
flutter logs | grep -i "flutter"

# Android-specific logs
adb logcat -s flutter

# Clear logs
adb logcat -c

# Save logs to file
adb logcat > debug.log
```

### 4. Common Issues and Solutions

#### Build Failures

**Issue**: Gradle build fails
```bash
# Clean and rebuild
flutter clean
cd android
./gradlew clean
cd ..
flutter pub get
flutter build apk
```

**Issue**: Dependency conflicts
```bash
# Check dependencies
cd android
./gradlew app:dependencies

# Update Gradle
cd android
./gradlew wrapper --gradle-version=8.0
```

**Issue**: Java version mismatch
```bash
# Check Java version (should be 17)
java -version

# Set JAVA_HOME if needed
export JAVA_HOME=/path/to/java17
```

#### Runtime Crashes

**Issue**: App crashes on startup
1. Check logs: `flutter logs`
2. Look for stack traces
3. Check `AndroidManifest.xml` permissions
4. Verify minimum SDK version in `android/app/build.gradle.kts`

**Issue**: Platform channel errors
1. Verify method names match between Dart and Kotlin/Java
2. Check parameter types are compatible
3. Add null safety checks
4. Review platform-specific code in `android/app/src/main/kotlin/`

#### Performance Issues

**Issue**: UI lag or jank
```bash
# Run in profile mode
flutter run --profile

# Enable performance overlay
flutter run --profile --trace-skia
```

**Issue**: Memory leaks
1. Use Flutter DevTools Memory tab
2. Check for retained references
3. Dispose controllers properly
4. Review image caching

**Issue**: Large APK size
```bash
# Analyze size
flutter build apk --analyze-size

# Enable R8 shrinking (already enabled in template)
# Check android/app/build.gradle.kts
```

### 5. Using Flutter DevTools

```bash
# Open DevTools
flutter pub global activate devtools
flutter pub global run devtools

# Or run automatically
flutter run --devtools
```

**DevTools Features**:
- **Inspector**: Widget tree visualization
- **Timeline**: Performance profiling
- **Memory**: Heap snapshots and leak detection
- **Network**: HTTP request monitoring
- **Logging**: Real-time log viewing
- **Debugger**: Breakpoints and step debugging

### 6. Android-Specific Debug Tools

#### Android Studio
1. Open `android/` folder in Android Studio
2. Use Android Profiler for deep analysis
3. Run layout inspector
4. Use APK Analyzer

#### ADB Commands
```bash
# Take screenshot
adb shell screencap /sdcard/screen.png
adb pull /sdcard/screen.png

# Record screen
adb shell screenrecord /sdcard/demo.mp4

# Get app info
adb shell dumpsys package com.cmwen.private_chat_hub

# Force stop app
adb shell am force-stop com.cmwen.private_chat_hub

# Clear app data
adb shell pm clear com.cmwen.private_chat_hub

# Monitor CPU usage
adb shell top | grep flutter
```

## Common Error Patterns

### Gradle Errors
- **Solution**: Clean build, check Java version, update dependencies
- **Files to check**: `android/build.gradle.kts`, `android/app/build.gradle.kts`

### Permission Errors
- **Solution**: Add permissions to `AndroidManifest.xml`
- **File**: `android/app/src/main/AndroidManifest.xml`

### Native Code Errors
- **Solution**: Check Kotlin/Java code in platform channels
- **Files**: `android/app/src/main/kotlin/`

### Resource Errors
- **Solution**: Check drawable/mipmap resources
- **Files**: `android/app/src/main/res/`

## Best Practices

1. **Always check Flutter doctor first**: `flutter doctor -v`
2. **Use verbose logging**: `flutter run --verbose` or `flutter build apk --verbose`
3. **Clean before rebuild**: `flutter clean` when in doubt
4. **Check device connection**: `flutter devices` and `adb devices`
5. **Read full stack traces**: Don't just look at the first error
6. **Test on multiple Android versions**: Emulators for API 21, 29, 33+
7. **Use profile mode for performance**: `flutter run --profile`
8. **Enable DevTools**: Best for deep debugging

## Quick Troubleshooting Checklist

- [ ] Run `flutter doctor -v` - all green?
- [ ] Run `flutter clean && flutter pub get`
- [ ] Check `adb devices` - device connected?
- [ ] Check Java version - is it 17?
- [ ] Review error logs - full stack trace?
- [ ] Try on different device/emulator
- [ ] Check `android/app/build.gradle.kts` config
- [ ] Verify `AndroidManifest.xml` settings
- [ ] Test with `flutter run --verbose`
- [ ] Use DevTools for deep analysis

## Additional Resources

- [Flutter Debugging Docs](https://docs.flutter.dev/testing/debugging)
- [Android Developer Debug Guide](https://developer.android.com/studio/debug)
- [Flutter DevTools](https://docs.flutter.dev/tools/devtools)
- [ADB Reference](https://developer.android.com/tools/adb)
