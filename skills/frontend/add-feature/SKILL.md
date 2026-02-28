---
name: add-feature
description: Scaffold complete feature with types, repository, API routes, components, store actions, and tests. Use when adding major new functionality like water tracking, sleep tracking, etc.
allowed-tools: Read, Write, Glob, Grep, Edit
---

# Add Feature

Scaffold a complete feature including types, database layer, API routes, UI components, state management, and tests.

## Usage

When user requests to add a new major feature, ask for:

1. **Feature name** (e.g., "Water Tracking", "Sleep Logging", "Weight Management")
2. **Feature description** (what data it tracks, why it matters)
3. **Data fields** needed (with types and validation rules)
4. **Dashboard components** needed (card, chart, statistics)
5. **Forms needed** (input forms, edit forms)
6. **Daily summary impact** (whether it affects health score, true/false)

## Implementation Process

This skill orchestrates the following steps:

### Step 1: Create TypeScript Types

Create file: `src/lib/types/{featureName}.ts`

```typescript
export interface FeatureEntity {
  id: string;
  date: string;
  field1: string;
  field2: number;
  createdAt: string;
}

// Validation schema types (for API)
export type FeatureInput = Omit<FeatureEntity, 'id' | 'createdAt'>;
```

### Step 2: Create Repository

Use `/generate-repository` skill to create:

- `src/lib/database/repositories/featureRepository.ts`
- CRUD methods
- Date-based queries

### Step 3: Create API Routes

Use `/generate-api-route` skill to create:

- `src/app/api/feature/route.ts`
- POST handler (create)
- DELETE handler (delete)
- Daily summary recalculation (if applicable)

### Step 4: Create Store Actions

Use `/generate-store-action` skill to add to `src/lib/store/healthStore.ts`:

- State property for feature data
- fetchFeature() action
- addFeature() action
- deleteFeature() action

### Step 5: Create Dashboard Card

Use `/generate-card` skill to create:

- `src/components/dashboard/{FeatureName}Card.tsx`
- Display key metrics
- Show trends or statistics
- Optional: Recharts integration

### Step 6: Create Input Form

Use `/generate-form` skill to create:

- `src/components/forms/{FeatureName}Form.tsx`
- Form fields for data entry
- Validation logic
- Store action integration

### Step 7: Create Tests

Use `/generate-test` skill to create:

- `src/__tests__/lib/database/repositories/{FeatureName}.test.ts`
- `src/__tests__/components/forms/{FeatureName}Form.test.tsx`
- `src/__tests__/app/api/feature/route.test.ts`

### Step 8: Update Dashboard Layout

Edit: `src/app/page.tsx` or `src/components/layout/MainLayout.tsx`

- Import new card component
- Add to dashboard grid
- Position appropriately

### Step 9: Update Navigation (if needed)

If feature has dedicated page:

- Create `src/app/feature/page.tsx`
- Create full page component
- Update navigation in Sidebar

### Step 10: Database Schema Update

Add to: `src/lib/database/schema.sql`

- Create new table
- Define columns with types
- Add indexes for common queries

## Full Feature Example: Water Tracking

### 1. Types (`src/lib/types/water.ts`)

```typescript
export interface WaterLog {
  id: string;
  date: string;
  amount: number; // in ml
  time: string; // HH:MM format
  source: 'water' | 'beverage' | 'food';
  createdAt: string;
}

export type WaterInput = Omit<WaterLog, 'id' | 'createdAt'>;
```

### 2. Repository → `/generate-repository`

- Entity: WaterLog
- Table: water_logs
- Methods: addWaterLog, getWaterLogsByDate, deleteWaterLog

### 3. API Routes → `/generate-api-route`

- POST /api/water → create water log
- DELETE /api/water?id=X → delete water log
- Daily summary recalculation: YES

### 4. Store Actions → `/generate-store-action`

