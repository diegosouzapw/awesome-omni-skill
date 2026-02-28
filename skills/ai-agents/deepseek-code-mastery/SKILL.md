---
name: deepseek-code-mastery
description: Referencia completa para operar DeepSeek Code (1M tokens por chat via web, open source, sin expiracion) como sistema multi-agente subordinado a Claude Code. v4.2 certificado — Serena auto-init en agent mode (3 tools de code intelligence sin intervencion manual), quantum merge con deduplicacion de clases ES6 (extract_classes + pick_better_implementation, strategy function_based), skills_dir fallback automatico en orchestrator, Phase 2 token tracking real. Token-Efficient Pipeline (save_response.py directo a disco, ~96% ahorro), Semantic Engine central (TF-IDF, cosine similarity, Bayesian Beta, temporal decay, Mann-Kendall), Skills por similaridad semantica, memoria dual (surgical + global), Intelligence Package (5 features), dual quantum sessions, multi-step, multi-session, converse, y protocolo de sesiones 3-fases. Usar cuando el usuario pida delegar codigo, generar funciones, crear features, o cualquier tarea de generacion masiva.
---

# DeepSeek Code Mastery — Guia Completa para Claude Code

## 1. Overview

**DeepSeek Code** es un asistente de programacion local basado en DeepSeek V3 (API o sesion web) con herramientas MCP (filesystem, shell, search, memory, Serena). Claude Code actua como **cirujano** (planifica, revisa, orquesta) y delega el **trabajo pesado de generacion de codigo** a DeepSeek Code como subordinado.

**Ubicacion del proyecto:** Directorio donde se clono/instalo DeepSeek Code (contiene `run.py`)
**Entry point:** `python run.py` (o `DeepSeekCode.exe`)
**Lenguaje:** Python 3 (PyQt5 para login web, asyncio para networking)

> **Nota sobre rutas:** En esta documentacion, `<DEEPSEEK_DIR>` se refiere al directorio raiz del proyecto DeepSeek Code (donde esta `run.py`) y `<APPDATA>` se refiere al directorio de datos de la aplicacion (`%APPDATA%\DeepSeek-Code` en Windows, `~/.config/DeepSeek-Code` en Linux/macOS). Detecta estas rutas automaticamente al ejecutar comandos.

### Modos de Operacion

| Modo | Flag CLI | Descripcion |
|------|----------|-------------|
| Interactivo | (ninguno) | Chat con herramientas, modo web o API |
| One-shot | `-q "pregunta"` | Pregunta unica, respuesta y sale |
| Agente | `--agent "meta"` | Autonomo multi-paso con herramientas |
| **Delegacion** | `--delegate "tarea"` | **PRINCIPAL**: Claude Code delega codigo |
| **Quantum** | `--quantum "tarea"` | Delegacion paralela dual + merge |
| **Multi-session** | `--multi "tarea"` | N instancias paralelas con roles |
| **Multi-step** | `--multi-step plan.json` | Plan multi-paso secuencial/paralelo |
| Conversacion | `--converse "msg"` | Dialogo multi-turno iterativo |
| **Requirements** | `--requirements doc.md` | **v2.2**: Documento de requisitos → plan ejecutable |
| **Health Report** | `--health-report` | **v2.2**: Reporte predictivo de salud del proyecto |

### Flags Globales Nuevos (v2.0)

| Flag | Descripcion |
|------|-------------|
| `--negotiate-skills` | DeepSeek elige sus propias skills del catalogo |
| `--multi "tarea"` | Modo multi-sesion (N instancias) |
| `--roles PRESET` | Preset de roles para multi-sesion |
| `--instances N` | Numero de instancias (0=auto) |
| `--pipeline` | Modo pipeline secuencial (gen -> review -> fix) |

### Sistema Adaptivo de Prompts (v2.0)

El sistema clasifica cada tarea en 5 niveles y adapta automaticamente:
- **CHAT (0)**: "Hola" → ~25 tokens de system prompt, 0 skills
- **SIMPLE (1)**: "Que es un closure?" → ~50 tokens, 0 skills
- **CODE_SIMPLE (2)**: "Arregla typo" → ~80 tokens, max 10K skills
- **CODE_COMPLEX (3)**: "Sistema de inventario" → ~600 tokens, max 40K skills
- **DELEGATION (4)**: Via --delegate → prompt completo, max 80K skills

### Negociacion de Skills (v2.0)

Con `--negotiate-skills`, DeepSeek recibe un catalogo compacto (~2K tokens) de las 51 skills disponibles y elige cuales necesita realmente. Esto reemplaza la inyeccion heuristica y reduce tokens desperdiciados.

### Multi-Session (v2.0)

Generaliza quantum (2 instancias) a N instancias con roles:
- **generate-review**: Generador + reviewer (default)
- **full-pipeline**: Generador + reviewer + tester
- **dual-generator**: Dos generadores independientes
- **specialist-pair**: Dos especialistas en dominios diferentes

---

## 2. Rutas y Configuracion

### Paths Criticos

```
Proyecto:    <DEEPSEEK_DIR>/                          (directorio raiz con run.py)
Config:      <APPDATA>/config.json                    (%APPDATA%\DeepSeek-Code en Windows)
WASM:        <APPDATA>/sha3_wasm_bg.wasm              (auto-descargado al primer login)
Skills:      <DEEPSEEK_DIR>/skills/                   (51 archivos .skill)
Memory:      <APPDATA>/memory.md
Surgical:    <APPDATA>/surgical_memory/
Global:      <APPDATA>/global_memory.json
EXE:         <DEEPSEEK_DIR>/dist/DeepSeekCode.exe     (si se compilo con PyInstaller)
```

### config.json (campos importantes)

```json
{
  "bearer_token": "...",       // Token de sesion web (auto via /login)
  "cookies": "...",            // Cookies de sesion web
  "api_key": "...",            // No usado (solo modo web)
  "wasm_path": "...",          // Ruta al WASM para firma de requests web
  "allowed_paths": [],         // Directorios permitidos (vacio = todo)
  "allowed_commands": [],      // Comandos shell permitidos
  "serena_enabled": true,      // Serena code intelligence
  "serena_project": "...",     // Proyecto activo en Serena
  "summary_threshold": 80,     // % de contexto antes de resumir
  "skills_dir": "...",         // Directorio de skills
  "auto_select_model": true,   // Auto deepseek-reasoner para CODE_COMPLEX+DELEGATION
  "thinking_enabled": true,    // Thinking mode en web sessions para codigo
  "pool_size": 5,              // Pool size para futuro multi-session (2-10, quantum actual usa 2)
  "chunk_threshold_tokens": 30000  // Umbral para chunking de templates
}
```

### Autenticacion

DeepSeek Code se conecta directamente a la **web de DeepSeek** (gratis, open source, sin expiracion). Cada chat tiene **1M tokens de contexto** — los chats son independientes y no comparten memoria entre si.

- **Modo Web**: `bearer_token` + `cookies` + WASM para firmar requests. Login via `/login` (PyQt5 WebEngine). Tokens expiran cada ~24-48h, re-login automatico si falla. **Gratis**.
- **Modo API**: `api_key` de platform.deepseek.com. Pago por tokens.
- **128,000 tokens de contexto** — las skills se inyectan generosamente (hasta 80K) dejando espacio para template + respuesta

---

## 3. Delegacion Oneshot (`--delegate`)

### Sintaxis Completa

```bash
python run.py --delegate "TAREA" [opciones] --json
```

**Opciones:**

| Flag | Descripcion | Ejemplo |
|------|-------------|---------|
| `--delegate "tarea"` | Descripcion de la tarea | `--delegate "crea endpoint REST /api/users"` |
| `--template archivo` | Archivo con esqueleto TODO | `--template game.js` |
| `--context archivo` | Archivo de referencia de estilo | `--context existing-code.js` |
| `--feedback "texto"` | Correccion de intento anterior | `--feedback "falta validacion de input"` |
| `--max-retries N` | Reintentos si validacion falla (default: 1) | `--max-retries 2` |
| `--no-validate` | Desactivar auto-validacion | `--no-validate` |
| `--project-context ruta` | CLAUDE.md del proyecto | `--project-context ./CLAUDE.md` |
| `--json` | Output JSON estructurado | `--json` |
| `--config ruta` | Config alternativa | `--config ./mi-config.json` |

