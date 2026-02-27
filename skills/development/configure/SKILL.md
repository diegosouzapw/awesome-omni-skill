---
name: configure
description: "Configure pattern-radar sources, weights, and domains."
allowed-tools:
  - mcp__pattern-radar__configure_sources
---

# Configure Your Radar

Customize which sources to scan and how to score relevance.

## Setting Domains

Your domains determine what's relevant to you:
```
configure_sources(domains: ["TypeScript", "React", "AI", "distributed systems"])
```

Domains can be:
- Programming languages (TypeScript, Python, Rust)
- Frameworks (React, Next.js, FastAPI)
- Technologies (WebAssembly, GraphQL, Kubernetes)
- Concepts (distributed systems, machine learning, security)

## Source Configuration

**Enable/disable sources:**
```
configure_sources(hackernews_enabled: true, github_enabled: true)
```

**Adjust weights** (0-2, higher = more influence):
```
configure_sources(
  hackernews_weight: 1.5,    # Prioritize HN
  github_weight: 0.5         # De-prioritize GitHub
)
```

**GitHub language filter:**
```
configure_sources(github_languages: ["TypeScript", "Rust", "Python"])
```

## Profile Integration

If shared-memory is installed, pattern-radar reads your profile automatically:
- Languages from `technical.languages`
- Frameworks from `technical.frameworks`
- Domains from `knowledge.domains`
- Interests from `knowledge.interests`

You can override or supplement with `configure_sources`.

## GitHub Token

For higher rate limits (5,000/hour vs 60/hour):

1. Create token at https://github.com/settings/tokens
2. Run pattern-radar setup: `node run.js`
3. Enter token when prompted

Token is saved to `~/.config/brain-jar/pattern-radar.json`

## Current Config

View current configuration by calling any radar tool—it shows your active settings.

## Reset to Defaults

Delete the config file:
```bash
rm ~/.config/brain-jar/pattern-radar.json
```
