---
name: advanced-statusline
description: Implement AI-powered statusline with session tracking, plan detection, workspace emojis, and intelligent caching for Claude Code
tags: [statusline, hooks, shell, starship, caching]
---

# Advanced Claude Code Statusline System

This skill implements a comprehensive statusline system for Claude Code featuring:
- 🤖 AI-powered session summaries using Claude Haiku
- 📝 Auto-generated session names (filters trivial inputs)
- 📋 Plan-session correlation tracking
- 🔧 Real-time assistant tool usage display
- 🦀 Workspace-aware emojis
- ⚡ Starship integration with proper escape sequence handling
- 🧹 Intelligent cache management (100MB threshold, 7-day retention)
- 💤 Idle-friendly (keeps cached values indefinitely until new user action)

## Architecture

### Components

1. **Main Statusline Wrapper** (`statusline-starship-wrapper.sh`)
   - Receives JSON input from Claude Code
   - Integrates with Starship for git/directory info
   - Generates 3-line output with workspace emoji, summary, session info
   - Reads from hook-generated caches

2. **Session Naming Hook** (`session-naming.sh`)
   - Trigger: `UserPromptSubmit`
   - Filters trivial inputs (yes, no, continue, < 10 chars)
   - **Immediate fallback write** before AI call (prevents "Unnamed Session" race condition)
   - Generates concise session names via Claude Haiku (5-second timeout)
   - Regenerates when new messages arrive
   - Uses word-boundary truncation for readable fallbacks

3. **Plan Tracking Hook** (`plan-tracking.sh`)
   - Triggers: `SessionStart`, `UserPromptSubmit`
   - Detects plans created within 10 minutes
   - Maintains session-plan correlation map
   - Shows 📋 indicator when plan is active

4. **Assistant Output Sampling Hook** (`assistant-output-sampling.sh`)
   - Trigger: `PostToolUse` (after every tool execution)
   - Records tool usage patterns
   - Shows top 3 tools as "🔧 Edit, Write, Bash"
   - Updates on every action

5. **Shared Cache Cleanup** (`cache-cleanup.sh`)
   - Runs randomly (1% chance) from all components
   - Only activates when total cache > 100MB
   - Deletes files older than 7 days
   - Protects recent activity

### Data Flow

```
User Input → UserPromptSubmit Hook → session-naming.sh → /tmp/claude-session-name-{session_id}.txt
                                   ↘ plan-tracking.sh → /tmp/claude-session-plan-{session_id}.txt

Tool Execution → PostToolUse Hook → assistant-output-sampling.sh → /tmp/claude-assistant-summary-{session_id}.txt

Statusline Refresh → statusline-starship-wrapper.sh → Reads all caches → Displays 3-line output
```

### Statusline Output Format

```
🦀 Sonnet 4.5 | [starship git status] | 63% ctx
Implementing advanced statusline 🔧 Edit, Write, Bash | abc-123-session-id
Advanced Statusline System 📋 plan-name-if-active
```

## Implementation

### Step 1: Create Hook Scripts

#### File: `~/.claude/hooks/session-naming.sh`

