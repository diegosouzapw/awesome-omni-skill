---
name: app-builder
description: Use cuando el usuario quiera crear una nueva aplicación, necesite scaffolding de proyecto, determinar tech stack, o construir algo desde cero. Keywords: nueva app, crear proyecto, scaffolding, full-stack, tech stack, boilerplate, from scratch.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Agent
---

# App Builder - Application Building Orchestrator

> Analyzes user's requests, determines tech stack, plans structure, and coordinates agents.

## 🎯 Selective Reading Rule

**Read ONLY files relevant to the request!** Check the content map, find what you need.

| File                    | Description                            | When to Read                        |
| ----------------------- | -------------------------------------- | ----------------------------------- |
| `project-detection.md`  | Keyword matrix, project type detection | Starting new project                |
| `tech-stack.md`         | 2026 default stack, alternatives       | Choosing technologies               |
| `agent-coordination.md` | Agent pipeline, execution order        | Coordinating multi-agent work       |
| `scaffolding.md`        | Directory structure, core files        | Creating project structure          |
| `feature-building.md`   | Feature analysis, error handling       | Adding features to existing project |
| `templates/SKILL.md`    | **Project templates**                  | Scaffolding new project             |

---

## 📦 Templates (14)

Quick-start scaffolding for new projects. **Read the matching template only!**

| Template                                                       | Tech Stack          | When to Use           |
| -------------------------------------------------------------- | ------------------- | --------------------- |
| [nextjs-fullstack](templates/nextjs-fullstack/TEMPLATE.md)     | Next.js 16 + Prisma | Full-stack web app    |
| [nextjs-saas](templates/nextjs-saas/TEMPLATE.md)               | Next.js 16 + Stripe | SaaS product          |
| [nextjs-static](templates/nextjs-static/TEMPLATE.md)           | Next.js 16 + Framer | Landing page          |
| [astro-static](templates/astro-static/TEMPLATE.md)             | Astro 5             | Static/content site   |
| [nuxt-app](templates/nuxt-app/TEMPLATE.md)                     | Nuxt 3 + Pinia      | Vue full-stack app    |
| [express-api](templates/express-api/TEMPLATE.md)               | Express + JWT       | REST API              |
| [python-fastapi](templates/python-fastapi/TEMPLATE.md)         | FastAPI             | Python API            |
| [react-native-app](templates/react-native-app/TEMPLATE.md)     | Expo + Zustand      | Mobile app            |
| [flutter-app](templates/flutter-app/TEMPLATE.md)               | Flutter + Riverpod  | Cross-platform mobile |
| [electron-desktop](templates/electron-desktop/TEMPLATE.md)     | Electron + React    | Desktop app           |
| [chrome-extension](templates/chrome-extension/TEMPLATE.md)     | Chrome MV3          | Browser extension     |
| [cli-tool](templates/cli-tool/TEMPLATE.md)                     | Node.js + Commander | CLI app               |
| [monorepo-turborepo](templates/monorepo-turborepo/TEMPLATE.md) | Turborepo + pnpm    | Monorepo              |

---

## 🔗 Related Agents

| Agent                 | Role                             |
| --------------------- | -------------------------------- |
| `project-planner`     | Task breakdown, dependency graph |
| `frontend-specialist` | UI components, pages             |
| `backend-specialist`  | API, business logic              |
| `database-architect`  | Schema, migrations               |
| `devops-engineer`     | Deployment, preview              |

---

## ⚠️ Common Mistakes

| Error                           | Fix                                             |
| ------------------------------- | ----------------------------------------------- |
| Elegir tech stack sin preguntar | Hacer preguntas Socráticas primero              |
| Crear código sin plan           | Crear `{task-slug}.md` antes de escribir código |
| Usar template incorrecto        | Leer tabla de templates y elegir el más cercano |
| Ignorar checkpoints             | Verificar PLAN.md existe antes de proceder      |

---

## Usage Example

```
User: "Make an Instagram clone with photo sharing and likes"

App Builder Process:
1. Project type: Social Media App
2. Tech stack: Next.js + Prisma + Cloudinary + Clerk
3. Create plan:
   ├─ Database schema (users, posts, likes, follows)
   ├─ API routes (12 endpoints)
   ├─ Pages (feed, profile, upload)
   └─ Components (PostCard, Feed, LikeButton)
4. Coordinate agents
5. Report progress
6. Start preview
```
