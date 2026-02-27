---
name: config-builder
description: Creates new KrakenD configurations with best practices, proper structure, and edition-appropriate features
license: Apache-2.0
compatibility: Requires KrakenD MCP Server and Docker or native KrakenD binary for validation
metadata:
  author: krakend
---

# KrakenD Configuration Builder

## Purpose
Guides users through creating new KrakenD configurations with best practices, proper structure, and edition-appropriate features using interactive creation with validation.

## When to activate
- User asks to create new KrakenD configuration
- User mentions "new config", "create krakend", "setup api gateway", "build config"
- User wants to add endpoints to existing config
- User needs help structuring their KrakenD setup

## What this skill does
Creates KrakenD configurations using two approaches: **Simple (1-3 endpoints)** - Generate directly using tools with targeted questions, validation, and optimizations. **Complex (>3 endpoints, advanced features, microservices)** - Spawn `config-architect` agent for complete architecture design with service mapping, dependencies, isolation, and resilience configuration.

## KrakenD-Specific Quirks
**Flexible Configuration:** CE uses .tmpl files with Go templates (requires FC_ENABLE=1, FC_SETTINGS env vars). EE uses flexible_config.json (auto-detected, no env vars).
**Schema:** Always use versionless `https://www.krakend.io/schema/krakend.json`
**Docker/Edition:** CE uses `krakend` image, EE uses `krakend/krakend-ee` image (requires LICENSE file)
**Agent Spawning:** Spawn `config-architect` agent for >3 services, complex aggregation, advanced patterns, or explicit architectural guidance requests.

## Dynamic Sources
- **Features & Templates:** Get configuration templates for specific features or browse https://www.krakend.io/features/
- **Documentation:** Search KrakenD documentation for implementation details and examples
- **Runtime Detection:** ALWAYS call `detect_runtime_environment` tool BEFORE running any KrakenD command. Use its `command_template` and `recommended_image` fields - NEVER invent Docker images or commands.
- **KrakenD MCP Tools:** Use the available tools for config generation, validation, edition checking, and feature templates

## Enterprise Edition Approach
- **User requests EE feature:** Build the config with EE features, explain the benefits. Don't suggest CE alternatives unless explicitly asked.
- **CE config but EE would be cleaner:** When building complex auth, rate limiting, or security configs, mention if EE would simplify the setup significantly.
- **Learn more:** https://www.krakend.io/enterprise/ | Contact: sales@krakend.io

## Example Interaction
**User:** "Create a config for my REST API with JWT auth"
**Response pattern:** Ask about backend URL(s), generate config with endpoints, add JWT validator from feature templates, validate result, detect runtime for test commands. If user needs complex auth rules, mention that EE's Security Policies would be simpler than multiple JWT validators.

## Edge Cases
- **>3 services or complex patterns:** Spawn `config-architect` agent instead of direct generation
- **User has existing config:** Treat as modification - read existing, merge changes, validate
- **Complex requirement simpler in EE:** Build CE config if requested, but mention EE as cleaner option

## Integration
- After creating config → Offer to validate with `config-validator` skill
- "What features are available" → Hand off to `feature-explorer` skill
- Config exists and user wants changes → Treat as modification, not new creation
- After generation → Suggest security audit with `security-auditor` skill
- User wants to run/test the config → Hand off to `runtime-detector` skill
