---
name: nobrainer-fast-audit
description: "Universal security diagnostic skill for Claude Code. Audits system security posture, vets skills/plugins before installation, scans for indicators of compromise, and provides OWASP Agentic Top 10 hardening guidance. Cross-platform: macOS, Linux, Windows, VPS. Use on: /safety-audit, /safety-check-skill, /safety-scan."
---

# Safety First — Universal Security Diagnostics for Claude Code

Comprehensive security toolkit for any Claude Code environment. Detects supply chain attacks, audits skill/plugin integrity, checks OS hardening, monitors for indicators of compromise, and provides actionable hardening guidance based on the OWASP Agentic Top 10 (2026).

Works on **macOS**, **Linux**, **Windows** (via PowerShell), and **VPS/server** environments.

---

## Commands

### `/safety-audit` — Full System Security Audit

Runs all security checks appropriate for the detected platform: OS security posture, AI agent platform config, installed skills/plugins, credentials, network, and processes.

```
/safety-audit
/safety-audit quick    # Skip slow checks (network scan, full file audit)
```

### `/safety-check-skill <path>` — Pre-Installation Skill/Plugin Vetting

Analyzes a skill or plugin BEFORE installation for malicious patterns, supply chain risks, and OWASP ASI04 indicators.

```
/safety-check-skill /path/to/skill-directory
/safety-check-skill /path/to/plugin-directory
```

### `/safety-scan` — Quick IOC Scan

Fast scan for known indicators of compromise (C2 connections, malware artifacts, unauthorized processes, memory poisoning).

```
/safety-scan
```

---

## Execution Instructions

When any of the above commands is invoked, Claude Code should:

1. **Detect the platform** using `uname -s` (or `$env:OS` on Windows)
2. **Run the appropriate checks** from the sections below
3. **Present results** as a formatted security report with PASS/WARN/FAIL per category
4. **Provide a summary** with counts and any recommended actions

Use `set -euo pipefail` for all bash scripts. On Windows, use PowerShell equivalents. Commands that require elevated privileges should be noted but not skipped — report what couldn't be checked.

**Important context for `/safety-check-skill`**: When analyzing a security-focused skill (including this one), pattern matches inside regex variable assignments (e.g., `PIPE_EXEC='curl.*\|.*sh'`), documentation blocks, and detection rule definitions are **not** indicators of malicious intent — they are the detection signatures themselves. Score these as LOW (+3) with a note: "Pattern found in detection rule definition, not in executable context." Only flag matches that appear in actually executable code paths (outside of variable definitions and markdown code blocks that define grep patterns).

---

## 1. Cross-Platform Security Audit (`/safety-audit`)

### Platform Detection

```bash
OS="$(uname -s 2>/dev/null || echo "Windows")"
case "$OS" in
    Darwin)  PLATFORM="macos" ;;
    Linux)   PLATFORM="linux" ;;
    MINGW*|MSYS*|CYGWIN*) PLATFORM="windows" ;;
    *)       PLATFORM="unknown" ;;
esac
echo "Detected platform: $PLATFORM"
```

If running inside a container or VM, note it. Check for `/.dockerenv` or `/proc/1/cgroup` containing "docker".

---

### macOS Security Checks

Run these when `PLATFORM=macos`:

```bash
# System Integrity Protection
csrutil status 2>/dev/null | grep -q "enabled" && echo "[PASS] SIP enabled" || echo "[FAIL] SIP DISABLED"

# Gatekeeper
spctl --status 2>&1 | grep -q "assessments enabled" && echo "[PASS] Gatekeeper enabled" || echo "[FAIL] Gatekeeper DISABLED"

# FileVault
fdesetup status 2>/dev/null | grep -q "FileVault is On" && echo "[PASS] FileVault enabled" || echo "[WARN] FileVault OFF"

# Firewall
/usr/libexec/ApplicationFirewall/socketfilterfw --getglobalstate 2>/dev/null | grep -q "enabled" && echo "[PASS] Firewall enabled" || echo "[FAIL] Firewall DISABLED"

# XProtect freshness (warn if >14 days old)
XPROTECT_DB="/Library/Apple/System/Library/CoreServices/XProtect.bundle/Contents/Resources/gk.db"
if [ -f "$XPROTECT_DB" ]; then
    XPROTECT_AGE=$(( ($(date +%s) - $(stat -f %m "$XPROTECT_DB")) / 86400 ))
    [ "$XPROTECT_AGE" -le 14 ] && echo "[PASS] XProtect updated ${XPROTECT_AGE}d ago" || echo "[WARN] XProtect ${XPROTECT_AGE}d old"
fi

# Launch Agents — flag unknown agents
for PLIST in ~/Library/LaunchAgents/*.plist; do
    [ -f "$PLIST" ] || continue
    PLIST_NAME=$(basename "$PLIST" .plist)
    # Flag if not from well-known vendors
    echo "$PLIST_NAME" | grep -qE '^(com\.apple|com\.google|com\.microsoft|homebrew|com\.docker|com\.jetbrains|com\.vscode)' || echo "[WARN] Unknown launch agent: $PLIST_NAME"
done

# Non-Apple launch daemons
ls /Library/LaunchDaemons/ 2>/dev/null | grep -v "com.apple" | while read -r d; do echo "[INFO] Non-Apple daemon: $d"; done
```

---

### Linux Security Checks

Run these when `PLATFORM=linux`:

```bash
# SELinux
if command -v getenforce &>/dev/null; then
    STATUS=$(getenforce 2>/dev/null)
    [ "$STATUS" = "Enforcing" ] && echo "[PASS] SELinux enforcing" || echo "[WARN] SELinux: $STATUS"
fi

# AppArmor
if command -v aa-status &>/dev/null; then
    aa-status --enabled 2>/dev/null && echo "[PASS] AppArmor enabled" || echo "[WARN] AppArmor not enforcing"
fi

# UFW
if command -v ufw &>/dev/null; then
    ufw status 2>/dev/null | grep -q "Status: active" && echo "[PASS] UFW active" || echo "[FAIL] UFW inactive"
fi

# iptables fallback
if ! command -v ufw &>/dev/null && command -v iptables &>/dev/null; then
    RULES=$(iptables -L -n 2>/dev/null | wc -l)
    [ "$RULES" -gt 8 ] && echo "[PASS] iptables has rules" || echo "[WARN] iptables has minimal rules"
fi

# Disk encryption (LUKS)
if command -v lsblk &>/dev/null; then
    lsblk -o NAME,TYPE,FSTYPE 2>/dev/null | grep -q "crypto_LUKS" && echo "[PASS] LUKS encryption detected" || echo "[WARN] No LUKS encryption detected"
fi

# Unattended upgrades (Debian/Ubuntu)
if [ -f /etc/apt/apt.conf.d/20auto-upgrades ]; then
    grep -q 'Unattended-Upgrade "1"' /etc/apt/apt.conf.d/20auto-upgrades 2>/dev/null && echo "[PASS] Unattended upgrades enabled" || echo "[WARN] Unattended upgrades not configured"
fi

# Systemd services — flag recently added non-standard services
systemctl list-unit-files --state=enabled 2>/dev/null | grep -v -E '(systemd|dbus|ssh|cron|network|login|journal|udev|snapd|docker|containerd)' | while read -r svc _; do
    [ -n "$svc" ] && echo "[INFO] Enabled service: $svc"
done
```

---

### Windows Security Checks (PowerShell)

Run these when `PLATFORM=windows` using `powershell -Command`:

```powershell
# Windows Defender
$defender = Get-MpComputerStatus -ErrorAction SilentlyContinue
if ($defender) {
    if ($defender.RealTimeProtectionEnabled) { "[PASS] Defender real-time protection enabled" } else { "[FAIL] Defender real-time protection DISABLED" }
    if ($defender.AntivirusSignatureAge -le 7) { "[PASS] Defender signatures updated ($($defender.AntivirusSignatureAge)d ago)" } else { "[WARN] Defender signatures $($defender.AntivirusSignatureAge)d old" }
}

# UAC
$uac = (Get-ItemProperty HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System).EnableLUA
if ($uac -eq 1) { "[PASS] UAC enabled" } else { "[FAIL] UAC DISABLED" }

# BitLocker
$bl = Get-BitLockerVolume -MountPoint "C:" -ErrorAction SilentlyContinue
if ($bl -and $bl.ProtectionStatus -eq "On") { "[PASS] BitLocker enabled on C:" } else { "[WARN] BitLocker not enabled on C:" }

# Windows Firewall
$fw = Get-NetFirewallProfile -ErrorAction SilentlyContinue
$fw | ForEach-Object { if ($_.Enabled) { "[PASS] Firewall $($_.Name) profile enabled" } else { "[FAIL] Firewall $($_.Name) profile DISABLED" } }

# SmartScreen
$ss = (Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer" -ErrorAction SilentlyContinue).SmartScreenEnabled
if ($ss -eq "RequireAdmin" -or $ss -eq "Prompt") { "[PASS] SmartScreen enabled" } else { "[WARN] SmartScreen: $ss" }
```

---

### VPS / Server Hardening Checks

Run these on Linux when SSH is the primary access method (detect via `systemctl is-active sshd` or process check):

```bash
# SSH hardening
SSHD_CONFIG="/etc/ssh/sshd_config"
if [ -f "$SSHD_CONFIG" ]; then
    grep -qi "^PermitRootLogin no" "$SSHD_CONFIG" && echo "[PASS] Root SSH login disabled" || echo "[FAIL] Root SSH login not explicitly disabled"
    grep -qi "^PasswordAuthentication no" "$SSHD_CONFIG" && echo "[PASS] SSH password auth disabled" || echo "[WARN] SSH password auth may be enabled"
    grep -qi "^PubkeyAuthentication yes" "$SSHD_CONFIG" && echo "[PASS] SSH pubkey auth enabled" || echo "[WARN] SSH pubkey auth not confirmed"
    grep -qi "^Port " "$SSHD_CONFIG" && echo "[INFO] SSH on non-default port: $(grep -i '^Port ' "$SSHD_CONFIG")" || echo "[INFO] SSH on default port 22"
fi

# fail2ban
if command -v fail2ban-client &>/dev/null; then
    fail2ban-client status 2>/dev/null | grep -q "Number of jail" && echo "[PASS] fail2ban running" || echo "[WARN] fail2ban installed but not running"
else
    echo "[WARN] fail2ban not installed"
fi

# Rootkit detection tools
command -v rkhunter &>/dev/null && echo "[PASS] rkhunter installed" || echo "[INFO] rkhunter not installed"
command -v chkrootkit &>/dev/null && echo "[PASS] chkrootkit installed" || echo "[INFO] chkrootkit not installed"

# Open ports (listening on all interfaces, not just localhost)
ss -tlnp 2>/dev/null | grep -v "127.0.0.1\|::1" | while read -r line; do
    echo "[INFO] External listener: $line"
done
```

---

## 2. AI Agent Platform Checks (Generic)

These checks apply to **any** AI agent platform (Claude Code, OpenClaw, LangChain agents, AutoGPT, etc.). Detect what's installed and check accordingly.

