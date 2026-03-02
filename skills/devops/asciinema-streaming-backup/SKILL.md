---
name: asciinema-streaming-backup
description: Real-time asciinema backup to GitHub orphan branch. TRIGGERS - streaming backup, asciinema backup, session backup, recording backup.
allowed-tools: Read, Bash, Glob, Write, Edit, AskUserQuestion
---

# asciinema-streaming-backup

Complete system for streaming asciinema recordings to GitHub with automatic brotli archival. Uses idle-detection for intelligent chunking, zstd for concatenatable streaming compression, and GitHub Actions for final brotli recompression.

> **Platform**: macOS, Linux
> **Isolation**: Uses Git orphan branch (separate history, cannot pollute main)

---

## Architecture Overview

```
┌─────────────────┐     zstd chunks      ┌─────────────────┐     Actions      ┌─────────────────┐
│  asciinema rec  │ ──────────────────▶  │  GitHub Orphan  │ ───────────────▶ │  brotli archive │
│  + idle-chunker │   (concatenatable)   │  gh-recordings  │                  │  (300x compress)│
└─────────────────┘                      └─────────────────┘                  └─────────────────┘
         │                                        │
         │ Idle ≥30s triggers chunk               │ Separate history
         ▼                                        │ Cannot PR to main
    ~/asciinema_recordings/                                 ▼
    └── repo-name/                          .github/workflows/
        └── chunks/*.zst                    └── recompress.yml
```

---

## Requirements

| Component         | Required | Installation             | Version       |
| ----------------- | -------- | ------------------------ | ------------- |
| **asciinema CLI** | Yes      | `brew install asciinema` | 3.0+ (Rust)   |
| **zstd**          | Yes      | `brew install zstd`      | Any           |
| **brotli**        | Yes      | `brew install brotli`    | Any           |
| **git**           | Yes      | Pre-installed            | 2.20+         |
| **gh CLI**        | Yes      | `brew install gh`        | Any           |
| **fswatch**       | Optional | `brew install fswatch`   | For real-time |

---

## Workflow Phases

### Phase 0: Preflight Validation

**Purpose**: Verify all tools installed, offer self-correction if missing.

```bash
/usr/bin/env bash << 'PREFLIGHT_EOF'
# preflight-check.sh - Validates all requirements

MISSING=()

# Check each tool
for tool in asciinema zstd brotli git gh; do
  if ! command -v "$tool" &>/dev/null; then
    MISSING+=("$tool")
  fi
done

if [[ ${#MISSING[@]} -gt 0 ]]; then
  echo "Missing tools: ${MISSING[*]}"
  echo ""
  echo "Install with:"
  echo "  brew install ${MISSING[*]}"
  exit 1
fi

# Check asciinema version (need 3.0+ for Rust version)
ASCIINEMA_VERSION=$(asciinema --version 2>&1 | grep -oE '[0-9]+\.[0-9]+' | head -1)
if [[ "${ASCIINEMA_VERSION%%.*}" -lt 3 ]]; then
  echo "Warning: asciinema $ASCIINEMA_VERSION detected. Version 3.0+ recommended."
  echo "Upgrade: brew upgrade asciinema"
fi

echo "All requirements satisfied"
PREFLIGHT_EOF
```

**AskUserQuestion** (if tools missing):

```yaml
AskUserQuestion:
  question: "Required tools are missing. How would you like to proceed?"
  header: "Preflight Check"
  options:
    - label: "Install all missing tools (Recommended)"
      description: "Run: brew install ${MISSING[*]}"
    - label: "Show manual installation commands"
      description: "Display commands without executing"
    - label: "Continue anyway (may fail later)"
      description: "Skip installation and proceed"
```

**Self-Correction**: If tools are missing, generate installation command and offer to run it.

---

### Phase 1: GitHub Account Detection

**Purpose**: Detect available GitHub accounts and let user choose which to use for recording storage.

#### Detection Sources

Probe these 5 sources to detect GitHub accounts:

| Source     | Command                                | What it finds                                     |
| ---------- | -------------------------------------- | ------------------------------------------------- |
| SSH config | `grep -A5 "Host github" ~/.ssh/config` | Match directives with IdentityFile                |
| SSH keys   | `ls ~/.ssh/id_ed25519_*`               | Account-named keys (e.g., `id_ed25519_terrylica`) |
| gh CLI     | `gh auth status`                       | Authenticated accounts                            |
| mise env   | `grep GH_ACCOUNT .mise.toml`           | GH_ACCOUNT variable                               |
| git config | `git config user.name`                 | Global git username                               |

#### Detection Script

