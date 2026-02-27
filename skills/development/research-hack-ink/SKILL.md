---
name: research
description: Use when the user requests research, investigation, best practices, comparisons, or evidence-backed recommendations.
---

# Research

## Overview

Turn an ambiguous question into a decision-ready recommendation backed by explicit evidence.

## When to use

- The user asks you to research, investigate, compare options, find best practices, or recommend a solution.
- The task requires reading existing materials (docs/links/code/logs) before deciding.
- The decision should be supported by multiple independent external sources plus any provided internal context.

## Inputs (ask for these up front)

- The exact question and the decision to be made.
- Existing materials: docs, links, code pointers, tickets, logs, dashboards, prior writeups.
- Constraints: timebox, budget, security/compliance constraints, target platforms, success criteria.
- Data sensitivity: what must not be shared externally (secrets, proprietary details, customer data).
- What "good" looks like: measurable outcomes and non-goals.

## Hard gates (non-negotiable)

1. **Read first**: do not recommend a solution before reading the provided materials (or explicitly stating none were provided).
2. **Confirm the pain**: restate the user's pain points/problems in your own words and confirm.
3. **Ask when unclear**: if anything blocks correctness, ask the user. Prefer 1-3 short questions per turn; prefer multiple-choice when possible.
4. **Evidence-backed decisions**:
    - Every recommendation must include an evidence map (claim -> evidence -> source).
    - Use websearch to find current best practices / broadly adopted solutions.
    - **Default: at least 3 independent external sources**.
    - **Fallback: 2 sources only if 3 is not feasible**, and you must say why (niche topic, paywalled sources, no primary docs, etc.).
    - Define **independent** as "not the same claim repeated via syndication or mutual citations". Prefer sources from different organizations, and prioritize primary documentation when available.
    - If websearch is unavailable (policy, environment, outage) or disallowed by the user, **stop** and ask whether to proceed with an internal-only (lower-confidence) answer.
5. **Alternatives**: present at least 2 viable options with tradeoffs (cost, complexity, risk, time-to-implement, operational burden).
6. **No evidence, no claim**: if a claim cannot be supported, label it as a hypothesis and say what evidence would confirm/deny it.
7. **No leaks**: do not include secrets, customer data, or proprietary identifiers in websearch queries or external citations.
8. **Architecture-first over minimal edits**: You may recommend breaking/destructive changes (rewrites, API changes, migrations) when they materially improve architecture, performance, or outcomes; do not constrain recommendations to minimal diffs. Always state blast radius and include a migration/rollback plan.

## Procedure

1. **Intake + scope**
    - Restate the question and the decision the user wants to make.
    - Define success criteria, constraints, and non-goals.
    - List assumptions; mark which ones need confirmation.

2. **Read existing materials**
    - Summarize facts, constraints, and what is already known.
    - Extract the user's pain points as a bullet list; ask "did I capture this correctly?"

3. **Clarify unknowns**
    - List open questions that materially affect the decision.
    - Ask the highest-leverage question(s) first.

4. **Set evaluation criteria**
    - Define the criteria you will use to judge solutions (examples: security, reliability, latency, DX, maturity/ecosystem, total cost, migration complexity).
    - If criteria conflict, ask the user to prioritize (e.g., reliability > cost > speed).

5. **Web research (best practices + option survey)**
    - Use websearch and keep a search log (queries + date).
    - Prefer primary sources: official docs, vendor guidance, standards, incident writeups, well-established engineering org posts.
    - Track publication/last-updated dates for key sources; call out when guidance is likely stale.
    - When possible, include at least one primary source (official docs/standards) and at least one non-vendor perspective among the default 3 sources.
    - Triangulate: do not rely on a single source for a key claim.
    - Capture the "why" behind best practices (constraints, failure modes, tradeoffs), not just a list of tools.
    - Use lateral reading for source vetting: investigate who is behind a source by reading what other reputable sources say about it.
    - Keep queries and excerpts non-sensitive; generalize details when searching.
    - If the topic is broad, start with an industry survey/landscape (what exists), then zoom in to 2-4 candidates.

6. **Synthesize**
    - Present 2-4 options.
    - Use a compact comparison table against the evaluation criteria.
    - Call out uncertainties and what evidence would reduce them.

7. **Recommend + justify**
    - Make a clear recommendation and specify conditions where you would choose a different option.
    - Provide an evidence map that ties each major claim to evidence and sources.
    - State your confidence level (high/medium/low) and why.
    - Include a reversal test: what new evidence would change your recommendation?

## Output template (deliver in-chat unless the user requests a file)

- **TL;DR**: recommendation in 1-3 bullets.
- **Problem statement**: what decision is being made and why now.
- **Pain points (confirmed)**: what hurts today.
- **What we know** (from provided materials): facts only; cite internal artifacts by name/path if available.
- **Constraints + success criteria**: including non-goals.
- **Open questions**: and which are answered vs pending.
- **Assumptions + limitations**: what you assumed, what you could not verify, and why.
- **Options considered**: 2-4 options, with a comparison table.
- **Industry best practices**: what's broadly adopted and why (with sources).
- **Recommendation**: what to do, why, and when not to.
- **Evidence map**: claim -> evidence -> source.
- **Risks + mitigations**: including rollout/migration plan at a high level if relevant.
- **Next steps**: concrete actions and owners.
- **Appendix**
    - **Search log**: query strings + date.
    - **Sources**: list of external sources (title, publisher, date, link; 3+ by default; 2 only with explicit justification).

## Evidence quality checklist (use for each key source)

- **Stop**: what is the claim, exactly?
- **Investigate the source**: who published it and why?
- **Find better coverage**: can you corroborate via more authoritative/primary sources?
- **Trace claims**: can you trace the claim back to original data, standards, or docs?

## Evidence tiers (practical heuristic)

Different domains have different norms. Use this as a default ordering, and explain when you deviate:

1. Standards, specifications, and official documentation (including changelogs and migration guides)
2. Peer-reviewed research, systematic reviews, and high-quality evidence syntheses (when applicable)
3. Incident reports and postmortems from well-established organizations (for operational claims)
4. Independent benchmarks and reproducible experiments (with methodology clearly stated)
5. Reputable books and long-form technical reports
6. Practitioner blog posts, community discussions, and vendor marketing (use carefully; corroborate)

## Notes

- If the topic is safety- or compliance-critical, treat web sources as guidance only and ask for domain constraints and required standards.
- If you cannot meet the 3-source default, say so early and propose a path to reach higher confidence (additional materials to request, which primary docs to locate, or a small experiment to run).
- If sources materially disagree, call it out explicitly and explain what you trust and why.

## Quick reference

- Minimum bar (default): 3 independent external sources + an evidence map (claim -> evidence -> source).
- Always include: options + tradeoffs + recommendation + confidence + “what would change my mind”.
- Keep a search log: queries + date.

## Common mistakes

- Recommending before reading provided materials or before confirming the actual pain points.
- Treating “two blog posts that cite each other” as independent evidence.
- Leaking internal identifiers in web queries (always generalize).

## References (for how to evaluate sources / record decisions)

- SIFT resources: `https://pressbooks.pub/webliteracy/front-matter/updated-resources-for-2021/`
- CRAAP test: `https://library.csuchico.edu/help/source-or-information-goodness-craap-test`
- Evidence-based management (CEBMa): `https://cebma.org/resources/frequently-asked-questions/what-is-evidence-based-management/`
- ADR pattern: `https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions`
- PRISMA 2020: `https://www.prisma-statement.org/`
- Thoughtworks Technology Radar: `https://www.thoughtworks.com/radar/`