### Flujo Interno de Delegacion

```
1. Carga config.json y crea DeepSeekCodeClient
2. SurgicalMemory.pre_delegation() -> briefing del proyecto (reglas, errores previos)
3. GlobalMemory.pre_delegation() -> briefing personal cross-proyecto (estilo, skills, errores)
4. build_delegate_skills_context():
   a. Carga CORE_SKILLS siempre (programming-foundations, data-structures, common-errors)
   b. Detecta skills Domain relevantes por keywords de la tarea
   c. Llena hasta el budget de tokens (core:15K + domain:45K + specialist:20K = 80K)
5. SessionOrchestrator.prepare_session_call() detecta inyecciones pendientes:
   - Skills relevantes (TF-IDF semantic detection)
   - SurgicalMemory briefing (contexto del proyecto)
   - GlobalMemory briefing (perfil personal cross-proyecto)
6. chat_in_session() ejecuta protocolo 3-fases:
   Fase 1: system_prompt (DELEGATE_SYSTEM_PROMPT base) -> "OK"
   Fase 2: pending_injections (skills, surgical, global) -> "Ack" por cada uno
   Fase 3: user_message limpio (task + template + context + feedback)
7. DeepSeek genera la respuesta (codigo puro, sin markdown)
8. delegate_validator valida: no truncamiento, TODOs completos, sin errores canvas
9. Si falla validacion y quedan retries: re-intenta con feedback de errores
10. SurgicalMemory.post_delegation() -> aprende de resultado del proyecto
11. GlobalMemory.post_delegation() -> aprende patrones personales cross-proyecto
12. Retorna JSON: { success, response, validation, retries_used, duration_s }
```

### Interpretacion del JSON de Respuesta

```json
{
  "success": true,
  "response": "// === TODO 1: ENEMY_TYPES ===\nlet ENEMY_TYPES = {...}\n...",
  "mode": "delegate",
  "had_template": true,
  "had_context": false,
  "duration_s": 12.5,
  "validation": {
    "valid": true,
    "truncated": false,
    "issues": [],
    "todos_found": ["ENEMY_TYPES", "renderMap"],
    "todos_missing": [],
    "stats": {"lines": 150, "functions": 8}
  },
  "token_usage": {
    "system_prompt": 7000,
    "skills_injected": 35000,
    "surgical_briefing": 1200,
    "global_briefing": 500,
    "template": 3000,
    "context_file": 0,
    "user_prompt": 250,
    "total_input": 46950,
    "response_estimated": 8500,
    "total_estimated": 55450,
    "context_remaining": 953050,
    "context_used_percent": "4.7%"
  }
}
```

**Campos clave:**
- `success: false` + `validation.truncated: true` = respuesta cortada, reducir scope
- `success: false` + `validation.todos_missing: ["renderMap"]` = TODOs no completados
- `success: true` + `response` = codigo listo para usar
- `token_usage.context_used_percent` = cuanto del 1M se consumio
- `token_usage.skills_injected` = tokens de conocimiento inyectado

### Cuando Usar --template vs Tarea Libre

**Usar --template cuando:**
- Tienes un archivo esqueleto con marcadores `// TODO: nombre` que DeepSeek debe rellenar
- Quieres controlar la estructura exacta del codigo
- El resultado debe integrarse en un archivo existente

**Usar tarea libre (sin --template) cuando:**
- Pides generar codigo desde cero
- La tarea es autocontenida (un endpoint, una funcion, un componente)
- No hay estructura preexistente que seguir

### Cuando Usar --context

Pasa `--context archivo.js` cuando quieres que DeepSeek **imite el estilo** de codigo existente: nombres de variables, patrones, estructura, convenciones del proyecto. DeepSeek lo lee y adapta su output para ser consistente.

### Cuando Usar --feedback

Pasa `--feedback "descripcion de errores"` cuando un intento anterior fallo y quieres que DeepSeek corrija errores especificos. El feedback se inyecta como "CORRECCION IMPORTANTE" en el prompt.

### Ejemplo Completo de Delegacion

```bash
# Tarea libre - generar desde cero
python run.py --delegate "crea un servidor Express con endpoints CRUD para usuarios, validacion Zod, y middleware de auth JWT" --json

# Con template - rellenar TODOs
python run.py --delegate "implementa las funciones de combate" --template game-combat.js --context game-player.js --json

# Con feedback - corregir intento anterior
python run.py --delegate "implementa sistema de inventario" --template inventory.js --feedback "falta drag-and-drop entre slots, el grid debe ser 6x4" --json
```

---

## 4. Quantum Bridge (`--quantum`)

### Que Es

Delegacion **paralela dual**: lanza 2 sesiones de DeepSeek simultaneamente, cada una con un "angulo" diferente (perspectiva/enfoque), y luego **mergea** las respuestas en una sola. Ideal para tareas complejas donde dos perspectivas producen mejor resultado.

### Sintaxis

```bash
python run.py --quantum "TAREA" [--template archivo] [--quantum-angles "angulo1,angulo2"] --json
```

### Angulos

**Automaticos** (sin `--quantum-angles`): el sistema detecta el tipo de tarea y asigna angulos:
- `game_full` -> "gameplay,visual"
- `fullstack` -> "frontend,backend"
- `refactor` -> "architecture,implementation"
- `template_split` -> "core_logic,visual_audio"

**Manuales** (con `--quantum-angles`):
```bash
--quantum-angles "logica,render"
--quantum-angles "api,database"
--quantum-angles "core,ui"
```

### Estrategias de Merge (en cascada)

1. **Template-guided**: Si hay template con TODOs, asigna cada TODO al angulo que mejor lo resolvio
2. **Function-based (v4.2)**: Extrae funciones, clases ES6, y variables de ambas respuestas. Deduplica clases y funciones con `pick_better_implementation()` (score por lineas de codigo, control flow, error handling, validacion). Compone: variables → clases → funciones. Threshold: >= 1 simbolo merged para usar esta estrategia.
3. **Raw**: Concatena ambas respuestas eliminando duplicados

### Cuando Usar Quantum vs Delegate Simple

| Escenario | Recomendacion |
|-----------|---------------|
| Tarea simple (<200 lineas resultado) | `--delegate` |
| Feature compleja multi-aspecto | `--quantum` |
| Template con >8 TODOs | `--quantum` (menos truncamiento) |
| Tarea fullstack (frontend+backend) | `--quantum --quantum-angles "frontend,backend"` |
| Correccion de errores | `--delegate --feedback` (no quantum) |

### Ejemplo

```bash
# Juego completo con gameplay + visual
python run.py --quantum "crea un juego de tower defense con oleadas, upgrades y particulas" --template tower-defense.js --json

# Fullstack
python run.py --quantum "crea API REST + frontend React para gestionar tareas" --quantum-angles "api,frontend" --json
```

---

## 5. Multi-Step (`--multi-step`)

### Que Es

Ejecuta un plan de multiples pasos secuenciales y/o paralelos. Cada paso es una delegacion independiente que puede depender de resultados anteriores.

### Sintaxis

```bash
python run.py --multi-step plan.json --json
# O inline:
python run.py --multi-step-inline '{"steps":[...]}' --json
```

### Formato del Plan JSON

```json
{
  "steps": [
    {
      "id": "step1",
      "task": "crea el modelo de datos de Usuario",
      "template": "models/user.ts",
      "validate": true,
      "max_retries": 2
    },
    {
      "id": "step2",
      "task": "crea los endpoints CRUD usando el modelo",
      "template": "routes/users.ts",
      "context_from": ["step1"],
      "validate": true
    },
    {
      "id": "step3a",
      "task": "crea tests unitarios para el modelo",
      "parallel_group": "tests"
    },
    {
      "id": "step3b",
      "task": "crea tests de integracion para los endpoints",
      "context_from": ["step2"],
      "parallel_group": "tests"
    },
    {
      "id": "step4",
      "task": "crea componente React complejo con formulario",
      "dual_mode": true
    }
  ]
}
```

