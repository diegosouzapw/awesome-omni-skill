---
name: konto-deploy
description: "Deploy and run Konto (personal finance dashboard) locally. Use when setting up a new Konto instance, troubleshooting installation, or helping users get started with Konto."
metadata:
  openclaw:
    emoji: "🦎"
    requires:
      bins: ["node", "npm", "openssl"]
    install:
      - id: clone
        kind: git
        repo: https://github.com/angelstreet/konto
        branch: main
        label: "Clone Konto repository"
      - id: deps
        kind: script
        cwd: "konto"
        run: "npm install"
        label: "Install dependencies"
      - id: env
        kind: script
        cwd: "konto"
        run: |
          if [ ! -f backend/.env ]; then
            cp .env.example backend/.env
            KEY=$(openssl rand -hex 32)
            sed -i "s/^DB_ENCRYPTION_KEY=$/DB_ENCRYPTION_KEY=$KEY/" backend/.env
            echo "Created backend/.env with generated encryption key"
          else
            echo "backend/.env already exists, skipping"
          fi
        label: "Configure environment"
---

# Konto — Local Deployment

Personal & professional finance dashboard. Bank sync, crypto, investments, budget, tax tools.

## Prerequisites

- Node.js 18+ and npm 9+
- `openssl` (for encryption key generation)

## Installation (3 commands)

```bash
git clone https://github.com/angelstreet/konto.git
cd konto
npm install
```

## Configuration

```bash
# Create env from template
cp .env.example backend/.env

# Generate and set encryption key
KEY=$(openssl rand -hex 32)
sed -i "s/^DB_ENCRYPTION_KEY=$/DB_ENCRYPTION_KEY=$KEY/" backend/.env
```

### Minimal config (works immediately)
Only `DB_ENCRYPTION_KEY` is required. Everything else is optional.

### Optional integrations
| Feature | Env vars | Sign up |
|---------|----------|---------|
| Bank sync | `POWENS_CLIENT_ID`, `POWENS_CLIENT_SECRET`, `POWENS_DOMAIN` | [powens.com](https://powens.com) |
| Production auth | `CLERK_SECRET_KEY`, `VITE_CLERK_PUBLISHABLE_KEY` | [clerk.com](https://clerk.com) |
| Coinbase | `COINBASE_CLIENT_ID`, `COINBASE_CLIENT_SECRET` | [developers.coinbase.com](https://developers.coinbase.com) |
| Google Drive | `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET` | [console.cloud.google.com](https://console.cloud.google.com) |

## Running

```bash
# Start both frontend + backend
npm run dev
```

- Frontend: http://localhost:3004/konto/
- Backend API: http://localhost:5004/api/
- Login: `user` / `user` (local dev, no Clerk needed)

## Sandbox / Demo Mode

Konto auto-seeds demo data for the default user:
- Bank accounts (checking, savings, investment)
- Crypto wallets (BTC, ETH, XRP)
- Investment positions (PEA, Assurance Vie, PER)
- 14 months of transaction history
- Real estate and vehicle assets

Just log in and explore — no external API keys needed.

## Building for Production

```bash
npm run build
# Frontend: serve frontend/dist/ as static files
# Backend: node backend/dist/index.js
```

### Vercel deployment
```bash
cd frontend && vercel
cd backend && vercel
```

## Ports

| Service | Port | URL |
|---------|------|-----|
| Frontend (dev) | 3004 | http://localhost:3004/konto/ |
| Backend API | 5004 | http://localhost:5004/api/ |

## Troubleshooting

| Problem | Solution |
|---------|----------|
| `ENCRYPTION_KEY` error | Run `openssl rand -hex 32` and set in `backend/.env` |
| Port 3004 in use | `lsof -i :3004` to find process, kill or change `VITE_DEV_PORT` |
| Port 5004 in use | Change `PORT` in `backend/.env` |
| Clerk errors locally | Leave `CLERK_SECRET_KEY` empty — local dev bypasses Clerk |
| Empty dashboard | Log in as `user/user` — demo data seeds on first backend start |
| Bank sync not working | Requires Powens API keys (optional for demo) |

## Tech Stack

| Layer | Tech |
|-------|------|
| Frontend | React 18 + TypeScript + Vite + Tailwind CSS + Recharts |
| Backend | Hono + TypeScript + Node.js |
| Database | SQLite (local) or Turso (cloud) |
| Auth | Clerk (optional) |

## API Endpoints (key ones)

| Endpoint | Description |
|----------|-------------|
| `GET /api/bank/accounts` | Bank accounts |
| `GET /api/investments` | Investment positions |
| `GET /api/transactions` | Transaction history |
| `GET /api/companies` | Companies (pro) |
| `GET /api/patrimoine/summary` | Net worth summary |
| `GET /api/preferences` | User preferences |

Full API docs: `docs/API.md` in the repo.
