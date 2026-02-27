---
name: new-saas-production
description: Production-ready SaaS boilerplate generator. Creates complete project with public landing pages, client dashboard, admin panel, T-Bank payment integration, and v12-style design system. FastAPI + Jinja2 + PostgreSQL stack.
triggers:
  - create new saas project
  - bootstrap saas application
  - create production project
  - new saas boilerplate
  - start new project
  - generate saas template
---

# New SaaS Production Skill

## Purpose

This skill generates a **complete production-ready SaaS boilerplate** with:
- **Public Site**: Landing pages, blog/news, legal pages (privacy, terms)
- **Client Dashboard**: User cabinet with billing, settings, core functionality
- **Admin Panel**: Content management, user management, analytics
- **Payment Integration**: T-Bank (Tinkoff) acquiring pre-configured
- **Design System**: Documatica v12.0 (via v12-style skill dependency)

## Stack

| Layer | Technology |
|-------|------------|
| Backend | FastAPI 0.109+ |
| Templates | Jinja2 |
| Database | PostgreSQL + SQLAlchemy 2.0 |
| Frontend | Bootstrap 5 + v12-style Design System |
| Icons | Iconify |
| Payment | T-Bank Acquiring API |
| Auth | JWT + bcrypt |
| Email | SMTP (Yandex/any) |
| Deploy | Docker + Nginx + SSL |

## Project Structure

```
{project_name}/
├── docker-compose.yml
├── .env.example
├── .github/
│   └── copilot-instructions.md
├── backend/
│   ├── Dockerfile
│   ├── requirements.txt
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py                 # FastAPI app, middleware, routers
│   │   ├── database.py             # SQLAlchemy setup
│   │   ├── models.py               # User, Payment, etc.
│   │   ├── core/
│   │   │   ├── config.py           # Settings from env
│   │   │   ├── templates.py        # Jinja2 setup
│   │   │   └── content.py          # YAML content loader
│   │   ├── api/
│   │   │   ├── auth.py             # Login, register, JWT
│   │   │   ├── payment.py          # T-Bank integration
│   │   │   └── billing.py          # Subscriptions, packages
│   │   ├── pages/                  # Public page routers
│   │   │   ├── __init__.py
│   │   │   ├── home.py
│   │   │   ├── landing.py
│   │   │   └── legal.py
│   │   ├── dashboard/              # Client cabinet routers
│   │   │   ├── __init__.py
│   │   │   ├── main.py
│   │   │   └── settings.py
│   │   ├── admin/                  # Admin panel routers
│   │   │   ├── __init__.py
│   │   │   ├── dashboard.py
│   │   │   ├── users.py
│   │   │   └── content.py
│   │   ├── services/
│   │   │   ├── billing.py
│   │   │   └── email.py
│   │   ├── templates/
│   │   │   ├── base.html
│   │   │   ├── base_public.html
│   │   │   ├── base_dashboard.html
│   │   │   ├── base_admin.html
│   │   │   ├── components/
│   │   │   ├── public/
│   │   │   ├── dashboard/
│   │   │   ├── admin/
│   │   │   ├── auth/
│   │   │   └── errors/
│   │   └── static/
│   │       ├── css/
│   │       │   ├── documatica.css    # v12.0 Design System
│   │       │   ├── dashboard.css
│   │       │   └── admin.css
│   │       ├── js/
│   │       └── images/
│   ├── content/
│   │   ├── home.yaml
│   │   └── navigation.yaml
│   └── data/                       # JSON storage for MVP
│       └── users.json
├── nginx/
│   └── default.conf
└── scripts/
    └── deploy.sh
```

## Generation Steps

When user triggers this skill:

### Step 1: Gather Project Info
Ask for:
1. **Project name** (slug, e.g. `invoicepro`)
2. **Project title** (display name, e.g. "InvoicePro")
3. **Domain** (e.g. `invoicepro.ru`)
4. **Brief description** (one sentence)

### Step 2: Create Base Structure
Generate files from `reference/` templates, replacing placeholders:
- `{{PROJECT_NAME}}` → project slug
- `{{PROJECT_TITLE}}` → display name
- `{{DOMAIN}}` → domain
- `{{DESCRIPTION}}` → brief description
- `{{YEAR}}` → current year

