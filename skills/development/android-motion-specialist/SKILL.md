---
name: android-motion-specialist
description: Expert Android developer for the Motion Detector project. Use this skill when working on Camera2 API integration, motion detection algorithms, Android networking (LAN sockets + Supabase Realtime), debugging crashes, or any Android/Kotlin development tasks specific to this sprint timing application.
---

# Android Motion Detection Specialist

## Overview

Specialized agent for developing and debugging the Motion Detector Android application - a sprint timing system using camera-based motion detection with tripwire functionality, multi-device synchronization via LAN sockets and Supabase Realtime.

## When to Use This Skill

Trigger this skill when:
- Working on Camera2 API integration or camera configuration
- Debugging motion detection algorithm or tripwire feature
- Implementing or debugging LAN socket networking
- Troubleshooting Supabase Realtime integration
- Analyzing Android crashes or performance issues
- Modifying lobby/participant management
- Optimizing frame processing or FPS control
- Any Kotlin/Android development on this project

## Core Capabilities

### 1. Camera2 Architecture Expert

**Specialization**: Camera2 API, image capture, preview management

**Key Tasks**:
- Configure camera characteristics and resolution selection
- Debug camera state transitions and lifecycle issues
- Optimize image format (YUV_420_888, NV21) for performance
- Implement FPS limiting and frame rate control
- Handle camera permissions and error recovery

**Reference**: Load `references/camera2-architecture.md` for component details

**Common Patterns**:
```kotlin
// Query supported resolutions
characteristicsHelper.getSupportedResolutions(cameraId, ImageFormat.YUV_420_888)

// Set FPS range
captureRequestBuilder.set(
    CaptureRequest.CONTROL_AE_TARGET_FPS_RANGE,
    Range(targetFps, targetFps)
)
```

### 2. Motion Detection Algorithm Specialist

**Specialization**: Frame difference algorithm, tripwire implementation, performance optimization

**Key Tasks**:
- Tune sensitivity and motion threshold parameters
- Implement and debug tripwire region detection
- Optimize frame processing (pixel sampling, resolution)
- Diagnose false positives/negatives
- Profile and reduce CPU usage

**Reference**: Load `references/motion-detection-algorithm.md` for algorithm details

**Key Parameters**:
- **sensitivity**: Pixel difference threshold (0-255) - Lower = more sensitive
- **motionThreshold**: Percentage of pixels (0.0-1.0) - Lower = more sensitive
- **tripwirePosition**: Horizontal position (0.0-1.0) - 0=left, 0.5=center, 1=right
- **tripwireWidth**: Width fraction (0.0-1.0) - Smaller = narrower detection zone

### 3. Networking Architecture Expert

**Specialization**: LAN sockets, Supabase Realtime, timing synchronization

**Key Tasks**:
- Debug LAN socket server/client connections
- Implement Supabase Realtime messaging
- Handle network state changes and reconnection
- Optimize latency for timing precision
- Troubleshoot WiFi discovery issues

**Reference**: Load `references/networking-architecture.md` for architecture

**Network Flow**:
1. **Discovery**: Supabase Realtime (4-digit session codes)
2. **Handshake**: Host broadcasts LAN IP:port via Realtime
3. **Connection**: Client connects TCP socket (port 9898)
4. **Timing**: LAN for low latency or Supabase fallback

### 4. Debugging & Performance Optimization

**Specialization**: Crash analysis, logcat interpretation, profiling

**Key Tasks**:
- Analyze stack traces and crash logs
- Use Android Studio Profiler (CPU, Memory, Network)
- Debug Camera2 state issues
- Profile frame processing performance
- Identify memory leaks and resource cleanup issues

**Reference**: Load `references/debugging-guide.md` for debugging strategies

**Common Tools**:
- Android Studio Debugger with breakpoints
- Logcat filtering: `tag:MotionDetector OR tag:Camera2Manager`
- CPU/Memory/Network profilers
- ADB commands for device inspection

## Project-Specific Knowledge

### Architecture Overview

**Main Components**:
- **MainActivity**: Camera screen with motion detection
- **LobbyActivity**: Device pairing and role assignment
- **Camera2Manager**: Camera2 API orchestrator
- **MotionDetector**: Frame difference algorithm with tripwire
- **LobbyRepository**: Supabase data access layer
- **LanSocketServer/Client**: TCP networking for timing

**Data Flow**:
```
Camera2 → ImageReader → MotionDetector → Timer
Lobby → Supabase → LAN Handshake → Timing Sync
```

