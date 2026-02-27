---
name: project-scaffold
description: "Generate project boilerplate from scaffold templates. Use when creating a new project after stack selection. Triggers on: scaffold project, generate boilerplate, create from template."
---

# Project Scaffold Skill

> ⛔ **CRITICAL: Initial scaffold commit respects user preference**
>
> **Trigger:** Before running `git commit` for initial scaffold.
>
> **Check:** Ask user before committing: "Create initial commit? [Y/n]"
> - If user confirms or default: Proceed with `git commit -m "Initial scaffold"`
> - If user declines: **Stop after `git add .`** — do not commit
>
> **Note:** New projects don't have `project.json` yet, so we ask the user directly.
> After scaffolding, if the user sets `git.autoCommit: false` in `project.json`,
> all future agent commits will require manual confirmation.

Generate project boilerplate based on selected stack archetype. This skill is invoked by `project-bootstrap` after stack selection.

---

## Prerequisites

This skill expects:
1. **StackDecision** — The selected stack from stack-advisor (or manual selection)
2. **Project path** — Where to generate files
3. **Project name** — Used for package.json, go.mod, etc.
4. **RequirementsManifest** (optional) — For entity-based schema generation

---

## Scaffold Selection

Match the selected stack to available scaffolds:

| Stack Archetype | Scaffold |
|-----------------|----------|
| `nextjs-supabase` | `~/.config/opencode/scaffolds/nextjs-supabase/` |
| `nextjs-prisma` | `~/.config/opencode/scaffolds/nextjs-prisma/` |
| `go-chi-postgres` | `~/.config/opencode/scaffolds/go-chi-postgres/` |
| `remix-supabase` | `~/.config/opencode/scaffolds/nextjs-supabase/` (adapt) |
| `python-fastapi` | `~/.config/opencode/scaffolds/python-fastapi/` |

If no exact scaffold match exists, use the closest archetype and adapt.

---

## Step 1: Load Scaffold Configuration

Read `scaffold.yaml` from the selected scaffold directory:

```bash
cat ~/.config/opencode/scaffolds/<scaffold-name>/scaffold.yaml
```

The configuration defines:
- `variables` — User prompts for customization
- `dependencies` — npm/go/pip packages to install
- `conditionalDependencies` — Feature-based additions
- `structure` — Directory tree to create
- `files` — Templates to render
- `postScaffold` — Commands to run after generation

---

## Step 2: Collect Variables

For each variable in `scaffold.yaml`, use defaults from context or prompt user:

```yaml
variables:
  - name: projectName
    prompt: "Project name"
    transform: kebab-case
    source: context.projectName  # Auto-fill from bootstrap context
  
  - name: description
    prompt: "Project description"
    source: context.description
  
  - name: supabaseProjectId
    prompt: "Supabase project ID (or 'local' for local dev)"
    default: local
```

**Auto-fill priority:**
1. Context from project-bootstrap (projectName, description, features)
2. Defaults from scaffold.yaml
3. Prompt user if neither available

---

## Step 3: Create Directory Structure

Create all directories defined in `structure`:

```bash
mkdir -p <project-path>/src/app
mkdir -p <project-path>/src/components/ui
mkdir -p <project-path>/src/hooks
mkdir -p <project-path>/src/lib/supabase
mkdir -p <project-path>/supabase/migrations
mkdir -p <project-path>/docs
```

---

## Step 4: Render Template Files

For each file in the scaffold's `files/` directory:

### 4.1 Identify Template Type

| Extension | Processing |
|-----------|------------|
| `.hbs` | Render with Handlebars, remove `.hbs` from output |
| `.template` | Render with Handlebars, remove `.template` from output |
| (no special extension) | Copy as-is |

### 4.2 Build Template Context

