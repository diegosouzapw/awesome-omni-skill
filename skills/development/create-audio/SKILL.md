---
name: create-audio
description: Generate audio from text using 13 TTS providers (local + cloud). Use when user wants to create audio files, convert text to speech, generate voiceovers, create audio with different voices, use voice cloning, multilingual TTS, or mentions /create-audio command. Supports Pocket TTS (CPU, 8 voices), MLX-Audio (Apple Silicon, 7 models, 50+ voices), ElevenLabs (cloud API, 32 languages, 10k+ voices), and Coqui TTS (open source, 4 models, voice cloning). Includes 32+ languages, voice cloning, speed control, and both local and cloud options.
---

# Create Audio

Generate high-quality audio from text using local TTS models that run on your CPU.

## Quick Start

**Basic usage:**
```bash
# Generate with default voice
scripts/create_audio.py --text "Hello world" --output hello.wav

# Use specific voice
scripts/create_audio.py --voice marius --text "Bonjour" --output greeting.wav

# From stdin
echo "Hello from stdin" | scripts/create_audio.py --output output.wav
```

## Available Providers

### Pocket TTS (Default)
- **Provider ID:** `pocket-tts`
- **Model:** Kyutai Labs Pocket TTS (100M parameters)
- **Performance:** ~6x real-time on MacBook Air M4, ~200ms first chunk
- **Requirements:** CPU only (no GPU needed)
- **Language:** English
- **Installation:** `pip install pocket-tts` or `uvx pocket-tts`

**Built-in Voices:**
- `alba` (default) - Female voice
- `marius` - Male voice
- `javert`, `jean`, `fantine`, `cosette`, `eponine`, `azelma` - Character voices

### MLX-Audio Providers (Apple Silicon Optimized)

MLX-Audio provides **7 different TTS models** optimized for Apple Silicon (M1/M2/M3/M4). Each model is registered as a separate provider.

**Installation:** `pip install mlx-audio`
**Requirements:** Apple Silicon Mac (M1/M2/M3/M4), Python 3.9+

#### 1. Kokoro (`mlx-audio-kokoro`)
- **Languages:** English, Japanese, Chinese, French, Spanish, Italian, Portuguese, Hindi
- **Voices:** 10 built-in voices + voice cloning
  - American: `af_heart`, `af_bella`, `af_nova`, `af_sky`, `am_adam`, `am_echo`
  - British: `bf_alice`, `bf_emma`, `bm_daniel`, `bm_george`
- **Best for:** Multilingual content, fast generation
- **Features:** Speed control, voice cloning

#### 2. CSM (`mlx-audio-csm`)
- **Languages:** English
- **Features:** Conversational Speech Model with voice cloning
- **Best for:** Natural conversations, voice cloning

#### 3. Dia (`mlx-audio-dia`)
- **Languages:** English
- **Best for:** Dialogue-focused content

#### 4. OuteTTS (`mlx-audio-oute`)
- **Languages:** English
- **Best for:** Efficient, fast generation

#### 5. Spark (`mlx-audio-spark`)
- **Languages:** English, Chinese
- **Best for:** Bilingual content

#### 6. Chatterbox (`mlx-audio-chatterbox`)
- **Languages:** 16 languages (en, es, fr, de, it, pt, pl, tr, ru, nl, cs, ar, zh, ja, hu, ko)
- **Best for:** Expressive multilingual content

#### 7. Soprano (`mlx-audio-soprano`)
- **Languages:** English
- **Best for:** High-quality English TTS

### ElevenLabs (Cloud API)

Premium cloud-based TTS with the highest quality voices and massive voice library.

**Provider ID:** `elevenlabs`

