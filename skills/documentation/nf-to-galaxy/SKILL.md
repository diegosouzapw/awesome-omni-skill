---
name: nf-to-galaxy
description: Router skill for Nextflow to Galaxy conversions - directs to appropriate sub-skill
---

# Nextflow to Galaxy Conversion (Router)

## Purpose

This is a **router skill** that directs you to the appropriate specialized skill based on what you're converting.

For complete pipelines, you are expected to include **bioinformatics best-practice sanity checks** and discuss **QC + reporting** requirements (e.g. MultiQC). See `nf-pipeline-to-galaxy-workflow/SKILL.md`.

---

## What Are You Converting?

### Option 1: Single Nextflow Process → Galaxy Tool

**Converting**: One Nextflow process to one Galaxy tool XML

**Example**: `HYPHY_FEL` process → `hyphy_fel.xml` tool

**Use skill**: `nf-process-to-galaxy-tool`

**When to use**:
- Creating a Galaxy tool wrapper for a specific bioinformatics tool
- You've identified a missing tool during pipeline conversion
- Converting a single nf-core module

---

### Option 2: Nextflow Subworkflow → Galaxy Workflow

**Converting**: Multiple connected Nextflow processes to a Galaxy workflow

**Example**: `PROCESS_VIRAL_NONRECOMBINANT` subworkflow → Galaxy `.ga` workflow

**Use skill**: `nf-subworkflow-to-galaxy-workflow`

**When to use**:
- Converting a logical group of processes
- Building a reusable workflow component
- Most processes already have Galaxy tools

---

### Option 3: Complete Nextflow Pipeline → Galaxy Solution

**Converting**: Full Nextflow pipeline to complete Galaxy solution

**Example**: `CAPHEINE` pipeline → Multiple Galaxy workflows + any missing tools

**Use skill**: `nf-pipeline-to-galaxy-workflow`

**When to use**:
- Converting an nf-core pipeline
- Large-scale conversion project
- Need to plan tool creation + workflow assembly

---

## Key Concepts

**Before using any sub-skill**, understand these mappings:

| Aspect | Nextflow | Galaxy |
|--------|----------|--------|
| **Unit of work** | Process (Groovy DSL) | Tool (XML) |
| **Organization** | Module (directory) | Tool directory |
| **Workflow format** | `.nf` files (code) | `.ga` files (JSON) |
| **Dependencies** | Containers or Conda | Conda via `<requirements>` |
| **Inputs/Outputs** | Channels | Datasets with datatypes |

**Critical**: One Nextflow **process** = One Galaxy **tool**

**Read**: `nextflow-galaxy-terminology.md` for detailed conceptual mapping

---

## Getting Started

**New to this skill?** See `README.md` for file organization and navigation.

**Using Galaxy integration?** See `../../galaxy-integration/README.md` and `../../galaxy-integration/galaxy-integration.md`.

---

## Shared Resources

### Core Guides
- **`../../galaxy-integration/galaxy-integration.md`** - Galaxy MCP/BioBlend: setup, tool checking, workflow testing
- **`check-tool-availability.md`** - Manual tool checking across repositories
- **`testing-and-validation.md`** - Routing page to canonical testing docs
- **`../../tool-dev/references/testing.md`** - Tool testing with Planemo

### Tool Discovery Order (Installed vs Available vs Missing)

Before you conclude a tool is "missing", distinguish:

1. **Installed on your target Galaxy instance** (ready to use in a `.ga`).
2. **Not installed, but has an existing wrapper** that can be installed (ToolShed / tools-iuc / known maintainers).
3. **No wrapper exists** (tool wrapper must be created).

If you determine (3) applies, use **`nf-process-to-galaxy-tool/`** to create a Galaxy tool wrapper.

When a tool is not installed on the target instance, follow the repository search order in **`check-tool-availability.md`**.

**Common ToolShed owners to check** (not exhaustive):
- `iuc`
- `devteam`
- `bgruening`
- `genouest`

### Reference Documentation
- **`nextflow-galaxy-terminology.md`** - Concept mappings (process→tool, workflow→.ga)
- **`process-to-tool.md`** - Process → Tool conversion details
- **`workflow-to-ga.md`** - Workflow → .ga conversion details
- **`container-mapping.md`** - Container → Conda package mapping
- **`datatype-mapping.md`** - File patterns → Galaxy datatypes
- **`tool-sources.md`** - Where to create tools (tools-iuc vs custom)

### Scripts & Examples
- **`scripts/`** - Automation scripts (see `scripts/README.md`)
- **`examples/`** - Complete conversion examples (see `examples/README.md`)

---

## Quick Decision Tree

```
What are you converting?
│
├─ Single process?
│  └─ Use: nf-process-to-galaxy-tool
│
├─ Subworkflow (multiple processes)?
│  └─ Use: nf-subworkflow-to-galaxy-workflow
│
└─ Complete pipeline?
   └─ Use: nf-pipeline-to-galaxy-workflow
```

---

## Sub-Skills

### `nf-process-to-galaxy-tool/`
Convert a single Nextflow process to a Galaxy tool XML.

### `nf-subworkflow-to-galaxy-workflow/`
Convert a Nextflow subworkflow to a Galaxy workflow.

### `nf-pipeline-to-galaxy-workflow/`
Convert a complete Nextflow pipeline to a Galaxy solution.
