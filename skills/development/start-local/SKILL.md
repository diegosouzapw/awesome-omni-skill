---
name: start-local
description: Start local development environment with auto-detected services in a persistent tmux session
user-invocable: true
---

# /start-local - Start Local Development Environment in tmux

Start local development environment with auto-detected services in a persistent tmux session.

## Usage

```bash
/start-local                 # Uses .env or .env.development
/start-local staging         # Uses .env.staging
/start-local production      # Uses .env.production
```

## Process

### Step 1: Determine Environment

```bash
ENVIRONMENT=${1:-development}
ENV_FILE=".env.${ENVIRONMENT}"

# Fallback to .env if specific file doesn't exist
if [ ! -f "$ENV_FILE" ] && [ "$ENVIRONMENT" = "development" ]; then
    ENV_FILE=".env"
fi

if [ ! -f "$ENV_FILE" ]; then
    echo "❌ Environment file not found: $ENV_FILE"
    ls -1 .env* 2>/dev/null
    exit 1
fi
```

### Step 2: Detect Project Type

```bash
detect_project_type() {
    if [ -f "package.json" ]; then
        grep -q "\"next\":" package.json && echo "nextjs" && return
        grep -q "\"vite\":" package.json && echo "vite" && return
        grep -q "\"react-scripts\":" package.json && echo "cra" && return
        grep -q "\"@vue/cli\":" package.json && echo "vue" && return
        echo "node"
    elif [ -f "requirements.txt" ] || [ -f "pyproject.toml" ]; then
        grep -q "django" requirements.txt pyproject.toml 2>/dev/null && echo "django" && return
        grep -q "flask" requirements.txt pyproject.toml 2>/dev/null && echo "flask" && return
        echo "python"
    elif [ -f "Cargo.toml" ]; then
        echo "rust"
    elif [ -f "go.mod" ]; then
        echo "go"
    else
        echo "unknown"
    fi
}

PROJECT_TYPE=$(detect_project_type)
```

### Step 3: Detect Required Services

```bash
NEEDS_SUPABASE=false
NEEDS_POSTGRES=false
NEEDS_REDIS=false

[ -f "supabase/config.toml" ] && NEEDS_SUPABASE=true
grep -q "postgres" "$ENV_FILE" 2>/dev/null && NEEDS_POSTGRES=true
grep -q "redis" "$ENV_FILE" 2>/dev/null && NEEDS_REDIS=true
```

### Step 4: Generate Random Port

```bash
DEV_PORT=$(shuf -i 3000-9999 -n 1)

while lsof -i :$DEV_PORT >/dev/null 2>&1; do
    DEV_PORT=$(shuf -i 3000-9999 -n 1)
done
```

### Step 5: Create tmux Session

```bash
PROJECT_NAME=$(basename "$(pwd)")
BRANCH=$(git branch --show-current 2>/dev/null || echo "main")
TIMESTAMP=$(date +%s)
SESSION="dev-${PROJECT_NAME}-${TIMESTAMP}"

tmux new-session -d -s "$SESSION" -n servers
```

### Step 6: Start Services

```bash
PANE_COUNT=0

# Main dev server
case $PROJECT_TYPE in
    nextjs|vite|cra|vue)
        tmux send-keys -t "$SESSION:servers.${PANE_COUNT}" "PORT=$DEV_PORT npm run dev 2>&1 | tee dev-server-${DEV_PORT}.log" C-m
        ;;
    django)
        tmux send-keys -t "$SESSION:servers.${PANE_COUNT}" "python manage.py runserver $DEV_PORT 2>&1 | tee dev-server-${DEV_PORT}.log" C-m
        ;;
    flask)
        tmux send-keys -t "$SESSION:servers.${PANE_COUNT}" "FLASK_RUN_PORT=$DEV_PORT flask run 2>&1 | tee dev-server-${DEV_PORT}.log" C-m
        ;;
    *)
        tmux send-keys -t "$SESSION:servers.${PANE_COUNT}" "PORT=$DEV_PORT npm run dev 2>&1 | tee dev-server-${DEV_PORT}.log" C-m
        ;;
esac

# Additional services (if needed)
if [ "$NEEDS_SUPABASE" = true ]; then
    PANE_COUNT=$((PANE_COUNT + 1))
    tmux split-window -v -t "$SESSION:servers"
    tmux send-keys -t "$SESSION:servers.${PANE_COUNT}" "supabase start" C-m
fi

if [ "$NEEDS_POSTGRES" = true ] && [ "$NEEDS_SUPABASE" = false ]; then
    PANE_COUNT=$((PANE_COUNT + 1))
    tmux split-window -v -t "$SESSION:servers"
    tmux send-keys -t "$SESSION:servers.${PANE_COUNT}" "docker-compose up postgres" C-m
fi

if [ "$NEEDS_REDIS" = true ]; then
    PANE_COUNT=$((PANE_COUNT + 1))
    tmux split-window -v -t "$SESSION:servers"
    tmux send-keys -t "$SESSION:servers.${PANE_COUNT}" "redis-server" C-m
fi

tmux select-layout -t "$SESSION:servers" tiled
```

### Step 7: Create Additional Windows

```bash
# Logs window
tmux new-window -t "$SESSION" -n logs
tmux send-keys -t "$SESSION:logs" "tail -f dev-server-${DEV_PORT}.log 2>/dev/null || sleep infinity" C-m

# Work window
tmux new-window -t "$SESSION" -n work

# Git window
tmux new-window -t "$SESSION" -n git
tmux send-keys -t "$SESSION:git" "git status" C-m
```

### Step 8: Save Metadata

```bash
cat > .tmux-dev-session.json <<EOF
{
  "session": "$SESSION",
  "project_name": "$PROJECT_NAME",
  "branch": "$BRANCH",
  "type": "$PROJECT_TYPE",
  "environment": "$ENVIRONMENT",
  "env_file": "$ENV_FILE",
  "dev_port": $DEV_PORT,
  "created": "$(date -Iseconds)"
}
EOF
```

### Step 9: Display Summary

```bash
echo ""
echo "✨ Dev Environment Started: $SESSION"
echo ""
echo "Environment: $ENVIRONMENT ($ENV_FILE)"
echo "Dev Server: http://localhost:$DEV_PORT"
[ "$NEEDS_SUPABASE" = true ] && echo "Supabase: http://localhost:54321"
echo ""
echo "Attach: tmux attach -t $SESSION"
echo "Detach: Ctrl+a d"
echo "Status: /tmux-status"
echo ""
```

## Notes

- Auto-detects framework/stack from project files
- Auto-detects services from .env and config files
- Random ports prevent conflicts
- Session persists across disconnects
- Metadata saved to `.tmux-dev-session.json`
