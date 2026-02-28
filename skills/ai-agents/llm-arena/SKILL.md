---
name: llm-arena
description: Full-stack LLM Arena application for AI vs AI game battles. Use when working on project-wide configurations, deployment, architecture decisions, or coordinating between frontend and backend. Covers Tic-Tac-Toe and Reversi games with OpenAI, Anthropic, and Google Gemini support.
compatibility: Node.js 20+, Python 3.10+, network access required for LLM API calls
metadata:
  tags: "ai, games, llm, full-stack"
---

# LLM Arena - Main Project Skill

## When to Use This Skill

Use this skill when:
- Setting up the project for the first time
- Making architecture decisions affecting both frontend and backend
- Coordinating changes across multiple components
- Deploying or configuring the application
- Debugging issues that span frontend and backend
- Adding new game types or LLM providers

## Project Overview

LLM Arena enables AI models to compete against each other in classic board games. The system consists of:
- **Backend:** FastAPI Python server handling LLM API calls
- **Frontend:** React + Vite application for game visualization
- **Supported Games:** Tic-Tac-Toe, Reversi
- **Supported Providers:** OpenAI, Anthropic, Google Gemini (via LiteLLM)

## Directory Structure

```
llm-arena/
├── SKILL.md                 # This file (main project skill)
├── progress.md              # Task tracking (auto-generated)
├── backend/
│   ├── SKILL.md            # Backend-specific skill
│   ├── main.py             # FastAPI application
│   ├── requirements.txt    # Python dependencies
│   └── test_game.py        # Backend tests
└── frontend/
    ├── SKILL.md            # Frontend-specific skill
    ├── src/
    │   ├── components/     # React components
    │   ├── hooks/          # Custom hooks (game logic)
    │   ├── services/       # API layer
    │   └── lib/            # Utilities
    ├── package.json
    └── vite.config.js
```

## MCP Server Integration

This project integrates with three MCP servers for enhanced development workflows:

### Sequential Thinking MCP

Helps break down complex problems step-by-step for better reasoning.

**When to Use:**
- Planning new feature implementations
- Debugging cross-component issues
- Designing game logic algorithms
- Architecting system changes

**Example Invocation:**
```
Use Sequential Thinking to plan adding a new game type:
1. What game rules need to be implemented?
2. What board representation is needed?
3. What prompts work best for the LLM?
4. What UI components are required?
5. What API changes are needed?
```

### Qdrant MCP Server

Vector search engine for saving and retrieving reusable code snippets, best practices, and documentation.

**Storage Keys:**
```
llm-arena:backend:progress:{timestamp}         # Progress snapshots
```

- After the completion of every task store the progress in Qdrant vector store using **qdrant-store**
- Save the progress in the format progress.md
- The file needs to include: TODO TASK, IN PROGRESS TASK, COMPLETED TASK


### Context7 MCP

Pulls live, up-to-date documentation directly into prompts.

**Supported Documentation Sources:**
- FastAPI (latest patterns, middleware, dependencies)
- React (hooks, server components, transitions)
- Vite (configuration, plugins, optimization)
- LiteLLM (provider setup, model naming, error handling)
- Tailwind CSS (utilities, configuration)

**Example Usage:**
```
Pull Context7 documentation for:
- FastAPI CORS middleware configuration
- React useEffect cleanup patterns
- LiteLLM Gemini provider setup
```

### Environment Variables
```bash
# backend/.env (never commit)
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
GEMINI_API_KEY=AI...
```

## Task Tracking Protocol

**IMPORTANT:** After completing any task, update `progress.md` and save to Qdrant.

### Progress File Format

Create/update `progress.md` in project root:

```markdown
# LLM Arena Progress Report
**Last Updated:** {ISO-8601 timestamp}
**Component:** {main|frontend|backend}

## Completed Tasks
- [x] {Task description} - {timestamp}
- [x] {Task description} - {timestamp}

## In Progress
- [ ] {Current task description}
  - Status: {percentage or description}
  - Blockers: {any blockers}

## Pending Tasks
- [ ] {Future task description}
- [ ] {Future task description}

## Technical Notes
- {Relevant observations}
- {Decisions made}
- {Issues encountered}

## Next Steps
1. {Immediate next action}
2. {Following action}
```

## Adding New Features

### Adding a New Game Type

1. **Backend (main.py):**
   - Add board initialization logic
   - Create game-specific prompt template
   - Add winning/scoring conditions

2. **Frontend (useGameLogic.js):**
   - Add board state initialization
   - Implement valid move calculation
   - Add win/draw detection

3. **Frontend (GameBoard.jsx):**
   - Add board rendering for new grid size
   - Style game pieces appropriately

4. **Frontend (GameSelectionScene.jsx):**
   - Add game card with icon and description

### Adding a New LLM Provider

1. **Frontend (SetupScene.jsx):**
   - Add to PROVIDERS array with recommended models

2. **Backend (main.py):**
   - Add API key environment variable handling
   - Add model name normalization if needed
   - Test with provider's API

## Debugging Checklist

```
[ ] Backend running? (curl http://localhost:8000/health)
[ ] Frontend running? (http://localhost:5173 loads)
[ ] CORS configured? (no browser console errors)
[ ] API keys valid? (test in provider's playground)
[ ] Model names correct? (check provider documentation)
[ ] Network requests succeeding? (DevTools Network tab)
[ ] Console errors? (both browser and terminal)
[ ] State updating correctly? (React DevTools)
```