---
name: agentic-jumpstart-frontend
description: Frontend UI patterns with shadcn/ui, Radix UI, Tailwind CSS v4, and Lucide icons. Use when building UI components, styling, layouts, buttons, cards, dialogs, forms, responsive design, or when the user mentions UI, styling, Tailwind, components, or design.
---

# Frontend UI Patterns

## shadcn/ui Components

### Card Pattern

All cards should use the shadcn Card component:

```typescript
import {
  Card,
  CardContent,
  CardDescription,
  CardFooter,
  CardHeader,
  CardTitle,
} from "~/components/ui/card";

function CourseCard({ course }: { course: Course }) {
  return (
    <Card>
      <CardHeader>
        <CardTitle>{course.title}</CardTitle>
        <CardDescription>{course.description}</CardDescription>
      </CardHeader>
      <CardContent>
        <p>{course.summary}</p>
      </CardContent>
      <CardFooter className="flex justify-between">
        <Button variant="outline">Preview</Button>
        <Button>Enroll</Button>
      </CardFooter>
    </Card>
  );
}
```

### Button Variants

```typescript
import { Button } from "~/components/ui/button";

// Primary action
<Button>Save Changes</Button>

// Secondary action
<Button variant="secondary">Cancel</Button>

// Outline
<Button variant="outline">Edit</Button>

// Destructive
<Button variant="destructive">Delete</Button>

// Ghost (subtle)
<Button variant="ghost">More</Button>

// Link style
<Button variant="link">Learn more</Button>

// Sizes
<Button size="sm">Small</Button>
<Button size="default">Default</Button>
<Button size="lg">Large</Button>
<Button size="icon"><Icon /></Button>

// Loading state
<Button disabled>
  <Loader2 className="mr-2 h-4 w-4 animate-spin" />
  Saving...
</Button>
```

### Form Components

```typescript
import { Input } from "~/components/ui/input";
import { Label } from "~/components/ui/label";
import { Textarea } from "~/components/ui/textarea";
import { Checkbox } from "~/components/ui/checkbox";
import { Switch } from "~/components/ui/switch";
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from "~/components/ui/select";

// Input with label
<div className="space-y-2">
  <Label htmlFor="email">Email</Label>
  <Input id="email" type="email" placeholder="Enter email" />
</div>

// Select
<Select value={value} onValueChange={setValue}>
  <SelectTrigger>
    <SelectValue placeholder="Select option" />
  </SelectTrigger>
  <SelectContent>
    <SelectItem value="option1">Option 1</SelectItem>
    <SelectItem value="option2">Option 2</SelectItem>
  </SelectContent>
</Select>

// Checkbox
<div className="flex items-center space-x-2">
  <Checkbox id="terms" checked={accepted} onCheckedChange={setAccepted} />
  <Label htmlFor="terms">Accept terms</Label>
</div>

// Switch
<div className="flex items-center space-x-2">
  <Switch id="notifications" checked={enabled} onCheckedChange={setEnabled} />
  <Label htmlFor="notifications">Enable notifications</Label>
</div>
```

### Dialog Pattern

```typescript
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogFooter,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from "~/components/ui/dialog";

function EditDialog({ item, onSave }: { item: Item; onSave: (data: Item) => void }) {
  const [open, setOpen] = useState(false);

  return (
    <Dialog open={open} onOpenChange={setOpen}>
      <DialogTrigger asChild>
        <Button variant="outline">Edit</Button>
      </DialogTrigger>
      <DialogContent className="sm:max-w-[425px]">
        <DialogHeader>
          <DialogTitle>Edit Item</DialogTitle>
          <DialogDescription>
            Make changes to the item. Click save when done.
          </DialogDescription>
        </DialogHeader>
        <form onSubmit={handleSubmit}>
          {/* Form fields */}
          <DialogFooter>
            <Button type="button" variant="outline" onClick={() => setOpen(false)}>
              Cancel
            </Button>
            <Button type="submit">Save</Button>
          </DialogFooter>
        </form>
      </DialogContent>
    </Dialog>
  );
}
```

### Dropdown Menu

```typescript
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuLabel,
  DropdownMenuSeparator,
  DropdownMenuTrigger,
} from "~/components/ui/dropdown-menu";
import { MoreHorizontal, Edit, Trash, Copy } from "lucide-react";

function ActionsMenu({ onEdit, onDelete, onDuplicate }) {
  return (
    <DropdownMenu>
      <DropdownMenuTrigger asChild>
        <Button variant="ghost" size="icon">
          <MoreHorizontal className="h-4 w-4" />
        </Button>
      </DropdownMenuTrigger>
      <DropdownMenuContent align="end">
        <DropdownMenuLabel>Actions</DropdownMenuLabel>
        <DropdownMenuSeparator />
        <DropdownMenuItem onClick={onEdit}>
          <Edit className="mr-2 h-4 w-4" />
          Edit
        </DropdownMenuItem>
        <DropdownMenuItem onClick={onDuplicate}>
          <Copy className="mr-2 h-4 w-4" />
          Duplicate
        </DropdownMenuItem>
        <DropdownMenuSeparator />
        <DropdownMenuItem onClick={onDelete} className="text-red-600">
          <Trash className="mr-2 h-4 w-4" />
          Delete
        </DropdownMenuItem>
      </DropdownMenuContent>
    </DropdownMenu>
  );
}
```

