---
name: Tero Voice Project Context
description: Load full project context, tech stack, status, and guidelines for the AI Receptionist SaaS project
---

# Tero Voice - AI Receptionist SaaS Project

## 📋 Product Overview

**Tero Voice** - AI Answering Service for Service Businesses

A marketing website for an AI receptionist service that helps service businesses (plumbers, dentists, carpenters, electricians, contractors) handle customer calls 24/7. The AI answers questions, schedules appointments, qualifies leads, and prevents missed calls.

### Key Value Propositions
- Never miss customer calls again
- 24/7 AI receptionist that sounds human
- Automatic appointment scheduling
- Lead qualification before callbacks
- Call analytics and transcripts
- Easy setup with no complex integrations

### Target Audience
Service businesses with high call volumes:
- Plumbers, Dentists, Carpenters, Electricians, Contractors, Multi-location providers

### Business Model
SaaS subscription with three tiers:
- Small Business: $49/month
- Growing Business: $129/month
- Enterprise: Custom pricing

---

## ⚠️ CRITICAL ARCHITECTURE NOTE

**Divergence Alert:**
The previous agent (Kiro) attempted to redesign the backend inference and optimization layer (likely pushing HuggingFace/Powers).
**The User's implementation (IBM failover / scaling) is superior and should be preferred.**
- **Do NOT** blindly follow Kiro's specs if they contradict the user's established IBM architecture.
- **Do NOT** replace existing robust scaling logic with Kiro's "Powers" unless explicitly asked.
- **Goal:** Restore/Preserve the IBM failover and optimization logic.

---

## 🛠️ Technology Stack

### Frontend
- **Framework**: React 18.2.0
- **Build Tool**: Vite 4.4.5
- **Styling**: Tailwind CSS 3.3.3
- **Routing**: React Router DOM 6.14.0
- **Icons**: Lucide React 0.263.1
- **Language**: JavaScript (JSX)

### Backend
- **Framework**: Flask (Python)
- **Database**: PostgreSQL on IONOS (74.208.227.161:5432)
- **Auth**: JWT tokens with bcrypt password hashing
- **Containerization**: Podman

### Development Commands
```bash
npm install        # Install dependencies
npm run dev        # Start dev server
npm run build      # Production build
npm run lint       # Run ESLint
```

---

## 📁 Project Structure

```
ai_website/
├── src/
│   ├── components/
│   │   ├── common/     # Header, Footer, Card
│   │   └── ui/         # Modal, Dropdown
│   ├── pages/          # Home, About, Products, NotFound
│   ├── services/       # API client (api.js)
│   └── styles/         # global.css
├── backend-setup/      # Flask API
├── .kiro/              # Kiro specs & steering (preserved)
├── .agent/             # Antigravity config (this)
└── docker-compose.yml  # Container orchestration
```

### Routes
- `/` - Home page (hero, features, pricing)
- `/about` - About page
- `/products` - Products page
- `*` - 404 page

---

## 📊 Current Status

### ✅ PHASE 1 COMPLETE
- Backend API with 16 endpoints (auth, clients, calls, analytics)
- PostgreSQL database configured (Models verified)
- **Persistence Layer**: `BillingService` and `UsageService` now use SQLAlchemy (removed mock data)
- JWT authentication and user isolation
- Podman containerization (Barebones Mini efficiency setup)
- Complete documentation

### 🏗️ ARCHITECTURE (Current State)
- **Type**: "Barebones Mini with Muscle" (Efficient VPS)
- **Logic**: Python/Flask API + Local Ollama + Redis + Postgres
- **Integration**: n8n + Neo4j + Pieces (Local Muscle - *Code Pending Discovery*)
- **Status**: Backend services optimized for persistence; waiting for IBM/n8n restoration logic.

### 🔄 PHASE 2 IN PROGRESS
- React frontend with Vite
- Tailwind CSS styling
- **UI/UX**: Multi-layer Parallax Wave Background (Fixed/Scrolling)
- API client with Axios (`src/services/api.js`)
- Environment configuration (`.env.local`)

### 📋 Active Specs (in `.kiro/specs/`)
- `member-portal-billing/` - Billing integration (4% complete)
- `paypal-mcp-integration/` - PayPal integration (COMPLETE)
- `single-page-hero-design/` - Hero design (COMPLETE)

---

## 🎯 Quality Guidelines
- Comment all code! Always comment your code!
- Always use the latest version of any library or framework
- never override the user's code unless explicitly asked
- never overwrite code in the project .old and .archive folders
-common sense is the best sense
- code examples are in the examples folder
at 420du/content/code-examples
### Before Code Generation
- Review the project context
- Look at the current date and time
- Review the project status
- check dependencies
- 
- Review requirements for security implications
- Review the requirements for the feature you - are about to implement
- Review the architecture for the feature you - are about to implement
- Inspect the code for the feature you - are about to implement
- Create a plan for the feature you are about to implement
- Make a to do list for the feature you are about to implement
- Identify potential vulnerabilities
- Plan for error handling
- Consider edge cases

### During Code Generation
- Follow ESLint standards
- Include error handling
- Add input validation
- Include security checks

### After Code Generation
- Run `npm run lint`
- Check for security issues
- Verify functionality

### Security Checklist
- XSS prevention (sanitize input)
- CSRF protection
- SQL injection prevention (use ORM)
- No hardcoded credentials
- Proper error handling (no stack traces exposed)

---

## 🔗 Key Files

| File | Purpose |
|------|---------|
| `src/services/api.js` | Centralized API client |
| `.env.local` | Environment config |
| `.kiro/specs/*/tasks.md` | Task tracking |
| `.kiro/steering/quality-assurance.md` | Full QA guidelines |
| `docker-compose.yml` | Container setup |

---

## 🚀 Quick Reference

### Database
- Host: `74.208.227.161:5432`
- Database: `ai_receptionist`
- User: `user`

### API
- Local: `http://localhost:8000`
- Endpoints documented in backend-setup

### When to Escalate
- Security vulnerability found
- Architecture redesign needed
- Complex debugging required
- Performance issues

---

## 📌 Remember

1. All `.kiro/` files are preserved for Kiro compatibility
2. Use `.agent/workflows/` for Antigravity commands
3. Specs in `.kiro/specs/` are still the source of truth
4. Follow quality assurance guidelines for all code
