---
name: skill-elevenlabs-tts-tool
description: ElevenLabs text-to-speech CLI tool guide
---

# When to use
- Converting text to speech with ElevenLabs API
- Exploring available voices and models
- Managing TTS subscriptions and usage
- Integrating TTS into workflows and pipelines

# ElevenLabs TTS Tool Skill

## Purpose

Comprehensive guide for the `elevenlabs-tts-tool` CLI - a professional command-line interface for ElevenLabs text-to-speech synthesis. Provides both direct audio playback and file output with support for 42+ premium voices and multiple models.

## When to Use This Skill

**Use this skill when:**
- Converting text to speech for notifications, audiobooks, or content creation
- Exploring and comparing different voice characteristics
- Managing ElevenLabs subscription quotas and usage
- Building voice-enabled workflows and automation
- Integrating TTS into Claude Code hooks or other tools

**Do NOT use this skill for:**
- Direct ElevenLabs API programming (use SDK docs instead)
- Custom voice cloning (requires ElevenLabs web interface)
- Real-time streaming TTS (tool focuses on file/playback generation)

## CLI Tool: elevenlabs-tts-tool

Professional text-to-speech CLI tool built with Python 3.13+, uv, and the ElevenLabs SDK.

### Installation

```bash
# Clone repository
git clone https://github.com/dnvriend/elevenlabs-tts-tool.git
cd elevenlabs-tts-tool

# Install globally with uv
uv tool install .

# Verify installation
elevenlabs-tts-tool --version
```

### Prerequisites

