---
name: useuser
description: Ask humans questions via native macOS dialogs and notifications. Use when you need clarification, confirmation, or direct input from the user through system-level dialogs rather than chat. Supports text input, multiple choice, yes/no confirmation, information display, and system notifications.
license: MIT
compatibility: macOS only (requires AppleScript via osascript)
metadata:
  author: willwillems
  version: "1.0.0"
---

# Useuser

This skill enables you to ask humans questions through native macOS system dialogs. Use this when you need direct user input, confirmation before taking actions, or want to display important information that requires acknowledgment.

## When to Use This Skill

- **Clarification needed**: When you're uncertain about requirements or need more context
- **Confirmation required**: Before performing destructive or irreversible actions
- **User preferences**: When you need to know which option the user prefers
- **Important notifications**: When you need to alert the user about something significant
- **Blocking questions**: When you cannot proceed without user input

## Available Dialog Types

### 1. Text Input Dialog

Ask open-ended questions where the user types a response.

```bash
scripts/ask_text.sh "What is the database connection string?" "postgresql://localhost:5432/mydb" 300
```

**Parameters:**
- `$1` - Question text (required)
- `$2` - Default value (optional, defaults to empty)
- `$3` - Timeout in seconds (optional, defaults to 300). Use `0` or `none` for no timeout.

**Returns:** The text entered by the user, or "CANCELLED" if cancelled, or "TIMEOUT" if timed out.

**No timeout example:**
```bash
scripts/ask_text.sh "What is your preferred username?" "" 0
```

### 2. Multiple Choice Dialog

Present a list of options for the user to choose from.

```bash
scripts/ask_choice.sh "Which authentication method should I use?" "OAuth2" "JWT" "Session-based" "API Keys"
```

**Parameters:**
- `$1` - Question/prompt text (required)
- `$2+` - Available choices (at least 2 required)

**Returns:** The selected choice text, or "CANCELLED" if cancelled.

### 3. Yes/No Confirmation Dialog

Get a simple yes or no answer.

```bash
scripts/ask_confirm.sh "Should I delete the old migration files?" "no" 300
```

**Parameters:**
- `$1` - Question text (required)
- `$2` - Default button: "yes" or "no" (optional, defaults to "no")
- `$3` - Timeout in seconds (optional, defaults to 300). Use `0` or `none` for no timeout.

**Returns:** "YES" or "NO", or "CANCELLED" if cancelled, or "TIMEOUT" if timed out.

### 4. Information Dialog

Display information that requires user acknowledgment.

```bash
scripts/show_info.sh "Migration completed successfully! 47 records updated." 300
```

**Parameters:**
- `$1` - Message to display (required)
- `$2` - Timeout in seconds (optional, defaults to 300). Use `0` or `none` for no timeout.

**Returns:** "OK" when acknowledged, or "CANCELLED" if cancelled, or "TIMEOUT" if timed out.

### 5. System Notification

Send a macOS notification (non-blocking).

```bash
scripts/notify.sh "Build completed successfully" "Development Server" "Ready for testing" "yes"
```

**Parameters:**
- `$1` - Notification message (required)
- `$2` - Title (optional, defaults to "Useuser")
- `$3` - Subtitle (optional)
- `$4` - Play sound: "yes" or "no" (optional, defaults to "no")

**Returns:** "OK" on success.

## Usage Examples

### Example 1: Getting User Preferences

When setting up a new project:

```bash
# Ask what framework to use
framework=$(scripts/ask_choice.sh "Which framework should I use for this project?" "React" "Vue" "Svelte" "Angular")

if [ "$framework" != "CANCELLED" ]; then
    echo "User selected: $framework"
fi
```

### Example 2: Confirming Destructive Actions

Before deleting files:

```bash
confirm=$(scripts/ask_confirm.sh "This will permanently delete 15 files. Are you sure?" "no")

if [ "$confirm" = "YES" ]; then
    # Proceed with deletion
    echo "Deleting files..."
else
    echo "Operation cancelled by user"
fi
```

### Example 3: Getting Configuration Values

When you need specific input:

```bash
api_key=$(scripts/ask_text.sh "Please enter your API key:" "" 120)

if [ "$api_key" != "CANCELLED" ] && [ "$api_key" != "TIMEOUT" ]; then
    echo "API key received"
fi
```

### Example 4: Notifying About Long Operations

When a background task completes:

```bash
scripts/notify.sh "Database backup completed" "Backup Tool" "All 3 databases backed up successfully" "yes"
```

## Error Handling

All scripts handle errors gracefully:

- **User cancellation**: Returns "CANCELLED"
- **Timeout**: Returns "TIMEOUT" (for dialogs that support it)
- **AppleScript errors**: Returns "ERROR: <message>"

Always check the return value before proceeding:

```bash
result=$(scripts/ask_confirm.sh "Continue with deployment?")

case "$result" in
    "YES")
        echo "Deploying..."
        ;;
    "NO")
        echo "Deployment cancelled by user"
        ;;
    "CANCELLED")
        echo "Dialog was dismissed"
        ;;
    "TIMEOUT")
        echo "No response received, timing out"
        ;;
    ERROR*)
        echo "An error occurred: $result"
        ;;
esac
```

## Best Practices

1. **Be specific**: Ask clear, specific questions that have actionable answers
2. **Provide context**: Include relevant context in your questions
3. **Use appropriate dialog type**: Use confirmation for yes/no, choice for options, text for open-ended
4. **Set reasonable timeouts**: Don't make users wait forever, but give them enough time
5. **Handle all cases**: Always handle cancellation and timeout scenarios
6. **Minimize interruptions**: Only ask when you truly need human input
7. **Use notifications sparingly**: Reserve notifications for important, non-blocking updates

## Platform Requirements

- **macOS only**: This skill uses AppleScript via `osascript` and only works on macOS
- **Terminal permissions**: The terminal app may need permission to display dialogs (System Preferences > Privacy & Security > Accessibility)