```bash
#!/bin/bash
set -euo pipefail

# Read hook input
input=$(cat)

session_id=$(echo "$input" | jq -r '.session_id // ""')
user_prompt=$(echo "$input" | jq -r '.user_prompt // ""')

# Exit if no session ID or prompt
if [ -z "$session_id" ] || [ -z "$user_prompt" ]; then
    exit 0
fi

# Filter out trivial inputs
if echo "$user_prompt" | grep -qiE '^(yes|no|ok|okay|sure|continue|try again|test it|fix it|run it|check|show|nope|yep|go ahead|proceed)\.?$'; then
    exit 0
fi

# Also filter out very short inputs (< 10 chars)
if [ ${#user_prompt} -lt 10 ]; then
    exit 0
fi

# Cache files
cache_file="/tmp/claude-session-name-${session_id}.txt"
timestamp_file="/tmp/claude-session-name-${session_id}.timestamp"
msg_count_file="/tmp/claude-session-name-${session_id}.msgcount"

# Check if we need to regenerate (similar to summary logic)
should_regenerate=false
history_file="$HOME/.claude/history.jsonl"

# Get current message count
current_msg_count=0
if [ -f "$history_file" ]; then
    current_msg_count=$(grep -c "\"sessionId\":\"${session_id}\"" "$history_file" 2>/dev/null || echo "0")
    current_msg_count=$(echo "$current_msg_count" | tr -d '\n\r ')
fi
current_msg_count=${current_msg_count:-0}

if [ ! -f "$cache_file" ]; then
    # No cache exists, generate new name
    should_regenerate=true
else
    cached_name=$(cat "$cache_file" 2>/dev/null || echo "")
    # If cache is empty or invalid, regenerate
    if [ -z "$cached_name" ] || [ ${#cached_name} -lt 5 ]; then
        should_regenerate=true
    else
        # Check if new messages exist since last generation
        last_msg_count=$(cat "$msg_count_file" 2>/dev/null | tr -d '\n\r ' || echo "0")
        last_msg_count=${last_msg_count:-0}
        if [ "$current_msg_count" -gt "$last_msg_count" ]; then
            should_regenerate=true
        fi
    fi
fi

# If we don't need to regenerate, exit early
if [ "$should_regenerate" = false ]; then
    exit 0
fi

# IMMEDIATELY write a fallback name to prevent "Unnamed Session" race condition
# This ensures cache file exists even if AI call times out or fails
# The AI call below will overwrite this with a better name if successful
fallback_name=""
for word in $user_prompt; do
    if [ -z "$fallback_name" ]; then
        # First word - capitalize it
        fallback_name="$(echo "${word:0:1}" | tr '[:lower:]' '[:upper:]')${word:1}"
    else
        # Check if adding this word would exceed limit
        test_name="$fallback_name $word"
        if [ ${#test_name} -le 37 ]; then
            fallback_name="$test_name"
        else
            # Would exceed limit, add ellipsis and break
            fallback_name="${fallback_name}..."
            break
        fi
    fi
done

# If we got through all words without truncating, don't add ellipsis
if [[ ! "$fallback_name" =~ \.\.\.$ ]]; then
    # Ensure it's not too long anyway
    if [ ${#fallback_name} -gt 40 ]; then
        fallback_name="${fallback_name:0:37}..."
    fi
fi

# Write the fallback name immediately to cache
echo "$fallback_name" > "$cache_file"
date +%s > "$timestamp_file"
echo "$current_msg_count" > "$msg_count_file"

# Generate session name using AI (with timeout)
schema='{"type":"object","properties":{"name":{"type":"string","maxLength":40}},"required":["name"]}'
prompt="Generate a concise session name (3-5 words, max 40 chars) for this task: ${user_prompt}"

ai_temp="/tmp/claude-session-name-ai-${session_id}-$$.json"

# Run with timeout (5 seconds - matches settings.json timeout)
(
    echo "$prompt" | claude --model haiku -p --no-session-persistence --output-format json --json-schema "$schema" 2>/dev/null > "$ai_temp"
) &
claude_pid=$!

for i in {1..5}; do
    if ! kill -0 $claude_pid 2>/dev/null; then
        break
    fi
    sleep 1
done

kill $claude_pid >/dev/null 2>&1 || true
wait $claude_pid >/dev/null 2>&1 || true

# Extract name from AI response
ai_session_name=""
if [ -f "$ai_temp" ] && [ -s "$ai_temp" ]; then
    ai_session_name=$(jq -r '.structured_output.name // empty' "$ai_temp" 2>/dev/null || echo "")
    rm -f "$ai_temp"
fi

# If AI succeeded with a valid name, overwrite the fallback cache
if [ -n "$ai_session_name" ] && [ ${#ai_session_name} -ge 5 ]; then
    echo "$ai_session_name" > "$cache_file"
    date +%s > "$timestamp_file"
    echo "$current_msg_count" > "$msg_count_file"
fi
# Otherwise, keep the immediate fallback that was already written

# Occasionally run shared cleanup (1% of the time to minimize overhead)
if [ $((RANDOM % 100)) -eq 0 ]; then
    bash "${CLAUDE_PLUGIN_ROOT}/hooks/scripts/cache-cleanup.sh" &
fi

exit 0
```

