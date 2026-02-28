---
name: deployment-and-ops
description: Deploy and operate the vehicle insurance data analysis platform. Use when user asks about local development setup, production deployment, server configuration, build process, service management, or troubleshooting deployment issues. Focuses on the project's actual simple deployment model using start_server.sh, not complex enterprise setups.
allowed-tools: Read, Bash, Grep, Glob
---

# Deployment and Operations Guide

You are assisting with deploying and operating the vehicle insurance data analysis platform. This project uses a **simple deployment model** suitable for internal teams and small-scale production.

## When to Use This Skill

Activate this skill when the user needs help with:
- Setting up local development environment
- Running the application (`start_server.sh`)
- Building frontend for production
- Deploying to a server
- Managing services (starting/stopping)
- Troubleshooting deployment issues
- Viewing logs and monitoring

## Project Deployment Model

**Current Approach**: Simple, single-server deployment
- **NOT using**: Docker, Kubernetes, complex CI/CD
- **NOT using**: Gunicorn/uWSGI in production yet
- **Currently using**: Direct Python execution via `start_server.sh`

This is appropriate for:
- Internal business tools
- Team size: < 50 users
- Data refreshed daily (not real-time)

## Quick Start (Local Development)

### Prerequisites Check

Guide the user to verify:

```bash
# Check Python (3.11+ required)
python3 --version

# Check Node.js (18+ recommended)
node -v

# Check if in correct directory
pwd  # Should show /path/to/签单日报dayreport
```

### Option 1: One-Command Start (Recommended)

```bash
# Start backend with the provided script
./start_server.sh

# Output:
# ========================================
#    每日签单平台趋势分析系统
# ========================================
# [1/3] 检查Python环境...
# [2/3] 检查依赖包...
# [3/3] 启动API服务器...
# 服务器地址: http://localhost:5000
```

**What this script does**:
1. Checks Python environment (python3 or python)
2. Auto-installs dependencies if missing (Flask, Pandas, etc.)
3. Starts Flask backend on port 5000
4. Serves static HTML from `/static/index.html`

### Option 2: Manual Start (for development)

```bash
# Terminal 1: Start backend
cd backend
python3 api_server.py

# Terminal 2: Start frontend dev server (if doing frontend dev)
cd frontend
npm install  # First time only
npm run dev  # Starts Vite on http://localhost:5173
```

## Environment Setup Details

### Python Environment

**Install dependencies**:
```bash
# From project root
pip3 install -r requirements.txt

# Requirements (from requirements.txt):
# - flask==3.0.0
# - flask-cors==4.0.0
# - numpy>=2.2,<3
# - pandas>=2.2.3,<3
# - openpyxl==3.1.2
```

**Optional: Use virtual environment**:
```bash
# Create venv (recommended for clean dependency management)
python3 -m venv .venv

# Activate
source .venv/bin/activate  # macOS/Linux
# .venv\Scripts\activate   # Windows

# Install deps
pip install -r requirements.txt
```

### Node.js Environment (Frontend Development Only)

**Only needed if user wants to modify Vue components**:

```bash
cd frontend

# Install dependencies
npm install

# Available commands
npm run dev      # Start dev server (http://localhost:5173)
npm run build    # Build for production (outputs to dist/)
npm run preview  # Preview production build
npm run lint     # Run ESLint
```

## Production Build

### Step 1: Build Frontend

```bash
cd frontend

# Production build
npm run build

# Output:
# vite v5.0.10 building for production...
# ✓ 104 modules transformed.
# dist/index.html                   0.48 kB
# dist/assets/index-xxx.js         80.68 kB │ gzip: 31.98 kB
# dist/assets/index-xxx.css        14.05 kB │ gzip:  2.86 kB
```

**Build artifacts**: `frontend/dist/`
- `index.html` - Entry point
- `assets/` - Bundled JS/CSS with content hashes

### Step 2: Deploy to Server

**Simple deployment (current method)**:

```bash
# 1. Copy entire project to server
scp -r /path/to/签单日报dayreport user@server:/opt/dayreport/

# 2. SSH to server
ssh user@server

# 3. Install Python deps
cd /opt/dayreport
pip3 install -r requirements.txt

# 4. Start service
./start_server.sh

# Or run in background
nohup ./start_server.sh > app.log 2>&1 &
```

### Step 3: Access Application

```
# If using start_server.sh (default):
http://server-ip:5000/static/index.html

# If using frontend dev server:
http://server-ip:5173
```

## Service Management

### Check if Services Running

```bash
# Check backend (port 5000)
lsof -i :5000
# Or
ps aux | grep api_server

# Check frontend dev server (port 5173)
lsof -i :5173
```

### Start/Stop Services

```bash
# Stop backend
pkill -f api_server

# Stop frontend dev server
# (Ctrl+C in terminal, or)
pkill -f vite

# Restart backend
cd backend && python3 api_server.py &

# Restart frontend
cd frontend && npm run dev &
```

### Background Execution

```bash
# Run backend in background
nohup python3 backend/api_server.py > backend.log 2>&1 &

# Get process ID
echo $!  # Save this PID

# Stop later
kill <PID>
```

## Log Management

### Backend Logs

**Location**: `backend/backend.log`

```bash
# View real-time logs
tail -f backend/backend.log

# View last 50 lines
tail -n 50 backend/backend.log

# Search for errors
grep -i "error" backend/backend.log

# View logs with timestamps
tail -f backend/backend.log | while read line; do echo "$(date): $line"; done
```

