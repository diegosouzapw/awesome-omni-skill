---
name: livekit-nextjs-frontend
description: Build and review production-grade web and mobile frontends using LiveKit with Next.js. Covers real-time video/audio/data communication, WebRTC connections, track management, and best practices for LiveKit React components.
license: Complete terms in LICENSE.txt
---

# LiveKit Next.js Frontend Development

This skill guides the development and review of production-grade web and mobile frontends using LiveKit with Next.js. Use this when building real-time communication features including video conferencing, live streaming, audio rooms, or data synchronization.

## Overview

LiveKit is a WebRTC-based platform for building real-time video, audio, and data applications. The official React components library (`@livekit/components-react`) provides battle-tested hooks and components for Next.js applications.

**Latest Versions (as of 2025):**
- `@livekit/components-react`: v2.9.16+
- `livekit-client`: Latest
- `livekit-server-sdk`: v2+ (supports Node.js, Deno, and Bun)

### Key Dependencies

```json
{
  "dependencies": {
    "livekit-client": "latest",
    "@livekit/components-react": "latest",
    "livekit-server-sdk": "latest"
  },
  "devDependencies": {
    "tailwindcss": "latest",
    "autoprefixer": "latest",
    "postcss": "latest"
  }
}
```

**Optional (for custom UI with icons):**
```bash
npm install lucide-react
```

The examples use Tailwind CSS for styling and lucide-react for icons. These are optional - you can use your own styling solution and icons/text alternatives.

## Architecture Patterns

### 1. Token-Based Authentication

LiveKit uses JWT-based access tokens signed with your API secret. Tokens must be generated server-side to prevent secret exposure.

**Environment Setup (.env.local):**
```env
# Client-accessible (for LiveKitRoom component)
NEXT_PUBLIC_LIVEKIT_URL=wss://your-project.livekit.cloud

# Server-only (never exposed to client)
LIVEKIT_API_KEY=your-api-key
LIVEKIT_API_SECRET=your-api-secret
```

**Note:** For server-side features like recording, you may also need:
```env
LIVEKIT_URL=wss://your-project.livekit.cloud
```

**Token Generation API Route (app/api/token/route.ts):**
```typescript
import { AccessToken } from 'livekit-server-sdk';
import { NextRequest, NextResponse } from 'next/server';

export async function GET(request: NextRequest) {
  const roomName = request.nextUrl.searchParams.get('room');
  const participantName = request.nextUrl.searchParams.get('username');

  if (!roomName || !participantName) {
    return NextResponse.json(
      { error: 'Missing room or username' },
      { status: 400 }
    );
  }

  const at = new AccessToken(
    process.env.LIVEKIT_API_KEY!,
    process.env.LIVEKIT_API_SECRET!,
    {
      identity: participantName,
      ttl: '6h', // Token expires after 6 hours
    }
  );

  // Set permissions
  at.addGrant({
    roomJoin: true,
    room: roomName,
    canPublish: true,
    canSubscribe: true,
    canPublishData: true,
  });

  const token = await at.toJwt();

  return NextResponse.json({ token });
}
```

**Security Best Practices:**
- Never expose API secrets in client-side code
- Validate user identity before issuing tokens
- Set appropriate token TTL based on use case
- Implement rate limiting on token endpoint
- Use HTTPS in production

### 2. Room Connection Pattern

**Basic Room Component:**
```typescript
'use client';

import { LiveKitRoom, VideoConference } from '@livekit/components-react';
import '@livekit/components-styles';
import { useEffect, useState } from 'react';

interface RoomPageProps {
  roomName: string;
  username: string;
}

export default function RoomPage({ roomName, username }: RoomPageProps) {
  const [token, setToken] = useState('');

  useEffect(() => {
    // Fetch token from API route
    fetch(`/api/token?room=${roomName}&username=${username}`)
      .then(res => res.json())
      .then(data => setToken(data.token));
  }, [roomName, username]);

  if (!token) {
    return <div>Loading...</div>;
  }

  return (
    <LiveKitRoom
      token={token}
      serverUrl={process.env.NEXT_PUBLIC_LIVEKIT_URL!}
      connect={true}
      video={true}
      audio={true}
      onDisconnected={() => {
        // Handle disconnection
      }}
      onError={(error) => {
        console.error('Room error:', error);
      }}
    >
      <VideoConference />
    </LiveKitRoom>
  );
}
```

