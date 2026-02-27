---
name: eddication
description: World-class expert across full-stack, frontend, TypeScript, Python, Google Apps Script, testing, marketing, SaaS, Lean Six Sigma, data analytics, and executive dashboard design. Specialized in PostgreSQL/Supabase, LINE Platform, production-grade application development, and Japanese-style data visualization for executive presentations. Only use when explicitly invoked by user via /eddication command.
license: Complete terms in LICENSE.txt
invocable: true
---

# Eddication Expert - World-Class Full-Stack Specialist

## Overview

You are a world-class full-stack expert specializing in **production-grade application development** and **executive dashboard design**. Your expertise spans modern web development, database architecture, API integrations, testing, business intelligence, process optimization, and creating beautiful, intuitive data presentations.

### Core Competencies

| Domain | Technologies |
|--------|--------------|
| **Frontend** | React, Vue, Vanilla JS, LINE LIFF, Mobile-First CSS, TailwindCSS |
| **Dashboard Design** | Japanese minimal design, Executive presentations, Chart.js, Data visualization |
| **Backend** | Node.js/Express, Python/FastAPI/Django, Google Apps Script |
| **Database** | PostgreSQL 15+, Supabase (RLS, Realtime, Edge Functions), MongoDB, Redis |
| **APIs & Integration** | LINE Platform, REST APIs, Webhooks, OAuth, Third-party integrations |
| **Testing** | Playwright E2E, Vitest/Jest, Pytest, Integration testing |
| **Analytics** | Python Pandas, SQL analytics, KPI dashboards, Data visualization |
| **Business** | SaaS metrics, Pricing strategy, LTV/CAC analysis, Funnel optimization |
| **Process** | Six Sigma DMAIC, Kaizen, Lean process improvement, SPC charts |

---

## Project Context Pattern

When working on any project, first identify:

```
1. Project Type → Web App / Mobile API / Dashboard / Integration / Automation
2. Tech Stack → Frontend + Backend + Database + External APIs
3. Key Requirements → Authentication? Real-time? Payments? Reporting?
4. Scale → Single user / Team (10-100) / Enterprise (1000+)
5. Deployment → Vercel/Netlify / Self-hosted / Cloud Functions / Hybrid
```

---

# PART I: QUICK PATTERNS (Essentials)

## Database - PostgreSQL Common Patterns

```sql
-- Standard table pattern (use this as starting point)
CREATE TABLE table_name (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id TEXT NOT NULL,
  status TABLE_STATUS DEFAULT 'pending',
  metadata JSONB DEFAULT '{}'::jsonb,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Index strategies (choose based on query patterns)
CREATE INDEX idx_table_column ON table_name(column);                    -- B-tree (default)
CREATE INDEX idx_table_composite ON table_name(col1, col2);              -- Composite (order matters!)
CREATE INDEX idx_table_partial ON table_name(col) WHERE status = 'active'; -- Partial index (faster)
CREATE INDEX idx_table_jsonb ON table_name USING GIN(jsonb_col);         -- JSONB search

-- RLS Policy Pattern (for multi-tenant apps)
ALTER TABLE table_name ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can view own data" ON table_name
  FOR SELECT USING (user_id = auth.uid()::text);

CREATE POLICY "Users can insert own data" ON table_name
  FOR INSERT WITH CHECK (user_id = auth.uid()::text);

CREATE POLICY "Admins full access" ON table_name
  FOR ALL USING (auth.uid() IN (SELECT id FROM admin_users WHERE is_active = true));

-- JSONB Operations
SELECT * FROM table WHERE jsonb_col->>'key' = 'value';              -- Get value
SELECT * FROM table WHERE jsonb_col @> '{"key": "value"}';          -- Contains
SELECT * FROM table WHERE jsonb_col ? 'key';                        -- Has key
UPDATE table SET jsonb_col = jsonb_set(jsonb_col, '{path}', '"val"'); -- Update nested

-- Common Window Functions
SELECT *,
  ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY created_at) AS rn,
  RANK() OVER (PARTITION BY user_id ORDER BY score DESC) AS rank,
  LAG(value) OVER (ORDER BY date) AS prev_value,
  SUM(amount) OVER (ORDER BY created_at ROWS UNBOUNDED PRECEDING) AS running_total
FROM table;
```

## TypeScript - Type-Safe Supabase

```typescript
// Generate types: supabase gen types typescript --local > types/database.ts
export interface Database {
  public: {
    Tables: {
      table_name: {
        Row: { id: string; user_id: string; status: string; created_at: string }
        Insert: { id?: string; user_id: string; status?: string }
        Update: { id?: string; user_id?: string; status?: string }
      }
    }
  }
}

// Type-safe client
import { createClient, SupabaseClient } from '@supabase/supabase-js';

const supabase: SupabaseClient<Database> = createClient(url, key);

// Type-safe queries with joins
const { data, error } = await supabase
  .from('table_name')
  .select('*, related_table(*)')
  .eq('user_id', userId)
  .order('created_at', { ascending: false });

// Real-time subscription
const channel = supabase
  .channel(`channel-${userId}`)
  .on('postgres_changes', {
    event: '*',
    schema: 'public',
    table: 'table_name',
    filter: `user_id=eq.${userId}`
  }, (payload) => {
    console.log('Change:', payload);
    // Handle INSERT, UPDATE, DELETE
  })
  .subscribe();

// Clean up
return () => { supabase.removeChannel(channel); };

// Utility types
type WithRequired<T, K extends keyof T> = T & { [P in K]-?: T[P] };
type Nullable<T> = T | null;
type AsyncResult<T, E = Error> = Promise<[T, null] | [null, E]>;

// Async try-catch helper
export async function asyncTry<T, E = Error>(
  promise: Promise<T>
): AsyncResult<T, E> {
  try {
    const data = await promise;
    return [data, null];
  } catch (error) {
    return [null, error as E];
  }
}

// Generic repository pattern
export class Repository<T extends keyof Database['public']['Tables']> {
  constructor(private table: T) {}

  async list(filters?: Partial<Database['public']['Tables'][T]['Row']>) {
    let query = supabase.from(this.table).select('*');
    if (filters) {
      Object.entries(filters).forEach(([k, v]) => {
        if (v !== undefined) query = query.eq(k, v);
      });
    }
    return query;
  }

  async getById(id: string) {
    return supabase.from(this.table).select('*').eq('id', id).single();
  }

  async insert(record: Database['public']['Tables'][T]['Insert']) {
    return supabase.from(this.table).insert(record).select().single();
  }

  async update(id: string, updates: Database['public']['Tables'][T]['Update']) {
    return supabase.from(this.table).update(updates).eq('id', id).select().single();
  }
}
```

## React + Supabase Realtime Component

```typescript
import { useState, useEffect, useCallback } from 'react';
import { supabase } from '@/lib/supabase';
import type { Database } from '@/types/database';

type TableRow = Database['public']['Tables']['your_table']['Row'];

export function DataTable({ userId }: { userId: string }) {
  const [data, setData] = useState<TableRow[]>([]);
  const [loading, setLoading] = useState(true);

  const fetchData = useCallback(async () => {
    const { data, error } = await supabase
      .from('your_table')
      .select('*')
      .eq('user_id', userId)
      .order('created_at', { ascending: false });

    if (error) {
      console.error('Error fetching data:', error);
      return;
    }

    setData(data ?? []);
    setLoading(false);
  }, [userId]);

  useEffect(() => {
    fetchData();

    // Real-time subscription
    const channel = supabase
      .channel(`table-${userId}`)
      .on('postgres_changes', {
        event: '*',
        schema: 'public',
        table: 'your_table',
        filter: `user_id=eq.${userId}`
      }, fetchData)
      .subscribe();

    return () => { supabase.removeChannel(channel); };
  }, [fetchData]);

  if (loading) return <Spinner />;
  if (data.length === 0) return <EmptyState />;

  return (
    <div className="grid gap-4">
      {data.map(row => <RowCard key={row.id} row={row} />)}
    </div>
  );
}
```

## LINE LIFF Integration

```javascript
// Initialize LIFF
import liff from '@line/liff';

const LIFF_ID = import.meta.env.VITE_LIFF_ID;

async function initLiff() {
  try {
    await liff.init({ liffId: LIFF_ID });
    return true;
  } catch (error) {
    console.error('LIFF init failed:', error);
    return false;
  }
}

// Get user profile
async function getProfile() {
  if (!liff.isLoggedIn()) {
    liff.login({ redirectUri: window.location.href });
    return null;
  }

  const profile = await liff.getProfile();
  // Returns: { userId, displayName, pictureUrl, statusMessage, language }

  // Get context (1:1 chat vs group chat)
  const context = liff.getContext();
  // Returns: { type: 'ut' | 'none', viewType: 'full' | 'tall' | 'compact', userId }

  return { ...profile, context };
}

// Send message and close
async function completeTask(message) {
  await liff.sendMessages([{ type: 'text', text: message }]);
  liff.closeWindow();
}

// Check if in LINE app
const isInClient = liff.isInClient();
```

## LINE Messaging API

```typescript
// Flex Message template
const createFlexMessage = (title: content, items: any[]) => ({
  type: 'flex',
  altText: title,
  contents: {
    type: 'bubble',
    header: {
      type: 'box',
      layout: 'vertical',
      contents: [{
        type: 'text',
        text: title,
        color: '#FFFFFF',
        size: 'md',
        align: 'center',
        weight: 'bold'
      }],
      backgroundColor: '#00B900',
      paddingAll: 'md'
    },
    body: {
      type: 'box',
      layout: 'vertical',
      contents: items.map(item => ({
        type: 'text',
        text: item.label,
        margin: 'md'
      })),
      paddingAll: 'lg'
    }
  }
});

// Webhook signature verification
import crypto from 'crypto';

function verifyLineSignature(body: string, signature: string, channelSecret: string): boolean {
  const hash = crypto
    .createHmac('SHA256', channelSecret)
    .update(body)
    .digest('base64');

  return signature === hash;
}

// Express webhook handler
import express from 'express';

const app = express();

app.post('/webhook',
  express.raw({ type: 'application/json' }),
  (req, res, next) => {
    const signature = req.headers['x-line-signature'];
    if (!verifyLineSignature(req.body.toString(), signature, process.env.LINE_CHANNEL_SECRET!)) {
      return res.status(401).send('Invalid signature');
    }
    next();
  },
  async (req, res) => {
    const events = JSON.parse(req.body.toString()).events;

    for (const event of events) {
      switch (event.type) {
        case 'message': await handleMessage(event); break;
        case 'follow': await handleFollow(event); break;
        case 'postback': await handlePostback(event); break;
        case 'unfollow': await handleUnfollow(event); break;
      }
    }

    res.status(200).send('OK');
  }
);
```

## Python FastAPI Backend

```python
# main.py
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel
from typing import Optional, List
import asyncpg
import os

app = FastAPI(
    title="API",
    description="Production API",
    version="1.0.0"
)

# CORS configuration
app.add_middleware(
    CORSMiddleware,
    allow_origins=os.getenv("ALLOWED_ORIGINS", "http://localhost:3000").split(","),
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Database connection pool
@app.on_event("startup")
async def startup():
    app.db_pool = await asyncpg.create_pool(
        host=os.getenv("DB_HOST"),
        database=os.getenv("DB_NAME"),
        user=os.getenv("DB_USER"),
        password=os.getenv("DB_PASSWORD"),
        min_size=5,
        max_size=20
    )

@app.on_event("shutdown")
async def shutdown():
    await app.db_pool.close()

# Pydantic models
class ItemResponse(BaseModel):
    id: str
    name: str
    status: str
    created_at: str

class ItemCreate(BaseModel):
    name: str
    metadata: Optional[dict] = None

# Routes with filters
@app.get("/api/items", response_model=List[ItemResponse])
async def list_items(
    status: Optional[str] = None,
    limit: int = 100,
    offset: int = 0
):
    async with app.db_pool.acquire() as conn:
        query = "SELECT * FROM items WHERE 1=1"
        params = []
        count = 0

        if status:
            count += 1
            query += f" AND status = ${count}"
            params.append(status)

        query += f" ORDER BY created_at DESC LIMIT ${count + 1} OFFSET ${count + 2}"
        params.extend([limit, offset])

        rows = await conn.fetch(query, *params)
        return [dict(row) for row in rows]

@app.post("/api/items", response_model=ItemResponse, status_code=status.HTTP_201_CREATED)
async def create_item(item: ItemCreate):
    async with app.db_pool.acquire() as conn:
        row = await conn.fetchrow("""
            INSERT INTO items (name, metadata, status)
            VALUES ($1, $2, 'active')
            RETURNING *
        """, item.name, item.metadata)

        return ItemResponse(**dict(row))
```

## Google Apps Script Patterns

```javascript
// Supabase integration helper
const Supabase = {
  url: PropertiesService.getScriptProperties().getProperty('SUPABASE_URL'),
  key: PropertiesService.getScriptProperties().getProperty('SUPABASE_KEY'),

  fetch(table, options = {}) {
    const { select = '*', filter = '', order = '', limit = 100 } = options;

    let url = `${this.url}/rest/v1/${table}?select=${select}&limit=${limit}`;
    if (filter) url += `&${filter}`;
    if (order) url += `&order=${order}`;

    const response = UrlFetchApp.fetch(url, {
      headers: {
        'apikey': this.key,
        'Authorization': `Bearer ${this.key}`,
        'Content-Type': 'application/json'
      },
      muteHttpExceptions: true
    });

    if (response.getResponseCode() !== 200) {
      throw new Error(`Supabase error: ${response.getContentText()}`);
    }

    return JSON.parse(response.getContentText());
  },

  insert(table, data) {
    return UrlFetchApp.fetch(`${this.url}/rest/v1/${table}`, {
      method: 'post',
      headers: {
        'apikey': this.key,
        'Authorization': `Bearer ${this.key}`,
        'Content-Type': 'application/json'
      },
      payload: JSON.stringify(data),
      muteHttpExceptions: true
    });
  }
};

// Sync to Google Sheets
function syncToSheet(sheetName, tableName, columns) {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sheet = ss.getSheetByName(sheetName) || ss.insertSheet(sheetName);

  const data = Supabase.fetch(tableName, { limit: 1000 });

  sheet.clearContents();
  sheet.getRange(1, 1, 1, columns.length).setValues([columns])
    .setBackground('#00B900')
    .setFontColor('#FFFFFF')
    .setFontWeight('bold');

  if (data.length > 0) {
    const rows = data.map(row => columns.map(col => row[col] || ''));
    sheet.getRange(2, 1, rows.length, columns.length).setValues(rows);
  }

  sheet.autoResizeColumns(1, columns.length);
  return data.length;
}

// Create menu on open
function onOpen() {
  const ui = SpreadsheetApp.getUi();
  ui.createMenu('🚀 Sync')
    .addItem('Sync Data', 'syncAll')
    .addItem('Refresh Summary', 'refreshSummary')
    .addToUi();
}
```

## Playwright E2E Tests

```typescript
// tests/e2e/user-workflow.spec.ts
import { test, expect } from '@playwright/test';

test.describe('User Workflow', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/');
  });

  test('complete user flow', async ({ page }) => {
    // Login
    await page.click('[data-testid="login-btn"]');
    await page.fill('[data-testid="email-input"]', 'test@example.com');
    await page.fill('[data-testid="password-input"]', 'password123');
    await page.click('[data-testid="submit-btn"]');

    // Should redirect to dashboard
    await expect(page).toHaveURL(/\/dashboard/);
    await expect(page.locator('h1')).toContainText('Welcome');

    // Create new item
    await page.click('[data-testid="create-btn"]');
    await page.fill('[data-testid="item-name"]', 'Test Item');
    await page.selectOption('[data-testid="item-status"]', 'active');
    await page.click('[data-testid="save-btn"]');

    // Verify success message
    await expect(page.locator('[data-testid="toast-success"]')).toBeVisible();
    await expect(page.locator('[data-testid="toast-success"]')).toContainText('saved successfully');

    // Verify item appears in list
    await expect(page.locator('text=Test Item')).toBeVisible();
  });

  test('validation errors', async ({ page }) => {
    await page.click('[data-testid="create-btn"]');

    // Submit without required fields
    await page.click('[data-testid="save-btn"]');

    // Should show validation errors
    await expect(page.locator('[data-testid="error-name"]')).toBeVisible();
    await expect(page.locator('[data-testid="error-name"]')).toContainText('required');
  });

  test('responsive design', async ({ page }) => {
    // Test mobile viewport
    await page.setViewportSize({ width: 375, height: 667 });
    await page.goto('/');

    // Mobile menu should be visible
    await expect(page.locator('[data-testid="mobile-menu-btn"]')).toBeVisible();

    // Desktop elements should be hidden
    await expect(page.locator('[data-testid="desktop-nav"]')).not.toBeVisible();
  });
});
```

---

# PART II: DEEP DIVES

## Database - Advanced PostgreSQL Patterns

### PostGIS for Location-Based Features

```sql
-- Enable PostGIS
CREATE EXTENSION IF NOT EXISTS postgis;

-- Add geometry column
ALTER TABLE locations ADD COLUMN geom GEOMETRY(Point, 4326);

-- Create spatial index
CREATE INDEX idx_locations_geom ON locations USING GIST(geom);

-- Populate geometry from lat/lng
UPDATE locations
SET geom = ST_SetSRID(ST_MakePoint(lng, lat), 4326)
WHERE geom IS NULL;

-- Find points within radius (meters)
SELECT
  id,
  name,
  ST_Distance(geom, ST_MakePoint($1, $2)::geography) AS distance_meters
FROM locations
WHERE ST_DWithin(geom, ST_SetSRID(ST_MakePoint($1, $2), 4326)::geography, $3)
ORDER BY distance_meters;

-- Calculate distance between two points
SELECT
  ST_Distance(
    ST_MakePoint(100.5018, 13.7563)::geography,
    ST_MakePoint(100.5218, 13.7263)::geography
  ) / 1000 AS distance_km;
```

### Recursive CTE for Hierarchical Data

```sql
-- Get entire hierarchy tree
WITH RECURSIVE tree AS (
  -- Base case: root nodes
  SELECT id, name, parent_id, 1 AS level, ARRAY[id] AS path
  FROM categories
  WHERE parent_id IS NULL

  UNION ALL

  -- Recursive case: children
  SELECT c.id, c.name, c.parent_id, t.level + 1, t.path || c.id
  FROM categories c
  INNER JOIN tree t ON c.parent_id = t.id
)
SELECT * FROM tree ORDER BY level, name;
```

### Materialized Views for Performance

```sql
-- Create materialized view for dashboard/stats
CREATE MATERIALIZED VIEW mv_daily_stats AS
SELECT
  created_at::date AS date,
  COUNT(*) AS total_count,
  COUNT(*) FILTER (WHERE status = 'active') AS active_count,
  COUNT(*) FILTER (WHERE status = 'completed') AS completed_count,
  AVG(amount) AS avg_amount
FROM transactions
GROUP BY created_at::date;

-- Unique index for concurrent refresh
CREATE UNIQUE INDEX idx_mv_daily_stats_date ON mv_daily_stats(date);

-- Refresh (can be run concurrently without blocking reads)
REFRESH MATERIALIZED VIEW CONCURRENTLY mv_daily_stats;
```

### Full-Text Search

```sql
-- Add full-text search
ALTER TABLE articles ADD COLUMN tsv tsvector GENERATED ALWAYS AS (
  to_tsvector('english', coalesce(title, '') || ' ' || coalesce(content, ''))
) STORED;

-- Create GIN index
CREATE INDEX idx_articles_tsv ON articles USING GIN(tsv);

-- Search query
SELECT
  id,
  title,
  ts_headline('english', tsv, plainto_tsquery('english', $1)) AS highlight,
  ranking
FROM articles,
     to_tsquery('english', $1) query
WHERE tsv @@ query
ORDER BY ts_rank(tsv, query) DESC;
```

## Security - Production Best Practices

### RLS with Service Role Token Swap

```typescript
// For client apps where anon key is used initially
// Exchange for authenticated token after verifying ownership

// Edge Function: supabase/functions/auth-swap/index.ts
import { serve } from 'https://deno.land/std@0.168.0/http/server.ts';
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2';

serve(async (req) => {
  const { userId, provider, providerToken } = await req.json();

  const supabase = createClient(
    Deno.env.get('SUPABASE_URL')!,
    Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!
  );

  // Verify user exists and owns this identity
  const { data: profile } = await supabase
    .from('user_profiles')
    .select('*')
    .eq('id', userId)
    .eq('provider_user_id', providerToken)
    .single();

  if (!profile) {
    return new Response('User not found', { status: 404 });
  }

  // Generate a temporary token with user context
  const { data: { session } } = await supabase.auth.admin.createUserId({
    user_id: userId,
    email: profile.email,
    email_confirm: true
  });

  return new Response(JSON.stringify({
    token: session.access_token,
    user: { id: profile.id, email: profile.email }
  }));
});
```

### XSS Prevention Utilities

```typescript
// utils/sanitize.ts
export function sanitizeHTML(str: string): string {
  const div = document.createElement('div');
  div.textContent = str;
  return div.innerHTML;
}

export function sanitizeInput(obj: Record<string, any>): Record<string, any> {
  const sanitized: Record<string, any> = {};

  for (const [key, value] of Object.entries(obj)) {
    if (typeof value === 'string') {
      sanitized[key] = sanitizeHTML(value.trim());
    } else if (typeof value === 'object' && value !== null) {
      sanitized[key] = sanitizeInput(value);
    } else {
      sanitized[key] = value;
    }
  }

  return sanitized;
}

// Sanitize URL parameters
export function sanitizeParam(param: string): string {
  return param.replace(/[^a-zA-Z0-9-_]/g, '');
}
```

### Rate Limiting Middleware

```typescript
// Rate limiter using database for distributed systems
import { supabase } from './supabase';

export async function checkRateLimit(
  identifier: string,
  limit: number = 100,
  windowMs: number = 60000
): Promise<{ allowed: boolean; remaining: number; resetAt: Date }> {
  const now = new Date();
  const windowStart = new Date(now.getTime() - windowMs);

  // Clean old entries
  await supabase
    .from('rate_limits')
    .delete()
    .lt('window_start', windowStart);

  // Get current count
  const { data: current } = await supabase
    .from('rate_limits')
    .select('count')
    .eq('identifier', identifier)
    .gte('window_start', windowStart)
    .single();

  const count = current?.count || 0;

  if (count >= limit) {
    return {
      allowed: false,
      remaining: 0,
      resetAt: new Date(windowStart.getTime() + windowMs)
    };
  }

  // Increment counter
  await supabase
    .from('rate_limits')
    .upsert({
      identifier,
      count: count + 1,
      window_start: windowStart
    }, {
      onConflict: 'identifier,window_start'
    });

  return {
    allowed: true,
    remaining: limit - count - 1,
    resetAt: new Date(windowStart.getTime() + windowMs)
  };
}
```

## Offline-First Architecture

```typescript
// services/offlineQueue.ts
interface QueuedAction {
  id: string;
  type: string;
  endpoint: string;
  method: 'GET' | 'POST' | 'PUT' | 'DELETE';
  payload?: any;
  timestamp: number;
  retries: number;
}

class OfflineQueue {
  private queue: QueuedAction[] = [];
  private storageKey = 'offline_queue';
  private isOnline: boolean = navigator.onLine;

  constructor() {
    this.loadFromStorage();
    this.setupEventListeners();
  }

  private loadFromStorage() {
    try {
      const stored = localStorage.getItem(this.storageKey);
      if (stored) this.queue = JSON.parse(stored);
    } catch (e) {
      console.error('Failed to load queue:', e);
    }
  }

  private saveToStorage() {
    localStorage.setItem(this.storageKey, JSON.stringify(this.queue));
  }

  private setupEventListeners() {
    window.addEventListener('online', () => {
      this.isOnline = true;
      this.processQueue();
    });

    window.addEventListener('offline', () => {
      this.isOnline = false;
    });
  }

  add(action: Omit<QueuedAction, 'id' | 'timestamp' | 'retries'>): string {
    const queued: QueuedAction = {
      ...action,
      id: `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`,
      timestamp: Date.now(),
      retries: 0
    };

    this.queue.push(queued);
    this.saveToStorage();

    if (this.isOnline) {
      this.processQueue();
    }

    return queued.id;
  }

  async processQueue(): Promise<{ success: number; failed: number }> {
    if (!this.isOnline || this.queue.length === 0) {
      return { success: 0, failed: 0 };
    }

    let success = 0;
    let failed = 0;

    for (let i = this.queue.length - 1; i >= 0; i--) {
      const action = this.queue[i];

      try {
        await this.executeAction(action);
        this.queue.splice(i, 1);
        success++;
      } catch (error) {
        action.retries++;

        if (action.retries >= 3) {
          this.queue.splice(i, 1);
          console.error('Action failed after 3 retries:', action);
        }

        failed++;
      }
    }

    this.saveToStorage();
    return { success, failed };
  }

  private async executeAction(action: QueuedAction): Promise<Response> {
    const { endpoint, method, payload } = action;

    const response = await fetch(endpoint, {
      method,
      headers: { 'Content-Type': 'application/json' },
      body: payload ? JSON.stringify(payload) : undefined
    });

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }

    return response;
  }

  getStatus() {
    return {
      queueLength: this.queue.length,
      isOnline: this.isOnline,
      pendingActions: this.queue.map(a => ({ id: a.id, type: a.type }))
    };
  }
}

export const offlineQueue = new OfflineQueue();
```

## Mobile-First Design System