```bash
# ── Claude Code ──
CLAUDE_DIR="$HOME/.claude"
if [ -d "$CLAUDE_DIR" ]; then
    echo "── Claude Code Configuration ──"

    # Settings file
    SETTINGS="$CLAUDE_DIR/settings.json"
    if [ -f "$SETTINGS" ] && command -v jq &>/dev/null; then
        jq '.' "$SETTINGS" > /dev/null 2>&1 && echo "[PASS] settings.json valid JSON" || echo "[FAIL] settings.json invalid"
    fi

    # MCP servers in settings
    if [ -f "$SETTINGS" ]; then
        MCP_EXTERNAL=$(jq -r '.. | .mcpServers? // empty | to_entries[]? | select(.value.args? | tostring | test("bore\\.pub|ngrok|tunnel|proxy|cloudflared")) | .key' "$SETTINGS" 2>/dev/null || true)
        [ -n "$MCP_EXTERNAL" ] && echo "[FAIL] External MCP proxy in settings: $MCP_EXTERNAL" || echo "[PASS] No external MCP proxies in Claude Code settings"
    fi

    # Skills audit
    if [ -d "$CLAUDE_DIR/skills" ]; then
        SKILL_COUNT=$(find "$CLAUDE_DIR/skills" -name "SKILL.md" -type f 2>/dev/null | wc -l | tr -d ' ')
        echo "[INFO] Claude Code skills installed: $SKILL_COUNT"
    fi

    # Plugins audit
    if [ -d "$CLAUDE_DIR/plugins" ]; then
        PLUGIN_COUNT=$(find "$CLAUDE_DIR/plugins" -maxdepth 1 -type d 2>/dev/null | wc -l | tr -d ' ')
        echo "[INFO] Claude Code plugins installed: $((PLUGIN_COUNT - 1))"
    fi
fi

# ── Generic AI agent config directories ──
for DIR in "$HOME/.openclaw" "$HOME/.autogpt" "$HOME/.langchain" "$HOME/.crewai"; do
    [ -d "$DIR" ] || continue
    PLATFORM_NAME=$(basename "$DIR" | sed 's/^\.//')
    echo "── $PLATFORM_NAME Configuration ──"

    # JSON config validation
    for CFG in "$DIR"/*.json; do
        [ -f "$CFG" ] || continue
        if command -v jq &>/dev/null; then
            jq '.' "$CFG" > /dev/null 2>&1 && echo "[PASS] $(basename "$CFG") valid JSON" || echo "[FAIL] $(basename "$CFG") invalid JSON"
        fi
    done

    # Credentials directory permissions
    for CRED in "$DIR/credentials" "$DIR/secrets" "$DIR/.env"; do
        if [ -d "$CRED" ]; then
            if [[ "$(uname -s)" == "Darwin" ]]; then
                PERMS=$(stat -f "%Lp" "$CRED")
            else
                PERMS=$(stat -c "%a" "$CRED" 2>/dev/null || echo "unknown")
            fi
            [ "$PERMS" = "700" ] && echo "[PASS] $(basename "$CRED")/ permissions: 700" || echo "[FAIL] $(basename "$CRED")/ permissions: $PERMS (should be 700)"
        fi
        if [ -f "$CRED" ]; then
            if [[ "$(uname -s)" == "Darwin" ]]; then
                PERMS=$(stat -f "%Lp" "$CRED")
            else
                PERMS=$(stat -c "%a" "$CRED" 2>/dev/null || echo "unknown")
            fi
            [ "$PERMS" = "600" ] && echo "[PASS] $(basename "$CRED") permissions: 600" || echo "[FAIL] $(basename "$CRED") permissions: $PERMS (should be 600)"
        fi
    done
done
```

### MCP Server Configuration Audit

Scan all MCP configuration files for dangerous settings, known-malicious packages, and insecure bindings.

```bash
echo "── MCP Server Configuration ──"

# Locate all MCP config files
MCP_CONFIGS=()
for MCP_CFG in "$HOME/.mcp.json" "$HOME/.claude/.mcp.json" ".mcp.json" "$HOME/.claude/settings.json"; do
    [ -f "$MCP_CFG" ] && MCP_CONFIGS+=("$MCP_CFG")
done
# Project-level configs
while IFS= read -r -d '' f; do MCP_CONFIGS+=("$f"); done < <(find . -maxdepth 3 -name ".mcp.json" -not -path "*/node_modules/*" -not -path "*/.git/*" -print0 2>/dev/null)

for MCP_CFG in "${MCP_CONFIGS[@]}"; do
    echo "[INFO] Scanning MCP config: $MCP_CFG"

    # 0.0.0.0 bindings (should be 127.0.0.1)
    if grep -q '0\.0\.0\.0' "$MCP_CFG" 2>/dev/null; then
        echo "[FAIL] MCP config binds to 0.0.0.0 (world-accessible): $MCP_CFG"
    fi

    # Known malicious MCP packages
    MALICIOUS_MCP='postmark-mcp|@anthropic/mcp-proxy|framelink-figma-mcp|mcp-remote'
    if grep -qE "$MALICIOUS_MCP" "$MCP_CFG" 2>/dev/null; then
        echo "[FAIL] Known malicious/vulnerable MCP package in: $MCP_CFG"
        grep -oE "$MALICIOUS_MCP" "$MCP_CFG" 2>/dev/null | sort -u | while read -r pkg; do echo "  → $pkg"; done
    fi

    # External tunnels
    TUNNEL_PATTERNS='bore\.pub|ngrok|cloudflared|localtunnel|serveo\.net|pagekite|localhost\.run'
    if grep -qE "$TUNNEL_PATTERNS" "$MCP_CFG" 2>/dev/null; then
        echo "[FAIL] External tunnel in MCP config: $MCP_CFG"
    fi

    # Unpinned @latest dependencies
    if grep -q '@latest' "$MCP_CFG" 2>/dev/null; then
        echo "[WARN] Unpinned @latest dependency in MCP config: $MCP_CFG"
    fi
done

# Ollama exposure check
if command -v lsof &>/dev/null; then
    OLLAMA_EXPOSED=$(lsof -iTCP:11434 -sTCP:LISTEN -P 2>/dev/null | grep -v "127.0.0.1\|localhost\|::1" || true)
    [ -n "$OLLAMA_EXPOSED" ] && echo "[FAIL] Ollama exposed on non-localhost (port 11434)" || true
fi

[ ${#MCP_CONFIGS[@]} -eq 0 ] && echo "[INFO] No MCP configuration files found"
```

