---
name: integration
description: Backend-Frontend integration patterns expert. Type-safe API contracts with Pydantic-Zod validation sync (Python FastAPI) or Prisma-TypeScript native (Next.js). Shadcn forms connected to backend, error handling, loading states. Use when creating full-stack features.
allowed-tools: Read, Write, Edit, Glob, Grep
---

# Integration Skill - Backend ↔ Frontend Patterns

> **Expert intégration backend (Python FastAPI ou Next.js) ↔ frontend React + shadcn**
>
> Inspiré de : Stripe API patterns, Vercel full-stack architecture, Prisma best practices

---

## Scope

**Chargé par:** executor agent (quand feature full-stack détectée)

**Deux chemins d'intégration:**

### Path A: Python FastAPI + React + shadcn (REST API)
- Backend: Python + FastAPI + Pydantic
- Frontend: React + Next.js + shadcn
- Communication: REST API (JSON)
- Validation: Pydantic backend ↔ Zod frontend (mirrored)

### Path B: Next.js Full-Stack + React + shadcn (Server Actions)
- Backend: Next.js Server Actions + Prisma
- Frontend: React + Next.js + shadcn
- Communication: Server Actions (RPC-like)
- Validation: Zod + Prisma native types

**Patterns couverts:**
1. **Type-safe API contracts** (Pydantic → TypeScript OU Prisma → TypeScript native)
2. **Validation synchronisée** (Pydantic ↔ Zod OU Zod + Prisma enums)
3. **Forms shadcn → Backend** (FastAPI routes OU Server Actions)
4. **Error handling** (standardisé backend → UI feedback)
5. **Loading states** (React Query OU useOptimistic)
6. **Database patterns** (Prisma best practices)

---

## Pattern #1: Type-Safe API Contract (Path A - FastAPI)

### Backend (Pydantic schemas)

```python
# backend/app/schemas/task.py
from pydantic import BaseModel, Field, validator
from datetime import datetime
from typing import Literal

class TaskBase(BaseModel):
    title: str = Field(..., min_length=1, max_length=200)
    description: str | None = None
    status: Literal["pending", "in_progress", "completed"] = "pending"
    priority: Literal["low", "medium", "high"] = "medium"

    @validator("title")
    def title_not_empty(cls, v):
        if not v.strip():
            raise ValueError("Title cannot be empty")
        return v.strip()

class TaskCreate(TaskBase):
    """Input creation (pas d'ID)"""
    pass

class TaskUpdate(BaseModel):
    """Update partiel (tous optionnels)"""
    title: str | None = Field(None, min_length=1, max_length=200)
    description: str | None = None
    status: Literal["pending", "in_progress", "completed"] | None = None
    priority: Literal["low", "medium", "high"] | None = None

class TaskResponse(TaskBase):
    """Output avec metadata DB"""
    id: str
    created_at: datetime
    updated_at: datetime
    user_id: str

    class Config:
        from_attributes = True
```

### Frontend (TypeScript types synchronisés)

```typescript
// frontend/src/types/task.ts

// Mirror exact Pydantic schemas
export type TaskStatus = "pending" | "in_progress" | "completed"
export type TaskPriority = "low" | "medium" | "high"

export interface TaskBase {
  title: string
  description?: string | null
  status: TaskStatus
  priority: TaskPriority
}

export interface TaskCreate extends TaskBase {
  // Pas d'ID pour création
}

export interface TaskUpdate {
  // Tous optionnels pour update partiel
  title?: string
  description?: string | null
  status?: TaskStatus
  priority?: TaskPriority
}

export interface TaskResponse extends TaskBase {
  id: string
  created_at: string  // ISO string du datetime Python
  updated_at: string
  user_id: string
}
```

**Principe:** Types frontend **MIRRORED** exactement depuis Pydantic pour type-safety.

---

## Pattern #2: Validation Synchronisée (Pydantic ↔ Zod)

### Backend Validation (Pydantic)

```python
# backend/app/schemas/task.py
class TaskCreate(BaseModel):
    title: str = Field(..., min_length=1, max_length=200)
    description: str | None = None

    @validator("title")
    def title_not_empty(cls, v):
        if not v.strip():
            raise ValueError("Title cannot be empty")
        return v.strip()
```

