---
name: scholarag
description: Build PRISMA 2020-compliant systematic literature review systems with RAG-powered analysis in VS Code. Use when researcher needs automated paper retrieval (Semantic Scholar, OpenAlex, arXiv), AI-assisted PRISMA screening (50% or 90% threshold), vector database creation (ChromaDB), or research conversation interface. Supports knowledge_repository (comprehensive, 15K+ papers, teaching/exploration) and systematic_review (publication-quality, 50-300 papers, meta-analysis) modes. Conversation-first workflow with 7 stages.
---

# ScholaRAG: Systematic Review Automation Skill

**For**: Claude Code (AI assistant in VS Code)
**Purpose**: Guide researchers through PRISMA 2020 systematic literature review + RAG-powered analysis

---

## Quick Start (5 minutes)

### For Researchers
1. **Initialize project**: `python scholarag_cli.py init`
2. **Paste Stage 1 prompt**: Copy from [https://www.scholarag.com/guide/01-introduction](https://www.scholarag.com/guide/01-introduction)
3. **Answer Claude's questions** → Config created automatically
4. **Proceed through 7 stages** conversationally

### For AI Assistants (Claude Code)
When researcher provides a ScholaRAG prompt:
1. **Check for HTML metadata block** (`<!-- METADATA ... -->` at top of prompt)
2. **Identify current stage** (1-7) from metadata `stage` field
3. **Follow conversation pattern** (from metadata `conversation_flow`)
4. **Validate completion** (using metadata `validation_rules`)
5. **Auto-execute commands** (when `auto_execute: true`)
6. **Update `.claude/context.json`** (track progress)
7. **Show next stage prompt** (from metadata `next_stage.prompt_file`)

**Researcher should NEVER touch terminal**. You execute all scripts automatically.

---

## 7-Stage Workflow Overview

| Stage | Name | Read This File | Duration | Auto-Execute |
|-------|------|----------------|----------|--------------|
| 1 | Research Setup | [skills/claude_only/stage1_research_setup.md](skills/claude_only/stage1_research_setup.md) | 15-20 min | ✅ `scholarag init` |
| 2 | Query Strategy | [skills/claude_only/stage2_query_strategy.md](skills/claude_only/stage2_query_strategy.md) | 15-25 min | ❌ Manual review |
| 3 | PRISMA Config | [skills/claude_only/stage3_prisma_config.md](skills/claude_only/stage3_prisma_config.md) | 20-30 min | ❌ Manual review |
| 4 | RAG Design | [skills/claude_only/stage4_rag_design.md](skills/claude_only/stage4_rag_design.md) | 10-15 min | ❌ Manual review |
| 5 | Execution | [skills/claude_only/stage5_execution.md](skills/claude_only/stage5_execution.md) | 2-4 hours | ✅ Run all 5 scripts |
| 6 | Research Conversation | [skills/claude_only/stage6_research_conversation.md](skills/claude_only/stage6_research_conversation.md) | Ongoing | ❌ Interactive |
| 7 | Documentation | [skills/claude_only/stage7_documentation.md](skills/claude_only/stage7_documentation.md) | 30-60 min | ✅ Generate PRISMA |

**Progressive Disclosure**: Load stage file **only when researcher enters that stage**. Don't preload all 7 stages (token waste).

---

## Critical Branching Points

### 🔴 project_type (Stage 1 Decision)

**Two modes available**:

| Mode | Threshold | Output | Best For |
|------|-----------|--------|----------|
| `knowledge_repository` | 50% (lenient) | 15K-20K papers | Teaching, AI assistant, exploration |
| `systematic_review` | 90% (strict) | 50-300 papers | Meta-analysis, publication |

**Quick decision**:
- Publishing systematic review? → `systematic_review` ✅
- Comprehensive domain coverage? → `knowledge_repository` ✅

**Detailed decision tree**: [skills/reference/project_type_decision_tree.md](skills/reference/project_type_decision_tree.md)

**When to read decision tree**:
- Researcher asks: "Which project_type should I choose?"
- Researcher says: "I'm unsure about my research goals"
- Stage 1 initialization (proactively offer decision helper)

---

### 🔴 Stage 6 Scenarios (7 Research Modes)

**Stage 6 branches into 7 specialized conversation scenarios**:

1. **overview** (Context Scanning): High-level themes, methods, findings
2. **hypothesis** (Hypothesis Validation): Evidence for/against with effect sizes
3. **statistics** (Statistical Extraction): RCT data table (tools, Cohen's d, samples)
4. **methods** (Methodology Comparison): RCT vs quasi vs mixed methods
5. **contradictions** (Contradiction Detection): Conflicting results + analysis
6. **policy** (Policy Translation): Actionable recommendations for stakeholders
7. **grant** (Future Research Design): Follow-up study design + hypotheses

**Details**: [skills/claude_only/stage6_research_conversation.md](skills/claude_only/stage6_research_conversation.md)

**When to read**: Stage 6 entry (researcher asks "What can I query?")

---

## Error Recovery

**When errors occur**: [skills/reference/error_recovery.md](skills/reference/error_recovery.md)

**Quick fixes** (common issues):

| Error | Quick Fix | Detailed Guide |
|-------|-----------|----------------|
| Too many papers (>30K) | Refine query in Stage 2, re-run fetch | error_recovery.md §2.1 |
| API key missing | Add `ANTHROPIC_API_KEY` to `.env` | error_recovery.md §3.1 |
| Low PDF success (<30%) | Filter for `open_access` in Stage 1 | error_recovery.md §4.1 |
| All papers excluded (0 papers) | Lower threshold or broaden query | error_recovery.md §3.2 |

---

## Reference Materials (Load Only When Needed)

**Progressive disclosure**: Don't preload these. Read **only when** researcher asks specific questions.

| Topic | File | When to Read |
|-------|------|--------------|
| API endpoints | [skills/reference/api_reference.md](skills/reference/api_reference.md) | Researcher asks about Semantic Scholar, OpenAlex, arXiv |
| Config schema | [skills/reference/config_schema.md](skills/reference/config_schema.md) | Researcher asks "What fields are in config.yaml?" |
| PRISMA checklist | [skills/reference/prisma_guidelines.md](skills/reference/prisma_guidelines.md) | Researcher asks about PRISMA 2020 compliance |
| Troubleshooting | [skills/reference/troubleshooting.md](skills/reference/troubleshooting.md) | Researcher reports errors not in Quick Fixes |

---

## Architecture Overview

**File dependencies**: [https://www.scholarag.com/codebook/architecture](https://www.scholarag.com/codebook/architecture)

**Key principle**: Scripts read from `config.yaml` (single source of truth), **never hardcode values**.

**Critical scripts** (read `project_type` from config):
- `03_screen_papers.py`: Sets threshold (50% or 90%)
- `07_generate_prisma.py`: Changes diagram title ("Knowledge Repository" vs "Systematic Review")

---

## For Codex Users

**If researcher is using OpenAI Codex instead of Claude Code**:

See [AGENTS.md](AGENTS.md) for bash-based task workflows.

Codex workflow differs:
- **Task-oriented** (not conversation-oriented)
- **Bash commands** (not validation rules)
- **Exit codes** (not metadata parsing)

**Universal reference files** (Claude + Codex both use):
- `skills/reference/project_type_decision_tree.md`
- `skills/reference/api_reference.md`
- `skills/reference/config_schema.md`

---

## Token Optimization Notes

**This file**: ~400 lines (loaded once per conversation)

**Stage-specific files**: ~300-500 lines each (loaded on-demand)

**Total per conversation**: ~700 lines (this file + current stage file)

**Previous approach**: ~2,000 lines (all context upfront)

**Token reduction**: **65%** ✅

**How it works**:
1. Researcher starts Stage 1 → You load this file + `stage1_research_setup.md`
2. Researcher moves to Stage 2 → You load `stage2_query_strategy.md` (Stage 1 file unloaded)
3. Reference files loaded **only when** researcher asks (e.g., "How does Semantic Scholar API work?")

---

## Metadata Block Format

**All prompts in `prompts/*.md` contain HTML comment metadata at top**:

```html
<!-- METADATA
stage: 1
stage_name: "Research Domain Setup"
expected_duration: "15-20 minutes"
conversation_mode: "interactive"
expected_turns: "6-10"
outputs:
  required:
    - project_name: "Descriptive name"
    - research_question: "Clear, answerable question"
    - project_type: "knowledge_repository OR systematic_review"
validation_rules:
  project_type:
    required: true
    allowed_values: ["knowledge_repository", "systematic_review"]
cli_commands:
  - command: "python scholarag_cli.py init ..."
    auto_execute: true
next_stage:
  stage: 2
  prompt_file: "prompts/02_query_strategy.md"
-->
```

**How to use**:
1. **Parse YAML inside HTML comment** (lines between `<!-- METADATA` and `-->`)
2. **Extract fields**: `stage`, `expected_turns`, `validation_rules`, `cli_commands`, `next_stage`
3. **Follow conversation pattern**: Ask questions matching `expected_turns` count
4. **Validate**: Check user inputs against `validation_rules`
5. **Execute**: Run `cli_commands` when conversation complete
6. **Transition**: Show prompt from `next_stage.prompt_file`

---

## Divergence Handling

**Common researcher confusions** (from metadata `divergence_handling`):

### Divergence 1: "Can you help me download PDFs?" (in Stage 1)
**Response**: "PDF downloading happens in Stage 4 (after screening in Stage 3). Right now in Stage 1, let's first define your research scope and choose project_type. We'll design queries in Stage 2, configure PRISMA in Stage 3, then download PDFs in Stage 4."

### Divergence 2: "I want to skip systematic review" (in Stage 1)
**Response**: "If you don't need publication-quality systematic review, choose `project_type: knowledge_repository` in the next question. This mode uses lenient filtering (50% threshold) for comprehensive domain coverage (15K-20K papers). It's perfect for teaching materials, AI assistants, or exploratory research."

### Divergence 3: "What's the difference between the two modes?" (in Stage 1)
**Response**: "Let me explain:

**knowledge_repository**:
- 50% threshold (lenient, removes only spam)
- 15,000-20,000 papers output
- For: Teaching, exploration, AI assistant

**systematic_review**:
- 90% threshold (strict, PRISMA 2020)
- 50-300 papers output
- For: Meta-analysis, publication

See full decision tree: [skills/reference/project_type_decision_tree.md](skills/reference/project_type_decision_tree.md)"

---

## Conversation Flow Example (Stage 1)

**Typical pattern** (6-10 turns):

1. **Turn 1**: Researcher provides research topic
   - **You ask**: "Is this for exploratory domain mapping or publication-quality systematic review?"

2. **Turn 2-3**: Researcher answers scope questions
   - **You suggest**: `project_type` based on answers, explain threshold implications
   - **Example**: "Based on your goal of meta-analysis, I recommend `systematic_review` mode with 90% screening threshold."

3. **Turn 4-5**: Researcher confirms project_type choice
   - **You suggest**: Year range, publication types, expected databases
   - **Example**: "For language learning studies, I recommend 2015-2025 (10 years) focusing on Semantic Scholar and ERIC."

4. **Turn 6-8**: Researcher provides final details (domain, year range)
   - **You summarize**: All decisions, ask for confirmation
   - **Example**: "Here's what I'll create: [summary]. Ready to initialize?"

5. **Turn 9-10**: Researcher confirms initialization
   - **You execute**: `scholarag_cli.py init`, create `config.yaml`, show next steps
   - **Example**: "✅ Project initialized! Next, let's design your search query in Stage 2."

---

## Completion Checklist (Stage-Specific)

**Stage 1 example** (from metadata `completion_checklist`):

- [ ] `project_name` is descriptive and unique (≥10 chars)
- [ ] `research_question` is specific and answerable (≥20 chars)
- [ ] `project_type` chosen with understanding of implications (50% vs 90%)
- [ ] `year_range` is realistic for scope (≤25 years, not before 2000)
- [ ] `config.yaml` created successfully (file exists, valid YAML)

**When all checked** → Auto-execute `scholarag_cli.py init` → Show Stage 2 prompt

---

## Example Commands You Will Execute

### Stage 1: Initialize
```bash
python scholarag_cli.py init \
  --name "AI-Chatbots-Language-Learning" \
  --question "How do AI chatbots improve speaking proficiency in EFL learners?" \
  --domain education
```

### Stage 5: Run Pipeline (All 5 Scripts)
```bash
# Fetch papers
python scripts/01_fetch_papers.py --project projects/YYYY-MM-DD_ProjectName

# Deduplicate
python scripts/02_deduplicate.py --project projects/YYYY-MM-DD_ProjectName

# Screen with AI
python scripts/03_screen_papers.py --project projects/YYYY-MM-DD_ProjectName

# Download PDFs
python scripts/04_download_pdfs.py --project projects/YYYY-MM-DD_ProjectName

# Build RAG
python scripts/05_build_rag.py --project projects/YYYY-MM-DD_ProjectName
```

### Stage 7: Generate PRISMA
```bash
python scripts/07_generate_prisma.py --project projects/YYYY-MM-DD_ProjectName
```

---

## Integration with .claude/context.json

**You should update this file after each stage**:

```json
{
  "current_stage": {
    "stage": 2,
    "name": "Query Strategy",
    "status": "in_progress",
    "started_at": "2025-10-24T10:30:00Z"
  },
  "completed_stages": [
    {
      "stage": 1,
      "name": "Research Setup",
      "completed_at": "2025-10-24T10:25:00Z",
      "outputs": {
        "project_name": "AI-Chatbots-Language-Learning",
        "research_question": "How do AI chatbots improve speaking proficiency?",
        "project_type": "systematic_review"
      }
    }
  ],
  "project": {
    "name": "AI-Chatbots-Language-Learning",
    "created": "2025-10-24",
    "research_question": "How do AI chatbots improve speaking proficiency in EFL learners?",
    "project_type": "systematic_review"
  }
}
```

**Purpose**: Track progress, enable `scholarag status` command to show current stage.

---

## FAQ for AI Assistants

### Q: Should I always read stage files in order (1→2→3...)?
**A**: No! Read **only the file for the current stage** researcher is in. Use progressive disclosure.

### Q: What if researcher jumps to Stage 5 without completing Stages 1-4?
**A**: Check `.claude/context.json` for completed stages. If missing prerequisites, politely redirect:
"Stage 5 requires config.yaml from Stage 1, search_query from Stage 2, and PRISMA criteria from Stage 3. Let's complete those first."

### Q: When should I read `skills/reference/` files?
**A**: **Only when researcher explicitly asks**. Examples:
- "How does Semantic Scholar API work?" → Read `api_reference.md`
- "What are all the config.yaml fields?" → Read `config_schema.md`
- "Why should I choose systematic_review?" → Read `project_type_decision_tree.md`

### Q: What if I don't understand metadata in prompts/*.md?
**A**: All metadata fields are documented in `skills/claude_only/metadata_spec.md`. Read that file if you encounter unknown fields.

---

## Additional Resources

**Detailed implementation guide**: See [CLAUDE.md](CLAUDE.md) for:
- 🎓 User profile (researchers with limited coding experience)
- How Claude Code should behave (DO/DON'T guidelines)
- Auto-execution patterns (echo pipes, CLI arguments)
- Full CLI reference and troubleshooting

**For Codex/Cursor users**: See [AGENTS.md](AGENTS.md) for task-based bash workflows

---

**Last Updated**: 2025-10-24 (v2.0 - Agent Skills Integration)
**Companion files**: CLAUDE.md (detailed guide), AGENTS.md (Codex workflows)
**Compatible with**: Claude Code v1.0+, Anthropic API
**Token Budget**: ~380 lines (this file) + ~300-500 lines (stage file) = ~700-900 lines per conversation