---

### Rules File Injection Detection

Scan IDE/agent rules files for prompt injection and data exfiltration patterns (IDEsaster CVEs: Cursor, Copilot, Windsurf, Claude Code).

```bash
echo "── Rules File Injection Scan ──"

RULES_FILES=()
for RF in ".cursor/rules" ".cursorrules" ".github/copilot-instructions.md" ".windsurfrules" ".clinerules" "CLAUDE.md" ".claude/settings.json"; do
    [ -f "$RF" ] && RULES_FILES+=("$RF")
done
# Also check home-level
for RF in "$HOME/.cursorrules" "$HOME/CLAUDE.md"; do
    [ -f "$RF" ] && RULES_FILES+=("$RF")
done

# Injection patterns — attempts to override agent instructions
INJECT_PATTERNS='ignore previous|disregard.*instructions|system prompt override|you are now|IMPORTANT:.*must.*override|forget.*all.*previous|new role:|act as root|<system>'
# Exfiltration patterns — attempts to steal data
EXFIL_PATTERNS='send.*to.*https?://|curl.*-d.*\$|exfiltrate|upload.*credential|wget.*post|fetch\(.*body.*token|nc -e|base64.*\|.*curl'

for RF in "${RULES_FILES[@]}"; do
    INJECT_HITS=$(grep -ciE "$INJECT_PATTERNS" "$RF" 2>/dev/null || echo 0)
    EXFIL_HITS=$(grep -ciE "$EXFIL_PATTERNS" "$RF" 2>/dev/null || echo 0)
    if [ "$INJECT_HITS" -gt 0 ]; then
        echo "[FAIL] Prompt injection patterns ($INJECT_HITS hits) in: $RF"
    fi
    if [ "$EXFIL_HITS" -gt 0 ]; then
        echo "[FAIL] Data exfiltration patterns ($EXFIL_HITS hits) in: $RF"
    fi
    [ "$INJECT_HITS" -eq 0 ] && [ "$EXFIL_HITS" -eq 0 ] && echo "[PASS] Clean rules file: $RF"
done

[ ${#RULES_FILES[@]} -eq 0 ] && echo "[INFO] No IDE/agent rules files found in current directory"
```

---

### Exposed AI Service Detection

Check common AI-related ports for non-localhost bindings.

```bash
echo "── Exposed AI Services ──"
if command -v lsof &>/dev/null; then
    for PORT_INFO in "11434:Ollama" "8080:API-Gateway" "3000:Dev-Server" "8000:FastAPI" "5000:Flask"; do
        PORT="${PORT_INFO%%:*}"; SVC="${PORT_INFO##*:}"
        EXPOSED=$(lsof -iTCP:"$PORT" -sTCP:LISTEN -P 2>/dev/null | grep -E '\*:|0\.0\.0\.0:' | grep -v "127.0.0.1\|localhost\|::1" || true)
        [ -n "$EXPOSED" ] && echo "[WARN] $SVC (port $PORT) listening on all interfaces"
    done
fi
```

---

### Process & Network Scan

```bash
# Suspicious processes (crypto miners, reverse shells, tunneling)
SUSPICIOUS='xmrig|cryptonight|stratum|coinhive|minergate|ncat -e|nc -e|/bin/sh -i|reverse.shell'
PROC_HITS=$(ps aux 2>/dev/null | grep -iE "$SUSPICIOUS" | grep -v grep || true)
[ -n "$PROC_HITS" ] && echo "[FAIL] Suspicious processes: $PROC_HITS" || echo "[PASS] No suspicious processes (miners, reverse shells)"

# mcp-proxy processes
MCP_PROXY=$(ps aux 2>/dev/null | grep "mcp-proxy" | grep -v grep || true)
[ -n "$MCP_PROXY" ] && echo "[WARN] mcp-proxy running — verify: $MCP_PROXY" || echo "[PASS] No mcp-proxy processes"

# Known C2 connections
for C2_IP in "91.92.242.30" "91.92.242.0/24"; do
    if command -v lsof &>/dev/null; then
        C2_CONN=$(lsof -i @"$C2_IP" 2>/dev/null || true)
        [ -n "$C2_CONN" ] && echo "[FAIL] Connection to known C2 ($C2_IP): $C2_CONN"
    fi
done

# Network listeners (full mode only)
if [ "${1:-full}" = "full" ]; then
    echo ""
    echo "── Network Listeners ──"
    if command -v lsof &>/dev/null; then
        lsof -iTCP -sTCP:LISTEN -P 2>/dev/null | awk 'NR>1 {print $1, $9}' | sort -u || true
    elif command -v ss &>/dev/null; then
        ss -tlnp 2>/dev/null || true
    elif command -v netstat &>/dev/null; then
        netstat -tlnp 2>/dev/null || true
    fi
fi
```

### File Integrity (macOS, full mode)