#### File: `~/.claude/hooks/plan-tracking.sh`

```bash
#!/bin/bash
set -euo pipefail

# Read hook input
input=$(cat)

session_id=$(echo "$input" | jq -r '.session_id // ""')
hook_event=$(echo "$input" | jq -r '.hook_event_name // ""')

if [ -z "$session_id" ]; then
    exit 0
fi

# Plan tracking file
plan_map="/tmp/claude-plan-session-map.jsonl"

# On SessionStart or UserPromptSubmit, check for recent plans
if [ "$hook_event" = "SessionStart" ] || [ "$hook_event" = "UserPromptSubmit" ]; then
    # Find most recent plan (within last 10 minutes)
    recent_plan=$(find ~/.claude/plans -name "*.md" -mmin -10 -type f 2>/dev/null | head -1)

    if [ -n "$recent_plan" ]; then
        plan_name=$(basename "$recent_plan" .md)

        # Record mapping
        echo "{\"session_id\":\"${session_id}\",\"plan\":\"${plan_name}\",\"timestamp\":$(date +%s)}" >> "$plan_map"

        # Cache current session's plan
        echo "$plan_name" > "/tmp/claude-session-plan-${session_id}.txt"
    fi
fi

# Keep plan map under control (keep only last 1000 entries)
if [ -f "$plan_map" ]; then
    tail -1000 "$plan_map" > "${plan_map}.tmp" && mv "${plan_map}.tmp" "$plan_map"
fi

# Occasionally run shared cleanup (1% of the time)
if [ $((RANDOM % 100)) -eq 0 ]; then
    bash ~/.claude/hooks/cache-cleanup.sh &
fi

exit 0
```

#### File: `~/.claude/hooks/assistant-output-sampling.sh`

```bash
#!/bin/bash
set -euo pipefail

# Read hook input
input=$(cat)

session_id=$(echo "$input" | jq -r '.session_id // ""')
hook_event=$(echo "$input" | jq -r '.hook_event_name // ""')

if [ -z "$session_id" ]; then
    exit 0
fi

# Only run on PostToolUse
if [ "$hook_event" != "PostToolUse" ]; then
    exit 0
fi

# Extract tool information
tool_name=$(echo "$input" | jq -r '.tool_name // ""')
tool_input=$(echo "$input" | jq -r '.tool_input // {}')

# Skip empty or system tools
if [ -z "$tool_name" ] || [ "$tool_name" = "null" ]; then
    exit 0
fi

# Cache file for assistant actions
actions_cache="/tmp/claude-assistant-actions-${session_id}.jsonl"

# Record action with timestamp
action_entry=$(jq -n \
    --arg tool "$tool_name" \
    --arg timestamp "$(date +%s)" \
    --argjson input "$tool_input" \
    '{tool: $tool, timestamp: $timestamp, input: $input}')

echo "$action_entry" >> "$actions_cache"

# Keep only last 50 actions
if [ -f "$actions_cache" ]; then
    tail -50 "$actions_cache" > "${actions_cache}.tmp" && mv "${actions_cache}.tmp" "$actions_cache"
fi

# Generate summary of recent actions (last 10)
recent_actions=$(tail -10 "$actions_cache" 2>/dev/null | jq -r '.tool' | sort | uniq -c | sort -rn | head -3 | awk '{print $2}' | paste -sd ', ' -)

if [ -n "$recent_actions" ]; then
    echo "$recent_actions" > "/tmp/claude-assistant-summary-${session_id}.txt"
fi

# Occasionally run shared cleanup (1% of the time)
if [ $((RANDOM % 100)) -eq 0 ]; then
    bash ~/.claude/hooks/cache-cleanup.sh &
fi

exit 0
```

