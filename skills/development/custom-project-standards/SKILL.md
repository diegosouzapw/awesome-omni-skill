---
name: custom-project-standards
description: Hệ thống tiêu chuẩn dự án đa năng (Standard Platform). Hỗ trợ Frontend, Backend, DevOps với nhiều tùy chọn ngôn ngữ/framework.
allowed-tools: Read, Write, Edit, Find
version: 2.0
---

# Custom Project Standard Platform (v2.1)

> **OPERATING MECHANISM:**
> 1. **Auto-Config:** Checks for `agent-project-config.md` in the workspace root. If found, loads Context/Role from there.
> 2. **Wizard Mode:** If NO config file is found, asks the User to determine Context & Stack.

---

## 1. Setup & Context Protocol (Configuration Protocol)

### Step 0: Check Config (Highest Priority)
Before asking the user, search for the `agent-project-config.md` file in the root directory.

**If found:**
Read the Frontmatter of the file to determine:
- `project_type` -> Determine Domain (Frontend/Backend...)
- `tech_stack` -> Determine Stack (NestJS/NextJS...)
- `role_mode` -> Determine Persona (Architect/Implementer...)

> **Note:** If Config exists, **SKIP** Steps 1 and 2 below, proceed directly to Step 3 (Load Context).

**If NOT found:**
Execute "Wizard" mode (Steps 1 & 2), then **propose the user to create a config file** for future sessions.

### Step 0.5: Analyze Environment (Existing Codebase Protocol)
Before applying any rigid rules or installing new packages, the Agent **MUST READ** the dependency management files (`package.json`, `go.mod`, `composer.json`, `requirements.txt`).

**Principles:**
1.  **Respect Legacy:** If the project already uses a specific library (e.g., `TypeORM` instead of `Prisma`), **USE IT**. Do not suggest switching unless explicitly requested.
2.  **Kit Default:** Only recommend the Kit's preferred packages (from `tech-stack.md`) if the project is brand new or lacks that specific capability.

### Step 1: Determine Project Type (Domain) - *Wizard Mode*
Ask the User or auto-detect:
*   [ ] **Frontend**
*   [ ] **Backend**
*   [ ] **DevOps**

### Step 2: Determine Language/Framework (Stack) - *Wizard Mode*
Based on the Domain, further determine:
*   **Frontend:** Next.js | Vue | React Native
*   **Backend:** NestJS | Python (FastAPI) | Go | Laravel
*   **DevOps:** Docker | K8s | AWS

### Step 3: Load Context (Important)
After determination (via Config or Wizard), the Agent **MUST READ** the following documents in order:

1.  **Core Principles (Global):** `rules/standards/design-principles.md` (MUST READ FIRST)
2.  **Domain Rules:** (e.g., `rules/backend/general.md`)
3.  **Stack Rules:** (e.g., `rules/backend/nestjs/convention.md`)
4.  **Role Mode (If available):** (e.g., `rules/roles/architect.md`)

> 💡 **Example:** Config is `Backend` + `NestJS`:
> -> Agent reads `rules/standards/design-principles.md`
> -> Agent reads `rules/backend/general.md`
> -> Agent reads `rules/backend/nestjs/convention.md`

---

## 2. Rules Map

### 🎨 Frontend
| Stack | Rule File | Template |
|-------|-----------|----------|
| **General** | `rules/frontend/general.md` | - |
| **Next.js** | `rules/frontend/nextjs/convention.md` | `templates/frontend/nextjs/` |
| **Vue/Nuxt** | `rules/frontend/vue/convention.md` | `templates/frontend/vue/` |

### ⚙️ Backend
| Stack | Rule File | Template |
|-------|-----------|----------|
| **General** | `rules/backend/general.md` | - |
| **NestJS** | `rules/backend/nestjs/convention.md` | `templates/backend/nestjs/` |
| **Laravel** | `rules/backend/laravel/convention.md` | - |
| **Go** | `rules/backend/go/convention.md` | - |
| **Python** | `rules/backend/python/convention.md` | `templates/backend/python/` |

### 🚀 DevOps
| Stack | Rule File |
|-------|-----------|
| **Docker** | `rules/devops/docker-standard.md` |
| **CI/CD** | `rules/devops/cicd-standard.md` |

---

## 3. Installation Guide

In case the User requests *"Create a new project"*, follow these steps:

1.  **Ask User:** "Do you want to create a Frontend or Backend project? Which language?"
2.  **Confirm:** "I will create a **Next.js** project following **Custom Standard v2**."
3.  **Execute:**
    *   Copy structure from `templates/frontend/nextjs/` (if available)
    *   Create `README.md` with content from `rules/frontend/nextjs/convention.md`
    *   Install required dependencies (check `tech-stack.md` in the subfolder).


