---
name: webrtc-timing-test
description: Measure WebRTC connection timing on Daily rooms. Use when testing Daily video call connection performance, measuring ICE negotiation time, benchmarking WebRTC setup latency, or when asked to test how long a Daily room takes to connect.
---

# WebRTC Connection Timing Test

Measure how long WebRTC connections take to establish on Daily video rooms.

## Prerequisites

- Chrome browser with Claude in Chrome extension
- Chrome window size: **705 x 942** pixels

## Procedure

### Step 1: Set up browser window

Resize Chrome window to 705 x 942 pixels using `mcp__claude-in-chrome__resize_window`.

### Step 2: Navigate to Daily room

Navigate to the Daily room URL (e.g., `kwindla.daily.co/gradient-ascent`). Wait for pre-join screen ("Get ready for your call") to load.

### Step 3: Inject interceptor

Execute `scripts/inject-interceptor.js` using `mcp__claude-in-chrome__javascript_tool`. This must be done BEFORE clicking Join.

### Step 4: Click Join

Execute `scripts/click-join.js` to click the Join button and record the exact time.

### Step 5: Retrieve results

Wait ~5 seconds for connection, then execute `scripts/retrieve-results.js` to get timing data.

## Key Metrics

| Metric | Description |
|--------|-------------|
| **totalConnectionTime** | Join click to `connectionState: connected` (user-perceived latency) |
| **callSetupTime** | Join click to RTCPeerConnection creation (Daily initialization) |
| **pureWebRTCTime** | RTCPeerConnection creation to connected (actual WebRTC negotiation) |
| **iceConnectionTime** | Join click to `iceConnectionState: connected` |

## Expected Results

| Metric | Typical Time |
|--------|--------------|
| callSetupTime | ~350-500ms |
| pureWebRTCTime | ~150-360ms |
| totalConnectionTime | ~500-900ms |

## Notes

- Daily creates 3 RTCPeerConnections (audio, video, data)
- The interceptor must be installed BEFORE clicking Join
- Network conditions affect ICE negotiation time
- Daily's iframe may cause timing variability between runs
