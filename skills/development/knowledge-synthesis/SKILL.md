---
name: knowledge-synthesis
description: 知识合成 — 将多来源信息融合为结构化知识，生成摘要、报告和知识图谱
version: "1.0.0"
license: MIT
metadata:
  author: blackplume233
  actant-tags: "synthesis,knowledge,summarization,researcher"
  layer: auxiliary
  agent: researcher
---

# Knowledge Synthesis

## Overview

Fuse multi-source information into structured knowledge; generate summaries, reports, and knowledge graphs.

## Information Fusion

- Merge overlapping content; resolve redundancy
- Align terminology across sources; normalize units and formats
- Preserve provenance (source, timestamp) for each fact

## Contradiction Resolution

- Detect conflicting claims across sources
- Apply source-evaluation scores to prefer higher-credibility sources
- Surface unresolved contradictions to user with context

## Structured Output Generation

- **Reports**: Executive summary, key findings, recommendations, appendix with sources
- **Summaries**: Bullet-point or paragraph form; configurable length
- **Comparisons**: Side-by-side analysis (e.g., tool A vs B); pros/cons tables

## Knowledge Graph Entry Creation

- Extract entities (people, orgs, concepts) and relations
- Create graph nodes and edges; support incremental updates
- Export to standard formats (JSON-LD, RDF) for downstream use

## Confidence Scoring

- Assign confidence (0–1) to synthesized claims based on source agreement and quality
- Include confidence in output; flag low-confidence items for review