### Frontend Validation (Zod - SYNCHRONISÉ)

```typescript
// frontend/src/schemas/task.ts
import { z } from "zod"

// Mirror EXACT validation Pydantic
export const taskCreateSchema = z.object({
  title: z.string()
    .min(1, "Title cannot be empty")
    .max(200, "Title too long")
    .refine(val => val.trim().length > 0, "Title cannot be empty"),
  description: z.string().optional().nullable(),
  status: z.enum(["pending", "in_progress", "completed"]).default("pending"),
  priority: z.enum(["low", "medium", "high"]).default("medium"),
})

export type TaskCreateInput = z.infer<typeof taskCreateSchema>
```

**RÈGLE CRITIQUE:** Validation frontend **identique** backend.
- Même min/max lengths
- Même regex patterns
- Même error messages (traduits si nécessaire)

**Principe:** Frontend validation = UX rapide, Backend validation = sécurité.
(Defense in depth - jamais trust client)

---

## Pattern #3: Shadcn Form → FastAPI Complete

### Backend API Route

```python
# backend/app/api/routes/tasks.py
from fastapi import APIRouter, HTTPException, Depends
from app.schemas.task import TaskCreate, TaskResponse
from app.services.task_service import task_service

router = APIRouter(prefix="/tasks", tags=["tasks"])

@router.post("/", response_model=TaskResponse, status_code=201)
async def create_task(
    task: TaskCreate,
    user_id: str = Depends(get_current_user)
):
    """Crée nouvelle task avec validation"""
    try:
        return task_service.create_task(user_id, task)
    except ValueError as e:
        raise HTTPException(status_code=400, detail=str(e))
    except Exception as e:
        raise HTTPException(status_code=500, detail="Internal server error")
```

### Frontend API Client

```typescript
// frontend/src/lib/api/tasks.ts
import { TaskCreate, TaskResponse } from "@/types/task"

const API_BASE = process.env.NEXT_PUBLIC_API_URL || "http://localhost:8000"

export async function createTask(task: TaskCreate): Promise<TaskResponse> {
  const response = await fetch(`${API_BASE}/api/tasks`, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "Authorization": `Bearer ${getToken()}`,
    },
    body: JSON.stringify(task),
  })

  if (!response.ok) {
    const error = await response.json()
    throw new Error(error.detail || "Failed to create task")
  }

  return response.json()
}
```

### Shadcn Form Component (Complete)

```tsx
// frontend/src/components/task-form.tsx
"use client"

import { zodResolver } from "@hookform/resolvers/zod"
import { useForm } from "react-hook-form"
import { useMutation, useQueryClient } from "@tanstack/react-query"
import { toast } from "sonner"

import { Button } from "@/components/ui/button"
import { Input } from "@/components/ui/input"
import { Textarea } from "@/components/ui/textarea"
import {
  Form,
  FormControl,
  FormField,
  FormItem,
  FormLabel,
  FormMessage,
} from "@/components/ui/form"
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from "@/components/ui/select"

import { taskCreateSchema, type TaskCreateInput } from "@/schemas/task"
import { createTask } from "@/lib/api/tasks"

export function TaskForm({ onSuccess }: { onSuccess?: () => void }) {
  const queryClient = useQueryClient()

  // Form avec Zod validation
  const form = useForm<TaskCreateInput>({
    resolver: zodResolver(taskCreateSchema),
    defaultValues: {
      title: "",
      description: "",
      status: "pending",
      priority: "medium",
    },
  })

  // Mutation avec React Query
  const mutation = useMutation({
    mutationFn: createTask,
    onSuccess: () => {
      // Invalidate cache pour refresh liste
      queryClient.invalidateQueries({ queryKey: ["tasks"] })
      toast.success("Task created successfully")
      form.reset()
      onSuccess?.()
    },
    onError: (error: Error) => {
      toast.error(error.message)
    },
  })

  const onSubmit = (data: TaskCreateInput) => {
    mutation.mutate(data)
  }

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-4">
        {/* Title */}
        <FormField
          control={form.control}
          name="title"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Title</FormLabel>
              <FormControl>
                <Input placeholder="Enter task title" {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />

        {/* Description */}
        <FormField
          control={form.control}
          name="description"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Description</FormLabel>
              <FormControl>
                <Textarea
                  placeholder="Enter task description"
                  {...field}
                  value={field.value || ""}
                />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />

        {/* Priority */}
        <FormField
          control={form.control}
          name="priority"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Priority</FormLabel>
              <Select onValueChange={field.onChange} defaultValue={field.value}>
                <FormControl>
                  <SelectTrigger>
                    <SelectValue placeholder="Select priority" />
                  </SelectTrigger>
                </FormControl>
                <SelectContent>
                  <SelectItem value="low">Low</SelectItem>
                  <SelectItem value="medium">Medium</SelectItem>
                  <SelectItem value="high">High</SelectItem>
                </SelectContent>
              </Select>
              <FormMessage />
            </FormItem>
          )}
        />

        {/* Submit */}
        <Button type="submit" disabled={mutation.isPending}>
          {mutation.isPending ? "Creating..." : "Create Task"}
        </Button>
      </form>
    </Form>
  )
}
```