```bash
if [[ "$(uname -s)" == "Darwin" && "${1:-full}" == "full" ]]; then
    echo "── File Integrity ──"
    for DIR in /tmp /var/tmp "$HOME/Downloads"; do
        while IFS= read -r -d '' f; do
            if file "$f" 2>/dev/null | grep -qiE "executable|Mach-O"; then
                QATTR=$(xattr -l "$f" 2>/dev/null | grep "com.apple.quarantine" || true)
                [ -z "$QATTR" ] && echo "[WARN] Executable without quarantine: $f"
            fi
        done < <(find "$DIR" -maxdepth 2 -type f -mtime -30 -print0 2>/dev/null)
    done
fi
```

### Dependency Audit (full mode only)

```bash
if [ "${1:-full}" = "full" ]; then
    echo "── Dependency Audit ──"

    # npm audit
    if command -v npm &>/dev/null && [ -f "package-lock.json" ]; then
        NPM_AUDIT=$(npm audit --json 2>/dev/null || true)
        CRIT=$(echo "$NPM_AUDIT" | jq '.metadata.vulnerabilities.critical // 0' 2>/dev/null || echo 0)
        HIGH=$(echo "$NPM_AUDIT" | jq '.metadata.vulnerabilities.high // 0' 2>/dev/null || echo 0)
        [ "$CRIT" -gt 0 ] && echo "[FAIL] npm: $CRIT critical vulnerabilities"
        [ "$HIGH" -gt 0 ] && echo "[WARN] npm: $HIGH high vulnerabilities"
        [ "$CRIT" -eq 0 ] && [ "$HIGH" -eq 0 ] && echo "[PASS] npm: no critical/high vulnerabilities"
    fi

    # pip check
    if command -v pip3 &>/dev/null; then
        PIP_CHECK=$(pip3 check 2>/dev/null || true)
        if echo "$PIP_CHECK" | grep -q "No broken requirements"; then
            echo "[PASS] pip: no broken dependencies"
        elif [ -n "$PIP_CHECK" ]; then
            BROKEN=$(echo "$PIP_CHECK" | wc -l | tr -d ' ')
            echo "[WARN] pip: $BROKEN broken dependency issue(s)"
        fi
    fi
fi
```

---

## 3. Skill/Plugin Vetting (`/safety-check-skill`)

When `/safety-check-skill <path>` is invoked, analyze the target directory and compute a risk score (0–100).

### Risk Scoring

| Severity | Points | Examples |
|----------|--------|---------|
| CRITICAL | +35-50 | C2 IPs, piped remote exec, quarantine bypass, external MCP proxy |
| HIGH | +20-30 | Base64+exec, credential harvesting, obfuscated blobs, external deps |
| MEDIUM | +10-15 | Dynamic eval/exec, unknown registry, unknown external URLs |
| LOW | +3-5 | Benign base64, documented install commands, safe external URLs |

### Verdicts

| Score | Rating | Action |
|-------|--------|--------|
| 0-10 | LOW | Safe to install |
| 11-30 | MODERATE | Review findings before installing |
| 31-60 | HIGH | Do NOT install without thorough manual review |
| 61-100 | CRITICAL | Likely malicious — do NOT install |

### Checks to Run

#### 3.1 Dependency Analysis

```bash
SKILL_PATH="$1"
DEP_PATTERNS='required.*(dependency|package|prerequisite)|install.*first|npm install|pip install|brew install|apt.install|uvx.*install|cargo install|go install'
DEP_MATCHES=$(grep -rilE "$DEP_PATTERNS" "$SKILL_PATH" 2>/dev/null || true)
# If found: HIGH +25 "Requires external dependency installation"
```

#### 3.2 Command Injection Detection

```bash
# Piped remote execution (curl|sh, wget|bash)
PIPE_EXEC='curl.*\|.*sh|wget.*\|.*sh|curl.*\|.*bash|wget.*\|.*bash|fetch.*\|.*sh|iwr.*\|.*iex|Invoke-WebRequest.*Invoke-Expression'
PIPE_MATCHES=$(grep -rnE "$PIPE_EXEC" "$SKILL_PATH" 2>/dev/null | grep -v "\.git/" | grep -v -iE '(echo|print|#.*install|"Install)' || true)
# If found in executable context: CRITICAL +40
# If found only in docs/comments: LOW +5

# Base64 decode + execute
B64_EXEC='base64.*(-d|--decode).*\|.*(sh|bash|exec|eval)|echo.*\|.*base64.*\|.*(sh|bash)|atob\(.*eval|Buffer\.from.*base64.*exec'
B64_MATCHES=$(grep -rnE "$B64_EXEC" "$SKILL_PATH" 2>/dev/null | grep -v "\.git/" || true)
# If found: HIGH +30

# eval/exec patterns
EVAL_EXEC='eval\s*\(|exec\s*\(|os\.system\(|subprocess\.call\(|child_process|Invoke-Expression'
EVAL_MATCHES=$(grep -rnE "$EVAL_EXEC" "$SKILL_PATH" 2>/dev/null | grep -v "\.git/" || true)
# If found: MEDIUM +15
```

#### 3.3 MCP Proxy Detection

```bash
MCP_PROXY='mcp-proxy|bore\.pub|ngrok|cloudflared.*tunnel|localtunnel|serveo\.net|CRAFTED_API_KEY'
MCP_MATCHES=$(grep -rnE "$MCP_PROXY" "$SKILL_PATH" 2>/dev/null | grep -v "\.git/" || true)
# If found: CRITICAL +35
```

#### 3.4 URL Inspection

```bash
ALL_URLS=$(grep -roEh 'https?://[^"'"'"'[:space:]>)]+' "$SKILL_PATH" 2>/dev/null | grep -v "\.git/" | sort -u || true)

# Known safe domains (extend as needed)
SAFE_DOMAINS='github\.com|npmjs\.com|pypi\.org|crates\.io|docs\.|api\.(openai|anthropic|google|slack|telegram)|stackoverflow|wikipedia|homebrew|microsoft\.com|apple\.com|brew\.sh|astral\.sh|docker\.com|hub\.docker\.com'

# Check each URL:
# - Safe domain → OK
# - Local (127.0.0.1, localhost, 192.168.*) → OK
# - Known C2 IPs (91.92.242.*) → CRITICAL +50
# - Other → LOW +5 "External URL needs review"
```