```bash
/usr/bin/env bash << 'DETECT_ACCOUNTS_EOF'
# detect-github-accounts.sh - Probe all sources for GitHub accounts
# Uses portable parallel arrays (works in bash 3.2+ and when wrapped for zsh)

ACCOUNT_NAMES=()
ACCOUNT_SOURCES=()

log() { echo "[detect] $*"; }

# Helper: add account with source (updates existing or appends new)
add_account() {
  local account="$1" source="$2"
  local idx
  for idx in "${!ACCOUNT_NAMES[@]}"; do
    if [[ "${ACCOUNT_NAMES[$idx]}" == "$account" ]]; then
      ACCOUNT_SOURCES[$idx]+="$source "
      return
    fi
  done
  ACCOUNT_NAMES+=("$account")
  ACCOUNT_SOURCES+=("$source ")
}

# 1. SSH config Match directives
if [[ -f ~/.ssh/config ]]; then
  while IFS= read -r line; do
    if [[ "$line" =~ IdentityFile.*id_ed25519_([a-zA-Z0-9_-]+) ]]; then
      add_account "${BASH_REMATCH[1]}" "ssh-config"
    fi
  done < ~/.ssh/config
fi

# 2. SSH key filenames
for keyfile in ~/.ssh/id_ed25519_*; do
  if [[ -f "$keyfile" && "$keyfile" != *.pub ]]; then
    account=$(basename "$keyfile" | sed 's/id_ed25519_//')
    add_account "$account" "ssh-key"
  fi
done

# 3. gh CLI authenticated accounts
if command -v gh &>/dev/null; then
  while IFS= read -r account; do
    [[ -n "$account" ]] && add_account "$account" "gh-cli"
  done < <(gh auth status 2>&1 | grep -oE 'Logged in to github.com account [a-zA-Z0-9_-]+' | awk '{print $NF}')
fi

# 4. mise env GH_ACCOUNT
if [[ -f .mise.toml ]]; then
  account=$(grep -E 'GH_ACCOUNT\s*=' .mise.toml 2>/dev/null | sed 's/.*=\s*"\([^"]*\)".*/\1/')
  [[ -n "$account" ]] && add_account "$account" "mise-env"
fi

# 5. git config user.name
git_user=$(git config user.name 2>/dev/null)
[[ -n "$git_user" ]] && add_account "$git_user" "git-config"

# Score and display
log "=== Detected GitHub Accounts ==="
RECOMMENDED=""
MAX_SOURCES=0
for idx in "${!ACCOUNT_NAMES[@]}"; do
  account="${ACCOUNT_NAMES[$idx]}"
  sources="${ACCOUNT_SOURCES[$idx]}"
  count=$(echo "$sources" | wc -w | tr -d ' ')
  log "$account: $count sources ($sources)"
  if (( count > MAX_SOURCES )); then
    MAX_SOURCES=$count
    RECOMMENDED="$account"
    RECOMMENDED_SOURCES="$sources"
  fi
done

echo ""
echo "RECOMMENDED=$RECOMMENDED"
echo "SOURCES=$RECOMMENDED_SOURCES"
DETECT_ACCOUNTS_EOF
```

#### AskUserQuestion

```yaml
AskUserQuestion:
  question: "Which GitHub account should be used for recording storage?"
  header: "GitHub Account Selection"
  options:
    - label: "${RECOMMENDED} (Recommended)"
      description: "Detected via: ${SOURCES}"
    # Additional detected accounts appear here dynamically
    - label: "Enter manually"
      description: "Type a GitHub username not listed above"
```

**Post-Selection**: If user selects an account, ensure gh CLI is using that account:

```bash
/usr/bin/env bash << 'POST_SELECT_EOF'
# Ensure gh CLI is authenticated as selected account
SELECTED_ACCOUNT="${1:?Usage: provide selected account}"

if ! gh auth status 2>&1 | grep -q "Logged in to github.com account $SELECTED_ACCOUNT"; then
  echo "Switching gh CLI to account: $SELECTED_ACCOUNT"
  gh auth switch --user "$SELECTED_ACCOUNT" 2>/dev/null || \
    echo "Warning: Could not switch accounts. Manual auth may be needed."
fi
POST_SELECT_EOF
```

---

### Phase 1.5: Current Repository Detection

**Purpose**: Detect current git repository context to provide intelligent defaults for Phase 2 questions.

#### Detection Script

