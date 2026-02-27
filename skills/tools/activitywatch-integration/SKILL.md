---
name: activitywatch-integration
description: Comprehensive guide for ActivityWatch setup, configuration, watchers, integrations, API usage, and automation. Covers aw-qt, aw-watcher modules, aw-client libraries, aw-sync, data export, MCP server integration, and package managers. Use when working with ActivityWatch components, creating custom watchers, querying data, setting up sync, integrating with analytics dashboards, or using the ActivityWatch API.
---

# ActivityWatch Integration Guide

## Purpose

Complete reference for working with ActivityWatch, its ecosystem of tools, watchers, client libraries, and integrations. Provides guidance for setup, configuration, custom watcher development, data analysis, and automation.

## When to Use

Activate this skill when:
- Setting up or configuring ActivityWatch
- Creating custom watchers or integrations
- Querying ActivityWatch data via API
- Integrating with external tools (Grafana, InfluxDB, AI agents)
- Troubleshooting ActivityWatch components
- Working with aw-client libraries
- Setting up multi-device sync
- Analyzing time-tracking data

## Table of Contents

- [Current Installation](#current-installation)
- [Core Components](#core-components)
- [Quick Start Guide](#quick-start-guide)
- [Tool Reference](#tool-reference)
- [Common Tasks](#common-tasks)
- [Troubleshooting](#troubleshooting)
- [Resources](#resources)

---

## Current Installation

**Status:** ✅ Installed and Running

**Details:**
- **Location:** `/Applications/ActivityWatch.app`
- **Version:** v0.13.2
- **Server:** http://localhost:5600
- **Installation Method:** Homebrew (`brew install --cask activitywatch`)

**Active Components:**
- `aw-qt` - System tray application (menu bar)
- `aw-server` - API server
- `aw-watcher-afk` - AFK detection
- `aw-watcher-window` - Window tracking

**Data Location:** `~/Library/Application Support/activitywatch/`

**Access:**
- Web UI: http://localhost:5600
- API: http://localhost:5600/api/0/

---

## Core Components

### 1. aw-qt (Desktop Application)

The main desktop tray app that manages everything.

**Key Features:**
- Autostart integration
- Watcher toggling
- Real-time activity views
- Settings management

**Access:** Menu bar icon (look for ActivityWatch icon)

### 2. aw-server (API Server)

REST API server for data storage and querying.

**Endpoints:**
- `/api/0/info` - Server information
- `/api/0/buckets` - List data buckets
- `/api/0/buckets/{id}/events` - Get/create events
- `/api/0/query` - Execute AQL queries

**See:** [API_REFERENCE.md](API_REFERENCE.md) for complete API documentation

### 3. aw-watcher Modules

Watchers are "agents" that track specific activities:

**Built-in:**
- `aw-watcher-window` - Active window/app tracking
- `aw-watcher-afk` - Keyboard/mouse activity (AFK detection)

**Optional:**
- `aw-watcher-web` - Browser tabs (install browser extension)
- `aw-watcher-vscode` - VS Code activity
- `aw-watcher-vim` - Vim/Neovim sessions

**Creating Custom Watchers:** See "Custom Watcher Development" section below

### 4. aw-client Libraries

Official SDKs for programmatic access:

**Available:**
- Python: `pip install aw-client`
- JavaScript: `aw-client-js`
- Go: `aw-client-go`
- Rust: `aw-client-rust`

**Use Cases:**
- Query and analyze data
- Create custom integrations
- Build automation scripts
- Export data to other formats

**See:** [API_REFERENCE.md](API_REFERENCE.md) for Python client examples

---

## Quick Start Guide

### Verify Installation

```bash
# Check if running
ps aux | grep activitywatch

# Test API
curl http://localhost:5600/api/0/info

# List buckets
curl http://localhost:5600/api/0/buckets
```

### Access Web UI

1. Open http://localhost:5600 in browser
2. View today's activity on Activity page
3. Configure settings in Settings page

### Enable Browser Extension

1. Visit https://activitywatch.net/downloads/
2. Install extension for your browser (Chrome, Firefox, Edge)
3. Extension will auto-connect to localhost:5600

### Python Client Setup

```bash
# Install
pip install aw-client

# Test
python3 -c "from aw_client import ActivityWatchClient; print(ActivityWatchClient().get_buckets())"
```

---

## Tool Reference

### aw-sync (Multi-Device Sync)

**Purpose:** Sync data across multiple devices via shared folder

**Installation:**
```bash
git clone https://github.com/ActivityWatch/aw-sync.git
cd aw-sync
pip install .
```

**Usage:**
```bash
# Sync to Dropbox
aw-sync --folder ~/Dropbox/ActivityWatch

# Automated sync (add to cron)
0 * * * * aw-sync --folder ~/sync/aw
```

**Cloud Options:**
- Dropbox
- Google Drive (via rclone)
- Syncthing (recommended for privacy)

### activitywatch-exporter (Analytics Integration)

**Purpose:** Export data to InfluxDB for Grafana dashboards

**Installation:**
```bash
pip install activitywatch-exporter
```

**Usage:**
```bash
activitywatch-exporter \
  --aw-url http://localhost:5600 \
  --influx-url http://localhost:8086 \
  --influx-db activitywatch
```

**See:** [INTEGRATIONS.md](INTEGRATIONS.md) for Grafana setup

### ActivityWatch MCP Server (AI Integration)

**Purpose:** AI agent interface for LLMs (Claude, Cursor)

**Installation:**
```bash
git clone https://github.com/Auriora/activitywatch-mcp.git
cd activitywatch-mcp
npm install
```

**Configuration for Claude Code:**
```json
{
  "mcpServers": {
    "activitywatch": {
      "command": "node",
      "args": ["/path/to/activitywatch-mcp/dist/index.js"],
      "env": {
        "AW_URL": "http://localhost:5600"
      }
    }
  }
}
```

**Example Queries:**
- "Summarize my coding hours this week"
- "What websites did I visit most yesterday?"
- "Show my most productive hours"

### Codewatch (Developer Focus)

**Purpose:** IDE-specific tracking with Git integration

**Features:**
- Multi-editor support
- Git commit analysis
- Project-based tracking
- Language statistics

**Installation:** Download from GitHub releases

---

## Common Tasks

### Query Data with Python

```python
from aw_client import ActivityWatchClient
from datetime import datetime, timedelta

client = ActivityWatchClient()

# Get today's window events
today = datetime.now().replace(hour=0, minute=0, second=0)
bucket_id = f"aw-watcher-window_{client.client_hostname}"
events = client.get_events(bucket_id, start=today)

# Calculate time per app
by_app = {}
for e in events:
    app = e["data"].get("app", "Unknown")
    by_app[app] = by_app.get(app, 0) + e["duration"]

# Print results
for app, duration in sorted(by_app.items(), key=lambda x: x[1], reverse=True):
    print(f"{app}: {duration/3600:.1f}h")
```

**See:** [API_REFERENCE.md](API_REFERENCE.md) for more examples

### Create Custom Watcher

```python
from aw_client import ActivityWatchClient
from datetime import datetime
import time

# Initialize
client = ActivityWatchClient("my-custom-watcher")
bucket_id = f"{client.client_name}_{client.client_hostname}"

# Create bucket
client.create_bucket(bucket_id, event_type="custom.activity")

# Send events
while True:
    event = {
        "timestamp": datetime.now(),
        "duration": 0,
        "data": {
            "label": "Custom activity",
            "value": get_custom_data()
        }
    }
    client.heartbeat(bucket_id, event, pulsetime=60)
    time.sleep(30)
```

### Export Data to CSV

```python
import pandas as pd
from aw_client import ActivityWatchClient

client = ActivityWatchClient()
bucket_id = f"aw-watcher-window_{client.client_hostname}"
events = client.get_events(bucket_id, limit=10000)

# Convert to DataFrame
df = pd.DataFrame([
    {
        "timestamp": e["timestamp"],
        "duration": e["duration"],
        "app": e["data"].get("app"),
        "title": e["data"].get("title")
    }
    for e in events
])

# Export
df.to_csv("activitywatch_export.csv", index=False)
print(f"✅ Exported {len(df)} events")
```

### Backup Data

```bash
#!/bin/bash
# backup-activitywatch.sh

DATE=$(date +%Y%m%d)
BACKUP_DIR=~/backups/activitywatch
mkdir -p $BACKUP_DIR

# Stop server
pkill -f aw-server

# Backup
tar -czf $BACKUP_DIR/aw-backup-$DATE.tar.gz \
  ~/Library/Application\ Support/activitywatch/

# Restart
/Applications/ActivityWatch.app/Contents/MacOS/aw-server &

echo "✅ Backup created: aw-backup-$DATE.tar.gz"
```

### Set Up Automation

**Daily Summary (cron):**
```bash
# Run at 6 PM daily
0 18 * * * /usr/bin/python3 /path/to/daily_summary.py
```

**Weekly Report (cron):**
```bash
# Run Sunday at 8 PM
0 20 * * 0 /usr/bin/python3 /path/to/weekly_report.py
```

**See:** [INTEGRATIONS.md](INTEGRATIONS.md) for Slack, email, and calendar integration examples

---

## Troubleshooting

### Server Not Starting

```bash
# Check if port in use
lsof -i :5600

# Kill existing process
pkill -f aw-server

# Start manually with debug
/Applications/ActivityWatch.app/Contents/MacOS/aw-server --verbose

# Check logs
tail -f ~/Library/Application\ Support/activitywatch/aw-server/aw-server.log
```

### Watchers Not Recording

```bash
# Check watcher status
curl http://localhost:5600/api/0/buckets

# Restart watchers
pkill -f aw-watcher
/Applications/ActivityWatch.app/Contents/MacOS/aw-watcher-afk &
/Applications/ActivityWatch.app/Contents/MacOS/aw-watcher-window &
```

### Database Issues

```bash
# Location
cd ~/Library/Application\ Support/activitywatch/aw-server/

# Check integrity
sqlite3 peewee-sqlite.v2.db "PRAGMA integrity_check;"

# Vacuum (reduce size)
sqlite3 peewee-sqlite.v2.db "VACUUM;"
```

### Permission Errors

```bash
# Fix permissions
chmod -R 755 ~/Library/Application\ Support/activitywatch/
```

---

## Configuration

### Server Settings

**Location:** `~/.config/activitywatch/aw-server/config.toml`

```toml
[server]
host = "127.0.0.1"  # Localhost only (recommended)
port = 5600         # Default port
cors_origins = "*"  # CORS settings
```

### Watcher Settings

**aw-watcher-afk:**
```toml
# ~/.config/activitywatch/aw-watcher-afk/config.toml
[aw-watcher-afk]
timeout = 180      # AFK timeout (seconds)
poll_time = 5      # Check interval
```

**aw-watcher-window:**
```toml
# ~/.config/activitywatch/aw-watcher-window/config.toml
[aw-watcher-window]
poll_time = 1           # Update interval
exclude_title = false   # Privacy: hide window titles
```

### Web UI Settings

Access via: http://localhost:5600/#/settings

**Categories:**
- Create rules to categorize activities
- Pattern matching on app, title, URL

**Example:**
```json
{
  "name": "Programming",
  "rule": {
    "$type": "regex",
    "regex": "VSCode|Terminal|vim"
  }
}
```

---

## Resources

### Official Documentation
- **Main Docs:** https://docs.activitywatch.net/
- **API Reference:** https://docs.activitywatch.net/en/latest/api/
- **Writing Watchers:** https://docs.activitywatch.net/en/latest/examples/writing-watchers.html

### Community
- **Forum:** https://forum.activitywatch.net/
- **Discord:** https://discord.gg/vDskV9q
- **GitHub:** https://github.com/ActivityWatch/activitywatch

### Related Projects
- **Browser Extension:** https://github.com/ActivityWatch/aw-watcher-web
- **VS Code Extension:** Marketplace → "ActivityWatch"
- **MCP Server:** https://github.com/Auriora/activitywatch-mcp

### Reference Files
- **[API_REFERENCE.md](API_REFERENCE.md)** - Complete REST API and AQL documentation
- **[INTEGRATIONS.md](INTEGRATIONS.md)** - Integration examples (Slack, Grafana, Sheets, etc.)

---

**Installation Status:** ✅ v0.13.2 via Homebrew
**Server:** ✅ http://localhost:5600
**Watchers:** ✅ afk, window active