```typescript
interface HealthState {
  waterLogs: WaterLog[];
  fetchDailyWaterLogs: (date: string) => Promise<void>;
  addWaterLog: (log: WaterInput) => Promise<void>;
  deleteWaterLog: (id: string) => Promise<void>;
}
```

### 5. Dashboard Card → `/generate-card`

- Card name: WaterIntakeCard
- Displays: Total ml today, Target vs actual, Timeline of entries
- Icon: Droplets
- Optional chart: Hourly water intake

### 6. Input Form → `/generate-form`

- Form name: WaterLogForm
- Fields: Amount (number), Time (time picker), Source (select)
- List: Shows today's water entries with ability to remove

### 7. Tests → `/generate-test`

- Repository tests: CRUD operations
- Form tests: Validation, submission
- API tests: POST, DELETE handlers

### 8. Update Dashboard

Add WaterIntakeCard to dashboard grid

## Key Considerations

### Database Schema

```sql
CREATE TABLE IF NOT EXISTS water_logs (
  id TEXT PRIMARY KEY,
  date TEXT NOT NULL,
  amount INTEGER NOT NULL,
  time TEXT NOT NULL,
  source TEXT NOT NULL,
  created_at TEXT NOT NULL,
  UNIQUE(date, time, source)
);

CREATE INDEX idx_water_logs_date ON water_logs(date);
```

### Health Score Integration

If feature impacts health score:

- Update `calculateHealthScore()` in `src/lib/utils/healthScoring.ts`
- Update daily summary calculation in `DailySummaryRepository`
- Document scoring formula

### Migrations

Document any schema changes needed:

- Create migration file or script
- Add instructions to DEVELOPMENT.md

## Checklist

Complete feature should have:

- [ ] TypeScript types defined
- [ ] Database repository with CRUD
- [ ] API routes (POST, DELETE, GET)
- [ ] Zustand store actions
- [ ] Dashboard card component
- [ ] Input/edit forms
- [ ] Comprehensive tests
- [ ] Dashboard integration
- [ ] Database schema
- [ ] Optional: dedicated page
- [ ] Documentation updated (if major feature)

## Coordination with Other Skills

This skill uses:

1. `/generate-repository` - for data access
2. `/generate-api-route` - for API endpoints
3. `/generate-store-action` - for state management
4. `/generate-card` - for dashboard widgets
5. `/generate-form` - for data entry
6. `/generate-test` - for test coverage

Each sub-skill handles a specific layer of the feature.

## Best Practices

1. **Start with types** - Define data structure first
2. **Database first** - Create repository before using in API
3. **API routes next** - Implement endpoints before UI
4. **Store actions** - Wire up state management before components
5. **UI components** - Build forms and cards
6. **Tests throughout** - Write tests for each layer
7. **Integration** - Add to dashboard and navigation
8. **Documentation** - Update relevant docs

## Output Structure

After using this skill, project structure should look like:

```
src/
├── lib/
│   ├── types/
│   │   └── waterTracking.ts
│   └── database/repositories/
│       └── waterTrackingRepository.ts
├── app/
│   └── api/
│       └── water/
│           └── route.ts
├── components/
│   ├── dashboard/
│   │   └── WaterIntakeCard.tsx
│   └── forms/
│       └── WaterLogForm.tsx
└── __tests__/
    ├── lib/
    │   └── database/repositories/
    │       └── WaterTracking.test.ts
    ├── components/
    │   └── forms/
    │       └── WaterLogForm.test.tsx
    └── app/
        └── api/water/
            └── route.test.ts
```

## Implementation Checklist

- [ ] Request feature details from user
- [ ] Create TypeScript types
- [ ] Generate repository
- [ ] Generate API routes
- [ ] Generate store actions
- [ ] Generate dashboard card
- [ ] Generate form component
- [ ] Generate tests
- [ ] Update dashboard integration
- [ ] Update navigation (if page)
- [ ] Verify TypeScript compilation
- [ ] Run tests
- [ ] Commit changes with proper messages