**Checklist connexion complète:**
- ✅ Zod schema mirrored depuis Pydantic
- ✅ TypeScript types synchronisés
- ✅ React Hook Form + Zod resolver
- ✅ React Query mutation (loading + error states)
- ✅ Toast notifications (success/error)
- ✅ Cache invalidation (refresh après create)
- ✅ Form reset après success

---

## Pattern #4: Error Handling (Backend → Frontend)

### Backend Error Responses (standardisé)

```python
# backend/app/api/routes/tasks.py
from fastapi import HTTPException
from pydantic import ValidationError

@router.post("/")
async def create_task(task: TaskCreate):
    try:
        return task_service.create_task(task)

    except ValueError as e:
        # Business logic error
        raise HTTPException(
            status_code=400,
            detail={"message": str(e), "type": "validation_error"}
        )

    except PermissionError as e:
        # Authorization error
        raise HTTPException(
            status_code=403,
            detail={"message": str(e), "type": "permission_error"}
        )

    except Exception as e:
        # Unexpected error (log + generic message)
        logger.error(f"Unexpected error: {e}", exc_info=True)
        raise HTTPException(
            status_code=500,
            detail={"message": "Internal server error", "type": "server_error"}
        )
```

### Frontend Error Handling

```typescript
// frontend/src/lib/api/client.ts
export class APIError extends Error {
  constructor(
    message: string,
    public status: number,
    public type?: string
  ) {
    super(message)
  }
}

export async function apiClient<T>(
  url: string,
  options?: RequestInit
): Promise<T> {
  const response = await fetch(url, {
    ...options,
    headers: {
      "Content-Type": "application/json",
      ...options?.headers,
    },
  })

  if (!response.ok) {
    const error = await response.json().catch(() => ({}))
    throw new APIError(
      error.detail?.message || error.detail || "Request failed",
      response.status,
      error.detail?.type
    )
  }

  return response.json()
}
```

```tsx
// frontend/src/components/task-form.tsx
const mutation = useMutation({
  mutationFn: createTask,
  onError: (error: APIError) => {
    // Error handling basé sur type
    switch (error.type) {
      case "validation_error":
        toast.error(`Validation error: ${error.message}`)
        break
      case "permission_error":
        toast.error("You don't have permission to perform this action")
        break
      case "server_error":
        toast.error("Server error. Please try again later.")
        break
      default:
        toast.error(error.message)
    }
  },
})
```

**Principe:** Errors backend standardisés avec types → Frontend gère selon type.
(Inspiration: Stripe API errors)

---

## Pattern #5: Loading States (React Query)

### Liste avec Loading/Error/Empty States

