---
name: dobacksoft-stabilsafe
description: Enforces DobackSoft/StabilSafe V3 conventions: API via frontend/src/config/api.ts, logger (no console.log), organizationId on requests, ports 9998/5174, iniciar.ps1 as sole startup, Windows/PowerShell, immutable menu modules, PDF export flow. Use when editing backend, frontend, or docs in this repo or when the user mentions DobackSoft, StabilSafe, iniciar.ps1, or project rules.
globs: ["backend/**", "frontend/**", "docs/**", "prisma/**", "AGENTS.md", ".cursorrules", ".cursor/**"]
---

# DobackSoft (StabilSafe V3) — Convenciones de proyecto

**Cuándo aplicar:** Al editar código o docs de este repo, o cuando el usuario cite reglas del proyecto, DobackSoft o iniciar.ps1. Para auth, multi-tenant, DoD y rutas por ámbito ver **AGENTS.md** en la raíz.

Actuar como programador en pareja: explicar qué se va a hacer y por qué antes de editar. Respetar modularidad, roles y flujo del producto.

## Reglas críticas (no violar)

| Regla | Acción |
|-------|--------|
| Inicio del sistema | **NUNCA** iniciar backend/frontend; el usuario usa `iniciar.ps1` |
| URLs y API | **NUNCA** hardcodear URLs; usar `frontend/src/config/api.ts` |
| Logging | **NUNCA** `console.log`; usar `logger` de `utils/logger` |
| Puertos | Backend **9998**, frontend **5174** — no cambiar |
| Menú | No crear módulos fuera del menú oficial V3 |
| Datos | No exponer datos entre organizaciones; siempre filtro `organizationId` |
| Dependencias | No usar >300 KB sin justificar |

## Código

- Un archivo por turno; un diff agrupado por archivo. Formato: `// ... existing code ...` donde aplique.
- Antes de editar: leer contexto (imports, funciones relacionadas). No código basura ni hashes largos. Máximo 3 intentos por error; si falla, pedir ayuda al usuario.
- Config: TypeScript estricto (no `any` sin justificación). Headers con `organizationId` en requests. Timeout configurable en API.

## Entorno y comandos

- **Windows**: scripts PowerShell (p. ej. `iniciar.ps1`), rutas Windows (`\`), comandos compatibles con PowerShell.
- Salida de comandos: añadir `| cat` cuando sea útil (git log, head, less). Procesos largos en segundo plano. Proponer comandos y pedir confirmación antes de ejecutar.

## Menú oficial V3 (inmutable)

- Panel de Control | Estabilidad | Telemetría (CAN/GPS) | IA | Geofences | Operaciones | Reportes | Administración (ADMIN) | Base de Conocimiento (ADMIN) | Mi Cuenta.

Diseño: React + Tailwind, Leaflet + TomTom para mapas. KPIs protagonistas en Panel; scroll solo donde haga falta. Comparadores: solo Estabilidad o solo CAN/GPS según módulo.

## Flujo obligatorio

Subida → Procesamiento automático → Visualización → Comparación → Exportación. PDF en 1 clic desde Panel, Sesiones, IA y comparadores.

## Checklist rápida por cambio

Antes de cerrar un cambio, comprobar:

- [ ] Modularidad y menú oficial V3
- [ ] Scroll solo donde haga falta; comparadores correctos (Estabilidad vs CAN/GPS)
- [ ] Roles ADMIN/MANAGER; flujo Subida→Exportación; PDF en 1 clic
- [ ] `config/api.ts` (no URLs hardcodeadas); `logger` (no console.log); filtro `organizationId`
- [ ] Compatible con `iniciar.ps1` (no iniciar backend/frontend por cuenta propia)

## Ejemplos

| Evitar | Usar |
|--------|------|
| `const base = 'http://localhost:9998'` | `import { apiBaseUrl } from '@/config/api'` (o equivalente del proyecto) |
| `console.log('debug', x)` | `logger.debug('debug', { x })` (desde `utils/logger`) |
| Lanzar `npm run dev` en backend/frontend | Recordar al usuario que use `iniciar.ps1` |

## Verificación en navegador

Cuando el usuario pida comprobar algo en el navegador (Swagger UI, login, pantallas), usar el MCP de navegador (cursor-ide-browser o chrome-devtools): listar pestañas, navegar a la URL (p. ej. `http://localhost:9998/api-docs/`), tomar snapshot para ver el contenido. Detalle del flujo lock/unlock y URLs: [reference.md](reference.md#verificación-en-navegador-mcp).

## Más detalle

Módulos, métricas por área y checklist extendida: [reference.md](reference.md).