```javascript
const context = {
  // From bootstrap/stack-advisor
  projectName: 'my-project',
  projectNamePascal: 'MyProject',
  projectNameCamel: 'myProject',
  description: 'A scheduling app',
  
  // From StackDecision
  stack: {
    frontend: { framework: 'nextjs', version: '15' },
    database: { provider: 'supabase', type: 'postgres' },
    styling: { framework: 'tailwind', version: '4' },
    auth: { provider: 'supabase' }
  },
  
  // From RequirementsManifest (if available)
  features: {
    authentication: true,
    multiTenant: true,
    payments: true,
    email: false,
    ai: false
  },
  
  // Entities for schema generation
  entities: [
    { name: 'Organization', description: 'Tenant/workspace' },
    { name: 'Project', description: 'Work container' },
    { name: 'Task', description: 'Individual work item' }
  ],
  
  // Computed helpers
  hasPayments: capabilities.payments,
  hasEmail: capabilities.email,
  hasAI: capabilities.ai,
  hasMultiTenant: capabilities.multiTenant,
  
  // Date/time
  year: '2026',
  date: '2026-02-19'
};
```

### 4.3 Render Each File

```javascript
for (const file of scaffoldConfig.files) {
  const templatePath = `${scaffoldDir}/files/${file.template}`;
  const outputPath = `${projectPath}/${file.output}`;
  
  if (file.template.endsWith('.hbs')) {
    const template = readFile(templatePath);
    const rendered = handlebars.compile(template)(context);
    writeFile(outputPath, rendered);
  } else {
    copyFile(templatePath, outputPath);
  }
}
```

---

## Step 5: Generate Database Schema (If Entities Provided)

If `RequirementsManifest.entities` exists, generate initial migration:

### 5.1 For Supabase

Generate `supabase/migrations/00001_initial_schema.sql`:

```sql
-- Generated from spec entities
-- {{date}}

-- Organizations (multi-tenant core)
{{#if hasMultiTenant}}
CREATE TABLE organizations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  slug TEXT UNIQUE NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE organization_members (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE,
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
  role TEXT NOT NULL DEFAULT 'member',
  created_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(organization_id, user_id)
);

ALTER TABLE organizations ENABLE ROW LEVEL SECURITY;
ALTER TABLE organization_members ENABLE ROW LEVEL SECURITY;

CREATE POLICY "org_member_select" ON organizations
  FOR SELECT USING (
    id IN (SELECT organization_id FROM organization_members WHERE user_id = auth.uid())
  );

CREATE POLICY "org_member_select" ON organization_members
  FOR SELECT USING (user_id = auth.uid());
{{/if}}

-- Entity tables
{{#each entities}}
{{#unless (isBuiltIn name)}}
CREATE TABLE {{snakeCase name}}s (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  {{#if ../hasMultiTenant}}
  organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE,
  {{/if}}
  name TEXT NOT NULL,
  -- TODO: Add fields for {{description}}
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

ALTER TABLE {{snakeCase name}}s ENABLE ROW LEVEL SECURITY;

{{#if ../hasMultiTenant}}
CREATE POLICY "{{snakeCase name}}_org_access" ON {{snakeCase name}}s
  FOR ALL USING (
    organization_id IN (
      SELECT organization_id FROM organization_members WHERE user_id = auth.uid()
    )
  );
{{/if}}

{{/unless}}
{{/each}}
```

### 5.2 For Prisma

Generate `prisma/schema.prisma`:

```prisma
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
  name      String?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  {{#if hasMultiTenant}}
  memberships OrganizationMember[]
  {{/if}}
}

{{#if hasMultiTenant}}
model Organization {
  id        String   @id @default(cuid())
  name      String
  slug      String   @unique
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  members   OrganizationMember[]
  {{#each entities}}
  {{#unless (isBuiltIn name)}}
  {{camelCase name}}s {{pascalCase name}}[]
  {{/unless}}
  {{/each}}
}

model OrganizationMember {
  id             String       @id @default(cuid())
  organization   Organization @relation(fields: [organizationId], references: [id], onDelete: Cascade)
  organizationId String
  user           User         @relation(fields: [userId], references: [id], onDelete: Cascade)
  userId         String
  role           String       @default("member")
  createdAt      DateTime     @default(now())

  @@unique([organizationId, userId])
}
{{/if}}

{{#each entities}}
{{#unless (isBuiltIn name)}}
model {{pascalCase name}} {
  id        String   @id @default(cuid())
  {{#if ../hasMultiTenant}}
  organization   Organization @relation(fields: [organizationId], references: [id], onDelete: Cascade)
  organizationId String
  {{/if}}
  name      String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
{{/unless}}
{{/each}}
```

