---
name: kumo-assistant
description: Kumo development assistant for MySQL database management tool. Use when working on Kumo features, understanding architecture, writing tests, or navigating the codebase. Helps with React components, API endpoints, database features, and Electron app development.
---

# Kumo Development Assistant

Expert assistant for developing Kumo - a comprehensive cross-platform MySQL/MongoDB database management tool built with TypeScript, React, and Electron.

## When to Use This Skill

Use this skill when:
- Adding new features to Kumo
- Understanding the feature-based architecture
- Working with database connections (MySQL/MongoDB)
- Creating or modifying UI components
- Writing API endpoints
- Adding tests (Vitest unit tests or Playwright E2E tests)
- Working with the Electron desktop app
- Understanding project conventions and patterns

## Quick Start Guide

### Project Overview

**Kumo** is a cross-platform database management application similar to Navicat, featuring:
- **Database Support**: MySQL and MongoDB with multi-connection management
- **Query Tools**: Monaco editor with SQL syntax highlighting, visual query builder (React Flow)
- **Data Management**: Advanced data viewer with inline editing, import/export (CSV, JSON, SQL, Excel)
- **Development Tools**: API tester, terminal, Playwright integration, Git browser
- **AI Features**: Anthropic Claude integration for query assistance

**Tech Stack**:
- Frontend: React 18, TypeScript, TanStack Query, Tailwind CSS, Monaco Editor
- Backend: Node.js, Express, mysql2, mongodb driver, socket.io
- Desktop: Electron with secure IPC
- Testing: Vitest (unit), Playwright (E2E)
- State: Zustand (client), TanStack Query (server)

### Architecture Patterns

**Three-Tier Architecture**:
```
React Frontend → Node.js API Server → MySQL/MongoDB Database
```

**Key Principles**:
- Feature-based folder structure in `src/features/`
- Server-side connection pooling (never direct database access from browser)
- RESTful API with WebSocket for streaming operations
- Separation of server state (TanStack Query) and client state (Zustand)
- TypeScript path aliases: `@/*` for `src/*`, `@server/*` for `server/*`

For detailed architecture information, see [architecture.md](architecture.md).

### Common Development Tasks

#### 1. Adding a New Feature

**Location**: `src/features/<feature-name>/`

**Pattern**: Each feature is self-contained with:
```
src/features/myFeature/
├── MyFeatureComponent.tsx    # Main component
├── components/               # Sub-components
│   ├── SubComponent1.tsx
│   └── SubComponent2.tsx
└── utils/                    # Feature-specific utilities
    └── helpers.ts
```

**Steps**:
1. Create feature folder in `src/features/`
2. Build React components using functional components + hooks
3. Use Zustand for local feature state if needed
4. Use TanStack Query for server data fetching
5. Follow Tailwind CSS for styling
6. Add tests in `src/test/` or `tests/`

See [features.md](features.md) for detailed feature development guide.

#### 2. Creating API Endpoints

**Location**: `server/routes/`

**Pattern**:
```typescript
// server/routes/myRoute.ts
import { Router } from 'express';

const router = Router();

router.get('/api/my-endpoint', async (req, res) => {
  try {
    // Business logic here
    res.json({ success: true, data: result });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

export default router;
```

**Register in** `server/index.ts`:
```typescript
import myRoute from './routes/myRoute';
app.use(myRoute);
```

#### 3. Database Operations

**MySQL Example**:
```typescript
import { query } from '@server/utils/database';

const results = await query(connectionId, 'SELECT * FROM users WHERE id = ?', [userId]);
```

**MongoDB Example**:
```typescript
import { getMongoClient } from '@server/services/mongodb';

const client = await getMongoClient(connectionId);
const db = client.db(databaseName);
const collection = db.collection(collectionName);
const docs = await collection.find({ status: 'active' }).toArray();
```

**Important**: Always use parameterized queries to prevent SQL injection.

#### 4. Adding Tests

**Unit Tests (Vitest)**:
```bash
npm run test:unit              # Run all unit tests
npm run test:unit:ui           # Run with UI
```

Location: `src/test/components/`

**E2E Tests (Playwright)**:
```bash
npm run test                   # Run all E2E tests
npm run test:ui                # Run with Playwright UI
npm run test:headed            # Run in headed mode
```

Location: `tests/`

See [testing.md](testing.md) for comprehensive testing guide.

### Project Structure

```
Kumo/
├── src/                           # Frontend React application
│   ├── features/                  # Feature modules (20+ features)
│   │   ├── apiTester/            # REST API testing
│   │   ├── connections/          # Database connection management
│   │   ├── dataViewer/           # Data grid with inline editing
│   │   ├── mongodb/              # MongoDB features
│   │   ├── query/                # SQL editor and execution
│   │   ├── queryBuilder/         # Visual query builder
│   │   ├── schema/               # Schema exploration
│   │   └── ...                   # 13+ more features
│   ├── components/               # Shared UI components
│   ├── hooks/                    # Custom React hooks
│   ├── services/                 # Frontend API clients
│   ├── store/                    # Zustand state management
│   ├── types/                    # TypeScript type definitions
│   └── utils/                    # Utility functions
├── server/                       # Node.js Express API
│   ├── routes/                   # API route handlers
│   ├── services/                 # Business logic
│   ├── types/                    # Server types
│   └── utils/                    # Server utilities
├── electron/                     # Electron main process
│   ├── main.cjs                  # Entry point
│   └── preload.js                # Preload script
├── tests/                        # E2E tests (Playwright)
├── openspec/                     # Spec-driven development
│   ├── project.md                # Project conventions
│   ├── specs/                    # Feature specifications
│   └── changes/                  # Change proposals
└── .claude/                      # Claude Code configuration
    ├── commands/                 # Slash commands
    └── skills/                   # Agent skills
```

