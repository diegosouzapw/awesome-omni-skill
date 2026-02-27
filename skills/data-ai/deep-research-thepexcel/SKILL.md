---
name: deep-research
description: Web research with Graph-of-Thoughts for fast-changing topics. Use when user requests research, analysis, investigation, or comparison requiring current information. Features hypothesis testing, source triangulation, claim verification, Red Team, self-critique, and gap analysis. Supports Quick/Standard/Deep/Exhaustive tiers. Creative Mode for cross-industry innovation.
---

# Deep Research

Enhanced research engine for topics where training data is outdated.

## Quick Start

### Standard Mode
```
CLASSIFY → LANDSCAPE SCAN → RECENCY PULSE → SCOPE → HYPOTHESIZE → PLAN → [PLAN PREVIEW*] → RETRIEVE
→ GAP ANALYSIS → TRIANGULATE → SYNTHESIZE → RED TEAM → SELF-CRITIQUE → PACKAGE
```
*Deep+ tier only

### LANDSCAPE SCAN (MANDATORY - Before Anything Else)
```
[Search for OVERVIEW first - NO known entity names in query!]
WebSearch: "[topic] landscape overview [current year]"
WebSearch: "top [topic] list [current year]"
WebSearch: "[topic] ecosystem players [current year]"

❌ WRONG: "DeepSeek Qwen performance 2025" (uses names you already know)
✅ RIGHT: "China open source LLM models list 2025" (discovers what exists)

→ Extract ALL entity names from results
→ List: Discovered (new to you) vs Confirmed (you knew)
→ THEN proceed to RECENCY PULSE
```
**Why:** You cannot research what you don't know exists. Scan the landscape FIRST.

### RECENCY PULSE (MANDATORY - After Landscape Scan)
```
[Search for LATEST news — within days/weeks, not just "this year"]
WebSearch: "[topic] latest news this week [current month] [current year]"
WebSearch: "[topic] new release announcement [current month] [current year]"
WebSearch: "[upstream provider 1] latest release [current year]"
WebSearch: "[upstream provider 2] latest release [current year]"

→ Check: anything released in the last 7-30 days?
→ If yes: add to entity list, flag as BREAKING/RECENT
→ THEN proceed to SCOPE with complete picture
```

**UPSTREAM CHECK (part of Recency Pulse):**
```
For any product/platform research, identify the SUPPLY CHAIN:
- Who MAKES the underlying technology? (e.g., OpenAI → GPT, Anthropic → Claude)
- Who DISTRIBUTES it? (e.g., Microsoft → Copilot, GitHub → Copilot)
- Who COMPETES with it? (e.g., Google → Gemini)

Search EACH upstream provider directly — don't rely on downstream announcements.

Example for "Microsoft Copilot":
  Upstream: OpenAI (GPT models), Anthropic (Claude models)
  Downstream: Microsoft (Copilot products)
  → Search "OpenAI latest model [month] [year]"
  → Search "Anthropic latest release [month] [year]"
  → Search "Microsoft Copilot new features [month] [year]"
```

**Why:** Downstream products lag behind upstream releases. A new model from
OpenAI/Anthropic may not appear in "Microsoft Copilot updates" for weeks.
If you only search downstream, you miss what's coming or just arrived.

**Anti-pattern:** ค้นแค่ "Microsoft Copilot new features 2026" แล้วหยุด
**Better:** ค้น upstream (OpenAI, Anthropic) + downstream (Microsoft) + "this week/month"

### Creative Mode
```
ABSTRACT → MAP (3-5 domains) → SEARCH → GENERALIZE → SYNTHESIZE
```
**Trigger:** "creative mode", "cross-industry", "what do others do"

**Example:** "ทำยังไงให้คนมา engage กับ online course มากขึ้น?"
→ ABSTRACT: "retention + engagement ในกิจกรรมที่ทำซ้ำ"
→ MAP: Gaming (streaks, XP), Fitness apps (habit loops), YouTube (thumbnails, hooks), Loyalty programs (tiers)
→ SEARCH each domain → GENERALIZE patterns → SYNTHESIZE recommendations

---

## Classification

| Type | When | Process | Example |
|------|------|---------|---------|
| **A** | Single fact | WebSearch → Answer | "Python 3.13 release date คือเมื่อไหร่?" |
| **B** | Multi-fact | Scan → Retrieve → Synthesize | "เปรียบเทียบ pricing ของ cloud GPU providers" |
| **C** | Judgment needed | Full 6 phases | "ควรใช้ Next.js หรือ Astro สำหรับ blog?" |
| **D** | Novel/conflicting | Full + Red Team | "AI จะแทนที่ data analyst ภายใน 3 ปีจริงไหม?" |

## Intensity Tiers

| Tier | Sources | When |
|------|---------|------|
| Quick | 5-10 | Simple question |
| Standard | 10-20 | Multi-faceted |
| Deep | 20-30 | Novel, high stakes |
| Exhaustive | 30+ | Critical decision |

---

## Search & Evidence

### Parallel Search (MANDATORY)
```
[Single message — always 2-3 queries at once]
WebSearch: "[topic] [current year]"
WebSearch: "[topic] limitations"
WebSearch: "[topic] vs alternatives"
```

