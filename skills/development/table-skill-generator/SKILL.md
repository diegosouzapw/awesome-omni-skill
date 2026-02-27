---
name: table-skill-generator
description: "Generate a project-specific data table patterns skill. Use for frontend apps to document table/list handling. Triggers on: generate table skill, create table patterns, table-skill-generator."
type: meta
generates: table-patterns
trigger:
  condition: has frontend app
---

# Table Skill Generator

Generate a project-specific `table-patterns` skill that documents exactly how data tables are built in THIS project.

---

## The Job

1. Read project context (`docs/project.json`)
2. Analyze existing table implementations
3. Ask clarifying questions about table patterns
4. Generate `docs/skills/table-patterns/SKILL.md`
5. Update `project.json` to record the generated skill

---

## Step 1: Read Project Context

```bash
cat docs/project.json
```

Look for:
- `apps[].type` — includes "frontend" or "fullstack"
- `stack.framework` — React, Vue, etc.

---

## Step 2: Analyze Existing Table Implementations

```bash
# Find table components
find . -type f \( -name "*Table*" -o -name "*DataGrid*" -o -name "*List*" \) -name "*.tsx" | grep -v node_modules | head -20

# Find table libraries
grep -E "@tanstack/react-table|ag-grid|react-table" package.json 2>/dev/null

# Find column definitions
grep -r "columnDef\|columns.*=\|ColumnDef" --include="*.tsx" | head -10

# Look at existing table
cat $(find . -type f -name "*Table*.tsx" | grep -v node_modules | head -1) 2>/dev/null | head -80
```

---

## Step 3: Clarifying Questions

```
I found the following table patterns:

Table Library: [detected]
UI Components: [detected]
Data Fetching: [detected]

Please confirm or correct:

1. What table library do you use?
   A. TanStack Table (react-table)
   B. AG Grid
   C. Custom table components
   D. shadcn/ui DataTable
   E. Other: [specify]

2. How is data fetched?
   A. Server Components (RSC)
   B. Client-side fetch (SWR, React Query)
   C. Server Actions
   D. API routes

3. What table features do you need?
   A. Basic display only
   B. Sorting
   C. Filtering
   D. Pagination
   E. All of the above
```

---

## Step 4: Generate the Skill

Create `docs/skills/table-patterns/SKILL.md`:

```markdown
---
name: table-patterns
description: "Build data tables with sorting, filtering, and pagination in [PROJECT_NAME]"
project-specific: true
generated-by: table-skill-generator
generated-at: [DATE]
---

# Table Patterns Skill

How to build data tables in this project.

---

## Quick Reference

| Task | Pattern |
|------|---------|
| Create table | Use DataTable + columns |
| Add sorting | Column header with sort button |
| Add filtering | Search input + URL params |
| Add pagination | Pagination component |

---

## Architecture

- **Table Library:** [TABLE_LIBRARY] (e.g., TanStack Table)
- **UI Components:** [UI_LIB] (e.g., shadcn/ui)
- **Data Fetching:** [FETCH_PATTERN] (e.g., Server Components)

---

## Basic Table Template

### Column Definitions

\`\`\`typescript
// components/entities/columns.tsx
'use client'

import { ColumnDef } from '@tanstack/react-table'
import { Entity } from '@/types'
import { Badge } from '@/components/ui/badge'
import { Button } from '@/components/ui/button'
import { MoreHorizontal, ArrowUpDown } from 'lucide-react'
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu'

export const columns: ColumnDef<Entity>[] = [
  {
    accessorKey: 'name',
    header: ({ column }) => (
      <Button
        variant="ghost"
        onClick={() => column.toggleSorting(column.getIsSorted() === 'asc')}
      >
        Name
        <ArrowUpDown className="ml-2 h-4 w-4" />
      </Button>
    ),
  },
  {
    accessorKey: 'status',
    header: 'Status',
    cell: ({ row }) => {
      const status = row.getValue('status') as string
      return (
        <Badge variant={status === 'active' ? 'default' : 'secondary'}>
          {status}
        </Badge>
      )
    },
  },
  {
    accessorKey: 'createdAt',
    header: 'Created',
    cell: ({ row }) => {
      return new Date(row.getValue('createdAt')).toLocaleDateString()
    },
  },
  {
    id: 'actions',
    cell: ({ row }) => {
      const entity = row.original
      
      return (
        <DropdownMenu>
          <DropdownMenuTrigger asChild>
            <Button variant="ghost" className="h-8 w-8 p-0">
              <MoreHorizontal className="h-4 w-4" />
            </Button>
          </DropdownMenuTrigger>
          <DropdownMenuContent align="end">
            <DropdownMenuItem onClick={() => navigator.clipboard.writeText(entity.id)}>
              Copy ID
            </DropdownMenuItem>
            <DropdownMenuItem>Edit</DropdownMenuItem>
            <DropdownMenuItem className="text-red-600">Delete</DropdownMenuItem>
          </DropdownMenuContent>
        </DropdownMenu>
      )
    },
  },
]
\`\`\`

### Data Table Component

\`\`\`typescript
// components/ui/data-table.tsx
'use client'

import {
  ColumnDef,
  flexRender,
  getCoreRowModel,
  getSortedRowModel,
  getPaginationRowModel,
  getFilteredRowModel,
  useReactTable,
  SortingState,
  ColumnFiltersState,
} from '@tanstack/react-table'
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from '@/components/ui/table'
import { Input } from '@/components/ui/input'
import { Button } from '@/components/ui/button'
import { useState } from 'react'

interface DataTableProps<TData, TValue> {
  columns: ColumnDef<TData, TValue>[]
  data: TData[]
  searchKey?: string
}

export function DataTable<TData, TValue>({
  columns,
  data,
  searchKey,
}: DataTableProps<TData, TValue>) {
  const [sorting, setSorting] = useState<SortingState>([])
  const [columnFilters, setColumnFilters] = useState<ColumnFiltersState>([])

  const table = useReactTable({
    data,
    columns,
    getCoreRowModel: getCoreRowModel(),
    getSortedRowModel: getSortedRowModel(),
    getPaginationRowModel: getPaginationRowModel(),
    getFilteredRowModel: getFilteredRowModel(),
    onSortingChange: setSorting,
    onColumnFiltersChange: setColumnFilters,
    state: {
      sorting,
      columnFilters,
    },
  })

  return (
    <div>
      {searchKey && (
        <div className="flex items-center py-4">
          <Input
            placeholder="Search..."
            value={(table.getColumn(searchKey)?.getFilterValue() as string) ?? ''}
            onChange={(event) =>
              table.getColumn(searchKey)?.setFilterValue(event.target.value)
            }
            className="max-w-sm"
          />
        </div>
      )}
      
      <div className="rounded-md border">
        <Table>
          <TableHeader>
            {table.getHeaderGroups().map((headerGroup) => (
              <TableRow key={headerGroup.id}>
                {headerGroup.headers.map((header) => (
                  <TableHead key={header.id}>
                    {header.isPlaceholder
                      ? null
                      : flexRender(
                          header.column.columnDef.header,
                          header.getContext()
                        )}
                  </TableHead>
                ))}
              </TableRow>
            ))}
          </TableHeader>
          <TableBody>
            {table.getRowModel().rows?.length ? (
              table.getRowModel().rows.map((row) => (
                <TableRow key={row.id}>
                  {row.getVisibleCells().map((cell) => (
                    <TableCell key={cell.id}>
                      {flexRender(cell.column.columnDef.cell, cell.getContext())}
                    </TableCell>
                  ))}
                </TableRow>
              ))
            ) : (
              <TableRow>
                <TableCell colSpan={columns.length} className="h-24 text-center">
                  No results.
                </TableCell>
              </TableRow>
            )}
          </TableBody>
        </Table>
      </div>
      
      <div className="flex items-center justify-end space-x-2 py-4">
        <Button
          variant="outline"
          size="sm"
          onClick={() => table.previousPage()}
          disabled={!table.getCanPreviousPage()}
        >
          Previous
        </Button>
        <Button
          variant="outline"
          size="sm"
          onClick={() => table.nextPage()}
          disabled={!table.getCanNextPage()}
        >
          Next
        </Button>
      </div>
    </div>
  )
}
\`\`\`

### Page Component

\`\`\`typescript
// app/entities/page.tsx
import { createClient } from '@/lib/supabase/server'
import { columns } from './columns'
import { DataTable } from '@/components/ui/data-table'

export default async function EntitiesPage() {
  const supabase = await createClient()
  
  const { data: entities } = await supabase
    .from('entities')
    .select('*')
    .order('created_at', { ascending: false })
  
  return (
    <div className="container py-10">
      <h1 className="text-2xl font-bold mb-6">Entities</h1>
      <DataTable columns={columns} data={entities ?? []} searchKey="name" />
    </div>
  )
}
\`\`\`

---

## Server-Side Pagination

For large datasets, paginate on the server:

### Page Component

\`\`\`typescript
// app/entities/page.tsx
import { createClient } from '@/lib/supabase/server'

interface Props {
  searchParams: { page?: string; limit?: string; search?: string }
}

export default async function EntitiesPage({ searchParams }: Props) {
  const page = parseInt(searchParams.page ?? '1')
  const limit = parseInt(searchParams.limit ?? '20')
  const search = searchParams.search ?? ''
  
  const supabase = await createClient()
  
  let query = supabase
    .from('entities')
    .select('*', { count: 'exact' })
  
  if (search) {
    query = query.ilike('name', \`%\${search}%\`)
  }
  
  const { data: entities, count } = await query
    .range((page - 1) * limit, page * limit - 1)
    .order('created_at', { ascending: false })
  
  return (
    <div className="container py-10">
      <DataTable
        columns={columns}
        data={entities ?? []}
        pageCount={Math.ceil((count ?? 0) / limit)}
        currentPage={page}
      />
    </div>
  )
}
\`\`\`

### URL-Based Pagination

\`\`\`typescript
// components/ui/data-table-pagination.tsx
'use client'

import { useRouter, useSearchParams } from 'next/navigation'
import { Button } from '@/components/ui/button'

interface PaginationProps {
  pageCount: number
  currentPage: number
}

export function DataTablePagination({ pageCount, currentPage }: PaginationProps) {
  const router = useRouter()
  const searchParams = useSearchParams()
  
  const setPage = (page: number) => {
    const params = new URLSearchParams(searchParams)
    params.set('page', page.toString())
    router.push(\`?\${params.toString()}\`)
  }
  
  return (
    <div className="flex items-center justify-between py-4">
      <span className="text-sm text-muted-foreground">
        Page {currentPage} of {pageCount}
      </span>
      <div className="flex gap-2">
        <Button
          variant="outline"
          size="sm"
          onClick={() => setPage(currentPage - 1)}
          disabled={currentPage <= 1}
        >
          Previous
        </Button>
        <Button
          variant="outline"
          size="sm"
          onClick={() => setPage(currentPage + 1)}
          disabled={currentPage >= pageCount}
        >
          Next
        </Button>
      </div>
    </div>
  )
}
\`\`\`

---

## Column Types

### Text Column

\`\`\`typescript
{
  accessorKey: 'name',
  header: 'Name',
}
\`\`\`

### Sortable Column

\`\`\`typescript
{
  accessorKey: 'name',
  header: ({ column }) => (
    <Button
      variant="ghost"
      onClick={() => column.toggleSorting(column.getIsSorted() === 'asc')}
    >
      Name
      <ArrowUpDown className="ml-2 h-4 w-4" />
    </Button>
  ),
}
\`\`\`

### Date Column

\`\`\`typescript
{
  accessorKey: 'createdAt',
  header: 'Created',
  cell: ({ row }) => format(new Date(row.getValue('createdAt')), 'PPP'),
}
\`\`\`

### Badge/Status Column

\`\`\`typescript
{
  accessorKey: 'status',
  header: 'Status',
  cell: ({ row }) => (
    <Badge variant={row.getValue('status') === 'active' ? 'default' : 'secondary'}>
      {row.getValue('status')}
    </Badge>
  ),
}
\`\`\`

### Actions Column

\`\`\`typescript
{
  id: 'actions',
  cell: ({ row }) => <EntityActions entity={row.original} />,
}
\`\`\`

---

## Checklist

When creating a data table:

- [ ] Define columns with proper accessors
- [ ] Add sorting for relevant columns
- [ ] Add search/filter if needed
- [ ] Add pagination (client or server)
- [ ] Add actions dropdown
- [ ] Handle empty state
- [ ] Handle loading state
- [ ] Test with various data sizes
```

---

## Step 5: Update project.json

Add to `skills.generated[]`:

```json
{
  "name": "table-patterns",
  "generatedFrom": "table-skill-generator",
  "generatedAt": "2026-02-20"
}
```