```tsx
// frontend/src/components/task-list.tsx
"use client"

import { useQuery } from "@tanstack/react-query"
import { Card } from "@/components/ui/card"
import { Skeleton } from "@/components/ui/skeleton"
import { Alert, AlertDescription } from "@/components/ui/alert"
import { getTasks } from "@/lib/api/tasks"

export function TaskList() {
  const { data, isLoading, error } = useQuery({
    queryKey: ["tasks"],
    queryFn: getTasks,
  })

  // Loading state
  if (isLoading) {
    return (
      <div className="space-y-4">
        {[...Array(3)].map((_, i) => (
          <Card key={i} className="p-4">
            <Skeleton className="h-6 w-3/4 mb-2" />
            <Skeleton className="h-4 w-full" />
          </Card>
        ))}
      </div>
    )
  }

  // Error state
  if (error) {
    return (
      <Alert variant="destructive">
        <AlertDescription>
          Failed to load tasks: {error.message}
        </AlertDescription>
      </Alert>
    )
  }

  // Empty state
  if (!data || data.length === 0) {
    return (
      <div className="text-center p-8 text-muted-foreground">
        No tasks yet. Create your first task!
      </div>
    )
  }

  // Data state
  return (
    <div className="space-y-4">
      {data.map((task) => (
        <Card key={task.id} className="p-4">
          <h3 className="font-semibold">{task.title}</h3>
          <p className="text-sm text-muted-foreground">{task.description}</p>
        </Card>
      ))}
    </div>
  )
}
```

**Checklist states:**
- ✅ Loading (Skeleton UI)
- ✅ Error (Alert destructive)
- ✅ Empty (Message vide)
- ✅ Data (Liste tasks)

---

## Pattern #6: Optimistic Updates

```tsx
// frontend/src/hooks/useTasks.ts
import { useMutation, useQueryClient } from "@tanstack/react-query"
import { updateTask } from "@/lib/api/tasks"
import type { TaskResponse, TaskUpdate } from "@/types/task"

export function useUpdateTask() {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: ({ id, data }: { id: string; data: TaskUpdate }) =>
      updateTask(id, data),

    // Optimistic update AVANT requête serveur
    onMutate: async ({ id, data }) => {
      // Cancel outgoing queries
      await queryClient.cancelQueries({ queryKey: ["tasks"] })

      // Snapshot previous value
      const previousTasks = queryClient.getQueryData<TaskResponse[]>(["tasks"])

      // Optimistically update cache
      queryClient.setQueryData<TaskResponse[]>(["tasks"], (old) =>
        old?.map((task) =>
          task.id === id ? { ...task, ...data } : task
        ) || []
      )

      // Return context avec previous value
      return { previousTasks }
    },

    // Si erreur → Rollback
    onError: (err, variables, context) => {
      if (context?.previousTasks) {
        queryClient.setQueryData(["tasks"], context.previousTasks)
      }
      toast.error("Failed to update task")
    },

    // Après success → Refetch pour sync
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ["tasks"] })
    },
  })
}
```

**Principe:** Update UI immédiatement (optimistic), rollback si erreur serveur.
(Stripe Dashboard pattern)

---

## Checklist Intégration Full-Stack

### Backend (FastAPI + Pydantic)
- [ ] Pydantic schemas définis (Base, Create, Update, Response)
- [ ] Validation Pydantic complète (min/max, regex, custom validators)
- [ ] API routes avec error handling standardisé
- [ ] Services layer (business logic séparée)
- [ ] Error responses structurées (message + type)

### Frontend (React + shadcn + Zod)
- [ ] TypeScript types mirrored depuis Pydantic
- [ ] Zod schemas synchronisés avec Pydantic validation
- [ ] API client avec error handling
- [ ] React Query setup (mutations + queries)
- [ ] Shadcn forms avec react-hook-form + Zod resolver
- [ ] Loading states (Skeleton UI)
- [ ] Error states (Alert components)
- [ ] Empty states (messages appropriés)
- [ ] Success feedback (toast notifications)
- [ ] Cache invalidation (après mutations)
- [ ] Optimistic updates (si applicable)

---

## Anti-Patterns à Éviter

❌ **Validation différente frontend vs backend**
```typescript
// Frontend: max 100 chars
z.string().max(100)

// Backend: max 200 chars
Field(..., max_length=200)

// → Incohérence = bugs
```

