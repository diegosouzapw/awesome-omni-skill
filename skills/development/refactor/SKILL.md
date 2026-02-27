---
name: refactor
description: Hunt for code smells, anti-patterns, and refactoring opportunities in the TaskBridge codebase using React/Next.js best practices. Fixes ONE issue at a time. Use when you want to improve code quality, extract utilities, or reorganize components.
---

# Code Refactoring Agent

Hunt for code smells, anti-patterns, and refactoring opportunities in the TaskBridge codebase using React/Next.js principal engineering best practices.

**Usage**: Run this skill manually from time to time to find and fix one refactoring opportunity at a time.

## Mission

Act as a senior principal engineer conducting code reviews with focus on:
- **Code organization** - Utils in components, business logic in UI
- **Separation of concerns** - API routes mixing data/business logic
- **DRY violations** - Duplicated code across files
- **Performance** - Unnecessary re-renders, inefficient patterns
- **Type safety** - Missing types, `any` usage, weak typing
- **Testing** - Untested business logic, missing edge cases

## Approach: Test-Driven Refactoring (One Issue at a Time)

1. **Scan** - Search codebase for anti-patterns
2. **Identify** - Pick ONE highest-priority code smell
3. **Test** - Write minimal test covering existing behavior (if applicable)
4. **Refactor** - Extract, reorganize, improve code
5. **Verify** - Run checks and tests to ensure nothing broke
6. **Report** - Summarize what was fixed
7. **Ask** - Check if user wants to continue with next refactoring

**Important**: Fix ONE issue per invocation, then ask before continuing.

## Code Smells to Hunt (8 Major Anti-Patterns)

### 1. Utility Functions in Components/Pages

**Anti-pattern**:
```typescript
// ❌ BAD: Utils inside component
export default function TaskCard() {
  const formatBudget = (amount: number) => {
    return new Intl.NumberFormat('en-US').format(amount)
  }

  const calculateDaysLeft = (deadline: Date) => {
    // calculation logic
  }

  return <div>{formatBudget(task.budget)}</div>
}
```

**Refactored**:
```typescript
// ✅ GOOD: Utils extracted to /lib/utils/
// /src/lib/utils/budget.ts
export function formatBudget(amount: number, locale: string = 'en-US') {
  return new Intl.NumberFormat(locale, {
    style: 'currency',
    currency: 'USD'
  }).format(amount)
}

// /src/lib/utils/date.ts
export function calculateDaysLeft(deadline: Date): number {
  const now = new Date()
  const diff = deadline.getTime() - now.getTime()
  return Math.ceil(diff / (1000 * 60 * 60 * 24))
}

// Component now clean
import { formatBudget } from '@/lib/utils/budget'
import { calculateDaysLeft } from '@/lib/utils/date'

export default function TaskCard() {
  return <div>{formatBudget(task.budget)}</div>
}
```

### 2. Business Logic in API Routes

**Anti-pattern**:
```typescript
// ❌ BAD: Business logic in API route
// /app/api/tasks/route.ts
export async function POST(request: Request) {
  const data = await request.json()

  // Direct DB queries and business rules mixed in route
  const task = await db.tasks.create({
    data: {
      ...data,
      status: 'open',
      created_at: new Date()
    }
  })

  // Validation logic in route
  if (task.budget < 10) {
    return Response.json({ error: 'Too low' }, { status: 400 })
  }

  return Response.json(task)
}
```

**Refactored**:
```typescript
// ✅ GOOD: Business logic in service layer
// /src/server/tasks/task.service.ts
export class TaskService {
  async createTask(data: CreateTaskInput): Promise<Task> {
    this.validateBudget(data.budget)

    return await this.taskRepository.create({
      ...data,
      status: 'open',
      created_at: new Date()
    })
  }

  private validateBudget(budget: number): void {
    if (budget < 10) {
      throw new ValidationError('Budget must be at least 10')
    }
  }
}

// /app/api/tasks/route.ts - Now just a thin controller
export async function POST(request: Request) {
  try {
    const data = await request.json()
    const taskService = new TaskService()
    const task = await taskService.createTask(data)
    return Response.json(task)
  } catch (error) {
    if (error instanceof ValidationError) {
      return Response.json({ error: error.message }, { status: 400 })
    }
    throw error
  }
}
```

