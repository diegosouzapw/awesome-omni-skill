---
name: handoff
description: Context handoff for session transitions. Use when context window is getting large, switching work phases, or user requests /handoff. Creates structured checkpoint documents for seamless continuation in new sessions.
metadata:
  short-description: Chuyển giao ngữ cảnh giữa các sessions
  version: 1.0.0
  author: wollfoo
  triggers:
    - /handoff
    - context overflow
    - phase transition
---

# Handoff Skill

**Mục đích**: Chuyển giao ngữ cảnh từ session hiện tại sang session mới một cách có cấu trúc.

---

## Trigger Analysis

Xác định thời điểm cần thực hiện handoff:

| Trigger | Điều kiện | Hành động |
|---------|-----------|-----------|
| **Context Overflow** | >80% context window | Bắt buộc handoff |
| **Phase Transition** | Chuyển từ planning → implementation | Khuyến nghị handoff |
| **Task Completion** | Xong 1 task, bắt đầu task mới | Khuyến nghị handoff |
| **Long Session** | >30 turns trong 1 conversation | Cảnh báo + gợi ý |
| **Manual** | User yêu cầu `/handoff` | Thực hiện ngay |

---

## Context Extraction

### Ưu tiên CAO (PHẢI giữ)
- **Decisions** – Các quyết định kiến trúc/thiết kế đã đưa ra
- **Current State** – Trạng thái hiện tại của task (% hoàn thành)
- **Modified Files** – Danh sách files đã thay đổi trong session
- **Pending Tasks** – Tasks còn chưa hoàn thành
- **Blockers** – Vấn đề/lỗi đang gặp phải

### Ưu tiên TRUNG (NÊN giữ)
- **Relevant Code** – Code snippets quan trọng (functions, configs)
- **Dependencies** – Thư viện/services đang sử dụng
- **Test Status** – Kết quả tests gần nhất

---

## Execution Steps

### Step 1: Collect Context
```
1. Scan recent conversation for decisions and context
2. Identify modified files using git diff or memory
3. Extract pending tasks from task.md or conversation
4. Note any blockers or issues encountered
```

### Step 2: Create Checkpoint File
Create file in `.handoff/checkpoints/` using template:
- Filename format: `YYYY-MM-DD_HH-MM_[task-name].md`
- Use template from `assets/handoff-template.md`

### Step 3: Generate Resume Prompt
Create a ready-to-use prompt that the user can paste into a new session:
- Include 2-3 sentence context summary
- List key files to reference with @mentions
- Specify the immediate next action
- Reference the checkpoint file location

### Step 4: Verification
- Confirm checkpoint file is saved
- Confirm resume prompt is generated
- List all files mentioned for easy access
- Provide clear next steps

---

## Output Format

When handoff is triggered, respond with:

```markdown
## 🔄 Handoff Created

### Checkpoint
📁 `.handoff/checkpoints/[filename].md`

### Resume Prompt
> [Ready-to-paste prompt for new session]

### Quick Resume
Trong session mới, paste prompt trên hoặc:
```
Tiếp tục task từ .handoff/checkpoints/[filename].md
```

### Next Steps
1. [Immediate action]
2. [Follow-up action]
```

---

## Commands

| Command | Description |
|---------|-------------|
| `/handoff` | Full handoff với goal tự detect |
| `/handoff [goal]` | Handoff với goal cụ thể |
| `/checkpoint` | Chỉ save checkpoint, không switch |
| `/resume [file]` | Load checkpoint và tiếp tục |

---

## Best Practices

### ✅ DO
- Handoff khi chuyển phase (planning → implementation)
- Giữ sessions ngắn: 15-20 turns
- Ghi rõ decisions và lý do
- Include file paths với relative paths

### ❌ DON'T
- Đợi đến overflow mới handoff
- Bỏ qua pending tasks trong handoff
- Handoff không có next steps rõ ràng
- Quên list modified files