✅ **Validation synchronisée exactement**

---

❌ **Types frontend pas à jour**
```typescript
// Backend ajouté field "priority"
// Frontend oublie → TypeScript errors partout
```

✅ **Types générés ou mirrored manuellement avec checklist**

---

❌ **Pas de loading states**
```tsx
{data?.map(...)}  // Pas de loading → Flash vide
```

✅ **Skeleton UI pendant loading**

---

❌ **Errors non gérés**
```tsx
onError: () => {}  // Error silencieux = mauvaise UX
```

✅ **Toast + error messages clairs**

---

## Workflow Executor avec Integration Skill

**Quand executor crée feature full-stack:**

```
1. executor détecte: feature nécessite backend + frontend
2. Load skills: backend + frontend + integration
3. Phase A - Backend:
   - Crée Pydantic schemas (selon integration patterns)
   - Crée API route avec error handling standardisé
   - Crée service layer
4. Phase B - Frontend:
   - Crée TypeScript types (mirrored Pydantic)
   - Crée Zod schemas (synchronized validation)
   - Crée API client
   - Crée shadcn form (complete pattern)
   - Loading/Error/Empty states
5. Validation:
   - Checklist intégration complète
   - Types synchronisés ✓
   - Validation identique ✓
   - Error handling ✓
```

---

## Principes

1. **Type-Safety First** - Types synchronisés Pydantic ↔ TypeScript
2. **Validation Mirrored** - Frontend validation = Backend validation
3. **Error Handling Standardisé** - Errors structurés backend → Frontend gère
4. **Loading States Obligatoires** - Skeleton UI, pas flash vide
5. **Optimistic Updates** - Meilleure UX, rollback si erreur
6. **Defense in Depth** - Validation frontend (UX) + backend (sécurité)

**Inspiré de:**
- Stripe Dashboard (error handling + loading states + optimistic updates)
- Vercel (full-stack TypeScript patterns)

---

**Version**: 1.0.0
**Last updated**: 2025-01-10
**Maintained by**: executor agent (loaded pour features full-stack)

---

## Pattern #7: Prisma + Next.js Full-Stack (Path B - Type-Safe Native)

### Database Schema (Prisma)

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  tasks     Task[]

  @@index([email])
}

model Task {
  id          String   @id @default(cuid())
  title       String
  description String?
  status      TaskStatus @default(PENDING)
  priority    TaskPriority @default(MEDIUM)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  user        User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  userId      String

  @@index([userId])
  @@index([status])
}

enum TaskStatus {
  PENDING
  IN_PROGRESS
  COMPLETED
}

enum TaskPriority {
  LOW
  MEDIUM
  HIGH
}
```

**Commandes Prisma:**
```bash
# Générer client TypeScript
npx prisma generate

# Créer migration
npx prisma migrate dev --name add_tasks

# Push schema (dev rapide)
npx prisma db push

# Ouvrir Prisma Studio
npx prisma studio
```

---

### Prisma Client Singleton (Pattern Vercel)

```typescript
// lib/prisma.ts
import { PrismaClient } from '@prisma/client'

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined
}

export const prisma =
  globalForPrisma.prisma ??
  new PrismaClient({
    log: process.env.NODE_ENV === 'development' ? ['query', 'error', 'warn'] : ['error'],
  })

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma
```

**Principe:** Singleton pour éviter épuisement connexions DB (Vercel best practice)

---

### Next.js API Route avec Prisma (Server Actions)

```typescript
// app/actions/tasks.ts
'use server'

import { revalidatePath } from 'next/cache'
import { prisma } from '@/lib/prisma'
import { z } from 'zod'
import { TaskStatus, TaskPriority } from '@prisma/client'

// Validation Zod (synchronized avec Prisma enums)
const taskCreateSchema = z.object({
  title: z.string().min(1, "Title required").max(200, "Title too long"),
  description: z.string().optional(),
  status: z.nativeEnum(TaskStatus).default(TaskStatus.PENDING),
  priority: z.nativeEnum(TaskPriority).default(TaskPriority.MEDIUM),
  userId: z.string().cuid(),
})