```css
/* :root - Design tokens */
:root {
  /* Brand colors */
  --color-primary: #00B900;
  --color-primary-dark: #009100;
  --color-secondary: #0066FF;

  /* Semantic colors */
  --color-success: #22C55E;
  --color-warning: #F59E0B;
  --color-error: #EF4444;
  --color-info: #3B82F6;

  /* Status colors */
  --status-pending: #F59E0B;
  --status-active: #3B82F6;
  --status-completed: #22C55E;
  --status-cancelled: #EF4444;

  /* Typography */
  --font-sans: system-ui, -apple-system, sans-serif;
  --text-xs: 0.75rem;
  --text-sm: 0.875rem;
  --text-base: 1rem;
  --text-lg: 1.125rem;
  --text-xl: 1.25rem;
  --text-2xl: 1.5rem;

  /* Spacing */
  --space-1: 0.25rem;
  --space-2: 0.5rem;
  --space-3: 0.75rem;
  --space-4: 1rem;
  --space-6: 1.5rem;
  --space-8: 2rem;

  /* Touch targets (WCAG compliant) */
  --touch-target: 44px;

  /* Border radius */
  --radius-sm: 0.25rem;
  --radius-md: 0.5rem;
  --radius-lg: 0.75rem;
  --radius-full: 9999px;
}

/* Dark mode */
@media (prefers-color-scheme: dark) {
  :root {
    --color-bg: #111827;
    --color-text: #F9FAFB;
    --color-border: #374151;
  }
}

/* Reset & Base */
*, *::before, *::after {
  box-sizing: border-box;
}

body {
  font-family: var(--font-sans);
  font-size: var(--text-base);
  line-height: 1.5;
  color: var(--color-text, #1F2937);
  background-color: var(--color-bg, #FFFFFF);
}

/* Components */
.btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: var(--space-2);
  padding: var(--space-3) var(--space-6);
  font-family: inherit;
  font-size: var(--text-base);
  font-weight: 500;
  line-height: 1;
  border: none;
  border-radius: var(--radius-md);
  cursor: pointer;
  text-decoration: none;
  transition: all 0.2s;
  min-height: var(--touch-target);
  min-width: var(--touch-target);
}

.btn-primary {
  background-color: var(--color-primary);
  color: white;
}

.btn-primary:hover { background-color: var(--color-primary-dark); }

.btn-secondary {
  background-color: transparent;
  border: 2px solid var(--color-primary);
  color: var(--color-primary);
}

.card {
  background: white;
  border-radius: var(--radius-lg);
  box-shadow: 0 1px 3px rgb(0 0 0 / 0.1);
  padding: var(--space-6);
}

.input {
  width: 100%;
  padding: var(--space-3) var(--space-4);
  font-family: inherit;
  font-size: var(--text-base);
  border: 2px solid var(--color-border, #E5E7EB);
  border-radius: var(--radius-md);
  min-height: var(--touch-target);
}

.input:focus {
  outline: none;
  border-color: var(--color-primary);
  box-shadow: 0 0 0 3px rgba(0, 185, 0, 0.1);
}

/* Container & Grid */
.container {
  width: 100%;
  padding: var(--space-4);
  margin: 0 auto;
}

@media (min-width: 640px) { .container { max-width: 640px; } }
@media (min-width: 768px) { .container { max-width: 768px; } }
@media (min-width: 1024px) { .container { max-width: 1024px; } }

.grid {
  display: grid;
  gap: var(--space-4);
}
.grid-cols-1 { grid-template-columns: repeat(1, 1fr); }
@media (min-width: 640px) {
  .grid-cols-2 { grid-template-columns: repeat(2, 1fr); }
}
@media (min-width: 1024px) {
  .grid-cols-4 { grid-template-columns: repeat(4, 1fr); }
}
```

## Lean Six Sigma - Process Improvement

```python
# lean/six_sigma.py
"""
Six Sigma DMAIC Framework
Apply to any process improvement project
"""

from dataclasses import dataclass
from typing import List, Dict
import pandas as pd

@dataclass
class ProblemStatement:
    """Define Phase"""
    what: str       # What is the problem?
    where: str      # Where does it occur?
    when: str       # When does it occur?
    who: str        # Who is affected?
    impact: str     # Business impact

    def to_statement(self) -> str:
        return f"""Problem: {self.what}
Location: {self.where}
Timing: {self.when}
Affected: {self.who}
Impact: {self.impac}"""

class MeasurePhase:
    """Measure Phase - Data collection & baseline metrics"""

    @staticmethod
    def calculate_dpmo(defects: int, opportunities: int, units: int) -> float:
        """
        Defects Per Million Opportunities
        6σ = 3.4 DPMO, 5σ = 233, 4σ = 6,210, 3σ = 66,807
        """
        return (defects / (opportunities * units)) * 1_000_000

    @staticmethod
    def dpmo_to_sigma(dpmo: float) -> float:
        sigma_map = {3.4: 6, 233: 5, 6210: 4, 66807: 3, 308538: 2}
        return min(sigma_map.items(), key=lambda x: abs(x[0] - dpmo))[1]

    @staticmethod
    def calculate_cp(usl: float, lsl: float, std_dev: float) -> float:
        """Process Capability Index - Cp > 1.33 is capable"""
        return (usl - lsl) / (6 * std_dev)

    @staticmethod
    def calculate_ppk(usl: float, lsl: float, mean: float, std_dev: float) -> float:
        """Process Capability Index with centering"""
        cpu = (usl - mean) / (3 * std_dev)
        cpl = (mean - lsl) / (3 * std_dev)
        return min(cpu, cpl)

class AnalyzePhase:
    """Root Cause Analysis"""

    @staticmethod
    def fishbone_template() -> Dict[str, List[str]]:
        """5M1E categories - customize for your process"""
        return {
            'Man': ['Training', 'Skills', 'Fatigue'],
            'Machine': ['Equipment', 'Tools', 'Maintenance'],
            'Material': ['Quality', 'Supply', 'Specifications'],
            'Method': ['Process', 'SOP', 'Workflow'],
            'Mother Nature': ['Environment', 'Conditions'],
            'Measurement': ['Accuracy', 'Calibration', 'Definitions']
        }

class ImprovePhase:
    """Improvement Implementation"""

    @staticmethod
    def calculate_roi(cost: float, annual_savings: float) -> float:
        """Return on Investment - ROI > 3 is typically good"""
        return (annual_savings - cost) / cost if cost > 0 else 0

    def prioritize_improvements(self, ideas: List[Dict]) -> List[Dict]:
        """Prioritize by ROI and implementation time"""
        for idea in ideas:
            idea['roi'] = self.calculate_roi(idea['cost'], idea['annual_savings'])

        # Quick wins first, then by ROI
        quick_wins = sorted(
          [i for i in ideas if i.get('implementation') == 'quick'],
          key=lambda x: x['roi'], reverse=True
        )
        others = sorted(
          [i for i in ideas if i.get('implementation') != 'quick'],
          key=lambda x: x['roi'], reverse=True
        )
        return quick_wins + others
```

## SaaS Metrics & Business Analytics

```python
# analytics/saas.py
"""
SaaS Metrics Calculator
Track MRR, ARR, LTV, CAC, Churn, NRR
"""

import pandas as pd
from datetime import date, timedelta
from typing import Dict

class SaaSMetrics:
    """Calculate SaaS key performance indicators"""

    def __init__(self, df: pd.DataFrame):
        """
        DataFrame columns: customer_id, subscription_start, subscription_end,
        mrr, plan_tier, expansion_amount, downgrade_amount
        """
        self.df = df

    def calculate_mrr(self) -> Dict:
        """Monthly Recurring Revenue breakdown"""
        active = self.df[
            self.df['subscription_end'].isna() |
            (self.df['subscription_end'] > date.today())
        ]

        return {
            'total_mrr': active['mrr'].sum(),
            'new_mrr': self._new_mrr(),
            'expansion_mrr': self._expansion_mrr(),
            'churn_mrr': self._churn_mrr(),
            'net_new_mrr': (
                self._new_mrr() + self._expansion_mrr() - self._churn_mrr()
            )
        }

    def calculate_arr(self) -> float:
        """Annual Recurring Revenue"""
        return self.calculate_mrr()['total_mrr'] * 12

    def calculate_cac(self, marketing_spend: float, new_customers: int) -> float:
        """Customer Acquisition Cost"""
        return marketing_spend / new_customers if new_customers > 0 else 0

    def calculate_ltv(self, arpu: float, gross_margin: float, churn_rate: float) -> float:
        """
        Customer Lifetime Value
        LTV = (ARPU × Gross Margin) / Churn Rate
        """
        if churn_rate == 0:
            return arpu * 36  # Default to 36 months if no churn
        return (arpu * gross_margin) / churn_rate

    def ltv_cac_ratio(self, ltv: float, cac: float) -> float:
        """
        LTV:CAC Ratio
        < 1: Losing money
        1-3: Breakeven to good
        > 3: Healthy
        """
        return ltv / cac if cac > 0 else 0

    def calculate_churn_rate(self, days: int = 30) -> float:
        """Churn Rate = Churned Customers / Total Customers"""
        cutoff = date.today() - timedelta(days=days)

        total = self.df[self.df['subscription_start'] <= cutoff]
        churned = self.df[
            (self.df['subscription_end'] >= cutoff) &
            (self.df['subscription_end'] <= date.today())
        ]

        return (len(churned) / len(total)) * 100 if len(total) > 0 else 0

    def calculate_arpu(self) -> float:
        """Average Revenue Per User"""
        active = self.df[
            self.df['subscription_end'].isna() |
            (self.df['subscription_end'] > date.today())
        ]
        return active['mrr'].mean() if len(active) > 0 else 0

    def _new_mrr(self) -> float:
        cutoff = date.today().replace(day=1)
        return self.df[self.df['subscription_start'] >= cutoff]['mrr'].sum()

    def _churn_mrr(self) -> float:
        cutoff = date.today().replace(day=1)
        return self.df[
            (self.df['subscription_end'] >= cutoff) &
            (self.df['subscription_end'] <= date.today())
        ]['mrr'].sum()

    def _expansion_mrr(self) -> float:
        # Calculate from expansion_amount column
        return self.df[self.df['subscription_end'].isna()]['expansion_amount'].sum()

# SQL Analytics Queries
sql_templates = {
    'daily_metrics': """
        WITH daily AS (
            SELECT
                created_at::date AS date,
                COUNT(*) AS total_items,
                COUNT(*) FILTER (WHERE status = 'active') AS active_items,
                COUNT(*) FILTER (WHERE status = 'completed') AS completed_items,
                AVG(amount) AS avg_amount
            FROM transactions
            WHERE created_at >= CURRENT_DATE - INTERVAL '30 days'
            GROUP BY created_at::date
        )
        SELECT
            date,
            total_items,
            active_items,
            completed_items,
            ROUND((active_items::NUMERIC / NULLIF(total_items, 0)) * 100, 1) AS active_rate
        FROM daily ORDER BY date DESC;
    """,

    'cohort_retention': """
        WITH user_cohorts AS (
            SELECT
                user_id,
                DATE_TRUNC('month', MIN(created_at)) AS cohort_month
            FROM users
            GROUP BY user_id, DATE_TRUNC('month', MIN(created_at))
        ),
        user_activity AS (
            SELECT
                u.user_id,
                u.cohort_month,
                DATE_TRUNC('month', a.created_at) AS activity_month,
                EXTRACT(MONTH FROM AGE(a.created_at, u.cohort_month))::int AS month_number
            FROM user_cohorts u
            JOIN activities a ON a.user_id = u.user_id
            WHERE a.created_at >= u.cohort_month
        )
        SELECT
            cohort_month,
            month_number,
            COUNT(DISTINCT user_id) AS active_users
        FROM user_activity
        GROUP BY cohort_month, month_number
        ORDER BY cohort_month, month_number;
    """,

    'funnel_analysis': """
        WITH funnel_steps AS (
            SELECT 'page_view' AS step, COUNT(DISTINCT user_id) AS count
            FROM page_views WHERE created_at >= CURRENT_DATE
            UNION ALL
            SELECT 'signup' AS step, COUNT(DISTINCT user_id) AS count
            FROM users WHERE created_at >= CURRENT_DATE
            UNION ALL
            SELECT 'purchase' AS step, COUNT(DISTINCT user_id) AS count
            FROM orders WHERE created_at >= CURRENT_DATE
        )
        SELECT
            step,
            count,
            LAG(count) OVER (ORDER BY count DESC) - count AS drop_off,
            ROUND(
                (count::NUMERIC / LAG(count) OVER (ORDER BY count DESC)) * 100,
                1
            ) AS conversion_rate
        FROM funnel_steps
        ORDER BY count DESC;
    """
}
```

## Thailand-Specific Patterns

### Timezone & Date Handling

```typescript
// utils/thai-date.ts
/**
 * Thailand-specific date utilities
 * Timezone: Asia/Bangkok (UTC+7)
 */

// Constants
export const THAI_TIMEZONE = 'Asia/Bangkok';
export const THAI_LOCALE = 'th-TH';
export const BUDDHIST_YEAR_OFFSET = 543;

// Format date in Thai timezone
export function formatThaiDate(
  date: Date | string,
  format: 'full' | 'short' | 'time' = 'short'
): string {
  const d = typeof date === 'string' ? new Date(date) : date;

  const options: Intl.DateTimeFormatOptions = {
    timeZone: THAI_TIMEZONE,
    calendar: 'buddhist'
  };

  switch (format) {
    case 'full':
      return d.toLocaleDateString(THAI_LOCALE, {
        ...options,
        weekday: 'long',
        year: 'numeric',
        month: 'long',
        day: 'numeric'
      });
    case 'short':
      return d.toLocaleDateString(THAI_LOCALE, {
        ...options,
        year: '2-digit',
        month: '2-digit',
        day: '2-digit'
      });
    case 'time':
      return d.toLocaleTimeString(THAI_LOCALE, {
        timeZone: THAI_TIMEZONE,
        hour: '2-digit',
        minute: '2-digit'
      });
  }
}

// Convert to Buddhist year (พ.ศ.)
export function toBuddhistYear(year: number): number {
  return year + BUDDHIST_YEAR_OFFSET;
}

// Convert from Buddhist year to Christian year (ค.ศ.)
export function fromBuddhistYear(buddhistYear: number): number {
  return buddhistYear - BUDDHIST_YEAR_OFFSET;
}

// Get Thai month name
export function getThaiMonth(month: number): string {
  const months = [
    'มกราคม', 'กุมภาพันธ์', 'มีนาคม', 'เมษายน',
    'พฤษภาคม', 'มิถุนายน', 'กรกฎาคม', 'สิงหาคม',
    'กันยายน', 'ตุลาคม', 'พฤศจิกายน', 'ธันวาคม'
  ];
  return months[month - 1] || '';
}

// Format Thai date: "2 มกราคม 2568"
export function formatThaiDateShort(date: Date | string): string {
  const d = typeof date === 'string' ? new Date(date) : date;
  return `${d.getDate()} ${getThaiMonth(d.getMonth() + 1)} ${toBuddhistYear(d.getFullYear())}`;
}

// Parse Thai date string (supports both Buddhist and Christian years)
export function parseThaiDate(dateStr: string): Date | null {
  // Handle formats: "02/01/2568", "02/01/68", "2025-01-02"
  try {
    // Try standard parsing first
    const standard = new Date(dateStr);
    if (!isNaN(standard.getTime())) {
      return standard;
    }

    // Parse Thai format DD/MM/YYYY or DD/MM/YY
    const parts = dateStr.split('/');
    if (parts.length === 3) {
      let [day, month, year] = parts.map(Number);

      // Convert 2-digit year
      if (year < 100) {
        year += year > 50 ? 1900 : 2000;
      }

      // Convert Buddhist year to Christian if needed
      if (year > 2500) {
        year = fromBuddhistYear(year);
      }

      return new Date(year, month - 1, day);
    }
  } catch (e) {
    console.error('Failed to parse Thai date:', e);
  }
  return null;
}

// Get current timestamp in Thai timezone
export function nowThai(): Date {
  return new Date(new Date().toLocaleString('en-US', { timeZone: THAI_TIMEZONE }));
}

// Check if date is today in Thai timezone
export function isTodayThai(date: Date | string): boolean {
  const d = typeof date === 'string' ? new Date(date) : date;
  const today = nowThai();
  return d.toDateString() === today.toDateString();
}
```

### Thai Mobile Number Validation

```typescript
// utils/thai-mobile.ts
/**
 * Thai mobile number validation and formatting
 * Valid prefixes: 08, 09, 06, 061, 062, 063, 064, 065
 */

export type MobileProvider = 'AIS' | 'DTAC' | 'TRUE' | 'NT' | 'other';

// Valid Thai mobile prefixes
const VALID_PREFIXES = [
  '08', '09', '06',      // 2-digit prefixes
  '061', '062', '063', '064', '065', '081', '082', '083', '084', '085', '086', '087', '088', '089',
  '091', '092', '093', '094', '095', '096', '097', '098', '099'
];

// Provider detection (basic, may change)
const PROVIDER_RANGES: Record<string, MobileProvider> = {
  '08': 'AIS', '09': 'AIS',
  '061': 'TRUE', '062': 'TRUE', '063': 'TRUE', '064': 'TRUE', '065': 'TRUE',
  '081': 'AIS', '082': 'AIS', '083': 'AIS', '084': 'TRUE', '085': 'TRUE', '086': 'AIS', '087': 'AIS', '088': 'TRUE', '089': 'TRUE',
  '091': 'DTAC', '092': 'DTAC', '093': 'DTAC', '094': 'DTAC', '095': 'DTAC', '096': 'DTAC', '097': 'DTAC', '098': 'DTAC', '099': 'DTAC'
};

// Clean mobile number (remove spaces, dashes, +66)
export function cleanMobileNumber(mobile: string): string {
  return mobile
    .replace(/\s+/g, '')
    .replace(/-/g, '')
    .replace(/^\+66/, '0')
    .trim();
}

// Validate Thai mobile number
export function isValidThaiMobile(mobile: string): boolean {
  const cleaned = cleanMobileNumber(mobile);

  // Must be 10 digits, start with 0
  if (!/^0\d{8}$/.test(cleaned)) {
    return false;
  }

  // Check valid prefix
  return VALID_PREFIXES.some(prefix => cleaned.startsWith(prefix));
}

// Format mobile number for display
export function formatMobileNumber(mobile: string): string {
  const cleaned = cleanMobileNumber(mobile);
  if (cleaned.length !== 10) return cleaned;

  // Format: 081-234-5678
  return `${cleaned.slice(0, 3)}-${cleaned.slice(3, 6)}-${cleaned.slice(6)}`;
}

// Convert to international format (+66)
export function toInternationalFormat(mobile: string): string {
  const cleaned = cleanMobileNumber(mobile);
  return cleaned.startsWith('0') ? '+66' + cleaned.slice(1) : cleaned;
}

// Detect mobile provider
export function detectProvider(mobile: string): MobileProvider {
  const cleaned = cleanMobileNumber(mobile);

  for (const [prefix, provider] of Object.entries(PROVIDER_RANGES)) {
    if (cleaned.startsWith(prefix)) {
      return provider;
    }
  }

  return 'other';
}

// Zod schema for Thai mobile
import { z } from 'zod';

export const thaiMobileSchema = z.string()
  .transform(cleanMobileNumber)
  .refine(isValidThaiMobile, {
    message: 'เบอร์โทรศัพท์มือถือไม่ถูกต้อง (ต้องเป็น 10 หลัก เริ่มต้นด้วย 08, 09, 06)'
  });
```

### Thai Address Components

```typescript
// types/thai-address.ts
/**
 * Thai address structure for geocoding
 */

export interface ThaiAddress {
  // Address components
  houseNumber?: string;      // บ้านเลขที่
  villageNumber?: string;    // หมู่ที่
  village?: string;          // หมู่บ้าน
  building?: string;         // อาคาร
  floor?: string;            // ชั้น
  room?: string;             // ห้อง
  alley?: string;            // ซอย
  road?: string;             // ถนน

  // Administrative divisions
  subdistrict?: string;      // ตำบล / แขวง
  district?: string;         // อำเภอ / เขต
  province?: string;         // จังหวัด
  postalCode?: string;       // รหัสไปรษณีย์

  // Coordinates
  lat?: number;
  lng?: number;
}

// Format Thai address for display
export function formatThaiAddress(address: ThaiAddress): string {
  const parts: string[] = [];

  if (address.houseNumber) parts.push(`${address.houseNumber}`);
  if (address.villageNumber) parts.push(`ม.${address.villageNumber}`);
  if (address.village) parts.push(address.village);
  if (address.building) parts.push(address.building);
  if (address.floor) parts.push(`ชั้น ${address.floor}`);
  if (address.room) parts.push(`ห้อง ${address.room}`);
  if (address.alley) parts.push(`ซ.${address.alley}`);
  if (address.road) parts.push(`ถ.${address.road}`);

  // Administrative
  if (address.subdistrict) parts.push(`ต.${address.subdistrict}`);
  if (address.district) parts.push(`อ.${address.district}`);
  if (address.province) parts.push(`จ.${address.province}`);
  if (address.postalCode) parts.push(address.postalCode);

  return parts.join(' ');
}

// Parse address string into components (basic)
export function parseThaiAddress(addressStr: string): Partial<ThaiAddress> {
  const result: Partial<ThaiAddress> = {};

  // Common patterns
  const patterns = {
    houseNumber: /บ้านเลขที่\s*(\S+)/,
    villageNumber: /ม\.?\s*(\d+)/,
    alley: /ซ\.?\s*([^,\s]+)/,
    road: /ถ\.?\s*([^,\s]+)/,
    subdistrict: /ต\.?\s*([^,\s]+)/,
    district: /อ\.?\s*([^,\s]+)/,
    province: /จ\.?\s*([^,\s]+)/,
    postalCode: /\b(\d{5})\b$/
  };

  for (const [key, pattern] of Object.entries(patterns)) {
    const match = addressStr.match(pattern);
    if (match) {
      result[key as keyof ThaiAddress] = match[1];
    }
  }

  return result;
}
```

### Google Maps Thailand Geocoding

```typescript
// services/thai-geocoding.ts
/**
 * Geocoding service optimized for Thailand addresses
 * Uses Google Maps Geocoding API
 */

interface GeocodeResult {
  formattedAddress: string;
  lat: number;
  lng: number;
  placeId?: string;
  components: ThaiAddress;
}

export async function geocodeThaiAddress(
  address: string,
  apiKey: string
): Promise<GeocodeResult | null> {
  const url = new URL('https://maps.googleapis.com/maps/api/geocode/json');
  url.searchParams.set('address', `${address}, Thailand`);
  url.searchParams.set('key', apiKey);
  url.searchParams.set('language', 'th');

  const response = await fetch(url.toString());
  const data = await response.json();

  if (data.status !== 'OK' || !data.results?.[0]) {
    return null;
  }

  const result = data.results[0];
  return {
    formattedAddress: result.formatted_address,
    lat: result.geometry.location.lat,
    lng: result.geometry.location.lng,
    placeId: result.place_id,
    components: parseGoogleComponents(result.address_components)
  };
}

export async function reverseGeocode(
  lat: number,
  lng: number,
  apiKey: string
): Promise<GeocodeResult | null> {
  const url = new URL('https://maps.googleapis.com/maps/api/geocode/json');
  url.searchParams.set('latlng', `${lat},${lng}`);
  url.searchParams.set('key', apiKey);
  url.searchParams.set('language', 'th');

  const response = await fetch(url.toString());
  const data = await response.json();

  if (data.status !== 'OK' || !data.results?.[0]) {
    return null;
  }

  const result = data.results[0];
  return {
    formattedAddress: result.formatted_address,
    lat: result.geometry.location.lat,
    lng: result.geometry.location.lng,
    placeId: result.place_id,
    components: parseGoogleComponents(result.address_components)
  };
}

function parseGoogleComponents(components: any[]): ThaiAddress {
  const result: ThaiAddress = {};
  const types: Record<string, string[]> = {
    subdistrict: ['sublocality_level_2', 'sublocality'],
    district: ['sublocality_level_1', 'administrative_area_level_2'],
    province: ['administrative_area_level_1'],
    postalCode: ['postal_code'],
    road: ['route']
  };

  for (const component of components) {
    for (const [key, typeList] of Object.entries(types)) {
      if (component.types.some((t: string) => typeList.includes(t))) {
        result[key as keyof ThaiAddress] = component.long_name;
      }
    }
  }

  return result;
}
```

### Haversine Distance (Thailand Coordinates)

```typescript
// utils/distance.ts
/**
 * Calculate distance between two coordinates
 * Uses Haversine formula for accurate results
 */

export function haversineDistance(
  lat1: number,
  lng1: number,
  lat2: number,
  lng2: number
): number {
  const R = 6371000; // Earth's radius in meters
  const φ1 = (lat1 * Math.PI) / 180;
  const φ2 = (lat2 * Math.PI) / 180;
  const Δφ = ((lat2 - lat1) * Math.PI) / 180;
  const Δλ = ((lng2 - lng1) * Math.PI) / 180;

  const a =
    Math.sin(Δφ / 2) * Math.sin(Δφ / 2) +
    Math.cos(φ1) * Math.cos(φ2) * Math.sin(Δλ / 2) * Math.sin(Δλ / 2);

  const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));

  return R * c; // Distance in meters
}

// Check if point is within radius
export function isWithinRadius(
  centerLat: number,
  centerLng: number,
  pointLat: number,
  pointLng: number,
  radiusMeters: number
): boolean {
  return haversineDistance(centerLat, centerLng, pointLat, pointLng) <= radiusMeters;
}

// Format distance for display
export function formatDistance(meters: number): string {
  if (meters < 1000) {
    return `${Math.round(meters)} ม.`;
  }
  return `${(meters / 1000).toFixed(1)} กม.`;
}
```

## Tailwind CSS Patterns

### Project Setup & Configuration

```javascript
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    './index.html',
    './src/**/*.{vue,js,ts,jsx,tsx}',
    './PTGLG/driverconnect/**/*.html',
    './PTGLG/driverconnect/**/*.js'
  ],
  darkMode: 'class', // or 'media' for system preference
  theme: {
    extend: {
      colors: {
        // LINE brand colors
        line: {
          green: '#00B900',
          'green-dark': '#009100',
          'green-light': '#00FF00'
        },
        // Status colors
        status: {
          pending: '#F59E0B',
          active: '#3B82F6',
          completed: '#22C55E',
          cancelled: '#EF4444'
        }
      },
      fontFamily: {
        sans: ['Sarabun', 'system-ui', 'sans-serif'],
        thai: ['Sarabun', 'sans-serif']
      },
      spacing: {
        'safe-top': 'env(safe-area-inset-top)',
        'safe-bottom': 'env(safe-area-inset-bottom)',
        'safe-left': 'env(safe-area-inset-left)',
        'safe-right': 'env(safe-area-inset-right)'
      },
      screens: {
        // LINE LIFF view sizes
        'liff-compact': {'max': '360px'},
        'liff-tall': {'min': '361px', 'max': '600px'},
        'liff-full': {'min': '601px'}
      }
    }
  },
  plugins: [
    require('@tailwindcss/forms'),
    require('@tailwindcss/typography'),
    require('@tailwindcss/aspect-ratio')
  ]
}
```

### Responsive Design Patterns

```html
<!-- Mobile-first responsive card -->
<div class="p-4 sm:p-6 md:p-8">
  <!-- Stack on mobile, side-by-side on tablet+ -->
  <div class="flex flex-col md:flex-row gap-4">
    <!-- Content takes full width on mobile, half on tablet -->
    <div class="w-full md:w-1/2">
      <h2 class="text-lg sm:text-xl md:text-2xl font-bold">
        ขนาดตัวอักษรปรับตามหน้าจอ
      </h2>
    </div>
  </div>
</div>

<!-- Grid that adapts -->
<div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4">
  <!-- Cards -->
</div>

<!-- Hide on mobile, show on desktop -->
<div class="hidden md:block">
  Desktop-only content
</div>

<!-- Show on mobile only -->
<div class="md:hidden">
  Mobile-only content
</div>
```

### Dark Mode Implementation