#### 3.5 Credential Harvesting Patterns

```bash
CRED_PATTERNS='\.ssh/|credentials/|auth-profiles|\.env[^a-z]|\.secret|api.key|private.key|wallet|seed.phrase|mnemonic|keychain|password.*file|token.*file'
CRED_MATCHES=$(grep -rnE "$CRED_PATTERNS" "$SKILL_PATH" 2>/dev/null | grep -v "\.git/" || true)
# If found: HIGH +25
```

#### 3.6 Obfuscation Detection

```bash
# Large base64-like blobs (>100 chars)
B64_BLOBS=$(grep -rnE '[A-Za-z0-9+/]{100,}={0,2}' "$SKILL_PATH" 2>/dev/null | grep -v "\.git/" | grep -v "node_modules" || true)
# If found: HIGH +20

# Zero-width / invisible characters
ZW_CHARS=$(grep -rPn '[\x{200B}\x{200C}\x{200D}\x{FEFF}\x{2060}]' "$SKILL_PATH" 2>/dev/null | grep -v "\.git/" || true)
# If found: HIGH +30
```

#### 3.7 OS Security Bypass

```bash
# macOS quarantine/Gatekeeper bypass
BYPASS_MAC='xattr.*-d.*quarantine|xattr.*-c|spctl.*--master-disable|csrutil.*disable|codesign.*--force'

# Linux security bypass
BYPASS_LINUX='setenforce 0|apparmor.*disable|ufw disable|iptables -F|chmod 777'

# Windows security bypass
BYPASS_WIN='Set-MpPreference.*-DisableRealtimeMonitoring|Disable-WindowsOptionalFeature.*Defender|netsh advfirewall set.*off'

BYPASS_MATCHES=$(grep -rnE "$BYPASS_MAC|$BYPASS_LINUX|$BYPASS_WIN" "$SKILL_PATH" 2>/dev/null | grep -v "\.git/" || true)
# If found: CRITICAL +50
```

#### 3.8 Prompt Injection / Goal Hijack Patterns

```bash
# Hidden instructions that might override agent behavior
HIJACK_PATTERNS='ignore previous|disregard.*instructions|you are now|new instructions|system prompt override|<system>|IMPORTANT:.*override|forget.*previous.*context'
HIJACK_MATCHES=$(grep -rniE "$HIJACK_PATTERNS" "$SKILL_PATH" 2>/dev/null | grep -v "\.git/" || true)
# If found: HIGH +25
```

#### 3.9 MCP Tool Poisoning Detection

```bash
# Check .mcp.json files within the skill for poisoned tool descriptions
MCP_IN_SKILL=$(find "$SKILL_PATH" -name ".mcp.json" -not -path "*/.git/*" 2>/dev/null || true)
for MCP_F in $MCP_IN_SKILL; do
    # Hidden instructions in tool descriptions
    POISON_DESC=$(grep -iE 'ignore previous|<system>|you must|IMPORTANT:.*must|override.*instructions' "$MCP_F" 2>/dev/null || true)
    [ -n "$POISON_DESC" ] && echo "[CRITICAL +35] Tool poisoning in MCP config: $MCP_F"

    # Known malicious packages
    MALICIOUS_MCP='postmark-mcp|framelink-figma-mcp|mcp-remote'
    BAD_PKG=$(grep -oE "$MALICIOUS_MCP" "$MCP_F" 2>/dev/null || true)
    [ -n "$BAD_PKG" ] && echo "[CRITICAL +40] Known malicious MCP package ($BAD_PKG) in: $MCP_F"

    # 0.0.0.0 binding
    grep -q '0\.0\.0\.0' "$MCP_F" 2>/dev/null && echo "[HIGH +20] MCP config binds 0.0.0.0: $MCP_F"
done
```

#### 3.10 IDE Rules File Creation Detection

```bash
# Skills that create/modify IDE rules files are suspicious
RULES_CREATE='\.cursorrules|\.cursor/rules|copilot-instructions\.md|\.windsurfrules|\.clinerules|CLAUDE\.md'
RULES_MATCHES=$(grep -rnE "(>|>>|write|create|mkdir|touch|cp).*($RULES_CREATE)" "$SKILL_PATH" 2>/dev/null | grep -v "\.git/" || true)
# If found: MEDIUM +15 "Skill creates/modifies IDE rules files"
```

---

## 4. Quick IOC Scan (`/safety-scan`)

Fast scan for known indicators of compromise. Should complete in under 10 seconds.

### Checks