#### File: `~/.claude/hooks/cache-cleanup.sh`

```bash
#!/bin/bash
# Shared cache cleanup utility - only cleans when total cache > 100MB
# Keeps recent files (last 7 days) even when cleaning

set -euo pipefail

# Calculate total cache size
cache_size=$(du -sm /tmp/claude-* 2>/dev/null | awk '{sum+=$1} END {print sum+0}')

# Only cleanup if > 100MB
if [ "$cache_size" -lt 100 ]; then
    exit 0
fi

# Cleanup old files (> 7 days) when cache is large
find /tmp -name "claude-session-summary-*.txt" -mtime +7 -delete 2>/dev/null
find /tmp -name "claude-session-summary-*.timestamp" -mtime +7 -delete 2>/dev/null
find /tmp -name "claude-session-summary-*.msgcount" -mtime +7 -delete 2>/dev/null
find /tmp -name "claude-session-name-*.txt" -mtime +7 -delete 2>/dev/null
find /tmp -name "claude-session-name-*.timestamp" -mtime +7 -delete 2>/dev/null
find /tmp -name "claude-session-name-*.msgcount" -mtime +7 -delete 2>/dev/null
find /tmp -name "claude-session-plan-*.txt" -mtime +7 -delete 2>/dev/null
find /tmp -name "claude-assistant-actions-*.jsonl" -mtime +7 -delete 2>/dev/null
find /tmp -name "claude-assistant-summary-*.txt" -mtime +7 -delete 2>/dev/null

# Also cleanup temporary AI files (always clean these up if > 1 hour old)
find /tmp -name "claude-ai-summary-*.json" -mmin +60 -delete 2>/dev/null
find /tmp -name "claude-session-name-ai-*.json" -mmin +60 -delete 2>/dev/null

exit 0
```

### Step 2: Create Main Statusline Script

#### File: `~/.claude/statusline-starship-wrapper.sh`

<details>
<summary>Click to expand full script (233 lines)</summary>