### 3. Duplicated Code / DRY Violations

**Look for**:
- Same logic in multiple components
- Copy-pasted functions with slight variations
- Repeated patterns across files

**Refactor to**:
- Shared utility functions in `/src/lib/utils/`
- Custom hooks in `/src/hooks/` or `/src/features/[feature]/hooks/`
- Shared components in `/src/components/ui/` or `/src/components/common/`

### 4. Large Components (God Components)

**Anti-pattern**:
```typescript
// ❌ BAD: 800-line component doing everything
export default function CreateTaskPage() {
  // 50 lines of state
  // 200 lines of handlers
  // 100 lines of validation
  // 400 lines of JSX
  // 50 lines of utils
}
```

**Refactored**:
```typescript
// ✅ GOOD: Extracted sections and hooks
// /src/app/[lang]/create-task/components/task-details-section.tsx
export function TaskDetailsSection() { /* ... */ }

// /src/app/[lang]/create-task/components/budget-section.tsx
export function BudgetSection() { /* ... */ }

// /src/app/[lang]/create-task/hooks/use-task-form.ts
export function useTaskForm() {
  // Form state and handlers
}

// /src/app/[lang]/create-task/lib/validation.ts
export function validateTaskForm(data: FormData) {
  // Validation logic
}

// Main page now orchestrates
export default function CreateTaskPage() {
  const { form, handleSubmit } = useTaskForm()

  return (
    <form onSubmit={handleSubmit}>
      <TaskDetailsSection />
      <BudgetSection />
    </form>
  )
}
```

### 5. Type Safety Issues

**Hunt for**:
- `any` types
- Missing return types
- Weak type assertions (`as any`)
- Untyped function parameters
- Optional chaining overuse (hiding type issues)

**Refactor to**:
- Proper TypeScript interfaces/types
- Strict type annotations
- Type guards where needed
- Zod schemas for runtime validation

### 6. Unused Code (DELETE, Don't Rename!)

**Critical Rule**: When finding unused code, **DELETE** it completely. Do NOT rename with underscore prefix.

**Hunt for and DELETE**:
- Unused imports: `import { Foo, Bar } from 'lib'` → Remove unused `Bar`
- Unused variables: `const [value, setValue] = useState()` → Remove unused `setValue`
- Unused function parameters: `function handler(event, data)` → Remove unused `event`
- Unused constants or helper functions

**Examples**:
```typescript
// ❌ DON'T rename with underscore
import { Foo, _Bar } from './lib'
const [value, _setValue] = useState()
function onClick(_event: Event, data: Data) {}

// ✅ DO delete completely
import { Foo } from './lib'
const [value] = useState()
function onClick(data: Data) {}
```

**Exception** (very rare): Only keep with underscore if required by external API signature that you cannot change.

### 7. Performance Anti-patterns

**Look for**:
- Missing React.memo on expensive components
- Inline function definitions in props
- Missing useMemo/useCallback
- Large objects in useEffect dependencies
- Unnecessary state updates

### 8. Server/Client Component Confusion (Next.js 15)

**Anti-pattern**:
```typescript
// ❌ BAD: Client component fetching data
'use client'
export default function TaskDetail() {
  const [task, setTask] = useState(null)

  useEffect(() => {
    fetch(`/api/tasks/${id}`).then(/* ... */)
  }, [id])
}
```

**Refactored**:
```typescript
// ✅ GOOD: Server component fetches, client handles interactivity
// /app/[lang]/tasks/[id]/page.tsx (Server Component)
export default async function TaskDetailPage({ params }) {
  const task = await taskService.getTask(params.id)
  return <TaskDetailContent task={task} />
}

// /app/[lang]/tasks/[id]/components/task-detail-content.tsx
'use client'
export function TaskDetailContent({ task }: { task: Task }) {
  // Only interactive features here
  const handleApply = () => { /* ... */ }
  return <div>...</div>
}
```

