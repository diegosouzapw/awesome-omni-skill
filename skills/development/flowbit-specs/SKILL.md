---
name: flowbit-specs
description: Spec-driven development rules for Flowbit. Always follow /specs as source of truth.
---

# Flowbit – Agent Skill

## 1. Source of truth
- Always read **all files under `/specs`** before writing or modifying any code.
- Treat `/specs` as the **single source of truth**.
- If code conflicts with specs, update the code, not the specs (unless explicitly instructed).

---

## 2. Product definition
Flowbit is an **Operations OS for SMEs**, not an ERP and not an accounting system.

Flowbit manages:
- Projects (sales/orders)
- Branching workflows using stages (areas) and dependencies
- Deadlines per stage and project-level timelines (planned vs actual)
- Unit quantity tracking per stage (qty_required, qty_done)
- Visual finance using **income/cost events** (no formal accounting)
- Products with BOM (materials per unit) and **project-level editable snapshots**

Out of scope:
- Formal accounting (PUC, taxes, invoices)
- Payroll
- Complex inventory or reservations
- Automated notifications
- Over-engineered abstractions

---

## 3. Architecture & stack
### Backend
- FastAPI
- SQLAlchemy
- Pydantic
- PostgreSQL
- Multi-tenant architecture: **every entity is scoped by `company_id`**

### Frontend
- React + TypeScript
- Vite
- Tailwind CSS
- **shadcn/ui** components (Radix-based)
- Icons from `lucide-react`

---

## 4. UI rules (very important)
- Build a **real SaaS dashboard**, not a demo or toy UI.
- Use sidebar + top header layout.
- Prefer `Card`, `Table`, `Tabs`, `Badge`, `Button`, `Dialog` from shadcn/ui.
- Avoid raw `<div>`-only layouts.
- Avoid inline styles.
- Pages must look production-ready and consistent.

---

## 5. Roles & permissions
Roles:
- `SUPER_ADMIN`: platform owner (can create companies).
- `COMPANY_ADMIN` (Boss): full access inside a company, including finance.
- `STAGE_WORKER`: access only to assigned stages.

Rules:
- Finance data is visible **only** to `COMPANY_ADMIN`.
- `STAGE_WORKER` users must never see:
  - prices
  - margins
  - income or cost events
- Stage access is mandatory to view or edit a `ProjectStage`.

---

## 6. Workflow rules (branching)
- Projects select which stages apply.
- Stages have dependencies (`edges`) that define prerequisites.
- Stage statuses:
  - `BLOCKED`
  - `READY`
  - `IN_PROGRESS`
  - `DONE`
  - `SKIPPED`
- A stage becomes `READY` only when **all selected prerequisites are `DONE` or `SKIPPED`**.
- When a stage becomes `DONE`, dependent stages must be re-evaluated.

---

## 7. Deadlines & timeline
Each `ProjectStage` tracks:
- `planned_due_date`
- `actual_ready_at`
- `actual_started_at`
- `actual_done_at`

Timeline rules:
- Show planned vs actual per stage.
- Delays must be computable from stored timestamps.
- Timers are optional but supported.

---

## 8. BOM & materials
- Product BOM defines materials per unit.
- On project creation, generate **project material requirement snapshots**:
  - quantities per unit
  - total quantities
- Project material requirements are **editable**.
- Editing project requirements must **not** modify the product BOM.

---

## 9. Finance (visual only)
- Financial events are simple records:
  - type: `INCOME` or `COST`
  - amount
  - date
  - category
- No accounting logic, no double entry, no taxes.
- Metrics:
  - total income per project
  - total cost per project
  - margin = income − cost

---

## 10. Development philosophy
- MVP first. Do not build future features prematurely.
- Prefer clarity over cleverness.
- Avoid over-abstraction.
- Code should look **production-ready**, not tutorial code.

---

## 11. When in doubt
If a requirement is unclear:
1. Check `/specs`.
2. If still unclear, ask before implementing.