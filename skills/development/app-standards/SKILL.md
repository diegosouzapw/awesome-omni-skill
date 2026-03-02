---
name: app-standards
description: All modes that write scripts or code follow these app standards for communication, modularization, simplification, naming conventions
---

# Standards-all-modes instructions

### Communication
Be brief; don't echo user requests.

### Modularization
**Scope**: Critical for Python, JS, and logic files.
- **Exception**: Do NOT apply this to CSS.

**Hard Limit**:
- **Enforce** a maximum of **450 lines of code** per file.
- **Split** larger files: Create more files with fewer functions rather than exceeding this limit.

**Utility Strategy**:
- **Extract** logic liberally into utility folders.
- **Naming Convention**: Use `utils/` or `utils_db/`.

### Naming Conventions: Domain-First
**Rationale**: Group related code by **Domain** (Subject) first, then **Specific** (Action/Qualifier).

#### Core Pattern
**Invert the standard naming order**:
- **Bad**: `{specific}_{domain}` (e.g., `edit_user`)
- **Good**: `{domain}_{specific}` (e.g., `user_edit`)

**Casing Rules**:
- **snake_case**: Files, functions, variables, DB tables/columns.
- **PascalCase**: Classes.

#### Transformation Examples
| Type | Old Pattern | **New Pattern (Target)** | Note |
| **Files** | `admin_dashboard_utils.py` | `dashboard_utils_admin.py` | Domain is `dashboard` |
| **Functions** | `edit_user` | `user_edit` | Domain is `user` |
| **Classes** | `AdminPerson` | `PersonAdmin` | Better: Use `Person` w/ type param |

#### Scope & Restrictions
**When to Apply**:
- **New Code**: **Always** apply this pattern.
- **Existing Code**: Apply **only** if you are already actively editing the file.

**Do NOT rename without explicit approval**:
- **Public APIs**: HTTP routes, library exports, CLI flags.
- **Database**: Tables and columns (requires migration).
- **Standards**: `__init__.py`, `setUp()`, `settings.py` (Django).

#### Refactoring Checklist
**If you rename a symbol, you MUST fix all references.**
Before finishing, verify:
1.  [ ] **Imports**: Updated in all other files?
2.  [ ] **Calls**: Function/Class usage updated everywhere?
3.  [ ] **Tests**: Do tests still pass?
4.  [ ] **Docs**: Updated docstrings/comments?
5.  [ ] **VS Code**: No errors in the Problems panel?