## Refactoring Process

### Phase 1: Reconnaissance

**Instructions**:
1. Use `Glob` to find files in target directory
2. Use `Grep` to search for code smell patterns:
   - Functions inside components: `const \w+ = \(.*\) =>` inside component files
   - Direct DB queries in routes: `await.*\.(insert|update|delete|select)` in `/app/api/`
   - `any` types: `grep -n ": any"` or `as any`
   - Large files: Check line counts, target 300+ line files
3. Read suspicious files and analyze
4. Create prioritized list of refactoring opportunities

### Phase 2: Test Coverage (if applicable)

**When to test**:
- Utility functions with logic (YES)
- Business logic in services (YES)
- UI components (optional, focus on logic)
- Simple data transformations (optional)

**Test approach**:
```typescript
// 1. Write minimal test covering current behavior
describe('formatBudget', () => {
  it('formats budget correctly', () => {
    expect(formatBudget(1000)).toBe('$1,000.00')
  })
})

// 2. Refactor the code
// 3. Run test to verify: npm run test
// 4. Add edge cases if needed
```

### Phase 3: Extract and Refactor

**Priority order**:
1. **High Impact**: Duplicated business logic, security issues
2. **Medium Impact**: Utils in components, large components
3. **Low Impact**: Minor optimizations, naming improvements

**Refactoring steps**:
1. Create new file in appropriate location:
   - Utils → `/src/lib/utils/[category].ts`
   - Hooks → `/src/hooks/use-[name].ts` or `/src/features/[feature]/hooks/`
   - Services → `/src/server/[domain]/[domain].service.ts`
   - Components → `/src/components/[category]/[name].tsx`
2. Move code to new location
3. Add proper TypeScript types
4. Update imports in original files
5. Test that everything works

### Phase 4: Verification

**Checklist**:
- [ ] Run `npm run check` (TypeScript + ESLint)
- [ ] Run tests if applicable: `npm run test` (when test suite exists)
- [ ] Verify in dev server: `npm run dev`
- [ ] Check that all imports resolve correctly
- [ ] Ensure no runtime errors
- [ ] Verify UI still works as expected

## Project-Specific Context

**TaskBridge Architecture**:
```
/src/
├── app/[lang]/              # Next.js pages (Server Components preferred)
├── features/[feature]/      # Self-contained business domains
│   ├── components/          # Feature-specific UI
│   ├── lib/                 # Feature data, types, utils
│   └── hooks/               # Feature-specific hooks
├── components/
│   ├── ui/                  # Design system (shadcn/ui, NextUI)
│   └── common/              # Shared layouts (Header, Footer)
├── lib/
│   ├── utils/               # 🎯 Extract utils HERE
│   ├── supabase/            # Database clients
│   └── intl/                # Translations
├── hooks/                   # 🎯 Extract global hooks HERE
├── server/                  # 🎯 Extract services HERE
│   └── [domain]/
│       ├── [domain].service.ts
│       ├── [domain].types.ts
│       └── [domain].repository.ts (if needed)
└── types/                   # Global TypeScript types
```

**Tech Stack Specifics**:
- **Next.js 15**: Use Server Components by default, Client Components only when needed
- **Supabase**: Data access through `createClient()`, prefer services over direct queries
- **TypeScript**: Strict mode, no `any` types
- **React Query**: For client-side data fetching (if applicable)
- **Zod**: For validation schemas
- **i18n**: All user-facing strings must be internationalized

## Common Targets for Refactoring

**High Priority** (check these first):
1. `/src/app/[lang]/create-task/page.tsx` - Large form component
2. `/src/app/[lang]/browse-tasks/page.tsx` - Search and filters
3. `/src/app/[lang]/tasks/[id]/` - Task detail pages
4. `/src/features/professionals/` - Professional listings
5. `/src/app/api/` routes - Business logic in API routes

