---
name: fullstack-template-generator
description: Generates a complete fullstack application template with Python FastAPI backend and React Vite frontend. Includes OpenAI ChatGPT integration, CORS configuration, comprehensive error handling, and a modern Tailwind CSS + shadcn/ui React UI. Use this skill when the user wants to bootstrap a new fullstack web application project with both API backend and web frontend components ready to go.
---

# Fullstack Template Generator

## Overview

This skill automates the creation of a production-ready fullstack application template featuring:

### Backend (Python + FastAPI)
- FastAPI framework with async support
- OpenAI ChatGPT API integration
- CORS middleware configured for frontend communication
- Comprehensive error handling and validation
- Environment-based configuration
- Auto-generated API documentation (Swagger UI)
- Pydantic models for request validation

### Frontend (React + Vite)
- Modern React 19 with Vite 7 for fast development
- Tailwind CSS 3 configured with PostCSS + autoprefixer
- shadcn/ui primitives (Button, Card, Input, Textarea) powered by class-variance-authority, clsx, and tailwind-merge
- Lucide icons and Framer Motion for polished micro-interactions
- Axios for API communication
- Clean, responsive chat UI built entirely with Tailwind utilities
- Error handling and loading states
- Hot Module Replacement (HMR)

## What This Skill Creates

When invoked, this skill generates a complete project structure with:

```
project-name/
├── README.md
├── backend/
│   ├── .env.example
│   ├── .gitignore
│   ├── main.py
│   ├── requirements.txt
│   └── tests/
│       └── __init__.py
└── frontend/
    ├── .gitignore
    ├── index.html
    ├── package.json
    ├── vite.config.js
    ├── eslint.config.js
    ├── tailwind.config.js
    ├── postcss.config.js
    ├── public/
    │   └── vite.svg
    └── src/
        ├── App.jsx
        ├── main.jsx
        ├── index.css
        ├── lib/
        │   └── utils.js
        ├── components/
        │   └── ui/
        ├── assets/
        ├── hooks/
        ├── pages/
        └── styles/
```

## When to Use This Skill

Invoke this skill when the user:
- Wants to create a new fullstack web application
- Needs both a REST API backend and React frontend
- Requests a Python + React project setup
- Asks for a FastAPI + Vite template
- Wants OpenAI integration in their application
- Needs a quick start for a full-stack project

## How to Generate the Template

1. **Ask the user for the project name** and target directory location.
2. **Create the directory structure** as shown above.
3. **Copy template files** from the `templates/` directory:
   - Backend files from `templates/backend/`
   - Frontend files from `templates/frontend/`
   - Root README from `templates/README.md.template`
4. **For `.template` files**: Remove the `.template` suffix when copying.
5. **Ensure Tailwind/shadcn assets are included**:
   - Copy `tailwind.config.js`, `postcss.config.js`, and `src/index.css`
   - Copy `src/lib/utils.js` and the `src/components/ui` directory so shadcn primitives are ready to use
6. **Customize as needed**: Update project names in package.json if requested.
7. **Provide setup instructions** to the user:
   - Backend setup (create .env, install dependencies)
   - Frontend setup (install dependencies)
   - How to run both servers

## Key Features Included

### Backend API Endpoints
- `GET /` - Health check endpoint
- `GET /test` - Test connectivity
- `POST /chat` - OpenAI ChatGPT integration
  - Accepts: `{"message": "...", "model": "gpt-4-turbo-preview"}`
  - Returns: AI response with token usage

### Frontend Features
- Chat interface with input and send button
- Test endpoint button
- Real-time loading states
- Error display and handling
- Tailwind CSS-powered light theme using shadcn/ui components, Lucide icons, and Framer Motion animations
- Responsive design

### Configuration
- Environment variable management (.env)
- CORS configured for localhost:5173
- OpenAI API key integration
- Comprehensive error handling

## Post-Generation Instructions for User

After generating the template, provide these instructions:

```bash
# Backend Setup
cd project-name/backend
python -m venv venv
# Activate venv (Windows: venv\Scripts\activate, Mac/Linux: source venv/bin/activate)
pip install -r requirements.txt
# Create .env file and add OPENAI_API_KEY
python -m uvicorn main:app --reload

# Frontend Setup (in new terminal)
cd project-name/frontend
npm install
npm run dev
```

## Notes

- The template includes comprehensive README.md with full documentation
- All configuration files are pre-configured and ready to use
- Template supports both development and production deployments
- Includes .gitignore files to prevent committing sensitive data
- Ready for Git initialization and version control
