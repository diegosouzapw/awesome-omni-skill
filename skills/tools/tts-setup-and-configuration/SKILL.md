---
name: TTS Setup and Configuration
description: This skill activates when users ask about text-to-speech setup, configuration, troubleshooting, voice selection, or TTS functionality. Trigger phrases include "how do I set up TTS", "configure text to speech", "TTS not working", "change TTS voice", "TTS help", "troubleshoot TTS", "TTS speed", "disable TTS", or any questions about the TTS plugin functionality.
version: 1.0.0
---

# TTS Setup and Configuration for Claude Code

Provide comprehensive guidance for setting up, configuring, and troubleshooting the TTS (text-to-speech) plugin for Claude Code. This skill covers installation prerequisites, configuration options, voice selection, troubleshooting, and usage patterns.

## Prerequisites and Installation

### kokoro-tts Installation

Before using this plugin, kokoro-tts must be installed. Guide the user through verification:

1. **Check if kokoro-tts is installed**:
   ```bash
   which kokoro-tts
   ```

2. **If not installed**, direct user to installation:
   - Official repository: https://github.com/thewh1teagle/kokoro
   - Follow installation instructions for their operating system
   - Verify installation: `kokoro-tts --help`

3. **Verify model files exist**:
   - Model: `~/.local/share/kokoro-tts/kokoro-v1.0.onnx`
   - Voices: `~/.local/share/kokoro-tts/voices-v1.0.bin`
   - If missing, kokoro-tts installation may be incomplete

### Plugin Installation

The TTS plugin should already be installed if the user is asking about it. If not:

1. Install plugin to Claude Code plugins directory
2. Enable in Claude Code settings or with `cc --plugin-dir`
3. Restart Claude Code for hooks to load

## Initial Configuration

### Quick Start

Guide user through quick configuration:

1. **Run the configure command**:
   ```bash
   /tts-plugin:configure
   ```

2. **Choose "Quick Setup"** for common settings:
   - TTS_ENABLED: true (to enable TTS)
   - TTS_VOICE: Select preferred voice (af_bella, af_sarah, etc.)
   - TTS_LANG: Select language (en-gb, en-us, etc.)
   - TTS_SPEED: Set speech speed (1.0-1.5 recommended)

3. **Save and restart Claude Code** for changes to take effect

### Test TTS

After configuration, test that it works:

```bash
/tts-plugin:test
```

Expected: Should hear "This is a test of the text to speech system"

## Voice and Language Selection

### Available Voices

Run `kokoro-tts --help-voices` to see all available voices.

Common voices:
- **af_bella** - Female, British accent (default)
- **af_sarah** - Female, British accent
- **af_sky** - Female, British accent
- **bf_emma** - Female, British accent
- **bf_isabella** - Female, British accent
- **am_adam** - Male, British accent
- **am_michael** - Male, British accent

### Available Languages

Run `kokoro-tts --help-languages` to see all available languages.

Common languages:
- **en-gb** - English (British) (default)
- **en-us** - English (American)
- **es** - Spanish
- **fr** - French
- **de** - German
- **ja** - Japanese

### Testing Different Voices

Test voices without changing configuration:

```bash
/tts-plugin:test --voice af_sarah
/tts-plugin:test --voice bf_emma --speed 1.5
/tts-plugin:test "Custom message" --voice am_michael
```

Once you find a voice you like, save it with `/tts-plugin:configure`.

## Configuration Options

### Basic Settings

**TTS_ENABLED** (true/false)
- Master enable/disable switch
- Affects all TTS hooks
- Default: true

**TTS_VOICE** (voice name)
- Voice to use for speech
- See kokoro-tts --help-voices for options
- Default: af_bella

**TTS_LANG** (language code)
- Language for speech
- See kokoro-tts --help-languages for options
- Default: en-gb

**TTS_SPEED** (0.5-2.0)
- Speech speed multiplier
- 1.0 = normal speed
- 1.3-1.5 = faster but still clear (recommended)
- Default: 1.3

### Advanced Settings

**TTS_PRETOOL_ENABLED** (true/false)
- Enable PreToolUse hook (speaks before each tool execution)
- Can be verbose, disabled by default
- Only active if TTS_ENABLED=true
- Default: false

**TTS_USE_TTS_SECTION** (true/false)
- Extract "## TTS Response" section from Claude's responses
- When true, only speaks TTS-optimized content
- When false, speaks entire response
- Default: true

**TTS_MAX_LENGTH** (number)
- Maximum characters to speak per message
- Prevents extremely long TTS
- Default: 5000