---

## Step 6: Resolve Dependencies

### 6.1 Merge Dependencies

Combine base dependencies with conditional dependencies:

```javascript
const allDeps = [...scaffoldConfig.dependencies.production];
const allDevDeps = [...scaffoldConfig.dependencies.development];

for (const cond of scaffoldConfig.conditionalDependencies) {
  if (evaluateCondition(cond.if, context)) {
    allDeps.push(...cond.add);
  }
}
```

### 6.2 For Node.js Projects

Generate or update `package.json`:

```json
{
  "name": "{{projectName}}",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev --turbopack",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "typecheck": "tsc --noEmit",
    "test": "jest",
    "test:e2e": "playwright test"
  },
  "dependencies": {
    {{#each dependencies}}
    "{{name}}": "{{version}}"{{#unless @last}},{{/unless}}
    {{/each}}
  },
  "devDependencies": {
    {{#each devDependencies}}
    "{{name}}": "{{version}}"{{#unless @last}},{{/unless}}
    {{/each}}
  }
}
```

### 6.3 For Go Projects

Generate `go.mod`:

```go
module {{projectName}}

go 1.23

require (
    github.com/go-chi/chi/v5 v5.0.12
    github.com/jackc/pgx/v5 v5.5.5
    {{#if hasAuth}}
    github.com/golang-jwt/jwt/v5 v5.2.1
    {{/if}}
)
```

---

## Step 7: Generate Environment Files

### 7.1 Create `.env.example`

```env
# Database
{{#if (eq stack.database.provider 'supabase')}}
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key
{{else}}
DATABASE_URL=postgresql://user:password@localhost:5432/{{projectName}}
{{/if}}

{{#if hasPayments}}
# Stripe
STRIPE_SECRET_KEY=sk_test_xxx
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_xxx
STRIPE_WEBHOOK_SECRET=whsec_xxx
{{/if}}

{{#if hasEmail}}
# Email
RESEND_API_KEY=re_xxx
{{/if}}

{{#if hasAI}}
# AI
OPENAI_API_KEY=sk-xxx
{{/if}}

# App
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### 7.2 Create `.env.local` (gitignored)

Copy `.env.example` to `.env.local` for immediate local development.

---

## Step 8: Generate Documentation Stubs

### 8.1 Create `docs/project.json`

This is handled by project-bootstrap, but scaffold ensures the structure:

```json
{
  "$schema": "https://opencode.ai/schemas/project.json",
  "name": "{{projectName}}",
  "description": "{{description}}",
  "stack": { /* from StackDecision */ },
  "features": { /* from context */ }
}
```

### 8.2 Create `AGENTS.md`

```markdown
# {{projectNamePascal}}

{{description}}

## Development

\`\`\`bash
npm run dev     # Start dev server (port 3000)
npm run build   # Production build
npm run test    # Run tests
\`\`\`

## Stack

- **Frontend:** Next.js 15 (App Router)
- **Database:** {{stack.database.provider}}
- **Styling:** Tailwind CSS v4
- **Auth:** {{stack.auth.provider}}

## Documentation

- [Architecture](docs/ARCHITECTURE.md)
- [Conventions](docs/CONVENTIONS.md)
```

---

## Step 9: Run Post-Scaffold Commands

Execute commands defined in `postScaffold`:

```yaml
postScaffold:
  - command: npm install
    workdir: .
  - command: npx supabase init
    condition: stack.database.provider == 'supabase'
  - command: git init
  - command: git add .
  - command: git commit -m "Initial scaffold from {{scaffoldName}}"
    requiresConfirmation: true
    confirmationPrompt: "Create initial commit? [Y/n]"
```

**Execution:**

```bash
cd <project-path>

# Install dependencies
npm install

# Initialize Supabase (if applicable)
npx supabase init --workdir .

# Initialize git
git init
git add .

# Ask user before committing (requiresConfirmation: true)
# If user confirms: git commit -m "Initial scaffold from nextjs-supabase"
# If user declines: skip commit, files remain staged
```