export type TaskCreateInput = z.infer<typeof taskCreateSchema>

export async function createTask(input: TaskCreateInput) {
  try {
    // Validation
    const validated = taskCreateSchema.parse(input)

    // Prisma create
    const task = await prisma.task.create({
      data: validated,
      include: {
        user: {
          select: { id: true, name: true, email: true }
        }
      }
    })

    // Revalidate cache
    revalidatePath('/dashboard')

    return { success: true, data: task }
  } catch (error) {
    if (error instanceof z.ZodError) {
      return { success: false, error: error.errors[0].message }
    }
    return { success: false, error: "Failed to create task" }
  }
}

export async function getTasks(userId: string) {
  const tasks = await prisma.task.findMany({
    where: { userId },
    include: {
      user: {
        select: { id: true, name: true }
      }
    },
    orderBy: { createdAt: 'desc' }
  })

  return tasks
}

export async function updateTask(id: string, userId: string, data: Partial<TaskCreateInput>) {
  try {
    // Check ownership
    const task = await prisma.task.findUnique({
      where: { id },
      select: { userId: true }
    })

    if (!task || task.userId !== userId) {
      return { success: false, error: "Task not found or access denied" }
    }

    // Update
    const updated = await prisma.task.update({
      where: { id },
      data,
    })

    revalidatePath('/dashboard')

    return { success: true, data: updated }
  } catch (error) {
    return { success: false, error: "Failed to update task" }
  }
}

export async function deleteTask(id: string, userId: string) {
  try {
    // Check ownership
    const task = await prisma.task.findUnique({
      where: { id },
      select: { userId: true }
    })

    if (!task || task.userId !== userId) {
      return { success: false, error: "Task not found or access denied" }
    }

    await prisma.task.delete({
      where: { id }
    })

    revalidatePath('/dashboard')

    return { success: true }
  } catch (error) {
    return { success: false, error: "Failed to delete task" }
  }
}
```

**Avantages Server Actions:**
- ✅ Type-safe natif (Prisma types auto-générés)
- ✅ Pas de route API explicite
- ✅ Revalidation cache Next.js intégrée
- ✅ Streaming support

---

### Frontend avec Server Actions + Prisma Types

```tsx
// app/dashboard/page.tsx (Server Component)
import { getTasks } from '@/app/actions/tasks'
import { TaskList } from '@/components/task-list'
import { getCurrentUser } from '@/lib/auth'

export default async function DashboardPage() {
  const user = await getCurrentUser()
  const tasks = await getTasks(user.id)

  return (
    <div className="container py-6">
      <h1 className="text-3xl font-bold mb-6">Dashboard</h1>
      <TaskList initialTasks={tasks} userId={user.id} />
    </div>
  )
}
```

```tsx
// components/task-list.tsx (Client Component)
'use client'

import { useState, useOptimistic } from 'react'
import { Task } from '@prisma/client'
import { deleteTask, updateTask } from '@/app/actions/tasks'
import { Button } from '@/components/ui/button'
import { Card } from '@/components/ui/card'
import { toast } from 'sonner'

type TaskWithUser = Task & {
  user: { id: string; name: string }
}

