---
name: swizzin-scripts-patterns
description: Coding patterns extracted from swizzin-scripts repository for Swizzin installer scripts
version: 1.0.0
source: local-git-analysis
analyzed_commits: 187
date_range: 2025-12-19 to 2026-01-30
---

# Swizzin Scripts Patterns

This skill teaches Claude the coding patterns, conventions, and workflows used in this repository of Swizzin installer scripts.

## Commit Conventions

This project uses **imperative mood commits** with action prefixes:

| Prefix   | Usage                                | Frequency        |
| -------- | ------------------------------------ | ---------------- |
| `Fix`    | Bug fixes and corrections            | 45 commits (24%) |
| `Add`    | New features, files, or capabilities | 57 commits (30%) |
| `Update` | Modifications to existing features   | 11 commits (6%)  |
| `Remove` | Deletions and cleanup                | 5 commits (3%)   |

**Examples of good commit messages:**

- `Fix LibreTranslate container permission error`
- `Add subdomain support to Lingarr installer`
- `Update docs for subgen Python 3.11 and GPU auto-detection`
- `Fix Lingarr: host networking, UrlBase support, remove broken auth rules`

**Anti-patterns to avoid:**

- Past tense: "Fixed" → use "Fix"
- Vague descriptions: "Updates" → be specific about what changed
- No prefix: Always start with an action word

## Code Architecture

```
swizzin-scripts/
├── *.sh                      # Main installer scripts
├── templates/                # Starter templates for new scripts
│   ├── template-binary.sh    # Single binary apps → /usr/bin
│   ├── template-python.sh    # Python/uv apps → /opt
│   ├── template-docker.sh    # Docker Compose apps → /opt
│   ├── template-subdomain.sh # Extended installers with subdomain
│   └── template-multiinstance.sh  # Multi-instance managers
├── panel_helpers.sh          # Shared panel integration utilities
├── backup/                   # BorgBackup system
├── watchdog/                 # Service health monitoring
├── configs/                  # Example configuration files
├── docs/plans/               # Design documents (YYYY-MM-DD-*-design.md)
├── CLAUDE.md                 # AI assistant context
└── README.md                 # User documentation
```

### Script Categories

| Type           | Pattern                     | Binary Location  | Examples                         |
| -------------- | --------------------------- | ---------------- | -------------------------------- |
| Single binary  | `template-binary.sh`        | `/usr/bin/<app>` | notifiarr, decypharr, cleanuparr |
| Python/uv      | `template-python.sh`        | `/opt/<app>/`    | byparr, huntarr, subgen          |
| Docker         | `template-docker.sh`        | `/opt/<app>/`    | lingarr, libretranslate          |
| Extended       | `template-subdomain.sh`     | varies           | plex, emby, jellyfin, seerr      |
| Multi-instance | `template-multiinstance.sh` | varies           | sonarr, radarr, bazarr           |

## Function Naming Convention

All internal functions use underscore prefix with pattern `_<action>_<appname>()`:

```bash
_install_notifiarr()    # Main installation logic
_systemd_notifiarr()    # Create systemd service
_nginx_notifiarr()      # Create nginx config
_remove_notifiarr()     # Removal logic
_load_panel_helper()    # Load panel helper (standard)
```

**Other common function names:**

- `_get_domain()` / `_prompt_domain()` - Domain management
- `_get_install_state()` - Detect subfolder vs subdomain
- `_create_subdomain_vhost()` - Nginx subdomain config
- `_validate_instance_name()` - Input validation
- `_ensure_base_installed()` - Prerequisite checks

## Script Structure Pattern

Every installer script follows this exact sequence:

```bash
#!/bin/bash
# <appname> installer
# STiXzoOR 2025
# Usage: bash <appname>.sh [--remove [--force]|--register-panel]

# 1. Source Swizzin utilities
. /etc/swizzin/sources/globals.sh
#shellcheck source=sources/functions/utils
. /etc/swizzin/sources/functions/utils

# 2. Panel helper (download and cache)
PANEL_HELPER_LOCAL="/opt/swizzin-extras/panel_helpers.sh"
PANEL_HELPER_URL="https://raw.githubusercontent.com/STiXzoOR/swizzin-scripts/main/panel_helpers.sh"
_load_panel_helper() { ... }

# 3. Logging setup
export log=/root/logs/swizzin.log
touch "$log"

# 4. App configuration variables
app_name="myapp"
app_port=$(port 10000 12000)
app_dir="/usr/bin"  # or /opt/myapp
app_configdir="/home/${user}/.config/Myapp"
app_icon_url="https://cdn.jsdelivr.net/gh/selfhst/icons@main/png/myapp.png"

# 5. Owner resolution (from swizdb or master user)
if ! app_owner="$(swizdb get "${app_name}/owner" 2>/dev/null)"; then
    app_owner="$(_get_master_username)"
fi
user="${app_owner}"

# 6. Core functions
_install_myapp() { ... }
_systemd_myapp() { ... }
_nginx_myapp() { ... }
_remove_myapp() { ... }

# 7. Main logic (argument parsing)
case "$1" in
    --remove) _remove_myapp "$@" ;;
    --register-panel) _load_panel_helper && panel_register_app ... ;;
    *) _install_myapp && _systemd_myapp && _nginx_myapp && ... ;;
esac
```

## Variable Quoting Standards

**Always quote variables and use braces for clarity:**