### Campos de StepSpec

| Campo | Tipo | Descripcion |
|-------|------|-------------|
| `id` | string | Identificador unico del paso |
| `task` | string | Descripcion de la tarea |
| `template` | string? | Ruta a template con TODOs |
| `context_from` | string[]? | IDs de pasos cuyo resultado se inyecta como contexto |
| `validate` | bool? | Activar validacion (default: true) |
| `max_retries` | int? | Reintentos (default: 1) |
| `parallel_group` | string? | Pasos con mismo grupo se ejecutan en paralelo |
| `dual_mode` | bool? | Usar Quantum Bridge para este paso |

### Flujo de Ejecucion

1. Agrupa pasos por `parallel_group`
2. Ejecuta grupos secuencialmente, pero pasos dentro del grupo en paralelo
3. `context_from` inyecta el `response` de pasos anteriores como `--context`
4. `dual_mode: true` ejecuta el paso via Quantum Bridge en vez de delegate simple
5. Resultado: JSON con array de resultados por paso

---

## 6. Sistema de Skills 3-Tier

### Arquitectura

DeepSeek Code tiene un sistema de inyeccion automatica de conocimiento en 3 niveles:

**Tier 1 - Core Knowledge** (SIEMPRE inyectado en delegacion, ~15K tokens):
- `programming-foundations` — SOLID, GoF patterns, Clean Code, Error Handling, Async, Testing
- `data-structures-algorithms` — O(), estructuras, algoritmos, spatial
- `common-errors-reference` — 27+ errores comunes con soluciones

**Tier 2 - Domain Skills** (por relevancia de keywords, hasta ~45K tokens):
- Se activan segun las palabras clave de la tarea
- Con 45K tokens de budget, pueden entrar 5-8 skills completas
- Ejemplos: `canvas-2d-reference`, `math-foundations`, `physics-simulation`, `typescript-advanced`, `database-patterns`, `security-auth-patterns`, `backend-node-patterns`, `css-modern-patterns`
- 46 skills mapeadas en SKILL_KEYWORD_MAP

**Tier 3 - Specialist** (si hay espacio, ~20K tokens):
- Skills de nicho que puntuaron pero no entraron en Tier 2
- Con 20K tokens, pueden entrar 2-3 skills adicionales

### Budget de Tokens (1M Contexto Web)

Cada chat de DeepSeek via web tiene **1M tokens de contexto** (cada ventana de chat es independiente, no comparten memoria entre si). El budget de skills esta
diseñado para maximizar conocimiento inyectado dejando amplio espacio para template y respuesta.

```
Modo Delegacion (1M contexto web):
  Core:       15,000 tokens (siempre presente, ~60K chars)
  Domain:     45,000 tokens (por relevancia, ~180K chars)
  Specialist: 20,000 tokens (si hay espacio, ~80K chars)
  TOTAL:      80,000 tokens para skills (~320K chars, 8% del contexto)

  + DELEGATE_SYSTEM_PROMPT:  ~7,000 tokens
  + SurgicalMemory briefing: ~3,000 tokens
  + GlobalMemory briefing:   ~2,000 tokens
  + Template + context:      variable
  = ~92,000 tokens de sistema (~9% del contexto de 1M)
  = ~908,000 tokens disponibles para la respuesta de DeepSeek

Modo Interactivo (1M contexto web): 80,000 tokens para skills
```

**Nota**: Con 1M de contexto via web, el budget de 80K para skills deja espacio masivo.
En tareas con templates grandes, el sistema auto-reduce skills inyectadas
como optimizacion. Para delegaciones simples sin template, el budget completo
funciona bien. DeepSeek es open source y no caduca.

### Skills Disponibles (51 total)

**Fundamentales (Core):** programming-foundations, data-structures-algorithms, common-errors-reference

**Juegos:** canvas-2d-reference, physics-simulation, game-genre-patterns, web-audio-api, sounds-on-the-web, procedural-generation, character-sprite, custom-algorithmic-art

**Web/Frontend:** modern-javascript-patterns, vercel-react-best-practices, server-side-rendering, css-modern-patterns, interface-design, ui-ux-pro-max, web-design-guidelines, building-native-ui, chat-ui

**Backend:** backend-node-patterns, security-auth-patterns, database-patterns, server-management, java-spring-boot

**TypeScript:** typescript-advanced

**3D/GPU:** webgpu, threejs-shaders, webgpu-threejs-tsl

**Otros:** claude-delegate, project-architecture, code-review-excellence, audit-website, launch-strategy, remotion-best-practices, json-canvas, document-chat-interface, official-skill-creator, skill-judge, apollo-mcp-server, cloudflare-mcp-server, audiocraft-audio-generation, manaseed-integration, rivetkit-client-javascript, ue57-rhi-api-migration, create-design-system-rules, custom-web-artifacts-builder, advanced-coding

### Seleccion Semantica de Skills (v2.3)

A partir de v2.3, la seleccion de skills usa **TF-IDF cosine similarity** como metodo primario:

1. `SemanticSkillIndex` vectoriza las descripciones de las 46 skills del keyword map
2. La tarea del usuario se vectoriza con el mismo TFIDFVectorizer
3. Se calculan cosine similarities y se ranquean las top-K skills
4. Si hay historial en GlobalMemory, `search_with_boost()` multiplica similarity * bayesian_success_rate
5. Si TF-IDF no encuentra resultados, fallback al keyword map original

Esto permite que tareas como "construir interfaz responsive" matcheen con `css-modern-patterns` aunque no contengan keywords exactos.

### Crear Skills Nuevas

Las skills son archivos `.skill` (ZIP conteniendo `SKILL.md` con frontmatter YAML).

Para crear una nueva:
1. Crear archivo `.md` con frontmatter `---\nname: mi-skill\ndescription: ...\n---`
2. Empaquetar como ZIP renombrando a `.skill`
3. Colocar en `<DEEPSEEK_DIR>/skills/`
4. Agregar keywords en `src/deepseek_code/skills/skill_constants.py` -> `SKILL_KEYWORD_MAP`
5. El SemanticSkillIndex se reconstruye automaticamente al siguiente uso

---

## 7. Sistema de Memoria Dual

### 7a. SurgicalMemory (Nivel 1 — Por Proyecto)

Sistema de memoria persistente que conecta Claude Code (cirujano/planificador) con DeepSeek Code (asistente/ejecutor). Aprende de cada delegacion para mejorar las siguientes.

**Flujo:**
```
ANTES de delegar:
  pre_delegation() ->
    1. Detecta proyecto (sube directorios buscando CLAUDE.md, .git, package.json)
    2. Carga store JSON del proyecto (%APPDATA%/surgical_memory/{name}_{hash}.json)
    3. Lee CLAUDE.md si existe
    4. Construye briefing con budget de 3000 tokens:
       Prioridad: rules > errors > conventions > architecture > patterns > claude_md

DESPUES de delegar:
  post_delegation() ->
    1. Aprende reglas de errores detectados
    2. Registra en historial: tarea, modo, exito, duracion
    3. Guarda store actualizado a disco
```

**Busqueda Semantica (v2.3):**
```python
# find_relevant(query, section, top_k) — busca entradas relevantes por TF-IDF
store.find_relevant("error innerHTML", "error_log", top_k=5)
# → retorna las 5 entradas mas semanticamente similares a la query
# Fallback Jaccard para corpus <5 entradas
```

**Compactacion Inteligente (v2.3):**
- `error_log` y `failure_analyses` ahora se ordenan por `relevance = temporal_decay(age) * (1 + 0.1*(freq-1))`
- Entradas recientes y frecuentes sobreviven; antiguas e infrecuentes se purgan primero

**Almacenamiento:**
```
%APPDATA%\DeepSeek-Code\surgical_memory\
  LiminalCrown_a1b2c3d4.json   # Store del proyecto LiminalCrown
  MyWebApp_e5f6g7h8.json       # Store de otro proyecto
```