- **Languages:** 32 languages (en, es, fr, de, it, pt, pl, uk, nl, sv, da, fi, no, cs, sk, el, ro, bg, hr, sr, mk, lv, lt, et, sl, hu, tr, vi, ar, hi, bn, ta, ko, zh, ja, etc.)
- **Voices:** 10,000+ voices in Voice Library
- **Features:** Professional voice cloning, voice design, emotional control
- **Requirements:** API key (ELEVEN_API_KEY environment variable)
- **Pricing:** Free tier available, paid plans for production
- **Best for:** Production-quality voiceovers, professional content
- **Installation:** `pip install elevenlabs`

**Popular Voices:**
- `rachel`, `drew`, `clyde`, `paul`, `domi` - English
- `antoni`, `thomas`, `charlie`, `george` - Various accents
- `emily`, `elli`, `charlotte`, `alice`, `matilda` - Female voices
- Or use any custom voice_id from the Voice Library

### Coqui TTS Providers (Open Source)

Coqui TTS provides 4 different models for various use cases - all free and open source.

**Installation:** `pip install TTS`
**Requirements:** Python 3.10-3.14, CPU or GPU

#### 1. XTTS v2 (`coqui-xtts_v2`)
- **Languages:** 17 languages (en, es, fr, de, it, pt, pl, tr, ru, nl, cs, ar, zh-cn, ja, hu, ko, hi)
- **Features:** Best quality, voice cloning with 3-10 seconds of audio
- **Best for:** Multilingual projects with voice cloning needs
- **Streaming:** <200ms latency

#### 2. VITS (`coqui-vits`)
- **Languages:** English
- **Features:** Fast, single speaker
- **Best for:** Quick English TTS without voice customization

#### 3. YourTTS (`coqui-yourtts`)
- **Languages:** English, French, Portuguese
- **Features:** Multilingual voice cloning, multi-speaker
- **Best for:** Voice cloning in multiple languages

#### 4. Bark (`coqui-bark`)
- **Languages:** 13 languages (en, de, es, fr, hi, it, ja, ko, pl, pt, ru, tr, zh)
- **Features:** Highly expressive, multi-speaker
- **Best for:** Expressive, natural-sounding speech

## Core Capabilities

### 1. Basic Text-to-Speech

Generate audio from any text input:

```bash
scripts/create_audio.py \
  --text "Your text here" \
  --output audio.wav
```

### 2. Voice Selection

Choose from built-in voices:

```bash
scripts/create_audio.py \
  --voice marius \
  --text "Hello in a different voice" \
  --output voice_demo.wav
```

### 3. Voice Cloning

Clone any voice by providing a WAV file:

```bash
scripts/create_audio.py \
  --voice /path/to/reference_voice.wav \
  --text "This will sound like the reference" \
  --output cloned.wav
```

### 4. Provider Selection

Switch between TTS providers (extensible architecture):

```bash
# List all available providers
scripts/create_audio.py --list

# Use specific provider
scripts/create_audio.py \
  --provider pocket-tts \
  --text "Hello" \
  --output audio.wav
```

### 5. Pipeline Integration

Integrate with other tools via stdin/stdout:

```bash
# From file
cat script.txt | scripts/create_audio.py --output narration.wav

# From command output
echo "This is generated text" | scripts/create_audio.py -o result.wav

# Chain with other tools
cat content.md | sed 's/#//g' | scripts/create_audio.py -o doc_audio.wav
```

## Common Use Cases

### Voiceover Generation
```bash
# Pocket TTS voiceover
scripts/create_audio.py \
  --voice alba \
  --text "Welcome to this tutorial about..." \
  --output intro_voiceover.wav

# MLX-Audio Kokoro (multilingual)
scripts/create_audio.py \
  --provider mlx-audio-kokoro \
  --voice af_heart \
  --text "Welcome to this tutorial" \
  --output intro_mlx.wav
```

### Multi-Voice Content
```bash
# Character 1 (Pocket TTS)
scripts/create_audio.py --voice jean --text "First character speaks" -o char1.wav

# Character 2 (MLX-Audio)
scripts/create_audio.py --provider mlx-audio-kokoro --voice bm_george --text "Second character responds" -o char2.wav
```