```bash
#!/bin/bash

# Read JSON input from stdin
input=$(cat)

# Extract current directory from JSON
cwd=$(echo "$input" | jq -r '.workspace.current_dir')

# Assign workspace emoji based on path
get_workspace_emoji() {
    local path="$1"
    local basename=$(basename "$path")

    # Semantic mapping for common project types
    case "$basename" in
        *caro*) echo "🦀" ;; # Rust crab
        *rust*) echo "🦀" ;;
        *node*|*npm*|*react*|*next*) echo "📦" ;;
        *python*|*py*) echo "🐍" ;;
        *go*|*golang*) echo "🐹" ;;
        *java*) echo "☕" ;;
        *docker*) echo "🐳" ;;
        *web*|*site*) echo "🌐" ;;
        *api*) echo "🔌" ;;
        *db*|*database*) echo "🗄️" ;;
        *docs*|*documentation*) echo "📚" ;;
        *test*) echo "🧪" ;;
        *)
            # Hash-based consistent emoji for unknown paths
            local emojis=("💼" "📁" "🛠️" "⚙️" "🔧" "📊" "🎯" "🚀" "💡" "🔬")
            local hash=$(echo -n "$path" | cksum | cut -d' ' -f1)
            local index=$((hash % ${#emojis[@]}))
            echo "${emojis[$index]}"
            ;;
    esac
}

workspace_emoji=$(get_workspace_emoji "$cwd")

# Extract model display name
model_name=$(echo "$input" | jq -r '.model.display_name')

# Extract session ID
session_id=$(echo "$input" | jq -r '.session_id')

# Read session name from cache (generated by hook)
session_name_cache="/tmp/claude-session-name-${session_id}.txt"
if [ -f "$session_name_cache" ]; then
    session_name=$(cat "$session_name_cache" 2>/dev/null || echo "")
fi

# Fallback if no cached name
if [ -z "$session_name" ]; then
    session_name=$(echo "$input" | jq -r '.session_name // "Unnamed Session"')
fi

# Read plan info from cache (generated by hook)
plan_cache="/tmp/claude-session-plan-${session_id}.txt"
plan_indicator=""
if [ -f "$plan_cache" ]; then
    plan_name=$(cat "$plan_cache" 2>/dev/null || echo "")
    if [ -n "$plan_name" ]; then
        plan_indicator=" 📋 $plan_name"
    fi
fi

# Read assistant action summary from cache (generated by PostToolUse hook)
assistant_summary_cache="/tmp/claude-assistant-summary-${session_id}.txt"
assistant_actions=""
if [ -f "$assistant_summary_cache" ]; then
    actions=$(cat "$assistant_summary_cache" 2>/dev/null || echo "")
    if [ -n "$actions" ]; then
        assistant_actions=" 🔧 $actions"
    fi
fi

# Calculate context window percentage
usage=$(echo "$input" | jq '.context_window.current_usage')
if [ "$usage" != "null" ]; then
    current=$(echo "$usage" | jq '.input_tokens + .cache_creation_input_tokens + .cache_read_input_tokens')
    size=$(echo "$input" | jq '.context_window.context_window_size')
    context_pct=$((current * 100 / size))
    context_display=" | ${context_pct}% ctx"
else
    context_display=""
fi

# Set up environment variables that Starship expects
export PWD="$cwd"
export STARSHIP_SHELL="bash"

# Change to the directory so git commands work correctly
cd "$cwd" 2>/dev/null || cd "$HOME"

# Run starship with the prompt command and strip the \[\] escape sequences
starship_output=$(starship prompt 2>/dev/null | sed 's/\\\[//g; s/\\\]//g' | tr -d '\n')

# Generate session summary (line 2)
summary_cache="/tmp/claude-session-summary-${session_id}.txt"
summary_timestamp="/tmp/claude-session-summary-${session_id}.timestamp"
summary_msg_count="/tmp/claude-session-summary-${session_id}.msgcount"

# Check if we need to regenerate the summary
# Regenerate ONLY if:
# 1. Cache doesn't exist, OR
# 2. New messages exist since last check
# Idle sessions keep their cached summary indefinitely
should_regenerate=false
history_file="$HOME/.claude/history.jsonl"

# Get current message count
current_msg_count=0
if [ -f "$history_file" ]; then
    current_msg_count=$(grep -c "\"sessionId\":\"${session_id}\"" "$history_file" 2>/dev/null || echo "0")
fi

if [ ! -f "$summary_cache" ]; then
    should_regenerate=true
else
    # Check if new messages exist
    last_msg_count=$(cat "$summary_msg_count" 2>/dev/null || echo "0")
    if [ "$current_msg_count" -gt "$last_msg_count" ]; then
        should_regenerate=true
    fi
fi

if [ "$should_regenerate" = true ]; then
    # Read from history.jsonl instead of transcript
    history_file="$HOME/.claude/history.jsonl"

    if [ -f "$history_file" ]; then
        # Process line-by-line to avoid jq -s parse errors
        first_user_msg=$(grep "\"sessionId\":\"${session_id}\"" "$history_file" | head -1 | jq -r '.display // ""')
        recent_user_msgs=$(grep "\"sessionId\":\"${session_id}\"" "$history_file" | tail -7 | jq -r '.display' | paste -sd ' ' -)
        user_msg_count=$(grep -c "\"sessionId\":\"${session_id}\"" "$history_file")
    else
        first_user_msg=""
        recent_user_msgs=""
        user_msg_count=0
    fi

    # PRIMARY METHOD: AI summary with structured output
    # Use ALL user messages (no keyword filtering - let AI determine what's meaningful)
    all_user_msgs=$(grep "\"sessionId\":\"${session_id}\"" "$history_file" | jq -r '.display')

    # Get first + last 5 user messages for context
    first_user_msgs=$(echo "$all_user_msgs" | head -5 | paste -sd '. ' -)
    last_user_msgs=$(echo "$all_user_msgs" | tail -5 | paste -sd '. ' -)

    # Use structured JSON output with examples
    schema='{"type":"object","properties":{"task":{"type":"string","maxLength":60}},"required":["task"]}'
    prompt="Summarize this development session in max 60 chars based on what the user asked for and worked on.

User's requests (first 5): $first_user_msgs

User's requests (last 5): $last_user_msgs

Examples: Configuring statusline, Adding AI summaries, Fixing build errors, Refactoring auth module"

    # Call Claude with structured output using temp file for timeout handling
    ai_temp="/tmp/claude-ai-summary-${session_id}-$$.json"

    (
        echo "$prompt" | claude --model haiku -p --no-session-persistence --output-format json --json-schema "$schema" 2>/dev/null > "$ai_temp"
    ) &
    claude_pid=$!

    # Wait up to 3 seconds for completion (statusline needs to be fast)
    for i in {1..3}; do
        if ! kill -0 $claude_pid 2>/dev/null; then
            break
        fi
        sleep 1
    done

    # Kill if still running (suppress all output)
    kill $claude_pid >/dev/null 2>&1
    wait $claude_pid >/dev/null 2>&1

    # Extract summary from temp file
    ai_summary=""
    if [ -f "$ai_temp" ] && [ -s "$ai_temp" ]; then
        ai_summary=$(jq -r '.structured_output.task // empty' "$ai_temp" 2>/dev/null)
        rm -f "$ai_temp"
    fi

    # Check if AI summary worked and is meaningful
    if [ -n "$ai_summary" ] && [ ${#ai_summary} -gt 10 ] && ! echo "$ai_summary" | grep -qiE "^(Ready|Awaiting|Clarify|Assist)"; then
        summary="$ai_summary"
    else
        # FALLBACK: Use first + last message
        first_line=$(echo "$first_user_msg" | head -n 1 | sed 's/^[[:space:]]*//;s/[[:space:]]*$//' | cut -c 1-47)
        last_user_msg=$(grep "\"sessionId\":\"${session_id}\"" "$history_file" | tail -1 | jq -r '.display // ""' | head -n 1 | sed 's/^[[:space:]]*//;s/[[:space:]]*$//' | cut -c 1-47)

        if [ "$user_msg_count" -le 1 ] || [ "$first_line" = "$last_user_msg" ]; then
            summary=$(echo "$first_line" | cut -c 1-100)
        else
            summary="${first_line} → ${last_user_msg}"
        fi
    fi

    # Clean up summary
    summary=$(echo "$summary" | tr -s ' ' | tr -d '\n\r' | sed 's/^[[:space:]]*//;s/[[:space:]]*$//')

    # Final fallback if still empty
    if [ -z "$summary" ]; then
        summary="Active session"
    fi

    # Cache the summary and message count
    echo "$summary" > "$summary_cache"
    date +%s > "$summary_timestamp"
    echo "$current_msg_count" > "$summary_msg_count"
else
    # Use cached summary
    summary=$(cat "$summary_cache" 2>/dev/null || echo "Active session")
fi

# Occasionally run shared cleanup (1% of the time to minimize overhead)
if [ $((RANDOM % 100)) -eq 0 ]; then
    bash ~/.claude/hooks/cache-cleanup.sh &
fi

# Combine workspace emoji, model name, starship output, and context percentage (line 1)
# Add session summary with session ID and assistant actions (line 2)
# Add session name with plan indicator (line 3)
printf "%s %s | %s%s\n%s%s | %s\n%s%s" "$workspace_emoji" "$model_name" "$starship_output" "$context_display" "$summary" "$assistant_actions" "$session_id" "$session_name" "$plan_indicator"
```