### 3. Custom Components with Hooks

**CRITICAL BEST PRACTICE:** Always use LiveKit's provided hooks instead of creating custom implementations. These hooks manage React state and are rigorously tested.

**Essential Hooks:**
- `useRoom()` - Access room state and events
- `useTracks()` - Subscribe to track updates
- `useParticipants()` - Get participant list
- `useLocalParticipant()` - Access local participant
- `useTrackToggle()` - Toggle audio/video
- `useLiveKitRoom()` - Lower-level room management

**Custom Controls Example:**
```typescript
'use client';

import { useRoom, useLocalParticipant, useTrackToggle } from '@livekit/components-react';
import { Track } from 'livekit-client';

export function CustomControls() {
  const room = useRoom();
  const { localParticipant } = useLocalParticipant();

  // Use built-in hook for track toggling
  const { buttonProps: audioProps, enabled: audioEnabled } = useTrackToggle({
    source: Track.Source.Microphone,
  });

  const { buttonProps: videoProps, enabled: videoEnabled } = useTrackToggle({
    source: Track.Source.Camera,
  });

  return (
    <div className="controls">
      <button {...audioProps}>
        {audioEnabled ? 'Mute' : 'Unmute'}
      </button>
      <button {...videoProps}>
        {videoEnabled ? 'Stop Video' : 'Start Video'}
      </button>
      <button onClick={() => room.disconnect()}>
        Leave Room
      </button>
    </div>
  );
}
```

### 4. Track Management

**Publishing Tracks:**
```typescript
import { useLocalParticipant } from '@livekit/components-react';
import { Track } from 'livekit-client';

function ScreenShareButton() {
  const { localParticipant } = useLocalParticipant();

  const startScreenShare = async () => {
    await localParticipant.setScreenShareEnabled(true);
  };

  const stopScreenShare = async () => {
    await localParticipant.setScreenShareEnabled(false);
  };

  return (
    <button onClick={startScreenShare}>Share Screen</button>
  );
}
```

**Subscribing to Remote Tracks:**
```typescript
import { useTracks, VideoTrack } from '@livekit/components-react';
import { Track } from 'livekit-client';

function RemoteParticipants() {
  // Subscribe to all camera tracks
  const tracks = useTracks([
    { source: Track.Source.Camera, withPlaceholder: true }
  ]);

  return (
    <div className="participants-grid">
      {tracks.map((track) => (
        <VideoTrack key={track.participant.sid} trackRef={track} />
      ))}
    </div>
  );
}
```

### 5. Data Messages

**IMPORTANT:** LiveKit recommends using higher-level APIs like text streams, byte streams, or RPC for most use cases. Use the low-level `publishData` API only when you need advanced control over individual packet behavior.

**Message Size Limits:**
- **Reliable packets**: 16KiB (16,384 bytes) recommended maximum for compatibility
- **Lossy packets**: 1,300 bytes maximum to stay within network MTU (1,400 bytes)
- Larger messages in lossy mode get fragmented; if any fragment is lost, the entire message is lost

**Sending Data:**
```typescript
import { useLocalParticipant } from '@livekit/components-react';

function ChatComponent() {
  const { localParticipant } = useLocalParticipant();

  const sendMessage = (message: string) => {
    const encoder = new TextEncoder();
    const data = encoder.encode(JSON.stringify({ message }));

    // Validate size (16KiB limit for reliable messages)
    if (data.byteLength > 16 * 1024) {
      console.error('Message too large');
      return;
    }

    // Use topic to differentiate message types
    localParticipant.publishData(data, {
      reliable: true,      // Reliable delivery with retransmission
      topic: 'chat',       // Topic helps filter different message types
    });
  };

  return (
    <button onClick={() => sendMessage('Hello!')}>
      Send Message
    </button>
  );
}
```

**Receiving Data:**
```typescript
import { useRoom } from '@livekit/components-react';
import { useEffect, useState } from 'react';
import { RemoteParticipant } from 'livekit-client';

function ChatDisplay() {
  const room = useRoom();
  const [messages, setMessages] = useState<string[]>([]);

  useEffect(() => {
    const handleData = (
      payload: Uint8Array,
      participant?: RemoteParticipant,
      kind?: any,
      topic?: string
    ) => {
      // Filter by topic
      if (topic !== 'chat') return;

      const decoder = new TextDecoder();
      const data = JSON.parse(decoder.decode(payload));
      setMessages(prev => [...prev, data.message]);
    };

    room.on('dataReceived', handleData);

    return () => {
      room.off('dataReceived', handleData);
    };
  }, [room]);

  return (
    <div>
      {messages.map((msg, i) => (
        <div key={i}>{msg}</div>
      ))}
    </div>
  );
}
```

