---
name: backend-nodejs
description: Node.js/TypeScript backend expert. Handles Express/Fastify API routes, TypeScript strict mode, Prisma ORM, Zod validation, error handling, configuration management. Use when project is Node.js backend (package.json + TypeScript server).
allowed-tools: Read, Write, Edit, Bash, Glob, Grep
---

# Backend Node.js Skill - API Development Expert

> **Expert backend Node.js/TypeScript pour API REST + GraphQL**
>
> Inspiré de : Vercel API design, Stripe backend patterns, Prisma best practices

---

## Scope

**Chargé par:** executor agent (SI projet Node.js détecté)

**Détection auto:**
- Fichier `package.json` existe
- Fichiers `server.ts` ou `index.ts` ou `src/server.ts`
- Dependencies: `express`, `fastify`, `@prisma/client`, etc

**Stack supporté:**
- **Runtime:** Node.js 18+ / Bun
- **Language:** TypeScript (strict mode)
- **Frameworks:** Express, Fastify, Hono
- **Database:** Prisma ORM
- **Validation:** Zod
- **Testing:** Vitest, Jest

---

## Conventions Strictes

### 1. Structure Projet (Obligatoire)

```
backend/
├── src/
│   ├── config/
│   │   └── env.ts           # Configuration centralisée (1 seul fichier)
│   ├── routes/
│   │   ├── index.ts         # Router principal
│   │   ├── tasks.ts
│   │   └── users.ts
│   ├── services/
│   │   ├── task.service.ts  # Business logic
│   │   └── user.service.ts
│   ├── middleware/
│   │   ├── auth.ts
│   │   ├── error.ts
│   │   └── validation.ts
│   ├── types/
│   │   └── index.ts         # TypeScript types partagés
│   ├── lib/
│   │   ├── prisma.ts        # Prisma client singleton
│   │   └── logger.ts
│   └── server.ts            # Entry point
├── prisma/
│   └── schema.prisma
├── package.json
├── tsconfig.json
└── .env
```

---

### 2. Configuration (1 seul fichier - 12-Factor App)

```typescript
// src/config/env.ts
import { z } from 'zod'

const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'production', 'test']).default('development'),
  PORT: z.string().default('3000'),
  DATABASE_URL: z.string(),
  JWT_SECRET: z.string().min(32),
  CORS_ORIGIN: z.string().default('http://localhost:3000'),
  LOG_LEVEL: z.enum(['debug', 'info', 'warn', 'error']).default('info'),
})

export const env = envSchema.parse(process.env)

export type Env = z.infer<typeof envSchema>
```

**Principe:** Validation config au démarrage, crash immédiat si config invalide.

---

### 3. Prisma Client Singleton (Pattern Vercel)

```typescript
// src/lib/prisma.ts
import { PrismaClient } from '@prisma/client'
import { env } from '@/config/env'

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined
}

export const prisma =
  globalForPrisma.prisma ??
  new PrismaClient({
    log: env.NODE_ENV === 'development'
      ? ['query', 'error', 'warn']
      : ['error'],
  })

if (env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma
}
```

**Interdiction:** Créer plusieurs instances PrismaClient (épuise connexions DB).

---

### 4. Routes Pattern (Express)

```typescript
// src/routes/tasks.ts
import { Router } from 'express'
import { z } from 'zod'
import { taskService } from '@/services/task.service'
import { validateRequest } from '@/middleware/validation'
import { requireAuth } from '@/middleware/auth'

const router = Router()

// Zod schemas
const createTaskSchema = z.object({
  body: z.object({
    title: z.string().min(1).max(200),
    description: z.string().optional(),
    status: z.enum(['PENDING', 'IN_PROGRESS', 'COMPLETED']).default('PENDING'),
  }),
})

// Routes
router.post(
  '/',
  requireAuth,
  validateRequest(createTaskSchema),
  async (req, res, next) => {
    try {
      const task = await taskService.createTask(req.user.id, req.body)
      res.status(201).json(task)
    } catch (error) {
      next(error)
    }
  }
)

router.get(
  '/',
  requireAuth,
  async (req, res, next) => {
    try {
      const tasks = await taskService.getTasks(req.user.id)
      res.json(tasks)
    } catch (error) {
      next(error)
    }
  }
)

router.patch(
  '/:id',
  requireAuth,
  validateRequest(z.object({
    params: z.object({ id: z.string().cuid() }),
    body: createTaskSchema.shape.body.partial(),
  })),
  async (req, res, next) => {
    try {
      const task = await taskService.updateTask(
        req.params.id,
        req.user.id,
        req.body
      )
      res.json(task)
    } catch (error) {
      next(error)
    }
  }
)

export default router
```

**Pattern:** Routes légères → Business logic dans services.

---

### 5. Services Pattern (Singleton + Business Logic)

