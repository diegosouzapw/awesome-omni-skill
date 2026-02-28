---
name: clawdbot-dashboard
description: A beautiful, feature-rich dashboard for Clawdbot that displays workspace stats, memory, tasks, goals, analytics, and installed skills. Works with any Clawdbot installation - no database needed, all data from workspace files.
category: dashboard
version: 1.0.0
author: thibautrey
homepage: https://github.com/thibautrey/clawdbot-dashboard
repository: https://github.com/thibautrey/clawdbot-dashboard
license: MIT
keywords:
  - dashboard
  - analytics
  - skills
  - tasks
  - memory
  - ui
  - workspace
tags:
  - visualization
  - productivity
  - stats
icon: 📊
metadata:
  clawdbot:
    emoji: 📊
    description: "Modern dashboard for Clawdbot workspace with analytics, skills browser, and more"
    requires:
      bins: ["node", "npm"]
    install:
      - id: npm-install
        kind: shell
        command: "cd clawdbot-dashboard && npm install && cd frontend && npm install && cd .."
        label: "Install dependencies (npm)"
      - id: start-dashboard
        kind: shell
        command: "cd clawdbot-dashboard && npm start"
        label: "Start dashboard (backend + frontend)"
---

# Clawdbot Dashboard

> A modern, feature-rich dashboard for Clawdbot that visualizes your workspace, skills, tasks, memory, and analytics.

## ✨ Features

### 📊 Dashboard
- Real-time workspace statistics
- Memory size tracking
- Daily logs counter
- Installed skills overview
- Interactive tips carousel

### 📈 Analytics
- Memory growth charts
- Daily logs analytics
- Workspace health status
- Data sources overview

### 🛠️ Skills Browser
- Browse all installed skills
- Filter by category
- View full SKILL.md content
- Skill metadata display

### ✅ Tasks & Crons
- Display Heartbeat checklist
- Show all cron jobs
- Schedule information
- Status tracking

### 🎯 Goals
- Track achievements
- Milestone progress
- Goal visualization

### 💭 Memory
- Browse MEMORY.md
- Full Markdown rendering
- Rich formatting support

## 🚀 Quick Start

```bash
# Install
clawdbot skills install clawdbot-dashboard

# Start
cd ~/.clawdbot/skills/clawdbot-dashboard
npm start

# Open browser
open http://localhost:5173
```

## 🏗️ Architecture

**Backend (Node.js + Express):**
- Port: 5174
- Reads from Clawdbot workspace (`/Users/[user]/clawd/`)
- REST API for all data
- 20-60s caching for performance

**Frontend (React + Vite):**
- Port: 5173
- Modern UI with Tailwind CSS
- Dark mode support
- Real-time updates

**Data Sources (Clawd-native):**
- `MEMORY.md` — Long-term memory
- `HEARTBEAT.md` — Daily checklist
- `memory/YYYY-MM-DD.md` — Daily logs
- `skills/*/SKILL.md` — Skill metadata
- Cron jobs via `clawdbot cron list`

## 📦 What's Included

```
clawdbot-dashboard/
├── backend/
│   ├── index.js          (Express server + API)
│   └── cache.js          (Cache system)
├── frontend/
│   ├── src/
│   │   ├── App.tsx       (Main layout + navigation)
│   │   ├── pages/        (Dashboard, Analytics, Skills, Tasks, etc.)
│   │   ├── components/   (Reusable UI components)
│   │   └── services/     (API client)
│   └── package.json
├── package.json
├── SKILL.md              (This file)
├── README.md
└── QUICKSTART.md
```

## 🎨 Pages

- **Dashboard** — Overview with stats & tips
- **Analytics** — Growth charts & workspace health
- **Skills Browser** — Browse installed skills
- **Tasks** — Heartbeat checklist + cron jobs
- **Goals** — Achievements & milestones
- **Memory** — Markdown memory viewer

## ⚙️ Configuration

No configuration needed! The dashboard auto-detects your Clawdbot workspace.

### Customization (Optional)

Edit `frontend/src/App.tsx` to:
- Change sidebar colors
- Add new pages
- Customize theme
- Modify refresh rates

## 🔧 Development

```bash
# Install dependencies
cd clawdbot-dashboard
npm install
cd frontend && npm install && cd ..

# Start dev servers
npm run dev

# Backend: http://localhost:5174
# Frontend: http://localhost:5173
```

## 📊 API Endpoints

- `GET /api/stats` — Workspace statistics
- `GET /api/memory` — MEMORY.md content
- `GET /api/tasks` — Heartbeat + crons
- `GET /api/goals` — Goals data
- `GET /api/skills` — Installed skills
- `GET /api/skills/:id/content` — Skill SKILL.md content
- `GET /api/tips` — Tips carousel

## 🌙 Dark Mode

Click the theme toggle in the sidebar footer. Preference saved to browser.

## 🚀 Performance

- **Cache system:** 20-60s refresh per endpoint
- **Lazy loading:** Pages load on demand
- **Minimal deps:** Only React, Express, Tailwind
- **Zero external data:** All local workspace files

## 📝 License

MIT — Use freely, modify as needed

## 🤝 Contributing

Ideas or improvements? Open an issue or PR on GitHub!

---

**Made for Clawdbot.** Works with any workspace. Zero setup needed. 🤖