**Patterns to Extract**:
- Budget formatting logic → `/src/lib/utils/budget.ts`
- Date calculations → `/src/lib/utils/date.ts`
- Task status helpers → `/src/lib/utils/task-status.ts`
- Form validation → `/src/lib/utils/validation.ts`
- Authentication checks → `/src/lib/utils/auth.ts`

**Unused Code to DELETE** (not rename):
- Unused imports, variables, function parameters
- Dead code paths
- Commented-out code blocks

## Execution Guidelines

**DO**:
- ✅ Fix ONE code smell per invocation
- ✅ Write test for existing behavior first (if applicable)
- ✅ Make small, incremental changes
- ✅ Run checks after each refactor
- ✅ Preserve existing functionality exactly
- ✅ Use TypeScript strictly (no `any`)
- ✅ Follow project architecture patterns
- ✅ Ask user before continuing to next issue
- ✅ Document complex logic with comments

**DON'T**:
- ❌ Fix multiple unrelated issues without asking
- ❌ Change functionality while refactoring
- ❌ Skip type annotations
- ❌ Leave unused imports/variables/parameters (DELETE them, don't rename with underscore)
- ❌ Rename unused code with underscore prefix (DELETE instead)
- ❌ Break existing tests
- ❌ Introduce new dependencies without discussion
- ❌ Refactor working code just for style preferences
- ❌ Continue refactoring without user confirmation

## Output Format

After fixing ONE refactoring issue, provide:

1. **🔍 Found**: What code smell was discovered and where
2. **💡 Impact**: Why this refactoring matters (performance, maintainability, etc.)
3. **🧪 Test**: If written, show the test code
4. **🔧 Refactored**: What was extracted/reorganized
5. **📁 Changes**: List of files created/modified
6. **✅ Verification**: Confirm `npm run check` passes
7. **📊 Results**: Metrics (lines saved, files improved, etc.)
8. **❓ Continue?**: Ask if user wants to find and fix the next issue

**Then STOP and wait for user response before continuing.**

## Example Session

```
User: "/refactor"

Agent:
🔍 Scanning codebase for anti-patterns...

Found 3 refactoring opportunities:
1. HIGH: formatBudget() duplicated in 4 files
2. MEDIUM: Large CreateTaskPage component (523 lines)
3. LOW: Missing types in task-card.tsx

Selecting highest priority: formatBudget() duplication

---

🔍 FOUND
Utility function formatBudget() duplicated in:
- /src/components/ui/task-card.tsx (line 45)
- /src/components/ui/posted-task-card.tsx (line 38)
- /src/app/[lang]/tasks/[id]/components/task-detail-content.tsx (line 92)
- /src/app/[lang]/create-task/components/budget-section.tsx (line 156)

💡 IMPACT
- Reduces code duplication (38 lines across 4 files)
- Creates single source of truth for budget formatting
- Improves maintainability and testability

🧪 TEST
Created /src/lib/utils/budget.test.ts:
✓ Test covering existing behavior - PASSING

🔧 REFACTORED
- Created /src/lib/utils/budget.ts with formatBudget(amount, locale, budgetType)
- Added proper TypeScript types
- Updated 4 files to import from utils
- Removed 38 lines of duplicated code

📁 CHANGES
- Created: /src/lib/utils/budget.ts
- Modified: task-card.tsx, posted-task-card.tsx, task-detail-content.tsx, budget-section.tsx

✅ VERIFICATION
✓ npm run check - PASSED (0 errors)
✓ All imports resolve correctly
✓ Tests pass

📊 RESULTS
- 38 lines removed
- 4 files improved
- 1 new reusable utility created

---

❓ Found 2 more issues. Continue with next refactoring? [y/n]
```

## Success Criteria

- Code is more maintainable and organized
- No functionality is broken
- All TypeScript checks pass
- Tests pass (if applicable)
- Code follows project architecture
- Reduction in code duplication
- Improved type safety
- Better separation of concerns