### Frontend Logs

**Browser console**: Open DevTools (F12) → Console tab

**Build logs**: Terminal output during `npm run build`

## Common Deployment Issues

### Issue 1: Port Already in Use

**Symptom**:
```
OSError: [Errno 48] Address already in use
```

**Solution**:
```bash
# Find and kill process using port 5000
lsof -i :5000
kill -9 <PID>

# Or change port in api_server.py
# app.run(host='0.0.0.0', port=5001)  # Use different port
```

### Issue 2: Dependencies Not Found

**Symptom**:
```
ModuleNotFoundError: No module named 'flask'
```

**Solution**:
```bash
# Verify Python environment
which python3
python3 -m pip list

# Reinstall dependencies
pip3 install -r requirements.txt

# If still failing, check if using correct Python
python3 -c "import flask; print(flask.__version__)"
```

### Issue 3: Permission Denied on start_server.sh

**Symptom**:
```
-bash: ./start_server.sh: Permission denied
```

**Solution**:
```bash
# Add execute permission
chmod +x start_server.sh

# Then run
./start_server.sh
```

### Issue 4: CSV Files Not Found

**Symptom**:
```
FileNotFoundError: 车险清单_2025年10-11月_合并.csv not found
```

**Solution**:
```bash
# Check if file exists
ls -la *.csv

# Check data directory
ls -la data/

# Verify file paths in data_processor.py match actual locations
```

### Issue 5: Frontend Build Fails

**Symptom**:
```
npm ERR! code ENOENT
```

**Solution**:
```bash
# Clean install
rm -rf node_modules package-lock.json
npm install

# If disk space issue
df -h  # Check available space

# If memory issue
NODE_OPTIONS=--max-old-space-size=4096 npm run build
```

## Performance Monitoring

### Check Resource Usage

```bash
# CPU and memory (macOS)
top -o CPU

# Specific process
ps aux | grep python3 | grep api_server

# Disk usage
df -h
du -sh /path/to/签单日报dayreport/*
```

### API Performance

```bash
# Test API response time
time curl http://localhost:5000/api/latest-date

# Multiple requests benchmark
for i in {1..10}; do
  time curl -s http://localhost:5000/api/kpi?period=day > /dev/null
done
```

## Data Backup

**Important files to backup**:

```bash
# 1. Merged CSV data
车险清单_2025年10-11月_合并.csv

# 2. Staff mapping
业务员机构团队归属.json

# 3. Raw Excel files (optional)
data/*.xlsx

# Backup command
tar -czf backup-$(date +%Y%m%d).tar.gz \
  车险清单_2025年10-11月_合并.csv \
  业务员机构团队归属.json \
  data/

# Restore
tar -xzf backup-20250108.tar.gz
```

## Advanced Deployment (Future)

**Note**: These are NOT currently implemented but can be added later:

### Option A: Nginx Reverse Proxy

```nginx
# /etc/nginx/sites-available/dayreport
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
    }
}
```

### Option B: Systemd Service

```ini
# /etc/systemd/system/dayreport.service
[Unit]
Description=Dayreport API Service

[Service]
Type=simple
WorkingDirectory=/opt/dayreport
ExecStart=/usr/bin/python3 backend/api_server.py
Restart=always

[Install]
WantedBy=multi-user.target
```

**Enable**:
```bash
sudo systemctl daemon-reload
sudo systemctl enable dayreport
sudo systemctl start dayreport
```

For detailed advanced deployment, refer to [ADVANCED_DEPLOYMENT.md](./ADVANCED_DEPLOYMENT.md).

## Environment Variables (Optional)

**Create `.env` file** (if needed for configuration):

```bash
# .env
FLASK_ENV=production
FLASK_DEBUG=False
DATA_DIR=/opt/dayreport/data
PORT=5000
```

**Load in Python**:
```python
# backend/api_server.py
import os
from dotenv import load_dotenv

load_dotenv()
PORT = int(os.getenv('PORT', 5000))
```

## Deployment Checklist

Before deploying to production:

- [ ] Frontend built (`npm run build`)
- [ ] Dependencies installed (`pip install -r requirements.txt`)
- [ ] CSV data files copied to server
- [ ] Mapping JSON file present
- [ ] Port 5000 accessible
- [ ] Logs directory writable
- [ ] Tested API endpoints (`curl localhost:5000/api/latest-date`)
- [ ] Browser can access UI

## Quick Reference

### Essential Commands

```bash
# Start
./start_server.sh

# Stop
pkill -f api_server

# Logs
tail -f backend/backend.log

# Build frontend
cd frontend && npm run build

# Check status
lsof -i :5000
ps aux | grep api_server
```

### File Locations

```
项目/
├── start_server.sh          # Main startup script
├── requirements.txt         # Python dependencies
├── backend/
│   ├── api_server.py       # Flask app (runs on :5000)
│   ├── data_processor.py   # Pandas logic
│   └── backend.log         # Runtime logs
├── frontend/
│   ├── package.json        # Node dependencies
│   └── dist/               # Built artifacts (after npm run build)
├── data/                   # Excel source files
└── *.csv                   # Processed CSV data
```

## Summary

This skill covers the **actual deployment model** used by this project:
- Simple startup via `start_server.sh`
- Direct Python execution (no container orchestration)
- Suitable for internal tools and small teams

**Key principle**: Start simple, scale when needed. The current deployment is appropriate for the project's scope.

For enterprise-grade deployment patterns (Docker, K8s, load balancing), refer to advanced guides only if project requirements change.