### 7b. GlobalMemory (Nivel 2 — Personal Cross-Proyecto)

Sistema de aprendizaje personal que acumula patrones del desarrollador a traves de TODOS los proyectos. Se vuelve mas inteligente con cada delegacion.

**Lo que aprende (7 dimensiones):**
1. Estilo de codigo: let vs const, camelCase vs snake_case, idioma de comentarios
2. Skills por exito real: rankea las 51 skills por tasa de exito (no solo keywords)
3. Complejidad optima: sweet spot de TODOs por template y tokens input (EMA)
4. Rendimiento por modo: delegate vs quantum vs multi-step (exito y duracion)
5. Errores cross-proyecto: patrones que se repiten en multiples proyectos
6. Keywords de tarea: que temas tienen mejor tasa de exito
7. Recomendaciones automaticas: briefing con skills a usar/evitar

**Briefing inyectado (~2000 tokens max):**
```
== PERFIL PERSONAL DEL DESARROLLADOR (GlobalMemory) ==
ESTILO DE CODIGO:
- Usar let (no const) — 100% preferencia historica
- Naming: camelCase
- Comentarios en espanol

SKILLS RECOMENDADAS (por tasa de exito real):
- canvas-2d-reference: 92% exito (12 usos)
- physics-simulation: 88% exito (8 usos)
EVITAR:
- threejs-shaders (30% exito, trunca 60%)

COMPLEJIDAD OPTIMA:
- TODOs por template: 5 (sweet spot historico)
- Input tokens optimo: ~40,000

RENDIMIENTO POR MODO:
- delegate: 85% exito (50 usos, ~45s promedio)
- quantum: 78% exito (12 usos, ~90s promedio)
== FIN PERFIL PERSONAL ==
```

**Almacenamiento:** `%APPDATA%\DeepSeek-Code\global_memory.json` (unico archivo)

**Bayesian Skill Ranking (v2.3):**
- Cada skill stat ahora incluye `bayesian_mean`, `bayesian_ci_lower`, `bayesian_ci_upper`
- Calculado via `BayesianEstimator(alpha=successes+1, beta=failures+1)`
- Con 2 exitos de 3 intentos: mean=0.6, CI 95%=[0.19, 1.0] (alta incertidumbre)
- Con 20 exitos de 25: mean=0.78, CI 95%=[0.61, 0.95] (confianza alta)

**Semantic Error Clustering (v2.3):**
- `add_cross_error()` ahora compara nuevos errores con existentes via cosine similarity
- Si similarity > 0.6, merge en vez de crear nueva entrada (reduce duplicados semanticos)

**Compactacion automatica:**
- Max 30 skill combos, max 20 cross-project errors, max 50 task keywords
- Purga skills con <2 inyecciones y >90 dias sin uso
- Cross-project errors ordenados por `temporal_decay(age, half_life=60) * count`

### Protocolo 3-fases de inyeccion (v2.6)

```
Fase 1 — System prompt:
  DEEPSEEK_CODE_IDENTITY + DELEGATE_BASE + bloques modulares (~7K tokens, via assemble_delegate_prompt())

Fase 2 — Inyecciones separadas (pending_injections):
  + skill:programming-foundations  (~15K tokens, core skills)
  + skill:domain-relevante         (~45K tokens, domain skills por TF-IDF)
  + memory:surgical-project        (~3K tokens, contexto del proyecto)
  + global:developer-profile       (~2K tokens, perfil personal)
  Total inyecciones: ~80K tokens, cada una como mensaje independiente

Fase 3 — User message limpio:
  build_delegate_prompt(task, template, context, feedback)
```

SessionOrchestrator trackea que inyecciones ya se enviaron a cada sesion,
evitando re-envios en llamadas subsiguientes (~92K tokens ahorrados por reuso).

---

## 8. Modo Interactivo y Agente

### Interactivo (default)

```bash
python run.py
# o
DeepSeekCode.exe
```

Chat con herramientas MCP: filesystem (read/write/edit/list/delete/move/copy), shell, web search, memory, Serena.

**Comandos internos:**
- `/exit` — Exit
- `/skills` — List available skills
- `/agent <goal>` — Run autonomous agent
- `/skill <name> [args]` — Run skill workflow
- `/serena [start|stop|status]` — Control Serena
- `/login` — Web login (PyQt5 WebEngine)
- `/test` — Verify connection
- `/health` — Session health check
- `/account` — Multi-account management
- `/lang` — Change language (English/Spanish/Japanese)
- `/keys` — API key help

### Agente Autonomo (`--agent`)

```bash
python run.py --agent "crea un proyecto React con auth, dashboard y API backend" --json
```

El agente planifica, ejecuta acciones con herramientas, evalua resultados, y adapta su plan. Incluye la palabra `COMPLETADO` cuando termina.

**Serena Auto-Init (v4.2):** En agent mode, Serena se inicializa automaticamente al arrancar — no requiere `/serena start` manual. Las 3 herramientas de code intelligence (`serena_search_for_pattern`, `serena_get_symbols_overview`, `serena_find_symbol`) quedan disponibles para que el agente navegue codigo de forma simbolica sin intervencion del usuario.

---

## 8.5. DeepSeek V3.2 — Auto-Select, Thinking, Chunking

### Auto-Select de Modelo

El sistema selecciona automaticamente el modelo optimo segun la complejidad:

| Nivel de Tarea | Modelo | max_tokens | Cuando |
|---------------|--------|-----------|--------|
| CHAT / SIMPLE | `deepseek-chat` | 1K-2K | Preguntas, conversacion |
| CODE_SIMPLE | `deepseek-chat` | 4K | Fixes simples, typos |
| CODE_COMPLEX | `deepseek-reasoner` | 8K | Sistemas, features grandes |
| DELEGATION | `deepseek-reasoner` | 16K | Via --delegate, templates |

`deepseek-reasoner` da 64K output max con chain-of-thought (vs 8K de deepseek-chat).
Se activa automaticamente — no necesita flags especiales.
Desactivar con `"auto_select_model": false` en config.json.

### Thinking Mode (Web Sessions)

Cuando `"thinking_enabled": true` en config, las sesiones web envian `thinking_enabled: true`
en el payload SSE, activando el razonamiento profundo de DeepSeek para codigo.

### Template Chunking

Templates >30K tokens se dividen automaticamente en chunks por bloques TODO:
1. Detecta `// === TODO 1A: nombre ===` markers
2. Divide en chunks de ~5K tokens cada uno
3. Cada chunk recibe contexto del output anterior
4. Si no hay TODOs, divide por lineas (fallback)

Umbral configurable: `"chunk_threshold_tokens": 30000` en config.json.

### Sesiones Paralelas (Quantum)

Quantum Bridge usa `DualSession` con 2 clientes en paralelo.
La funcion `create_pool_clients(N)` esta disponible en quantum_helpers.py
para futuras implementaciones multi-session, pero actualmente el runner
quantum solo usa 2 sesiones.

```python
# quantum_helpers.py - disponible para uso futuro
from cli.quantum_helpers import create_pool_clients
clients = create_pool_clients(config, mcp_server)  # Default: 5
```

Config `"pool_size": N` (clamp: 2-10) para cuando se integre multi-session.

---

## 8.6. Intelligence Package (v2.2) — 5 Features Revolucionarias

El Intelligence Package convierte a DeepSeek Code de un generador de codigo en un **sistema que aprende, predice y se auto-corrige**. Todas las features son fail-safe: si fallan, el flujo principal continua sin interrupcion.

### Feature 1: Introspective Debugging

Cuando una delegacion falla, en vez de retry ciego, analiza la **causa raiz** correlacionando con el historial de errores del SurgicalMemory.

```
Antes (retry ciego):
  Falla → "intenta de nuevo" → misma falla → frustración

Ahora (root cause analysis):
  Falla → analiza patron → "truncation_many_todos" detectado
       → fix: "Dividir template: TODOs 1-5 en chunk A, 6-10 en chunk B"
       → retry dirigido con feedback especifico → exito
```

