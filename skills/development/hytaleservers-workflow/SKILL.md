---
name: hytaleservers-workflow
description: "Standard workflow for HyTaleServers.tech development"
---

# HyTaleServers Workflow

## Project Overview

**Name:** Hytaleservers.tech  
**Description:** Мониторинг и топ-лист серверов Hytale  
**Stack:** Next.js 16 + Supabase + Tailwind CSS + shadcn/ui  
**Status:** Production ready  
**Version:** 0.1.0

## Development Flow

1. Create feature branch: `git checkout -b feature/xxx`
2. Make changes
3. Test locally: `npm run dev` (port 3000)
4. Commit: `git commit -m "feat: xxx"`
5. Push: `git push origin feature/xxx`
6. Create PR on GitHub
7. After merge → auto-deploy to VPS (port 3003)

## Available Commands

### Development
- `npm run dev` - dev server (port 3000)
- `npm run build` - production build
- `npm start` - production server
- `npm run lint` - linting

### Setup
- `npm run create-admin` - create admin user
- `npm run deploy` - deploy to VPS

## Project Structure

```
hytaleservers-tech/
├── app/                    # Next.js App Router
│   ├── api/                # API routes (11 endpoints)
│   │   ├── servers/        # Servers API
│   │   ├── categories/     # Categories API
│   │   ├── admin/          # Admin API
│   │   └── ...             # Other endpoints
│   ├── page.tsx            # Main page
│   ├── layout.tsx          # Root layout
│   └── globals.css         # Global styles
├── components/             # React components
│   ├── server/             # Server components
│   ├── shared/             # Shared components
│   └── ui/                 # shadcn/ui components
├── lib/                    # Utilities
│   ├── supabase/           # Supabase clients
│   │   ├── client.ts       # Browser client
│   │   └── server.ts       # Server client
│   └── utils/              # Helper functions
├── supabase/               # Database
│   ├── migrations/         # SQL migrations (001-003)
│   └── seed.sql            # Test data
├── scripts/                # Utility scripts
│   ├── create-admin.ts     # Create admin
│   ├── setup-storage.ts    # Setup storage
│   └── deploy-vps.sh       # Deploy script
└── .env                    # Environment variables (VPS)
```

## Database

### Supabase Configuration
- **URL:** https://ncxelqwplkhlhvbmdatf.supabase.co
- **Storage buckets:** server-logos, server-banners
- **Current setup:** Single Supabase project for both local and production

### Tables
- `servers` - Hytale servers
  - id (UUID, PK)
  - name, slug, ip, port
  - descriptions, logo_url, banner_url
  - owner_email, secret_key
  - status, current_players, max_players
  - rating, uptime_percentage, total_votes

- `categories` - Server categories
  - id (UUID, PK)
  - name, slug, icon, description

- `server_categories` - Junction table
  - server_id (FK servers.id)
  - category_id (FK categories.id)

- `votes` - Votes for servers
  - id (UUID, PK)
  - server_id (FK servers.id)
  - user_id (FK auth.users.id)
  - ip_address, user_agent, voted_at

- `admin_users` - Admin users
  - email (PK)
  - password_hash

### Migrations
1. `001_initial_schema.sql` - Main schema (servers, categories, votes)
2. `002_profiles.sql` - User profiles
3. `003_storage.sql` - Storage buckets (logos, banners)

### Seed Data
- 14 categories (survival, pvp, pve, rpg, creative, minigames, etc.)
- 3 test servers (HyWorld Survival, PvP Arena, Creative Build)

## Environment Variables

### Local Development (.env.local)
```bash
# Supabase Configuration
NEXT_PUBLIC_SUPABASE_URL=https://ncxelqwplkhlhvbmdatf.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=...
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_DEFAULT_KEY=sb_publishable_h4MJcTegWY2W93zWfGmkVQ_E2QSDnRH
SUPABASE_SERVICE_ROLE_KEY=...

# Site Configuration
NEXT_PUBLIC_SITE_URL=http://localhost:3000

# Analytics (empty for local)
NEXT_PUBLIC_GA_ID=
NEXT_PUBLIC_YANDEX_METRIKA_ID=

# Optional: Telegram Bot (empty for local)
TELEGRAM_BOT_TOKEN=
TELEGRAM_WEBHOOK_SECRET=

# Optional: Stripe (empty for local)
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=
```

### Production (.env on VPS)
```bash
# Same as local except:
NEXT_PUBLIC_SITE_URL=http://localhost:3003
# (or https://hytaleservers.tech in future)
```

### Port Configuration
- **Local (dev):** 3000 (Next.js default)
- **VPS (production):** 3003 (PM2 config)

## Deployment

### Automatic (GitHub Actions)
Push to `main` branch → auto-deploy to VPS

**Required GitHub Secrets:**
- `GH_SSH_PRIVATE_KEY` - Private SSH key
- `VPS_HOST` - VPS IP or domain
- `VPS_USER` - User (usually `root`)
- `VPS_PATH` - Project path (`/root/hytaleservers-tech`)