## Code Conventions

### TypeScript Style

- **2-space indentation**
- **Single quotes** for strings
- **Trailing commas** in multi-line objects/arrays
- **PascalCase** for components, **camelCase** for functions/variables
- **kebab-case** for file names

### React Patterns

- **Functional components** with hooks (no class components)
- **Props destructuring** in component signatures
- **Custom hooks** for reusable logic (prefix with `use`)
- **Memoization** with `useMemo`/`useCallback` for expensive operations

### State Management

- **Server State**: Use TanStack Query (`useQuery`, `useMutation`)
- **Client State**: Use Zustand stores for UI state
- **Form State**: Use controlled components with local `useState`

### API Patterns

- **RESTful endpoints** with proper HTTP methods (GET, POST, PUT, DELETE)
- **Error handling** with try-catch and appropriate status codes
- **Parameterized queries** to prevent SQL injection
- **Connection pooling** for database connections

## Development Workflow

### Running the Application

```bash
# Terminal 1: Frontend (Vite dev server)
npm run dev                    # http://localhost:5174

# Terminal 2: Backend API
npm run dev:server             # http://localhost:3001

# Terminal 3 (optional): Electron app
npm run dev:electron
```

### Building for Production

```bash
# Build frontend and server
npm run build

# Build Electron app for specific platform
npm run build:electron:win     # Windows
npm run build:electron:mac     # macOS
npm run build:electron:linux   # Linux
```

### Code Quality

```bash
npm run format                 # Format code with Prettier
npm run type-check             # TypeScript type checking
npm run lint                   # ESLint code linting
npm run clean                  # Clean build artifacts
```

## Key Features Overview

1. **Database Connections** (`src/features/connections/`)
   - Multi-database connection management
   - Secure credential storage
   - SSH tunnel support

2. **Query Editor** (`src/features/query/`)
   - Monaco editor with SQL syntax highlighting
   - Query execution and results display
   - Query history and saved queries
   - AI-powered query suggestions

3. **Visual Query Builder** (`src/features/queryBuilder/`)
   - Drag-and-drop query construction
   - React Flow for visual representation
   - Smart join suggestions based on foreign keys

4. **Data Viewer** (`src/features/dataViewer/`)
   - TanStack Table for virtualized data grid
   - Inline editing with validation
   - Pagination, filtering, sorting

5. **Schema Management** (`src/features/schema/`)
   - Visual schema browser
   - Relationship diagrams
   - Index management

6. **API Tester** (`src/features/apiTester/`)
   - REST API testing
   - cURL parsing and generation
   - Request/response management

7. **Git Integration** (`src/features/git/`)
   - Repository browser
   - Pull request viewer
   - Code diff viewer

## Security Considerations

- **Never expose credentials**: Store in OS keychain (desktop) or encrypted (web)
- **Parameterized queries**: Always use placeholders for user input
- **Connection limits**: Max 10 connections per database
- **Input validation**: Validate all user input on both client and server
- **Secure IPC**: Electron preload script for secure communication

## Performance Guidelines

- **Large result sets**: Warn at 10k rows, require confirmation for 100k+
- **Virtual scrolling**: Use TanStack Virtual for large lists
- **Code splitting**: Lazy load features with React.lazy()
- **Streaming**: Use streaming for large file imports/exports
- **Connection pooling**: Reuse database connections efficiently

## OpenSpec Integration

Kumo uses OpenSpec for spec-driven development. See `openspec/AGENTS.md` for:
- Creating change proposals
- Writing spec deltas
- Implementing changes
- Archiving completed work

## Getting Help

**Documentation**:
- [architecture.md](architecture.md) - Detailed architecture patterns
- [features.md](features.md) - Feature development guide
- [testing.md](testing.md) - Testing strategies and examples
- `openspec/project.md` - Project conventions
- `README.md` - Setup and usage guide

**Commands**:
```bash
openspec list                  # View active changes
openspec list --specs          # View feature specifications
npm run test:report            # View test reports
```

## Tips for AI Assistants

1. **Check existing features first**: Similar functionality may already exist
2. **Follow feature structure**: Keep features self-contained in `src/features/`
3. **Use path aliases**: Import with `@/*` instead of relative paths
4. **Write tests**: Add tests when creating new features
5. **Update specs**: Use OpenSpec for significant changes
6. **Security first**: Validate input, use parameterized queries
7. **Performance aware**: Consider virtual scrolling for large data sets
8. **Type safety**: Leverage TypeScript for better DX

## Next Steps

- Read [architecture.md](architecture.md) for in-depth technical patterns
- Review [features.md](features.md) before adding new features
- Check [testing.md](testing.md) for testing best practices
- Explore existing features in `src/features/` for examples
- Review `openspec/project.md` for project conventions