</details>

### Step 3: Make Scripts Executable

```bash
chmod +x ~/.claude/hooks/*.sh
chmod +x ~/.claude/statusline-starship-wrapper.sh
```

### Step 4: Configure Settings

Add to `~/.claude/settings.json`:

```json
{
  "statusLine": {
    "type": "command",
    "command": "bash ~/.claude/statusline-starship-wrapper.sh"
  },
  "hooks": {
    "UserPromptSubmit": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "bash ~/.claude/hooks/session-naming.sh",
            "timeout": 5
          },
          {
            "type": "command",
            "command": "bash ~/.claude/hooks/plan-tracking.sh",
            "timeout": 2
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "bash ~/.claude/hooks/assistant-output-sampling.sh",
            "timeout": 1
          }
        ]
      }
    ],
    "SessionStart": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "bash ~/.claude/hooks/plan-tracking.sh",
            "timeout": 2
          }
        ]
      }
    ]
  }
}
```

## Claude Code Internals

### JSON Input Format

Claude Code passes JSON to the statusline command via stdin:

```json
{
  "session_id": "abc-123-def",
  "session_name": "My Session",
  "workspace": {
    "current_dir": "/path/to/project"
  },
  "model": {
    "display_name": "Sonnet 4.5"
  },
  "context_window": {
    "current_usage": {
      "input_tokens": 50000,
      "output_tokens": 10000,
      "cache_creation_input_tokens": 5000,
      "cache_read_input_tokens": 30000
    },
    "context_window_size": 200000
  }
}
```

