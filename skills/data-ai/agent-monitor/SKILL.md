---
name: agent-monitor
description: Detailed monitoring of Pi agents in WezTerm. Shows full task, recent activity, and last tool output for each agent.
---

# Agent Monitor

Get detailed status of each Pi agent.

## Run Detailed Status Check

```bash
echo "🧠 DETAILED AGENT STATUS - $(date '+%H:%M:%S')"
echo ""

PANES=$(bb wezterm list 2>/dev/null)
if [ $? -ne 0 ] || [ -z "$PANES" ]; then
  echo "⚠️ WezTerm not accessible"
  exit 0
fi

echo "$PANES" | jq -r '.data[] | "\(.pane_id)|\(.title)|\(.cwd)"' | while IFS='|' read -r PANE_ID TITLE CWD; do
  CWD_CLEAN=$(echo "$CWD" | sed 's|file://||')
  PROJECT=$(echo "$CWD_CLEAN" | xargs basename 2>/dev/null || echo "unknown")
  
  [[ "$TITLE" != *"π"* ]] && continue
  
  SESSION_PATH=$(echo "$CWD_CLEAN" | sed 's|/|-|g' | sed 's|^-||')
  SESSION=$(ls -t "$HOME/.pi/agent/sessions/--${SESSION_PATH}--"/*.jsonl 2>/dev/null | head -1)
  [ -z "$SESSION" ] && continue
  
  STOP=$(tail -1 "$SESSION" | jq -r '.message.stopReason // "null"')
  ROLE=$(tail -1 "$SESSION" | jq -r '.message.role // "unknown"')
  [ "$STOP" = "stop" ] && [ "$ROLE" = "assistant" ] && STATUS="✅ IDLE" || STATUS="🔄 WORKING"
  
  echo "═══════════════════════════════════════════════════════════════"
  echo "## Pane $PANE_ID - $PROJECT - $STATUS"
  echo ""
  
  echo "**Last user request:**"
  tail -100 "$SESSION" | jq -r 'select(.type=="message" and .message.role=="user") | .message.content | if type=="array" then .[0].text else . end' 2>/dev/null | tail -1
  echo ""
  
  echo "**Recent activity (last 5 messages):**"
  tail -50 "$SESSION" | jq -r 'select(.type=="message") | "\(.message.role): \(.message.content | if type=="array" then ([.[] | if .type=="text" then .text[:150] elif .type=="toolCall" then "[\(.name)]" else .type end] | join(" ")) else .[:150] end)"' 2>/dev/null | tail -5
  echo ""
  
  echo "**Last tool output (truncated):**"
  tail -20 "$SESSION" | jq -r 'select(.type=="message" and .message.role=="toolResult") | .message.content | if type=="array" then .[0].text else . end' 2>/dev/null | tail -1 | head -c 300
  echo ""
  echo ""
done

echo "═══════════════════════════════════════════════════════════════"
```

## Output Explained

For each agent shows:
- **Pane + Project + Status** (idle/working)
- **Last user request** - the full task they were given
- **Recent activity** - last 5 messages (role + truncated content/tool calls)
- **Last tool output** - what the last command returned

## After Review

Analyze each agent and decide:
1. **Is it stuck?** Same output for too long → send "status?"
2. **Is it done?** ✅ IDLE → assign new task
3. **Is it duplicated?** Multiple panes same work → stop extras
4. **Does it need help?** Errors in output → intervene

## Actions

```bash
# Send task
bb wezterm send <pane_id> "your task"

# Check full session
tail -100 <session_file> | jq '.message'

# Interrupt
bb wezterm send <pane_id> "stop, summarize progress"
```
