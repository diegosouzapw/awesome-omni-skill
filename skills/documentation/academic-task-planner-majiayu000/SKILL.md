---
name: academic-task-planner
description: Transform academic course assignment PDFs into structured, actionable markdown checklists with dates, unique IDs, and custom tags. Asks when the user will start, assigns tasks only to weekdays (Monday-Friday), respects weekends automatically, and calculates forum deadlines 3 days before due date. Use this skill when the user uploads academic PDFs or asks to create a task plan from course assignments.
---

# Academic Task Planner Skill

## Purpose
Transform academic course assignment PDFs into structured, actionable markdown checklists with dates, unique IDs, and custom tags. Assigns tasks to weekdays only, respecting real student schedules.

## When to Use This Skill
- User uploads a course assignment guide PDF (like UNAD's "Guía de aprendizaje")
- User requests a task breakdown or planning for an academic assignment
- User asks to create a checklist from an academic document

## Core Workflow

### Step 1: PDF Analysis
When a PDF is uploaded or referenced:
1. Read the complete PDF using `read_pdf_content` tool
2. Extract key information:
   - **Course name and code**
   - **Assignment title**
   - **Official start date** (when assignment opens)
   - **Due date** (final deadline)
   - **Points/weight**
   - **Deliverables** (what must be submitted)
   - **Required steps** (what must be done)
   - **Platform requirements** (where to submit)
   - **Forum participation requirements** (if any)

### Step 2: Interactive Data Collection

Ask the user TWO questions in sequence:

#### Question 1: Start Date
```
He analizado tu guía. Información encontrada:
- **Curso:** [Course Name] ([Code])
- **Tarea:** [Assignment Title]
- **Fecha oficial de apertura:** [Official Start Date]
- **Fecha de entrega:** [Due Date]
- **Valor:** [Points] puntos

¿Cuándo vas a iniciar esta tarea?
(Puedes usar la fecha oficial o indicar otra fecha si prefieres empezar después)

Formato: YYYY-MM-DD o "mañana" o "próximo lunes", etc.
```

Wait for user response. Parse the date they provide.

#### Question 2: Tags
```
Perfecto, iniciarás el [parsed start date].

¿Qué tags quieres agregar a cada tarea?
Ejemplo: #estudio #cibercultura #universidad

Por favor proporciona los tags separados por espacios:
```

Wait for user response before proceeding.

### Step 3: Calculate Critical Dates

Based on the extracted dates and user input:
1. **User start date**: The date the user will actually begin working
2. **Official due date**: Final submission deadline from PDF
3. **Forum deadline**: **3 days before the final due date**
4. **Available working days**: Calculate weekdays between start and due date

### Step 4: Weekend Avoidance Logic

**CRITICAL RULE: Avoid weekends for regular tasks**

When assigning dates to tasks:

✅ **DO assign to weekdays:**
- Monday through Friday for ALL regular tasks
- Reading, research, creation, editing, review tasks

❌ **DO NOT assign to weekends (Saturday/Sunday)** EXCEPT:
- Forum participation deadline (if it falls on weekend)
- Final submission deadline (if it falls on weekend)
- These are immovable dates from the PDF

**Date Assignment Algorithm:**

```python
# Pseudo-code for date assignment
current_date = user_start_date

for task in tasks:
    if task.is_critical_deadline:  # Forum or final submission
        task.date = task.fixed_deadline_date  # Use exact date even if weekend
    else:
        # Assign to next available weekday
        while current_date is weekend:
            current_date = skip_to_next_monday()

        task.date = current_date
        current_date = next_day(current_date)
```

**Weekend Handling Examples:**

```
❌ WRONG:
- [ ] Leer artículo... 📅 2025-11-02 (Saturday)

✅ CORRECT:
- [ ] Leer artículo... 📅 2025-11-04 (Monday)

✅ EXCEPTION - Critical deadline on weekend:
- [ ] Publicar en foro (FECHA LÍMITE) 📅 2025-11-23 (Saturday) ⚠️

✅ EXCEPTION - Final submission on weekend:
- [ ] Subir documento Word (ENTREGA FINAL) 📅 2025-11-24 (Sunday) ⚠️
```

### Step 5: Task Breakdown Strategy

Create tasks with **moderate granularity**:
- **Not too detailed**: Avoid micro-tasks like "open document" or "click button"
- **Not too broad**: Avoid vague tasks like "complete entire assignment"
- **Balance**: Each task should take 30 minutes to 2-3 hours to complete

#### Task Categories and Typical Breakdown:

**Reading/Research Phase:**
- One task per major reading material
- One task for exploring digital resources (OVAs, videos, etc.)
- One task for reviewing previous work (if applicable)

**Creation Phase:**
For each deliverable, create 3-5 tasks:
- Planning/outlining
- Content creation
- Visual/technical elements
- Review and refinement
- Finalization and testing links/access

**Forum Participation:**
- One task: share main deliverable in forum
- One task: comment on peer work
- One task: capture screenshots for evidence
- **Note**: Forum deadline date is FIXED even if it's a weekend

**Final Documentation:**
- One task: write reflection
- One task: format bibliography
- One task: assemble final document
- One task: quality check (spelling, links, format)
- One task: plagiarism check (Turnitin/similar)
- One task: submit to platform
- **Note**: Final submission date is FIXED even if it's a weekend

### Step 6: Generate Markdown Structure

```markdown
# ✅ CHECKLIST [ASSIGNMENT NAME]

**Periodo:** [user start date] - [due date]
**Valor:** [points] puntos
**⚠️ Fecha límite foro:** [3 days before due date] [⚠️ add if weekend]
**⚠️ Fecha de entrega:** [due date] [⚠️ add if weekend]

---

## 📚 PHASE 1: [Phase Name] ([date range - weekdays only])

### 📖 [Section Name]
- [ ] [Task description] 📅 YYYY-MM-DD (Weekday) 🆔 [6-char-id] [user-tags]
- [ ] [Task description] 📅 YYYY-MM-DD (Weekday) 🆔 [6-char-id] [user-tags]

### 🎯 [Section Name]
- [ ] [Task description] 📅 YYYY-MM-DD (Weekday) 🆔 [6-char-id] [user-tags]

---

## 🗺️ PHASE 2: [Phase Name] ([date range - weekdays only])

[Continue pattern...]

---

## 💬 PARTICIPACIÓN EN FORO (⚠️ FECHA LÍMITE)

- [ ] [Task] 📅 [Forum Deadline - may be weekend] ⚠️ 🆔 [id] [tags]
- [ ] [Task] 📅 [Forum Deadline - may be weekend] ⚠️ 🆔 [id] [tags]

---

## 📝 ENTREGA FINAL (⚠️ FECHA LÍMITE)

- [ ] [Task] 📅 [Due Date - may be weekend] ⚠️ 🆔 [id] [tags]
- [ ] [Task] 📅 [Due Date - may be weekend] ⚠️ 🆔 [id] [tags]

---

## 📋 VERIFICACIÓN FINAL PRE-ENTREGA

- [ ] [Checklist item] 📅 [due date or day before] 🆔 [id] [tags]

---

**Total de tareas:** [count]
**Curso:** [course name] ([code])
**Días laborables disponibles:** [weekday count]
```

## ID Generation Rules

Generate unique 6-character IDs using this pattern:
- Mix of lowercase letters and numbers
- Format: `[digit][letter][letter][letter][digit][digit]`
- Examples: `0zwo06`, `8kj3m1`, `5tyu89`
- Ensure each ID is unique within the document

## Date Assignment Strategy

### Phase 1: Calculate Available Days
1. Count weekdays between user start date and due date
2. Identify if forum/submission deadlines fall on weekends
3. Reserve those critical dates

### Phase 2: Distribute Tasks
1. **Start with user-provided start date**
2. **Assign only to weekdays (Mon-Fri)** for regular tasks
3. **Skip weekends** - jump from Friday to next Monday
4. **Group related tasks** on same or consecutive days
5. **Reserve last 2-3 weekdays** before deadline for final tasks

### Phase 3: Mark Critical Dates
- Add ⚠️ symbol to tasks that fall on weekends (forum/submission only)
- Clearly indicate these are immovable deadlines

### Example Calendar Progression:

```
Week 1:
Mon 2025-10-30: Task 1
Tue 2025-10-31: Task 2
Wed 2025-11-01: Task 3
Thu 2025-11-02: Task 4
Fri 2025-11-03: Task 5
[SKIP WEEKEND]
Mon 2025-11-04: Task 6
...

Final Week:
Thu 2025-11-21: Task 38
Fri 2025-11-22: Task 39
Sat 2025-11-23: ⚠️ FORO DEADLINE (Exception)
Sun 2025-11-24: ⚠️ FINAL SUBMISSION (Exception)
```

## Task Quantity Guidelines

Target task counts by assignment duration (counting weekdays only):
- **1-2 weeks (10-14 weekdays):** 15-25 tasks
- **3-4 weeks (15-20 weekdays):** 30-50 tasks
- **5+ weeks (25+ weekdays):** 50-70 tasks

Adjust based on complexity, not just duration.

## Special Considerations

### Forum Participation
Always include:
- Calculate deadline: **exactly 3 days before final due date**
- If forum deadline falls on weekend: Keep the date, add ⚠️ symbol
- Minimum tasks:
  1. Post main content/link
  2. Comment on at least one peer
  3. Capture evidence (screenshots)

### Final Submission
Always include:
- If final deadline falls on weekend: Keep the date, add ⚠️ symbol
- Prepare final tasks for the weekday(s) immediately before
- Include clear warning about the weekend deadline

### Document Assembly
Final phase tasks (assign to last 2-3 weekdays before deadline):
- Cover page creation
- All deliverable links
- Evidence captures
- Reflection/learning
- Bibliography/references
- Format review
- Link verification
- Plagiarism check
- Platform submission

### Platform-Specific Tasks
If the PDF mentions:
- **Miro**: Include account creation and public sharing
- **Video/Reel**: Include filming, editing, publishing, link copying
- **Presentations**: Include slide creation, design, and export
- **Turnitin**: Include upload and similarity report review

## Output Format

1. **Always create the markdown file** in `/mnt/user-data/outputs/`
2. **Filename format**: `[CourseName]_Tarea[Number]_Checklist.md`
   - Example: `Cibercultura_Tarea4_Checklist.md`
3. **Use the user's provided tags** on every single task
4. **Include weekday count** at the bottom
5. **Mark weekend deadlines** with ⚠️ symbol

## Quality Checklist

Before delivering, verify:
- [ ] User was asked for start date
- [ ] User was asked for tags
- [ ] All dates are in YYYY-MM-DD format
- [ ] Regular tasks are assigned ONLY to weekdays (Mon-Fri)
- [ ] Weekend dates appear ONLY for critical deadlines (forum, submission)
- [ ] Critical weekend deadlines have ⚠️ symbol
- [ ] Forum deadline is exactly 3 days before due date
- [ ] All IDs are unique and follow the pattern
- [ ] User's tags appear on every task
- [ ] Tasks are moderately granular (not too detailed, not too vague)
- [ ] Phases are logically organized by type of work
- [ ] Final verification section is included
- [ ] Weekday count is mentioned in document
- [ ] File is saved in outputs directory

## Example Interaction

**User:** *uploads PDF*
"Crea un plan de tareas para esta guía"

**Assistant:**
"Perfecto, voy a analizar tu guía de aprendizaje y crear un checklist organizado.

Primero, déjame revisar el documento..."

*[reads PDF and extracts info]*

"He encontrado:
- **Curso:** Cibercultura (403037)
- **Tarea:** Tarea 4 - Ser digital
- **Fecha oficial de apertura:** 24 de octubre de 2025
- **Fecha de entrega:** 24 de noviembre de 2025
- **Valor:** 120 puntos

**¿Cuándo vas a iniciar esta tarea?**
(Puedes usar la fecha oficial del 24 de octubre, o indicar otra fecha si prefieres empezar después)

Formato: YYYY-MM-DD o 'mañana' o 'próximo lunes', etc."

**User:** "30 de octubre"

**Assistant:**
"Perfecto, iniciarás el 30 de octubre de 2025.

Eso te da 18 días laborables (lunes a viernes) para completar la tarea.

**¿Qué tags quieres agregar a cada tarea?**
Ejemplo: #estudio #cibercultura #unad

Por favor proporciona los tags separados por espacios:"

**User:** "#estudio #cibercultura"

**Assistant:**
"Excelente, voy a crear tu checklist con:
- Inicio: 30 de octubre (miércoles)
- Tags: #estudio #cibercultura
- Tareas asignadas solo en días laborables (lunes a viernes)
- **Fecha límite del foro:** 21 de noviembre (viernes)
- **Fecha de entrega final:** 24 de noviembre (lunes)

Organizaré las tareas en 4 fases principales:
1. Semana 1: Lectura y comprensión (6 tareas)
2. Semana 2: Creación del mapa mental (6 tareas)
3. Semana 3: Producción del reel (8 tareas)
4. Semana 4: Documentación y entrega (16 tareas)

Nota: He evitado asignar tareas en fines de semana. Solo las fechas límite del foro y entrega final pueden caer en fin de semana si así están establecidas.

Generando tu checklist..."

*[creates markdown file]*

"[View Cibercultura_Tarea4_Checklist.md](computer:///mnt/user-data/outputs/Cibercultura_Tarea4_Checklist.md)

He creado tu checklist con 43 tareas distribuidas en 18 días laborables. Todas las tareas están asignadas a días de lunes a viernes, excepto las fechas límite obligatorias. ¡Éxito con tu tarea!"

## Error Handling

**If user provides ambiguous start date:**
- Parse natural language ("mañana", "próximo lunes", "en 3 días")
- Confirm the calculated date with user before proceeding

**If start date is after official start date:**
- Acknowledge: "Iniciarás [X] días después de la apertura oficial"
- Recalculate available time
- Warn if time is tight

**If start date creates impossible timeline:**
- Alert user: "Con esta fecha de inicio, solo tendrás [X] días laborables, lo cual puede ser muy ajustado"
- Ask if they want to adjust or proceed anyway

**If PDF is unclear or missing information:**
- Clearly state what information is missing
- Ask user to provide the missing details
- Proceed with reasonable defaults if user confirms

**If critical deadline falls on weekend:**
- Keep the exact date
- Add ⚠️ warning symbol
- Mention it in the summary

## Tips for Best Results

1. **Read the entire PDF** before starting to break down tasks
2. **Always ask for start date** - don't assume they start on official opening date
3. **Strictly avoid weekends** for regular tasks - students work Mon-Fri
4. **Preserve critical dates** even if they're weekends - deadlines are deadlines
5. **Identify ALL deliverables** - don't miss any requirements
6. **Look for evaluation criteria** - these hint at important tasks
7. **Note any sequencing requirements** - some tasks must be done before others
8. **Check for technical requirements** - platforms, formats, tools
9. **Always ask for tags** - don't assume or skip this step
10. **Count available weekdays** - this determines realistic task distribution

## Maintenance Notes

This skill should be updated if:
- User feedback indicates tasks are too granular or too broad
- Weekend avoidance logic needs refinement
- New common assignment types emerge (e.g., podcast creation, AR projects)
- Platform requirements change (e.g., new submission systems)
- Date calculation logic needs adjustment

---

**Version:** 2.0
**Created:** October 2025
**Last Updated:** October 2025 (Added start date question + weekend avoidance)
**Skill Type:** Academic Planning & Organization
