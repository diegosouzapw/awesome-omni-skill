---
name: project-setup
description: Interactive project setup for new repos using this template. Detects placeholder text in project.mdc, asks about tech stack, recommends skills, and configures the project. Trigger automatically when project.mdc contains "[Project Name]".
---

# Project Setup Skill

Interactive onboarding flow for new projects created from the ai-project-start template. This skill guides users through configuring their project with the right tech stack and skills.

## Trigger Condition

This skill should trigger automatically when:
- `project.mdc` contains the placeholder `[Project Name]`

Or when the user explicitly asks to set up their project.

---

## Phase 1: Detection & Welcome

### Check for New Project

Read `.cursor/rules/project.mdc` and check for placeholder text:

```
[Project Name]
[One-line description of the project]
[e.g., Next.js 14 App Router, TypeScript, Tailwind, shadcn/ui]
```

If placeholders exist, proceed with setup. If project appears configured, ask:
> "Your project.mdc appears to be configured. Would you like to reconfigure it, or is there something specific I can help with?"

### Welcome Message

```
👋 Welcome to your new project!

Let's get you set up. I'll ask a few questions about what you're building,
then recommend skills to import and configure your project.mdc file.

This takes about 2 minutes.
```

---

## Phase 2: Gather Project Info

### Ask Questions

Ask these questions one at a time or use the AskQuestion tool:

**1. Project Name**
> What's the name of your project?

**2. Description**
> In one sentence, what are you building?

**3. Project Type**
> What type of project is this?
- Web App (Next.js)
- API Only (NestJS)
- Full Stack (Next.js + NestJS)
- CLI Tool
- Other

**4. Dashboard Style** (if Web App or Full Stack)
> Which dashboard layout do you want to start with?
- SaaS Admin (sidebar nav, user management, billing pages)
- Analytics (charts, metrics, data tables)
- E-commerce (products, orders, inventory)
- Minimal Shell (just auth + empty dashboard skeleton)

**5. Database**
> What database are you using?
- PostgreSQL (Drizzle ORM)
- PostgreSQL (Prisma ORM)
- SQLite
- None / External API only

**6. Authentication**
> How will you handle auth?
- Better Auth
- NextAuth
- External auth service
- None needed yet

**7. Deployment**
> Where will you deploy?
- Railway
- Vercel
- AWS
- Docker / Self-hosted
- Not decided yet

**8. State Management** (if Web App or Full Stack)
> How will you manage frontend state?
- Tanstack Query + Effector
- Zustand
- React state only

**9. Task Tracking**
> Do you use Linear for task tracking?
- Yes
- No / Other

---

## Phase 3: Recommend Skills

Based on answers, recommend skills from the cursor-skills repo.

### Skill Recommendation Matrix

| Tech Choice | Recommended Skills |
|-------------|-------------------|
| Next.js | `nextjs-16`, `ui-shadcn-studio` |
| NestJS | `nestjs` |
| PostgreSQL + Drizzle | `db-postgres` |
| PostgreSQL + Prisma | `db-prisma` |
| Better Auth | `auth-better-auth` |
| Railway | `infra-railway` |
| Docker | `docker-local` |
| REST API | `api-rest` |
| Tanstack Query + Effector | `effector`, `state-management` |
| Linear tasks | `linear` |
| Any frontend project | `feature-build`, `ui-design-system`, `ui-shadcn-studio` |
| Any project | `feature-build`, `documentation`, `versioning` |

### Present Recommendations

```markdown
## Recommended Skills

Based on your stack, I recommend importing these skills:

**Core (always useful):**
- `feature-build` - Complete feature development lifecycle
- `documentation` - Project documentation standards
- `versioning` - Semantic versioning with CHANGELOG

**For your stack:**
- `nextjs-16` - Next.js 16 patterns and conventions
- `db-postgres` - PostgreSQL with Drizzle ORM
- `effector` - Effector state management
- `linear` - Linear issue tracking

### How to Import Skills

1. Open Cursor Settings (Cmd+Shift+J)
2. Go to the **Rules** tab
3. Click **Add Rule** → **Remote Rule (GitHub)**
4. Enter: `https://github.com/aussiegingersnap/cursor-skills`
5. Select the skills you want to import