```bash
/usr/bin/env bash << 'DETECT_REPO_EOF'
# Detect current repository context for intelligent defaults

CURRENT_REPO_URL=""
CURRENT_REPO_OWNER=""
CURRENT_REPO_NAME=""
DETECTED_FROM=""

# Check if we're in a git repository
if git rev-parse --git-dir &>/dev/null; then
  # Try origin remote first
  if git remote get-url origin &>/dev/null; then
    CURRENT_REPO_URL=$(git remote get-url origin)
    DETECTED_FROM="origin remote"
  # Fallback to first available remote
  elif [[ -n "$(git remote)" ]]; then
    REMOTE=$(git remote | head -1)
    CURRENT_REPO_URL=$(git remote get-url "$REMOTE")
    DETECTED_FROM="$REMOTE remote"
  fi

  # Parse owner and name from URL (SSH or HTTPS)
  if [[ -n "$CURRENT_REPO_URL" ]]; then
    if [[ "$CURRENT_REPO_URL" =~ github\.com[:/]([^/]+)/([^/.]+) ]]; then
      CURRENT_REPO_OWNER="${BASH_REMATCH[1]}"
      CURRENT_REPO_NAME="${BASH_REMATCH[2]%.git}"
    fi
  fi
fi

# Output for Claude to parse
echo "CURRENT_REPO_URL=$CURRENT_REPO_URL"
echo "CURRENT_REPO_OWNER=$CURRENT_REPO_OWNER"
echo "CURRENT_REPO_NAME=$CURRENT_REPO_NAME"
echo "DETECTED_FROM=$DETECTED_FROM"
DETECT_REPO_EOF
```

**Claude Action**: Store detected values (`CURRENT_REPO_OWNER`, `CURRENT_REPO_NAME`, `DETECTED_FROM`) for use in subsequent AskUserQuestion calls. If no repo detected, proceed without defaults.

---

### Phase 2: Core Configuration

**Purpose**: Gather essential configuration from user.

#### 2.1 Repository URL

**If current repo detected** (from Phase 1.5):

```yaml
AskUserQuestion:
  question: "Which repository should store the recordings?"
  header: "Repository"
  options:
    - label: "${CURRENT_REPO_OWNER}/${CURRENT_REPO_NAME} (Recommended)"
      description: "Current repo detected from ${DETECTED_FROM}"
    - label: "Create dedicated repo: ${GITHUB_ACCOUNT}/asciinema-recordings"
      description: "Separate repository for all recordings"
    - label: "Enter different repository"
      description: "Specify another repository (user/repo format)"
```

**If no current repo detected**:

```yaml
AskUserQuestion:
  question: "Enter the GitHub repository URL for storing recordings:"
  header: "Repository URL"
  options:
    - label: "Create dedicated repo: ${GITHUB_ACCOUNT}/asciinema-recordings"
      description: "Separate repository for all recordings (Recommended)"
    - label: "Enter repository manually"
      description: "SSH (git@github.com:user/repo.git), HTTPS, or shorthand (user/repo)"
```

**URL Normalization** (handles multiple formats):

```bash
/usr/bin/env bash << 'NORMALIZE_URL_EOF'
# Normalize to SSH format for consistent handling
normalize_repo_url() {
  local url="$1"

  # Shorthand: user/repo -> git@github.com:user/repo.git
  if [[ "$url" =~ ^[a-zA-Z0-9_-]+/[a-zA-Z0-9_.-]+$ ]]; then
    echo "git@github.com:${url}.git"
  # HTTPS: https://github.com/user/repo -> git@github.com:user/repo.git
  elif [[ "$url" =~ ^https://github\.com/([^/]+)/([^/]+)/?$ ]]; then
    echo "git@github.com:${BASH_REMATCH[1]}/${BASH_REMATCH[2]%.git}.git"
  # Already SSH format
  else
    echo "$url"
  fi
}

URL="${1:?Usage: provide URL to normalize}"
normalize_repo_url "$URL"
NORMALIZE_URL_EOF
```

**Confirmation for free-form input** (if user selected "Enter different/manually"):

```yaml
AskUserQuestion:
  question: "You entered '${USER_INPUT}'. Normalized to: ${NORMALIZED_URL}. Is this correct?"
  header: "Confirm Repository"
  options:
    - label: "Yes, use ${NORMALIZED_URL}"
      description: "Proceed with this repository"
    - label: "No, let me re-enter"
      description: "Go back to repository selection"
```

#### 2.2 Recording Directory

```yaml
AskUserQuestion:
  question: "Where should recordings be stored locally?"
  header: "Recording Directory"
  options:
    - label: "~/asciinema_recordings/${RESOLVED_REPO_NAME} (Recommended)"
      description: "Example: ~/asciinema_recordings/alpha-forge"
    - label: "Custom path"
      description: "Enter a different directory path"
```

**Note**: `${RESOLVED_REPO_NAME}` is the actual repo name from Phase 1.5 or Phase 2.1, not a variable placeholder. Display the concrete path to user.

#### 2.3 Branch Name

```yaml
AskUserQuestion:
  question: "What should the orphan branch be named?"
  header: "Branch Name"
  options:
    - label: "asciinema-recordings (Recommended)"
      description: "Matches ~/asciinema_recordings/ parent directory pattern"
    - label: "gh-recordings"
      description: "GitHub-prefixed alternative (gh = GitHub storage)"
    - label: "recordings"
      description: "Minimal name"
    - label: "Custom"
      description: "Enter a custom branch name"
```

