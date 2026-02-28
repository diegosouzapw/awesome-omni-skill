---
name: essay-writing
description: "Writing support (drafting, evaluation and revision) for executives and tech leaders. Support everything from internal culture-building to external technical branding."
disable-model-invocation: false
user-invocation: true
model: opus
globs:
  - "drafts/**/*.md"
  - "essays/**/*.md"
---

# Essay Writing Skill

Provide high-impact essay writing support for technology leaders (CTOs, VPoEs, Engineering Managers, etc.).

## Target Platforms & Tones

Adapt the writing style based on the intended audience and platforms.

| Publication Type          | Primary Audience                     | Recommended Tone                             |
|:--------------------------|:-------------------------------------|:---------------------------------------------|
| **Internal (Notes/Wiki)** | Employees, Engineers, Managers       | Casual, personal, transparent, authentic     |
| **External (Tech Blog)**  | Industry peers, potential candidates | Polished, passionate, logical, authoritative |

---

## Workflow

Identify the current phase of the request and reference the appropriate guidelines in the `references/` directory.

### Phase 1: Outlining & Structure
* **Reference:** Analyze `references/writing-style.md` to apply the correct structure pattern (Pattern A for Internal, Pattern B for External).
* **Key Principle:** Propose **"Assertive Headings"**—headings should be statements or takeaways that convey the author's message, not generic labels.

### Phase 2: Drafting
* **Reference:** Apply the tone and constraints found in `references/writing-style.md` and `references/rules.md`.
* **Key Principle:** Avoid generic "AI-speak." Write in an **authentic voice** that reflects the author's unique perspective and experience.

### Phase 3: Review & Revision
Select the specific mode based on the user's prompt:

| Keyword                        | Mode              | Description                                                                                              |
| :----------------------------- | :---------------- | :------------------------------------------------------------------------------------------------------- |
| **"Evaluate" / 「評価して」**    | **Scoring**       | Rate the draft on 5 dimensions (0.0–5.0). Output a diagnostic report **without** auto-fixing the text.   |
| **"Revise" / 「推敲して」 **     | **Self-Revision** | A full cycle: Evaluate → Line-level fixes → Structural reorganization → Polishing → Final Re-evaluation. |
| **"Review" / 「レビューして」 ** | **Peer Feedback** | Act as a peer reviewer. Provide feedback using the **Good / Question / Suggestion** format.              |

## Usage Examples

### 1. Outlining
* **EN:** "Give me an outline for an internal note regarding 'Our Philosophy on Engineering Career Paths'."
* **JP:** 「エンジニアのキャリア形成の考え方」について、社内向けの構成案を出して
* **Action:** Apply `references/writing-style.md` (Pattern A)

### 2. Drafting
* **EN:** "Write a draft based on this outline. Keep the tone passionate and lead-by-example."
* **JP:** この構成案をもとに下書きを書いて。情熱的で、背中で語るようなトーンで。
* **Action:** Reference `references/writing-style.md` + `references/rules.md`

### 3. Evaluation or Revision
* **EN:** "Evaluate this draft's impact and then revise it for a public tech blog."
* **JP:** この下書きを評価して。その後、テックブログ掲載用に推敲して。
* **Action:** Execute `references/review.md` Part 1 (Score) → Part 2 & 3 (Revision)

### 4. Peer Review / レビュー
* **EN:** "Review this draft from a hiring candidate's perspective."
* **JP:** 採用候補者の視点で、この下書きをレビューして。
* **Action:** Execute `references/review.md` Part 4

