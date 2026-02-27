---
name: stt-tts-service
description: Lightweight local speech-to-text and text-to-speech service for OpenClaw
version: 1.0.0
author: community
tags:
  - speech
  - audio
  - transcription
  - synthesis
  - voice
---

# STT-TTS Service

A lightweight, local speech-to-text (STT) and text-to-speech (TTS) service that runs on any device connected to your OpenClaw server. Perfect for voice-enabled workflows and flexible resource allocation.

## Features

- **Speech-to-Text**: Transcribe audio using faster-whisper (4x faster than OpenAI Whisper)
- **Text-to-Speech**: Generate natural speech using piper-tts or pyttsx3 fallback
- **100% Local**: No cloud APIs, works offline after initial model download
- **Flexible Deployment**: Run on any device - Raspberry Pi, laptop, or GPU server
- **HTTP API**: Simple REST endpoints for easy integration

## Quick Start

### Installation

```bash
# Clone or download this skill
cd stt-tts-service

# Install dependencies
pip install -r requirements.txt

# Start the service
python main.py
```

### Docker Deployment

```bash
docker build -t stt-tts-service .
docker run -p 8765:8765 stt-tts-service
```

## API Endpoints

### POST /stt - Speech to Text

Transcribe audio files to text.

```bash
curl -X POST http://localhost:8765/stt \
  -F "audio=@recording.wav"
```

**Response:**
```json
{
  "text": "Hello, this is the transcribed text.",
  "language": "en",
  "duration": 3.5
}
```

### POST /tts - Text to Speech

Convert text to audio.

```bash
curl -X POST http://localhost:8765/tts \
  -H "Content-Type: application/json" \
  -d '{"text": "Hello world", "voice": "default"}' \
  --output speech.wav
```

**Parameters:**
- `text` (required): Text to synthesize
- `voice` (optional): Voice ID to use
- `speed` (optional): Speech rate multiplier (0.5-2.0)

### GET /health

Health check endpoint.

```bash
curl http://localhost:8765/health
```

### GET /models

List available models and voices.

```bash
curl http://localhost:8765/models
```

## WebSocket Streaming (Real-time Voice)

For real-time voice conversations, use WebSocket endpoints:

### WS /ws/stt - Streaming Speech-to-Text

Stream audio and receive transcriptions in real-time.

```javascript
const ws = new WebSocket('ws://localhost:8765/ws/stt');

// Send audio chunks (16kHz, 16-bit, mono PCM)
ws.send(audioBuffer);

// Receive transcriptions
ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log(data.text);  // Transcribed text
};

// Flush remaining audio
ws.send(JSON.stringify({action: "flush"}));
```

### WS /ws/tts - Streaming Text-to-Speech

Send text and receive audio chunks in real-time.

```javascript
const ws = new WebSocket('ws://localhost:8765/ws/tts');

// Send text to synthesize
ws.send(JSON.stringify({text: "Hello world"}));

// Receive audio chunks
ws.onmessage = (event) => {
  if (event.data instanceof Blob) {
    // Audio chunk - play it
    playAudio(event.data);
  }
};
```

### WS /ws/voice - Full Duplex Voice Conversation

Stream audio input and receive audio output for real-time voice-to-voice.

```javascript
const ws = new WebSocket('ws://localhost:8765/ws/voice');

// Stream microphone audio
navigator.mediaDevices.getUserMedia({audio: true})
  .then(stream => {
    // Send audio chunks to WebSocket
  });

// Handle responses
ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  if (data.type === "transcript") {
    // User's speech transcribed - send to your AI
    sendToAI(data.text);
  }
};

// Send AI response to be spoken
ws.send(JSON.stringify({action: "speak", text: aiResponse}));
```

## Configuration

Set environment variables or edit `config.py`:

| Variable | Default | Description |
|----------|---------|-------------|
| `STT_MODEL` | `base` | Whisper model: tiny, base, small, medium |
| `TTS_ENGINE` | `auto` | TTS engine: piper, pyttsx3, auto |
| `DEVICE` | `auto` | Compute device: cpu, cuda, auto |
| `HOST` | `0.0.0.0` | Server bind address |
| `PORT` | `8765` | Server port |

## Model Sizes

| STT Model | Size | Speed | Accuracy |
|-----------|------|-------|----------|
| tiny | ~75MB | Fastest | Basic |
| base | ~150MB | Fast | Good |
| small | ~500MB | Medium | Better |
| medium | ~1.5GB | Slower | Best |

## OpenClaw Integration

Register this service with your OpenClaw server:

```bash
openclaw service register http://device-ip:8765
```

Then use in your workflows:
```yaml
- action: stt
  input: ${audio_file}
  output: transcription
  
- action: tts
  input: "Hello, ${user_name}!"
  output: greeting_audio
```

## Requirements

- Python 3.9+
- 2GB RAM minimum (4GB recommended for medium model)
- ~500MB disk space (plus model storage)