**Naming Convention**: The default `asciinema-recordings` matches the parent directory `~/asciinema_recordings/` for consistency.

---

### Phase 3: Advanced Configuration

**Purpose**: Allow customization of compression and behavior parameters.

#### Configuration Parameters

| Parameter      | Default | Options                                     |
| -------------- | ------- | ------------------------------------------- |
| Idle threshold | 30s     | 15s, 30s (Recommended), 60s, Custom (5-300) |
| zstd level     | 3       | 1 (fast), 3 (Recommended), 6, Custom (1-22) |
| Brotli level   | 9       | 6, 9 (Recommended), 11, Custom (1-11)       |
| Auto-push      | Yes     | Yes (Recommended), No                       |
| Poll interval  | 5s      | 2s, 5s (Recommended), 10s                   |

#### AskUserQuestion Sequence

**3.1 Idle Threshold**:

```yaml
AskUserQuestion:
  question: "How long should the chunker wait before creating a chunk?"
  header: "Idle Threshold"
  options:
    - label: "15 seconds"
      description: "More frequent chunks, smaller files"
    - label: "30 seconds (Recommended)"
      description: "Balanced chunk size and frequency"
    - label: "60 seconds"
      description: "Larger chunks, less frequent uploads"
    - label: "Custom (5-300 seconds)"
      description: "Enter a custom threshold"
```

**3.2 zstd Compression Level**:

```yaml
AskUserQuestion:
  question: "What zstd compression level for streaming chunks?"
  header: "zstd Level"
  options:
    - label: "1 (Fast)"
      description: "Fastest compression, larger files"
    - label: "3 (Recommended)"
      description: "Good balance of speed and compression"
    - label: "6 (Better compression)"
      description: "Slower but smaller chunks"
    - label: "Custom (1-22)"
      description: "Enter a custom level"
```

**3.3 Brotli Compression Level**:

```yaml
AskUserQuestion:
  question: "What brotli compression level for final archives?"
  header: "Brotli Level"
  options:
    - label: "6"
      description: "Faster archival, slightly larger files"
    - label: "9 (Recommended)"
      description: "Great compression with reasonable speed"
    - label: "11 (Maximum)"
      description: "Best compression, slowest (may timeout on large files)"
    - label: "Custom (1-11)"
      description: "Enter a custom level"
```

**3.4 Auto-Push**:

```yaml
AskUserQuestion:
  question: "Should chunks be automatically pushed to GitHub?"
  header: "Auto-Push"
  options:
    - label: "Yes (Recommended)"
      description: "Push immediately after each chunk"
    - label: "No"
      description: "Manual push when ready"
```

**3.5 Poll Interval**:

```yaml
AskUserQuestion:
  question: "How often should the chunker check for idle state?"
  header: "Poll Interval"
  options:
    - label: "2 seconds"
      description: "More responsive, slightly higher CPU"
    - label: "5 seconds (Recommended)"
      description: "Good balance"
    - label: "10 seconds"
      description: "Lower resource usage"
```

---

### Phase 4: Orphan Branch Setup

**Purpose**: Create or configure the orphan branch with GitHub Actions workflow.

#### Check for Existing Branch

```bash
/usr/bin/env bash << 'CHECK_BRANCH_EOF'
# Check if branch exists on remote
REPO_URL="${1:?Usage: provide repo URL}"
BRANCH="${2:-asciinema-recordings}"  # From Phase 2 (default changed)

if git ls-remote --heads "$REPO_URL" "$BRANCH" 2>/dev/null | grep -q "$BRANCH"; then
  echo "Branch '$BRANCH' already exists on remote"
  echo "BRANCH_EXISTS=true"
else
  echo "Branch '$BRANCH' does not exist"
  echo "BRANCH_EXISTS=false"
fi
CHECK_BRANCH_EOF
```

#### AskUserQuestion (if branch exists)

```yaml
AskUserQuestion:
  question: "Branch '${BRANCH}' already exists on remote. How should we proceed?"
  header: "Existing Branch"
  options:
    - label: "Clone locally (Recommended)"
      description: "Use existing branch, clone to local directory"
    - label: "Reset and recreate fresh"
      description: "Delete remote branch and start over (DESTRUCTIVE)"
    - label: "Keep existing and verify"
      description: "Check existing setup matches configuration"
    - label: "Show manual instructions"
      description: "Display commands without executing"
```

#### Branch Creation (if new)

