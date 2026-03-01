---
name: android-development
description: Android development patterns for Kotlin/Java including MediaProjection, Accessibility Service, Socket.IO, and foreground services. Use when working on TitanMirror or other Android projects.
---

# 📱 Android Development Skill

## Use Cases
- Screen mirroring (MediaProjection)
- Remote control (Accessibility Service)
- Real-time communication (Socket.IO)
- Background services

---

## MediaProjection Setup

### 1. Request Permission
```kotlin
val mediaProjectionManager = getSystemService(MEDIA_PROJECTION_SERVICE) as MediaProjectionManager
startActivityForResult(mediaProjectionManager.createScreenCaptureIntent(), REQUEST_CODE)
```

### 2. Create Projection
```kotlin
override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
    if (requestCode == REQUEST_CODE && resultCode == RESULT_OK) {
        mediaProjection = mediaProjectionManager.getMediaProjection(resultCode, data!!)
        startCapture()
    }
}
```

### 3. ImageReader for Frames
```kotlin
val imageReader = ImageReader.newInstance(width, height, PixelFormat.RGBA_8888, 5)
imageReader.setOnImageAvailableListener({ reader ->
    val image = reader.acquireLatestImage()
    // Process frame...
    image?.close()
}, handler)
```

---

## Foreground Service (Android 14+)

> ⚠️ **CRITICAL**: Must call `startForeground()` immediately in `onStartCommand()`

```kotlin
override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
    // FIRST: Start foreground immediately!
    val notification = createNotification()
    startForeground(NOTIFICATION_ID, notification, ServiceInfo.FOREGROUND_SERVICE_TYPE_MEDIA_PROJECTION)
    
    // THEN: Extract intent and start work
    val resultCode = intent?.getIntExtra("resultCode", -1) ?: -1
    val data = intent?.getParcelableExtra("data", Intent::class.java)
    
    // Add delay before capture
    Handler(Looper.getMainLooper()).postDelayed({
        startCapture(resultCode, data)
    }, 1500)
    
    return START_NOT_STICKY
}
```

---

## Socket.IO Connection

```kotlin
val socket = IO.socket("http://192.168.1.x:3000")

socket.on(Socket.EVENT_CONNECT) {
    Log.d("Socket", "Connected!")
    socket.emit("register", deviceInfo)
}

socket.on("command") { args ->
    val command = args[0] as JSONObject
    handleCommand(command)
}

socket.connect()
```

---

## Accessibility Service

```kotlin
class RemoteControlService : AccessibilityService() {
    
    override fun onAccessibilityEvent(event: AccessibilityEvent?) {
        // Handle accessibility events
    }
    
    fun performClick(x: Int, y: Int) {
        val path = Path().apply { moveTo(x.toFloat(), y.toFloat()) }
        val gesture = GestureDescription.Builder()
            .addStroke(GestureDescription.StrokeDescription(path, 0, 100))
            .build()
        dispatchGesture(gesture, null, null)
    }
    
    fun performSwipe(startX: Int, startY: Int, endX: Int, endY: Int) {
        val path = Path().apply {
            moveTo(startX.toFloat(), startY.toFloat())
            lineTo(endX.toFloat(), endY.toFloat())
        }
        val gesture = GestureDescription.Builder()
            .addStroke(GestureDescription.StrokeDescription(path, 0, 300))
            .build()
        dispatchGesture(gesture, null, null)
    }
}
```

---

## Decision Tree

```
Android task?
├── Screen capture? → MediaProjection + ImageReader
├── Background work? → Foreground Service (Android 14 rules!)
├── Remote control? → Accessibility Service
├── Network comm? → Socket.IO
└── Permissions? → Runtime request + Manifest
```

---

## Common Pitfalls

| ปัญหา | สาเหตุ | แก้ไข |
|-------|--------|-------|
| Black screen | startForeground timing | Call immediately in onStartCommand |
| Crash on Android 14 | Wrong intent extraction | Use `getParcelableExtra(key, Class)` |
| Low FPS | Buffer too small | Increase ImageReader buffer to 5 |
| Service killed | Not foreground | Add FOREGROUND_SERVICE permission |