### Tabs

```typescript
import { Tabs, TabsContent, TabsList, TabsTrigger } from "~/components/ui/tabs";

function SettingsTabs() {
  return (
    <Tabs defaultValue="general">
      <TabsList>
        <TabsTrigger value="general">General</TabsTrigger>
        <TabsTrigger value="security">Security</TabsTrigger>
        <TabsTrigger value="notifications">Notifications</TabsTrigger>
      </TabsList>
      <TabsContent value="general">
        <GeneralSettings />
      </TabsContent>
      <TabsContent value="security">
        <SecuritySettings />
      </TabsContent>
      <TabsContent value="notifications">
        <NotificationSettings />
      </TabsContent>
    </Tabs>
  );
}
```

## Page Layout Pattern

```typescript
import { Page, PageHeader } from "~/components/page";

function CoursesPage() {
  return (
    <Page>
      <PageHeader
        title="Courses"
        description="Manage your courses and content"
        action={<Button>Add Course</Button>}
      />
      <div className="grid gap-6 md:grid-cols-2 lg:grid-cols-3">
        {courses.map((course) => (
          <CourseCard key={course.id} course={course} />
        ))}
      </div>
    </Page>
  );
}
```

## Tailwind CSS Patterns

### Spacing & Layout

```typescript
// Container with max width
<div className="container mx-auto px-4">

// Grid layouts
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">

// Flex layouts
<div className="flex items-center justify-between gap-4">

// Stack with gap
<div className="flex flex-col gap-4">
// or
<div className="space-y-4">
```

### Responsive Design

```typescript
// Mobile-first responsive
<div className="text-sm md:text-base lg:text-lg">

// Hide/show based on breakpoint
<div className="hidden md:block">  // Hidden on mobile
<div className="block md:hidden">  // Only on mobile

// Responsive grid
<div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4">
```

### Common Utility Combinations

```typescript
// Card shadow and rounded
className="rounded-lg border bg-card p-6 shadow-sm"

// Interactive element
className="cursor-pointer hover:bg-accent transition-colors"

// Truncate text
className="truncate"

// Line clamp
className="line-clamp-2"

// Focus ring
className="focus:outline-none focus:ring-2 focus:ring-ring focus:ring-offset-2"
```

## Lucide Icons

```typescript
import {
  ChevronRight,
  ChevronDown,
  Plus,
  Trash,
  Edit,
  Search,
  Settings,
  User,
  LogOut,
  Menu,
  X,
  Check,
  AlertCircle,
  Info,
  Loader2,
} from "lucide-react";

// Standard icon size in buttons
<Button>
  <Plus className="mr-2 h-4 w-4" />
  Add Item
</Button>

// Icon button
<Button size="icon" variant="ghost">
  <Settings className="h-4 w-4" />
</Button>

// Loading spinner
<Loader2 className="h-4 w-4 animate-spin" />

// Icon colors
<AlertCircle className="h-4 w-4 text-red-500" />
<Check className="h-4 w-4 text-green-500" />
<Info className="h-4 w-4 text-blue-500" />
```

## Loading Skeleton

```typescript
import { Skeleton } from "~/components/ui/skeleton";

function CardSkeleton() {
  return (
    <Card>
      <CardHeader>
        <Skeleton className="h-6 w-3/4" />
        <Skeleton className="h-4 w-1/2" />
      </CardHeader>
      <CardContent>
        <Skeleton className="h-20 w-full" />
      </CardContent>
      <CardFooter>
        <Skeleton className="h-10 w-24" />
      </CardFooter>
    </Card>
  );
}
```

## cn Utility for Class Merging

```typescript
import { cn } from "~/lib/utils";

function StatusBadge({ status }: { status: "active" | "pending" | "inactive" }) {
  return (
    <span
      className={cn(
        "inline-flex items-center rounded-full px-2 py-1 text-xs font-medium",
        {
          "bg-green-100 text-green-700": status === "active",
          "bg-yellow-100 text-yellow-700": status === "pending",
          "bg-gray-100 text-gray-700": status === "inactive",
        }
      )}
    >
      {status}
    </span>
  );
}
```

## Frontend Checklist

- [ ] Use shadcn/ui Card for card layouts
- [ ] Use shadcn/ui Dialog for modals
- [ ] Use shadcn/ui Button with appropriate variants
- [ ] Use Page and PageHeader components
- [ ] Use Lucide icons with consistent sizing (h-4 w-4)
- [ ] Use cn() for conditional class merging
- [ ] Mobile-first responsive design
- [ ] Loading states with Skeleton components
- [ ] Consistent spacing with gap utilities
- [ ] Accessible form labels and ARIA attributes