### Step 3: Setup Files to Create

**Root Level:**
- `docker-compose.yml`
- `.env.example`
- `.gitignore`
- `README.md`

**Backend Core:**
- `backend/Dockerfile`
- `backend/requirements.txt`
- `backend/app/main.py`
- `backend/app/database.py`
- `backend/app/models.py`
- `backend/app/core/config.py`
- `backend/app/core/templates.py`
- `backend/app/core/content.py`

**API Layer:**
- `backend/app/api/__init__.py`
- `backend/app/api/auth.py`
- `backend/app/api/payment.py`
- `backend/app/api/billing.py`

**Pages (Public):**
- `backend/app/pages/__init__.py`
- `backend/app/pages/home.py`
- `backend/app/pages/landing.py`
- `backend/app/pages/legal.py`

**Dashboard (Client Cabinet):**
- `backend/app/dashboard/__init__.py`
- `backend/app/dashboard/main.py`
- `backend/app/dashboard/settings.py`

**Admin Panel:**
- `backend/app/admin/__init__.py`
- `backend/app/admin/dashboard.py`
- `backend/app/admin/users.py`

**Services:**
- `backend/app/services/billing.py`
- `backend/app/services/email.py`

**Templates - Base:**
- `backend/app/templates/base.html`
- `backend/app/templates/base_public.html`
- `backend/app/templates/base_dashboard.html`
- `backend/app/templates/base_admin.html`

**Templates - Components:**
- `backend/app/templates/components/header.html`
- `backend/app/templates/components/footer.html`
- `backend/app/templates/components/sidebar.html`

**Templates - Pages:**
- `backend/app/templates/public/home.html`
- `backend/app/templates/public/landing.html`
- `backend/app/templates/auth/login.html`
- `backend/app/templates/auth/register.html`
- `backend/app/templates/dashboard/main.html`
- `backend/app/templates/dashboard/settings.html`
- `backend/app/templates/admin/dashboard.html`
- `backend/app/templates/errors/404.html`

**Static Files:**
- `backend/app/static/css/documatica.css` (from v12-style)
- `backend/app/static/css/dashboard.css`
- `backend/app/static/js/app.js`

**Content:**
- `backend/content/home.yaml`
- `backend/content/navigation.yaml`

**Deployment:**
- `nginx/default.conf`
- `scripts/deploy.sh`

### Step 4: Post-Generation Instructions

After generating, provide:

```bash
# 1. Copy environment file
cp .env.example .env

# 2. Edit .env with your settings:
# - SECRET_KEY (generate with: openssl rand -hex 32)
# - DATABASE_URL
# - TBANK_TERMINAL_KEY
# - TBANK_PASSWORD

# 3. Start with Docker
docker-compose up -d

# 4. Open http://localhost:8000
```

## T-Bank Payment Integration

Pre-configured endpoints:
- `POST /api/v1/payment/create` - Create payment
- `POST /api/v1/payment/webhook` - T-Bank webhook receiver
- `GET /dashboard/payment/success` - Success redirect
- `GET /dashboard/payment/fail` - Fail redirect

Supports:
- Subscription payments
- Pay-per-use packages
- Mock mode for testing

## Design System (v12-style)

This skill depends on **v12-style** for UI components.

Key classes:
- `.docu-h1`, `.docu-h2` - Typography
- `.docu-tag`, `.docu-body` - Text styles
- `rounded-[3rem]` - Section containers
- `rounded-2xl` - Buttons, inputs
- `bg-blue-600` - Primary accent
- `#FBBF24` (docu-gold) - AI/system indicators

## Files Reference

See `reference/` directory for complete file templates:
- `main.py.template` - FastAPI app entry
- `models.py.template` - SQLAlchemy models
- `payment.py.template` - T-Bank integration
- `base.html.template` - Root HTML template
- `base_dashboard.html.template` - Dashboard layout
- And more...

## Notes

- All templates use **Russian** for UI text
- **No emojis** anywhere in the project
- Environment variables for all secrets
- JSON files for MVP data storage (upgrade to PostgreSQL for production)
- Docker-ready from day one
