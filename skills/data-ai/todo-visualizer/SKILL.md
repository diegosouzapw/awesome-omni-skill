---
name: todo-visualizer
description: Read local Markdown TODO files (supports Chinese/English), analyze task completion status, and generate beautiful single-page HTML dashboards with timeline visualization, interactive task lists, and persistent progress tracking.
license: MIT
compatibility: universal
---

# Todo Visualizer Agent Skill

## When to Use

Use this skill when the user wants to:
- Visualize TODO files in an interactive dashboard
- Track task completion progress
- Generate HTML reports from markdown TODOs (especially for learning roadmaps)
- Analyze timelines containing weeks/days (supports "Week/Day" and "第X周/第X天" formats)

## How to Use

Follow these steps to generate an HTML dashboard from a TODO file:

### Step 1: Read the TODO File

Use the `Read` tool to read the markdown TODO file path provided by the user.

### Step 2: Analyze the Content

Use LLM analysis to extract and structure the information.
**IMPORTANT**: You should check the `examples/` directory for reference formats if the input file structure is complex.
- `examples/table_style.md`: Standard table format
- `examples/list_style.md`: Standard Markdown list format (`- [ ]`)
- `examples/mixed_style.md`: Mixed format

**Extraction Rules:**

1.  **Identify the structure**: Look for sections representing time periods.
    *   **Weeks**: "Week 1", "第一周", "Week One", etc.
    *   **Days**: "Day 1", "第一天", "Monday", "周一", etc.
    *   **If no explicit timeline**: Group tasks by their Headers (H1, H2) as "Weeks" or "Categories".

2.  **Extract tasks**: Parse each task with its associated metadata.
    *   **Focus**: Often found in table columns like "Focus", "重点", "主题".
    *   **Tasks**: The actual work items.
    *   **Nested Lists**: Flatten nested tasks into the main list but preserve hierarchy in the text (e.g., "Parent > Child"). Count every checkbox as a task.

3.  **Detect completion status**: Analyze patterns that indicate completed vs incomplete tasks.
    *   **Completed**: `✓`, `DONE`, `✅`, `[x]`, `[X]`, `完成`, `已完成`.
    *   **Incomplete**: Empty cells, `[ ]`, `TODO`, `未开始`.
    *   **Note**: If a row has mixed signals, prioritize explicit completion markers.

4.  **Organize by timeline**: Structure tasks chronologically.

### Step 3: Read the Template

Read the HTML template from `assets/template.html` using the `Read` tool.

### Step 4: Inject the Data

Replace the placeholder data in the template with the extracted TODO information.
You need to replace the `const todoData = { ... };` block in the script.

**Data Structure Requirements:**
*   `title`: Document title (e.g., "Linux Learning Roadmap").
*   `weeks`: Array of week objects.
    *   `weekNumber`: Integer (1, 2, ...).
    *   `title`: Display title (e.g., "Week 1: Basics" or "第一周：基础知识").
    *   `days`: Array of day objects.
        *   `day`: Display string (e.g., "Day 1" or "Section 1").
        *   `focus`: (Optional) Focus area string.
        *   `tasks`: Array of task objects.
            *   `id`: Unique string ID (e.g., "w1-d1-1").
            *   `text`: Task description.
            *   `completed`: Boolean `true` or `false`.
*   `totalTasks`: Integer count.
*   `completedTasks`: Integer count.

### Step 5: Write the Output

Use the `Write` tool to save the generated HTML to a file (e.g., `todo-dashboard.html`) in the **same directory** as the source TODO file.
*   If the user provided a specific output filename, use it.
*   Otherwise, append `.html` to the original filename or use `dashboard.html`.

## Example

### Input (List Format)

```markdown
# Project Alpha

## Phase 1
- [x] Setup Repo
- [ ] Design DB
    - [ ] Users Table
    - [ ] Posts Table
```

### Extracted Data Structure

```json
{
  "title": "Project Alpha",
  "weeks": [
    {
      "weekNumber": 1,
      "title": "Phase 1",
      "days": [
        {
          "day": "Tasks",
          "focus": "General",
          "tasks": [
            {
              "id": "1-1",
              "text": "Setup Repo",
              "completed": true
            },
            {
              "id": "1-2",
              "text": "Design DB",
              "completed": false
            },
            {
              "id": "1-3",
              "text": "Design DB > Users Table",
              "completed": false
            },
            {
              "id": "1-4",
              "text": "Design DB > Posts Table",
              "completed": false
            }
          ]
        }
      ]
    }
  ],
  "totalTasks": 4,
  "completedTasks": 1
}
```

## Constraints

- **Single HTML output**: The output must be a single, self-contained HTML file.
- **Client-side only**: No backend dependencies.
- **Local Storage**: The template handles local storage, you just provide the initial state.
- **Encoding**: Ensure UTF-8 encoding for Chinese character support.
- **Robustness**: If completion status is ambiguous, default to `false`.