### Hook Input Format

Hooks receive different JSON based on event type:

**UserPromptSubmit:**
```json
{
  "session_id": "abc-123",
  "user_prompt": "The text the user typed",
  "hook_event_name": "UserPromptSubmit",
  ...
}
```

**PostToolUse:**
```json
{
  "session_id": "abc-123",
  "tool_name": "Edit",
  "tool_input": {"file_path": "/path", ...},
  "tool_result": {...},
  "hook_event_name": "PostToolUse",
  ...
}
```

### File Locations

- **History:** `~/.claude/history.jsonl` - User messages only
- **Transcripts:** `~/.claude/projects/-{project-path}/{session-id}.jsonl`
- **Plans:** `~/.claude/plans/{plan-name}.md`
- **Settings:** `~/.claude/settings.json`

## Starship Integration

### Escape Sequence Handling

Starship outputs bash readline escapes (`\[` and `\]`) that must be stripped:

```bash
starship_output=$(starship prompt 2>/dev/null | sed 's/\\\[//g; s/\\\]//g' | tr -d '\n')
```

**Why:** Statusline doesn't use readline, these escapes cause display issues.

### Environment Setup

Starship requires these environment variables:

```bash
export PWD="$cwd"
export STARSHIP_SHELL="bash"
cd "$cwd" 2>/dev/null || cd "$HOME"
```

**Why:** Starship uses PWD for directory display and git commands need correct working directory.

## Cache Management Strategy

### Design Principles

1. **Idle-Friendly:** Never regenerate if no user activity
2. **Threshold-Based:** Only cleanup when cache > 100MB
3. **Recency Protection:** Keep files < 7 days always
4. **Low Overhead:** 1% random execution, background processing

### Cache Files

| File Pattern | Purpose | Lifetime |
|-------------|---------|----------|
| `claude-session-name-*.txt` | Auto-generated session names | Until 100MB + 7 days |
| `claude-session-name-*.timestamp` | Name generation timestamps | Until 100MB + 7 days |
| `claude-session-name-*.msgcount` | Message count tracking | Until 100MB + 7 days |
| `claude-session-summary-*.txt` | AI-generated summaries | Until 100MB + 7 days |
| `claude-session-summary-*.timestamp` | Summary generation timestamps | Until 100MB + 7 days |
| `claude-session-summary-*.msgcount` | Summary message count tracking | Until 100MB + 7 days |
| `claude-session-plan-*.txt` | Active plan detection | Until 100MB + 7 days |
| `claude-assistant-summary-*.txt` | Tool usage summaries | Until 100MB + 7 days |
| `claude-assistant-actions-*.jsonl` | Full action logs | Until 100MB + 7 days |
| `claude-ai-summary-*.json` | Temp AI responses | 1 hour (always) |
| `claude-session-name-ai-*.json` | Temp session naming AI responses | 1 hour (always) |
| `claude-plan-session-map.jsonl` | Plan correlation history | Kept at 1000 entries |