### Claim Types

| Type | Requirements | Example |
|------|--------------|---------|
| **C1** (Key claim) | Quote + 2+ sources + confidence | "Next.js มี market share 42%" |
| **C2** (Supporting) | Citation required | "Vercel เป็นผู้พัฒนา Next.js" |
| **C3** (Common knowledge) | Cite if contested | "React เป็น library ยอดนิยม" |

### Confidence Format (C1 claims)
```
**Claim:** [Statement]
**Confidence:** HIGH/MEDIUM/LOW
**Reason:** [Why this confidence level]
**Sources:** [1][2]
```

### Anti-Hallucination
- Every C1 cites [N] immediately
- Use "According to [1]..."
- Admit: "No sources found for X"

---

## Research Sufficiency

**"เมื่อไหร่ถึงจะพอ?"**

| Signal | หมายความว่า |
|--------|-----------|
| **Saturation** | 3 sources ต่อเนื่องไม่ให้ข้อมูลใหม่ → พอแล้ว |
| **Convergence** | หลาย sources สรุปเหมือนกัน → confidence สูง |
| **Contradiction** | Sources ขัดแย้งกัน → ต้อง dig deeper หรือ flag uncertainty |
| **Diminishing returns** | เพิ่ม search แต่ได้แค่ rephrase ของเดิม → หยุดได้ |

**Quick tier:** หยุดเมื่อ saturation
**Standard:** หยุดเมื่อ convergence + gap analysis ไม่เจอ gap สำคัญ
**Deep/Exhaustive:** หยุดเมื่อ Red Team challenge ไม่พบจุดอ่อนใหม่

---

## Facilitation Guide

### Progress Reporting

```
ทุกๆ 5-8 sources → update ผู้ใช้:
"สรุปที่พบจนถึงตอนนี้: [key findings]
ยังมีคำถามค้าง: [gaps]
จะ search ต่อเรื่อง [next direction] นะคะ"
```

### When to Ask User

| สถานการณ์ | ถามว่า |
|-----------|-------|
| Topic กว้างเกินไป | "อยากเน้นมุมไหนคะ? [option A] หรือ [option B]?" |
| เจอ sub-topic น่าสนใจ | "เจอเรื่อง X ที่เกี่ยวข้อง — อยากให้ขุดลึกไหมคะ?" |
| Sources ขัดแย้ง | "แหล่ง A บอกว่า X แต่แหล่ง B บอกว่า Y — พี่ระ lean ทางไหนคะ?" |
| Deep+ tier, plan ready | "นี่คือ plan สำหรับ research — approve ก่อนไปต่อนะคะ" |

### Don't Ask — Just Do

- Type A questions → ตอบเลย
- Choosing search queries → ทำเลย ไม่ต้องถาม
- Formatting output → ใช้ template ได้เลย

---

## Tools & Fallbacks

### URL Fallback

If WebFetch returns 403:
```bash
curl -s --max-time 60 "https://r.jina.ai/https://example.com"
```

### GitHub Repository Research

เจอ repo น่าสนใจ → **ถาม user ก่อน clone:**
```
"เจอ repo ที่น่าสนใจ: [repo-name] — ต้องการให้ clone มาศึกษา code ไหมคะ?"
```

If agreed:
```bash
mkdir -p /mnt/d/githubresearch && cd /mnt/d/githubresearch && git clone [repo-url]
```

Key files: `package.json`/`pyproject.toml` → `src/` main logic → `README.md`

---

## References

| Topic | File | Grep Pattern |
|-------|------|--------------|
| Phase details | [standard-mode.md](./references/standard-mode.md) | `grep -n "^## Phase"` |
| Creative mode | [creative-mode.md](./references/creative-mode.md) | `grep -n "^## Phase C"` |
| Agent prompts | [agent-templates.md](./references/agent-templates.md) | `grep -n "^## "` |
| Progress/recovery | [progress-recovery.md](./references/progress-recovery.md) | — |
| Report template | [report_template.md](./assets/report_template.md) | — |
| **Query generation** | [query-framework.md](./references/query-framework.md) | QUEST Matrix |
| **Perspective audit** | [perspective-checklist.md](./references/perspective-checklist.md) | COMPASS Checklist |
| **Researcher thinking** | [researcher-thinking.md](./references/researcher-thinking.md) | THINK Protocol |

| Script | Purpose |
|--------|---------|
| `scripts/validate_report.py` | 9-check quality validation |

## Output File (MANDATORY)

After completing research, **ALWAYS save to markdown file**:

```
research/[topic-slug]-[YYYY-MM-DD].md
```

**Example:** `research/china-opensource-ai-2025-01-04.md`

- Create `research/` folder if it doesn't exist
- **Why:** Research takes effort. Save it for future reference.

---

## Related Skills

- `/boost-intel` — Apply critical thinking to research findings
- `/generate-creative-ideas` — Creative Mode for cross-industry innovation
- `/skill-creator-thepexcel` — Research domain expertise for skill creation
- `/extract-expertise` — Research to prepare expert interviews