export function TaskList({ 
  initialTasks, 
  userId 
}: { 
  initialTasks: TaskWithUser[]
  userId: string
}) {
  const [tasks, setTasks] = useState(initialTasks)
  const [optimisticTasks, addOptimisticTask] = useOptimistic(
    tasks,
    (state, deletedId: string) => state.filter(t => t.id !== deletedId)
  )

  const handleDelete = async (taskId: string) => {
    // Optimistic update
    addOptimisticTask(taskId)

    // Server action
    const result = await deleteTask(taskId, userId)

    if (result.success) {
      setTasks(tasks.filter(t => t.id !== taskId))
      toast.success("Task deleted")
    } else {
      // Rollback optimistic update
      setTasks(tasks)
      toast.error(result.error)
    }
  }

  const handleStatusChange = async (taskId: string, newStatus: Task['status']) => {
    const result = await updateTask(taskId, userId, { status: newStatus })

    if (result.success) {
      setTasks(tasks.map(t => 
        t.id === taskId ? { ...t, status: newStatus } : t
      ))
      toast.success("Status updated")
    } else {
      toast.error(result.error)
    }
  }

  return (
    <div className="space-y-4">
      {optimisticTasks.map(task => (
        <Card key={task.id} className="p-4">
          <div className="flex items-center justify-between">
            <div>
              <h3 className="font-semibold">{task.title}</h3>
              <p className="text-sm text-muted-foreground">{task.description}</p>
              <p className="text-xs text-muted-foreground mt-1">
                Status: {task.status} | Priority: {task.priority}
              </p>
            </div>
            <div className="flex gap-2">
              <Button
                size="sm"
                variant="outline"
                onClick={() => handleStatusChange(
                  task.id,
                  task.status === 'PENDING' ? 'IN_PROGRESS' : 'COMPLETED'
                )}
              >
                Next Status
              </Button>
              <Button
                size="sm"
                variant="destructive"
                onClick={() => handleDelete(task.id)}
              >
                Delete
              </Button>
            </div>
          </div>
        </Card>
      ))}
    </div>
  )
}
```

---

### Prisma Relations & Include Patterns

```typescript
// Get task with user
const task = await prisma.task.findUnique({
  where: { id },
  include: {
    user: true,  // Include relation complète
  }
})

// Select specific fields only (optimisation)
const task = await prisma.task.findUnique({
  where: { id },
  include: {
    user: {
      select: { id: true, name: true, email: true }
    }
  }
})

// Nested includes (deep relations)
const user = await prisma.user.findUnique({
  where: { id },
  include: {
    tasks: {
      where: { status: 'PENDING' },
      orderBy: { createdAt: 'desc' },
      take: 10,
    }
  }
})

// Count relations
const user = await prisma.user.findUnique({
  where: { id },
  include: {
    _count: {
      select: { tasks: true }
    }
  }
})
```

---

### Prisma Transactions (ACID guarantees)

```typescript
// Sequential operations transaction
const result = await prisma.$transaction(async (tx) => {
  // 1. Create task
  const task = await tx.task.create({
    data: { title: "New task", userId }
  })

  // 2. Update user stats
  await tx.user.update({
    where: { id: userId },
    data: { 
      tasksCount: { increment: 1 }
    }
  })

  // 3. Create notification
  await tx.notification.create({
    data: {
      userId,
      message: `Task "${task.title}" created`
    }
  })

  return task
})

// Rollback automatique si erreur
```

**Interactive transactions (complexes):**
```typescript
const result = await prisma.$transaction(
  async (tx) => {
    // Multiple queries avec logique conditionnelle
    const user = await tx.user.findUnique({ where: { id: userId } })
    
    if (!user) throw new Error("User not found")
    
    if (user.tasksCount >= 100) {
      throw new Error("Task limit reached")
    }

    return await tx.task.create({ data: taskData })
  },
  {
    maxWait: 5000,  // Max wait acquire lock
    timeout: 10000, // Max transaction duration
  }
)
```

---

### Prisma Middleware (Logging, Soft Delete)

```typescript
// lib/prisma.ts
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

// Logging middleware
prisma.$use(async (params, next) => {
  const before = Date.now()
  const result = await next(params)
  const after = Date.now()

  console.log(`Query ${params.model}.${params.action} took ${after - before}ms`)

  return result
})

// Soft delete middleware
prisma.$use(async (params, next) => {
  if (params.model === 'Task') {
    if (params.action === 'delete') {
      // Change to update with deletedAt
      params.action = 'update'
      params.args['data'] = { deletedAt: new Date() }
    }

    if (params.action === 'findMany' || params.action === 'findFirst') {
      // Filter out soft deleted
      params.args.where = { ...params.args.where, deletedAt: null }
    }
  }

  return next(params)
})

export { prisma }
```

---

## Pattern #8: Prisma + Zod Full Validation

### Generate Zod from Prisma (automatique)

```bash
# Install zod-prisma
npm install zod-prisma-types

# Update prisma schema
# prisma/schema.prisma
generator zod {
  provider = "zod-prisma-types"
  output   = "../src/lib/zod"
}