The skills will be added to your `.cursor/skills/` directory.
```

---

## Phase 4: Update project.mdc

### Generate Updated Content

Replace placeholders with gathered info:

```markdown
---
alwaysApply: true
---

# {Project Name}

NEVER EDIT MY ENV FILES FOR ME, ALWAYS GIVE ME DIRECT INSTRUCTIONS WITH EXAMPLES TO COPY ON WHAT TO EDIT

## What We're Building

{Description}

## Tech Stack

- **Frontend**: {Frontend stack}
- **Backend**: {Backend stack if applicable}
- **Database**: {Database choice}
- **Auth**: {Auth choice}
- **State**: {State management}
- **Deploy**: {Deployment target}
- **Skills**: Cursor native skills

## Core Constraints

- **KISS**: Simplest solution that works
- **DRY**: Don't repeat—types in code, schema in migrations, patterns in docs
- {Any project-specific constraints}

## Documentation

| Doc | Purpose |
|-----|---------|
| [TASKS.md](../TASKS.md) | Current sprint + backlog |
| [CHANGELOG.md](../CHANGELOG.md) | Progress updates |
| [docs/architecture.md](../docs/architecture.md) | Technical patterns + decisions |

## Git Standards

**Conventional Commits**: `<type>(<scope>): <subject>`

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `chore`, `ci`, `build`, `revert`

```bash
feat: add user authentication
fix(api): handle null response
docs: update architecture patterns
```

**Security**:
- ❌ Never commit `.env` files or secrets
- ✅ Secrets in `.env.local` (gitignored)
- ✅ `NEXT_PUBLIC_` prefix for client-safe vars

## Environment

See `.env.example` for required variables.

## Workflow

When a Cursor plan is complete, provide a summary block for manual git commits:

```
### Done

**Summary**: [Brief description of changes]

**Commits** (ohmyzsh):
gaa && gcmsg "feat(scope): description"
gcmsg "fix(scope): description"
gp
```

User will copy/paste these commands. Lefthook validates commit format and blocks secrets.
```

### Confirm Before Writing

> I'll update your project.mdc with this configuration. Here's a preview:
> 
> [Show key fields]
>
> Does this look correct? (y/n)

---

## Phase 5: Finalize Setup

### Update Files

1. **Update project.mdc** with gathered configuration
2. **Update TASKS.md** with initial sprint items based on project type
3. **Update README.md** with project name and description

### Completion Message

```markdown
## Setup Complete! 🎉

Your project is configured with:
- **Project**: {Name}
- **Stack**: {Summary}
- **Skills**: {Count} recommended (import via Cursor Settings)

### Next Steps

1. Import recommended skills via Cursor Settings (Cmd+Shift+J → Rules)
2. Configure your `.env.local` based on `.env.example`
3. Start building! Use `/feature-build` to work on tasks

### Quick Commands

```bash
# Start dev server
npm run dev

# Git workflow (ohmyzsh)
gaa && gcmsg "feat(scope): description"
gp
```

Happy building!
```

---

## Error Handling

### If User Skips Questions

Use sensible defaults:
- Project Type: Web App
- Database: None (can add later)
- Auth: None (can add later)
- Deploy: Not decided
- State: React state only

### If Import Fails

Provide manual import instructions:
```bash
# Clone cursor-skills and copy desired skills
git clone https://github.com/aussiegingersnap/cursor-skills /tmp/cursor-skills
cp -r /tmp/cursor-skills/skills/{skill-name} .cursor/skills/
```

---

## Quick Reference

### Minimal Setup Flow

For users who want fast setup:

1. "What's your project name and one-line description?"
2. "Web app, API, or full stack?"
3. Generate config with defaults
4. List recommended skills
5. Done

### Full Setup Flow

For users who want complete configuration:

1. All 8 questions
2. Detailed skill recommendations with explanations
3. Preview and confirm project.mdc
4. Update all project files
5. Detailed next steps