```typescript
// src/services/task.service.ts
import { prisma } from '@/lib/prisma'
import { TaskStatus } from '@prisma/client'

export class TaskService {
  async createTask(
    userId: string,
    data: { title: string; description?: string; status?: TaskStatus }
  ) {
    // Business logic validation
    const existingCount = await prisma.task.count({
      where: { userId, status: 'PENDING' },
    })

    if (existingCount >= 100) {
      throw new Error('Maximum pending tasks limit reached')
    }

    // Create with transaction if multiple operations
    return prisma.task.create({
      data: {
        ...data,
        userId,
      },
      include: {
        user: {
          select: { id: true, name: true, email: true },
        },
      },
    })
  }

  async getTasks(userId: string) {
    return prisma.task.findMany({
      where: { userId },
      include: {
        user: {
          select: { id: true, name: true },
        },
      },
      orderBy: { createdAt: 'desc' },
    })
  }

  async updateTask(
    id: string,
    userId: string,
    data: Partial<{ title: string; description?: string; status?: TaskStatus }>
  ) {
    // Check ownership
    const task = await prisma.task.findUnique({
      where: { id },
      select: { userId: true },
    })

    if (!task || task.userId !== userId) {
      throw new Error('Task not found or access denied')
    }

    return prisma.task.update({
      where: { id },
      data,
    })
  }

  async deleteTask(id: string, userId: string) {
    // Check ownership
    const task = await prisma.task.findUnique({
      where: { id },
      select: { userId: true },
    })

    if (!task || task.userId !== userId) {
      throw new Error('Task not found or access denied')
    }

    return prisma.task.delete({
      where: { id },
    })
  }
}

// Singleton export
export const taskService = new TaskService()
```

**Principe:** 1 service = 1 ressource, toute business logic centralisée.

---

### 6. Error Handling Middleware (Standardisé)

```typescript
// src/middleware/error.ts
import { Request, Response, NextFunction } from 'express'
import { ZodError } from 'zod'
import { Prisma } from '@prisma/client'
import { logger } from '@/lib/logger'

export class AppError extends Error {
  constructor(
    message: string,
    public statusCode: number = 500,
    public type: string = 'server_error'
  ) {
    super(message)
  }
}

export function errorHandler(
  error: Error,
  req: Request,
  res: Response,
  next: NextFunction
) {
  // Zod validation error
  if (error instanceof ZodError) {
    return res.status(400).json({
      message: 'Validation error',
      type: 'validation_error',
      errors: error.errors.map(e => ({
        path: e.path.join('.'),
        message: e.message,
      })),
    })
  }

  // Prisma errors
  if (error instanceof Prisma.PrismaClientKnownRequestError) {
    if (error.code === 'P2002') {
      return res.status(409).json({
        message: 'Resource already exists',
        type: 'conflict_error',
      })
    }
    if (error.code === 'P2025') {
      return res.status(404).json({
        message: 'Resource not found',
        type: 'not_found_error',
      })
    }
  }

  // App errors (custom)
  if (error instanceof AppError) {
    return res.status(error.statusCode).json({
      message: error.message,
      type: error.type,
    })
  }

  // Unknown errors
  logger.error('Unexpected error:', error)
  return res.status(500).json({
    message: 'Internal server error',
    type: 'server_error',
  })
}
```

---

### 7. Validation Middleware (Zod)

```typescript
// src/middleware/validation.ts
import { Request, Response, NextFunction } from 'express'
import { AnyZodObject, ZodError } from 'zod'

export function validateRequest(schema: AnyZodObject) {
  return async (req: Request, res: Response, next: NextFunction) => {
    try {
      await schema.parseAsync({
        body: req.body,
        query: req.query,
        params: req.params,
      })
      next()
    } catch (error) {
      if (error instanceof ZodError) {
        return res.status(400).json({
          message: 'Validation error',
          type: 'validation_error',
          errors: error.errors.map(e => ({
            path: e.path.join('.'),
            message: e.message,
          })),
        })
      }
      next(error)
    }
  }
}
```

---

### 8. Server Entry Point

```typescript
// src/server.ts
import express from 'express'
import cors from 'cors'
import helmet from 'helmet'
import { env } from '@/config/env'
import { errorHandler } from '@/middleware/error'
import { logger } from '@/lib/logger'
import router from '@/routes'

const app = express()

// Security middleware
app.use(helmet())
app.use(cors({ origin: env.CORS_ORIGIN }))
app.use(express.json({ limit: '10mb' }))

// Health check
app.get('/health', (req, res) => {
  res.json({ status: 'ok', timestamp: new Date().toISOString() })
})

// API routes
app.use('/api', router)

// Error handler (MUST be last)
app.use(errorHandler)

// Start server
const server = app.listen(env.PORT, () => {
  logger.info(`Server running on port ${env.PORT}`)
  logger.info(`Environment: ${env.NODE_ENV}`)
})

// Graceful shutdown
process.on('SIGTERM', () => {
  logger.info('SIGTERM received, shutting down gracefully')
  server.close(() => {
    logger.info('Server closed')
    process.exit(0)
  })
})

export default app
```

---