**TTS_STATE_DIR** (path)
- Directory for session state files
- Tracks what's been spoken to avoid repeats
- Default: ~/.local/state/claude-tts/session-state

**TTS_LOG_DIR** (path)
- Directory for TTS log files
- Useful for debugging
- Default: ~/.local/state/claude-tts/logs

### Configuration File Location

All settings stored in: `~/.claude/tts-plugin.env`

Edit this file directly or use `/tts-plugin:configure` for interactive setup.

## How TTS Works

### Event Flow

1. **User submits prompt** (UserPromptSubmit hook):
   - Interrupts any ongoing TTS playback
   - Injects instruction asking Claude to add "## TTS Response" section
   - Marks current messages as read

2. **Claude generates response**:
   - Includes TTS-optimized content in "## TTS Response" section
   - This section has markdown removed, simplified paths, natural language

3. **Claude finishes responding** (Stop hook):
   - Extracts new messages from transcript
   - Looks for "## TTS Response" section
   - Speaks the text using kokoro-tts
   - Updates session state

4. **Optional: Before each tool** (PreToolUse hook, if enabled):
   - Speaks any new text that appeared
   - Provides real-time audio feedback

5. **Session ends** (SessionEnd hook):
   - Cleans up session state files

### TTS Response Section

Claude adds a "## TTS Response" section to responses when TTS is active. This section contains:
- Same information as main response
- Markdown formatting removed
- URLs converted to domain names
- File paths simplified to filenames
- Natural, spoken language

Users see both the normal response AND the TTS Response section. TTS only speaks the TTS Response section (if TTS_USE_TTS_SECTION=true).

## Common Use Cases

### Enable TTS for Session

Temporarily enable TTS without saving:
```bash
/tts-plugin:enable
```

### Enable TTS Permanently

Save enable setting to .env:
```bash
/tts-plugin:enable --persistent
```

### Disable TTS Temporarily

Stop TTS for current session:
```bash
/tts-plugin:disable
```

### Disable TTS Permanently

Save disable setting to .env:
```bash
/tts-plugin:disable --persistent
```

### Change Voice

```bash
/tts-plugin:configure
# Choose Quick Setup
# Select new voice
```

### Adjust Speed

```bash
/tts-plugin:configure
# Choose Quick Setup
# Set new speed (1.0-2.0)
```

### Enable PreToolUse Hook

For real-time TTS during tool execution:
```bash
/tts-plugin:configure
# Choose Advanced Setup
# Set TTS_PRETOOL_ENABLED=true
```

Note: This can be verbose as it speaks before each tool.

## Troubleshooting

### TTS Not Speaking

**Problem**: TTS is enabled but nothing plays

**Solutions**:
1. Verify kokoro-tts is installed: `which kokoro-tts`
2. Test TTS directly: `/tts-plugin:test`
3. Check TTS_ENABLED in configuration: `/tts-plugin:configure`
4. Verify system audio is working
5. Check logs: `~/.local/state/claude-tts/logs/`
6. Restart Claude Code (hooks load at startup)

### TTS Interrupted Immediately

**Problem**: TTS starts but stops right away

**Solutions**:
1. Don't submit new prompts while TTS is speaking
2. UserPromptSubmit hook intentionally interrupts TTS
3. This is normal behavior to prevent overlapping speech

### Wrong Voice or Language

**Problem**: TTS uses different voice than expected

**Solutions**:
1. Check current configuration: `/tts-plugin:configure` → View Current Settings
2. Update configuration: `/tts-plugin:configure` → Quick Setup
3. Restart Claude Code after changes
4. Test new settings: `/tts-plugin:test`

### TTS Too Fast/Slow

**Problem**: Speech speed is uncomfortable

**Solutions**:
1. Test different speeds: `/tts-plugin:test --speed 1.0` (try 0.8-1.5)
2. Save preferred speed: `/tts-plugin:configure` → Quick Setup → Set TTS_SPEED
3. Restart Claude Code after changes

### TTS Too Verbose

**Problem**: TTS speaks too much or too often

**Solutions**:
1. Disable PreToolUse hook: `/tts-plugin:configure` → Advanced → TTS_PRETOOL_ENABLED=false
2. Reduce max length: `/tts-plugin:configure` → Advanced → TTS_MAX_LENGTH=2000
3. Disable completely: `/tts-plugin:disable --persistent`

### TTS Speaks Full Response

**Problem**: TTS speaks entire response instead of optimized section