Patrones conocidos: `truncation`, `missing_todos`, `innerHTML_violation`, `const_usage`. Se integra automaticamente en el review phase de `collaboration.py` (via `chat_in_session()` con protocolo 3-fases).

### Feature 2: Shadow Learning

Aprende de las **correcciones manuales** del usuario. Despues de cada delegacion exitosa, compara lo que DeepSeek genero con lo que el usuario realmente commiteo (via `git diff`).

```
Ciclo de aprendizaje:
  1. DeepSeek genera codigo sin try/catch
  2. Usuario agrega try/catch manualmente y commitea
  3. Shadow Learning detecta patron: "added_error_handling" (git diff)
  4. Siguiente delegacion: "CORRECCIONES APRENDIDAS: [3x] Usuario siempre agrega try/catch"
  5. DeepSeek genera codigo CON try/catch automaticamente
```

7 clasificadores: error_handling, logging, null_check, type_annotation, let_to_const, const_to_let, added_comments. Tambien detecta renombramientos sistematicos.

### Feature 3: Git Intelligence (MCP Tool)

Detecta y resuelve conflictos de merge usando AI con contexto del proyecto. Expuesto como **MCP tool** `resolve_conflicts` para que Claude Code lo invoque directamente.

```bash
# Detectar conflictos
python -c "from deepseek_code.intelligence.git_intel import detect_conflicts; print(detect_conflicts('.'))"

# Via MCP tool (Claude Code invoca directamente):
# resolve_conflicts(action="detect", project_path="/proyecto")
# resolve_conflicts(action="preview", project_path="/proyecto")
# resolve_conflicts(action="resolve", project_path="/proyecto", auto_apply=true)
```

Acciones: `detect` (lista archivos), `preview` (muestra conflictos), `resolve` (resuelve con heuristica o AI).

### Feature 4: Requirements Pipeline

Parsea un documento de requisitos (markdown/texto) y genera un **plan multi-paso ejecutable** compatible con `--multi-step`.

```bash
# Solo generar plan
python run.py --requirements features.md --json

# Generar plan Y ejecutarlo automaticamente
python run.py --requirements features.md --auto-execute --json
```

Deteccion automatica de:
- **Prioridad**: MUST/REQUIRED → "must", SHOULD → "should", COULD → "could"
- **Dependencias**: "depende de:", "requires:", "after:", "prerequisitos:"
- **Formato**: Headers markdown, listas numeradas, parrafos libres

Genera plan con topological sort + parallel grouping para maxima eficiencia.

### Feature 5: Predictive Intelligence

Analiza memorias existentes (Surgical + Global) para generar un **reporte de salud predictivo**.

```bash
python run.py --health-report --json
# o con proyecto especifico:
python run.py --health-report --project-context ./CLAUDE.md --json
```

Detecta:
- **File risks**: Archivos cerca del limite de 400 LOC (75%+)
- **Error clusters**: Errores recurrentes agrupados por tipo con tendencias
- **Tech debt trends**: failure_rate creciente, duracion aumentando, errores repetidos
- **Risk level**: healthy / warning / critical con recomendaciones priorizadas

**Bayesian Composite Risk (v2.3):**
- `bayesian_risk_score` (0-100): 50% failure rate + 20% trend severity + 30% file/debt risks
- `confidence_intervals`: CI 95% para delegation success rate via Beta posterior
- `trend_slopes`: Mann-Kendall trend direction para duration y error frequency series

### Protocolo de Inyeccion con Intelligence Data (v2.6)

```
Fase 1 — System prompt:
  DEEPSEEK_CODE_IDENTITY + DELEGATE_BASE + bloques modulares (~7K tokens)

Fase 2 — Inyecciones via SessionOrchestrator:
  + skill:core-skills            (~15K tokens, siempre inyectados)
  + skill:domain-detected        (~45K tokens, TF-IDF semantic match)
  + memory:surgical-project      (~3K tokens, proyecto + intelligence data)
  + global:developer-profile     (~2K tokens, perfil personal)
  Total: ~80K tokens en mensajes separados (trackeo por sesion)

Fase 3 — User message limpio (task + template)
```

El briefing de SurgicalMemory ahora incluye seccion 5.5 "Intelligence Data":
- Shadow corrections aprendidas (frecuencia >= 2)
- Reglas de prevencion de failure analyses

---

## 8.7. Semantic Engine (v2.3) — Motor Central de Inteligencia

El Semantic Engine es un modulo **100% Python puro** (zero dependencies) que reemplaza heuristicas basicas con algoritmos reales. Todos los subsistemas lo consumen.

### Componentes del Engine

| Componente | Algoritmo | Uso |
|:----------:|:---------:|:----|
| **TFIDFVectorizer** | TF-IDF con sparse dicts | Vectoriza texto para similaridad semantica |
| **cosine_similarity** | Dot product / magnitudes | Compara vectores TF-IDF (0.0=nada, 1.0=identico) |
| **BayesianEstimator** | Beta(alpha, beta) | Modela exito/fallo con intervalos de confianza |
| **temporal_decay** | `e^(-ln(2) * age/half_life)` | Peso exponencial por edad (reciente > antiguo) |
| **mann_kendall_trend** | Pares concordantes/discordantes | Detecta tendencias monotonicas en series |
| **weighted_score** | Ponderacion multi-factor | Combina metricas con pesos configurables |

### Tokenizacion

```
Input: "sistema de inventario drag-and-drop"
→ lowercase + strip accents
→ unigrams: ["sistema", "de", "inventario", "drag", "and", "drop"]
→ bigrams: ["sistema_de", "de_inventario", "inventario_drag", ...]
→ TF-IDF vector (sparse dict)
```

### Impacto en Subsistemas

**Skills (SemanticSkillIndex):**
- Antes: 46-entry keyword map (match exacto)
- Ahora: TF-IDF cosine similarity sobre descripciones de skills
- `search_with_boost()`: similaridad * bayesian_success_rate (skills con historial de exito suben)
- Fallback a keywords si TF-IDF no encuentra resultados

**SurgicalMemory (find_relevant):**
- Antes: FIFO (primero-entra-ultimo-sale)
- Ahora: `relevance = temporal_decay(age) * (1 + 0.1*(freq-1))`
- `find_relevant(query, section)`: busqueda TF-IDF sobre entradas de memoria
- Compactacion: ordena por relevancia en vez de FIFO
- Fallback Jaccard para corpus pequeños (<5 entradas)

**GlobalMemory (Bayesian):**
- Skill stats: `BayesianEstimator.from_stats(successes, total)` → mean, CI 95%
- Error clustering: merge semantico si cosine_similarity > 0.6
- Compactacion: `temporal_decay(age, half_life=60) * count`

**Predictive Intelligence (Bayesian composite risk):**
- `composite_risk = 0.5*failure_rate + 0.2*trend_severity + 0.3*file_debt_risks` (0-100)
- Confidence intervals via normal approximation del Beta posterior
- Mann-Kendall trend slopes para delegation duration y error frequency
- HealthReport ahora incluye: `bayesian_risk_score`, `confidence_intervals`, `trend_slopes`

### Archivos del Semantic Engine

```
src/deepseek_code/intelligence/semantic_engine.py  (258 LOC) — Motor central
src/deepseek_code/intelligence/predictor_bayesian.py (145 LOC) — Helper Bayesiano
src/deepseek_code/skills/semantic_skill_index.py   (104 LOC) — Indice semantico de skills
```

---

## 9. Patrones de Uso Optimos

### Receta: Generar Codigo de Juego

```bash
# 1. Crear template con TODOs
# 2. Pasar un juego existente similar como contexto
python run.py --delegate "implementa juego SHMUP con oleadas, power-ups y boss" \
  --template shmup-template.js \
  --context existing-game.js \
  --json
```

**Tip:** DeepSeek inyecta automaticamente: canvas-2d-reference, physics-simulation, game-genre-patterns, web-audio-api, math-foundations.

### Receta: Backend Completo

