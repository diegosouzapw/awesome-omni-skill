---
name: update-notes
description: Condense learnings from this session, so that they can be used to make better decisions on future runs. With these notes, you don't need to rely on auto-compaction to keep a conversation going.
---

Review this session and update the markdown files in `.claude/docs/`. Before adding an item, make sure you are not duplicating existing contents. Only add new items.

1. **architecture.md** — Append patterns/conventions discovered
2. **decisions.md** — Append: `- [YYYY-MM-DD] Chose X over Y because Z` 
3. **failures.md** — Append: `- Don't try X — causes Y because Z`
4. **plan.md** — Replace entirely with current status + TODOs

Create the directory if needed. Commit changes with `docs: update session notes`. Then suggest `/clear`.