**Reference**: Load `references/project-structure.md` for complete file layout

### Role-Based Timing System

**Roles**:
- **START**: Triggers when runner enters start gate
- **SPLIT**: Optional mid-course checkpoint
- **FINISH**: Final timing at finish line

**Assignment Flow**:
1. Host assigns roles via LobbyActivity dropdown
2. Roles sync via Supabase Realtime to `participants` table
3. Device with matching role can trigger timing event
4. Event broadcasts to all devices (LAN or Supabase mode)
5. Timing stored in `timing_events` table

### Configuration Files

**Supabase Setup** (`local.properties` - gitignored):
```properties
supabase.url=https://xxxxx.supabase.co
supabase.anon.key=your-anon-key
```

**Build Injection** (`app/build.gradle.kts`):
```kotlin
buildConfigField("String", "SUPABASE_URL", localProps["supabase.url"])
buildConfigField("String", "SUPABASE_ANON_KEY", localProps["supabase.anon.key"])
```

## Workflow Decision Tree

### Starting a New Task

**1. Is it camera-related?**
- YES → Load `references/camera2-architecture.md`
- Focus on Camera2Manager, CaptureSessionManager, CameraPreview
- Check camera lifecycle and state management

**2. Is it motion detection?**
- YES → Load `references/motion-detection-algorithm.md`
- Focus on MotionDetector.kt
- Consider parameter tuning and tripwire logic

**3. Is it networking?**
- YES → Load `references/networking-architecture.md`
- Focus on LanSocket classes, LobbyRepository
- Check both Supabase and LAN code paths

**4. Is it a crash or bug?**
- YES → Load `references/debugging-guide.md`
- Analyze stack trace first
- Check common issues section for similar patterns

**5. Need project context?**
- YES → Load `references/project-structure.md`
- Understand component relationships
- Review data flow diagrams

### Implementing a Feature

**Step 1**: Understand the requirement
- Which component does it affect?
- Does it require camera, motion, or network changes?

**Step 2**: Read relevant references
- Load applicable reference docs based on domain
- Review existing implementations in that file
- Check for similar patterns elsewhere

**Step 3**: Plan the implementation
- Identify specific files to modify
- Consider threading (main, camera, network)
- Plan error handling and null safety

**Step 4**: Implement with best practices
- Use Kotlin idioms (data classes, sealed classes, null safety)
- Handle lifecycle properly (onCreate, onPause, onDestroy)
- Add logging for debugging
- Follow existing code style

**Step 5**: Test thoroughly
- Unit tests for pure logic (MotionDetector)
- Manual testing on physical device
- Check edge cases (rotation, backgrounding, permissions)

### Debugging a Problem

**Step 1**: Gather information
- Read crash log or error message carefully
- Note timestamp and affected component
- Reproduce the issue consistently

**Step 2**: Locate the issue
- Find crash location in stack trace
- Read surrounding code in that file
- Load `references/debugging-guide.md` for similar issues

**Step 3**: Form hypothesis
- What caused this? Null reference? State issue? Threading?
- Is it lifecycle-related (camera closing early)?
- Check logcat for warnings before crash

**Step 4**: Test hypothesis
- Add strategic breakpoints or logging
- Run with Android Studio debugger attached
- Verify assumptions about state

**Step 5**: Implement fix
- Fix root cause, not just symptoms
- Add null checks or state guards
- Update related code if needed
- Test fix thoroughly and check for regressions

## Common Implementation Patterns

### Camera Initialization Pattern
```kotlin
// 1. Check permission
if (ContextCompat.checkSelfPermission(this, CAMERA) == GRANTED) {
    camera2Manager.openCamera(cameraId)
}

// 2. Camera opened callback
override fun onOpened(camera: CameraDevice) {
    camera2Manager.createCaptureSession(surfaces, callbacks)
}

// 3. Session configured callback
override fun onConfigured(session: CameraCaptureSession) {
    session.setRepeatingRequest(previewRequest, callback, handler)
}
```

### Motion Detection Pattern
```kotlin
// 1. Receive frame from ImageReader
override fun onImageAvailable(reader: ImageReader) {
    val image = reader.acquireLatestImage() ?: return

    // 2. Extract Y plane (grayscale)
    val plane = image.planes[0]
    val data = ByteArray(plane.buffer.remaining())
    plane.buffer.get(data)

    // 3. Detect motion with tripwire
    val motion = motionDetector.detectMotionFromByteArray(
        data, image.width, image.height
    )

    // 4. Handle result
    if (motion && !timerStarted) {
        timerManager.start()
        timerStarted = true
    }

    // 5. Cleanup
    image.close()
}
```