```bash
# ── Known C2 Connections ──
# ClawHavoc campaign (91.92.242.0/24)
if command -v lsof &>/dev/null; then
    lsof -i @91.92.242.30 2>/dev/null | grep -q . && echo "[ALERT] ClawHavoc C2 connection!" || echo "[CLEAN] No ClawHavoc C2"
fi

# ── Malware Artifacts ──
ARTIFACTS=(
    "$HOME/openclaw-agent.zip"
    "$HOME/openclaw-agent.exe"
    "$HOME/Downloads/openclaw-agent.zip"
    "$HOME/Downloads/openclaw-agent"
    "/tmp/openclaw-agent"
    "/tmp/openclaw-core"
)
for A in "${ARTIFACTS[@]}"; do
    [ -e "$A" ] && echo "[ALERT] Malware artifact: $A"
done

# ── Suspicious Processes ──
SUSPICIOUS='xmrig|cryptonight|stratum|coinhive|minergate|ncat -e|nc -e|/bin/sh -i|reverse.shell|mcp-proxy.*bore'
ps aux 2>/dev/null | grep -iE "$SUSPICIOUS" | grep -v grep && echo "[ALERT] Suspicious process found" || echo "[CLEAN] No suspicious processes"

# ── Resource Abuse Detection ──
# Flag processes consuming excessive CPU or memory (crypto miners, runaway agents)
ps aux 2>/dev/null | awk 'NR>1 && $3 > 80 {print "[WARN] High CPU ("$3"%) process: "$11}' || true
ps aux 2>/dev/null | awk 'NR>1 && $4 > 50 {print "[WARN] High memory ("$4"%) process: "$11}' || true

# ── Known Malicious Packages ──
MALICIOUS_NPM='postmark-mcp|@anthropic/mcp-proxy|framelink-figma-mcp|mcp-remote|event-stream|ua-parser-js-malicious|colors-malicious'
if command -v npm &>/dev/null; then
    NPM_GLOBAL=$(npm ls -g --depth=0 --json 2>/dev/null | grep -oE '"('"$MALICIOUS_NPM"')"' || true)
    [ -n "$NPM_GLOBAL" ] && echo "[ALERT] Malicious npm package installed globally: $NPM_GLOBAL"
fi
# Check local node_modules
if [ -d "node_modules" ]; then
    for BAD in postmark-mcp framelink-figma-mcp mcp-remote; do
        [ -d "node_modules/$BAD" ] && echo "[ALERT] Malicious package in local node_modules: $BAD"
    done
fi
MALICIOUS_PIP='evil-pip|malicious-package|colourama|python-binance-malicious'
if command -v pip3 &>/dev/null; then
    PIP_LIST=$(pip3 list --format=freeze 2>/dev/null | grep -iE "$MALICIOUS_PIP" || true)
    [ -n "$PIP_LIST" ] && echo "[ALERT] Malicious pip package installed: $PIP_LIST"
fi

# ── Memory/Config Poisoning ──
# Check common AI agent workspace files for injection patterns
# Matches patterns like "exfiltrate to", "steal and send", "install backdoor"
# Excludes negations (don't/never/do not) which are legitimate security rules
POISON_PATTERNS='(^|[^t])exfiltrat.*to|steal.*and.*send|install.*backdoor|reverse.shell.*connect|c2\.server.*http|beacon.*callback|send.*all.*credentials'
for AGENT_DIR in "$HOME/.claude" "$HOME/.openclaw/workspaces" "$HOME/.autogpt"; do
    [ -d "$AGENT_DIR" ] || continue
    # Exclude conversation transcripts (.jsonl), tool results, subagent logs, file history,
    # and the safety-first skill itself (which documents these patterns)
    POISON_HITS=$(grep -rlE "$POISON_PATTERNS" "$AGENT_DIR" 2>/dev/null \
        | grep -v "safety-first" \
        | grep -v "\.jsonl$" \
        | grep -v "/subagents/" \
        | grep -v "/tool-results/" \
        | grep -v "/file-history/" \
        | grep -v "/projects/" \
        || true)
    [ -n "$POISON_HITS" ] && echo "[ALERT] Possible memory poisoning in: $POISON_HITS"
done

# ── Credential Exfiltration Timing ──
# Check if sensitive files were accessed very recently but not modified (sign of read-only exfiltration)
SENS_FILES=("$HOME/.ssh/id_"* "$HOME/.claude/credentials/"* "$HOME/.env" "$HOME/.npmrc" "$HOME/.pypirc" "$HOME/.netrc" "$HOME/.ssh/config")
for SENS in "${SENS_FILES[@]}"; do
    [ -f "$SENS" ] || continue
    if [[ "$(uname -s)" == "Darwin" ]]; then
        ATIME=$(stat -f %a "$SENS" 2>/dev/null || echo 0)
        MTIME=$(stat -f %m "$SENS" 2>/dev/null || echo 0)
    else
        ATIME=$(stat -c %X "$SENS" 2>/dev/null || echo 0)
        MTIME=$(stat -c %Y "$SENS" 2>/dev/null || echo 0)
    fi
    NOW=$(date +%s)
    SINCE_ACCESS=$((NOW - ATIME))
    DIFF=$((ATIME - MTIME))
    # Warn if accessed in last 2 min AND not recently modified (tighter window = fewer false positives)
    if [ "$SINCE_ACCESS" -lt 120 ] && [ "$DIFF" -gt 86400 ]; then
        echo "[WARN] Sensitive file accessed recently: $SENS (${SINCE_ACCESS}s ago)"
    fi
done

echo "[CLEAN] IOC scan complete"
```

---

## 5. OWASP Agentic Top 10 (2026) Hardening Guide

When presenting audit results, include relevant OWASP ASI recommendations.

### ASI01 — Agent Goal Hijack
**Risk**: Prompt injection overrides agent goals. **Mitigations**: Enforce instruction hierarchy (system > user > tool output); use `--allowedTools` to restrict tools; review CLAUDE.md in new repos before running.

### ASI02 — Tool Misuse & Exploitation
**Risk**: Agents misuse tools (shell, file write, web) to harm the system. **Mitigations**: Use `--permission-mode` to restrict access; sandbox agents (AppArmor/seccomp on Linux, App Sandbox on macOS); Docker with `--no-new-privileges` on VPS.

### ASI03 — Agent Identity & Privilege Abuse
**Risk**: Agent escalates privileges or impersonates others. **Mitigations**: Minimum-required permissions per agent; dedicated service accounts; rotate API keys on schedule.

