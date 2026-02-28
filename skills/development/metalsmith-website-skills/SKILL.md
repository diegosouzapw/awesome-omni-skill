---
name: metalsmith-website-skills
version: 1.0.0
description: Build static websites using Metalsmith's component-based architecture. Triggers on phrases like "build me a website", "create a landing page", "help me make a site". Includes JavaScript and CSS development standards.
---

# Metalsmith Website Skills

This skill bundle enables building professional static websites through conversation. The user describes what they want, Claude handles the implementation.

## Included Skills

This bundle contains three complementary skills:

1. **component-builder/** - The main workflow for building Metalsmith websites
2. **javascript-development/** - JavaScript patterns and standards for any custom code
3. **css-layout-development/** - CSS layout patterns using modern intrinsic design

## When to Use Each Skill

- **Starting a website**: Read `component-builder/SKILL.md` first
- **Writing JavaScript** (browser scripts, custom build code): Consult `javascript-development/SKILL.md`
- **Creating CSS layouts** (custom components, style modifications): Consult `css-layout-development/SKILL.md`

## Quick Start Workflow

When a user wants to build a website:

1. Read `component-builder/SKILL.md` for the full workflow
2. Follow the 6 phases: Setup → Discovery → Components → Pages → Review → Publish
3. When writing any JavaScript, apply patterns from `javascript-development/SKILL.md`
4. When writing any CSS, apply patterns from `css-layout-development/SKILL.md`

## GitHub Integration

This workflow assumes the user will commit their site to GitHub for deployment via Netlify.

**For new users without GitHub:**
1. Guide them to create a free account at github.com
2. Help them create a new repository for their website
3. Configure git in their project to point to that repo
4. Handle commits and pushes on their behalf

**For returning users:**
1. Clone their existing repository
2. Make the requested changes
3. Commit and push the updates

## Target Audience

Users who:
- Have the Claude Desktop app installed
- Can follow basic instructions
- Want a website without learning technical details

Users do NOT need to:
- Know how to code
- Understand Git, Node.js, or build tools
- Have any development environment set up

Claude handles all technical aspects invisibly.
