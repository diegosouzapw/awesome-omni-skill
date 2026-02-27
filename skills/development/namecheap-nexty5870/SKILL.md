---
name: namecheap
description: Domain registration and management via Namecheap API.
metadata: {"clawdis":{"emoji":"🌐","requires":{"bins":["node"],"env":["NAMECHEAP_API_USER","NAMECHEAP_API_KEY"]},"primaryEnv":"NAMECHEAP_API_KEY"}}
---

# Namecheap Skill

Domain registration, availability checking, DNS management via Namecheap API.

## Check Availability

```bash
~/.nvm/versions/node/v22.21.1/bin/node {baseDir}/scripts/check.mjs example.com
~/.nvm/versions/node/v22.21.1/bin/node {baseDir}/scripts/check.mjs example.com example.io example.app
```

## Get Pricing

```bash
~/.nvm/versions/node/v22.21.1/bin/node {baseDir}/scripts/price.mjs com
~/.nvm/versions/node/v22.21.1/bin/node {baseDir}/scripts/price.mjs io app dev
```

## Search Available Domains

```bash
~/.nvm/versions/node/v22.21.1/bin/node {baseDir}/scripts/search.mjs myawesome startup
```

## List Your Domains

```bash
~/.nvm/versions/node/v22.21.1/bin/node {baseDir}/scripts/list.mjs
~/.nvm/versions/node/v22.21.1/bin/node {baseDir}/scripts/list.mjs --expiring
```

## Register Domain

```bash
~/.nvm/versions/node/v22.21.1/bin/node {baseDir}/scripts/register.mjs example.com           # Dry run
~/.nvm/versions/node/v22.21.1/bin/node {baseDir}/scripts/register.mjs example.com --confirm # Execute
```

⚠️ Registration will charge your Namecheap account!

## Renew Domain

```bash
~/.nvm/versions/node/v22.21.1/bin/node {baseDir}/scripts/renew.mjs example.com --confirm
```

## DNS Management

```bash
~/.nvm/versions/node/v22.21.1/bin/node {baseDir}/scripts/dns.mjs example.com                    # List
~/.nvm/versions/node/v22.21.1/bin/node {baseDir}/scripts/dns.mjs example.com --add A @ 1.2.3.4 # Add
~/.nvm/versions/node/v22.21.1/bin/node {baseDir}/scripts/dns.mjs example.com --delete A @      # Delete
```

## Setup

See README.md for full setup instructions. Required env vars:

- `NAMECHEAP_API_USER` - Your Namecheap username
- `NAMECHEAP_API_KEY` - Your API key
- `NAMECHEAP_SANDBOX` - Set to "true" for testing (optional)

## Notes

- API requires IP whitelisting in Namecheap dashboard
- Use sandbox mode for testing: `NAMECHEAP_SANDBOX=true`
- Registration requires contact info env vars (see README)