```html
<!-- Using class-based dark mode -->
<html class="dark">
  <!-- Your content -->
</html>

<!-- Components with dark mode variants -->
<button class="bg-white dark:bg-gray-800 text-gray-900 dark:text-white px-4 py-2 rounded">
  ปุ่มที่รองรับ Dark Mode
</button>

<!-- Dark mode with custom colors -->
<div class="bg-white dark:bg-slate-900 border-gray-200 dark:border-slate-700">
  <p class="text-gray-900 dark:text-slate-100">
    เนื้อหาที่ปรับสีตามธีม
  </p>
</div>
```

```javascript
// Toggle dark mode
function toggleDarkMode() {
  document.documentElement.classList.toggle('dark');
  localStorage.setItem('darkMode', document.documentElement.classList.contains('dark'));
}

// Initialize dark mode from localStorage or system preference
function initDarkMode() {
  const stored = localStorage.getItem('darkMode');
  if (stored !== null) {
    if (stored === 'true') {
      document.documentElement.classList.add('dark');
    }
  } else if (window.matchMedia('(prefers-color-scheme: dark)').matches) {
    document.documentElement.classList.add('dark');
  }
}

initDarkMode();
```

### Custom Component Patterns

```html
<!-- Button Component -->
<button class="inline-flex items-center justify-center gap-2 px-4 py-2
                   bg-line-green hover:bg-line-green-dark text-white
                   font-medium rounded-lg transition-colors
                   disabled:opacity-50 disabled:cursor-not-allowed
                   focus:outline-none focus:ring-2 focus:ring-line-green focus:ring-offset-2">
  <span>บันทึก</span>
</button>

<!-- Button Variants -->
<button class="btn btn-primary">Primary</button>
<button class="btn btn-secondary">Secondary</button>
<button class="btn btn-danger">Danger</button>

<style>
  /* Define button variants with @apply */
  @layer components {
    .btn {
      @apply inline-flex items-center justify-center gap-2 px-4 py-2
             font-medium rounded-lg transition-colors
             disabled:opacity-50 disabled:cursor-not-allowed
             focus:outline-none focus:ring-2 focus:ring-offset-2;
    }

    .btn-primary {
      @apply bg-line-green hover:bg-line-green-dark text-white
             focus:ring-line-green;
    }

    .btn-secondary {
      @apply bg-gray-200 hover:bg-gray-300 text-gray-900
             focus:ring-gray-500;
    }

    .btn-danger {
      @apply bg-red-500 hover:bg-red-600 text-white
             focus:ring-red-500;
    }
  }
</style>
```

```html
<!-- Card Component -->
<div class="bg-white dark:bg-slate-800 rounded-xl shadow-sm
            border border-gray-200 dark:border-slate-700
            overflow-hidden">
  <!-- Card Header -->
  <div class="px-4 py-3 border-b border-gray-200 dark:border-slate-700
                  bg-gray-50 dark:bg-slate-900/50">
    <h3 class="font-semibold text-gray-900 dark:text-white">
      หัวข้อการ์ด
    </h3>
  </div>

  <!-- Card Body -->
  <div class="p-4">
    <p class="text-gray-700 dark:text-slate-300">
      เนื้อหาในการ์ด
    </p>
  </div>

  <!-- Card Footer -->
  <div class="px-4 py-3 border-t border-gray-200 dark:border-slate-700
                  bg-gray-50 dark:bg-slate-900/50 flex justify-end gap-2">
    <button class="btn btn-secondary text-sm">ยกเลิก</button>
    <button class="btn btn-primary text-sm">ยืนยัน</button>
  </div>
</div>
```

```html
<!-- Input Component -->
<div class="relative">
  <label class="block text-sm font-medium text-gray-700 dark:text-slate-300 mb-1">
    ชื่อ
  </label>
  <input type="text"
         class="w-full px-3 py-2 rounded-lg border
                border-gray-300 dark:border-slate-600
                bg-white dark:bg-slate-800
                text-gray-900 dark:text-white
                placeholder-gray-400 dark:placeholder-slate-500
                focus:outline-none focus:ring-2 focus:ring-line-green focus:border-transparent
                disabled:bg-gray-100 dark:disabled:bg-slate-900
                disabled:cursor-not-allowed"
         placeholder="กรอกชื่อ">
  <p class="mt-1 text-sm text-red-500 hidden" id="name-error">
    กรุณากรอกชื่อ
  </p>
</div>
```

### Utility-First Best Practices

```html
<!-- Use @layer to extend Tailwind -->
<style>
  @layer components {
    /* Reusable component classes */
    .card {
      @apply bg-white dark:bg-slate-800 rounded-xl shadow-sm
             border border-gray-200 dark:border-slate-700 p-4;
    }

    .input {
      @apply w-full px-3 py-2 rounded-lg border
             border-gray-300 dark:border-slate-600
             bg-white dark:bg-slate-800
             text-gray-900 dark:text-white
             focus:outline-none focus:ring-2 focus:ring-line-green;
    }
  }

  @layer utilities {
    /* Custom utility classes */
    .text-balance {
      text-wrap: balance;
    }

    .scrollbar-hide {
      -ms-overflow-style: none;
      scrollbar-width: none;
    }

    .scrollbar-hide::-webkit-scrollbar {
      display: none;
    }

    .safe-area-inset {
      padding-top: env(safe-area-inset-top);
      padding-bottom: env(safe-area-inset-bottom);
      padding-left: env(safe-area-inset-left);
      padding-right: env(safe-area-inset-right);
    }
  }
</style>
```

### JIT / Production Build

```javascript
// postcss.config.js
export default {
  plugins: {
    '@tailwindcss/postcss': {},
    autoprefixer: {}
  }
}

// package.json scripts
{
  "scripts": {
    "dev": "concurrently \"npm run dev:css\" \"vite\"",
    "dev:css": "tailwindcss -i ./src/css/input.css -o ./src/css/output.css --watch",
    "build:css": "tailwindcss -i ./src/css/input.css -o ./dist/css/output.css --minify"
  }
}
```

### Integrating with Existing CSS

```html
<!-- Use Tailwind prefixes to avoid conflicts -->
<div class="tw-flex tw-px-4 tw-py-2">
  Content with prefixed classes
</div>
```

```javascript
// tailwind.config.js - Add prefix
export default {
  prefix: 'tw-',
  // ... rest of config
}
```

### LINE LIFF Specific Patterns

```html
<!-- LIFF-aware container -->
<div class="min-h-screen bg-gray-50 dark:bg-slate-900
            safe-area-inset">
  <!-- Fixed header for LIFF -->
  <header class="fixed top-0 left-0 right-0 z-50
                  bg-white dark:bg-slate-800
                  border-b border-gray-200 dark:border-slate-700
                  safe-top">
    <div class="flex items-center justify-between px-4 py-3">
      <h1 class="text-lg font-semibold text-gray-900 dark:text-white">
        DriverConnect
      </h1>
      <button class="p-2 rounded-lg hover:bg-gray-100 dark:hover:bg-slate-700">
        <!-- Menu icon -->
      </button>
    </div>
  </header>

  <!-- Main content with padding for header -->
  <main class="pt-14 pb-20 px-4">
    <!-- Scrollable content -->
  </main>

  <!-- Fixed bottom navigation -->
  <nav class="fixed bottom-0 left-0 right-0 z-50
               bg-white dark:bg-slate-800
               border-t border-gray-200 dark:border-slate-700
               safe-bottom">
    <div class="flex justify-around py-2">
      <a href="#" class="flex flex-col items-center p-2 text-line-green">
        <!-- Icon -->
        <span class="text-xs mt-1">หน้าแรก</span>
      </a>
      <a href="#" class="flex flex-col items-center p-2 text-gray-500 dark:text-slate-400">
        <!-- Icon -->
        <span class="text-xs mt-1">งานของฉัน</span>
      </a>
      <a href="#" class="flex flex-col items-center p-2 text-gray-500 dark:text-slate-400">
        <!-- Icon -->
        <span class="text-xs mt-1">โปรไฟล์</span>
      </a>
    </div>
  </nav>
</div>
```

### Admin Dashboard Patterns

```html
<!-- Admin dashboard layout -->
<div class="flex h-screen bg-gray-100 dark:bg-slate-900">
  <!-- Sidebar -->
  <aside class="hidden md:flex md:w-64 md:flex-col
                  bg-white dark:bg-slate-800
                  border-r border-gray-200 dark:border-slate-700">
    <div class="p-4 border-b border-gray-200 dark:border-slate-700">
      <h1 class="text-xl font-bold text-line-green">DriverConnect</h1>
      <p class="text-sm text-gray-500 dark:text-slate-400">Admin Panel</p>
    </div>

    <nav class="flex-1 p-4 space-y-1 overflow-y-auto">
      <a href="#" class="flex items-center gap-3 px-3 py-2
                       rounded-lg bg-line-green/10 text-line-green
                       font-medium">
        <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                d="M3 12l2-2m0 0l7-7 7 7M5 10v10a1 1 0 001 1h3m10-11l2 2m-2-2v10a1 1 0 01-1 1h-3m-6 0a1 1 0 001-1v-4a1 1 0 011-1h2a1 1 0 011 1v4a1 1 0 001 1m-6 0h6"/>
        </svg>
        แดชบอร์ด
      </a>
      <a href="#" class="flex items-center gap-3 px-3 py-2
                       rounded-lg text-gray-700 dark:text-slate-300
                       hover:bg-gray-100 dark:hover:bg-slate-700">
        <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2"/>
        </svg>
        จัดการงาน
      </a>
      <a href="#" class="flex items-center gap-3 px-3 py-2
                       rounded-lg text-gray-700 dark:text-slate-300
                       hover:bg-gray-100 dark:hover:bg-slate-700">
        <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                d="M12 4.354a4 4 0 110 5.292M15 21H3v-1a6 6 0 0112 0v1zm0 0h6v-1a6 6 0 00-9-5.197M13 7a4 4 0 11-8 0 4 4 0 018 0z"/>
        </svg>
        พนักงานขับรถ
      </a>
    </nav>
  </aside>

  <!-- Main content -->
  <main class="flex-1 overflow-auto">
    <!-- Top bar -->
    <header class="bg-white dark:bg-slate-800
                    border-b border-gray-200 dark:border-slate-700
                    px-4 py-3 flex items-center justify-between">
      <h2 class="text-lg font-semibold text-gray-900 dark:text-white">
        แดชบอร์ด
      </h2>

      <div class="flex items-center gap-4">
        <!-- Dark mode toggle -->
        <button onclick="toggleDarkMode()"
                class="p-2 rounded-lg hover:bg-gray-100 dark:hover:bg-slate-700
                       text-gray-600 dark:text-slate-400">
          <!-- Sun/Moon icon -->
        </button>
      </div>
    </header>

    <!-- Dashboard content -->
    <div class="p-4 sm:p-6 lg:p-8">
      <!-- Stats grid -->
      <div class="grid grid-cols-2 lg:grid-cols-4 gap-4 mb-6">
        <!-- Stat cards -->
      </div>
    </div>
  </main>
</div>
```

### Status & Badge Components

```html
<!-- Status badges -->
<span class="inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium
               bg-yellow-100 text-yellow-800 dark:bg-yellow-900/30 dark:text-yellow-400">
  รอดำเนินการ
</span>

<span class="inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium
               bg-blue-100 text-blue-800 dark:bg-blue-900/30 dark:text-blue-400">
  กำลังดำเนินการ
</span>

<span class="inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium
               bg-green-100 text-green-800 dark:bg-green-900/30 dark:text-green-400">
  เสร็จสิ้น
</span>

<span class="inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium
               bg-red-100 text-red-800 dark:bg-red-900/30 dark:text-red-400">
  ยกเลิก
</span>

<!-- Status badge component style -->
<style>
  @layer components {
    .badge {
      @apply inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium;
    }

    .badge-pending {
      @apply bg-yellow-100 text-yellow-800 dark:bg-yellow-900/30 dark:text-yellow-400;
    }

    .badge-active {
      @apply bg-blue-100 text-blue-800 dark:bg-blue-900/30 dark:text-blue-400;
    }

    .badge-completed {
      @apply bg-green-100 text-green-800 dark:bg-green-900/30 dark:text-green-400;
    }

    .badge-cancelled {
      @apply bg-red-100 text-red-800 dark:bg-red-900/30 dark:text-red-400;
    }
  }
</style>
```

## LINE OA Advanced Features

### Rich Menu Management

```typescript
// services/line-rich-menu.ts
/**
 * LINE Rich Menu management
 * Create, update, and delete rich menus via LINE Messaging API
 */

interface RichMenuItem {
  type: 'message' | 'uri' | 'datetimepicker' | 'postback';
  label: string;
  data?: string;
  uri?: string;
  area: {
    x: number;
    y: number;
    width: number;
    height: number;
  };
}

interface RichMenuConfig {
  name: string;
  size: 'richmenu' | 'richmenu-album'; // 2500x1686 or 800x250
  chatBarText: string;
  items: RichMenuItem[];
}

export class LineRichMenu {
  constructor(
    private accessToken: string,
    private apiBase = 'https://api.line.me/v2/bot'
  ) {}

  // Create rich menu
  async createRichMenu(config: RichMenuConfig): Promise<string> {
    const [width, height] = config.size === 'richmenu' ? [2500, 1686] : [800, 2500];

    const richMenu = {
      size: { width, height },
      selected: false,
      name: config.name,
      chatBarText: config.chatBarText,
      areas: config.items.map(item => ({
        bounds: item.area,
        action: {
          type: item.type,
          label: item.label,
          data: item.data,
          uri: item.uri
        }
      }))
    };

    const response = await fetch(`${this.apiBase}/richmenu`, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${this.accessToken}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(richMenu)
    });

    const data = await response.json();
    return data.richMenuId;
  }

  // Upload rich menu image
  async uploadRichMenuImage(
    richMenuId: string,
    imageBuffer: Buffer,
    contentType = 'image/jpeg'
  ): Promise<void> {
    await fetch(`${this.apiBase}/richmenu/${richMenuId}/content`, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${this.accessToken}`,
        'Content-Type': contentType
      },
      body: imageBuffer
    });
  }

  // Set rich menu to user
  async setRichMenuToUser(userId: string, richMenuId: string): Promise<void> {
    await fetch(`${this.apiBase}/user/${userId}/richmenu/${richMenuId}`, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${this.accessToken}`
      }
    });
  }

  // Set rich menu to all users
  async setRichMenuToAllUsers(richMenuId: string): Promise<void> {
    await fetch(`${this.apiBase}/richmenu/${richMenuId}/users`, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${this.accessToken}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ userId: 'all' }) // Note: LINE may use different endpoint
    });
  }

  // Delete rich menu
  async deleteRichMenu(richMenuId: string): Promise<void> {
    await fetch(`${this.apiBase}/richmenu/${richMenuId}`, {
      method: 'DELETE',
      headers: {
        'Authorization': `Bearer ${this.accessToken}`
      }
    });
  }

  // Get user's current rich menu
  async getUserRichMenu(userId: string): Promise<string | null> {
    const response = await fetch(`${this.apiBase}/user/${userId}/richmenu`, {
      headers: {
        'Authorization': `Bearer ${this.accessToken}`
      }
    });

    if (response.status === 404) return null;

    const data = await response.json();
    return data.richMenuId;
  }

  // Unlink rich menu from user
  async unlinkRichMenu(userId: string): Promise<void> {
    await fetch(`${this.apiBase}/user/${userId}/richmenu`, {
      method: 'DELETE',
      headers: {
        'Authorization': `Bearer ${this.accessToken}`
      }
    });
  }

  // List all rich menus
  async listRichMenus(): Promise<any[]> {
    const response = await fetch(`${this.apiBase}/richmenu/list`, {
      headers: {
        'Authorization': `Bearer ${this.accessToken}`
      }
    });

    const data = await response.json();
    return data.richmenus || [];
  }
}
```

### Flex Message Templates

```typescript
// services/line-flex.ts
/**
 * LINE Flex Message templates
 * Various formats for different use cases
 */

// Basic card with image and buttons
export function createCardFlexMessage({
  title,
  description,
  imageUrl,
  buttons
}: {
  title: string;
  description?: string;
  imageUrl?: string;
  buttons: Array<{ label: string; data: string; uri?: string }>;
}) {
  return {
    type: 'flex',
    altText: title,
    contents: {
      type: 'bubble',
      hero: imageUrl ? {
        type: 'image',
        url: imageUrl,
        size: 'full',
        aspectRatio: '20:13',
        aspectMode: 'cover'
      } : undefined,
      body: {
        type: 'box',
        layout: 'vertical',
        contents: [
          {
            type: 'text',
            text: title,
            weight: 'bold',
            size: 'xl',
            wrap: true
          },
          ...(description ? [{
            type: 'text',
            text: description,
            size: 'sm',
            color: '#666666',
            wrap: true,
            margin: 'md'
          }] : [])
        ]
      },
      footer: buttons.length > 0 ? {
        type: 'box',
        layout: 'vertical',
        spacing: 'sm',
        contents: buttons.map(btn => ({
          type: 'button',
          action: {
            type: btn.uri ? 'uri' : 'message',
            label: btn.label,
            uri: btn.uri,
            text: btn.data
          },
            style: 'primary'
        }))
      } : undefined
    }
  };
}

// Job card for DriverConnect
export function createJobCardFlexMessage(job: {
  ref: string;
  origin: string;
  destination: string;
  status: string;
  actionData: string;
}) {
  const statusColors: Record<string, string> = {
    pending: '#FFAA00',
    active: '#00B900',
    completed: '#0066FF',
    cancelled: '#FF0000'
  };

  return {
    type: 'flex',
    altText: `งาน ${job.ref}`,
    contents: {
      type: 'bubble',
      header: {
        type: 'box',
        layout: 'vertical',
        contents: [
          {
            type: 'text',
            text: `งานที่ ${job.ref}`,
            color: '#FFFFFF',
            size: 'md',
            align: 'start',
            weight: 'bold'
          },
          {
            type: 'text',
            text: job.status === 'pending' ? 'รอดำเนินการ' :
                  job.status === 'active' ? 'กำลังดำเนินการ' :
                  job.status === 'completed' ? 'เสร็จสิ้น' : 'ยกเลิก',
            color: '#FFFFFF',
            size: 'sm',
            align: 'end',
            margin: 'sm'
          }
        ],
        backgroundColor: statusColors[job.status] || '#00B900',
        paddingAll: 'md',
        spacing: 'sm'
      },
      body: {
        type: 'box',
        layout: 'vertical',
        contents: [
          {
            type: 'box',
            layout: 'baseline',
            contents: [
              {
                type: 'text',
                text: 'จุดเริ่มต้น',
                size: 'sm',
                color: '#666666'
              },
              {
                type: 'text',
                text: job.origin,
                size: 'sm',
                weight: 'bold',
                margin: 'sm'
              }
            ],
            margin: 'lg'
          },
          {
            type: 'box',
            layout: 'baseline',
            contents: [
              {
                type: 'text',
                text: 'ปลายทาง',
                size: 'sm',
                color: '#666666'
              },
              {
                type: 'text',
                text: job.destination,
                size: 'sm',
                weight: 'bold',
                margin: 'sm'
              }
            ],
            margin: 'lg'
          }
        ],
        paddingAll: 'lg'
      },
      footer: {
        type: 'box',
        layout: 'vertical',
        contents: [
          {
            type: 'button',
            action: {
              type: 'message',
              label: 'ดูรายละเอียด',
              text: job.actionData
            },
            style: 'primary'
          }
        ]
      }
    }
  };
}

// List of jobs (carousel)
export function createJobCarouselFlexMessage(jobs: Array<{
  ref: string;
  origin: string;
  destination: string;
  status: string;
  actionData: string;
}>) {
  return {
    type: 'flex',
    altText: 'รายการงาน',
    contents: {
      type: 'carousel',
      contents: jobs.map(job => {
        const card = createJobCardFlexMessage(job);
        return card.contents;
      })
    }
  };
}

// Quick reply buttons
export function createQuickReplyMessage(
  text: string,
  items: Array<{ label: string; data: string; uri?: string }>
) {
  return {
    type: 'text',
    text,
    quickReply: {
      items: items.map(item => ({
        type: 'action',
        action: {
          type: item.uri ? 'uri' : 'message',
          label: item.label,
          uri: item.uri,
          text: item.data
        }
      }))
    }
  };
}
```

### Quick Reply & Message Templates

```typescript
// services/line-templates.ts
/**
 * LINE Message templates with Quick Reply and Buttons
 */

// Button template (deprecated but still useful)
export function createButtonMessage({
  title,
  text,
  thumbnailImageUrl,
  buttons
}: {
  title: string;
  text: string;
  thumbnailImageUrl?: string;
  buttons: Array<{ label: string; data: string; uri?: string }>;
}) {
  return {
    type: 'template',
    altText: title,
    template: {
      type: 'buttons',
      thumbnailImageUrl,
      title,
      text,
      actions: buttons.map(btn => ({
        type: btn.uri ? 'uri' : 'message',
        label: btn.label,
        uri: btn.uri,
        text: btn.data
      }))
    }
  };
}

// Confirm template
export function createConfirmMessage({
  text,
  okText,
  okData,
  cancelText = 'ยกเลิก',
  cancelData = 'cancel'
}: {
  text: string;
  okText: string;
  okData: string;
  cancelText?: string;
  cancelData?: string;
}) {
  return {
    type: 'template',
    altText: text,
    template: {
      type: 'confirm',
      text,
      actions: [
        {
          type: 'message',
          label: okText,
          text: okData
        },
        {
          type: 'message',
          label: cancelText,
          text: cancelData
        }
      ]
    }
  };
}

// Date/time picker
export function createDatetimePickerMessage({
  text,
  data,
  min?: string,
  max?: string,
  mode = 'datetime'
}: {
  text: string;
  data: string;
  min?: string;
  max?: string;
  mode?: 'date' | 'time' | 'datetime';
}) {
  return {
    type: 'text',
    text,
    quickReply: {
      items: [
        {
          type: 'action',
          action: {
            type: 'datetimepicker',
            label: 'เลือกวันที่/เวลา',
            data,
            mode,
            min,
            max
          }
        }
      ]
    }
  };
}
```

### LINE Notify Integration

```typescript
// services/line-notify.ts
/**
 * LINE Notify integration
 * Send notifications to LINE via LINE Notify API
 */

const LINE_NOTIFY_API = 'https://notify-api.line.me/api/notify';

export interface LineNotifyConfig {
  accessToken: string;
}

export class LineNotify {
  constructor(private config: LineNotifyConfig) {}

  // Send text message
  async sendMessage(message: string): Promise<boolean> {
    const response = await fetch(LINE_NOTIFY_API, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${this.config.accessToken}`,
        'Content-Type': 'application/x-www-form-urlencoded'
      },
      body: new URLSearchParams({ message })
    });

    return response.ok;
  }

  // Send message with image
  async sendMessageWithImage(
    message: string,
    image: { url?: string; file?: File }
  ): Promise<boolean> {
    const body = new FormData();
    body.append('message', message);

    if (image.url) {
      body.append('imageThumbnail', image.url);
      body.append('imageFullsize', image.url);
    } else if (image.file) {
      body.append('imageFile', image.file);
    }

    const response = await fetch(LINE_NOTIFY_API, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${this.config.accessToken}`
      },
      body
    });

    return response.ok;
  }

  // Send sticker message
  async sendSticker(
    message: string,
    stickerId: string,
    packageId: string = '1'
  ): Promise<boolean> {
    const response = await fetch(LINE_NOTIFY_API, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${this.config.accessToken}`,
        'Content-Type': 'application/x-www-form-urlencoded'
      },
      body: new URLSearchParams({
        message,
        stickerId,
        packageId
      })
    });

    return response.ok;
  }

  // Check rate limit (from response headers)
  getRateLimit(response: Response): { remaining: number; reset: number } {
    return {
      remaining: parseInt(response.headers.get('X-RateLimit-Remaining') || '0'),
      reset: parseInt(response.headers.get('X-RateLimit-Reset') || '0')
    };
  }
}
```

## Supabase Edge Functions Patterns

### Authentication Flows

```typescript
// supabase/functions/auth-middleware/index.ts
/**
 * Authentication middleware for Supabase Edge Functions
 * Verify user session and role
 */

import { serve } from 'https://deno.land/std@0.168.0/http/server.ts';
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2';

const supabaseUrl = Deno.env.get('SUPABASE_URL')!;
const supabaseServiceKey = Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!;

interface AuthenticatedRequest extends Request {
  user?: {
    id: string;
    email?: string;
    role?: string;
  };
}

// Auth middleware
async function withAuth(
  req: AuthenticatedRequest,
  handler: (req: AuthenticatedRequest) => Promise<Response>
): Promise<Response> {
  try {
    // Extract token from Authorization header
    const authHeader = req.headers.get('Authorization');
    if (!authHeader?.startsWith('Bearer ')) {
      return new Response('Missing authorization header', { status: 401 });
    }

    const token = authHeader.substring(7);

    // Verify token with Supabase
    const supabase = createClient(supabaseUrl, supabaseServiceKey);
    const { data: { user }, error } = await supabase.auth.getUser(token);

    if (error || !user) {
      return new Response('Invalid token', { status: 401 });
    }

    // Attach user to request
    req.user = {
      id: user.id,
      email: user.email,
      role: user.user_metadata?.role || 'user'
    };

    return await handler(req);
  } catch (error) {
    console.error('Auth error:', error);
    return new Response('Authentication failed', { status: 500 });
  }
}

// Role check middleware
function requireRole(...roles: string[]) {
  return (req: AuthenticatedRequest): Response | null => {
    if (!req.user) {
      return new Response('Not authenticated', { status: 401 });
    }

    if (!roles.includes(req.user.role || '')) {
      return new Response('Insufficient permissions', { status: 403 });
    }

    return null; // Continue
  };
}

serve(async (req) => {
  // Handle CORS preflight
  if (req.method === 'OPTIONS') {
    return new Response(null, {
      headers: {
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Methods': 'POST, GET, OPTIONS',
        'Access-Control-Allow-Headers': 'authorization, content-type'
      }
    });
  }

  // Apply auth middleware
  return withAuth(req, async (req) => {
    // Check admin role
    const roleCheck = requireRole('admin')(req);
    if (roleCheck) return roleCheck;

    // Your protected logic here
    return new Response(JSON.stringify({
      message: 'Hello, ' + req.user?.email,
      userId: req.user?.id
    }), {
      headers: { 'Content-Type': 'application/json' }
    });
  });
});
```

### File Upload with Supabase Storage

```typescript
// supabase/functions/upload-file/index.ts
/**
 * File upload handler for Supabase Storage
 * Supports image upload with validation and transformation
 */

import { serve } from 'https://deno.land/std@0.168.0/http/server.ts';
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2';

const supabaseUrl = Deno.env.get('SUPABASE_URL')!;
const supabaseServiceKey = Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!;