```bash
# Quantum con angulos api + database
python run.py --quantum "crea API REST completa para e-commerce: productos, carrito, ordenes, auth JWT" \
  --quantum-angles "api,database" \
  --json
```

**Tip:** DeepSeek inyecta: backend-node-patterns, security-auth-patterns, database-patterns, typescript-advanced.

### Receta: Feature Multi-Archivo

```bash
# Crear plan JSON
echo '{
  "steps": [
    {"id": "model", "task": "modelo TypeScript de Usuario con Zod schema"},
    {"id": "api", "task": "endpoints Express CRUD", "context_from": ["model"]},
    {"id": "tests", "task": "tests con vitest", "context_from": ["model", "api"]}
  ]
}' > plan.json

python run.py --multi-step plan.json --json
```

### Receta: Debugging/Correccion

```bash
# Primer intento
python run.py --delegate "crea sistema de inventario drag-and-drop" --template inventory.js --json

# Si falla, corregir con feedback
python run.py --delegate "crea sistema de inventario drag-and-drop" \
  --template inventory.js \
  --feedback "el drag no funciona: falta preventDefault en dragover, los items no se renderizan en el slot destino" \
  --json
```

### Receta: Generacion Rapida sin Template

```bash
# Solo una funcion/componente
python run.py --delegate "funcion JavaScript que implementa A* pathfinding en una grid 2D con obstaculos" --json

# Un componente React completo
python run.py --delegate "componente React TypeScript de tabla paginada con sorting, filtering y seleccion multiple" --json
```

---

## 10. Errores Comunes y Soluciones

### Truncamiento (respuesta cortada)

**Sintoma:** `validation.truncated: true`, respuesta incompleta
**Causa:** deepseek-chat tiene limite de 8K output (4K default). deepseek-reasoner soporta hasta 64K output (incluye CoT).
**Soluciones:**
1. Usar `--template` para que solo responda los TODOs (no repita codigo)
2. Reducir numero de TODOs en el template
3. Usar `--quantum` (split en 2 sesiones = doble capacidad)
4. Agregar en la tarea: "funciones de maximo 20 lineas, sin comentarios"

### Validacion Fallida

**Sintoma:** `success: false`, `validation.errors` tiene mensajes
**Causa:** Auto-validador detecto problemas
**Soluciones:**
1. `--max-retries 2` para auto-reintentar con feedback
2. Re-ejecutar con `--feedback "errores especificos"`
3. `--no-validate` si la validacion es falso positivo

### WASM No Encontrado (modo web)

**Sintoma:** Error "WASM not found" o "FileNotFoundError"
**Causa:** `sha3_wasm_bg.wasm` no esta en APPDATA
**Soluciones:**
1. DeepSeek Code lo auto-descarga al conectarse
2. Verificar ruta en config.json `wasm_path`
3. Re-login: `python run.py` -> `/login`

### Credenciales Expiradas

**Sintoma:** Error 401, "Invalid token"
**Causa:** Bearer token o cookies expiraron (tipicamente cada 24-48h)
**Solucion:** Re-login web: `python run.py` -> `/login` -> inicia sesion en DeepSeek

### Security Hook (desarrollo)

**Sintoma:** Write tool rechazado con "Setting innerHTML..."
**Causa:** Hook de seguridad en el proyecto detecta patrones peligrosos en archivos
**Solucion:** No escribir la cadena literal "innerHTML" concatenada en archivos. Usar `textContent`, `DOMParser`, o `document.createElement()`.

---

## 10b. Internationalization (i18n)

DeepSeek Code supports 3 languages: **English** (default), **Spanish**, and **Japanese**.

### Language Selection Flow

1. **First run**: After the ASCII art banner, a language selector appears before login
2. **Subsequent runs**: Language is loaded from `config.json` (`"lang": "en"`)
3. **Runtime change**: Use `/lang` command in interactive mode to switch

### Architecture

- **`cli/i18n.py`**: Central module with `t(key, **kwargs)` function
- **155 translated keys** across EN and ES, 36 for JA
- **Fallback chain**: current_lang → English → raw key
- **Japanese**: 36 most visible keys translated, rest falls back to English
- **Persistence**: `"lang"` field in `config.json`

### Adding Translations

1. Add key-value pairs to `_STRINGS["en"]` and `_STRINGS["es"]` in `cli/i18n.py`
2. Optionally add to `_STRINGS["ja"]` (fallback to English if missing)
3. Use `t("key_name")` in CLI code instead of hardcoded strings
4. For formatted strings: `t("greeting", name="Alice")` with `"greeting": "Hello {name}"`

---

## 11. Estructura del Proyecto DeepSeek Code

```
<DEEPSEEK_DIR>/
├── run.py                           # Entry point -> cli.main
├── src\
│   ├── cli\
│   │   ├── main.py                  # DeepSeekCodeApp, argparse, modes
│   │   ├── i18n.py                  # Internationalization (en/es/ja), t() function
│   │   ├── oneshot.py               # run_delegate_oneshot(), run_agent_oneshot()
│   │   ├── multi_step.py            # run_multi_step() + multi_step_helpers.py
│   │   ├── quantum_runner.py        # run_quantum() -> DualSession
│   │   ├── collaboration.py        # 3-phase protocol: briefing/execution/review + chunking
│   │   ├── commands.py              # Router for /skills, /agent, /serena
│   │   ├── commands_helpers.py      # Helpers: /login, /health, /account, knowledge_skill
│   │   ├── config_loader.py         # load_config(), APPDATA_DIR, SKILLS_DIR
│   │   ├── session_commands.py      # v2.5: /new, /chats, /switch, /close
│   │   ├── chat_manager.py          # v2.5: Chat lifecycle management
│   │   ├── onboarding.py            # First-time wizard with language selector
│   │   └── ui_theme.py              # Terminal UI rendering
│   └── deepseek_code\
│       ├── client\
│       │   ├── deepseek_client.py   # DeepSeekCodeClient (web + API)
│       │   ├── session_chat.py      # v2.5: chat_in_session() + Phase 3 sanitization
│       │   ├── api_caller.py        # Auto-select model, max_tokens, build_api_params
│       │   ├── ai_protocol.py       # AI-to-AI protocol (negotiate, briefing, review)
│       │   ├── task_classifier.py   # TaskLevel enum, classify_task()
│       │   ├── prompt_builder.py    # Build adaptive system prompts
│       │   ├── context_manager.py   # Token estimation + progressive summarization
│       │   └── template_chunker.py  # Smart chunking por TODOs/lineas
│       ├── agent\
│       │   ├── engine.py            # AgentEngine (loop autonomo)
│       │   └── prompts.py           # DELEGATE_SYSTEM_PROMPT, build_delegate_prompt()
│       ├── skills\
│       │   ├── loader.py            # SkillLoader (cache + load_multiple)
│       │   ├── skill_injector.py    # build_delegate_skills_context() 3-tier + semantic
│       │   ├── skill_constants.py   # CORE_SKILLS, SKILL_KEYWORD_MAP, budgets
│       │   └── semantic_skill_index.py # v2.3: TF-IDF semantic skill matching
│       ├── quantum\
│       │   ├── dual_session.py      # DualSession.parallel_chat()
│       │   ├── angle_detector.py    # Auto-deteccion de angulos
│       │   ├── merge_engine.py      # 3 estrategias de merge (v4.2: class dedup)
│       │   ├── merge_helpers.py     # extract_functions, extract_classes, extract_variable_declarations, deduplicate_lines
│       │   ├── quantum_runner.py    # Orquestador quantum
│       │   └── quantum_helpers.py   # create_shared_mcp_server, create_client, create_pool_clients
│       ├── surgical\
│       │   ├── integration.py       # pre_delegation(), post_delegation()
│       │   ├── store.py             # SurgicalStore (JSON persistente)
│       │   ├── collector.py         # detect_project_root(), extract_claude_md()
│       │   ├── injector.py          # build_briefing() con budget tokens
│       │   └── learner.py           # learn_from_delegation()
│       ├── global_memory\
│       │   ├── global_store.py      # GlobalStore (JSON cross-proyecto)
│       │   ├── global_learner.py    # Extraccion de patrones personales
│       │   ├── global_injector.py   # Briefing perfil personal (2K tokens)
│       │   └── global_integration.py # Fachada fail-safe pre/post
│       ├── delegate\
│       │   ├── bridge_utils.py      # Utilidades compartidas
│       │   └── delegate_validator.py # Validacion de respuestas
│       ├── sessions\                # v2.5-2.6: Sesiones persistentes
│       │   ├── session_store.py     # SessionStore + ChatSession (JSON persistente)
│       │   ├── session_namespace.py # build_session_name(), slugify(), parse_session_name()
│       │   ├── session_orchestrator.py # SessionOrchestrator (3-phase flow, v4.2: skills_dir fallback)
│       │   ├── summary_engine.py    # Auto-summary + topic extraction
│       │   └── knowledge_transfer.py # Cross-session knowledge transfer
│       ├── intelligence\
│       │   ├── semantic_engine.py   # v2.3: Motor central (TF-IDF, Bayesian, decay)
│       │   ├── predictor_bayesian.py # v2.3: Composite risk, trends, confidence
│       │   ├── debugger.py          # Introspective debugging (root cause analysis)
│       │   ├── shadow_learner.py    # Shadow learning (git diff corrections)
│       │   ├── git_intel.py         # Git intelligence (conflict resolution)
│       │   ├── requirements_parser.py # Requirements pipeline (doc → plan)
│       │   ├── predictor.py         # Predictive intelligence (health reports)
│       │   └── integration.py       # Fachada fail-safe
│       ├── tools\                   # 15 herramientas MCP (incluye resolve_conflicts)
│       ├── server\                  # MCPServer protocol
│       ├── security\                # RateLimiter, sandbox
│       ├── serena\                  # SerenaManager + native tools
│       └── auth\                    # web_login.py (PyQt5)
├── skills\                          # 51 archivos .skill (ZIP con SKILL.md)
└── dist\                            # DeepSeekCode.exe (PyInstaller)
```