### Network Handshake Pattern
```kotlin
// HOST SIDE
// 1. Create lobby and get session code
val sessionCode = lobbyRepository.createLobby(deviceId, deviceName)

// 2. Start LAN server
lanServer = LanSocketServer(port = 9898)
lanServer.start()

// 3. Broadcast LAN offer via Realtime
channel.broadcast("lan_offer", mapOf(
    "ip" to localIp,
    "port" to 9898
))

// CLIENT SIDE
// 1. Join lobby with session code
lobbyRepository.joinLobby(sessionCode, deviceId, deviceName)

// 2. Subscribe to LAN offer
channel.on("lan_offer") { payload ->
    val hostIp = payload["ip"] as String
    val port = payload["port"] as Int

    // 3. Connect to host
    lanClient = LanSocketClient(hostIp, port)
    lanClient.connect()
}
```

## Best Practices

### Code Quality
- Use Kotlin data classes for data models
- Prefer sealed classes for state representation
- Use coroutines for async operations (avoid callbacks)
- Handle nullable types with safe calls (`?.`) or Elvis (`?:`)
- Add KDoc comments for public APIs

### Error Handling
- Wrap Camera2 calls in try-catch (CameraAccessException)
- Check for null before using Camera/Session references
- Handle network exceptions gracefully with retry logic
- Provide user feedback via Toast or Snackbar

### Resource Management
- Close ImageReader when done (onPause/onDestroy)
- Release camera in onPause, reopen in onResume
- Close sockets in cleanup methods
- Cancel coroutines properly (lifecycleScope)

### Threading
- Camera operations on HandlerThread (not main)
- Frame processing on background thread or coroutine
- UI updates always on main thread (use Handler/runOnUiThread)
- Network I/O on IO dispatcher (Dispatchers.IO)

### Logging
- Use consistent log tags (e.g., "Camera2Manager", "MotionDetector")
- Log state transitions and important events
- Log errors with full stack traces
- Use BuildConfig.DEBUG to disable verbose logs in release

## Quick Reference

### Common ADB Commands
```bash
# Install APK
adb install -r app/build/outputs/apk/debug/app-debug.apk

# View logs in realtime
adb logcat -v time | grep "MotionDetector\|Camera2"

# Clear logs
adb logcat -c

# Force stop app
adb shell am force-stop com.motiondetector

# Check WiFi connection
adb shell dumpsys wifi | grep "SSID"
```

### Common Gradle Commands
```bash
# Clean build
./gradlew clean

# Build debug APK
./gradlew assembleDebug

# Install on connected device
./gradlew installDebug

# Run unit tests
./gradlew test
```

### Logcat Filters
```
# Camera-related logs
tag:Camera2Manager OR tag:CaptureSessionManager OR tag:CameraPreview

# Motion detection
tag:MotionDetector

# Networking
tag:LanSocket OR tag:LobbyRepository OR tag:Supabase

# Crashes and errors only
level:error OR level:assert
```

## Integration with Task Tool

### When to Use Subagents

**Use `subagent_type=Explore`** when:
- Investigating unfamiliar codebase architecture
- Finding all usages of a class across the project
- Understanding data flow across multiple components

**Use `subagent_type=code-reviewer`** when:
- Reviewing code before committing
- Checking for common Android anti-patterns
- Validating null safety and resource cleanup

**Use `subagent_type=debugger`** when:
- Complex crash analysis spanning multiple files
- Performance bottleneck investigation
- Memory leak detection across components

### When to Use This Skill Directly

Use this skill directly (without subagents) when:
- You know exactly which file to modify
- Implementing a well-defined feature
- Making targeted bug fixes in a single component
- The task is self-contained within one domain

## Resources

Detailed reference documentation available in `references/`:

- **camera2-architecture.md**: Camera2 API components, workflows, threading
- **motion-detection-algorithm.md**: Frame difference algorithm, tripwire, tuning
- **networking-architecture.md**: LAN sockets, Supabase Realtime, timing sync
- **debugging-guide.md**: Debugging tools, common issues, crash analysis
- **project-structure.md**: File layout, data flow, dependencies, build config

Load references as needed based on the specific task domain. References contain code examples, common patterns, and troubleshooting guides.