```bash
/usr/bin/env bash << 'SETUP_ORPHAN_EOF'
# setup-orphan-branch.sh - Creates asciinema-recordings orphan branch

REPO_URL="${1:?Usage: setup-orphan-branch.sh <repo_url> [branch] [local_dir] [brotli_level]}"
BRANCH="${2:-asciinema-recordings}"  # Default changed to match parent dir pattern
LOCAL_DIR="${3:-$HOME/asciinema_recordings/$(basename "$REPO_URL" .git)}"
BROTLI_LEVEL="${4:-9}"  # Embedded from Phase 3 selection

# Create temporary clone for setup
TEMP_DIR=$(mktemp -d)
trap "rm -rf $TEMP_DIR" EXIT

git clone --depth 1 "$REPO_URL" "$TEMP_DIR"
cd "$TEMP_DIR"

# Create orphan branch
git checkout --orphan "$BRANCH"
git rm -rf .

# Setup directory structure
mkdir -p .github/workflows chunks archives

# Create workflow with user-selected brotli level (EMBEDDED at creation time)
cat > .github/workflows/recompress.yml << WORKFLOW_EOF
name: Recompress to Brotli

on:
  push:
    branches: [$BRANCH]
    paths: ['chunks/**/*.zst']
  workflow_dispatch:

jobs:
  recompress:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4

      - name: Install compression tools
        run: sudo apt-get update && sudo apt-get install -y zstd brotli

      - name: Recompress chunks to brotli
        run: |
          if compgen -G "chunks/*.zst" > /dev/null; then
            mkdir -p archives
            ARCHIVE_NAME="archive_\$(date +%Y%m%d_%H%M%S).cast.br"
            ls -1 chunks/*.zst | sort | xargs cat | zstd -d | brotli -${BROTLI_LEVEL} -o "archives/\$ARCHIVE_NAME"
            rm -f chunks/*.zst
            echo "Created: archives/\$ARCHIVE_NAME"
            echo "ARCHIVE_NAME=\$ARCHIVE_NAME" >> \$GITHUB_ENV
          else
            echo "No chunks to process"
          fi

      - name: Commit archive
        if: env.ARCHIVE_NAME != ''
        uses: stefanzweifel/git-auto-commit-action@v5
        with:
          commit_message: "chore: archive recording to brotli (\${{ env.ARCHIVE_NAME }})"
          file_pattern: 'archives/*.br chunks/'
WORKFLOW_EOF

# Create placeholder files
echo '# Recording chunks (zstd compressed)' > chunks/README.md
echo '# Brotli archives (final compressed)' > archives/README.md

# Create README
cat > README.md << 'README_EOF'
# Recording Storage (Orphan Branch)

This branch stores asciinema recording backups. It is completely isolated from the main codebase.

## Structure

- `chunks/` - Streaming zstd-compressed chunks (auto-deleted after archival)
- `archives/` - Final brotli-compressed recordings (~300x compression)

## How It Works

1. Local idle-chunker monitors asciinema recording
2. When idle ≥30s, creates zstd chunk and pushes here
3. GitHub Action concatenates chunks and recompresses to brotli
4. Chunks are deleted, archive is retained

## Isolation Guarantee

This is an orphan branch with no shared history with main.
Git refuses to merge: "refusing to merge unrelated histories"
README_EOF

# Commit and push
git add .
git commit -m "init: recording storage (orphan branch)"
git push -u origin "$BRANCH"

cd -

# Clone to local recordings directory
mkdir -p "$(dirname "$LOCAL_DIR")"
git clone --single-branch --branch "$BRANCH" --depth 1 "$REPO_URL" "$LOCAL_DIR"
echo "Setup complete: $LOCAL_DIR"
SETUP_ORPHAN_EOF
```

---

### Phase 5: Local Environment Setup

**Purpose**: Configure local directory and generate chunker script with user parameters.

#### Setup Local Directory

```bash
/usr/bin/env bash << 'SETUP_LOCAL_EOF'
REPO_NAME="${1:?Usage: provide repo name}"
REPO_URL="${2:?Usage: provide repo URL}"
BRANCH="${3:-asciinema-recordings}"

LOCAL_DIR="$HOME/asciinema_recordings/${REPO_NAME}"

# Ensure directories exist
mkdir -p "$LOCAL_DIR/chunks"
mkdir -p "$LOCAL_DIR/archives"

# Clone if not present
if [[ ! -d "$LOCAL_DIR/.git" ]]; then
  git clone --single-branch --branch "$BRANCH" --depth 1 "$REPO_URL" "$LOCAL_DIR"
fi

echo "LOCAL_DIR=$LOCAL_DIR"
SETUP_LOCAL_EOF
```

#### Generate Customized idle-chunker.sh

Generate the chunker script with user-selected parameters embedded:

```bash
/usr/bin/env bash << 'GEN_CHUNKER_EOF'
# Parameters from Phase 3 (passed as arguments)
LOCAL_DIR="${1:?Usage: provide LOCAL_DIR}"
IDLE_THRESHOLD="${2:-30}"
ZSTD_LEVEL="${3:-3}"
POLL_INTERVAL="${4:-5}"
PUSH_ENABLED="${5:-true}"

cat > "$LOCAL_DIR/idle-chunker.sh" << CHUNKER_EOF
#!/usr/bin/env bash
# idle-chunker.sh - Generated with user configuration
#
# Configuration (embedded from setup):
#   IDLE_THRESHOLD=${IDLE_THRESHOLD}
#   ZSTD_LEVEL=${ZSTD_LEVEL}
#   POLL_INTERVAL=${POLL_INTERVAL}
#   PUSH_ENABLED=${PUSH_ENABLED}

set -euo pipefail

CAST_FILE="\${1:?Usage: idle-chunker.sh <cast_file>}"

# Embedded configuration
IDLE_THRESHOLD=${IDLE_THRESHOLD}
ZSTD_LEVEL=${ZSTD_LEVEL}
POLL_INTERVAL=${POLL_INTERVAL}
PUSH_ENABLED=${PUSH_ENABLED}

cd "\$(dirname "\$0")"
last_pos=0

echo "Monitoring: \$CAST_FILE"
echo "Idle threshold: \${IDLE_THRESHOLD}s | zstd level: \${ZSTD_LEVEL} | Poll: \${POLL_INTERVAL}s"

while [[ -f "\$CAST_FILE" ]] || sleep 2; do
  [[ -f "\$CAST_FILE" ]] || continue
  mtime=\$(stat -f%m "\$CAST_FILE" 2>/dev/null || stat -c%Y "\$CAST_FILE")
  idle=\$((\$(date +%s) - mtime))
  size=\$(stat -f%z "\$CAST_FILE" 2>/dev/null || stat -c%s "\$CAST_FILE")

  if (( idle >= IDLE_THRESHOLD && size > last_pos )); then
    chunk="chunks/chunk_\$(date +%Y%m%d_%H%M%S).cast"
    tail -c +\$((last_pos + 1)) "\$CAST_FILE" > "\$chunk"
    zstd -\${ZSTD_LEVEL} --rm "\$chunk"

    if [[ "\$PUSH_ENABLED" == "true" ]]; then
      git add chunks/ && git commit -m "chunk \$(date +%H:%M)" && git push
    fi

    last_pos=\$size
    echo "[\$(date +%H:%M:%S)] Created: \${chunk}.zst"
  fi

  sleep \$POLL_INTERVAL
done
CHUNKER_EOF

chmod +x "$LOCAL_DIR/idle-chunker.sh"
echo "Generated: $LOCAL_DIR/idle-chunker.sh"
GEN_CHUNKER_EOF
```

#### Display Configuration Summary

```bash
/usr/bin/env bash << 'SETUP_EOF'
echo ""
echo "=== Setup Complete ==="
echo ""
echo "Configuration:"
echo "  Repository: $REPO_URL"
echo "  Branch: $BRANCH"
echo "  Local directory: $LOCAL_DIR"
echo ""
echo "Parameters:"
echo "  Idle threshold: ${IDLE_THRESHOLD}s"
echo "  zstd level: $ZSTD_LEVEL"
echo "  Brotli level: $BROTLI_LEVEL"
echo "  Auto-push: $PUSH_ENABLED"
echo "  Poll interval: ${POLL_INTERVAL}s"
echo ""
echo "To start recording:"
echo "  1. asciinema rec /path/to/session.cast"
echo "  2. $LOCAL_DIR/idle-chunker.sh /path/to/session.cast"
SETUP_EOF
```

---

### Phase 6: Autonomous Validation

**Purpose**: Claude executes validation tests automatically, displaying results in CLI. Only interrupts user when human action is required.

#### Validation Test Categories

| Test                        | Autonomous? | Reason                      |
| --------------------------- | ----------- | --------------------------- |
| 1. Tool preflight           | ✅ YES      | Bash checks tools           |
| 2. zstd round-trip          | ✅ YES      | Synthetic test data         |
| 3. Brotli round-trip        | ✅ YES      | Synthetic test data         |
| 4. zstd concatenation       | ✅ YES      | Critical for streaming      |
| 5. Git/gh auth check        | ✅ YES      | Query auth status           |
| 6. Orphan branch validation | ✅ YES      | Check remote/local          |
| 7. Workflow file check      | ✅ YES      | Read file contents          |
| 8. GitHub Actions trigger   | ✅ YES      | `gh workflow run` + watch   |
| 9. Recording test           | ❌ USER     | Requires starting asciinema |
| 10. Chunker live test       | ❌ USER     | Requires active recording   |

#### Autonomous Execution

Claude runs the validation script and displays formatted results:

```
╔════════════════════════════════════════════════════════════════╗
║ AUTONOMOUS VALIDATION - Claude Code Executes All Tests         ║
╠════════════════════════════════════════════════════════════════╣
║                                                                 ║
║  Phase 1: Tool Check                                           ║
║  ─────────────────                                             ║
║  [RUN] Checking asciinema... ✓ installed (v3.0.0)              ║
║  [RUN] Checking zstd... ✓ installed (v1.5.5)                   ║
║  [RUN] Checking brotli... ✓ installed (v1.1.0)                 ║
║  [RUN] Checking git... ✓ installed (v2.43.0)                   ║
║  [RUN] Checking gh... ✓ installed (v2.40.0)                    ║
║                                                                 ║
║  Phase 2: Compression Tests                                    ║
║  ────────────────────────                                      ║
║  [RUN] zstd round-trip... ✓ PASSED                             ║
║  [RUN] brotli round-trip... ✓ PASSED                           ║
║  [RUN] zstd concatenation... ✓ PASSED (critical for streaming) ║
║                                                                 ║
║  Phase 3: Repository Validation                                ║
║  ─────────────────────────────                                 ║
║  [RUN] Checking gh auth... ✓ authenticated as terrylica        ║
║  [RUN] Checking orphan branch... ✓ gh-recordings exists        ║
║  [RUN] Checking local clone... ✓ ~/asciinema_recordings/repo   ║
║  [RUN] Checking workflow file... ✓ recompress.yml present      ║
║                                                                 ║
║  Phase 4: GitHub Actions Test                                  ║
║  ─────────────────────────────                                 ║
║  [RUN] Triggering workflow_dispatch... ✓ triggered             ║
║  [RUN] Watching run #12345... ⏳ in_progress                   ║
║  [RUN] Watching run #12345... ✓ completed (success)            ║
║                                                                 ║
║  ═══════════════════════════════════════════════════════════   ║
║  AUTONOMOUS TESTS: 8/8 PASSED                                  ║
║  ═══════════════════════════════════════════════════════════   ║
╚════════════════════════════════════════════════════════════════╝
```

#### User-Required Tests

Only TWO tests require user action:

**Test 9: Recording Validation**

```yaml
AskUserQuestion:
  question: "Ready to test recording? This requires you to start asciinema in another terminal."
  header: "Recording Test"
  options:
    - label: "Guide me through it (Recommended)"
      description: "Step-by-step instructions"
    - label: "Skip this test"
      description: "I'll verify manually later"
    - label: "I've already verified recording works"
      description: "Mark as passed"
```

If "Guide me through it" selected, display:

```
╔════════════════════════════════════════════════════════════════╗
║ USER ACTION REQUIRED: Recording Test                           ║
╠════════════════════════════════════════════════════════════════╣
║                                                                 ║
║  In a NEW terminal, run:                                       ║
║  ┌────────────────────────────────────────────────────────┐    ║
║  │ asciinema rec ~/asciinema_recordings/test_session.cast │    ║
║  └────────────────────────────────────────────────────────┘    ║
║                                                                 ║
║  Then type a few commands and exit with Ctrl+D                 ║
║                                                                 ║
║  Come back here when done.                                     ║
╚════════════════════════════════════════════════════════════════╝
```

Then Claude autonomously validates the created file:

```bash
# Claude runs after user confirms:
[RUN] Checking test_session.cast exists... ✓
[RUN] Validating JSON header... ✓ {"version": 2, ...}
[RUN] Checking line count... ✓ 23 events recorded
```

**Test 10: Chunker Live Test**

```yaml
AskUserQuestion:
  question: "Ready to test live chunking? This requires running recording + chunker simultaneously."
  header: "Chunker Test"
  options:
    - label: "Guide me (Recommended)"
      description: "Two-terminal workflow instructions"
    - label: "Skip - I trust the setup"
      description: "Skip live test"
```

#### Full Validation Script

See [references/autonomous-validation.md](./references/autonomous-validation.md) for the complete validation script.

#### Troubleshooting on Failure

If any test fails, Claude displays inline troubleshooting:

```
[RUN] Checking gh auth... ✗ FAILED

      Troubleshooting:
      1. Run: gh auth login
      2. Select: GitHub.com
      3. Choose: HTTPS or SSH
      4. Follow prompts to authenticate

      Then re-run validation.
```

---

## Quick Start

### First-Time Setup

```bash
/usr/bin/env bash << 'PREFLIGHT_EOF'
# 1. Check requirements
for tool in asciinema zstd brotli git gh; do
  command -v "$tool" &>/dev/null && echo "$tool: OK" || echo "$tool: MISSING"
done

# 2. Create orphan branch (replace with your repo)
REPO="git@github.com:YOUR/REPO.git"
./setup-orphan-branch.sh "$REPO"

# 3. Validate setup
./validate-setup.sh "$HOME/asciinema_recordings/REPO"
PREFLIGHT_EOF
```

### Recording Session

```bash
/usr/bin/env bash << 'SKILL_SCRIPT_EOF'
# Terminal 1: Start recording
WORKSPACE=$(basename "$PWD")
asciinema rec $PWD/tmp/${WORKSPACE}_$(date +%Y-%m-%d_%H-%M).cast

# Terminal 2: Start idle-chunker
~/asciinema_recordings/REPO/idle-chunker.sh $PWD/tmp/${WORKSPACE}_*.cast
SKILL_SCRIPT_EOF
```