# Generate
npx prisma generate
```

**Résultat auto-généré:**
```typescript
// src/lib/zod/index.ts (auto-generated)
import { z } from 'zod'

export const TaskSchema = z.object({
  id: z.string().cuid(),
  title: z.string().min(1).max(200),
  description: z.string().nullable(),
  status: z.enum(['PENDING', 'IN_PROGRESS', 'COMPLETED']),
  priority: z.enum(['LOW', 'MEDIUM', 'HIGH']),
  createdAt: z.date(),
  updatedAt: z.date(),
  userId: z.string().cuid(),
})

export const TaskCreateSchema = TaskSchema.omit({
  id: true,
  createdAt: true,
  updatedAt: true,
})

export const TaskUpdateSchema = TaskCreateSchema.partial()
```

**Usage dans Server Actions:**
```typescript
import { TaskCreateSchema } from '@/lib/zod'

export async function createTask(input: unknown) {
  const validated = TaskCreateSchema.parse(input)  // Auto-validated
  return prisma.task.create({ data: validated })
}
```

---

## Checklist Prisma + Next.js Integration

### Setup
- [ ] Prisma schema défini (`schema.prisma`)
- [ ] Enums définis pour status/priority/etc
- [ ] Relations définies (`@relation`)
- [ ] Indexes créés (`@@index`)
- [ ] Prisma client généré (`npx prisma generate`)
- [ ] Migrations appliquées (`npx prisma migrate dev`)

### Backend (Server Actions)
- [ ] Prisma client singleton (`lib/prisma.ts`)
- [ ] Zod validation schemas (auto-generated ou manuels)
- [ ] Server actions avec error handling
- [ ] Ownership checks (userId validation)
- [ ] Cache revalidation (`revalidatePath`)
- [ ] Transactions si opérations multiples

### Frontend (React)
- [ ] Prisma types importés (`@prisma/client`)
- [ ] Server Components fetch data (Prisma direct)
- [ ] Client Components actions (Server Actions)
- [ ] Optimistic updates (`useOptimistic`)
- [ ] Loading states (Suspense boundaries)
- [ ] Error boundaries

### Performance
- [ ] Select only needed fields (pas `include: { user: true }` si besoin juste id)
- [ ] Indexes sur colonnes filtrées/triées
- [ ] Connection pooling configuré (Vercel/Railway)
- [ ] Pagination si listes longues

---

## Quand utiliser quel Path?

| Path | Stack | Cas d'usage |
|------|-------|-------------|
| **Path A (FastAPI)** | Python + FastAPI + Pydantic + React + shadcn | Backend Python séparé, API REST, microservices, machine learning intégré |
| **Path B (Prisma)** | Next.js + Prisma + React + shadcn | Full-stack Next.js, Server Actions, déploiement Vercel simplifié |

**Recommandation:**
- **Path A (FastAPI)** si besoin backend Python pur ou API séparée
- **Path B (Prisma)** si stack Next.js full-stack avec Server Actions

---

## Workflow Executor avec Integration Skill + Prisma

**User:** "Crée CRUD tasks avec Prisma"

**Executor:**
```
1. Load skills: frontend + backend + integration
2. Détecte: Prisma mentionné → Prisma pattern
3. Backend (Server Actions):
   - Prisma schema (Task model + enums)
   - npx prisma migrate dev
   - Server actions (create, get, update, delete)
   - Zod validation (nativeEnum TaskStatus)
4. Frontend:
   - Server Component (getTasks Prisma direct)
   - Client Component (Server Actions + useOptimistic)
   - Shadcn form
5. Checklist Prisma ✓
```

---

**Patterns couverts:** ✅
- **Path A (FastAPI):** Pydantic → Zod sync, REST API, React Query
- **Path B (Prisma):** Prisma native types, Server Actions, useOptimistic
- Prisma best practices (schema, singleton, relations, transactions, middleware)
- Error handling standardisé
- Loading states (Skeleton UI)
- Optimistic updates

**Version**: 1.2.0
**Updated**: 2025-01-10 - Nettoyé références tRPC/SQLAlchemy, focus FastAPI + Prisma
