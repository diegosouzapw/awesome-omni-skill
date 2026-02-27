---
name: jarvis-autonomous
description: Fully autonomous AI agent with self-improvement, GitHub automation, and Telegram control. Creates projects, learns continuously, and manages itself proactively.
metadata: {"openclaw":{"emoji":"🤖","requires":{"bins":["gh","git"],"env":["TELEGRAM_BOT_TOKEN"]},"homepage":"https://github.com/apu242007/jarvis-autonomous"}}
---

# JARVIS Autonomous Agent

A fully autonomous AI agent that creates value without constant supervision. Self-improves, manages projects, and operates via Telegram.

## Features

### 🧠 Complete Autonomy
- Continuous self-improvement via learning system
- Automatic hourly heartbeats
- Daily knowledge backup to GitHub
- Proactive task management

### 📱 Telegram Control
Six powerful commands:
- `/create <name>` - Create complete GitHub projects
- `/backup` - Manual backup of learnings
- `/estado` - Security system monitoring
- `/heartbeat` - Manual heartbeat execution
- `/learnings` - View recent learnings
- `/help` - Command list

### 🔄 Self-Improvement Loop
- Categorized learning system (best_practice, error_resolution, knowledge_gap)
- Automatic pattern promotion
- Persistent memory across sessions
- Error tracking and auto-correction

### 🚀 Automatic Project Generation
- Complete GitHub repository creation
- Base structure (README, .gitignore)
- Automatic push
- Project logging

## Architecture
```
jarvis-autonomous/
├── SOUL.md           # Identity and principles
├── USER.md           # User profile
├── AGENTS.md         # Operations manual
├── MEMORY.md         # Long-term memory
├── TOOLS.md          # Tool configuration
├── .learnings/       # Learning system
│   ├── LEARNINGS.md
│   ├── ERRORS.md
│   └── FEATURE_REQUESTS.md
├── memory/           # Daily logs
├── scripts/          # Automation scripts
│   ├── heartbeat.sh
│   ├── self-improve.sh
│   ├── github-auto.sh
│   └── create-project.sh
└── notes/            # Proactive ideas
```

## Setup

### Prerequisites
- GitHub CLI authenticated: `gh auth login`
- Telegram bot token
- Python 3.8+ with asyncio

### Installation
```bash
# Clone the workspace structure
mkdir -p ~/jarvis-autonomous
cd ~/jarvis-autonomous

# Create base files
cat > SOUL.md << 'SOULEOF'
# JARVIS - Autonomous AI Agent

## Identity
Autonomous AI agent specialized in:
- Self-directed project creation
- GitHub automation and CI/CD
- Proactive task generation
- Self-learning and adaptation

## Core Principles
1. Autonomy First - Anticipate, don't wait
2. Build, Don't Ask - Create solutions proactively
3. Learn Continuously - Every interaction improves
4. Fail Forward - Errors are learning data
5. Document Everything - Knowledge compounds
SOULEOF

# Create learnings directory
mkdir -p .learnings memory notes/areas scripts

# Copy scripts from skill folder
cp {baseDir}/scripts/*.sh ./scripts/
chmod +x ./scripts/*.sh

# Setup cron jobs
(crontab -l 2>/dev/null; echo "0 * * * * $HOME/jarvis-autonomous/scripts/heartbeat.sh") | crontab -
```

### Telegram Bot Setup

1. Create bot via @BotFather
2. Get token
3. Configure in your bot code:
```python
import sys
sys.path.append('/path/to/jarvis-autonomous')
from telegram_integration import JarvisTelegram

# Initialize
bot = JarvisTelegram(token="YOUR_TOKEN")
bot.run()
```

## Usage Examples

### Create a Project
```
/create my-awesome-app "An AI-powered productivity tool"
```

### Manual Backup
```
/backup
```

### Check System Status
```
/estado
```

## Configuration

Environment variables:
- `JARVIS_WORKSPACE` - Path to workspace (default: ~/jarvis-autonomous)
- `TELEGRAM_BOT_TOKEN` - Your Telegram bot token
- `GH_REPO` - GitHub repo for backups

## Tech Stack

- Python + AsyncIO
- Telegram Bot API (telebot)
- GitHub CLI (gh)
- Cron + Bash scripts
- ClawdHub skills (autonomous-feature-planner, self-improving-agent)

## Integrations

Works with ClawdHub skills:
- `autonomous-feature-planner` - Plan features autonomously
- `self-improving-agent` - Enhanced self-improvement
- `agent-orchestrator` - Multi-agent coordination
- `github-action-gen` - CI/CD generation

## Use Cases

1. **Autonomous Development** - Create and manage projects hands-free
2. **Knowledge Preservation** - Auto-backup learnings
3. **Security Monitoring** - Detect and report anomalies
4. **Mobile Management** - Full control via smartphone

## Metrics

- 24/7 uptime with auto-restart
- 6 Telegram commands
- 2+ active cron jobs
- Continuous learning and improvement

## License

MIT - Use, modify, distribute freely

## Created By

Mariano (@apu242007)
Built with Claude AI, Python, and determination.

---

*"An agent that doesn't just execute commands—it anticipates your needs."*