### 9. TypeScript Config (Strict Mode)

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "lib": ["ES2022"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

---

## Naming Conventions

### Variables & Functions (camelCase)

```typescript
const userId = "123"
const currentUser = await getUser()

async function createTask() {}
async function getUserTasks() {}
```

### Classes & Types (PascalCase)

```typescript
class TaskService {}
type TaskResponse = {}
interface UserData {}
```

### Files (kebab-case)

```
task.service.ts
user.controller.ts
auth.middleware.ts
```

### Constants (SCREAMING_SNAKE_CASE)

```typescript
const MAX_TASKS_PER_USER = 100
const DEFAULT_PAGE_SIZE = 20
```

---

## Anti-Patterns à Éviter

❌ **Multiple PrismaClient instances**
```typescript
// ❌ INTERDIT
const prisma = new PrismaClient()  // Dans chaque fichier
```

✅ **Singleton pattern**
```typescript
// ✅ CORRECT
import { prisma } from '@/lib/prisma'
```

---

❌ **Business logic dans routes**
```typescript
// ❌ INTERDIT
router.post('/', async (req, res) => {
  const task = await prisma.task.create({ data: req.body })
  res.json(task)
})
```

✅ **Business logic dans services**
```typescript
// ✅ CORRECT
router.post('/', async (req, res) => {
  const task = await taskService.createTask(req.user.id, req.body)
  res.json(task)
})
```

---

❌ **Pas de validation**
```typescript
// ❌ INTERDIT
router.post('/', async (req, res) => {
  const task = await taskService.createTask(req.body)  // Pas de validation
})
```

✅ **Zod validation middleware**
```typescript
// ✅ CORRECT
router.post('/', validateRequest(createTaskSchema), async (req, res) => {
  const task = await taskService.createTask(req.body)
})
```

---

❌ **Errors non catchés**
```typescript
// ❌ INTERDIT
router.get('/', async (req, res) => {
  const tasks = await taskService.getTasks()  // Crash si erreur
})
```

✅ **Try/catch + next(error)**
```typescript
// ✅ CORRECT
router.get('/', async (req, res, next) => {
  try {
    const tasks = await taskService.getTasks()
    res.json(tasks)
  } catch (error) {
    next(error)  // Error handler middleware
  }
})
```

---

## Checklist Feature API

**Avant créer route:**
- [ ] Service existe? (anti-duplication)
- [ ] Zod schema défini?
- [ ] Auth middleware si protected?
- [ ] Error handling (try/catch + next)?

**Après créer route:**
- [ ] Route exportée dans `routes/index.ts`?
- [ ] Types TypeScript définis?
- [ ] Ownership checks (si user data)?

---

## Exemples Complets

### Exemple 1: CRUD Tasks API

**Structure créée:**
```
src/
├── routes/tasks.ts          # Routes HTTP
├── services/task.service.ts # Business logic
├── types/task.ts            # TypeScript types
```

**Workflow:**
1. Zod schema validation
2. Route appelle service
3. Service utilise Prisma
4. Error handling middleware

---

### Exemple 2: Auth Middleware

```typescript
// src/middleware/auth.ts
import { Request, Response, NextFunction } from 'express'
import jwt from 'jsonwebtoken'
import { env } from '@/config/env'
import { AppError } from '@/middleware/error'

declare global {
  namespace Express {
    interface Request {
      user: { id: string; email: string }
    }
  }
}

export async function requireAuth(
  req: Request,
  res: Response,
  next: NextFunction
) {
  try {
    const token = req.headers.authorization?.replace('Bearer ', '')

    if (!token) {
      throw new AppError('Authentication required', 401, 'auth_error')
    }

    const decoded = jwt.verify(token, env.JWT_SECRET) as {
      id: string
      email: string
    }

    req.user = decoded
    next()
  } catch (error) {
    if (error instanceof jwt.JsonWebTokenError) {
      return next(new AppError('Invalid token', 401, 'auth_error'))
    }
    next(error)
  }
}
```

---

## Workflow Executor avec Backend-Node Skill

**Détection auto:**
```
1. executor scan projet
2. Trouve package.json + src/server.ts
3. Load backend-nodejs skill (au lieu de backend Python)
4. Applique conventions Node.js/TypeScript
```

**Feature création:**
```
User: "Crée CRUD tasks API"

executor:
1. Load skills: backend-nodejs + frontend + integration
2. Crée structure selon conventions Node.js
3. Zod validation + Prisma + Express routes
4. Services singleton pattern
5. Error handling standardisé
```

---

## Principes

1. **TypeScript Strict Mode** - Zéro `any`, types partout
2. **Configuration Centralisée** - 1 seul `env.ts` (12-Factor App)
3. **Services Singleton** - 1 service par ressource, réutilisable
4. **Validation Zod** - Toutes inputs validées
5. **Error Handling Standardisé** - Errors structurés + types
6. **Prisma Best Practices** - Singleton client, select optimization
7. **Separation of Concerns** - Routes → Services → Prisma

**Inspiré de:**
- Vercel API Routes design
- Stripe backend architecture (services pattern)
- Prisma documentation (singleton, transactions)
- Express.js best practices

---

**Version**: 1.0.0
**Created**: 2025-01-10
**Maintained by**: executor agent (loaded si projet Node.js détecté)
