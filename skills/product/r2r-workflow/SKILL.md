---
name: r2r-workflow
description: Explain the Research-to-Roadmap (r2r) workflow visually. Shows the 6-phase pipeline JD built for transforming R&D projects into production roadmaps.
---

# R2R Workflow Explanation

When triggered, display this workflow explanation:

---

## Research to Roadmap (r2r)

A Claude Code plugin I built overnight to transform R&D projects, POCs, and lab experiments into production-ready roadmaps.

### The Pipeline

```
┌─────────┐   ┌───────────┐   ┌────────────┐   ┌─────────┐   ┌──────────┐   ┌────────┐
│ ASSESS  │ → │ DECOMPOSE │ → │ PRIORITIZE │ → │ ROADMAP │ → │ VALIDATE │ → │ EXPORT │
└─────────┘   └───────────┘   └────────────┘   └─────────┘   └──────────┘   └────────┘
```

### What Each Phase Does

**1. ASSESS** `/r2r:assess`
- Analyzes research artifacts (POCs, spikes, ADRs)
- Determines what was actually proven vs assumed
- Gives a production readiness score (1-10)
- Identifies gaps and technical debt

**2. DECOMPOSE** `/r2r:decompose`
- Breaks research into shippable components
- Single-responsibility units
- T-shirt sizing (S/M/L/XL)
- Maps dependencies between components

**3. PRIORITIZE** `/r2r:prioritize`
- Effort vs impact scoring
- Assigns to horizons:
  - H1 (0-3 months): Ship now
  - H2 (3-6 months): Build next
  - H3 (6-12 months): Explore later
- Identifies quick wins

**4. ROADMAP** `/r2r:roadmap`
- Generates milestone-based roadmap
- Creates Mermaid Gantt timeline
- Maps release phases (Internal → Private → Public → GA)
- Multiple stakeholder views (exec/eng/PM)

**5. VALIDATE** `/r2r:validate`
- Pre-mortem analysis ("The project failed because...")
- Assumption audit
- Dependency validation
- Kill criteria check
- Pass/Caution/Fail status

**6. EXPORT** `/r2r:export`
- GitHub Issues + Milestones
- Linear issues
- Markdown summary
- JSON for automation

### Why I Built This

Most R&D work dies in the gap between "cool demo" and "production system." This plugin creates a structured path across that gap, based on patterns from GitHub Next, Linear, Vercel, and Figma's labs.

### Try It

```bash
/r2r:full ./path/to/research
```

Or run `/r2r-demo` to see sample output.

---

Keep the response concise. Don't add extra commentary beyond the workflow explanation.