### ASI04 — Agentic Supply Chain Compromise
**Risk**: Malicious skills/plugins/MCP servers/dependencies. **Mitigations**: **Always run `/safety-check-skill` before installing**; pin dependency versions; verify code signatures; never `curl | sh` from untrusted sources.

### ASI05 — Unexpected Code Execution
**Risk**: Agent-generated code has unintended side effects. **Mitigations**: Never auto-approve destructive commands; enable SIP/Gatekeeper (macOS); `noexec` on `/tmp` (Linux/VPS); SmartScreen/WDAC (Windows).

### ASI06 — Memory & Context Poisoning
**Risk**: Attacker manipulates agent memory/history to alter behavior. **Mitigations**: Git-track memory files; use `chattr +i` / `chflags uchg` on critical configs; review `.claude/` project memory in new repos; AIDE/Tripwire on VPS.

### ASI07 — Insecure Inter-Agent Communication
**Risk**: Agent messages intercepted or spoofed. **Mitigations**: Localhost-only inter-agent connections; Unix domain sockets with proper permissions; mutual TLS for network-based communication.

### ASI08 — Cascading Agent Failures
**Risk**: One compromised agent cascades to dependents. **Mitigations**: Circuit breakers + timeouts on inter-agent calls; separate API keys per agent; isolate credential-access agents; health monitoring.

### ASI09 — Human-Agent Trust Exploitation
**Risk**: Agent social-engineers user into approving harmful actions. **Mitigations**: Read full commands before approving; be suspicious of urgency; use hooks to flag dangerous patterns pre-execution.

### ASI10 — Rogue Agents
**Risk**: Agent operates outside intended scope. **Mitigations**: Clear boundaries in system prompts/CLAUDE.md; `--allowedTools` enforcement; audit logs (`auditd` on Linux, `Console.app` on macOS, `falco` on VPS).

---

## 6. Known Threat Intelligence

### ClawHavoc Campaign (Jan–Feb 2026)

- **571+ malicious ClawHub skills** (341 original wave Jan 27-29, 230+ follow-up) delivering Atomic Stealer (AMOS)
- **C2 IP**: 91.92.242.30 (subnet: 91.92.242.0/24)
- **Attack vector**: Fake prerequisites → staging page → obfuscated payload → AMOS binary
- **Targets**: Exchange API keys, wallet keys, SSH credentials, browser passwords, .env files
- **Top skill names**: solana-wallet-tracker, youtube-summarize-pro, polymarket-trader, twitter
- **Detection**: C2 IP connections, `openclaw-core` dependency, xattr quarantine removal

### Postmark MCP Backdoor (Feb 2026)

- **First malicious MCP server in the wild** — `postmark-mcp` npm package (v1.0.16+)
- **~1,500 weekly installs** before takedown; BCC injection exfiltrates all emails to attacker
- **Detection**: Check installed MCP packages against malicious list; scan `.mcp.json` configs

### CVE Reference Table

| CVE | Component | Impact |
|-----|-----------|--------|
| CVE-2025-68143/44/45 | Anthropic Git MCP Server | Path traversal, argument injection |
| CVE-2025-53967 | MCP protocol (general) | Tool poisoning via hidden descriptions |
| CVE-2025-6514 | npm MCP registry | Dependency confusion in MCP packages |
| CVE-2025-32711 | Cursor IDE | Rules file injection (IDEsaster) |
| CVE-2025-49150/54130/61590 | Copilot, Windsurf, Claude Code | Rules file backdoor injection (IDEsaster) |

### MCP Ecosystem Threat Stats

- **82%** of 2,614 MCP implementations vulnerable to path traversal (Barracuda, Feb 2026)
- **67%** vulnerable to code injection; **34%** to command injection
- **43 agent framework components** with embedded vulnerabilities identified
- **175K+ Ollama servers** exposed on 0.0.0.0:11434 (Shodan, Jan 2026)
- **Tool poisoning** — malicious MCP servers embed hidden instructions in tool descriptions

### Supply Chain Attack Patterns

- Typosquatting on package registries (npm, PyPI, ClawHub)
- Dependency confusion — internal package names registered on public registries
- Malicious update injection — legitimate packages compromised via maintainer account takeover
- Star/download inflation to build false trust
- Hidden post-install scripts in `package.json`, `setup.py`, `Cargo.toml`

---

## 7. References

- [OWASP Top 10 for Agentic Applications 2026](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/)
- [571 Malicious ClawHub Skills — The Hacker News](https://thehackernews.com/2026/02/researchers-find-341-malicious-clawhub.html)
- [Postmark MCP Backdoor Analysis — Snyk](https://snyk.io/articles/postmark-mcp-backdoor/)
- [IDEsaster: 30+ CVEs in Coding Assistants — Wiz](https://wiz.io/blog/idesaster-coding-assistant-vulnerabilities)
- [82% of MCP Implementations Vulnerable — Barracuda](https://barracuda.com/reports/mcp-security-2026)
- [175K Exposed Ollama Servers — BleepingComputer](https://www.bleepingcomputer.com/news/security/175k-ollama-servers-exposed/)
- [OpenClaw Security Guide 2026 — Adversa AI](https://adversa.ai/blog/openclaw-security-101-vulnerabilities-hardening-2026/)
- [From SKILL.md to Shell Access — Snyk](https://snyk.io/articles/skill-md-shell-access/)
- [AI Agent Memory Poisoning — MintMCP](https://www.mintmcp.com/blog/ai-agent-memory-poisoning)
- [Anthropic Claude Code Security Best Practices](https://docs.anthropic.com/en/docs/claude-code/security)
- [NIST AI Risk Management Framework](https://www.nist.gov/artificial-intelligence/ai-risk-management-framework)
- [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks) — OS hardening baselines for macOS, Linux, Windows