### Manual
```bash
./scripts/deploy-vps.sh

# Or manually
git pull origin main
npm install --production
npm run build
pm2 restart hytaleservers
```

## API Routes

### Public
- `GET /api/servers` - List servers (with pagination, filters, sorting)
  - Query params: page, limit, sort, category, search, status
- `GET /api/servers/[id]` - Server details
- `GET /api/categories` - List categories
- `GET /api/servers/test` - Test endpoint

### Auth Required
- `GET /api/my-servers` - User's servers
- `POST /api/upload` - Upload images (logo/banner)
- `GET /api/admin-moderation/[id]` - Moderation queue

### Admin
- `POST /api/admin/login` - Admin login (sets cookie)
- `POST /api/admin/logout` - Admin logout (clears cookie)

### Debug
- `GET /api/test` - API test
- `GET /api/debug` - Debug info
- `GET /api/supabase-test` - Supabase connection test

## Important Files

### Documentation
- `HISTORY.md` - **MANDATORY** - Project change history (iterations, decisions, problems) - UPDATE AFTER EVERY ITERATION
- `LOCAL_SETUP.md` - Local setup instructions for AI assistant
- `PROJECT_MAP.md` - Detailed project documentation (553 lines)
- `README.md` - Main README with setup instructions


### Configuration
- `package.json` - Dependencies and scripts
- `tsconfig.json` - TypeScript configuration
- `tailwind.config.ts` - Tailwind CSS configuration
- `next.config.ts` - Next.js configuration
- `ecosystem.config.js` - PM2 configuration
- `.gitignore` - Git ignore rules

### Scripts
- `scripts/create-admin.ts` - Create admin user (hardcoded password)
- `scripts/setup-storage.ts` - Storage setup instructions
- `scripts/fix-storage.ts` - Storage fixes
- `scripts/deploy-vps.sh` - Deploy script (bash)

## Tech Stack Details

### Frontend
- **Framework:** Next.js 16.1.1 (App Router)
- **UI Library:** React 19.2.3
- **Language:** TypeScript 5
- **Styling:**
  - Tailwind CSS 4
  - shadcn/ui (New York style)
  - Radix UI (Select, Slot)
  - Framer Motion 12.26.2 (animations)
  - class-variance-authority, clsx, tailwind-merge

### State & Data
- **State Management:** Zustand 5.0.10
- **Data Fetching:** TanStack Query 5.90.17
- **Validation:** Zod 4.3.5

### Backend & API
- **API Framework:** Next.js API Routes
- **Server-Side Rendering:** Next.js SSR with @supabase/ssr

### Database
- **Provider:** Supabase (PostgreSQL)
- **Client:** @supabase/supabase-js 2.90.1
- **SSR Client:** @supabase/ssr 0.8.0

### Deployment
- **Process Manager:** PM2 (production)
- **Port:** 3003 (VPS), 3000 (local)
- **Environment:** Linux (Ubuntu on VPS), Windows (local)

### Development
- **Package Manager:** npm
- **Build Tool:** Turbopack (Next.js 16)
- **Linting:** ESLint 9 + eslint-config-next

## Current Status

### ✅ Working
- Next.js 16 with App Router
- Supabase connection (ANON KEY)
- API endpoints (servers, categories, admin, etc.)
- PM2 production server (port 3003)
- Supabase Skills for OpenCode (5 skills)
- shadcn/ui components
- Tailwind CSS 4
- Middleware for admin routes
- Image upload to Supabase Storage
- Server moderation system

### ⚠️ Issues
- Service Role KEY needs update (currently using ANON KEY in ecosystem.config.js)
- No tests implemented
- Analytics not configured (GA, Yandex Metrika)
- Telegram Bot not configured
- Stripe not configured

### ❌ Not Implemented
- Vote system
- Server monitoring (ping)
- Admin panel UI
- Server registration form (public)
- Filters and search on main page
- Pagination
- SEO optimization (sitemap, robots.txt)

## Development Guidelines

### Code Style
- Follow existing patterns in components
- Use TypeScript for all new files
- Use shadcn/ui components for UI
- Follow Supabase SSR patterns for server components

### Database Changes
1. Create migration in `supabase/migrations/XXX_description.sql`
2. Apply migration via Supabase CLI or Dashboard
3. Update TypeScript types if needed
4. Test locally and on production

### Adding New Features
1. Create feature branch
2. Implement feature
3. Test locally
4. Commit with conventional commits (feat:, fix:, etc.)
5. Push and create PR
6. After merge → auto-deploy

### Adding New API Routes
- Place in `app/api/[resource]/route.ts`
- Use `lib/supabase/server.ts` for server client
- Return JSON responses
- Add error handling

### Documentation Management

**MANDATORY: Update HISTORY.md after every iteration**

After completing development work (feature, bug fix, refactoring, etc.), you MUST update `HISTORY.md`:

**When to update:**
- After any iteration of development (not just git commits)
- After debugging sessions
- After architectural decisions
- After testing and fixes

**How to update:**
1. Add new entry at the top of HISTORY.md with current date
2. Include ALL relevant sections:
   - 📝 Изменения кода - what was changed
   - 💡 Принятые решения - why decisions were made
   - 🐛 Проблемы и решения - issues encountered and fixed
   - ✅ Задачи (TODO) - completed and pending tasks with tags [bug]/[feature]/[refactor] and priority high/medium/low
   - 🔧 Технические заметки - important technical details
   - 🎯 Результат - outcome of the iteration

**Format:**
```markdown
## [ГГГГ-ММ-ДД] - Итерация N

### 📝 Изменения кода
- [Description]

### 💡 Принятые решения
- **[Decision]**: [Description]
  - Почему: [Reason]
  - Альтернативы: [Alternatives]

### 🐛 Проблемы и решения
- **[Problem]**: [Description]
  - Решение: [How fixed]
  - Время: [Time spent]

### ✅ Задачи (TODO)
- [x] [Task] - [tag] [priority]
- [ ] [Task] - [tag] [priority]

### 🔧 Технические заметки
- [Technical details]

### 🎯 Результат
[Outcome summary]
```

**CRITICAL RULES:**
- ALWAYS update HISTORY.md before moving to next task
- Include ALL changes, even minor ones
- Tag tasks with [bug]/[feature]/[refactor]
- Mark task priority as high/medium/low
- Document both successes and failures
- Keep entries machine-readable (AI needs to parse them)


## Testing Checklist

### Before Pushing
- [ ] Run `npm run lint` - check for linting errors
- [ ] Test locally with `npm run dev`
- [ ] Test API endpoints
- [ ] Check console for errors
- [ ] Verify Supabase connection
- [ ] **CRITICAL: Update HISTORY.md** with all changes, decisions, and problems


### After Deployment
- [ ] Check PM2 status: `pm2 status`
- [ ] Check PM2 logs: `pm2 logs hytaleservers`
- [ ] Test API endpoints on production
- [ ] Check UI in browser
- [ ] Verify Supabase connection

## Future Features

### Priority 1 - Critical
1. Update Service Role KEY
2. Implement vote system
3. Implement server monitoring (ping)
4. Add pagination to servers list
5. Add filters and search to main page

### Priority 2 - Important
6. Implement admin panel UI
7. Implement server registration form (public)
8. Setup analytics (GA, Yandex Metrika)
9. Add tests (Jest/Vitest)
10. Setup CI/CD for staging

### Priority 3 - Nice to Have
11. Implement Telegram Bot for notifications
12. Implement Stripe for premium features
13. SEO optimization (sitemap, robots.txt)
14. Performance monitoring
15. Error tracking (Sentry, etc.)

## Useful Commands

### Development
```bash
npm run dev              # Start dev server
npm run build            # Build for production
npm start                # Start production server
npm run lint             # Check linting
```

### Database
```bash
npm run create-admin     # Create admin user
# (Currently hardcoded password in script)
```

### Deployment
```bash
npm run deploy          # Deploy to VPS
# or
./scripts/deploy-vps.sh
```

### PM2 (on VPS)
```bash
pm2 status              # Check status
pm2 logs hytaleservers # View logs
pm2 restart hytaleservers
pm2 stop hytaleservers
pm2 start hytaleservers
```

### Git
```bash
git checkout -b feature/xxx
git add .
git commit -m "feat: xxx"
git push origin feature/xxx
```

## Troubleshooting

### Issue: Dev server not starting
- Check port 3000 is free: `lsof -i :3000` (Mac/Linux) or `netstat -ano | findstr :3000` (Windows)
- Check .env.local exists and is valid
- Check node_modules are installed

### Issue: Supabase connection error
- Check .env.local has correct Supabase URL and keys
- Check Supabase project is accessible
- Check console for error details

### Issue: Build fails
- Check for TypeScript errors
- Check for missing dependencies
- Check linting errors

### Issue: Deploy fails
- Check SSH connection to VPS
- Check PM2 is running on VPS
- Check deploy script permissions

## Resources

### Documentation
- [Next.js Docs](https://nextjs.org/docs)
- [Supabase Docs](https://supabase.com/docs)
- [shadcn/ui Docs](https://ui.shadcn.com)
- [Tailwind CSS Docs](https://tailwindcss.com/docs)

### Project
- Supabase Dashboard: https://supabase.com/dashboard/project/ncxelqwplkhlhvbmdatf
- Local URL: http://localhost:3000
- Production URL: http://localhost:3003 (or https://hytaleservers.tech)

### External
- GitHub Repository: (to be created)
- Issue Tracker: (GitHub Issues)
- Documentation: (PROJECT_MAP.md, README.md, LOCAL_SETUP.md)

---

**Last Updated:** January 17, 2026  
**Maintainer:** AI Assistant (OpenCode)  
**Status:** Active Development