```bash
# CORRECT
touch "$log"
chown "${user}:${user}" "$config_dir"
mkdir -p "${app_dir}/${app_name}"
if [[ -f "$file" ]]; then

# INCORRECT
touch $log
chown $user:$user $config_dir
mkdir -p $app_dir/$app_name
```

## Swizzin API Functions

| Function                   | Purpose              | Example                            |
| -------------------------- | -------------------- | ---------------------------------- |
| `port <start> <end>`       | Allocate free port   | `app_port=$(port 10000 12000)`     |
| `apt_install <pkgs>`       | Install apt packages | `apt_install curl jq`              |
| `swizdb get/set`           | Persistent storage   | `swizdb set "myapp/port" "$port"`  |
| `_get_master_username`     | Primary Swizzin user | `user=$(_get_master_username)`     |
| `ask "question?" Y/N`      | Interactive prompt   | `if ask "Continue?" Y; then`       |
| `echo_progress_start/done` | Progress indicators  | `echo_progress_start "Installing"` |
| `echo_error/info/query`    | Status messages      | `echo_error "Failed to install"`   |

## Panel Registration Pattern

```bash
_load_panel_helper

panel_register_app \
    "$app_name" \
    "$app_port" \
    "$app_lockname" \
    "$app_baseurl" \
    "$app_icon_name" \
    "$app_icon_url"
```

## Lock File Convention

- Location: `/install/.<appname>.lock`
- Created at end of successful install
- Checked before install to prevent duplicate installs
- Removed during uninstall

```bash
# Check if already installed
if [[ -f "/install/.${app_lockname}.lock" ]]; then
    echo_error "${app_name} is already installed"
    exit 1
fi

# Create lock file at end
touch "/install/.${app_lockname}.lock"
```

## Nginx Configuration Patterns

### Subfolder Mode (location block)

```nginx
location /${app_name}/ {
    proxy_pass http://127.0.0.1:${app_port}/;
    proxy_http_version 1.1;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

### Subdomain Mode (dedicated vhost)

- File: `/etc/nginx/sites-available/<appname>`
- Symlink: `/etc/nginx/sites-enabled/<appname>`
- Let's Encrypt via `box install letsencrypt`

## Systemd Service Pattern

```ini
[Unit]
Description=${app_pretty}
After=network-online.target

[Service]
User=${user}
Group=${user}
Type=simple
ExecStart=${app_binary} --config ${app_configdir}/config.yml
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

For Docker apps:

```ini
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/bin/docker compose -f /opt/${app_name}/docker-compose.yml up -d
ExecStop=/usr/bin/docker compose -f /opt/${app_name}/docker-compose.yml down
```

## Design Document Pattern

Before implementing complex features, create a design document:

- Location: `docs/plans/YYYY-MM-DD-<feature>-design.md`
- Include: Goal, Architecture, Tech Stack, Design Decisions table, File Layout, Tasks
- Reference: `> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans`

## Common Workflows

### Adding a New Binary App

1. Copy `templates/template-binary.sh` to `<appname>.sh`
2. Search/replace `myapp` → `yourapp`, `Myapp` → `Yourapp`
3. Customize:
   - Architecture mapping in `_install_<app>()`
   - Config file format
   - Systemd service options
   - Nginx location config
4. Add entry to `swizzin-app-info` APP_CONFIGS
5. Update backup/restore scripts
6. Update README.md and CLAUDE.md

### Adding Subdomain Support

1. Copy `templates/template-subdomain.sh`
2. Implement:
   - `_get_domain()` / `_prompt_domain()`
   - `_create_subdomain_vhost()`
   - `_revert_subdomain()`
3. Handle Organizr SSO exclusion if needed
4. Support `--subdomain` and `--subdomain --revert` flags

### Fixing a Bug

1. Identify root cause in existing script
2. Make minimal change to fix
3. Test on Swizzin system
4. Commit with `Fix <description>` message

## Testing Approach

- No automated tests; manual testing on Swizzin systems
- Test both install and remove paths
- Verify nginx config with `nginx -t`
- Check systemd status after install
- Confirm panel registration

## Environment Variable Bypass

Scripts support automation via environment variables:

```bash
# Skip interactive prompts
<APP>_DOMAIN="app.example.com"
<APP>_LE_HOSTNAME="app.example.com"
<APP>_LE_INTERACTIVE="yes"  # for CloudFlare DNS
<APP>_OWNER="username"
DN_API_KEY="xxxxx"  # for notifiarr
```

## Files Modified Together

Based on git history, these files change together:

| Primary File  | Co-changed Files                                         |
| ------------- | -------------------------------------------------------- |
| `lingarr.sh`  | `CLAUDE.md`, `README.md`, `templates/template-docker.sh` |
| `zurg.sh`     | `bootstrap/lib/apps.sh`, `decypharr.sh`                  |
| `*.sh` (new)  | `CLAUDE.md`, `README.md`, `swizzin-app-info`             |
| `organizr.sh` | Related subdomain scripts                                |

## Anti-Patterns to Avoid

1. **Don't skip panel helper** - Always include `_load_panel_helper()`
2. **Don't hardcode ports** - Use `port 10000 12000`
3. **Don't use `[ ]` for complex tests** - Prefer `[[ ]]`
4. **Don't forget lock files** - Both create and check
5. **Don't commit without testing remove** - Always verify cleanup
6. **Don't use interactive flags with git** - No `-i` for rebase/add
7. **Don't echo to communicate** - Use proper Swizzin functions