---

## 12. Gestion Estrategica de Tokens (1M Budget Web)

### Contexto: 1M Tokens Disponibles por Chat

Cada chat de DeepSeek via web ofrece **1,000,000 tokens de contexto** (cada ventana es independiente). La clave
es inyectar el conocimiento justo para la tarea, equilibrando skills con espacio
para template y respuesta.

### Desglose del Consumo por Delegacion

Cada delegacion consume tokens en estas categorias:

```
COMPONENTE                   TOKENS ESTIMADOS    FORMULA
─────────────────────────────────────────────────────────
DELEGATE_SYSTEM_PROMPT       ~7,000              len(texto) / 3.5
Skills Tier 1 (Core)        ~5,000-15,000       depende de 3 skills
Skills Tier 2 (Domain)      ~5,000-45,000       depende de relevancia
Skills Tier 3 (Specialist)  ~0-20,000           si hay espacio
SurgicalMemory briefing     ~500-3,000          budget max 3000
GlobalMemory briefing       ~0-2,000            budget max 2000
Template (si existe)         variable            len(template) / 3.5
Context file (si existe)     variable            len(context) / 3.5
User prompt                  ~100-500            len(task) / 3.5
─────────────────────────────────────────────────────────
TOTAL SISTEMA:              ~20,000-92,000       (2-9% del contexto de 1M)
DISPONIBLE RESPUESTA:       ~908,000-980,000     (91-98% del contexto de 1M)
```

### JSON de Respuesta con token_usage

El JSON de delegacion incluye un campo `token_usage` con el desglose:

```json
{
  "success": true,
  "response": "...",
  "token_usage": {
    "system_prompt": 7000,
    "skills_injected": 35000,
    "surgical_briefing": 1200,
    "global_briefing": 500,
    "template": 3000,
    "context_file": 0,
    "user_prompt": 250,
    "total_input": 46950,
    "response_estimated": 8500,
    "total_estimated": 55450,
    "context_remaining": 953050,
    "context_used_percent": "4.7%"
  }
}
```

### Formulas de Estimacion

DeepSeek Code usa dos formulas segun el componente:
- **skill_injector**: `len(text) // 4` (conservador, ~4 chars/token)
- **context_manager**: `math.ceil(len(text) / 3.5)` (mas preciso)

Para calcular manualmente: `tokens ≈ caracteres / 3.5`

### Optimizacion del Budget

**Regla general**: Equilibrar skills inyectadas con espacio para respuesta.
Con 1M de contexto web, hay espacio generoso pero conviene ser eficiente.

- **Tarea simple** (1 funcion): Core + 1-2 Domain = ~25K input, ~975K para respuesta
- **Tarea media** (1 archivo): Core + 3-4 Domain = ~40K input, ~960K para respuesta
- **Tarea compleja** (multi-aspecto): Core + 5-8 Domain = ~60K input, ~940K para respuesta
- **Quantum** (2 sesiones): Cada sesion tiene su propio contexto de 1M

---

## 12b. Flujo Token-Eficiente (v2.4) — Pipe Directo a Disco

### Problema

Cuando Claude delega a DeepSeek y luego reescribe el codigo con Write, los tokens de
salida se duplican: DeepSeek genera ~5000 tokens, Claude los recibe Y los vuelve a
emitir. Esto desperdicia ~96% de los tokens de Claude en re-escritura mecanica.

### Solucion: save_response.py

El helper `deepseek_code.tools.save_response` intercepta la salida JSON de DeepSeek y
la guarda directo al disco. Claude solo recibe metadata ligera (~200 tokens).

```
ANTES (ineficiente):
  DeepSeek → [5000 tok codigo] → Claude → [5000 tok Write] → disco
  Total Claude: ~10,000 tokens

AHORA (eficiente):
  DeepSeek → [5000 tok codigo] → pipe → disco
  Claude recibe: ~200 tok metadata
  Total Claude: ~400 tokens (ahorro 96%)
```

### Uso por Modo

**Delegacion (archivo unico):**
```bash
cd <DEEPSEEK_DIR> && python run.py --delegate "TAREA" --json 2>/dev/null | \
    python -m deepseek_code.tools.save_response --output "ruta/archivo.ext" --preview 5
```

**Delegacion (multi-archivo auto-split):**
```bash
cd <DEEPSEEK_DIR> && python run.py --delegate "TAREA" --json 2>/dev/null | \
    python -m deepseek_code.tools.save_response --split --dir "ruta/proyecto/" --preview 3
```

**Quantum:**
```bash
cd <DEEPSEEK_DIR> && python run.py --quantum "TAREA" --json 2>/dev/null | \
    python -m deepseek_code.tools.save_response --output "ruta/archivo.ext" --preview 5
```

**Multi-step:**
```bash
cd <DEEPSEEK_DIR> && python run.py --multi-step plan.json --json 2>/dev/null | \
    python -m deepseek_code.tools.save_response --split --dir "ruta/proyecto/" --preview 3
```

**Solo metadata (sin guardar):**
```bash
cd <DEEPSEEK_DIR> && python run.py --delegate "TAREA" --json 2>/dev/null | \
    python -m deepseek_code.tools.save_response --meta-only
```

### Opciones de save_response

| Flag | Descripcion |
|------|-------------|
| `--output FILE` | Guarda respuesta completa en un archivo |
| `--split` | Auto-detecta limites de archivos y separa |
| `--dir DIR` | Directorio base para `--split` (default: `.`) |
| `--meta-only` | Solo imprime metadata, descarta respuesta |
| `--preview N` | Incluye primeras N lineas en la metadata |

### Metadata de Salida

Claude recibe un JSON ligero como este:

```json
{
  "success": true,
  "mode": "delegate",
  "duration_s": 45.2,
  "continuations": 0,
  "response_lines": 380,
  "response_chars": 12400,
  "files_written": 1,
  "saved_to": "ruta/archivo.ext",
  "preview": "<!DOCTYPE html>\n<html>..."
}
```

### Regla para Claude

**NUNCA uses Write para guardar codigo generado por DeepSeek.** Siempre usa pipe
directo. Solo usa Edit para correcciones quirurgicas despues de guardar.