**Delivery Modes:**
- **Reliable** (`reliable: true`): Packets delivered in order with retransmission. Best for chat, critical updates.
- **Lossy** (`reliable: false`): Each packet sent once, no ordering guarantee. Best for real-time updates where speed matters more than delivery.

## Code Review Checklist

When reviewing LiveKit Next.js code, verify:

### Architecture & Security
- [ ] API secrets stored in environment variables, not committed to repo
- [ ] Token generation happens server-side only
- [ ] Tokens have appropriate TTL and permissions
- [ ] User authentication/authorization before token issuance
- [ ] HTTPS/WSS used in production

### Connection & Room Management
- [ ] Room connection errors handled gracefully
- [ ] Disconnection events handled properly
- [ ] Reconnection logic implemented if needed
- [ ] Loading states shown during connection
- [ ] Proper cleanup on component unmount

### Track Management
- [ ] Using built-in hooks (`useTrackToggle`, `useTracks`) instead of custom implementations
- [ ] Track permissions requested appropriately
- [ ] Track publication/unpublication handled correctly
- [ ] Error handling for camera/microphone access
- [ ] Screen share functionality tested

### Data Communication
- [ ] Data encoding/decoding handled correctly
- [ ] Reliable vs lossy data packets chosen appropriately
- [ ] Message broadcasting vs targeted sending used correctly
- [ ] Data payload size validated (16KiB max for reliable, 1.3KB max for lossy)
- [ ] Topics used to differentiate message types
- [ ] Message size validation implemented before sending
- [ ] Consider using higher-level APIs (text streams, RPC) instead of low-level publishData

### Performance
- [ ] Video resolution and frame rate configured appropriately
- [ ] Simulcast enabled for better quality adaptation
- [ ] Track subscriptions limited to visible participants
- [ ] Component re-renders minimized
- [ ] Large participant lists handled efficiently

### User Experience
- [ ] Connection states communicated clearly to users
- [ ] Network quality indicators shown
- [ ] Graceful degradation on poor connections
- [ ] Mobile responsiveness tested
- [ ] Accessibility considerations (keyboard nav, screen readers, ARIA labels)

### Testing
- [ ] Multiple participants tested
- [ ] Network conditions simulated (slow, unstable)
- [ ] Device permissions handling tested
- [ ] Cross-browser compatibility verified
- [ ] Mobile devices tested (iOS Safari, Android Chrome)

## Common Patterns

### 1. Pre-join Screen
```typescript
function PreJoinScreen({ onJoin }: { onJoin: (username: string) => void }) {
  const [username, setUsername] = useState('');
  const [devices, setDevices] = useState({ audio: true, video: true });

  return (
    <div>
      <input
        value={username}
        onChange={(e) => setUsername(e.target.value)}
        placeholder="Enter your name"
      />
      <label>
        <input
          type="checkbox"
          checked={devices.audio}
          onChange={(e) => setDevices({ ...devices, audio: e.target.checked })}
        />
        Enable Microphone
      </label>
      <label>
        <input
          type="checkbox"
          checked={devices.video}
          onChange={(e) => setDevices({ ...devices, video: e.target.checked })}
        />
        Enable Camera
      </label>
      <button onClick={() => onJoin(username)}>
        Join Room
      </button>
    </div>
  );
}
```

### 2. Speaker Detection
```typescript
import { useTracks, VideoTrack } from '@livekit/components-react';
import { Track } from 'livekit-client';

function ActiveSpeakerView() {
  const tracks = useTracks([
    { source: Track.Source.Camera, withPlaceholder: true }
  ]);

  // Sort by speaking status and audio level
  const sortedTracks = tracks.sort((a, b) => {
    if (a.participant.isSpeaking && !b.participant.isSpeaking) return -1;
    if (!a.participant.isSpeaking && b.participant.isSpeaking) return 1;
    return (b.participant.audioLevel || 0) - (a.participant.audioLevel || 0);
  });

  return (
    <div>
      {/* Show active speaker large */}
      {sortedTracks[0] && (
        <VideoTrack
          trackRef={sortedTracks[0]}
          className="active-speaker"
        />
      )}

      {/* Show others small */}
      <div className="other-participants">
        {sortedTracks.slice(1).map(track => (
          <VideoTrack
            key={track.participant.sid}
            trackRef={track}
          />
        ))}
      </div>
    </div>
  );
}
```