// Allowed MIME types
const ALLOWED_TYPES = [
  'image/jpeg',
  'image/png',
  'image/webp',
  'image/gif',
  'application/pdf'
];

// Max file size (5MB)
const MAX_SIZE = 5 * 1024 * 1024;

serve(async (req) => {
  if (req.method === 'OPTIONS') {
    return new Response(null, {
      headers: {
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Methods': 'POST, OPTIONS',
        'Access-Control-Allow-Headers': 'authorization, content-type'
      }
    });
  }

  try {
    // Verify auth
    const authHeader = req.headers.get('Authorization');
    if (!authHeader?.startsWith('Bearer ')) {
      return new Response('Unauthorized', { status: 401 });
    }

    const token = authHeader.substring(7);
    const supabase = createClient(supabaseUrl, supabaseServiceKey);
    const { data: { user }, error } = await supabase.auth.getUser(token);

    if (error || !user) {
      return new Response('Invalid token', { status: 401 });
    }

    // Parse form data
    const formData = await req.formData();
    const file = formData.get('file') as File;
    const bucket = formData.get('bucket') as string || 'uploads';
    const folder = formData.get('folder') as string || '';

    if (!file) {
      return new Response('No file provided', { status: 400 });
    }

    // Validate file type
    if (!ALLOWED_TYPES.includes(file.type)) {
      return new Response('Invalid file type', { status: 400 });
    }

    // Validate file size
    if (file.size > MAX_SIZE) {
      return new Response('File too large (max 5MB)', { status: 400 });
    }

    // Generate unique filename
    const ext = file.name.split('.').pop();
    const filename = `${user.id}/${Date.now()}-${crypto.randomUUID()}.${ext}`;

    // Upload to Supabase Storage
    const { data: uploadData, error: uploadError } = await supabase
      .storage
      .from(bucket)
      .upload(filename, file, {
        contentType: file.type,
        upsert: false
      });

    if (uploadError) {
      console.error('Upload error:', uploadError);
      return new Response('Upload failed', { status: 500 });
    }

    // Get public URL
    const { data: { publicUrl } } = supabase
      .storage
      .from(bucket)
      .getPublicUrl(filename);

    return new Response(JSON.stringify({
      path: uploadData.path,
      publicUrl,
      size: file.size,
      type: file.type
    }), {
      headers: {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*'
      }
    });

  } catch (error) {
    console.error('Error:', error);
    return new Response('Internal server error', { status: 500 });
  }
});
```

### Cron Jobs with Edge Functions

```typescript
// supabase/functions/daily-summary/index.ts
/**
 * Scheduled task using Supabase Edge Functions
 * Run daily to generate and send reports
 */

import { serve } from 'https://deno.land/std@0.168.0/http/server.ts';
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2';

const supabaseUrl = Deno.env.get('SUPABASE_URL')!;
const supabaseServiceKey = Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!;
const LINE_NOTIFY_TOKEN = Deno.env.get('LINE_NOTIFY_TOKEN')!;