### Multilingual Content
```bash
# English
scripts/create_audio.py \
  --provider mlx-audio-kokoro \
  --voice af_heart \
  --params '{"lang": "en"}' \
  --text "Hello world" \
  --output en.wav

# Japanese
scripts/create_audio.py \
  --provider mlx-audio-kokoro \
  --params '{"lang": "ja"}' \
  --text "こんにちは世界" \
  --output ja.wav

# French
scripts/create_audio.py \
  --provider mlx-audio-kokoro \
  --params '{"lang": "fr"}' \
  --text "Bonjour le monde" \
  --output fr.wav
```

### Custom Voice Branding
```bash
# Pocket TTS voice cloning
scripts/create_audio.py \
  --voice /path/to/your_voice_sample.wav \
  --text "All content in my voice" \
  --output branded_audio.wav

# MLX-Audio CSM voice cloning
scripts/create_audio.py \
  --provider mlx-audio-csm \
  --voice custom \
  --params '{"custom_voice_path": "/path/to/voice.wav"}' \
  --text "Cloned voice with CSM" \
  --output csm_cloned.wav
```

### Speed Control
```bash
# Faster speech (1.5x)
scripts/create_audio.py \
  --provider mlx-audio-kokoro \
  --voice af_nova \
  --params '{"speed": 1.5}' \
  --text "This will be faster" \
  --output fast.wav

# Slower speech (0.8x)
scripts/create_audio.py \
  --provider mlx-audio-kokoro \
  --voice bf_emma \
  --params '{"speed": 0.8}' \
  --text "This will be slower" \
  --output slow.wav
```

### ElevenLabs Production Quality
```bash
# Set API key first
export ELEVEN_API_KEY="your_api_key_here"

# Use default voice
scripts/create_audio.py \
  --provider elevenlabs \
  --voice rachel \
  --text "Professional quality voiceover" \
  --output professional.wav

# Use custom voice from Voice Library
scripts/create_audio.py \
  --provider elevenlabs \
  --voice custom \
  --params '{"voice_id": "your_voice_id_here"}' \
  --text "Custom voice from library" \
  --output custom.wav

# Advanced settings (stability, similarity, style)
scripts/create_audio.py \
  --provider elevenlabs \
  --voice rachel \
  --params '{"stability": 60, "similarity_boost": 80, "style": 20, "speed": 1.1}' \
  --text "Fine-tuned voice settings" \
  --output tuned.wav

# Multilingual with language enforcement
scripts/create_audio.py \
  --provider elevenlabs \
  --voice antoni \
  --params '{"language_code": "es"}' \
  --text "Hola mundo" \
  --output spanish.wav
```

### Coqui TTS Voice Cloning
```bash
# XTTS v2 voice cloning (best quality)
scripts/create_audio.py \
  --provider coqui-xtts_v2 \
  --voice custom \
  --params '{"speaker_wav": "/path/to/reference.wav", "language": "en"}' \
  --text "Cloned voice with XTTS v2" \
  --output xtts_cloned.wav

# YourTTS multilingual cloning
scripts/create_audio.py \
  --provider coqui-yourtts \
  --voice custom \
  --params '{"speaker_wav": "/path/to/voice.wav", "language": "fr"}' \
  --text "Bonjour le monde" \
  --output yourtts_fr.wav

# VITS fast English
scripts/create_audio.py \
  --provider coqui-vits \
  --text "Fast English synthesis" \
  --output vits.wav

# Bark expressive speech
scripts/create_audio.py \
  --provider coqui-bark \
  --voice custom \
  --params '{"speaker_idx": 0}' \
  --text "Very expressive and natural sounding" \
  --output bark.wav
```

## Adding New TTS Providers

The skill uses an extensible provider architecture. To add a new provider:

1. **Create provider class** in `scripts/<provider>_provider.py`:
```python
from tts_provider import TTSProvider

class NewProvider(TTSProvider):
    @property
    def name(self) -> str:
        return "new-provider"

    @property
    def supported_voices(self) -> List[str]:
        return ["voice1", "voice2"]

    # Implement other required methods...
```

2. **Register provider** in `scripts/provider_registry.py`:
```python
from new_provider import NewProvider
ProviderRegistry.register(NewProvider)
```

3. **Use immediately:**
```bash
scripts/create_audio.py --provider new-provider --voice voice1 --text "Test"
```

See `scripts/pocket_tts_provider.py` for a complete implementation example.

## Advanced Usage

### Provider-Specific Parameters

Pass provider-specific parameters as JSON:

```bash
scripts/create_audio.py \
  --voice custom \
  --params '{"custom_voice_path": "/path/to/voice.wav"}' \
  --text "Using custom voice" \
  --output result.wav
```

### Batch Processing

Generate multiple audio files:

```bash
# Simple loop
for voice in alba marius jean; do
  scripts/create_audio.py \
    --voice $voice \
    --text "Sample text" \
    --output "${voice}_sample.wav"
done
```

## Installation Requirements

### Pocket TTS
```bash
# Via pip
pip install pocket-tts

# Or use uvx (no installation needed)
uvx pocket-tts generate --text "Test"
```

**Dependencies:**
- Python 3.10+ (supports up to 3.14)
- PyTorch 2.5+ (CPU version)
- scipy (for WAV file writing)

## Troubleshooting

**Model not loading?**
- First run takes longer (downloads model)
- Model is cached for subsequent uses
- Check internet connection for initial download

**Import errors?**
```bash
pip install pocket-tts scipy
```

**Voice file not found?**
- Ensure WAV files are valid audio files
- Use absolute paths for custom voices
- Check file permissions

## Technical Details

### Architecture

```
create_audio.py (CLI)
    ↓
provider_registry.py (Provider management)
    ↓
tts_provider.py (Base interface)
    ↓
[pocket_tts_provider.py, future_provider.py, ...]
```

### Provider Interface

All providers implement:
- `name`: Provider identifier
- `supported_voices`: List of available voices
- `default_voice`: Fallback voice
- `generate_audio()`: Core synthesis method
- `get_provider_params()`: Provider-specific options

### Output Format

- **Format:** WAV (uncompressed)
- **Sample Rate:** Provider-specific (Pocket TTS: 16kHz)
- **Channels:** Mono
- **Bit Depth:** 16-bit PCM

## Resources

### Scripts
- `create_audio.py` - Main CLI tool
- `tts_provider.py` - Base provider interface
- `pocket_tts_provider.py` - Pocket TTS implementation
- `provider_registry.py` - Provider management system

### References

**Pocket TTS:**
- [Pocket TTS GitHub](https://github.com/kyutai-labs/pocket-tts)
- [Pocket TTS Blog](https://kyutai.org/blog/2026-01-13-pocket-tts)
- [Kyutai Labs](https://kyutai.org/tts)

**MLX-Audio:**
- [MLX-Audio GitHub](https://github.com/Blaizzy/mlx-audio)
- [MLX-Audio PyPI](https://pypi.org/project/mlx-audio/)
- [MLX-Audio Tutorial](https://blog.johnys.io/local-text-to-speech-tts-and-voice-cloning-with-mlx-audio/)

**ElevenLabs:**
- [ElevenLabs API Documentation](https://elevenlabs.io/docs/api-reference/text-to-speech/convert)
- [ElevenLabs Voice Library](https://elevenlabs.io/app/voice-library)
- [Get API Key](https://elevenlabs.io/app/settings/api-keys)

**Coqui TTS:**
- [Coqui TTS GitHub](https://github.com/coqui-ai/TTS)
- [Coqui TTS Documentation](https://docs.coqui.ai/)
- [Coqui TTS PyPI](https://pypi.org/project/TTS/)
