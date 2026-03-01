---
name: output-handoff
description: Handle output distribution from processed brain dumps to appropriate destinations. Use when determining where outputs should go.
---

# Output Handoff Strategy Skill

This skill guides the handoff of processed brain dump outputs to their final destinations.

## The Handoff Decision Tree

```
Processed Brain Dump
        ↓
What is the OUTPUT TYPE?
        ↓
┌─────────────┬─────────────────┬─────────────────┬─────────────────┐
│  Content    │  Technical      │  Architecture   │  Reference      │
│  (Written)  │  (Code/Repos)   │  (Agents/MCP)   │  (Notes/Ideas)  │
└─────────────┴─────────────────┴─────────────────┴─────────────────┘
        ↓              ↓                    ↓               ↓
    outputs/      projects/ +          projects/ +      ideas/ or
    [type]/       GitHub repo          Plan first       processed/
                                          ↓
                                     completed/
                                    (when done)
```

## Output Type 1: Content (any written output)

### When to Use
- Blog posts
- LinkedIn articles
- Emails and messages
- Video scripts
- Tech session outlines
- Presentations
- Social media content
- Documentation

### Handoff Destination
`outputs/content/YYYY-MM-DD-[slug].md`

### Template
```markdown
---
date: "YYYY-MM-DD"
source: "inbox/dump-..."
type: [blog|linkedin|email|presentation|video-script|other]
status: draft
---

# [Title]

**Platform:** [Platform]
**Status:** Draft (ready for review)

---

[Polished content]

---

## Publishing Checklist
- [ ] Final review
- [ ] Add tags/hashtags
- [ ] Copy to [platform]
- [ ] Publish
- [ ] Update status: published
```

### Post-Handoff
Content is ready for manual copy-paste to publishing platform.

## Completed Ideas

When an idea has been fully implemented:
1. Move file from `ideas/` to `completed/`
2. Update frontmatter:
   - `status: completed`
   - `completed_date: "YYYY-MM-DD"`
   - `outcome: "Brief description of what was accomplished"`
3. Commit the change

This keeps `ideas/` focused on active/someday ideas while maintaining a record of accomplishments.

## Output Type 2: Technical Projects

### When to Use
- Coding projects
- Scripts and tools
- APIs and integrations
- Anything requiring version control

## Output Type 3: Architecture (Agents, Orchestrators, MCP)

### When to Use
- Copilot Studio solutions
- Agent systems
- Orchestrators
- MCP server designs
- AI assistant workflows

### Handoff Destination
1. **Pointer file:** `projects/[project-name].md`
2. **Plan first:** Handoff to `@solution-planner` for architecture planning
3. **Then:** Handoff to `@repo-creator` for GitHub repo (after planning)

> **Tip:** If unsure where something goes, invoke `@brain-dump-orchestrator` for routing guidance.

### Pointer File Template
```markdown
---
date: "YYYY-MM-DD"
source: "inbox/dump-..."
status: planning
output_type: ARCHITECTURE
---

# [Project Name]

**Status:** Planning
**Architecture Plan:** [TO BE CREATED - USE @solution-planner]
**Repo:** [TO BE CREATED AFTER PLANNING]
**Source:** [Link to dump]

## Project Brief
[Brief from processing]

## Next Actions
- [ ] Run @solution-planner to create architecture plan
- [ ] Review plan
- [ ] Handoff to @repo-creator to create GitHub repository
```

### Workflow
1. Create pointer file in `projects/`
2. Offer "Plan First" handoff to `@solution-planner`
3. After planning, user reviews and approves
4. Handoff to `@repo-creator` to create repository