**Handling `requiresConfirmation`:**

When a postScaffold command has `requiresConfirmation: true`:
1. Display the `confirmationPrompt` to the user
2. If user confirms (Y/Enter): execute the command
3. If user declines (n): skip the command, continue with next step
4. Report which commands were skipped

---

## Step 10: Summary Output

Report what was generated:

```
═══════════════════════════════════════════════════════════════════════
                      SCAFFOLD COMPLETE
═══════════════════════════════════════════════════════════════════════

✅ Created project: /Users/dev/code/my-project

📁 Directory structure:
   src/
   ├── app/           Next.js App Router
   ├── components/    React components
   ├── hooks/         Custom hooks
   └── lib/           Utilities + Supabase clients
   supabase/
   └── migrations/    Database migrations
   docs/              Documentation

📦 Dependencies installed:
   • next@15, react@19, react-dom@19
   • @supabase/supabase-js, @supabase/ssr
   • tailwindcss@4, clsx, tailwind-merge
   • stripe, @stripe/stripe-js (payments feature)

🗃️ Database schema generated:
   • organizations (multi-tenant)
   • organization_members
   • projects (from spec)
   • tasks (from spec)

📝 Files created:
   • package.json
   • tsconfig.json
   • next.config.ts
   • tailwind.config.ts
   • src/app/layout.tsx
   • src/app/page.tsx
   • src/lib/supabase/client.ts
   • src/lib/supabase/server.ts
   • supabase/migrations/00001_initial_schema.sql
   • .env.example
   • .env.local
   • AGENTS.md

🚀 Next steps:
   1. cd /Users/dev/code/my-project
   2. Update .env.local with your Supabase credentials
   3. Run `npm run dev` to start development
   4. Review docs/drafts/prd-mvp.md for your user stories

═══════════════════════════════════════════════════════════════════════
```

---

## Handlebars Helpers

The scaffold system provides these custom helpers:

| Helper | Description | Example |
|--------|-------------|---------|
| `kebabCase` | Convert to kebab-case | `{{kebabCase projectName}}` → `my-project` |
| `camelCase` | Convert to camelCase | `{{camelCase name}}` → `myProject` |
| `pascalCase` | Convert to PascalCase | `{{pascalCase name}}` → `MyProject` |
| `snakeCase` | Convert to snake_case | `{{snakeCase name}}` → `my_project` |
| `upperCase` | Convert to UPPER_CASE | `{{upperCase name}}` → `MY_PROJECT` |
| `eq` | Equality check | `{{#if (eq type 'supabase')}}` |
| `isBuiltIn` | Check if entity is built-in | `{{#unless (isBuiltIn name)}}` |

**Built-in entities** (skipped in schema generation):
- User
- Organization
- OrganizationMember

---

## Error Handling

### Scaffold Not Found

```
❌ No scaffold found for archetype: django-postgres

Available scaffolds:
  • nextjs-supabase
  • nextjs-prisma
  • go-chi-postgres

Would you like to:
  A. Use closest match (nextjs-prisma) and adapt
  B. Generate minimal structure only
  C. Cancel

> _
```

### Missing Variables

```
⚠️ Missing required variable: supabaseProjectId

Enter Supabase project ID (or 'local' for local dev):
> _
```

### Post-Scaffold Command Failed

```
⚠️ Post-scaffold command failed: npm install

Error: ENOENT: npm not found

The scaffold is complete but dependencies were not installed.
Run manually: cd /path/to/project && npm install
```

---

## Output

Return the scaffold result to the calling agent:

```json
{
  "success": true,
  "projectPath": "/Users/dev/code/my-project",
  "scaffold": "nextjs-supabase",
  "filesCreated": [
    "package.json",
    "tsconfig.json",
    "src/app/layout.tsx",
    "..."
  ],
  "dependenciesInstalled": true,
  "gitInitialized": true,
  "schemaGenerated": true,
  "entities": ["Organization", "Project", "Task"]
}
```
