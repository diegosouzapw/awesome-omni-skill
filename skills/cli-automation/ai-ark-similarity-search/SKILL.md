---
name: AI Ark Similarity Search
description: |
  Find lookalike companies based on a seed company profile using AI Ark's
  AI-powered similarity matching. Provide a company domain or name and get
  back companies with similar firmographic profiles ranked by similarity score.
  Use when a user says "find companies similar to", "lookalike companies",
  "find more like", "companies like Stripe", or wants to expand their target
  account list from a seed company or their best customer.
  This is AI Ark's unique differentiator — no Apollo equivalent exists.
  Each search costs ~2.5 credits. Accepts up to 5 seed domains.
metadata:
  author: sixty-ai
  version: "2"
  category: enrichment
  skill_type: atomic
  is_active: true
  agent_affinity:
    - prospecting

  triggers:
    - pattern: "find companies similar to"
      intent: "similarity_search"
      confidence: 0.90
      examples:
        - "find 100 companies similar to Stripe"
        - "find companies like HubSpot"
        - "show me lookalikes for Notion"
        - "find companies similar to our top customer"
        - "get companies that look like Salesforce"
        - "find companies similar to stripe.com"
        - "show me 50 companies like Figma"
    - pattern: "lookalike companies"
      intent: "lookalike"
      confidence: 0.90
      examples:
        - "build a lookalike list from Figma"
        - "get lookalike companies for our top customer"
        - "create a lookalike prospecting list based on our best accounts"
        - "find lookalikes for our closed-won deals"
    - pattern: "ai ark similarity"
      intent: "ai_ark_similarity"
      confidence: 0.95
      examples:
        - "use ai ark similarity search"
        - "ai ark lookalike search"
    - pattern: "expand from seed company"
      intent: "seed_expansion"
      confidence: 0.75
      examples:
        - "use this company as a seed to find similar ones"
        - "find more companies like this one"
        - "expand my account list using these companies as seeds"

  keywords:
    - "similar to"
    - "lookalike"
    - "like"
    - "similarity"
    - "ai ark"
    - "seed company"
    - "find more like"
    - "companies like"
    - "same profile"
    - "ICP match"

  required_context: []

  inputs:
    - name: seed_company_domain
      type: string
      description: "Domain of the seed company (e.g. 'stripe.com', 'notion.so'). Up to 5 domains can be passed as array for multi-seed matching."
      required: false
    - name: seed_company_name
      type: string
      description: "Name of the seed company (used if domain not provided — AI Ark resolves to domain internally)"
      required: false
    - name: match_count
      type: number
      description: "Number of similar companies to return (default: 25, max: 100)"
      required: false
    - name: preview_mode
      type: boolean
      description: "Return only 5 results for quick preview before committing to a full search"
      required: false

  outputs:
    - name: companies
      type: array
      description: "List of lookalike companies with firmographic data (sorted by similarity)"
    - name: seed_profile
      type: object
      description: "The resolved profile of the seed company"
    - name: total_count
      type: number
      description: "Total lookalikes found in AI Ark database"
    - name: estimated_credit_cost
      type: object
      description: "Credit cost breakdown: { search_cost, description }"

  requires_capabilities:
    - ai_ark_api

  priority: high

  tags:
    - enrichment
    - companies
    - similarity
    - lookalike
    - ai-ark
---

## Available Context
@_platform-references/org-variables.md

# AI Ark Similarity Search

## Goal
Find companies similar to a seed company using AI Ark's AI-powered similarity matching engine.
This uses the `lookalikeDomains` parameter — AI Ark's unique differentiator with no Apollo equivalent.

## Credit Cost
Each similarity search costs **~2.5 credits** (uses the company search endpoint with `lookalikeDomains`).
Use `preview_mode: true` to fetch 5 results first (same cost) before running a full search.
Always warn the user before executing.

## Required Capabilities
- **AI Ark API**: Similarity search via `ai-ark-search` edge function (`action: 'company_search'` with `lookalike_domains`)

## Inputs
- `seed_company_domain`: Domain of the target company (preferred). Up to 5 domains for multi-seed matching.
  Examples: 'stripe.com', 'notion.so', 'hubspot.com', 'figma.com', 'salesforce.com'
- `seed_company_name`: Company name fallback if domain unknown
- `match_count`: How many lookalikes to find (default: 25, max: 100)
- `preview_mode`: Fetch 5 results first for quick validation

## Execution
1. Identify the seed company domain (ask user if unclear)
2. Warn the user: "This search will cost ~2.5 AI Ark credits. Proceed?" (or offer preview first)
3. Optionally run with `preview_mode: true` to show 5 results
4. On confirmation, call `ai-ark-search` with seed domain(s)
5. Present results as a table for review

## Output Contract
Return a table with columns:
- Company Name, Domain, Industry, Employee Count, Employee Range, Location, Description, Technologies, LinkedIn, Website

Show `total_count` to indicate how many lookalikes exist in the database.

## Key Differentiator
Uses AI Ark's `lookalikeDomains` parameter on the company search endpoint.
Accepts up to 5 seed domains or LinkedIn URLs. AI Ark matches by firmographic profile similarity
(industry, size, tech stack, growth signals) — not keyword matching.

## Common Patterns
- Seed from best customer -> find 100 lookalikes -> people search for VPs -> outbound
- Seed from closed-won deal -> lookalikes -> pipeline expansion
- Multi-seed: pass 3-5 top accounts as seeds -> find companies matching that ICP profile
