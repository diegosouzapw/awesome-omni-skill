---
name: archive-tasks
description: Archiva tareas completadas de TASKS.md a TASKS-DONE.md. Usar automáticamente cuando TASKS.md tenga muchas tareas completadas o supere 20K tokens.
model: haiku
---

# Archive Tasks

Archiva tareas completadas de `docs/llm/TASKS.md` a `docs/llm/TASKS-DONE.md`.

## Cuándo Usar Este Skill (Automático)

Aplicar automáticamente cuando:
- TASKS.md tiene más de 50 tareas completadas `[x]`
- TASKS.md supera los 20,000 tokens (error al leer completo)
- El usuario menciona que TASKS.md es muy grande
- Se detecta que TASKS.md no se puede leer entero

## Modelo

**OBLIGATORIO**: Usar `haiku`. Notificar: "Usando Haiku para archivar tareas"

## Proceso

1. Leer `docs/llm/TASKS.md` (en partes si es muy grande)
2. Identificar tareas completadas (líneas con `[x]` o `- [x]`)
3. Leer o crear `docs/llm/TASKS-DONE.md`
4. Añadir tareas completadas al inicio con fecha
5. Eliminar tareas completadas de TASKS.md
6. Guardar ambos archivos
7. Reportar cuántas tareas fueron archivadas

## Formato de TASKS-DONE.md

```markdown
# Tareas Completadas

## Archivado: YYYY-MM-DD

### [Sección original]
- [x] Tarea completada 1
- [x] Tarea completada 2

---

## Archivado: YYYY-MM-DD (anterior)
...
```

## Reglas

- **NO eliminar** tareas pendientes `[ ]` o en progreso
- **NO modificar** la estructura de secciones de TASKS.md
- **PRESERVAR** el contexto de cada tarea (su sección padre)
- **AÑADIR** fecha de archivado para trazabilidad

## Ejemplo de Output

```
Usando Haiku para archivar tareas

Analizando TASKS.md...
- 73 tareas completadas encontradas
- 12 tareas pendientes

Archivando a TASKS-DONE.md...
Done: 73 tareas movidas, TASKS.md reducido
```