### Regeneration Logic

**Session Name:**
- Immediate fallback write (prevents race condition)
- AI upgrade attempt with 5-second timeout
- Regenerates when message count increases
- Readable word-boundary truncation fallbacks

**AI Summary:**
- Regenerate ONLY when new user messages exist
- Idle sessions keep last summary indefinitely

**Plan Detection:**
- Check on SessionStart and new user messages
- Detects plans created within 10 minutes

**Assistant Actions:**
- Update after every tool execution
- Shows top 3 tools from last 10 actions

## Troubleshooting

### Statusline Not Showing

1. Check script permissions: `chmod +x ~/.claude/statusline-starship-wrapper.sh`
2. Test manually: `echo '{"session_id":"test",...}' | bash ~/.claude/statusline-starship-wrapper.sh`
3. Check Starship installed: `which starship`
4. Verify jq installed: `which jq`

### AI Summaries Not Generating

1. Check Claude CLI: `which claude`
2. Test AI call: `echo "test" | claude --model haiku -p --no-session-persistence`
3. Check timeout (3 seconds might be tight for first call)
4. Verify cache files exist: `ls -lh /tmp/claude-session-summary-*`

### Hooks Not Running

1. Hooks load at session start - restart Claude Code
2. Check hook scripts exist and are executable
3. Test hook directly: `echo '{"session_id":"test","user_prompt":"hello"}' | bash ~/.claude/hooks/session-naming.sh`
4. Check settings.json syntax

### Cache Growing Too Large

1. Check total cache: `du -sm /tmp/claude-* 2>/dev/null | awk '{sum+=$1} END {print sum " MB"}'`
2. Manually trigger cleanup: `bash ~/.claude/hooks/cache-cleanup.sh`
3. Reduce retention (edit cache-cleanup.sh, change `-mtime +7` to `-mtime +3`)

### Escape Sequences Visible

If you see `\[` or `\]` in statusline:
1. Check sed command in statusline script
2. Verify pattern: `'s/\\\[//g; s/\\\]//g'` (three backslashes)
3. Test: `echo "\\[test\\]" | sed 's/\\\[//g; s/\\\]//g'`

## Customization

### Add More Workspace Emojis

Edit `get_workspace_emoji()` function in statusline script:

```bash
case "$basename" in
    *myproject*) echo "🎯" ;;
    *backend*) echo "⚙️" ;;
    # Add more...
esac
```

### Change AI Summary Length

Modify schema in statusline script:

```bash
schema='{"type":"object","properties":{"task":{"type":"string","maxLength":80}},"required":["task"]}'
```

### Adjust Cache Threshold

Edit `cache-cleanup.sh`:

```bash
# Change 100MB threshold to 50MB
if [ "$cache_size" -lt 50 ]; then
```

### Change Cleanup Age

Edit `cache-cleanup.sh`:

```bash
# Change 7 days to 3 days
find /tmp -name "claude-session-summary-*.txt" -mtime +3 -delete
```

## Performance Characteristics

- **Statusline refresh:** ~50-100ms (cached), ~3-5s (AI regeneration)
- **Hook execution:** 1-5 seconds per hook
- **Cache cleanup:** Background, non-blocking
- **Memory footprint:** ~1-2MB per session (caches)
- **Disk usage:** 60-100MB typical cache size

## Security Considerations

- All cache files in `/tmp` (world-readable on most systems)
- Session IDs visible in file names
- No sensitive data should be in summaries
- AI calls use `--no-session-persistence` (no data retention)
- Scripts use `set -euo pipefail` for safety

## Future Enhancements

- [ ] Support for remote sessions (different transcript paths)
- [ ] Integration with episodic memory for cross-session context
- [ ] Configurable emoji sets via settings
- [ ] Multi-line summary support for complex sessions
- [ ] Git branch-based session grouping
- [ ] Export session summaries to markdown reports