- **Python**: 3.13 or higher
- **API Key**: ElevenLabs API key (get from https://elevenlabs.io/app/settings/api-keys)
- **Environment Variable**: `export ELEVENLABS_API_KEY='your-api-key'`

### Quick Start

```bash
# Set API key
export ELEVENLABS_API_KEY='your-api-key'

# Basic text-to-speech
elevenlabs-tts-tool synthesize "Hello world"

# Use different voice
elevenlabs-tts-tool synthesize "Hello" --voice adam

# Save to file
elevenlabs-tts-tool synthesize "Text" --output speech.mp3
```

## Progressive Disclosure

<details>
<summary><strong>📖 Core Commands (Click to expand)</strong></summary>

### synthesize - Convert Text to Speech

Convert text to speech using ElevenLabs API. Supports direct playback or file output.

**Usage:**
```bash
elevenlabs-tts-tool synthesize [TEXT] [OPTIONS]
```

**Arguments:**
- `TEXT`: Text to synthesize (optional if --stdin used)
- `--stdin, -s`: Read text from stdin instead of argument
- `--voice, -v NAME`: Voice name or ID (default: rachel)
- `--model, -m ID`: Model ID (default: eleven_turbo_v2_5)
- `--output, -o PATH`: Save to audio file instead of playing
- `--format, -f FORMAT`: Output format (default: mp3_44100_128)

**Examples:**
```bash
# Basic usage - play through speakers
elevenlabs-tts-tool synthesize "Hello world"

# Use different voice
elevenlabs-tts-tool synthesize "Hello" --voice adam

# Use specific model
elevenlabs-tts-tool synthesize "Hello" --model eleven_multilingual_v2

# Emotional expression (requires eleven_v3 model)
elevenlabs-tts-tool synthesize "[happy] Welcome to our service!" --model eleven_v3

# Multiple emotions
elevenlabs-tts-tool synthesize "[excited] Great news! [cheerfully] Your project is approved!" --model eleven_v3

# Add pauses with SSML
elevenlabs-tts-tool synthesize "Point one <break time=\"0.5s\" /> Point two <break time=\"0.5s\" /> Point three."

# Read from stdin
echo "Text from pipeline" | elevenlabs-tts-tool synthesize --stdin

# Save to file
elevenlabs-tts-tool synthesize "Text" --output speech.mp3

# Pipeline integration
cat document.txt | elevenlabs-tts-tool synthesize --stdin --output audiobook.mp3
```

**Output:**
Plays audio through default speakers or saves to specified file format.

**Available Formats:**
- `mp3_44100_128` (default): MP3, 44.1kHz, 128kbps
- `mp3_44100_64`: MP3, 44.1kHz, 64kbps
- `mp3_22050_32`: MP3, 22.05kHz, 32kbps
- `pcm_44100`: PCM WAV, 44.1kHz (requires Pro tier)

---

### list-voices - Show Available Voices

List all available ElevenLabs voices with characteristics.

**Usage:**
```bash
elevenlabs-tts-tool list-voices
```

**Examples:**
```bash
# List all voices
elevenlabs-tts-tool list-voices

# Filter by gender
elevenlabs-tts-tool list-voices | grep female
elevenlabs-tts-tool list-voices | grep male

# Filter by accent
elevenlabs-tts-tool list-voices | grep British
elevenlabs-tts-tool list-voices | grep American

# Filter by age
elevenlabs-tts-tool list-voices | grep young
elevenlabs-tts-tool list-voices | grep middle_aged

# Combine filters
elevenlabs-tts-tool list-voices | grep "female.*young.*British"
```

**Output:**
```
Voice           Gender     Age          Accent          Description
====================================================================================================
rachel          female     young        American        Calm and friendly American voice...
adam            male       middle_aged  American        Deep, authoritative American male...
charlotte       female     middle_aged  British         Smooth, professional British voice...
...
====================================================================================================
Total: 42 voices available
```

**Popular Voices:**
- **rachel**: Calm, friendly American female (default)
- **adam**: Deep, authoritative American male
- **charlotte**: Professional British female
- **josh**: Young, casual American male
- **bella**: Expressive Italian female

---

### list-models - Show TTS Models

List all available ElevenLabs TTS models with characteristics and use cases.

**Usage:**
```bash
elevenlabs-tts-tool list-models
```

**Examples:**
```bash
# List all models
elevenlabs-tts-tool list-models

# Filter by status
elevenlabs-tts-tool list-models | grep stable
elevenlabs-tts-tool list-models | grep deprecated

# Find low-latency models
elevenlabs-tts-tool list-models | grep -i "ultra-low"

# Find multilingual models
elevenlabs-tts-tool list-models | grep -i "multilingual"
```

**Output:**
Comprehensive model information including:
- Model ID and version
- Quality and latency characteristics
- Language support (mono vs multilingual)
- Character limits
- Best use cases
- Special features (emotions, etc.)

**Key Models:**
- **eleven_turbo_v2_5**: Fast, high-quality (default, best value)
- **eleven_flash_v2_5**: Ultra-low latency (real-time applications)
- **eleven_multilingual_v2**: 29 languages, production quality
- **eleven_v3**: Most expressive with emotion tags (alpha, 2x cost)

**Cost Multipliers:**
- Turbo/Flash models: 1x cost
- Multilingual v2: 1x cost
- v3 models: 2x cost (half the minutes/tokens)

---

### info - Show Subscription Info

Display subscription tier, character usage, quota limits, and historical usage.

**Usage:**
```bash
elevenlabs-tts-tool info [--days N]
```

**Arguments:**
- `--days, -d N`: Number of days of historical usage to display (default: 7)

**Examples:**
```bash
# View subscription with last 7 days of usage
elevenlabs-tts-tool info

# View last 30 days of usage
elevenlabs-tts-tool info --days 30

# Quick quota check (1 day)
elevenlabs-tts-tool info --days 1

# Check usage before long generation
elevenlabs-tts-tool info --days 1 && elevenlabs-tts-tool synthesize "Long text..."
```

**Output Information:**
- Subscription tier and status
- Character usage (used/limit/remaining)
- Quota reset date
- Historical usage breakdown by day
- Average daily usage
- Projected monthly usage
- Warnings when approaching quota limits

**Use Cases:**
- Monitor character quota consumption
- Track usage patterns over time
- Plan when to upgrade subscription tier
- Avoid hitting quota limits unexpectedly
- Identify high-usage periods

---

### update-voices - Update Voice Table

Fetch latest voices from ElevenLabs API and update local lookup table.

**Usage:**
```bash
elevenlabs-tts-tool update-voices [--output PATH]
```

**Arguments:**
- `--output, -o PATH`: Output file path (default: ~/.config/elevenlabs-tts-tool/voices_lookup.json)

**Examples:**
```bash
# Update default voice lookup (user config directory)
elevenlabs-tts-tool update-voices

# Save to custom location
elevenlabs-tts-tool update-voices --output custom_voices.json

# Update before listing voices
elevenlabs-tts-tool update-voices && elevenlabs-tts-tool list-voices
```

**Behavior:**
- Fetches all premade voices from ElevenLabs API
- Saves to user config directory by default (`~/.config/elevenlabs-tts-tool/`)
- Creates config directory if it doesn't exist
- Updates take precedence over package default
- Persists across package reinstalls

---

### pricing - Show Pricing Information

Display ElevenLabs pricing tiers and feature comparison.

**Usage:**
```bash
elevenlabs-tts-tool pricing
```

**Examples:**
```bash
# View full pricing table
elevenlabs-tts-tool pricing

# Find specific tier information
elevenlabs-tts-tool pricing | grep Creator
elevenlabs-tts-tool pricing | grep "44.1kHz PCM"
```

**Output Information:**
- Pricing tiers (Free, Starter, Creator, Pro, Scale, Business)
- Minutes included per tier
- Additional minute costs
- Audio quality options
- Concurrency limits
- Priority levels
- API formats by tier
- Model cost multipliers

**Key Insights:**
- Free tier: 10,000-20,000 characters/month
- v3 models cost 2x (half the minutes/tokens)
- Use Flash v2.5 for high-volume integrations
- Reserve v3 for content requiring emotional expression
- PCM 44.1kHz requires Pro tier

---

### completion - Shell Completion

Generate shell completion scripts for bash, zsh, or fish.

**Usage:**
```bash
elevenlabs-tts-tool completion [bash|zsh|fish]
```

**Installation:**
```bash
# Bash (add to ~/.bashrc)
eval "$(elevenlabs-tts-tool completion bash)"

# Zsh (add to ~/.zshrc)
eval "$(elevenlabs-tts-tool completion zsh)"

# Fish (save to completion file)
elevenlabs-tts-tool completion fish > ~/.config/fish/completions/elevenlabs-tts-tool.fish
```

**Features:**
- Tab-complete commands and subcommands
- Tab-complete options and flags
- Context-aware completion for file paths

</details>

<details>
<summary><strong>⚙️  Advanced Features (Click to expand)</strong></summary>

### Emotion Control (v3 Models)

ElevenLabs v3 model (`eleven_v3`) supports **Audio Tags** for emotional expression.

**Available Emotion Tags:**
- Basic emotions: `[happy]`, `[excited]`, `[sad]`, `[angry]`, `[nervous]`, `[curious]`
- Delivery styles: `[cheerfully]`, `[playfully]`, `[mischievously]`, `[resigned tone]`, `[flatly]`, `[deadpan]`
- Speech characteristics: `[whispers]`, `[laughs]`, `[gasps]`, `[sighs]`, `[pauses]`, `[hesitates]`, `[stammers]`, `[gulps]`

**Usage Examples:**
```bash
# Basic emotion (requires eleven_v3 model)
elevenlabs-tts-tool synthesize "[happy] Welcome to our service!" --model eleven_v3

# Multiple emotions in sequence
elevenlabs-tts-tool synthesize "[excited] Great news! [cheerfully] Your project is approved!" --model eleven_v3

# Combine emotions with pauses
elevenlabs-tts-tool synthesize "[happy] Hello! <break time=\"0.5s\" /> [curious] How are you today?" --model eleven_v3

# Whispered speech
elevenlabs-tts-tool synthesize "[whispers] This is a secret message." --model eleven_v3

# Playful delivery
elevenlabs-tts-tool synthesize "[playfully] Guess what I found!" --model eleven_v3
```

**Best Practices:**
- Place tags at the beginning of phrases
- Align text content with emotional intent
- Test with different voices for best results
- Use sparingly - let AI infer emotion from context when possible
- Remember: v3 models cost 2x as much (half the minutes/tokens)

---

### Pause Control (SSML)

Add natural pauses using SSML `<break>` tags.

**Syntax:**
```xml
<break time="X.Xs" />
```

**Examples:**
```bash
# 1-second pause
elevenlabs-tts-tool synthesize "Welcome <break time=\"1.0s\" /> to our service."

# Multiple pauses
elevenlabs-tts-tool synthesize "Point one <break time=\"0.5s\" /> Point two <break time=\"0.5s\" /> Point three."

# Short pause for emphasis
elevenlabs-tts-tool synthesize "Think about this <break time=\"0.3s\" /> carefully."

# Combine with emotions (requires eleven_v3)
elevenlabs-tts-tool synthesize "[happy] Hello! <break time=\"0.5s\" /> [cheerfully] How are you?" --model eleven_v3
```

**Limitations:**
- Maximum pause duration: 3 seconds
- Recommended: 2-4 breaks per generation
- Too many breaks can cause:
  - AI speedup
  - Audio artifacts
  - Background noise
  - Generation instability

**Alternative Methods:**
- Dashes (`-` or `—`) for shorter pauses (less consistent)
- Ellipses (`...`) for hesitation (may add nervous tone)
- SSML `<break>` is most reliable

---

### Verbosity Control

Multi-level verbosity for progressive detail control.

**Verbosity Levels:**
- **No flag** (default): WARNING level - only critical issues
- **`-v`**: INFO level - high-level operations, important events
- **`-vv`**: DEBUG level - detailed operations, API calls, validation steps
- **`-vvv`**: TRACE level - full HTTP requests/responses, ElevenLabs SDK internals

**Usage:**
```bash
# Quiet mode (warnings only)
elevenlabs-tts-tool synthesize "Hello world"

# INFO level
elevenlabs-tts-tool -v synthesize "Hello world"

# DEBUG level (detailed operations)
elevenlabs-tts-tool -vv synthesize "Hello world"

# TRACE level (shows HTTP requests/responses)
elevenlabs-tts-tool -vvv synthesize "Hello world"
```

**Dependent Library Logging:**
At trace level (`-vvv`), the following libraries enable DEBUG logging:
- `elevenlabs` - ElevenLabs SDK internals
- `httpx` / `httpcore` - HTTP request/response details
- `urllib3` - Low-level HTTP operations

---

### Pipeline Integration

The tool is designed for composition with other CLI tools.

**Design Principles:**
- JSON output to stdout, logs/errors to stderr
- Stdin support for text input
- Exit codes for success/failure detection
- Shell completion for productivity

**Examples:**
```bash
# Read from file
cat document.txt | elevenlabs-tts-tool synthesize --stdin --output audio.mp3

# Combine with other tools
gemini-google-search-tool query "AI news" | \
    elevenlabs-tts-tool synthesize --stdin --voice charlotte --output news.mp3

# Conditional execution
make build && elevenlabs-tts-tool synthesize "[cheerfully] Build successful!" || \
    elevenlabs-tts-tool synthesize "[sad] Build failed. Check logs."

# Process multiple texts
for text in "First" "Second" "Third"; do
    elevenlabs-tts-tool synthesize "$text" --output "${text}.mp3"
done
```

---

### Claude Code Integration

Use `elevenlabs-tts-tool` as notification system for Claude Code hooks.

**Use Cases:**

1. **Task Completion Alerts**
```bash
# After long-running task
elevenlabs-tts-tool synthesize "[excited] Task completed successfully!"
```

2. **Error Notifications**
```bash
# On build failure
elevenlabs-tts-tool synthesize "[nervous] Error detected. Please check output."
```

3. **Custom Workflows**
```bash
# Shell script integration
make build && elevenlabs-tts-tool synthesize "[cheerfully] Build successful!" || \
    elevenlabs-tts-tool synthesize "[sad] Build failed. Check logs."
```

4. **Multi-Tool Integration**
```bash
# Combine with other CLI tools
gemini-google-search-tool query "AI news" | \
    elevenlabs-tts-tool synthesize --stdin --voice charlotte --output news.mp3
```

**Hook Configuration:**

Create hooks in `~/.config/claude-code/hooks.json`:

```json
{
  "hooks": {
    "after_command": {
      "type": "bash",
      "command": "elevenlabs-tts-tool synthesize \"[happy] Task completed!\" --voice rachel"
    },
    "on_error": {
      "type": "bash",
      "command": "elevenlabs-tts-tool synthesize \"[nervous] Error occurred!\" --voice adam"
    }
  }
}
```

**Benefits:**
- Audio alerts for completed tasks without monitoring terminal
- Error notifications while away from screen
- Multi-step automation with voice feedback
- Voice-enabled AI agent pipelines

</details>

<details>
<summary><strong>🔧 Troubleshooting (Click to expand)</strong></summary>

### Common Issues

**Issue: "API key not found" error**
```bash
# Symptom
Error: ELEVENLABS_API_KEY environment variable not set
```

**Solution:**
1. Get API key from https://elevenlabs.io/app/settings/api-keys
2. Export as environment variable:
   ```bash
   export ELEVENLABS_API_KEY='your-api-key'
   ```
3. Add to shell profile for persistence:
   ```bash
   echo 'export ELEVENLABS_API_KEY="your-api-key"' >> ~/.bashrc
   source ~/.bashrc
   ```

---

**Issue: "Voice not found" error**
```bash
# Symptom
ValueError: Voice 'unknown' not found in lookup table
```

**Solution:**
1. List available voices:
   ```bash
   elevenlabs-tts-tool list-voices
   ```
2. Update voice table if needed:
   ```bash
   elevenlabs-tts-tool update-voices
   ```
3. Use correct voice name (case-insensitive):
   ```bash
   elevenlabs-tts-tool synthesize "Hello" --voice rachel
   ```

---

**Issue: Character quota exceeded**
```bash
# Symptom
Error: Character quota exceeded for this month
```

**Solution:**
1. Check current usage:
   ```bash
   elevenlabs-tts-tool info
   ```
2. Wait until quota reset date
3. Consider upgrading subscription tier:
   ```bash
   elevenlabs-tts-tool pricing
   ```
4. Use more efficient models (Flash/Turbo vs v3)

---

**Issue: Audio quality issues**

**Symptom:** Poor audio quality or artifacts

**Solution:**
1. Try different output format:
   ```bash
   elevenlabs-tts-tool synthesize "Text" --format mp3_44100_128
   ```
2. Use higher-quality model:
   ```bash
   elevenlabs-tts-tool synthesize "Text" --model eleven_multilingual_v2
   ```
3. For professional content, use PCM format (requires Pro tier):
   ```bash
   elevenlabs-tts-tool synthesize "Text" --format pcm_44100
   ```

---

**Issue: Emotional tags not working**

**Symptom:** Emotion tags like `[happy]` are spoken literally

**Solution:**
1. Ensure using v3 model:
   ```bash
   elevenlabs-tts-tool synthesize "[happy] Text" --model eleven_v3
   ```
2. Place tags at beginning of phrases
3. Test with different voices (some work better than others)

---

**Issue: Too many SSML breaks causing issues**

**Symptom:** Audio artifacts, speedup, or noise with multiple `<break>` tags

**Solution:**
1. Limit to 2-4 breaks per generation
2. Use maximum 3 seconds per break
3. Consider splitting into multiple generations:
   ```bash
   elevenlabs-tts-tool synthesize "Part 1" --output part1.mp3
   elevenlabs-tts-tool synthesize "Part 2" --output part2.mp3
   ```

---

### Getting Help

```bash
# Main help
elevenlabs-tts-tool --help

# Command-specific help
elevenlabs-tts-tool synthesize --help
elevenlabs-tts-tool list-voices --help
elevenlabs-tts-tool info --help

# Version information
elevenlabs-tts-tool --version
```

**Additional Resources:**
- **GitHub Issues**: https://github.com/dnvriend/elevenlabs-tts-tool/issues
- **ElevenLabs Docs**: https://elevenlabs.io/docs
- **API Reference**: https://elevenlabs.io/docs/api-reference

</details>

## Free Tier Limitations

**ElevenLabs Free Tier (2024-2025):**
- ✅ 10,000-20,000 characters per month
- ✅ All 42 premade voices
- ✅ Create up to 3 custom voices
- ✅ MP3 formats (all bitrates)
- ✅ Basic SSML support (`<break>`, phonemes)
- ✅ Emotional tags (v3 models)
- ✅ Full API access
- ❌ No commercial license (personal/experimentation only)
- ❌ PCM 44.1kHz format (requires Pro tier)
- ⚠️ Max 2,500 characters per single generation

**Upgrade Tiers:**
- **Starter ($5/month)**: 30,000 characters, commercial license
- **Creator ($22/month)**: 100,000 characters, PCM formats
- **Pro ($99/month)**: 500,000 characters, PCM 44.1kHz, highest priority
- **Scale ($330/month)**: 2,000,000 characters
- **Business (custom)**: Custom limits and features

**Rate Limits:** Not publicly documented - expect reasonable use restrictions on free tier

## Exit Codes

- `0`: Success
- `1`: General error (validation, API error, etc.)

## Output Formats

**Audio Formats:**
- `mp3_44100_128`: MP3, 44.1kHz, 128kbps (default, best quality)
- `mp3_44100_64`: MP3, 44.1kHz, 64kbps (good quality, smaller)
- `mp3_22050_32`: MP3, 22.05kHz, 32kbps (acceptable quality, smallest)
- `pcm_44100`: PCM WAV, 44.1kHz, uncompressed (requires Pro tier)

**Text Formats:**
- Human-readable tables for list commands
- Structured output with clear sections
- Errors to stderr, audio/data to stdout

## Best Practices

1. **Use Turbo v2.5 for High Volume**: Default model offers best value (1x cost, fast, high quality)
2. **Reserve v3 for Emotional Content**: Use v3 only when emotion tags needed (costs 2x)
3. **Monitor Quota Regularly**: Check `info` command before large generations
4. **Update Voices Periodically**: Run `update-voices` monthly to get latest voices
5. **Test Voices for Your Use Case**: Different voices work better for different content types
6. **Use SSML Breaks Sparingly**: Limit to 2-4 breaks per generation for stability
7. **Pipeline for Efficiency**: Combine with other tools for automated workflows
8. **Set Verbosity Appropriately**: Use `-vv` or `-vvv` for debugging, default for production

## Resources

- **GitHub Repository**: https://github.com/dnvriend/elevenlabs-tts-tool
- **ElevenLabs Documentation**: https://elevenlabs.io/docs
- **API Reference**: https://elevenlabs.io/docs/api-reference
- **Voice Library**: https://elevenlabs.io/voice-library
- **Python SDK**: https://github.com/elevenlabs/elevenlabs-python
- **Claude Code**: https://docs.anthropic.com/claude-code