**Solutions**:
1. Ensure TTS_USE_TTS_SECTION=true in configuration
2. Verify Claude is adding "## TTS Response" section (check responses)
3. If section missing, TTS instruction injection may not be working
4. Check UserPromptSubmit hook is active (restart Claude Code)

### Model Files Not Found

**Problem**: Error about missing model or voices files

**Solutions**:
1. Verify model file exists: `ls ~/.local/share/kokoro-tts/kokoro-v1.0.onnx`
2. Verify voices file exists: `ls ~/.local/share/kokoro-tts/voices-v1.0.bin`
3. If missing, reinstall kokoro-tts
4. Update paths in configuration if files are in different location

### Changes Not Taking Effect

**Problem**: Configuration changes don't work

**Solutions**:
1. **Restart Claude Code** - hooks and configuration load at startup
2. Verify changes saved to ~/.claude/tts-plugin.env
3. Check for syntax errors in .env file
4. Test with `/tts-plugin:test` to verify settings

## Best Practices

### Recommended Settings

For most users:
```bash
TTS_ENABLED=true
TTS_PRETOOL_ENABLED=false       # Disable verbose PreToolUse
TTS_VOICE=af_bella              # Or your preferred voice
TTS_LANG=en-gb                  # Or your preferred language
TTS_SPEED=1.3                   # Faster but clear
TTS_USE_TTS_SECTION=true        # Use optimized content
TTS_MAX_LENGTH=5000             # Prevent overly long TTS
```

### When to Enable PreToolUse Hook

**Enable** if you want:
- Real-time audio feedback during long operations
- To hear progress updates as tools execute
- Maximum awareness of what Claude is doing

**Disable** if you want:
- Less verbose TTS (recommended)
- Only hear final responses
- Avoid hearing tool execution messages

### Speed Recommendations

- **0.8-1.0**: Slower, easier to understand, good for learning
- **1.0-1.3**: Normal speed, comfortable listening
- **1.3-1.5**: Faster but clear, efficient (recommended)
- **1.5-2.0**: Very fast, may be hard to follow

### Voice Selection Tips

1. Test multiple voices: `/tts-plugin:test --voice VOICE_NAME`
2. Consider accent preference (British vs American)
3. Consider gender preference (male vs female voices)
4. Listen for clarity at your preferred speed
5. Save favorite: `/tts-plugin:configure`

## Advanced Topics

### Temporarily Disabling TTS

To quickly silence TTS without changing configuration:
```bash
/tts-plugin:disable
```

Re-enable when ready:
```bash
/tts-plugin:enable
```

### Session State Management

TTS tracks what's been spoken in session state files:
- Location: ~/.local/state/claude-tts/session-state/
- Files named: claude-tts-state-{session_id}.txt
- Automatically cleaned up on SessionEnd
- Prevents re-speaking already-heard messages

### Log Files

TTS logs to:
- Location: ~/.local/state/claude-tts/logs/
- Files named: claude-tts-{session_id}.log
- Useful for debugging
- Check these if TTS isn't working

### Custom Model/Voices Paths

If kokoro-tts files are in non-standard location:
```bash
/tts-plugin:configure
# Choose Advanced Setup
# Set TTS_MODEL=/custom/path/to/model.onnx
# Set TTS_VOICES=/custom/path/to/voices.bin
```

## Quick Reference

### Commands

- `/tts-plugin:enable [--persistent]` - Enable TTS
- `/tts-plugin:disable [--persistent]` - Disable TTS
- `/tts-plugin:configure` - Configure settings
- `/tts-plugin:test [message] [--voice V] [--speed S] [--lang L]` - Test TTS

### Configuration File

- **Location**: ~/.claude/tts-plugin.env
- **Template**: Plugin includes .env.example
- **Edit**: Use `/tts-plugin:configure` or edit file directly

### Key Settings

- **TTS_ENABLED**: Master on/off switch
- **TTS_VOICE**: Voice selection
- **TTS_SPEED**: Speech speed (0.5-2.0)
- **TTS_PRETOOL_ENABLED**: Enable verbose PreToolUse hook

### Getting Help

1. Ask about TTS setup: "How do I configure TTS?"
2. Run test: `/tts-plugin:test`
3. Check configuration: `/tts-plugin:configure` → View Current Settings
4. Check logs: `~/.local/state/claude-tts/logs/`
5. Check kokoro-tts: `kokoro-tts --help`

---

When helping users with TTS:
1. Start by understanding their issue or goal
2. Guide them through relevant commands
3. Provide specific troubleshooting steps
4. Test changes with `/tts-plugin:test`
5. Remind to restart Claude Code after configuration changes