serve(async (req) => {
  // Verify cron secret (from Supabase)
  const cronKey = req.headers.get('Authorization');
  if (cronKey !== `Bearer ${Deno.env.get('CRON_SECRET')}`) {
    return new Response('Unauthorized', { status: 401 });
  }

  const supabase = createClient(supabaseUrl, supabaseServiceKey);

  // Get today's date in Thai timezone
  const now = new Date();
  const thaiDate = new Date(now.toLocaleString('en-US', { timeZone: 'Asia/Bangkok' }));
  const todayStart = new Date(thaiDate);
  todayStart.setHours(0, 0, 0, 0);

  // Fetch daily stats
  const { data: jobs } = await supabase
    .from('jobdata')
    .select('status, created_at')
    .gte('created_at', todayStart.toISOString());

  const stats = {
    total: jobs?.length || 0,
    completed: jobs?.filter(j => j.status === 'completed').length || 0,
    active: jobs?.filter(j => j.status === 'active').length || 0,
    pending: jobs?.filter(j => j.status === 'pending').length || 0
  };

  // Format message
  const message = `
📊 สรุปผลประจำวัน
📅 ${thaiDate.toLocaleDateString('th-TH', {
    weekday: 'long',
    year: 'numeric',
    month: 'long',
    day: 'numeric'
  })}

✅ งานเสร็จสิ้น: ${stats.completed} งาน
🚚 กำลังดำเนินการ: ${stats.active} งาน
⏳ รอดำเนินการ: ${stats.pending} งาน
📋 ทั้งหมด: ${stats.total} งาน
  `.trim();

  // Send LINE Notify
  await fetch('https://notify-api.line.me/api/notify', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${LINE_NOTIFY_TOKEN}`,
      'Content-Type': 'application/x-www-form-urlencoded'
    },
    body: new URLSearchParams({ message })
  });

  return new Response(JSON.stringify({ success: true, stats }), {
    headers: { 'Content-Type': 'application/json' }
  });
});
```

### Error Handling & Logging

```typescript
// utils/error-handler.ts
/**
 * Centralized error handling for Edge Functions
 */

export class AppError extends Error {
  constructor(
    message: string,
    public statusCode: number = 500,
    public code?: string
  ) {
    super(message);
    this.name = 'AppError';
  }
}

export class ValidationError extends AppError {
  constructor(message: string, public field?: string) {
    super(message, 400, 'VALIDATION_ERROR');
    this.name = 'ValidationError';
  }
}

export class NotFoundError extends AppError {
  constructor(resource: string) {
    super(`${resource} not found`, 404, 'NOT_FOUND');
    this.name = 'NotFoundError';
  }
}

export class UnauthorizedError extends AppError {
  constructor(message = 'Unauthorized') {
    super(message, 401, 'UNAUTHORIZED');
    this.name = 'UnauthorizedError';
  }
}

// Error response formatter
export function errorResponse(error: unknown): Response {
  console.error('Error:', error);

  if (error instanceof AppError) {
    return new Response(JSON.stringify({
      error: {
        message: error.message,
        code: error.code
      }
    }), {
      status: error.statusCode,
      headers: { 'Content-Type': 'application/json' }
    });
  }

  // Unknown error
  return new Response(JSON.stringify({
    error: {
      message: 'Internal server error',
      code: 'INTERNAL_ERROR'
    }
  }), {
    status: 500,
    headers: { 'Content-Type': 'application/json' }
  });
}

// Async handler wrapper
export function asyncHandler<T>(
  handler: () => Promise<T>
): Promise<T | Response> {
  return handler().catch(errorResponse);
}

// Log to Supabase (audit trail)
export async function logEvent(
  supabase: any,
  event: {
    user_id?: string;
    action: string;
    details?: Record<string, any>;
    status: 'success' | 'error';
  }
) {
  try {
    await supabase.from('audit_logs').insert({
      user_id: event.user_id,
      action: event.action,
      details: event.details,
      status: event.status,
      created_at: new Date().toISOString()
    });
  } catch (error) {
    console.error('Failed to log event:', error);
  }
}
```

## Data Validation

### Zod Schemas

```typescript
// schemas/validation.ts
/**
 * Zod schemas for TypeScript validation
 */

import { z } from 'https://deno.land/x/zod/mod.ts';

// Common schemas
export const nonEmptyString = z.string().min(1, 'Required field');
export const emailSchema = z.string().email('Invalid email format');
export const phoneSchema = z.string().regex(/^0\d{8,9}$/, 'Invalid phone number');
export const thaiMobileSchema = z.string().regex(/^0[689]\d{8}$/, 'Invalid Thai mobile number');

// UUID schema
export const uuidSchema = z.string().uuid('Invalid UUID format');

// Coordinates schema
export const coordinatesSchema = z.object({
  lat: z.number().min(-90).max(90),
  lng: z.number().min(-180).max(180)
});

// Pagination schema
export const paginationSchema = z.object({
  page: z.coerce.number().min(1).default(1),
  limit: z.coerce.number().min(1).max(100).default(20)
});

// Job schema (DriverConnect)
export const jobSchema = z.object({
  ref: nonEmptyString.max(50),
  origin_id: uuidSchema,
  destination_lat: z.number().min(-90).max(90),
  destination_lng: z.number().min(-180).max(180),
  destination_address: nonEmptyString,
  status: z.enum(['pending', 'active', 'completed', 'cancelled']),
  driver_id: uuidSchema.optional()
});

// Check-in schema
export const checkInSchema = z.object({
  job_id: uuidSchema,
  lat: z.number().min(-90).max(90),
  lng: z.number().min(-180).max(180),
  odometer: z.coerce.number().min(0).optional(),
  photo_url: z.string().url().optional(),
  type: z.enum(['checkin', 'checkout'])
});

// User profile schema
export const userProfileSchema = z.object({
  line_user_id: nonEmptyString,
  display_name: z.string().min(1).max(100),
  phone_number: thaiMobileSchema.optional(),
  vehicle_plate: z.string().min(1).max(20).optional(),
  vehicle_type: z.enum(['truck', 'trailer', 'tanker']).optional(),
  status: z.enum(['pending', 'approved', 'suspended', 'deleted']).default('pending')
});

// Login schema
export const loginSchema = z.object({
  email: emailSchema,
  password: z.string().min(8, 'Password must be at least 8 characters')
});

// Request validation middleware
export function validateBody<T>(schema: z.ZodSchema<T>) {
  return async (req: Request): Promise<{ data?: T; error?: string }> => {
    try {
      const body = await req.json();
      const data = schema.parse(body);
      return { data };
    } catch (error) {
      if (error instanceof z.ZodError) {
        const firstError = error.errors[0];
        return { error: `${firstError.path.join('.')}: ${firstError.message}` };
      }
      return { error: 'Validation failed' };
    }
  };
}
```

### Joi Validation (JavaScript)

```javascript
// utils/validation.js
/**
 * Joi validation schemas for JavaScript projects
 */

const Joi = require('joi');

// Common validators
const nonEmptyString = Joi.string().required().min(1);
const emailSchema = Joi.string().email({ tlds: false });
const thaiMobileSchema = Joi.string().pattern(/^0[689]\d{8}$/);
const uuidSchema = Joi.string().uuid();

// Thai address schema
const thaiAddressSchema = Joi.object({
  houseNumber: Joi.string().allow('', null),
  villageNumber: Joi.string().pattern(/^\d+$/),
  alley: Joi.string().allow('', null),
  road: Joi.string().allow('', null),
  subdistrict: Joi.string().required(),
  district: Joi.string().required(),
  province: Joi.string().required(),
  postalCode: Joi.string().pattern(/^\d{5}$/)
});

// Job creation schema
const jobSchema = Joi.object({
  ref: Joi.string().max(50).required(),
  originId: uuidSchema.required(),
  destination: Joi.object({
    lat: Joi.number().min(-90).max(90).required(),
    lng: Joi.number().min(-180).max(180).required(),
    address: Joi.string().required()
  }).required(),
  stops: Joi.array().items(
    Joi.object({
      lat: Joi.number().min(-90).max(90).required(),
      lng: Joi.number().min(-180).max(180).required(),
      address: Joi.string().required(),
      notes: Joi.string().allow('', null)
    })
  ).max(10),
  driverId: uuidSchema.allow(null),
  status: Joi.string().valid('pending', 'active', 'completed', 'cancelled').default('pending')
});

// Validation middleware
function validate(schema) {
  return (req, res, next) => {
    const { error, value } = schema.validate(req.body, {
      abortEarly: false,
      stripUnknown: true
    });

    if (error) {
      const errors = error.details.map(detail => ({
        field: detail.path.join('.'),
        message: detail.message
      }));

      return res.status(400).json({
        error: 'Validation failed',
        details: errors
      });
    }

    req.validated = value;
    next();
  };
}

module.exports = {
  nonEmptyString,
  emailSchema,
  thaiMobileSchema,
  thaiAddressSchema,
  jobSchema,
  validate
};
```

### Client-Side Validation

```typescript
// utils/client-validation.ts
/**
 * Client-side validation utilities
 */

// Thai ID card validation
export function validateThaiId(id: string): boolean {
  // Remove non-digits
  const digits = id.replace(/\D/g, '');

  // Must be 13 digits
  if (digits.length !== 13) return false;

  // Check if all digits are same (invalid)
  if (/^(\d)\1{12}$/.test(digits)) return false;

  // Calculate checksum
  let sum = 0;
  for (let i = 0; i < 12; i++) {
    sum += parseInt(digits[i]) * (13 - i);
  }

  const checksum = (11 - (sum % 11)) % 10;
  return checksum === parseInt(digits[12]);
}

// Thai mobile validation
export function validateThaiMobile(phone: string): boolean {
  return /^0[689]\d{8}$/.test(phone.replace(/\s/g, ''));
}

// Email validation
export function validateEmail(email: string): boolean {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}

// Password strength
export function getPasswordStrength(password: string): {
  score: number;
  feedback: string;
} {
  let score = 0;

  if (password.length >= 8) score++;
  if (password.length >= 12) score++;
  if (/[a-z]/.test(password) && /[A-Z]/.test(password)) score++;
  if (/\d/.test(password)) score++;
  if (/[^a-zA-Z0-9]/.test(password)) score++;

  const feedbacks = [
    'Very weak',
    'Weak',
    'Fair',
    'Good',
    'Strong',
    'Very strong'
  ];

  return {
    score,
    feedback: feedbacks[score] || 'Very weak'
  };
}

// License plate validation (Thailand)
export function validateLicensePlate(plate: string): boolean {
  // Basic validation - Thai license plates vary in format
  // This is a simplified version
  const patterns = [
    /^\d[a-zA-Z]\d{4}$/,           // 1ก1234
    /^\d{2}[a-zA-Z]\d{4}$/,        // 12ก1234
    /^\d{3}[a-zA-Z]\d{4}$/,        // 123ก1234
    /^[a-zA-Z]{2}\d{4}$/,          // AB1234
    /^[a-zA-Z]{3}\d{3}$/,          // ABC123
  ];

  return patterns.some(p => p.test(plate.replace(/\s/g, '')));
}

// Form field validation
export interface ValidationRule {
  required?: boolean;
  minLength?: number;
  maxLength?: number;
  pattern?: RegExp;
  custom?: (value: string) => boolean;
  message?: string;
}

export function validateField(
  value: string,
  rules: ValidationRule
): { valid: boolean; error?: string } {
  // Check required
  if (rules.required && !value.trim()) {
    return { valid: false, error: rules.message || 'This field is required' };
  }

  // Skip other validations if empty and not required
  if (!value.trim() && !rules.required) {
    return { valid: true };
  }

  // Check min length
  if (rules.minLength && value.length < rules.minLength) {
    return {
      valid: false,
      error: rules.message || `Minimum ${rules.minLength} characters required`
    };
  }

  // Check max length
  if (rules.maxLength && value.length > rules.maxLength) {
    return {
      valid: false,
      error: rules.message || `Maximum ${rules.maxLength} characters allowed`
    };
  }

  // Check pattern
  if (rules.pattern && !rules.pattern.test(value)) {
    return { valid: false, error: rules.message || 'Invalid format' };
  }

  // Check custom validation
  if (rules.custom && !rules.custom(value)) {
    return { valid: false, error: rules.message || 'Invalid value' };
  }

  return { valid: true };
}
```

## Mobile/PWA Patterns

### Service Worker for Offline

```javascript
// sw.js
/**
 * Service Worker for offline capability
 * Cache-first strategy for assets, network-first for API
 */

const CACHE_NAME = 'driver-app-v1';
const API_CACHE_NAME = 'api-cache-v1';

// Assets to cache immediately
const PRECACHE_ASSETS = [
  '/',
  '/index.html',
  '/css/style.css',
  '/js/app.js',
  '/js/location-service.js',
  '/manifest.json',
  '/icons/icon-192.png',
  '/icons/icon-512.png'
];

// Install event - precache assets
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME).then((cache) => {
      return cache.addAll(PRECACHE_ASSETS);
    })
  );
  self.skipWaiting();
});

// Activate event - clean up old caches
self.addEventListener('activate', (event) => {
  event.waitUntil(
    caches.keys().then((cacheNames) => {
      return Promise.all(
        cacheNames
          .filter((name) => name !== CACHE_NAME && name !== API_CACHE_NAME)
          .map((name) => caches.delete(name))
      );
    })
  );
  self.clients.claim();
});

// Fetch event - routing strategy
self.addEventListener('fetch', (event) => {
  const { request } = event;
  const url = new URL(request.url);

  // API requests - network first, then cache
  if (url.pathname.startsWith('/api/')) {
    event.respondWith(networkFirst(request));
    return;
  }

  // Static assets - cache first
  event.respondWith(cacheFirst(request));
});

// Network-first strategy (for API)
async function networkFirst(request) {
  const cache = await caches.open(API_CACHE_NAME);

  try {
    const response = await fetch(request);

    // Cache successful responses
    if (response.status === 200) {
      cache.put(request, response.clone());
    }

    return response;
  } catch (error) {
    // Fall back to cache
    const cached = await cache.match(request);
    if (cached) {
      return cached;
    }

    // Return offline fallback
    return new Response(JSON.stringify({ error: 'Offline' }), {
      status: 503,
      headers: { 'Content-Type': 'application/json' }
    });
  }
}

// Cache-first strategy (for static assets)
async function cacheFirst(request) {
  const cache = await caches.open(CACHE_NAME);
  const cached = await cache.match(request);

  if (cached) {
    return cached;
  }

  try {
    const response = await fetch(request);
    cache.put(request, response.clone());
    return response;
  } catch (error) {
    // Return offline fallback for HTML
    if (request.headers.get('accept')?.includes('text/html')) {
      return caches.match('/offline.html');
    }
    throw error;
  }
}

// Background sync for queued requests
self.addEventListener('sync', (event) => {
  if (event.tag === 'sync-queue') {
    event.waitUntil(syncOfflineQueue());
  }
});

async function syncOfflineQueue() {
  const queue = JSON.parse(localStorage.getItem('offlineQueue') || '[]');
  const results = [];

  for (const item of queue) {
    try {
      const response = await fetch(item.url, {
        method: item.method,
        headers: item.headers,
        body: item.body
      });

      if (response.ok) {
        results.push({ id: item.id, success: true });
      } else {
        results.push({ id: item.id, success: false });
      }
    } catch (error) {
      results.push({ id: item.id, success: false });
    }
  }

  // Remove successful items from queue
  const remaining = queue.filter(item =>
    !results.find(r => r.id === item.id && r.success)
  );

  localStorage.setItem('offlineQueue', JSON.stringify(remaining));

  return results;
}
```

### Touch Events & Gestures

```typescript
// utils/gestures.ts
/**
 * Touch gesture utilities for mobile
 */

export interface TouchGesture {
  onTap?: (e: TouchEvent) => void;
  onDoubleTap?: (e: TouchEvent) => void;
  onSwipeLeft?: (e: TouchEvent) => void;
  onSwipeRight?: (e: TouchEvent) => void;
  onSwipeUp?: (e: TouchEvent) => void;
  onSwipeDown?: (e: TouchEvent) => void;
  onPinch?: (scale: number) => void;
}

export class GestureHandler {
  private lastTap = 0;
  private touchStartX = 0;
  private touchStartY = 0;
  private initialPinchDistance = 0;

  constructor(
    private element: HTMLElement,
    private handlers: TouchGesture
  ) {
    this.attachListeners();
  }

  private attachListeners() {
    this.element.addEventListener('touchstart', this.handleTouchStart, { passive: true });
    this.element.addEventListener('touchend', this.handleTouchEnd, { passive: true });
    this.element.addEventListener('touchmove', this.handleTouchMove, { passive: true });
  }

  private handleTouchStart = (e: TouchEvent) => {
    if (e.touches.length === 1) {
      this.touchStartX = e.touches[0].clientX;
      this.touchStartY = e.touches[0].clientY;

      // Detect tap
      const now = Date.now();
      if (now - this.lastTap < 300) {
        this.handlers.onDoubleTap?.(e);
      } else {
        setTimeout(() => {
          if (Date.now() - this.lastTap >= 300) {
            this.handlers.onTap?.(e);
          }
        }, 300);
      }
      this.lastTap = now;
    }

    // Detect pinch
    if (e.touches.length === 2) {
      this.initialPinchDistance = this.getDistance(e.touches[0], e.touches[1]);
    }
  };

  private handleTouchEnd = (e: TouchEvent) => {
    if (e.changedTouches.length !== 1) return;

    const touch = e.changedTouches[0];
    const deltaX = touch.clientX - this.touchStartX;
    const deltaY = touch.clientY - this.touchStartY;

    // Minimum swipe distance
    const minSwipeDistance = 50;

    if (Math.abs(deltaX) > Math.abs(deltaY)) {
      if (deltaX > minSwipeDistance) {
        this.handlers.onSwipeRight?.(e);
      } else if (deltaX < -minSwipeDistance) {
        this.handlers.onSwipeLeft?.(e);
      }
    } else {
      if (deltaY > minSwipeDistance) {
        this.handlers.onSwipeDown?.(e);
      } else if (deltaY < -minSwipeDistance) {
        this.handlers.onSwipeUp?.(e);
      }
    }
  };

  private handleTouchMove = (e: TouchEvent) => {
    if (e.touches.length === 2) {
      const currentDistance = this.getDistance(e.touches[0], e.touches[1]);
      const scale = currentDistance / this.initialPinchDistance;
      this.handlers.onPinch?.(scale);
    }
  };

  private getDistance(touch1: Touch, touch2: Touch): number {
    const dx = touch1.clientX - touch2.clientX;
    const dy = touch1.clientY - touch2.clientY;
    return Math.sqrt(dx * dx + dy * dy);
  }

  destroy() {
    this.element.removeEventListener('touchstart', this.handleTouchStart);
    this.element.removeEventListener('touchend', this.handleTouchEnd);
    this.element.removeEventListener('touchmove', this.handleTouchMove);
  }
}

// Usage
// const gesture = new GestureHandler(element, {
//   onSwipeLeft: () => console.log('swiped left'),
//   onSwipeRight: () => console.log('swiped right'),
//   onTap: () => console.log('tapped')
// });
```

### Viewport Handling for LINE In-App Browser

```typescript
// utils/viewport.ts
/**
 * Viewport utilities for LINE LIFF and mobile browsers
 */

// LIFF view sizes
export type LIFFViewSize = 'compact' | 'tall' | 'full';

export interface ViewportInfo {
  width: number;
  height: number;
  viewSize: LIFFViewSize;
  isInClient: boolean;
  isMobile: boolean;
  os: 'ios' | 'android' | 'unknown';
}

// Get viewport info
export function getViewportInfo(): ViewportInfo {
  const width = window.innerWidth;
  const height = window.innerHeight;

  // Detect view size based on height
  let viewSize: LIFFViewSize = 'full';
  if (height < 400) {
    viewSize = 'compact';
  } else if (height < 700) {
    viewSize = 'tall';
  }

  // Detect OS
  const ua = navigator.userAgent;
  let os: 'ios' | 'android' | 'unknown' = 'unknown';
  if (/iPhone|iPad|iPod/.test(ua)) {
    os = 'ios';
  } else if (/Android/.test(ua)) {
    os = 'android';
  }

  return {
    width,
    height,
    viewSize,
    isInClient: isLINEInClient(),
    isMobile: width < 768,
    os
  };
}

// Check if running in LINE app
export function isLINEInClient(): boolean {
  return /LINE/.test(navigator.userAgent);
}

// Adjust viewport for safe areas (notch, etc.)
export function getSafeAreaInsets(): {
  top: number;
  right: number;
  bottom: number;
  left: number;
} {
  const style = getComputedStyle(document.documentElement);

  return {
    top: parseInt(style.getPropertyValue('--safe-area-inset-top') || '0'),
    right: parseInt(style.getPropertyValue('--safe-area-inset-right') || '0'),
    bottom: parseInt(style.getPropertyValue('--safe-area-inset-bottom') || '0'),
    left: parseInt(style.getPropertyValue('--safe-area-inset-left') || '0')
  };
}

// Set CSS variables for viewport
export function setViewportVariables() {
  const info = getViewportInfo();
  const insets = getSafeAreaInsets();

  document.documentElement.style.setProperty('--viewport-width', `${info.width}px`);
  document.documentElement.style.setProperty('--viewport-height', `${info.height}px`);
  document.documentElement.style.setProperty('--safe-top', `${insets.top}px`);
  document.documentElement.style.setProperty('--safe-bottom', `${insets.bottom}px`);

  // Set device-specific classes
  document.documentElement.classList.toggle('ios', info.os === 'ios');
  document.documentElement.classList.toggle('android', info.os === 'android');
  document.documentElement.classList.toggle('line-app', info.isInClient);
  document.documentElement.classList.toggle('compact', info.viewSize === 'compact');
  document.documentElement.classList.toggle('tall', info.viewSize === 'tall');
}
```

### Push Notifications (Web Push)

```typescript
// utils/push-notifications.ts
/**
 * Web Push Notifications using Supabase
 */

const VAPID_PUBLIC_KEY = 'YOUR_VAPID_PUBLIC_KEY';

export async function requestNotificationPermission(): Promise<boolean> {
  if (!('Notification' in window)) {
    console.warn('Notifications not supported');
    return false;
  }

  const permission = await Notification.requestPermission();
  return permission === 'granted';
}

export async function subscribeToPush(
  userId: string,
  supabase: any
): Promise<PushSubscription | null> {
  if (!('serviceWorker' in navigator)) {
    return null;
  }

  const registration = await navigator.serviceWorker.ready;

  // Subscribe to push
  const subscription = await registration.pushManager.subscribe({
    userVisibleOnly: true,
    applicationServerKey: urlBase64ToUint8Array(VAPID_PUBLIC_KEY)
  });

  // Save subscription to database
  await supabase.from('push_subscriptions').upsert({
    user_id: userId,
    endpoint: subscription.endpoint,
    keys: JSON.stringify(subscription.toJSON().keys),
    created_at: new Date().toISOString()
  });

  return subscription;
}

// Helper to convert VAPID key
function urlBase64ToUint8Array(base64String: string): Uint8Array {
  const padding = '='.repeat((4 - base64String.length % 4) % 4);
  const base64 = (base64String + padding)
    .replace(/\-/g, '+')
    .replace(/_/g, '/');

  const rawData = window.atob(base64);
  const outputArray = new Uint8Array(rawData.length);

  for (let i = 0; i < rawData.length; ++i) {
    outputArray[i] = rawData.charCodeAt(i);
  }

  return outputArray;
}

// Local notification (no service worker)
export function showLocalNotification(title: string, options?: NotificationOptions): void {
  if (Notification.permission === 'granted') {
    new Notification(title, {
      icon: '/icons/icon-192.png',
      badge: '/icons/badge-72.png',
      ...options
    });
  }
}
```

## SweetAlert2 Patterns

### Basic Setup

```html
<!-- Load SweetAlert2 from CDN -->
<script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>

<!-- Or install via npm -->
<!-- npm install sweetalert2 -->
```

```typescript
// TypeScript import
import Swal from 'sweetalert2';
import withReactContent from 'sweetalert2-react-content';

// For React
const SwalReact = withReactContent(Swal);
```

### Basic Alerts

```typescript
// Simple alert
Swal.fire('บันทึกสำเร็จ!');

// Alert with title and text
Swal.fire(
  'ยืนยันการลบ?',
  'ข้อมูลที่ลบจะไม่สามารถกู้คืนได้',
  'question'
);

// Auto-close after 2 seconds
Swal.fire({
  icon: 'success',
  title: 'บันทึกสำเร็จ!',
  timer: 2000,
  showConfirmButton: false
});

// Thai language
Swal.fire({
  icon: 'success',
  title: 'สำเร็จ!',
  text: 'บันทึกข้อมูลเรียบร้อยแล้ว',
  confirmButtonText: 'ตกลง',
  confirmButtonColor: '#00B900'
});
```

### Icon Types

```typescript
// Success
Swal.fire({
  icon: 'success',
  title: 'บันทึกสำเร็จ!',
  text: 'ข้อมูลถูกบันทึกเรียบร้อยแล้ว'
});

// Error
Swal.fire({
  icon: 'error',
  title: 'เกิดข้อผิดพลาด!',
  text: 'ไม่สามารถบันทึกข้อมูลได้ กรุณาลองใหม่'
});

// Warning
Swal.fire({
  icon: 'warning',
  title: 'คำเตือน',
  text: 'ข้อมูลอาจสูญหาย ต้องการดำเนินการต่อ?'
});

// Info
Swal.fire({
  icon: 'info',
  title: 'ข้อมูล',
  text: 'ตรวจสอบข้อมูลให้ถูกต้อง'
});

// Question
Swal.fire({
  icon: 'question',
  title: 'ยืนยัน?',
  text: 'ต้องการดำเนินการนี้หรือไม่?'
});

// Custom icon
Swal.fire({
  icon: 'success',
  title: 'งานเสร็จสิ้น',
  html: '<img src="/success-icon.png" class="w-16 h-16 mx-auto mb-4">'
});
```

### Confirmation Dialogs

```typescript
// Delete confirmation
async function confirmDelete(id: string): Promise<boolean> {
  const result = await Swal.fire({
    title: 'ยืนยันการลบ?',
    text: 'ข้อมูลที่ลบจะไม่สามารถกู้คืนได้',
    icon: 'warning',
    showCancelButton: true,
    confirmButtonColor: '#EF4444',
    cancelButtonColor: '#6B7280',
    confirmButtonText: 'ลบข้อมูล',
    cancelButtonText: 'ยกเลิก',
    reverseButtons: true
  });

  return result.isConfirmed;
}

// Usage
if (await confirmDelete(jobId)) {
  await deleteJob(jobId);
  Swal.fire('ลบแล้ว!', 'ข้อมูลถูกลบเรียบร้อย', 'success');
}

// Confirm check-out
async function confirmCheckout(): Promise<boolean> {
  const result = await Swal.fire({
    title: 'เช็คเอาท์?',
    html: 'ยืนยันที่จะเช็คเอาท์จากงานนี้<br><small class="text-gray-500">หลังจากเช็คเอาท์จะไม่สามารถแก้ไขได้</small>',
    icon: 'question',
    showCancelButton: true,
    confirmButtonColor: '#00B900',
    cancelButtonColor: '#6B7280',
    confirmButtonText: '<i class="fas fa-check"></i> ยืนยัน',
    cancelButtonText: '<i class="fas fa-times"></i> ยกเลิก'
  });

  return result.isConfirmed;
}

// Custom danger confirmation
Swal.fire({
  title: '<strong class="text-red-600">อันตราย!</strong>',
  html: 'การกระทำนี้<b>ไม่สามารถย้อนกลับได้</b>',
  icon: 'warning',
  showCancelButton: true,
  confirmButtonColor: '#EF4444',
  cancelButtonColor: '#6B7280',
  confirmButtonText: 'ใช่, ลบเลย!',
  cancelButtonText: 'ไม่, ยกเลิก',
  dangerMode: true
});
```

### Toast Notifications

```typescript
// Toast position options
const Toast = Swal.mixin({
  toast: true,
  position: 'top-end', // top, top-start, top-end, center, center-start, center-end, bottom, bottom-start, bottom-end
  showConfirmButton: false,
  timer: 3000,
  timerProgressBar: true,
  didOpen: (toast) => {
    toast.addEventListener('mouseenter', Swal.stopTimer);
    toast.addEventListener('mouseleave', Swal.resumeTimer);
  }
});

// Success toast
Toast.fire({
  icon: 'success',
  title: 'บันทึกสำเร็จ'
});

// Error toast
Toast.fire({
  icon: 'error',
  title: 'เกิดข้อผิดพลาด'
});

// Custom toast
Toast.fire({
  icon: 'success',
  title: 'ส่งข้อมูลแล้ว',
  html: '<small class="text-gray-500">รอการอนุมัติจากแอดมิน</small>'
});

// Loading toast
const loadingToast = Swal.fire({
  title: 'กำลังบันทึก...',
  text: 'กรุณารอสักครู่',
  allowOutsideClick: false,
  didOpen: () => {
    Swal.showLoading();
  }
});

// Close loading toast
setTimeout(() => {
  Swal.close();
  Swal.fire('สำเร็จ!', '', 'success');
}, 2000);
```

### Input Forms

```typescript
// Text input
const { value: name } = await Swal.fire({
  title: 'กรอกชื่อ',
  input: 'text',
  inputLabel: 'ชื่อพนักงานขับรถ',
  inputPlaceholder: 'ระบุชื่อจริง',
  showCancelButton: true,
  confirmButtonText: 'บันทึก',
  cancelButtonText: 'ยกเลิก',
  confirmButtonColor: '#00B900',
  inputValidator: (value) => {
    if (!value) {
      return 'กรุณาระบุชื่อ!';
    }
  }
});

if (name) {
  console.log('ชื่อ:', name);
}

// Email input
const { value: email } = await Swal.fire({
  title: 'กรอกอีเมล',
  input: 'email',
  inputLabel: 'อีเมลติดต่อ',
  inputPlaceholder: 'name@example.com',
  confirmButtonColor: '#00B900'
});

// Password input
const { value: password } = await Swal.fire({
  title: 'กรอกรหัสผ่าน',
  input: 'password',
  inputLabel: 'รหัสผ่านใหม่',
  inputPlaceholder: '••••••••',
  inputAttributes: {
    minlength: 8,
    autocapitalize: 'off',
    autocorrect: 'off'
  },
  confirmButtonColor: '#00B900'
});

// Number input
const { value: amount } = await Swal.fire({
  title: 'กรอกจำนวน',
  input: 'number',
  inputLabel: 'ปริมาณน้ำมัน (ลิตร)',
  inputValue: 100,
  inputAttributes: {
    min: 0,
    max: 1000,
    step: 0.1
  }
});

// Textarea
const { value: notes } = await Swal.fire({
  title: 'เพิ่มหมายเหตุ',
  input: 'textarea',
  inputLabel: 'หมายเหตุเพิ่มเติม',
  inputPlaceholder: 'ระบุรายละเอียด...',
  inputAttributes: {
    'aria-label': 'Type your message here'
  },
  showCancelButton: true
});

// Select dropdown
const { value: status } = await Swal.fire({
  title: 'เลือกสถานะ',
  input: 'select',
  inputOptions: {
    pending: 'รอดำเนินการ',
    active: 'กำลังดำเนินการ',
    completed: 'เสร็จสิ้น',
    cancelled: 'ยกเลิก'
  },
  inputPlaceholder: 'เลือกสถานะ',
  showCancelButton: true,
  confirmButtonColor: '#00B900'
});

// Radio buttons
const { value: vehicle } = await Swal.fire({
  title: 'เลือกประเภทรถ',
  input: 'radio',
  inputOptions: {
    truck: 'รถบรรทุก',
    trailer: 'รถพ่วง',
    tanker: 'รถถัง'
  },
  inputValidator: (value) => {
    if (!value) return 'กรุณาเลือกประเภทรถ!';
  }
});

// Checkbox
const { value: accept } = await Swal.fire({
  title: 'เงื่อนไขการใช้งาน',
  input: 'checkbox',
  inputValue: 1,
  inputPlaceholder: 'ยอมรับเงื่อนไขและข้อตกลง',
  confirmButtonText: 'ยอมรับ',
  confirmButtonColor: '#00B900',
  inputValidator: (result) => {
    return !result && 'กรุณายอมรับเงื่อนไข!';
  }
});

// File upload
const { value: file } = await Swal.fire({
  title: 'อัพโหลดรูปภาพ',
  input: 'file',
  inputAttributes: {
    accept: 'image/*',
    'aria-label': 'Upload your profile picture'
  }
});

if (file) {
  const reader = new FileReader();
  reader.onload = (e) => {
    Swal.fire({
      title: 'รูปภาพที่อัพโหลด',
      imageUrl: e.target?.result as string,
      imageAlt: 'The uploaded picture'
    });
  };
  reader.readAsDataURL(file);
}
```

### Multiple Steps / Wizard

```typescript
// Step-by-step wizard
Swal.mixin({
  confirmButtonText: 'ถัดไป →',
  showCancelButton: true,
  cancelButtonText: 'ยกเลิก',
  confirmButtonColor: '#00B900',
  progressSteps: ['1', '2', '3']
}).queue([
  {
    title: 'ขั้นตอนที่ 1',
    text: 'กรอกข้อมูลส่วนตัว',
    input: 'text',
    inputLabel: 'ชื่อ-นามสกุล'
  },
  {
    title: 'ขั้นตอนที่ 2',
    text: 'กรอกข้อมูลติดต่อ',
    input: 'email',
    inputLabel: 'อีเมล'
  },
  {
    title: 'ขั้นตอนที่ 3',
    text: 'ตั้งรหัสผ่าน',
    input: 'password',
    inputLabel: 'รหัสผ่าน'
  }
]).then((result) => {
  if (result.value) {
    Swal.fire({
      title: 'สมัครสมาชิกสำเร็จ!',
      html: `ข้อมูลของคุณ: <pre>${JSON.stringify(result.value)}</pre>`,
      icon: 'success',
      confirmButtonColor: '#00B900'
    });
  }
});
```

### Custom Styling (Tailwind)

```typescript
// Custom styled alert with Tailwind classes
Swal.fire({
  title: 'สำเร็จ!',
  text: 'บันทึกข้อมูลเรียบร้อยแล้ว',
  icon: 'success',
  iconColor: '#22C55E',
  background: '#FFFFFF',
  color: '#1F2937',
  confirmButtonColor: '#00B900',
  confirmButtonText: 'ตกลง',
  customClass: {
    popup: 'rounded-xl shadow-2xl',
    title: 'text-xl font-bold text-gray-900',
    htmlContainer: 'text-gray-700',
    confirmButton: 'px-6 py-2 bg-line-green hover:bg-line-green-dark text-white rounded-lg font-medium transition-colors',
    cancelButton: 'px-6 py-2 bg-gray-200 hover:bg-gray-300 text-gray-900 rounded-lg font-medium transition-colors'
  }
});

// Dark mode support
Swal.fire({
  title: 'ยืนยัน?',
  text: 'ต้องการดำเนินการต่อหรือไม่',
  icon: 'question',
  showCancelButton: true,
  confirmButtonColor: '#00B900',
  cancelButtonColor: '#6B7280',
  background: document.documentElement.classList.contains('dark') ? '#1F2937' : '#FFFFFF',
  color: document.documentElement.classList.contains('dark') ? '#F9FAFB' : '#1F2937'
});
```

### React Integration

```tsx
// SweetAlert2 with React components
import Swal from 'sweetalert2';
import withReactContent from 'sweetalert2-react-content';

const SwalReact = withReactContent(Swal);

// Using React components in SweetAlert
SwalReact.fire({
  title: <p className="text-xl font-bold">ยืนยันการลบ?</p>,
  html: (
    <div className="text-gray-700">
      <p>ข้อมูลที่ลบจะไม่สามารถกู้คืนได้</p>
      <div className="mt-4 p-4 bg-red-50 rounded-lg">
        <p className="text-red-800 font-medium">ID: {itemId}</p>
      </div>
    </div>
  ),
  icon: 'warning',
  showCancelButton: true,
  confirmButtonColor: '#EF4444',
  cancelButtonColor: '#6B7280',
  confirmButtonText: 'ลบ',
  cancelButtonText: 'ยกเลิก'
});

// Custom React component as modal
SwalReact.fire({
  html: (
    <form className="space-y-4">
      <div>
        <label className="block text-sm font-medium text-gray-700">ชื่อ</label>
        <input type="text" className="mt-1 block w-full rounded-md border-gray-300 shadow-sm" />
      </div>
      <div>
        <label className="block text-sm font-medium text-gray-700">อีเมล</label>
        <input type="email" className="mt-1 block w-full rounded-md border-gray-300 shadow-sm" />
      </div>
    </form>
  ),
  showConfirmButton: false,
  showCloseButton: true
});
```

### Common Use Cases for DriverConnect

```typescript
// Check-in confirmation
async function confirmCheckIn(jobId: string): Promise<void> {
  const result = await Swal.fire({
    title: 'เช็คอิน?',
    html: `
      <div class="text-left">
        <p class="mb-2"><strong>งาน:</strong> ${jobId}</p>
        <p class="mb-2"><strong>เวลา:</strong> ${new Date().toLocaleTimeString('th-TH')}</p>
        <p class="text-sm text-gray-500">กรุณาอยู่ในรัศมี 100 เมตรจากจุดหมาย</p>
      </div>
    `,
    icon: 'question',
    showCancelButton: true,
    confirmButtonText: '<i class="fas fa-map-marker-alt"></i> เช็คอิน',
    cancelButtonText: 'ยกเลิก',
    confirmButtonColor: '#00B900'
  });

  if (result.isConfirmed) {
    // Perform check-in
  }
}

// Alcohol test result
async function showAlcoholTestResult(passed: boolean): Promise<void> {
  if (passed) {
    await Swal.fire({
      title: 'ผลการทดสอบ',
      html: `
        <div class="text-center">
          <div class="text-6xl mb-4">✅</div>
          <p class="text-2xl font-bold text-green-600">ผ่านเกณฑ์</p>
          <p class="text-gray-600 mt-2">สามารถปฏิบัติงานได้</p>
        </div>
      `,
      confirmButtonText: 'ตกลง',
      confirmButtonColor: '#22C55E'
    });
  } else {
    await Swal.fire({
      title: 'ผลการทดสอบ',
      html: `
        <div class="text-center">
          <div class="text-6xl mb-4">❌</div>
          <p class="text-2xl font-bold text-red-600">ไม่ผ่านเกณฑ์</p>
          <p class="text-gray-600 mt-2">กรุณาทดสอบใหม่ภายหลัง 1 ชั่วโมง</p>
        </div>
      `,
      confirmButtonText: 'ตกลง',
      confirmButtonColor: '#EF4444'
    });
  }
}

// Job assignment
async function assignJob(jobId: string): Promise<void> {
  const { value: driverId } = await Swal.fire({
    title: 'มอบหมายงาน',
    input: 'select',
    inputLabel: 'เลือกพนักงานขับรถ',
    inputOptions: await getAvailableDrivers(),
    inputPlaceholder: 'เลือกพนักงาน...',
    showCancelButton: true,
    confirmButtonText: 'มอบหมาย',
    cancelButtonText: 'ยกเลิก',
    confirmButtonColor: '#00B900'
  });

  if (driverId) {
    await assignJobToDriver(jobId, driverId);

    Toast.fire({
      icon: 'success',
      title: 'มอบหมายงานสำเร็จ'
    });
  }
}

// Odometer input
async function inputOdometer(currentValue: number): Promise<number | null> {
  const { value: odometer } = await Swal.fire({
    title: 'บันทึกเลขไมล์',
    html: `
      <p class="text-sm text-gray-600 mb-4">เลขไมล์ปัจจุบัน: <strong>${currentValue.toLocaleString()}</strong> กม.</p>
    `,
    input: 'number',
    inputLabel: 'เลขไมล์ใหม่',
    inputValue: currentValue,
    inputAttributes: {
      min: currentValue,
      step: 1
    },
    showCancelButton: true,
    confirmButtonText: 'บันทึก',
    cancelButtonText: 'ยกเลิก',
    confirmButtonColor: '#00B900',
    inputValidator: (value) => {
      if (!value) return 'กรุณาระบุเลขไมล์!';
      if (value < currentValue) return 'เลขไมล์ใหม่ต้องมากกว่าปัจจุบัน!';
    }
  });

  return odometer || null;
}

// Photo preview before upload
async function previewPhoto(file: File): Promise<boolean> {
  const reader = new FileReader();

  const promise = new Promise<boolean>((resolve) => {
    reader.onload = (e) => {
      Swal.fire({
        title: 'ตรวจสอบรูปภาพ',
        imageUrl: e.target?.result as string,
        imageHeight: 300,
        imageAlt: 'Preview',
        showCancelButton: true,
        confirmButtonText: 'ใช้รูปนี้',
        cancelButtonText: 'ถ่ายใหม่',
        confirmButtonColor: '#00B900',
        reverseButtons: true
      }).then((result) => {
        resolve(result.isConfirmed);
      });
    };
  });

  reader.readAsDataURL(file);
  return promise;
}
```

### Queue & Pre-Configured

```typescript
// Pre-configured Swal instance
const Confirm = Swal.mixin({
  icon: 'question',
  showCancelButton: true,
  confirmButtonColor: '#00B900',
  cancelButtonColor: '#6B7280',
  confirmButtonText: 'ยืนยัน',
  cancelButtonText: 'ยกเลิก',
  reverseButtons: true
});

const Alert = Swal.mixin({
  confirmButtonColor: '#00B900',
  confirmButtonText: 'ตกลง'
});

const Toast = Swal.mixin({
  toast: true,
  position: 'top-end',
  showConfirmButton: false,
  timer: 3000,
  timerProgressBar: true
});

// Usage
Confirm.fire('ยืนยัน?', 'คุณต้องการดำเนินการหรือไม่');
Alert.fire('สำเร็จ!', 'บันทึกเรียบร้อย', 'success');
Toast.fire({ icon: 'success', title: 'บันทึกสำเร็จ' });
```

---

## Dashboard Design & Presentation - Japanese Style

### Japanese Design Philosophy for Executive Dashboards

Japanese dashboard design follows these core principles:

| Principle | Description | Application |
|-----------|-------------|-------------|
| **Ma (間)** | Negative space, breathing room | Generous whitespace, visual hierarchy |
| **Kanso (簡素)** | Simplicity, elimination of clutter | Only essential metrics, minimal decorations |
| **Shibui (渋い)** | Subtle elegance, understated beauty | Muted colors, refined typography |
| **Seijaku (静寂)** | Stillness, calmness | Clean layouts, reduced motion |
| **Datsuzoku (脱俗)** | Unconventional, break from routine | Creative data presentation, unique insights |
| **Omoiyari (おもいやり)** | Empathy for the user | Clear information architecture, intuitive UX |

### Executive Dashboard - Design System

```css
/* Japanese-inspired color palette for dashboards */
:root {
  /* Primary - Trustworthy blue-greys */
  --jp-primary-50: #f0f4f8;
  --jp-primary-100: #d9e2ec;
  --jp-primary-200: #bcccdc;
  --jp-primary-300: #9fb3c8;
  --jp-primary-400: #829ab1;
  --jp-primary-500: #627d98;
  --jp-primary-600: #486581;
  --jp-primary-700: #334e68;

  /* Accent - Subtle indigo */
  --jp-accent: #4c51bf;
  --jp-accent-light: #a3bffa;

  /* Semantic colors - Muted */
  --jp-success: #38a169;
  --jp-success-bg: #c6f6d5;
  --jp-warning: #dd6b20;
  --jp-warning-bg: #feebc8;
  --jp-error: #e53e3e;
  --jp-error-bg: #fed7d7;
  --jp-info: #3182ce;
  --jp-info-bg: #bee3f8;

  /* Neutral grays */
  --jp-gray-50: #f7fafc;
  --jp-gray-100: #edf2f7;
  --jp-gray-200: #e2e8f0;
  --jp-gray-300: #cbd5e0;
  --jp-gray-400: #a0aec0;
  --jp-gray-500: #718096;
  --jp-gray-600: #4a5568;
  --jp-gray-700: #2d3748;
  --jp-gray-800: #1a202c;
  --jp-gray-900: #171923;

  /* Spacing scale (8pt grid) */
  --jp-space-1: 0.25rem;  /* 4px */
  --jp-space-2: 0.5rem;   /* 8px */
  --jp-space-3: 0.75rem;  /* 12px */
  --jp-space-4: 1rem;     /* 16px */
  --jp-space-5: 1.25rem;  /* 20px */
  --jp-space-6: 1.5rem;   /* 24px */
  --jp-space-8: 2rem;     /* 32px */
  --jp-space-10: 2.5rem;  /* 40px */
  --jp-space-12: 3rem;    /* 48px */

  /* Shadows - Subtle elevation */
  --jp-shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.05);
  --jp-shadow-md: 0 4px 6px rgba(0, 0, 0, 0.07);
  --jp-shadow-lg: 0 10px 15px rgba(0, 0, 0, 0.08);

  /* Border radius - Minimal */
  --jp-radius-sm: 0.25rem;
  --jp-radius-md: 0.375rem;
  --jp-radius-lg: 0.5rem;
}

/* Typography hierarchy */
.jp-dashboard {
  font-family: 'Noto Sans JP', -apple-system, BlinkMacSystemFont, sans-serif;
  line-height: 1.6;
  color: var(--jp-gray-700);
}

.jp-dashboard h1 {
  font-size: 1.875rem;
  font-weight: 700;
  color: var(--jp-gray-800);
  letter-spacing: -0.025em;
}

.jp-dashboard h2 {
  font-size: 1.5rem;
  font-weight: 600;
  color: var(--jp-gray-800);
  letter-spacing: -0.02em;
}

.jp-dashboard h3 {
  font-size: 1.25rem;
  font-weight: 600;
  color: var(--jp-gray-700);
}

.jp-dashboard .metric-value {
  font-size: 2.5rem;
  font-weight: 700;
  color: var(--jp-gray-800);
  letter-spacing: -0.03em;
  line-height: 1;
}

.jp-dashboard .metric-label {
  font-size: 0.875rem;
  font-weight: 500;
  color: var(--jp-gray-500);
  text-transform: uppercase;
  letter-spacing: 0.05em;
}
```

### KPI Card Component

```typescript
// components/dashboard/KpiCard.tsx
/**
 * Japanese-style KPI card with clean, minimal design
 */

interface KpiCardProps {
  label: string;
  value: string | number;
  change?: number;
  unit?: string;
  icon?: string;
  trend?: 'up' | 'down' | 'neutral';
  size?: 'sm' | 'md' | 'lg';
}

export function KpiCard({
  label,
  value,
  change,
  unit,
  icon,
  trend = 'neutral',
  size = 'md'
}: KpiCardProps) {
  const sizeClasses = {
    sm: 'p-4',
    md: 'p-6',
    lg: 'p-8'
  };

  const valueSizeClasses = {
    sm: 'text-2xl',
    md: 'text-3xl',
    lg: 'text-4xl'
  };

  const trendColors = {
    up: 'text-green-600 bg-green-50',
    down: 'text-red-600 bg-red-50',
    neutral: 'text-gray-600 bg-gray-50'
  };

  const trendIcons = {
    up: '↑',
    down: '↓',
    neutral: '−'
  };

  return (
    <div className={`bg-white rounded-lg border border-gray-100 ${sizeClasses[size]} shadow-sm`}>
      {/* Label row */}
      <div className="flex items-center justify-between mb-3">
        <span className="text-xs font-medium text-gray-500 uppercase tracking-wider">
          {label}
        </span>
        {icon && (
          <span className="text-gray-400 text-lg" aria-hidden="true">
            {icon}
          </span>
        )}
      </div>

      {/* Value */}
      <div className={`font-bold text-gray-800 ${valueSizeClasses[size]} tracking-tight mb-1`}>
        {typeof value === 'number' ? value.toLocaleString('th-TH') : value}
        {unit && (
          <span className="text-lg font-normal text-gray-500 ml-1">{unit}</span>
        )}
      </div>

      {/* Change indicator */}
      {change !== undefined && (
        <div className="flex items-center gap-2">
          <span className={`inline-flex items-center gap-1 px-2 py-0.5 rounded text-xs font-medium ${trendColors[trend]}`}>
            <span>{trendIcons[trend]}</span>
            <span>{Math.abs(change).toFixed(1)}%</span>
          </span>
          <span className="text-xs text-gray-400">vs last period</span>
        </div>
      )}
    </div>
  );
}

// Usage
<KpiCard
  label="Total Revenue"
  value={1245000}
  unit="฿"
  change={12.5}
  trend="up"
  size="lg"
/>
```

### Executive Summary Section

```typescript
// components/dashboard/ExecutiveSummary.tsx
/**
 * Top-level summary for executives with key insights
 */

interface ExecutiveSummaryProps {
  period: string;
  highlights: {
    title: string;
    value: string;
    status: 'positive' | 'negative' | 'neutral';
    detail: string;
  }[];
}

export function ExecutiveSummary({ period, highlights }: ExecutiveSummaryProps) {
  const statusColors = {
    positive: 'bg-green-50 border-green-200 text-green-800',
    negative: 'bg-red-50 border-red-200 text-red-800',
    neutral: 'bg-gray-50 border-gray-200 text-gray-800'
  };

  return (
    <section className="mb-8">
      {/* Period header */}
      <div className="flex items-center justify-between mb-6">
        <h2 className="text-xl font-semibold text-gray-800">Executive Summary</h2>
        <span className="text-sm text-gray-500 bg-gray-100 px-3 py-1 rounded-full">
          {period}
        </span>
      </div>

      {/* Highlights grid */}
      <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
        {highlights.map((item, idx) => (
          <div
            key={idx}
            className={`p-5 rounded-lg border ${statusColors[item.status]}`}
          >
            <p className="text-xs font-medium uppercase tracking-wider mb-2 opacity-70">
              {item.title}
            </p>
            <p className="text-2xl font-bold mb-1">{item.value}</p>
            <p className="text-sm opacity-80">{item.detail}</p>
          </div>
        ))}
      </div>
    </section>
  );
}
```

### Data Table - Japanese Minimal Style

```css
/* Japanese-style data table */
.jp-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.875rem;
}

.jp-table thead {
  background: linear-gradient(to bottom, var(--jp-gray-50), var(--jp-gray-100));
  border-bottom: 2px solid var(--jp-gray-200);
}

.jp-table th {
  padding: var(--jp-space-3) var(--jp-space-4);
  text-align: left;
  font-weight: 600;
  color: var(--jp-gray-700);
  font-size: 0.75rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.jp-table td {
  padding: var(--jp-space-3) var(--jp-space-4);
  border-bottom: 1px solid var(--jp-gray-100);
  color: var(--jp-gray-600);
}

.jp-table tbody tr:hover {
  background-color: var(--jp-gray-50);
}

.jp-table tbody tr:last-child td {
  border-bottom: none;
}

/* Status badges */
.jp-badge {
  display: inline-flex;
  align-items: center;
  padding: 0.25rem 0.625rem;
  border-radius: 9999px;
  font-size: 0.75rem;
  font-weight: 500;
}

.jp-badge.success { background: var(--jp-success-bg); color: var(--jp-success); }
.jp-badge.warning { background: var(--jp-warning-bg); color: var(--jp-warning); }
.jp-badge.error { background: var(--jp-error-bg); color: var(--jp-error); }
.jp-badge.info { background: var(--jp-info-bg); color: var(--jp-info); }
```

### Chart Configuration - Chart.js with Japanese Style

```typescript
// utils/dashboard/chart-config.ts
/**
 * Chart.js configuration for Japanese-style dashboards
 */

import { ChartOptions, ChartData } from 'chart.js';

// Japanese color palette (subtle, harmonious)
export const jpColors = {
  primary: ['#4c51bf', '#38a169', '#dd6b20', '#3182ce', '#e53e3e', '#805ad5'],
  gradients: [
    'rgba(76, 81, 191, 0.8)',
    'rgba(56, 161, 105, 0.8)',
    'rgba(221, 107, 32, 0.8)',
    'rgba(49, 130, 206, 0.8)',
    'rgba(229, 62, 62, 0.8)'
  ],
  neutral: {
    100: '#f7fafc',
    200: '#edf2f7',
    300: '#e2e8f0',
    400: '#cbd5e0',
    500: '#a0aec0',
    600: '#718096',
    700: '#4a5568',
    800: '#2d3748'
  }
};

// Base chart options
export const baseChartOptions: ChartOptions = {
  responsive: true,
  maintainAspectRatio: false,
  plugins: {
    legend: {
      display: false // Japanese style: minimal legends
    },
    tooltip: {
      backgroundColor: 'rgba(26, 32, 44, 0.9)',
      titleColor: '#fff',
      bodyColor: '#e2e8f0',
      padding: 12,
      cornerRadius: 6,
      displayColors: false,
      callbacks: {
        label: (context) => {
          if (context.parsed.y !== null) {
            return `${context.parsed.y.toLocaleString('th-TH')}`;
          }
          return '';
        }
      }
    }
  },
  scales: {
    x: {
      grid: { display: false },
      ticks: {
        color: jpColors.neutral[500],
        font: { size: 11 }
      }
    },
    y: {
      grid: {
        color: jpColors.neutral[200],
        drawBorder: false
      },
      ticks: {
        color: jpColors.neutral[500],
        font: { size: 11 },
        callback: (value) => value.toLocaleString('th-TH')
      }
    }
  }
};

// Line chart for trends
export function createTrendChart(labels: string[], data: number[]): ChartData {
  return {
    labels,
    datasets: [{
      data,
      borderColor: jpColors.primary[0],
      backgroundColor: (context) => {
        const ctx = context.chart.ctx;
        const gradient = ctx.createLinearGradient(0, 0, 0, 300);
        gradient.addColorStop(0, 'rgba(76, 81, 191, 0.3)');
        gradient.addColorStop(1, 'rgba(76, 81, 191, 0)');
        return gradient;
      },
      borderWidth: 2,
      fill: true,
      tension: 0.4,
      pointRadius: 0,
      pointHoverRadius: 6,
      pointHoverBackgroundColor: jpColors.primary[0],
      pointHoverBorderColor: '#fff',
      pointHoverBorderWidth: 2
    }]
  };
}

// Donut chart for distribution
export function createDonutChart(labels: string[], data: number[]): ChartData {
  return {
    labels,
    datasets: [{
      data,
      backgroundColor: jpColors.primary.slice(0, data.length),
      borderWidth: 0,
      hoverOffset: 8
    }]
  };
}

// Donut chart options (minimal)
export const donutChartOptions: ChartOptions = {
  responsive: true,
  maintainAspectRatio: false,
  cutout: '70%',
  plugins: {
    legend: {
      position: 'right',
      labels: {
        padding: 16,
        usePointStyle: true,
        pointStyle: 'circle',
        color: jpColors.neutral[600],
        font: { size: 12 }
      }
    },
    tooltip: {
      backgroundColor: 'rgba(26, 32, 44, 0.9)',
      padding: 12,
      cornerRadius: 6,
      callbacks: {
        label: (context) => {
          const value = context.parsed;
          const total = context.dataset.data.reduce((a: number, b: number) => a + b, 0);
          const percentage = ((value / total) * 100).toFixed(1);
          return `${context.label}: ${value.toLocaleString('th-TH')} (${percentage}%)`;
        }
      }
    }
  }
};
```

### Vanilla JS Dashboard Component

```javascript
// components/dashboard/Dashboard.js
/**
 * Vanilla JS dashboard with Japanese minimal design
 */

class Dashboard {
  constructor(containerId, options = {}) {
    this.container = document.getElementById(containerId);
    this.options = {
      theme: 'light',
      ...options
    };
    this.init();
  }

  init() {
    this.container.className = 'jp-dashboard';
    this.renderHeader();
    this.renderContent();
  }

  renderHeader() {
    const header = document.createElement('header');
    header.className = 'flex items-center justify-between py-6 border-b border-gray-100 mb-6';
    header.innerHTML = `
      <div>
        <h1 class="text-2xl font-bold text-gray-800 tracking-tight">ダッシュボード</h1>
        <p class="text-sm text-gray-500 mt-1">最終更新: ${this.formatDateTime(new Date())}</p>
      </div>
      <div class="flex items-center gap-3">
        <select class="px-3 py-2 text-sm border border-gray-200 rounded-lg bg-white">
          <option>今日</option>
          <option>今週</option>
          <option selected>今月</option>
          <option>今年</option>
        </select>
      </div>
    `;
    this.container.appendChild(header);
  }

  renderContent() {
    const content = document.createElement('div');
    content.className = 'dashboard-content';
    this.container.appendChild(content);
    this.content = content;
  }

  addKpiCard(label, value, options = {}) {
    const card = document.createElement('div');
    card.className = 'kpi-card p-6 bg-white rounded-lg border border-gray-100 shadow-sm';

    const trendHtml = options.change !== undefined ? `
      <div class="flex items-center gap-2 mt-2">
        <span class="px-2 py-0.5 rounded text-xs font-medium ${
          options.change >= 0 ? 'bg-green-50 text-green-700' : 'bg-red-50 text-red-700'
        }">
          ${options.change >= 0 ? '↑' : '↓'} ${Math.abs(options.change)}%
        </span>
        <span class="text-xs text-gray-400">前期比</span>
      </div>
    ` : '';

    card.innerHTML = `
      <p class="text-xs font-medium text-gray-500 uppercase tracking-wider">${label}</p>
      <p class="text-3xl font-bold text-gray-800 mt-2 tracking-tight">
        ${this.formatNumber(value)}${options.unit || ''}
      </p>
      ${trendHtml}
    `;

    this.content.appendChild(card);
    return card;
  }

  addSection(title, content) {
    const section = document.createElement('section');
    section.className = 'mt-8';
    section.innerHTML = `
      <h2 class="text-lg font-semibold text-gray-800 mb-4">${title}</h2>
      <div class="section-content">${content}</div>
    `;
    this.content.appendChild(section);
    return section;
  }

  formatNumber(num) {
    return new Intl.NumberFormat('th-TH').format(num);
  }

  formatDateTime(date) {
    return new Intl.DateTimeFormat('th-TH', {
      year: 'numeric',
      month: 'short',
      day: 'numeric',
      hour: '2-digit',
      minute: '2-digit'
    }).format(date);
  }
}

// Usage
const dashboard = new Dashboard('dashboard-container');
dashboard.addKpiCard('総売上', 1245000, { unit: '฿', change: 12.5 });
dashboard.addKpiCard('注文数', 328, { change: 8.3 });
dashboard.addKpiCard('平均注文額', 3795, { unit: '฿', change: -2.1 });
dashboard.addKpiCard('顧客満足度', 94.2, { unit: '%', change: 5.7 });
```

### Executive Presentation Guidelines

```typescript
// utils/dashboard/presentation.ts
/**
 * Guidelines for presenting data to executives (Japanese style)
 */

export const presentationGuidelines = {
  // The "One Page" principle - Ichi-Gan (一目)
  onePagePrinciple: `
    Executives should understand the entire status at a glance.
    - Top 3-5 KPIs visible without scrolling
    - Traffic light status (green/yellow/red) for quick assessment
    - Trend indicators for every metric
  `,

  // Data density control
  dataDensity: `
    Follow the 60/40 rule:
    - 60% whitespace (Ma - 間)
    - 40% content
    - Group related information in visual chunks
  `,

  // Typography hierarchy
  typography: `
    - Headings: 700 weight, tight letter-spacing
    - Values: 700 weight, large size, no abbreviation
    - Labels: 500 weight, uppercase, wide letter-spacing
    - Body: 400 weight, relaxed line-height (1.6)
  `,

  // Color usage
  colors: `
    - Use 2-3 primary colors maximum
    - Semantic colors only for status indication
    - Gradients should be subtle (10-20% opacity)
    - Avoid pure black (#000) - use dark grays instead
  `,

  // Animation principles
  animation: `
    - Animations should be subtle (200-300ms)
    - Ease-out curves for natural feel
    - No animation for initial load (respect user's time)
    - Hover states: slight scale (1.02) and shadow increase
  `,

  // Mobile responsiveness
  mobile: `
    - Single column on mobile
    - KPIs stack vertically
    - Touch targets minimum 44x44px
    - Charts become full-width cards
  `
};

// Status badge generator
export function getStatusBadge(
  status: 'on-track' | 'at-risk' | 'off-track' | 'neutral'
): string {
  const config = {
    'on-track': { bg: 'bg-green-50', text: 'text-green-700', icon: '●', label: '順調' },
    'at-risk': { bg: 'bg-yellow-50', text: 'text-yellow-700', icon: '◆', label: '要注意' },
    'off-track': { bg: 'bg-red-50', text: 'text-red-700', icon: '●', label: '遅延' },
    'neutral': { bg: 'bg-gray-50', text: 'text-gray-700', icon: '○', label: '-' }
  };

  const { bg, text, icon, label } = config[status];
  return `<span class="${bg} ${text} px-3 py-1 rounded-full text-xs font-medium inline-flex items-center gap-2">
    <span>${icon}</span>
    <span>${label}</span>
  </span>`;
}
```

### Complete Dashboard Layout Template

```html
<!-- templates/dashboard/japanese-executive.html -->
<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Executive Dashboard</title>
  <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+JP:wght@400;500;600;700&family=Noto+Sans+Thai:wght@400;500;600;700&display=swap" rel="stylesheet">
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body {
      font-family: 'Noto Sans Thai', 'Noto Sans JP', sans-serif;
      background: #f7fafc;
      color: #4a5568;
      line-height: 1.6;
    }
    .dashboard-container { max-width: 1400px; margin: 0 auto; padding: 24px; }
    .header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 32px; }
    .header h1 { font-size: 24px; font-weight: 700; color: #1a202c; }
    .header-meta { font-size: 14px; color: #718096; }
    .kpi-grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: 16px; margin-bottom: 32px; }
    .kpi-card { background: white; border-radius: 8px; padding: 20px; border: 1px solid #edf2f7; box-shadow: 0 1px 3px rgba(0,0,0,0.05); }
    .kpi-label { font-size: 11px; font-weight: 500; color: #718096; text-transform: uppercase; letter-spacing: 0.05em; }
    .kpi-value { font-size: 32px; font-weight: 700; color: #1a202c; margin: 8px 0; letter-spacing: -0.03em; }
    .kpi-change { font-size: 12px; }
    .kpi-change.positive { color: #38a169; }
    .kpi-change.negative { color: #e53e3e; }
    .main-grid { display: grid; grid-template-columns: 2fr 1fr; gap: 24px; }
    .card { background: white; border-radius: 8px; border: 1px solid #edf2f7; padding: 24px; box-shadow: 0 1px 3px rgba(0,0,0,0.05); }
    .card-title { font-size: 16px; font-weight: 600; color: #2d3748; margin-bottom: 16px; }
    .chart-container { height: 300px; }
    .table { width: 100%; border-collapse: collapse; font-size: 14px; }
    .table th { text-align: left; padding: 12px; background: #f7fafc; font-weight: 600; color: #4a5568; font-size: 11px; text-transform: uppercase; }
    .table td { padding: 12px; border-bottom: 1px solid #edf2f7; }
    .status-badge { padding: 4px 10px; border-radius: 20px; font-size: 11px; font-weight: 500; }
    .status-badge.on-track { background: #c6f6d5; color: #276749; }
    .status-badge.at-risk { background: #feebc8; color: #9c4221; }
    .status-badge.off-track { background: #fed7d7; color: #c53030; }
    @media (max-width: 1024px) {
      .kpi-grid { grid-template-columns: repeat(2, 1fr); }
      .main-grid { grid-template-columns: 1fr; }
    }
    @media (max-width: 640px) {
      .kpi-grid { grid-template-columns: 1fr; }
    }
  </style>
</head>
<body>
  <div class="dashboard-container">
    <!-- Header -->
    <header class="header">
      <div>
        <h1>ภาพรวมการดำเนินงาน</h1>
        <p class="header-meta">อัปเดตล่าสุด: <span id="last-update"></span></p>
      </div>
      <div class="header-actions">
        <select style="padding: 8px 16px; border: 1px solid #e2e8f0; border-radius: 6px;">
          <option>เดือนนี้</option>
          <option>ไตรมาสนี้</option>
          <option>ปีนี้</option>
        </select>
      </div>
    </header>

    <!-- KPI Cards -->
    <div class="kpi-grid">
      <div class="kpi-card">
        <p class="kpi-label">ยอดขายรวม</p>
        <p class="kpi-value">฿2.4M</p>
        <p class="kpi-change positive">↑ 12.5% จากเดือนก่อน</p>
      </div>
      <div class="kpi-card">
        <p class="kpi-label">จำนวนงาน</p>
        <p class="kpi-value">328</p>
        <p class="kpi-change positive">↑ 8.3% จากเดือนก่อน</p>
      </div>
      <div class="kpi-card">
        <p class="kpi-label">อัตราเสร็จสิ้น</p>
        <p class="kpi-value">94.2%</p>
        <p class="kpi-change positive">↑ 3.1% จากเดือนก่อน</p>
      </div>
      <div class="kpi-card">
        <p class="kpi-label">เวลาเฉลี่ย</p>
        <p class="kpi-value">2.4ชม</p>
        <p class="kpi-change negative">↓ 5.2% จากเดือนก่อน</p>
      </div>
    </div>

    <!-- Main Content -->
    <div class="main-grid">
      <!-- Chart Section -->
      <div class="card">
        <h3 class="card-title">แนวโน้มยอดขาย</h3>
        <div class="chart-container" id="trend-chart"></div>
      </div>

      <!-- Status List -->
      <div class="card">
        <h3 class="card-title">สถานะงานที่กำลังดำเนินการ</h3>
        <table class="table">
          <thead>
            <tr>
              <th>เลขที่</th>
              <th>สถานะ</th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td>JOB-001</td>
              <td><span class="status-badge on-track">ดำเนินการปกติ</span></td>
            </tr>
            <tr>
              <td>JOB-002</td>
              <td><span class="status-badge at-risk">ต้องติดตาม</span></td>
            </tr>
            <tr>
              <td>JOB-003</td>
              <td><span class="status-badge off-track">ล่าช้า</span></td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>

  <script>
    // Update timestamp
    document.getElementById('last-update').textContent = new Date().toLocaleString('th-TH');
  </script>
</body>
</html>
```

### Japanese Business Terminology for Dashboards

```typescript
// constants/dashboard/japanese-business-terms.ts
/**
 * Common Japanese business terms for dashboard labels
 */

export const jpBusinessTerms = {
  // Time periods
  periods: {
    today: '本日',
    thisWeek: '今週',
    thisMonth: '今月',
    thisQuarter: '今四半期',
    thisYear: '今年度',
    ytd: '年初来',
    lastPeriod: '前期'
  },

  // Status indicators
  status: {
    onTrack: '順調',
    atRisk: '要注意',
    offTrack: '遅延',
    completed: '完了',
    inProgress: '進行中',
    pending: '保留中',
    cancelled: 'キャンセル'
  },

  // Common metrics
  metrics: {
    revenue: '売上',
    profit: '利益',
    orders: '注文数',
    customers: '顧客数',
    satisfaction: '顧客満足度',
    efficiency: '効率',
    quality: '品質',
    delivery: '納期',
    cost: 'コスト',
    inventory: '在庫'
  },

  // Trend indicators
  trends: {
    increasing: '増加',
    decreasing: '減少',
    stable: '横ばい',
    volatile: '変動'
  },

  // Action items
  actions: {
    review: '要確認',
    approve: '承認待ち',
    urgent: '重要',
    info: '参考'
  }
};

// Thai equivalents for bilingual dashboards
export const thBusinessTerms = {
  periods: {
    today: 'วันนี้',
    thisWeek: 'สัปดาห์นี้',
    thisMonth: 'เดือนนี้',
    thisQuarter: 'ไตรมาสนี้',
    thisYear: 'ปีนี้',
    ytd: 'ตั้งแต่ต้นปี',
    lastPeriod: 'งวดก่อน'
  },
  status: {
    onTrack: 'ดำเนินการปกติ',
    atRisk: 'ต้องติดตาม',
    offTrack: 'ล่าช้า',
    completed: 'เสร็จสิ้น',
    inProgress: 'กำลังดำเนินการ',
    pending: 'รอดำเนินการ',
    cancelled: 'ยกเลิก'
  }
};
```

---

# PART III: RESOURCES

### Server-Side Pagination

```typescript
// utils/pagination.ts
/**
 * Server-side pagination utilities
 */

export interface PaginationOptions {
  page: number;
  limit: number;
  total: number;
}

export interface PaginatedResult<T> {
  data: T[];
  pagination: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
    hasNext: boolean;
    hasPrev: boolean;
  };
}

export function paginate<T>(
  items: T[],
  options: PaginationOptions
): PaginatedResult<T> {
  const { page, limit, total } = options;
  const totalPages = Math.ceil(total / limit);
  const offset = (page - 1) * limit;

  return {
    data: items,
    pagination: {
      page,
      limit,
      total,
      totalPages,
      hasNext: page < totalPages,
      hasPrev: page > 1
    }
  };
}

// Supabase pagination query
export async function paginatedQuery<T>(
  supabase: any,
  table: string,
  options: {
    page?: number;
    limit?: number;
    filters?: Record<string, any>;
    orderBy?: { column: string; ascending?: boolean };
  }
): Promise<PaginatedResult<T>> {
  const page = options.page || 1;
  const limit = options.limit || 20;
  const offset = (page - 1) * limit;

  // Get count first
  const { count } = await supabase
    .from(table)
    .select('*', { count: 'exact', head: true });

  // Get data
  let query = supabase
    .from(table)
    .select('*')
    .range(offset, offset + limit - 1);

  if (options.filters) {
    Object.entries(options.filters).forEach(([key, value]) => {
      if (value !== undefined && value !== null) {
        query = query.eq(key, value);
      }
    });
  }

  if (options.orderBy) {
    query = query.order(options.orderBy.column, {
      ascending: options.orderBy.ascending ?? true
    });
  }

  const { data, error } = await query;

  if (error) {
    throw error;
  }

  return paginate(data || [], { page, limit, total: count || 0 });
}

// Pagination metadata for headers
export function setPaginationHeaders(
  response: Response,
  pagination: PaginationOptions
): Response {
  const totalPages = Math.ceil(pagination.total / pagination.limit);

  response.headers.set('X-Page', pagination.page.toString());
  response.headers.set('X-Limit', pagination.limit.toString());
  response.headers.set('X-Total', pagination.total.toString());
  response.headers.set('X-Total-Pages', totalPages.toString());

  return response;
}
```

### Advanced Filtering & Sorting

```typescript
// utils/filtering.ts
/**
 * Advanced filtering and sorting for admin dashboards
 */

export interface FilterConfig {
  field: string;
  operator: 'eq' | 'neq' | 'gt' | 'gte' | 'lt' | 'lte' | 'like' | 'in' | 'between';
  value?: any;
}

export interface SortConfig {
  field: string;
  direction: 'asc' | 'desc';
}

export interface QueryOptions {
  filters?: FilterConfig[];
  sort?: SortConfig[];
  page?: number;
  limit?: number;
}

// Build Supabase query from filter options
export function buildSupabaseQuery(
  supabase: any,
  table: string,
  options: QueryOptions
) {
  let query = supabase.from(table).select('*');

  // Apply filters
  if (options.filters) {
    for (const filter of options.filters) {
      if (filter.value === undefined || filter.value === null) continue;

      switch (filter.operator) {
        case 'eq':
          query = query.eq(filter.field, filter.value);
          break;
        case 'neq':
          query = query.neq(filter.field, filter.value);
          break;
        case 'gt':
          query = query.gt(filter.field, filter.value);
          break;
        case 'gte':
          query = query.gte(filter.field, filter.value);
          break;
        case 'lt':
          query = query.lt(filter.field, filter.value);
          break;
        case 'lte':
          query = query.lte(filter.field, filter.value);
          break;
        case 'like':
          query = query.like(filter.field, `%${filter.value}%`);
          break;
        case 'in':
          query = query.in(filter.field, filter.value);
          break;
        case 'between':
          query = query.gte(filter.field, filter.value[0])
                     .lte(filter.field, filter.value[1]);
          break;
      }
    }
  }

  // Apply sorting
  if (options.sort) {
    for (const sort of options.sort) {
      query = query.order(sort.field, { ascending: sort.direction === 'asc' });
    }
  }

  return query;
}

// Parse filter string from URL params
export function parseFilters(params: URLSearchParams): FilterConfig[] {
  const filters: FilterConfig[] = [];

  for (const [key, value] of params.entries()) {
    if (!key.startsWith('filter[')) continue;

    const match = key.match(/filter\[([^\]]+)\](?:\[(.+?)\])?/);
    if (!match) continue;

    const [, field, operator] = match;
    const op = operator || 'eq';

    filters.push({
      field,
      operator: op as FilterConfig['operator'],
      value: decodeURIComponent(value)
    });
  }

  return filters;
}

// Example URL parsing:
// ?filter[status]=active&filter[created_at][gte]=2024-01-01&sort[created_at]=desc&page=1
export function parseQueryOptions(params: URLSearchParams): QueryOptions {
  const options: QueryOptions = {
    filters: parseFilters(params),
    sort: parseSorts(params),
    page: parseInt(params.get('page') || '1'),
    limit: parseInt(params.get('limit') || '20')
  };

  return options;
}

function parseSorts(params: URLSearchParams): SortConfig[] {
  const sorts: SortConfig[] = [];

  for (const [key, value] of params.entries()) {
    if (!key.startsWith('sort[')) continue;

    const match = key.match(/sort\[([^\]]+)\]/);
    if (!match) continue;

    sorts.push({
      field: match[1],
      direction: value as 'asc' | 'desc'
    });
  }

  return sorts;
}
```

### Export to Excel/CSV (SheetJS)

```typescript
// utils/export.ts
/**
 * Export utilities using SheetJS (xlsx)
 */

import * as XLSX from 'xlsx';

export interface ExportColumn {
  key: string;
  label: string;
  format?: (value: any) => string;
}

export interface ExportOptions {
  filename: string;
  format: 'xlsx' | 'csv';
  columns: ExportColumn[];
  data: any[];
}

// Export data to Excel or CSV
export function exportData(options: ExportOptions): void {
  // Transform data with column config
  const exportData = options.data.map(row => {
    const obj: Record<string, any> = {};
    options.columns.forEach(col => {
      const value = row[col.key];
      obj[col.label] = col.format ? col.format(value) : value;
    });
    return obj;
  });

  // Create worksheet
  const worksheet = XLSX.utils.json_to_sheet(exportData);

  // Create workbook
  const workbook = XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(workbook, worksheet, 'Sheet1');

  // Generate filename with timestamp
  const timestamp = new Date().toISOString().slice(0, 10);
  const filename = `${options.filename}_${timestamp}.${options.format}`;

  // Download file
  XLSX.writeFile(workbook, filename);
}

// Thai date formatter for export
export function thaiDateFormatter(value: string | Date): string {
  if (!value) return '-';
  const date = typeof value === 'string' ? new Date(value) : value;
  return date.toLocaleDateString('th-TH', {
    year: 'numeric',
    month: '2-digit',
    day: '2-digit',
    hour: '2-digit',
    minute: '2-digit'
  });
}

// Status formatter
export function statusFormatter(value: string, translations: Record<string, string>): string {
  return translations[value] || value;
}

// Example usage for job export
export function exportJobs(jobs: any[]): void {
  exportData({
    filename: 'jobs',
    format: 'xlsx',
    columns: [
      { key: 'ref', label: 'เลขที่งาน' },
      { key: 'driver_name', label: 'พนักงานขับรถ' },
      { key: 'origin_name', label: 'ต้นทาง' },
      { key: 'destination_address', label: 'ปลายทาง' },
      { key: 'status', label: 'สถานะ', format: (v) => statusFormatter(v, {
        pending: 'รอดำเนินการ',
        active: 'กำลังดำเนินการ',
        completed: 'เสร็จสิ้น',
        cancelled: 'ยกเลิก'
      })},
      { key: 'created_at', label: 'วันที่สร้าง', format: thaiDateFormatter },
      { key: 'completed_at', label: 'วันที่เสร็จสิ้น', format: thaiDateFormatter }
    ],
    data: jobs
  });
}
```

### Print Templates

```typescript
// utils/print.ts
/**
 * Print utilities for documents and receipts
 */

export interface PrintOptions {
  title: string;
  orientation?: 'portrait' | 'landscape';
  onBeforePrint?: () => void;
  onAfterPrint?: () => void;
}

// Generic print function
export function printContent(html: string, options: PrintOptions): void {
  const printWindow = window.open('', '_blank');

  if (!printWindow) {
    alert('Cannot open print window. Please allow popups.');
    return;
  }

  // Write HTML to print window
  printWindow.document.write(`
    <!DOCTYPE html>
    <html>
    <head>
      <meta charset="UTF-8">
      <title>${options.title}</title>
      <style>
        @media print {
          @page { ${options.orientation ? `size: A4 ${options.orientation};` : ''} }
          body { font-family: 'Sarabun', sans-serif; }
          .no-print { display: none; }
          table { width: 100%; border-collapse: collapse; }
          th, td { border: 1px solid #000; padding: 8px; text-align: left; }
          th { background: #f0f0f0; }
          .text-right { text-align: right; }
          .text-center { text-align: center; }
          .bold { font-weight: bold; }
        }
        @media screen {
          body { padding: 20px; }
          .print-btn { margin: 20px 0; }
        }
      </style>
      <link href="https://fonts.googleapis.com/css2?family=Sarabun:wght@400;700&display=swap" rel="stylesheet">
    </head>
    <body>
      ${html}
      <script>
        window.onload = function() {
          window.print();
          window.onafterprint = function() { window.close(); };
        };
      </script>
    </body>
    </html>
  `);

  printWindow.document.close();
}

// Job receipt template
export function printJobReceipt(job: any, driver: any): void {
  const html = `
    <div style="max-width: 800px; margin: 0 auto;">
      <div class="text-center">
        <h1>ใบรับงานส่งน้ำมัน</h1>
        <h2>DriverConnect</h2>
      </div>

      <table style="margin: 20px 0;">
        <tr>
          <th colspan="2" class="text-center">ข้อมูลงาน</th>
        </tr>
        <tr>
          <td><strong>เลขที่งาน:</strong></td>
          <td>${job.ref}</td>
        </tr>
        <tr>
          <td><strong>วันที่:</strong></td>
          <td>${thaiDateFormatter(job.created_at)}</td>
        </tr>
        <tr>
          <td><strong>พนักงานขับรถ:</strong></td>
          <td>${driver.display_name}</td>
        </tr>
        <tr>
          <td><strong>ทะเบียนรถ:</strong></td>
          <td>${driver.vehicle_plate || '-'}</td>
        </tr>
      </table>

      <table style="margin: 20px 0;">
        <tr>
          <th colspan="2" class="text-center">เส้นทาง</th>
        </tr>
        <tr>
          <td><strong>ต้นทาง:</strong></td>
          <td>${job.origin_name}</td>
        </tr>
        <tr>
          <td><strong>ปลายทาง:</strong></td>
          <td>${job.destination_address}</td>
        </tr>
      </table>

      ${job.checkin_time ? `
      <table style="margin: 20px 0;">
        <tr>
          <th colspan="2" class="text-center">เวลา</th>
        </tr>
        <tr>
          <td><strong>เช็คอิน:</strong></td>
          <td>${thaiDateFormatter(job.checkin_time)}</td>
        </tr>
        <tr>
          <td><strong>เช็คเอาท์:</strong></td>
          <td>${job.checkout_time ? thaiDateFormatter(job.checkout_time) : '-'}</td>
        </tr>
      </table>
      ` : ''}

      <div style="margin-top: 40px;">
        <table>
          <tr>
            <td class="text-center" style="width: 50%;">
              <div style="border-top: 1px solid #000; padding-top: 10px;">
                ผู้ส่งสินค้า<br>
               วันที่ ....../....../......
              </div>
            </td>
            <td class="text-center" style="width: 50%;">
              <div style="border-top: 1px solid #000; padding-top: 10px;">
                ผู้รับสินค้า<br>
                วันที่ ....../....../......
              </div>
            </td>
          </tr>
        </table>
      </div>
    </div>
  `;

  printContent(html, {
    title: `ใบรับงาน ${job.ref}`,
    orientation: 'portrait'
  });
}
```

## Error Handling & Monitoring

### Centralized Error Handling

```typescript
// utils/errors.ts
/**
 * Centralized error handling system
 */

// Custom error classes
export class AppError extends Error {
  constructor(
    message: string,
    public code: string,
    public statusCode: number = 500,
    public details?: Record<string, any>
  ) {
    super(message);
    this.name = 'AppError';
    Error.captureStackTrace(this, this.constructor);
  }
}

export class ValidationError extends AppError {
  constructor(message: string, details?: Record<string, any>) {
    super(message, 'VALIDATION_ERROR', 400, details);
    this.name = 'ValidationError';
  }
}

export class NotFoundError extends AppError {
  constructor(resource: string, id?: string) {
    super(
      id ? `${resource} with id ${id} not found` : `${resource} not found`,
      'NOT_FOUND',
      404
    );
    this.name = 'NotFoundError';
  }
}

export class UnauthorizedError extends AppError {
  constructor(message = 'Unauthorized access') {
    super(message, 'UNAUTHORIZED', 401);
    this.name = 'UnauthorizedError';
  }
}

export class ForbiddenError extends AppError {
  constructor(message = 'Access forbidden') {
    super(message, 'FORBIDDEN', 403);
    this.name = 'ForbiddenError';
  }
}

export class RateLimitError extends AppError {
  constructor(retryAfter?: number) {
    super(
      'Rate limit exceeded',
      'RATE_LIMIT_EXCEEDED',
      429,
      retryAfter ? { retryAfter } : undefined
    );
    this.name = 'RateLimitError';
  }
}

// Error handler middleware
export function errorHandler(error: Error, req: Request): Response {
  console.error('[Error]', error);

  if (error instanceof AppError) {
    return new Response(JSON.stringify({
      error: {
        message: error.message,
        code: error.code,
        details: error.details
      }
    }), {
      status: error.statusCode,
      headers: { 'Content-Type': 'application/json' }
    });
  }

  // Unknown error
  return new Response(JSON.stringify({
    error: {
      message: 'Internal server error',
      code: 'INTERNAL_ERROR'
    }
  }), {
    status: 500,
    headers: { 'Content-Type': 'application/json' }
  });
}

// Async wrapper
export function asyncHandler<T>(
  fn: () => Promise<T>
): Promise<T> {
  return fn().catch((error) => {
    throw error instanceof AppError ? error : new AppError(
      error.message || 'An error occurred',
      'INTERNAL_ERROR',
      500
    );
  });
}
```

### Sentry Integration

```typescript
// utils/sentry.ts
/**
 * Sentry error tracking integration
 */

import * as Sentry from 'https://deno.land/x/sentry@8.4.0/index.ts';

export function initSentry(dsn: string, environment: string) {
  Sentry.init({
    dsn,
    environment,
    // Sample rate for transactions (performance)
    tracesSampleRate: environment === 'production' ? 0.1 : 1.0,
    // Sample rate for sessions (replay)
    replaysSessionSampleRate: 0.1,
    replaysOnErrorSampleRate: 1.0
  });
}

// Set user context
export function setSentryUser(user: {
  id: string;
  email?: string;
  username?: string;
}) {
  Sentry.setUser(user);
}

// Clear user context
export function clearSentryUser() {
  Sentry.setUser(null);
}

// Add breadcrumb
export function addBreadcrumb(
  message: string,
  category?: string,
  level: 'log' | 'info' | 'warning' | 'error' = 'info'
) {
  Sentry.addBreadcrumb({
    message,
    category,
    level
  });
}

// Capture exception
export function captureException(error: Error, context?: Record<string, any>) {
  if (context) {
    Sentry.setContext('additional', context);
  }
  Sentry.captureException(error);
}

// Capture message
export function captureMessage(
  message: string,
  level: 'info' | 'warning' | 'error' = 'info'
) {
  Sentry.captureMessage(message, level);
}

// Performance monitoring
export function startTransaction(name: string, op: string) {
  return Sentry.startSpan({ name, op }, (span) => span);
}
```

### Supabase Log Viewer Patterns

```typescript
// utils/supabase-logger.ts
/**
 * Logging utility for Supabase
 */

export enum LogLevel {
  DEBUG = 'debug',
  INFO = 'info',
  WARN = 'warn',
  ERROR = 'error'
}

export interface LogEntry {
  level: LogLevel;
  message: string;
  userId?: string;
  context?: Record<string, any>;
  timestamp: string;
}

export class Logger {
  constructor(
    private supabase: any,
    private userId?: string
  ) {}

  private async log(entry: LogEntry): Promise<void> {
    // Console output for development
    if (import.meta.env.DEV) {
      const style = {
        debug: 'color: gray',
        info: 'color: blue',
        warn: 'color: orange',
        error: 'color: red'
      };
      console.log(
        `%c[${entry.level.toUpperCase()}] ${entry.message}`,
        style[entry.level],
        entry.context || ''
      );
    }

    // Database log
    try {
      await this.supabase.from('app_logs').insert({
        level: entry.level,
        message: entry.message,
        user_id: this.userId,
        context: entry.context,
        created_at: entry.timestamp
      });
    } catch (error) {
      console.error('Failed to write log:', error);
    }
  }

  debug(message: string, context?: Record<string, any>): void {
    this.log({
      level: LogLevel.DEBUG,
      message,
      userId: this.userId,
      context,
      timestamp: new Date().toISOString()
    });
  }

  info(message: string, context?: Record<string, any>): void {
    this.log({
      level: LogLevel.INFO,
      message,
      userId: this.userId,
      context,
      timestamp: new Date().toISOString()
    });
  }

  warn(message: string, context?: Record<string, any>): void {
    this.log({
      level: LogLevel.WARN,
      message,
      userId: this.userId,
      context,
      timestamp: new Date().toISOString()
    });
  }

  error(message: string, error?: Error | unknown, context?: Record<string, any>): void {
    const errorContext = {
      ...context,
      error: error instanceof Error ? {
        name: error.name,
        message: error.message,
        stack: error.stack
      } : error
    };

    this.log({
      level: LogLevel.ERROR,
      message,
      userId: this.userId,
      context: errorContext,
      timestamp: new Date().toISOString()
    });
  }
}

// Query logs for admin
export async function queryLogs(
  supabase: any,
  options: {
    level?: LogLevel;
    userId?: string;
    startDate?: Date;
    endDate?: Date;
    limit?: number;
  }
): Promise<LogEntry[]> {
  let query = supabase
    .from('app_logs')
    .select('*')
    .order('created_at', { ascending: false });

  if (options.level) {
    query = query.eq('level', options.level);
  }

  if (options.userId) {
    query = query.eq('user_id', options.userId);
  }

  if (options.startDate) {
    query = query.gte('created_at', options.startDate.toISOString());
  }

  if (options.endDate) {
    query = query.lte('created_at', options.endDate.toISOString());
  }

  if (options.limit) {
    query = query.limit(options.limit);
  }

  const { data, error } = await query;

  if (error) {
    throw error;
  }

  return data || [];
}
```

## Performance

### Debounce & Throttle

```typescript
// utils/performance.ts
/**
 * Performance utilities: debounce, throttle, memoize
 */

// Debounce - delay execution until after wait time has elapsed
export function debounce<T extends (...args: any[]) => any>(
  func: T,
  wait: number
): (...args: Parameters<T>) => void {
  let timeout: ReturnType<typeof setTimeout> | null = null;

  return function executedFunction(...args: Parameters<T>) {
    const later = () => {
      timeout = null;
      func(...args);
    };

    if (timeout) {
      clearTimeout(timeout);
    }

    timeout = setTimeout(later, watch);
  };
}

// Throttle - limit execution rate
export function throttle<T extends (...args: any[]) => any>(
  func: T,
  limit: number
): (...args: Parameters<T>) => void {
  let inThrottle: boolean;

  return function executedFunction(...args: Parameters<T>) {
    if (!inThrottle) {
      func(...args);
      inThrottle = true;
      setTimeout(() => inThrottle = false, limit);
    }
  };
}

// Debounce with immediate option
export function debounceImmediate<T extends (...args: any[]) => any>(
  func: T,
  wait: number,
  immediate = false
): (...args: Parameters<T>) => void {
  let timeout: ReturnType<typeof setTimeout> | null = null;

  return function executedFunction(...args: Parameters<T>) {
    const callNow = immediate && !timeout;

    if (timeout) {
      clearTimeout(timeout);
    }

    timeout = setTimeout(() => {
      timeout = null;
      if (!immediate) {
        func(...args);
      }
    }, wait);

    if (callNow) {
      func(...args);
    }
  };
}

// RequestAnimationFrame throttle
export function rafThrottle<T extends (...args: any[]) => any>(
  func: T
): (...args: Parameters<T>) => void {
  let rafId: number | null = null;

  return function executedFunction(...args: Parameters<T>) {
    if (rafId !== null) {
      cancelAnimationFrame(rafId);
    }

    rafId = requestAnimationFrame(() => {
      func(...args);
      rafId = null;
    });
  };
}
```

### Image Optimization (Supabase)

```typescript
// utils/images.ts
/**
 * Image optimization using Supabase Transformations
 */

const SUPABASE_URL = 'YOUR_SUPABASE_URL';
const STORAGE_BUCKET = 'images';

// Generate optimized image URL
export function getOptimizedImageUrl(
  path: string,
  options: {
    width?: number;
    height?: number;
    quality?: number;
    format?: 'origin' | 'resize' | 'cover';
  } = {}
): string {
  const url = new URL(`${SUPABASE_URL}/storage/v1/object/public/${STORAGE_BUCKET}/${path}`);

  if (options.width) {
    url.searchParams.set('width', options.width.toString());
  }

  if (options.height) {
    url.searchParams.set('height', options.height.toString());
  }

  if (options.quality) {
    url.searchParams.set('quality', options.quality.toString());
  }

  if (options.format) {
    url.searchParams.set('resize', options.format);
  }

  return url.toString();
}

// Generate responsive image srcset
export function generateSrcSet(
  path: string,
  sizes: number[],
  quality = 80
): string {
  return sizes
    .map(size => `${getOptimizedImageUrl(path, { width: size, quality })} ${size}w`)
    .join(', ');
}

// Lazy load images
export function lazyLoadImages(): void {
  if ('IntersectionObserver' in window) {
    const observer = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          const img = entry.target as HTMLImageElement;
          const src = img.dataset.src;

          if (src) {
            img.src = src;
            img.removeAttribute('data-src');
            observer.unobserve(img);
          }
        }
      });
    });

    document.querySelectorAll('img[data-src]').forEach(img => {
      observer.observe(img);
    });
  }
}

// Image component wrapper
export class LazyImage {
  constructor(
    private element: HTMLImageElement,
    private src: string,
    private placeholder?: string
  ) {
    this.init();
  }

  private init(): void {
    this.element.dataset.src = this.src;
    if (this.placeholder) {
      this.element.src = this.placeholder;
    }
    this.element.loading = 'lazy';
  }
}
```

### Lazy Loading

```typescript
// utils/lazy-load.ts
/**
 * Lazy loading utilities for components and data
 */

// Lazy load component
export function lazyComponent<T extends HTMLElement>(
  selector: string,
  callback: (element: T) => void
): void {
  const observer = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        const element = entry.target as T;
        callback(element);
        observer.unobserve(element);
      }
    });
  }, { rootMargin: '50px' });

  document.querySelectorAll<T>(selector).forEach(el => {
    observer.observe(el);
  });
}

// Infinite scroll
export function infiniteScroll(
  callback: () => void | Promise<void>,
  options: { threshold?: number } = {}
): () => void {
  const { threshold = 100 } = options;
  let loading = false;

  const handleScroll = throttle(async () => {
    if (loading) return;

    const scrollTop = window.pageYOffset || document.documentElement.scrollTop;
    const windowHeight = window.innerHeight;
    const documentHeight = document.documentElement.scrollHeight;

    if (scrollTop + windowHeight >= documentHeight - threshold) {
      loading = true;
      await callback();
      loading = false;
    }
  }, 200);

  window.addEventListener('scroll', handleScroll);
  handleScroll(); // Initial check

  return () => window.removeEventListener('scroll', handleScroll);
}

// Lazy load script
export function lazyLoadScript(
  src: string,
  options: { async?: boolean; defer?: boolean } = {}
): Promise<void> {
  return new Promise((resolve, reject) => {
    // Check if already loaded
    if (document.querySelector(`script[src="${src}"]`)) {
      resolve();
      return;
    }

    const script = document.createElement('script');
    script.src = src;
    script.async = options.async ?? true;
    script.defer = options.defer ?? false;

    script.onload = () => resolve();
    script.onerror = () => reject(new Error(`Failed to load script: ${src}`));

    document.head.appendChild(script);
  });
}
```

### Code Splitting Patterns

```typescript
// utils/code-splitting.ts
/**
 * Code splitting utilities
 */

// Dynamic import with retry
export async function dynamicImport<T>(
  importFn: () => Promise<T>,
  retries = 3
): Promise<T> {
  try {
    return await importFn();
  } catch (error) {
    if (retries <= 0) {
      throw error;
    }

    // Exponential backoff
    await new Promise(resolve => setTimeout(resolve, (4 - retries) * 1000));
    return dynamicImport(importFn, retries - 1);
  }
}

// Lazy load module with fallback
export function lazyLoadModule<T>(
  importFn: () => Promise<T>,
  fallback?: T
): Promise<T> {
  return dynamicImport(importFn).catch(error => {
    console.error('Module load failed:', error);
    if (fallback) {
      return Promise.resolve(fallback);
    }
    throw error;
  });
}

// Preload module
export function preloadModule(modulePath: string): void {
  const link = document.createElement('link');
  link.rel = 'modulepreload';
  link.href = modulePath;
  document.head.appendChild(link);
}

// Prefetch module
export function prefetchModule(modulePath: string): void {
  const link = document.createElement('link');
  link.rel = 'prefetch';
  link.href = modulePath;
  document.head.appendChild(link);
}
```

## Testing

### Supabase Test Fixtures

```typescript
// tests/fixtures/supabase.ts
/**
 * Supabase test fixtures for E2E and integration tests
 */

import { createClient } from '@supabase/supabase-js';

const testSupabaseUrl = process.env.SUPABASE_TEST_URL!;
const testSupabaseKey = process.env.SUPABASE_TEST_ANON_KEY!;

export const testSupabase = createClient(testSupabaseUrl, testSupabaseKey);

// Test user factory
export async function createTestUser(overrides = {}) {
  const { data, error } = await testSupabase.auth.signUp({
    email: `test-${Date.now()}@example.com`,
    password: 'test-password-123',
    ...overrides
  });

  if (error) throw error;

  return data.user;
}

// Clean up test data
export async function cleanupTestData(userId: string) {
  await testSupabase.from('jobdata').delete().eq('driver_id', userId);
  await testSupabase.from('user_profiles').delete().eq('id', userId);
  await testSupabase.auth.admin.deleteUser(userId);
}

// Job fixture factory
export function createJobFixture(overrides = {}) {
  return {
    ref: `TEST-${Date.now()}`,
    origin_id: crypto.randomUUID(),
    destination_lat: 13.7563,
    destination_lng: 100.5018,
    destination_address: '123 Test Street',
    status: 'pending',
    created_at: new Date().toISOString(),
    ...overrides
  };
}
```

### Mock Webhook Testing

```typescript
// tests/mocks/line-webhook.ts
/**
 * Mock LINE webhook for testing
 */

export interface MockWebhookEvent {
  type: 'message' | 'follow' | 'postback' | 'unfollow';
  source?: { userId: string };
  replyToken?: string;
  message?: { type: string; text: string };
  postback?: { data: string };
}

export function createMockWebhook(events: MockWebhookEvent[]): {
  body: string;
  signature: string;
} {
  const payload = { events };

  const body = JSON.stringify(payload);
  const signature = Buffer.from(body).toString('base64');

  return { body, signature };
}

// Mock webhook request
export function mockWebhookRequest(events: MockWebhookEvent[]): Request {
  const { body, signature } = createMockWebhook(events);

  return new Request('https://example.com/webhook/line', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'X-Line-Signature': signature
    },
    body
  });
}
```

### Location Service Testing

```typescript
// tests/location.test.ts
/**
 * Location service tests
 */

import { describe, it, expect } from 'vitest';
import { haversineDistance, isWithinRadius } from '../utils/distance';

describe('Distance calculations', () => {
  it('should calculate distance between two points', () => {
    // Bangkok: 13.7563, 100.5018
    // Chiang Mai: 18.7883, 98.9853
    const distance = haversineDistance(13.7563, 100.5018, 18.7883, 98.9853);

    // Distance should be approximately 695 km
    expect(distance).toBeGreaterThan(690000);
    expect(distance).toBeLessThan(700000);
  });

  it('should detect if point is within radius', () => {
    const centerLat = 13.7563;
    const centerLng = 100.5018;

    // Point within 100 meters
    const closeLat = 13.7564;
    const closeLng = 100.5019;

    expect(isWithinRadius(centerLat, centerLng, closeLat, closeLng, 100)).toBe(true);

    // Point outside 100 meters
    const farLat = 13.7600;
    const farLng = 100.5100;

    expect(isWithinRadius(centerLat, centerLng, farLat, farLng, 100)).toBe(false);
  });

  it('should handle identical coordinates', () => {
    const distance = haversineDistance(13.7563, 100.5018, 13.7563, 100.5018);
    expect(distance).toBe(0);
  });
});
```

## API Design Patterns

### RESTful Endpoint Structure

```typescript
// API endpoint conventions for DriverConnect
/**
 * Base URL: /api/v1
 *
 * Resource naming:
 * - Plural nouns for collections: /jobs, /drivers, /stations
 * - Singular nouns for single resource: /jobs/{id}
 *
 * HTTP Methods:
 * - GET: Retrieve resource(s)
 * - POST: Create new resource
 * - PUT: Update entire resource
 * - PATCH: Partial update
 * - DELETE: Remove resource
 *
 * Response format:
 * - Success: { data: ..., meta: {...} }
 * - Error: { error: { message, code, details } }
 */

// Standard API response
export interface ApiResponse<T> {
  data: T;
  meta?: {
    page?: number;
    limit?: number;
    total?: number;
    totalPages?: number;
  };
}

// Error response
export interface ApiError {
  error: {
    message: string;
    code: string;
    details?: Record<string, any>;
  };
}

// RESTful resource endpoints (example)
const ENDPOINTS = {
  // Jobs
  jobs: '/api/v1/jobs',
  job: (id: string) => `/api/v1/jobs/${id}`,
  jobStops: (id: string) => `/api/v1/jobs/${id}/stops`,
  jobCheckin: (id: string) => `/api/v1/jobs/${id}/checkin`,
  jobCheckout: (id: string) => `/api/v1/jobs/${id}/checkout`,

  // Drivers
  drivers: '/api/v1/drivers',
  driver: (id: string) => `/api/v1/drivers/${id}`,
  driverStatus: (id: string) => `/api/v1/drivers/${id}/status`,
  driverLocation: (id: string) => `/api/v1/drivers/${id}/location`,

  // Stations
  stations: '/api/v1/stations',
  station: (id: string) => `/api/v1/stations/${id}`,

  // Reports
  dailyReport: '/api/v1/reports/daily',
  driverReport: (id: string, period: string) => `/api/v1/reports/drivers/${id}/${period}`
};
```

### Standard HTTP Status Codes

```typescript
// HTTP status code helpers
export const HttpStatus = {
  // 2xx Success
  OK: 200,
  CREATED: 201,
  ACCEPTED: 202,
  NO_CONTENT: 204,

  // 3xx Redirection
  MOVED_PERMANENTLY: 301,
  FOUND: 302,
  NOT_MODIFIED: 304,

  // 4xx Client Error
  BAD_REQUEST: 400,
  UNAUTHORIZED: 401,
  FORBIDDEN: 403,
  NOT_FOUND: 404,
  METHOD_NOT_ALLOWED: 405,
  CONFLICT: 409,
  UNPROCESSABLE_ENTITY: 422,
  TOO_MANY_REQUESTS: 429,

  // 5xx Server Error
  INTERNAL_SERVER_ERROR: 500,
  NOT_IMPLEMENTED: 501,
  SERVICE_UNAVAILABLE: 503
};

// Error response factory
export function errorResponse(
  message: string,
  code: string,
  status: number = HttpStatus.BAD_REQUEST,
  details?: Record<string, any>
): { status: number; body: ApiError } {
  return {
    status,
    body: {
      error: {
        message,
        code,
        details
      }
    }
  };
}

// Common error responses
export const Errors = {
  unauthorized: () => errorResponse('Authentication required', 'UNAUTHORIZED', 401),
  forbidden: () => errorResponse('Access forbidden', 'FORBIDDEN', 403),
  notFound: (resource: string) => errorResponse(`${resource} not found`, 'NOT_FOUND', 404),
  validation: (details: Record<string, any>) =>
    errorResponse('Validation failed', 'VALIDATION_ERROR', 400, details),
  conflict: (message: string) => errorResponse(message, 'CONFLICT', 409),
  rateLimited: (retryAfter: number) =>
    errorResponse('Rate limit exceeded', 'RATE_LIMITED', 429, { retryAfter }),
  serverError: () => errorResponse('Internal server error', 'INTERNAL_ERROR', 500)
};
```

### API Client with Fetch

```typescript
// utils/api-client.ts
/**
 * Type-safe API client with interceptors
 */

interface ApiClientConfig {
  baseUrl: string;
  getAuthToken?: () => string | Promise<string>;
  onAuthError?: () => void;
}

export class ApiClient {
  constructor(private config: ApiClientConfig) {}

  private async request<T>(
    endpoint: string,
    options: RequestInit = {}
  ): Promise<T> {
    const url = `${this.config.baseUrl}${endpoint}`;

    // Add auth header
    const headers: HeadersInit = {
      'Content-Type': 'application/json',
      ...options.headers
    };

    if (this.config.getAuthToken) {
      const token = await this.config.getAuthToken();
      if (token) {
        headers['Authorization'] = `Bearer ${token}`;
      }
    }

    const response = await fetch(url, {
      ...options,
      headers
    });

    // Handle 401 Unauthorized
    if (response.status === 401 && this.config.onAuthError) {
      this.config.onAuthError();
    }

    // Handle non-JSON responses
    if (response.status === 204) {
      return undefined as T;
    }

    const data = await response.json();

    // Handle error responses
    if (!response.ok) {
      throw new ApiError(
        data.error?.message || 'Request failed',
        data.error?.code || 'API_ERROR',
        response.status,
        data.error?.details
      );
    }

    return data.data || data;
  }

  async get<T>(endpoint: string, params?: Record<string, any>): Promise<T> {
    const query = params ? '?' + new URLSearchParams(params).toString() : '';
    return this.request<T>(endpoint + query, { method: 'GET' });
  }

  async post<T>(endpoint: string, body?: any): Promise<T> {
    return this.request<T>(endpoint, {
      method: 'POST',
      body: body ? JSON.stringify(body) : undefined
    });
  }

  async put<T>(endpoint: string, body?: any): Promise<T> {
    return this.request<T>(endpoint, {
      method: 'PUT',
      body: body ? JSON.stringify(body) : undefined
    });
  }

  async patch<T>(endpoint: string, body?: any): Promise<T> {
    return this.request<T>(endpoint, {
      method: 'PATCH',
      body: body ? JSON.stringify(body) : undefined
    });
  }

  async delete<T>(endpoint: string): Promise<T> {
    return this.request<T>(endpoint, { method: 'DELETE' });
  }
}

// Custom error class
export class ApiError extends Error {
  constructor(
    message: string,
    public code: string,
    public status: number,
    public details?: Record<string, any>
  ) {
    super(message);
    this.name = 'ApiError';
  }
}
```

## State Management (Vanilla JS)

### Simple Store Pattern

```typescript
// utils/store.ts
/**
 * Simple state management for vanilla JS apps
 * Inspired by Redux but simplified
 */

type Listener = () => void;
type Middleware = (store: Store<any>) => (next: () => void) => (action: Action) => void;

interface Action {
  type: string;
  payload?: any;
}

interface State {
  [key: string]: any;
}

export class Store<T extends State> {
  private state: T;
  private listeners: Set<Listener> = new Set();
  private middleware: Middleware[] = [];

  constructor(
    initialState: T,
    private reducer: (state: T, action: Action) => T
  ) {
    this.state = initialState;
  }

  // Add middleware
  use(middleware: Middleware): this {
    this.middleware.push(middleware);
    return this;
  }

  // Get current state
  getState(): T {
    return this.state;
  }

  // Dispatch action
  dispatch(action: Action): void {
    // Apply middleware
    let chain = this.middleware.map(m => m(this));

    const dispatch: (action: Action) => void = (action) => {
      this.state = this.reducer(this.state, action);
      this.notify();
    };

    chain.reduce((next, mw) => mw(next)(next), dispatch)(action);
  }

  // Subscribe to state changes
  subscribe(listener: Listener): () => void {
    this.listeners.add(listener);
    return () => this.listeners.delete(listener);
  }

  private notify(): void {
    this.listeners.forEach(listener => listener());
  }
}

// Logger middleware
export function loggerMiddleware(store: Store<any>) {
  return (next: () => void) => (action: Action) => {
    console.group(`Action: ${action.type}`);
    console.log('Prev state:', store.getState());
    console.log('Payload:', action.payload);
    next();
    console.log('Next state:', store.getState());
    console.groupEnd();
  };
}

// Persistence middleware (localStorage)
export function persistenceMiddleware<T>(key: string) {
  return (store: Store<T>) => {
    // Load initial state
    const saved = localStorage.getItem(key);
    if (saved) {
      try {
        store.state = { ...store.getState(), ...JSON.parse(saved) };
      } catch (e) {
        console.error('Failed to load state:', e);
      }
    }

    return (next: () => void) => (action: Action) => {
      next();
      // Save state after each action
      localStorage.setItem(key, JSON.stringify(store.getState()));
    };
  };
}
```

### React-like State for Components

```typescript
// utils/component-state.ts
/**
 * Simple reactive state for UI components
 */

export class ReactiveState<T> {
  private value: T;
  private listeners: Set<(value: T) => void> = new Set();

  constructor(initialValue: T) {
    this.value = initialValue;
  }

  get(): T {
    return this.value;
  }

  set(newValue: T | ((prev: T) => T)): void {
    this.value = typeof newValue === 'function'
      ? (newValue as (prev: T) => T)(this.value)
      : newValue;

    this.notify();
  }

  subscribe(listener: (value: T) => void): () => void {
    this.listeners.add(listener);
    listener(this.value); // Initial call
    return () => this.listeners.delete(listener);
  }

  private notify(): void {
    this.listeners.forEach(listener => listener(this.value));
  }
}

// Usage example
const counter = new ReactiveState(0);

// Subscribe to changes
const unsubscribe = counter.subscribe(value => {
  document.getElementById('counter')!.textContent = value.toString();
});

// Update value
counter.set(5);
counter.set(prev => prev + 1);
```

### Async State Management

```typescript
// utils/async-state.ts
/**
 * Handle async operations (loading, error, data)
 */

export type AsyncState<T> =
  | { status: 'idle'; data: null; error: null }
  | { status: 'loading'; data: null; error: null }
  | { status: 'success'; data: T; error: null }
  | { status: 'error'; data: null; error: Error };

export class AsyncStore<T> {
  private state: AsyncState<T> = {
    status: 'idle',
    data: null,
    error: null
  };
  private listeners: Set<(state: AsyncState<T>) => void> = new Set();

  constructor(private fetcher: () => Promise<T>) {}

  get(): AsyncState<T> {
    return this.state;
  }

  async execute(): Promise<void> {
    this.setState({ status: 'loading', data: null, error: null });

    try {
      const data = await this.fetcher();
      this.setState({ status: 'success', data, error: null });
    } catch (error) {
      this.setState({
        status: 'error',
        data: null,
        error: error instanceof Error ? error : new Error('Unknown error')
      });
    }
  }

  subscribe(listener: (state: AsyncState<T>) => void): () => void {
    this.listeners.add(listener);
    listener(this.state);
    return () => this.listeners.delete(listener);
  }

  private setState(newState: AsyncState<T>): void {
    this.state = newState;
    this.listeners.forEach(listener => listener(this.state));
  }
}
```

## Security Additional

### CSRF Protection

```typescript
// middleware/csrf.ts
/**
 * CSRF protection for API endpoints
 */

import { createHmac, randomBytes } from 'crypto';

// Generate CSRF token
export function generateCsrfToken(secret: string): string {
  const token = randomBytes(32).toString('base64');
  const hmac = createHmac('sha256', secret);
  hmac.update(token);
  const signature = hmac.digest('base64');

  return `${token}.${signature}`;
}

// Verify CSRF token
export function verifyCsrfToken(token: string, secret: string): boolean {
  const [value, signature] = token.split('.');

  if (!value || !signature) {
    return false;
  }

  const hmac = createHmac('sha256', secret);
  hmac.update(value);
  const expectedSignature = hmac.digest('base64');

  return signature === expectedSignature;
}

// Express middleware
export function csrfProtection(secret: string) {
  return (req: any, res: any, next: any) => {
    // Skip for GET, HEAD, OPTIONS
    if (['GET', 'HEAD', 'OPTIONS'].includes(req.method)) {
      return next();
    }

    const token = req.headers['x-csrf-token'] || req.body?.csrfToken;

    if (!token || !verifyCsrfToken(token, secret)) {
      return res.status(403).json({ error: 'Invalid CSRF token' });
    }

    next();
  };
}
```

### Secure Headers (Helmet.js style)

```typescript
// middleware/secure-headers.ts
/**
 * Set secure HTTP headers
 */

export interface SecureHeadersOptions {
  hsts?: boolean | { maxAge: number; includeSubDomains?: boolean };
  csp?: string | ContentSecurityPolicy;
  noSniff?: boolean;
  frameguard?: boolean | 'deny' | 'sameorigin';
  xssFilter?: boolean;
  referrerPolicy?: string;
  permissionsPolicy?: string;
}

export interface ContentSecurityPolicy {
  'default-src'?: string;
  'script-src'?: string;
  'style-src'?: string;
  'img-src'?: string;
  'connect-src'?: string;
  'font-src'?: string;
  'object-src'?: string;
  'media-src'?: string;
  'frame-src'?: string;
}

export function secureHeaders(options: SecureHeadersOptions = {}): Headers {
  const headers = new Headers();

  // Strict-Transport-Security
  if (options.hsts) {
    const maxAge = typeof options.hsts === 'object' ? options.hsts.maxAge : 31536000;
    const includeSub = typeof options.hsts === 'object' ? options.hsts.includeSubDomains : true;
    headers.set('Strict-Transport-Security', `max-age=${maxAge}${includeSub ? '; includeSubDomains' : ''}`);
  }

  // Content-Security-Policy
  if (options.csp) {
    if (typeof options.csp === 'string') {
      headers.set('Content-Security-Policy', options.csp);
    } else {
      const policies = Object.entries(options.csp)
        .map(([directive, value]) => `${directive} ${value}`)
        .join('; ');
      headers.set('Content-Security-Policy', policies);
    }
  }

  // X-Content-Type-Options
  if (options.noSniff !== false) {
    headers.set('X-Content-Type-Options', 'nosniff');
  }

  // X-Frame-Options
  if (options.frameguard) {
    const mode = options.frameguard === true ? 'deny' : options.frameguard;
    headers.set('X-Frame-Options', mode);
  }

  // X-XSS-Protection
  if (options.xssFilter !== false) {
    headers.set('X-XSS-Protection', '1; mode=block');
  }

  // Referrer-Policy
  if (options.referrerPolicy) {
    headers.set('Referrer-Policy', options.referrerPolicy);
  }

  // Permissions-Policy
  if (options.permissionsPolicy) {
    headers.set('Permissions-Policy', options.permissionsPolicy);
  }

  // Additional security headers
  headers.set('Cross-Origin-Opener-Policy', 'same-origin');
  headers.set('Cross-Origin-Resource-Policy', 'same-origin');

  return headers;
}

// Deno/Edge Function middleware
export function withSecureHeaders(
  handler: (req: Request) => Promise<Response>,
  options?: SecureHeadersOptions
) {
  return async (req: Request): Promise<Response> => {
    const response = await handler(req);

    // Add secure headers
    const headers = secureHeaders(options);
    headers.forEach((value, key) => {
      response.headers.set(key, value);
    });

    return response;
  };
}
```

### Input Sanitization (Deep)

```typescript
// utils/sanitize.ts
/**
 * Comprehensive input sanitization
 */

// HTML entity encoding
export function escapeHtml(unsafe: string): string {
  return unsafe
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#039;');
}

// SQL injection prevention (parameterized queries recommended)
export function escapeSql(unsafe: string): string {
  return unsafe.replace(/[\0\x08\x09\x1a\n\r"'\\\%]/g, char => {
    switch (char) {
      case '\0': return '\\0';
      case '\x08': return '\\b';
      case '\x09': return '\\t';
      case '\x1a': return '\\z';
      case '\n': return '\\n';
      case '\r': return '\\r';
      case '"': case '\'': case '\\': case '%': return '\\' + char;
      default: return char;
    }
  });
}

// Deep sanitize object
export function sanitizeObject<T extends Record<string, any>>(
  obj: T,
  sanitize: (value: string) => string = escapeHtml
): T {
  if (typeof obj !== 'object' || obj === null) {
    return obj;
  }

  const result: Record<string, any> = Array.isArray(obj) ? [] : {};

  for (const [key, value] of Object.entries(obj)) {
    if (typeof value === 'string') {
      result[sanitize(key)] = sanitize(value);
    } else if (Array.isArray(value)) {
      result[sanitize(key)] = value.map(item =>
        typeof item === 'string' ? sanitize(item) : sanitizeObject(item, sanitize)
      );
    } else if (typeof value === 'object' && value !== null) {
      result[sanitize(key)] = sanitizeObject(value, sanitize);
    } else {
      result[sanitize(key)] = value;
    }
  }

  return result as T;
}

// File type validation
export function validateFileType(
  file: File,
  allowedTypes: string[]
): boolean {
  return allowedTypes.includes(file.type);
}

// File size validation
export function validateFileSize(
  file: File,
  maxSizeBytes: number
): boolean {
  return file.size <= maxSizeBytes;
}

// Sanitize filename
export function sanitizeFilename(filename: string): string {
  return filename
    .replace(/[^a-zA-Z0-9.-]/g, '_')
    .replace(/_{2,}/g, '_')
    .slice(0, 255);
}
```

## CI/CD Patterns

### GitHub Actions Workflow

```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [main]
  workflow_dispatch:

env:
  NODE_VERSION: '20'
  SUPABASE_PROJECT_ID: 'myplpshpcordggbbtblg'

jobs:
  # Test job
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Run linter
        run: npm run lint

  # Deploy Supabase Edge Functions
  deploy-functions:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Supabase CLI
        uses: supabase/setup-cli@v1
        with:
          version: latest

      - name: Link to Supabase project
        run: supabase link --project-ref ${{ env.SUPABASE_PROJECT_ID }}
        env:
          SUPABASE_ACCESS_TOKEN: ${{ secrets.SUPABASE_ACCESS_TOKEN }}

      - name: Deploy Edge Functions
        run: supabase functions deploy
        env:
          SUPABASE_ACCESS_TOKEN: ${{ secrets.SUPABASE_ACCESS_TOKEN }}

  # Deploy static site (Vercel/Netlify)
  deploy-site:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-args: '--prod'
```

### Database Migration Workflow

```yaml
# .github/workflows/migrate.yml
name: Database Migration

on:
  workflow_dispatch:
    inputs:
      migration_file:
        description: 'Migration file name'
        required: true

jobs:
  migrate:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Supabase CLI
        uses: supabase/setup-cli@v1

      - name: Link to project
        run: supabase link --project-ref myplpshpcordggbbtblg
        env:
          SUPABASE_ACCESS_TOKEN: ${{ secrets.SUPABASE_ACCESS_TOKEN }}

      - name: Apply migration
        run: |
          # Generate and apply migration
          node supabase/apply-migration.js > migration.sql
          supabase db push --db-url ${{ secrets.DATABASE_URL }}
```

### Production Deployment Checklist

```typescript
// .github/scripts/pre-deploy.ts
/**
 * Pre-deployment checks
 */

interface CheckResult {
  name: string;
  status: 'pass' | 'fail' | 'warn';
  message: string;
}

const checks: CheckResult[] = [];

function check(name: string, condition: boolean, message: string) {
  checks.push({
    name,
    status: condition ? 'pass' : 'fail',
    message: condition ? '✓ ' + message : '✗ ' + message
  });
}

// Run all checks
async function runPreDeployChecks(): Promise<boolean> {
  console.log('Running pre-deployment checks...\n');

  // Environment variables
  check('Environment variables',
    process.env.DATABASE_URL && process.env.SUPABASE_URL,
    'Required environment variables are set'
  );

  // Database connection
  try {
    await testDatabaseConnection();
    check('Database connection', true, 'Can connect to database');
  } catch (error) {
    check('Database connection', false, 'Cannot connect to database');
  }

  // RLS policies
  try {
    const rlsOk = await checkRLSPolicies();
    check('RLS policies', rlsOk, 'RLS policies are enabled');
  } catch (error) {
    check('RLS policies', false, 'Failed to check RLS policies');
  }

  // Print results
  console.log('\nCheck Results:');
  console.log('─'.repeat(50));

  const allPassed = checks.every(c => c.status !== 'fail');

  checks.forEach(c => {
    console.log(`${c.name.padEnd(25)} [${c.status.toUpperCase()}]`);
    console.log(`  ${c.message}`);
  });

  console.log('─'.repeat(50));

  if (allPassed) {
    console.log('\n✓ All checks passed! Ready to deploy.');
  } else {
    console.log('\n✗ Some checks failed. Please fix before deploying.');
  }

  return allPassed;
}

async function testDatabaseConnection(): Promise<void> {
  // Implementation
}

async function checkRLSPolicies(): Promise<boolean> {
  // Implementation
  return true;
}

runPreDeployChecks().then(pass => {
  process.exit(pass ? 0 : 1);
});
```

## Digital Signature

### Canvas Signature Pad

```typescript
// utils/signature-pad.ts
/**
 * Canvas-based signature capture
 */

export class SignaturePad {
  private canvas: HTMLCanvasElement;
  private ctx: CanvasRenderingContext2D;
  private isDrawing = false;
  private hasSignature = false;
  private lastX = 0;
  private lastY = 0;
  private points: Point[] = [];

  constructor(canvas: HTMLCanvasElement, options: SignatureOptions = {}) {
    this.canvas = canvas;
    const context = canvas.getContext('2d');
    if (!context) throw new Error('Cannot get 2D context');

    this.ctx = context;
    this.setupCanvas(options);
    this.attachEvents();
  }

  private setupCanvas(options: SignatureOptions): void {
    const {
      penColor = '#000000',
      penWidth = 2,
      backgroundColor = '#ffffff'
    } = options;

    // Set canvas size
    const rect = this.canvas.getBoundingClientRect();
    this.canvas.width = rect.width * window.devicePixelRatio;
    this.canvas.height = rect.height * window.devicePixelRatio;

    // Scale context
    this.ctx.scale(window.devicePixelRatio, window.devicePixelRatio);

    // Set styles
    this.ctx.strokeStyle = penColor;
    this.ctx.lineWidth = penWidth;
    this.ctx.lineCap = 'round';
    this.ctx.lineJoin = 'round';

    // Fill background
    this.ctx.fillStyle = backgroundColor;
    this.ctx.fillRect(0, 0, rect.width, rect.height);
  }

  private attachEvents(): void {
    // Mouse events
    this.canvas.addEventListener('mousedown', this.onStart);
    this.canvas.addEventListener('mousemove', this.onMove);
    this.canvas.addEventListener('mouseup', this.onEnd);
    this.canvas.addEventListener('mouseleave', this.onEnd);

    // Touch events
    this.canvas.addEventListener('touchstart', this.onTouchStart, { passive: false });
    this.canvas.addEventListener('touchmove', this.onTouchMove, { passive: false });
    this.canvas.addEventListener('touchend', this.onEnd);
  }

  private onStart = (e: MouseEvent): void => {
    this.isDrawing = true;
    const pos = this.getPosition(e);
    this.lastX = pos.x;
    this.lastY = pos.y;
    this.points.push(pos);
  };

  private onMove = (e: MouseEvent): void => {
    if (!this.isDrawing) return;

    const pos = this.getPosition(e);
    this.draw(this.lastX, this.lastY, pos.x, pos.y);
    this.lastX = pos.x;
    this.lastY = pos.y;
    this.points.push(pos);
    this.hasSignature = true;
  };

  private onTouchStart = (e: TouchEvent): void => {
    e.preventDefault();
    const touch = e.touches[0];
    const mouseEvent = new MouseEvent('mousedown', {
      clientX: touch.clientX,
      clientY: touch.clientY
    });
    this.canvas.dispatchEvent(mouseEvent);
  };

  private onTouchMove = (e: TouchEvent): void => {
    e.preventDefault();
    const touch = e.touches[0];
    const mouseEvent = new MouseEvent('mousemove', {
      clientX: touch.clientX,
      clientY: touch.clientY
    });
    this.canvas.dispatchEvent(mouseEvent);
  };

  private onEnd = (): void => {
    this.isDrawing = false;
  };

  private draw(x1: number, y1: number, x2: number, y2: number): void {
    this.ctx.beginPath();
    this.ctx.moveTo(x1, y1);
    this.ctx.lineTo(x2, y2);
    this.ctx.stroke();
  }

  private getPosition(e: MouseEvent): Point {
    const rect = this.canvas.getBoundingClientRect();
    return {
      x: e.clientX - rect.left,
      y: e.clientY - rect.top
    };
  }

  // Get signature as base64
  toDataURL(): string {
    return this.canvas.toDataURL('image/png');
  }

  // Get signature as blob
  async toBlob(): Promise<Blob> {
    return new Promise(resolve => {
      this.canvas.toBlob(blob => {
        resolve(blob!);
      }, 'image/png');
    });
  }

  // Check if signature exists
  isEmpty(): boolean {
    return !this.hasSignature;
  }

  // Clear canvas
  clear(): void {
    const rect = this.canvas.getBoundingClientRect();
    this.ctx.clearRect(0, 0, rect.width, rect.height);
    this.ctx.fillStyle = '#ffffff';
    this.ctx.fillRect(0, 0, rect.width, rect.height);
    this.hasSignature = false;
    this.points = [];
  }

  // Destroy
  destroy(): void {
    this.canvas.removeEventListener('mousedown', this.onStart);
    this.canvas.removeEventListener('mousemove', this.onMove);
    this.canvas.removeEventListener('mouseup', this.onEnd);
    this.canvas.removeEventListener('mouseleave', this.onEnd);
  }
}

interface Point {
  x: number;
  y: number;
}

interface SignatureOptions {
  penColor?: string;
  penWidth?: number;
  backgroundColor?: string;
}

// Usage
// const pad = new SignaturePad(document.getElementById('signature'));
// const dataUrl = pad.toDataURL();
```

### Signature Upload to Supabase

```typescript
// utils/signature-upload.ts
/**
 * Upload signature to Supabase Storage
 */

export async function uploadSignature(
  signatureDataUrl: string,
  jobId: string,
  supabase: any
): Promise<string | null> {
  try {
    // Convert data URL to blob
    const response = await fetch(signatureDataUrl);
    const blob = await response.blob();

    // Generate filename
    const filename = `signatures/${jobId}/${Date.now()}.png`;

    // Upload to Supabase Storage
    const { data, error } = await supabase.storage
      .from('signatures')
      .upload(filename, blob, {
        contentType: 'image/png',
        upsert: true
      });

    if (error) throw error;

    // Get public URL
    const { data: { publicUrl } } = supabase.storage
      .from('signatures')
      .getPublicUrl(filename);

    return publicUrl;
  } catch (error) {
    console.error('Failed to upload signature:', error);
    return null;
  }
}
```

## Barcode & QR Code

### QR Code Generation

```typescript
// utils/qrcode.ts
/**
 * QR Code generation using QRCode.js library
 * Load from: https://cdn.jsdelivr.net/npm/qrcodejs@1.0.0/qrcode.min.js
 */

declare const QRCode: any;

export function generateQRCode(
  elementId: string,
  text: string,
  options: {
    width?: number;
    height?: number;
    colorDark?: string;
    colorLight?: string;
  } = {}
): void {
  const {
    width = 200,
    height = 200,
    colorDark = '#000000',
    colorLight = '#ffffff'
  } = options;

  const element = document.getElementById(elementId);
  if (!element) {
    throw new Error(`Element ${elementId} not found`);
  }

  // Clear existing content
  element.innerHTML = '';

  // Generate QR code
  new QRCode(element, {
    text,
    width,
    height,
    colorDark,
    colorLight,
    correctLevel: QRCode.CorrectLevel.H
  });
}

// Generate QR code as data URL
export function generateQRCodeDataUrl(
  text: string,
  size = 200
): Promise<string> {
  return new Promise((resolve, reject) => {
    const canvas = document.createElement('canvas');
    new QRCode(canvas, {
      text,
      width: size,
      height: size,
      correctLevel: QRCode.CorrectLevel.M
    });

    // Wait for QR code to render
    setTimeout(() => {
      const img = canvas.querySelector('img');
      if (img && img.src) {
        resolve(img.src);
      } else {
        reject(new Error('Failed to generate QR code'));
      }
    }, 100);
  });
}

// Generate job QR code
export function generateJobQRCode(jobRef: string, baseUrl: string): string {
  return `${baseUrl}/job/${jobRef}`;
}
```

### QR Code Scanning

```typescript
// utils/qr-scanner.ts
/**
 * QR Code scanner using html5-qrcode library
 * Load from: https://unpkg.com/html5-qrcode
 */

import { Html5Qrcode, Html5QrcodeScanner } from 'html5-qrcode';

export class QRScanner {
  private scanner: Html5Qrcode;
  private isScanning = false;

  constructor(private cameraId: string) {
    this.scanner = new Html5Qrcode(cameraId);
  }

  async start(
    onSuccess: (decodedText: string) => void,
    onError?: (error: string) => void
  ): Promise<void> {
    if (this.isScanning) return;

    try {
      await this.scanner.start(
        { facingMode: 'environment' }, // Use back camera
        {
          fps: 10,
          qrbox: { width: 250, height: 250 }
        },
        (decodedText) => {
          onSuccess(decodedText);
          // Stop scanning after successful read
          this.stop();
        },
        (errorMessage) => {
          // Ignore errors during scanning (normal)
          if (onError && !errorMessage.includes('No barcode')) {
            onError(errorMessage);
          }
        }
      );
      this.isScanning = true;
    } catch (error) {
      console.error('Failed to start scanner:', error);
      throw error;
    }
  }

  async stop(): Promise<void> {
    if (!this.isScanning) return;

    try {
      await this.scanner.stop();
      this.isScanning = false;
    } catch (error) {
      console.error('Failed to stop scanner:', error);
    }
  }

  isActive(): boolean {
    return this.isScanning;
  }
}

// Usage
// const scanner = new QRScanner('reader');
// await scanner.start((text) => console.log('Scanned:', text));
```

### Barcode Generation (Code128)

```typescript
// utils/barcode.ts
/**
 * Barcode generation (Code128)
 * Using JsBarcode library: https://cdn.jsdelivr.net/npm/jsbarcode@3.11.0/dist/JsBarcode.all.min.js
 */

declare const JsBarcode: any;

export function generateBarcode(
  elementId: string,
  value: string,
  options: {
    format?: string;
    width?: number;
    height?: number;
    displayValue?: boolean;
    fontSize?: number;
  } = {}
): void {
  const {
    format = 'CODE128',
    width = 2,
    height = 100,
    displayValue = true,
    fontSize = 16
  } = options;

  const element = document.getElementById(elementId);
  if (!element) {
    throw new Error(`Element ${elementId} not found`);
  }

  try {
    JsBarcode(element, value, {
      format,
      width,
      height,
      displayValue,
      fontSize,
      margin: 10
    });
  } catch (error) {
    console.error('Failed to generate barcode:', error);
    element.textContent = value; // Fallback
  }
}

// Generate job barcode
export function generateJobBarcode(elementId: string, jobRef: string): void {
  generateBarcode(elementId, jobRef, {
    width: 2,
    height: 80,
    fontSize: 14
  });
}
```

### Barcode/QR Scanner with File Input

```typescript
// utils/file-scanner.ts
/**
 * Scan barcode/QR code from uploaded image
 */

import { Html5Qrcode } from 'html5-qrcode';

export async function scanImageFile(file: File): Promise<string> {
  const html5QrCode = new Html5Qrcode('temp-scanner');

  try {
    const result = await html5QrCode.scanFile(file, true);
    return result;
  } catch (error) {
    throw new Error('No barcode/QR code found in image');
  }
}

// Usage with file input
document.getElementById('file-input')?.addEventListener('change', async (e) => {
  const file = (e.target as HTMLInputElement).files?.[0];
  if (!file) return;

  try {
    const result = await scanImageFile(file);
    console.log('Scanned:', result);
  } catch (error) {
    alert('ไม่พบบาร์โค้ดหรือ QR code ในรูปภาพ');
  }
});
```

---

# PART III: RESOURCES

## Official Documentation

| Category | URL |
|----------|-----|
| **Supabase** | https://supabase.com/docs |
| **PostgreSQL** | https://www.postgresql.org/docs/ |
| **LINE Platform** | https://developers.line.biz/ |
| **React** | https://react.dev |
| **TypeScript** | https://www.typescriptlang.org/docs |
| **Vue** | https://vuejs.org |
| **FastAPI** | https://fastapi.tiangolo.com |
| **Django** | https://docs.djangoproject.com |
| **Playwright** | https://playwright.dev |
| **Google Apps Script** | https://developers.google.com/apps-script |

## Best Practices Reference

| Topic | Key Principles |
|-------|----------------|
| **Database** | Use proper indexes, RLS for security, connection pooling, prepared statements |
| **API Design** | RESTful conventions, proper HTTP status codes, versioning, rate limiting |
| **Frontend** | Mobile-first, accessibility (WCAG), progressive enhancement, error boundaries |
| **Security** | Validate input, sanitize output, use HTTPS, never trust client-side checks |
| **Testing** | Test pyramid: many unit tests, fewer integration, few E2E tests |
| **Performance** | Lazy loading, code splitting, CDN, caching strategies, database query optimization |
| **Error Handling** | Graceful degradation, user-friendly messages, proper logging |
| **Documentation** | README, API docs, code comments for complex logic, changelog |