---

## 13. Referencia Rapida de Comandos

```bash
# ═══ FLUJO EFICIENTE (v2.4 — RECOMENDADO) ═══

# Delegacion → archivo unico
python run.py --delegate "TAREA" --json 2>/dev/null | \
    python -m deepseek_code.tools.save_response -o "ruta/archivo.ext" --preview 5

# Delegacion → multi-archivo
python run.py --delegate "TAREA" --json 2>/dev/null | \
    python -m deepseek_code.tools.save_response --split -d "ruta/proyecto/"

# Quantum → archivo unico
python run.py --quantum "TAREA" --json 2>/dev/null | \
    python -m deepseek_code.tools.save_response -o "ruta/archivo.ext" --preview 5

# Multi-step → multi-archivo
python run.py --multi-step plan.json --json 2>/dev/null | \
    python -m deepseek_code.tools.save_response --split -d "ruta/proyecto/"

# ═══ FLUJO DIRECTO (cuando necesitas ver el codigo) ═══

python run.py --delegate "TAREA" --json
python run.py --delegate "TAREA" --template FILE --json
python run.py --delegate "TAREA" --feedback "ERRORES" --json
python run.py --quantum "TAREA" --quantum-angles "a,b" --json
python run.py --multi-step plan.json --json

# ═══ INTELLIGENCE PACKAGE (v2.2) ═══

python run.py --requirements features.md --json
python run.py --requirements features.md --auto-execute --json
python run.py --health-report --json
python run.py --health-report --project-context ./CLAUDE.md --json

# ═══ OTROS ═══

python run.py -q "pregunta rapida" --json
python run.py --agent "meta autonoma" --json
python run.py                            # Interactivo

# ═══ FLAGS GLOBALES ═══

--max-retries N       # Reintentos (default: 1)
--no-validate         # Sin validacion
--project-context X   # CLAUDE.md del proyecto
--config X            # Config alternativa
--json                # Output JSON
--negotiate-skills    # DeepSeek elige sus skills
--requirements X      # v2.2: Documento de requisitos
--auto-execute        # v2.2: Ejecutar plan auto-generado
--health-report       # v2.2: Reporte de salud predictivo
```

---

## 13c. Sesiones Persistentes y Knowledge Transfer (v2.5-v2.6)

### Concepto Central

Cada chat con DeepSeek es una **sesion** con nombre, ID persistente y estado. El Session Orchestrator envia el system prompt y skills SOLO la primera vez (Phase 1/2), luego reutiliza el chat existente — ahorrando ~99.8% de tokens en llamadas subsecuentes.

### Arquitectura de 3 Fases

```
Phase 1: System prompt → DeepSeek responde "OK" (SOLO primera vez)
Phase 2: Inyecciones (skills, memoria, knowledge) → cada una con ACK rastreado
Phase 3: Mensaje del usuario LIMPIO (sin instrucciones de acknowledgment)
```

**REGLA CRITICA:** Phase 3 NUNCA debe contener frases como "di solo OK" o "responde unicamente OK". El sistema sanitiza automaticamente estas frases si se detectan.

### SessionStore — Persistencia

```python
# Modulo: deepseek_code.sessions.session_store
store = SessionStore(path)           # Carga/crea archivo JSON
session = store.create(name, chat_id) # Nueva sesion
session = store.get(name)            # Recuperar sesion existente
store.update(name, parent_message_id=N, add_context='skill:X')
store.update_summary(name, topic='JWT Auth', summary='Designed login')
store.close(name)                    # Cerrar sesion
store.close_all()                    # Cerrar todas
digest = store.get_session_digest(name) # Resumen para routing
```

Campos clave de `ChatSession`:
- `chat_session_id`: ID del chat en DeepSeek
- `parent_message_id`: Encadena mensajes (2 → 4 → 6...)
- `mode`: delegate | converse | quantum
- `injected_contexts`: Set de contextos ya inyectados
- `message_count`: Contador de mensajes enviados
- `topic`, `summary`: Auto-generados para routing inteligente

### Session Namespace

```python
# Modulo: deepseek_code.sessions.session_namespace
build_session_name('delegate', 'auth-module')  # → 'delegate:auth-module'
parse_session_name('delegate:auth-module')      # → ('delegate', 'auth-module', 'delegate:auth-module')
slugify('My Task Name!')                        # → 'my-task-name'
```

### Knowledge Transfer (v2.6)

Permite transferir conocimiento entre sesiones sin re-enviar todo el contexto:

```python
# Modulo: deepseek_code.sessions.knowledge_transfer
knowledge = extract_knowledge(store, 'delegate:source')
transferable = list_transferable_sessions(store)
injection = transfer_knowledge(store, 'delegate:source', 'converse:target')
# injection = {'type': 'knowledge', 'content': '...', ...}
```

Tracking bidireccional:
- `session.knowledge_sent_to` — lista de sesiones a las que envio conocimiento
- `session.knowledge_received_from` — lista de sesiones de las que recibio

### Session Orchestrator (v2.6)

Orquesta todo el flujo de sesiones:

```python
# Modulo: deepseek_code.sessions.session_orchestrator
orch = SessionOrchestrator(store, skills_dir, appdata_dir)
call = orch.prepare_session_call(
    mode='delegate', identifier='auth',
    user_message='create login', base_system_prompt='...',
    task_text='create login form'
)
# call = {session_name, system_prompt, pending_injections, ...}
routing = orch.get_routing_digest()
# routing = {active_sessions: [...], ...}
```

### Summary Engine

Auto-genera resumenes de sesiones para routing inteligente:

```python
# Modulo: deepseek_code.sessions.summary_engine
summary = generate_local_summary(session, user_message, response)
update_session_summary(store, name, user_msg, response, force=False)
# Nota: requiere message_count >= 2 a menos que force=True
```

### CLI de Sesiones

```bash
# Desde plugin (Claude Code)
--session "nombre"        # Usar/crear sesion con nombre
--session-list            # Listar sesiones activas
--session-close "nombre"  # Cerrar sesion
--session-close-all       # Cerrar todas
--session-digest "nombre" # Obtener resumen para routing
--transfer-from "source"  # Transferir conocimiento de otra sesion

# Desde CLI interactiva
/new [nombre]    # Crear nuevo chat
/chats           # Listar chats activos
/switch <nombre> # Cambiar a otro chat
/close [nombre]  # Cerrar chat
```

### Flujo de Token Savings

```
Primera llamada:  System prompt (~3K tokens) + Skills (~5K) + Task
Segunda llamada:  Solo Task (~100 tokens) ← 99.8% ahorro
Tercera llamada:  Solo Task (~100 tokens) ← sesion reutilizada
```

---

## 14. Decision Tree: Que Modo Usar

```
Tarea de codigo para DeepSeek?
├── Una sola funcion/componente/archivo?
│   ├── Simple (<100 lineas esperadas) -> --delegate "tarea" --json
│   ├── Con estructura predefinida     -> --delegate "tarea" --template X --json
│   └── Corregir intento anterior      -> --delegate "tarea" --feedback "..." --json
│
├── Tarea compleja multi-aspecto?
│   ├── Frontend + Backend             -> --quantum --quantum-angles "frontend,backend"
│   ├── Logica + Visual                -> --quantum --quantum-angles "logica,visual"
│   └── Template con >8 TODOs         -> --quantum --template X
│
├── Multiples archivos independientes?
│   └── Crear plan.json con steps      -> --multi-step plan.json --json
│
├── Documento de requisitos?
│   ├── Solo generar plan              -> --requirements doc.md --json
│   └── Generar y ejecutar plan        -> --requirements doc.md --auto-execute --json
│
├── Diagnostico del proyecto?
│   └── Detectar tech debt y riesgos   -> --health-report --json
│
├── Conflictos de merge?
│   └── MCP tool resolve_conflicts     -> detect/preview/resolve
│
└── Tarea autonoma con herramientas?
    └── Necesita leer/escribir archivos -> --agent "meta" --json
```