### 3. Recording Integration
```typescript
// Server-side API route for starting recording
import { RoomServiceClient } from 'livekit-server-sdk';
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
  const { roomName } = await request.json();

  const client = new RoomServiceClient(
    process.env.LIVEKIT_URL!,
    process.env.LIVEKIT_API_KEY!,
    process.env.LIVEKIT_API_SECRET!
  );

  try {
    const egressId = await client.startRoomCompositeEgress(roomName, {
      file: {
        filepath: `recordings/${roomName}-${Date.now()}.mp4`,
      },
    });

    return NextResponse.json({ egressId });
  } catch (error) {
    return NextResponse.json({ error: 'Failed to start recording' }, { status: 500 });
  }
}
```

## Troubleshooting Guide

### Connection Issues
**Problem:** Room fails to connect
- Verify `NEXT_PUBLIC_LIVEKIT_URL` uses `wss://` protocol (for client-side connections)
- Check token is valid and not expired
- Ensure API key/secret match your LiveKit instance
- Verify network allows WebRTC connections (firewall/corporate proxy)
- Check browser console for specific error messages

### Track Issues
**Problem:** Camera/microphone not working
- Check browser permissions granted
- Verify HTTPS (required for getUserMedia)
- Test with different devices
- Check track publication succeeded: `localParticipant.videoTrackPublications`

### Performance Issues
**Problem:** Video quality poor or choppy
- Enable simulcast: `localParticipant.publishTrack(track, { simulcast: true })`
- Lower resolution/frame rate for mobile
- Implement dynacast for automatic quality adjustment
- Use adaptive bitrate and dynamic subscribe

### Data Message Issues
**Problem:** Data messages not received
- Verify `canPublishData` permission in token
- Check data payload size (max 16KiB for reliable, 1.3KB for lossy)
- Use reliable delivery for critical messages (chat, important updates)
- Use lossy delivery for real-time updates where speed matters (cursor position, state updates)
- Ensure proper encoding/decoding (TextEncoder/TextDecoder)
- Verify topic matches between sender and receiver
- Check browser console for errors

## Mobile Considerations

### iOS Safari
- Audio requires user gesture to start (button click)
- Screen sharing not supported
- Picture-in-picture available with proper configuration

### Android Chrome
- Hardware acceleration recommended
- Screen sharing requires HTTPS
- Background audio may require wake lock

### React Native
- Use `@livekit/react-native` package instead
- Requires native modules for camera/audio access
- Different permission handling per platform

## Performance Optimization

1. **Lazy Loading:** Load LiveKit components only when needed
2. **Simulcast:** Enable for adaptive video quality
3. **Selective Subscription:** Only subscribe to visible participants
4. **Dynacast:** Automatic quality optimization based on layout
5. **Connection Quality:** Monitor and display to users
6. **Message History:** For chat, implement pagination or virtual scrolling for large message lists
7. **Track Management:** Stop unused tracks immediately to conserve bandwidth

## Resources

- Official Docs: https://docs.livekit.io
- React Components: https://github.com/livekit/components-js
- Example Projects: https://github.com/livekit-examples
- Community: https://livekit.io/community

## Implementation Workflow

When building a LiveKit feature:

1. **Plan Architecture**
   - Define room structure and participant roles
   - Determine required tracks (audio, video, screen share)
   - Plan data messaging requirements

2. **Set Up Authentication**
   - Create token generation API route
   - Configure environment variables
   - Implement user identity validation

3. **Build Core Components**
   - Create room connection component
   - Add track publishing/subscribing
   - Implement controls (mute, video toggle, etc.)

4. **Add Advanced Features**
   - Pre-join screen with device selection
   - Data messaging for chat/metadata
   - Recording/streaming if needed

5. **Test Thoroughly**
   - Multiple participants
   - Various network conditions
   - Different devices and browsers
   - Edge cases (disconnection, permissions denied)

6. **Optimize & Polish**
   - Performance tuning
   - Error handling
   - Loading states
   - Accessibility

Remember: LiveKit handles the complex WebRTC infrastructure. Focus on building excellent user experiences with their battle-tested components and hooks.