---

## TodoWrite Task Templates

### Template: Full Setup

```
1. [Preflight] Validate all tools installed (asciinema, zstd, brotli, git, gh)
2. [Preflight] AskUserQuestion: offer installation for missing tools
3. [Account] Detect GitHub accounts from 5 sources
4. [Account] AskUserQuestion: select GitHub account
5. [Config] AskUserQuestion: repository URL
6. [Config] AskUserQuestion: recording directory
7. [Config] AskUserQuestion: branch name
8. [Advanced] AskUserQuestion: idle threshold
9. [Advanced] AskUserQuestion: zstd level
10. [Advanced] AskUserQuestion: brotli level
11. [Advanced] AskUserQuestion: auto-push
12. [Advanced] AskUserQuestion: poll interval
13. [Branch] Check if orphan branch exists on remote
14. [Branch] AskUserQuestion: handle existing branch
15. [Branch] Create orphan branch if needed
16. [Branch] Create GitHub Actions workflow with embedded parameters
17. [Local] Clone orphan branch to ~/asciinema_recordings/
18. [Local] Generate idle-chunker.sh with embedded parameters
19. [Validate] Run autonomous validation (8 tests)
20. [Validate] AskUserQuestion: recording test (user action)
21. [Validate] AskUserQuestion: chunker live test (user action)
22. [Guide] Display configuration summary and usage instructions
```

### Template: Recording Session

```
1. [Context] Detect workspace from $PWD
2. [Context] Generate datetime for filename
3. [Context] Ensure tmp/ directory exists
4. [Command] Generate asciinema rec command
5. [Command] Generate idle-chunker command
6. [Guide] Display two-terminal workflow instructions
```

---

## Troubleshooting

### "Cannot push to orphan branch"

**Cause**: Authentication or permissions issue.

**Fix**:

```bash
# Check gh auth status
gh auth status

# Re-authenticate if needed
gh auth login
```

### "Chunks not being created"

**Cause**: Idle threshold not reached, or file not growing.

**Fix**:

- Verify recording is active: `tail -f $CAST_FILE`
- Lower threshold: `IDLE_THRESHOLD=15`
- Check file permissions

### "GitHub Action not triggering"

**Cause**: Workflow file missing or wrong branch filter.

**Fix**:

```bash
# Verify workflow exists
cat ~/asciinema_recordings/REPO/.github/workflows/recompress.yml

# Check branch filter includes gh-recordings
grep -A2 "branches:" ~/asciinema_recordings/REPO/.github/workflows/recompress.yml
```

### "Brotli archive empty or corrupted"

**Cause**: zstd chunks not concatenating properly (overlapping data).

**Fix**: Ensure idle-chunker uses `last_chunk_pos` to avoid overlap:

```bash
/usr/bin/env bash << 'PREFLIGHT_EOF_2'
# Check for overlaps - each chunk should be sequential
for f in chunks/*.zst; do
  zstd -d "$f" -c | head -1
done
PREFLIGHT_EOF_2
```

---

## Key Design Decisions

| Decision                | Rationale                                          |
| ----------------------- | -------------------------------------------------- |
| **zstd for streaming**  | Supports frame concatenation (brotli doesn't)      |
| **brotli for archival** | Best compression ratio (~300x for .cast files)     |
| **Orphan branch**       | Complete isolation, can't pollute main history     |
| **Idle-based chunking** | Semantic breakpoints, not mid-output splits        |
| **Shallow clone**       | Minimal disk usage, can't accidentally access main |
| **30s idle threshold**  | Balances chunk frequency vs semantic completeness  |

---

## Post-Change Checklist

After modifying this skill:

1. [ ] Orphan branch creation scripts use heredoc wrapper
2. [ ] All bash blocks compatible with zsh (no declare -A, no grep -P)
3. [ ] GitHub Actions workflow validates brotli recompression
4. [ ] Idle chunker handles both macOS and Linux stat syntax
5. [ ] Detection flow outputs parseable key=value format
6. [ ] References validate links to external documentation

---

## Reference Documentation

- [Idle Chunker Script](./references/idle-chunker.md) - Complete chunker implementation
- [GitHub Workflow](./references/github-workflow.md) - Full Actions workflow
- [Setup Scripts](./references/setup-scripts.md) - All setup and validation scripts
- [Autonomous Validation](./references/autonomous-validation.md) - Validation script and user-required tests
- [asciinema 3.0 Docs](https://docs.asciinema.org/)
- [zstd Frame Format](https://github.com/facebook/zstd)
- [Git Orphan Branches](https://graphite.dev/guides/git-orphan-branches)
