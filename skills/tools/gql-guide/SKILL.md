---
name: gql:guide
description: Interactive guide to soda-gql features and patterns
user-invocable: true
argument-hint: [topic or question]
allowed-tools: Read, Grep, Glob, AskUserQuestion
---

# GraphQL Guide Skill

This skill provides interactive guidance on soda-gql features, syntax patterns, and best practices. Use `$ARGUMENTS` to route to specific topics, or ask the user to choose a topic.

## Topic Routing

Parse `$ARGUMENTS` to determine the user's intent, or use AskUserQuestion to offer topic selection.

### Available Topics

1. **tagged-template** — Tagged template syntax for fragments and operations
2. **fragment** — Fragment definitions and spreading patterns
3. **operation** — Query, mutation, and subscription operations
4. **union** — Union type handling and member selection
5. **directive** — GraphQL directives (@include, @skip, custom directives)
6. **metadata** — Fragment metadata and field-level callbacks
7. **setup** — Project setup, config, and initial codegen
8. **lsp** — LSP integration, editor setup, and diagnostics
9. **codegen** — Schema codegen, typegen, and build integration
10. **colocation** — Fragment colocation patterns ($colocate directive)

### Topic Selection

If `$ARGUMENTS` is empty or unclear, use AskUserQuestion:

**Question:** "What would you like guidance on?"
**Options:**
- "Tagged template syntax" → tagged-template
- "Fragment patterns" → fragment
- "Operations (queries/mutations)" → operation
- "Union types" → union
- "Directives" → directive
- "Metadata and callbacks" → metadata
- "Project setup" → setup
- "LSP and editor integration" → lsp
- "Codegen and build tools" → codegen
- "Fragment colocation" → colocation

## Topic Content

For each topic, provide:
1. Concept explanation
2. References to documentation files
3. Code examples from playground
4. Common patterns and anti-patterns
5. Related topics

---

## Topic 1: tagged-template

### Concept

soda-gql supports tagged template syntax for writing GraphQL fragments and operations. Only `gql` is exported from the generated runtime:

```typescript
import { gql } from "<outdir-path>"; // e.g. "@/graphql-system"

// Fragment with tagged template
const userFragment = gql.default(({ fragment }) =>
  fragment("UserFields", "User")`{
    id
    name
    email
  }`(),
);

// Operation with tagged template
const getUserQuery = gql.default(({ query }) =>
  query("GetUser")`($id: ID!) {
    user(id: $id) {
      id
      name
    }
  }`(),
);
```

### When to Use Tagged Template vs Callback Builder

**Decision Tree:**

1. **Fragment definition:**
   - ✅ Use tagged template: Simple field selection, no aliases
   - ❌ Use callback builder: Field aliases needed

2. **Fragment spreading (Fragment → Fragment):**
   - ✅ Use tagged template interpolation: `...${otherFragment}`

3. **Operation definition:**
   - ✅ Use tagged template: No fragment spreads via `.spread()`, no aliases, no $colocate
   - ❌ Use callback builder: Has fragment spreads via `.spread()`, aliases, or $colocate

4. **Special features:**
   - Metadata callbacks with variable access → callback builder only
   - Operations with `.spread()` → callback builder only

### Key Constraint

**Both tagged templates reject interpolation with values:**
- `fragment("Name", "Type")\`${value}\`` → ❌ Throws error
- `query("Name")\`{ field(id: ${id}) }\`` → ❌ Throws error

The only valid interpolation is fragment-to-fragment spreading: `fragment("Name", "Type")\`...${otherFragment} ...\``.

Operations with fragment spreads MUST use callback builder syntax with `.spread()` instead of tagged template interpolation.

### Documentation References

- **Callback-only features:** `playgrounds/vite-react/src/graphql/callback-builder-features.ts`
- **Fragment patterns:** `playgrounds/vite-react/src/graphql/fragment-spread-patterns.md`

### Code Examples

**Tagged template fragment:**
```typescript
// playgrounds/vite-react/src/graphql/fragments.ts
const userFields = gql.default(({ fragment }) =>
  fragment("UserFields", "User")`{
    id
    name
    email
    createdAt
  }`(),
);
```

**Tagged template operation (no spreads):**
```typescript
// playgrounds/vite-react/src/graphql/operations.ts
const simpleQuery = gql.default(({ query }) =>
  query("GetUsers")`{
    users {
      id
      name
    }
  }`(),
);
```

**Callback builder operation (with spreads):**
```typescript
// playgrounds/vite-react/src/graphql/callback-builder-features.ts
const userQuery = gql.default(({ query, $var }) =>
  query.operation({
    name: "GetUser",
    variables: {
      ...$var("id").ID("!"),
    },
    fields: ({ f, $ }) => ({
      ...f.user({ id: $.id })(({ f }) => ({
        ...userFields.spread(),
      })),
    }),
  }),
);
```

### Common Patterns

✅ **Simple fragment with tagged template:**
```typescript
const fields = gql.default(({ fragment }) =>
  fragment("UserBasic", "User")`{
    id
    name
    email
  }`(),
);
```

✅ **Fragment spreading another fragment (tagged template):**
```typescript
const extendedFields = gql.default(({ fragment }) =>
  fragment("ExtendedUser", "User")`{
    ...${userFields}
    createdAt
    updatedAt
  }`(),
);
```

❌ **Operation with fragment spread (WRONG - tagged template):**
```typescript
// This will FAIL - cannot use tagged template interpolation for fragment spreads in operations
const badQuery = gql.default(({ query }) =>
  query("BadQuery")`{
    user {
      ${userFields}
    }
  }`(),
);
```

✅ **Operation with fragment spread (CORRECT - callback builder):**
```typescript
const userQuery = gql.default(({ query, $var }) =>
  query.operation({
    name: "GetUser",
    variables: { ...$var("id").ID("!") },
    fields: ({ f, $ }) => ({
      ...f.user({ id: $.id })(({ f }) => ({
        ...userFields.spread(),
      })),
    }),
  }),
);
```

### Related Topics

- **fragment** — Fragment definition and spreading patterns
- **operation** — Operation structure and variable handling
- **metadata** — Callback-builder-only metadata feature

---

## Topic 2: fragment

### Concept

Fragments define reusable field selections with type safety. They can be spread into operations or composed into other fragments.

### Fragment Types

1. **Tagged template fragment** — Simple field selection
2. **Callback builder fragment** — Aliases, nested spreads, metadata
3. **Fragment spreading** — Composing fragments together

### Variable Declaration Pattern

**"Fragments declare requirements; operations declare contract"**

- Fragments can declare variables they need in the GraphQL definition
- Operations must explicitly declare ALL variables, including fragment requirements
- No auto-merge: operation variables are the source of truth

### Documentation References

- **Spread patterns:** `playgrounds/vite-react/src/graphql/fragment-spread-patterns.md`
- **Nested spreads:** `playgrounds/vite-react/src/graphql/nested-fragment-verification.ts`

### Code Examples

**Simple fragment:**
```typescript
const userBasic = gql.default(({ fragment }) =>
  fragment("UserBasic", "User")`{
    id
    name
    email
  }`(),
);
```

**Fragment with variables:**
```typescript
const userConditional = gql.default(({ fragment }) =>
  fragment("ConditionalUser", "User")`($includeEmail: Boolean!) {
    id
    name
    email @include(if: $includeEmail)
  }`(),
);
```

**Fragment spreading (Fragment → Fragment):**
```typescript
const userExtended = gql.default(({ fragment }) =>
  fragment("ExtendedUser", "User")`{
    ...${userBasic}
    createdAt
    updatedAt
  }`(),
);
```

**Operation spreading fragment (callback builder):**
```typescript
const getUserQuery = gql.default(({ query, $var }) =>
  query.operation({
    name: "GetUser",
    variables: {
      ...$var("id").ID("!"),
      ...$var("includeEmail").Boolean("!"),
    },
    fields: ({ f, $ }) => ({
      ...f.user({ id: $.id })(({ f }) => ({
        ...userConditional.spread({ includeEmail: $.includeEmail }),
      })),
    }),
  }),
);
```

### Common Patterns

✅ **Fragment composition via tagged template:**
```typescript
const baseFields = gql.default(({ fragment }) =>
  fragment("UserBase", "User")`{ id name }`(),
);
const extendedFields = gql.default(({ fragment }) =>
  fragment("UserExtended", "User")`{
    ...${baseFields}
    email
  }`(),
);
```

✅ **Operation declares all variables:**
```typescript
const getUserQuery = gql.default(({ query, $var }) =>
  query.operation({
    name: "GetUser",
    variables: {
      ...$var("id").ID("!"),
      ...$var("includeEmail").Boolean("!"), // ALL variables, including fragment requirements
    },
    fields: ({ f, $ }) => ({
      ...f.user({ id: $.id })(({ f }) => ({
        ...userConditional.spread({ includeEmail: $.includeEmail }),
      })),
    }),
  }),
);
```

❌ **Auto-merge expectation (WRONG):**
```typescript
// Fragment declares $includeEmail
const frag = gql.default(({ fragment }) =>
  fragment("F", "User")`($includeEmail: Boolean!) {
    id name email @include(if: $includeEmail)
  }`(),
);

// Operation does NOT auto-inherit variables — must declare includeEmail explicitly
const badQuery = gql.default(({ query, $var }) =>
  query.operation({
    name: "GetUser",
    variables: { ...$var("id").ID("!") }, // Missing includeEmail!
    fields: ({ f, $ }) => ({
      ...f.user({ id: $.id })(({ f }) => ({
        ...frag.spread(), // Will fail — $includeEmail not in scope
      })),
    }),
  }),
);
```

### Related Topics

- **tagged-template** — Syntax for fragment definitions
- **operation** — How operations use fragments
- **metadata** — Fragment-level metadata callbacks

---

## Topic 3: operation

### Concept

Operations define the GraphQL query/mutation/subscription structure with variables, arguments, and field selections.

### Operation Types

1. **Tagged template operation** — Simple operations without fragment spreads
2. **Callback builder operation** — Complex operations with spreads, aliases, $colocate

### Variable Handling

**Operations declare the contract:**
- All variables must be declared at operation level
- Fragment variables are NOT auto-merged
- In callback builder, use `$var("name").Type("!")` to declare variables

### Documentation References

- **Examples:** `playgrounds/vite-react/src/graphql/operations.ts`
- **Callback-only features:** `playgrounds/vite-react/src/graphql/callback-builder-features.ts`

### Code Examples

**Simple query (tagged template):**
```typescript
const getUsers = gql.default(({ query }) =>
  query("GetUsers")`{
    users {
      id
      name
    }
  }`(),
);
```

**Query with variables (tagged template):**
```typescript
const getUser = gql.default(({ query }) =>
  query("GetUser")`($id: ID!) {
    user(id: $id) {
      id
      name
      email
    }
  }`(),
);
```

**Query with fragment spread (callback builder):**
```typescript
const getUserWithFragment = gql.default(({ query, $var }) =>
  query.operation({
    name: "GetUserWithFragment",
    variables: {
      ...$var("id").ID("!"),
    },
    fields: ({ f, $ }) => ({
      ...f.user({ id: $.id })(({ f }) => ({
        ...userFields.spread(),
      })),
    }),
  }),
);
```

**Mutation (tagged template):**
```typescript
const createUser = gql.default(({ mutation }) =>
  mutation("CreateUser")`($input: CreateUserInput!) {
    createUser(input: $input) {
      id
      name
    }
  }`(),
);
```

### Common Patterns

✅ **Simple query (tagged template):**
```typescript
const getUserQuery = gql.default(({ query }) =>
  query("GetUser")`($id: ID!) {
    user(id: $id) {
      id
      name
      posts {
        id
        title
      }
    }
  }`(),
);
```

✅ **Operation with multiple variables (callback builder):**
```typescript
const getUserPosts = gql.default(({ query, $var }) =>
  query.operation({
    name: "GetUserPosts",
    variables: {
      ...$var("id").ID("!"),
      ...$var("limit").Int(),
      ...$var("offset").Int(),
    },
    fields: ({ f, $ }) => ({
      ...f.user({ id: $.id })(({ f }) => ({
        ...f.posts({ limit: $.limit, offset: $.offset })(({ f }) => ({
          id: f.id,
          title: f.title,
        })),
      })),
    }),
  }),
);
```

### Related Topics

- **tagged-template** — Operation syntax options
- **fragment** — Using fragments in operations
- **directive** — Adding directives to operations

---

## Topic 4: union

### Concept

Union types in GraphQL represent a value that could be one of several types. soda-gql handles union types using standard GraphQL inline fragment syntax in tagged templates.

### Union Member Selection

Union handling uses standard GraphQL inline fragment syntax (`... on TypeName { fields }`). Always include `__typename` for type discrimination:

```typescript
const searchQuery = gql.default(({ query }) =>
  query("Search")`($term: String!) {
    search(term: $term) {
      __typename
      ... on User {
        id
        name
      }
      ... on Organization {
        id
        name
        members
      }
    }
  }`(),
);
```

### Documentation References

- **Union verification:** `playgrounds/vite-react/src/graphql/union-type-verification.ts`
- **Callback-only features:** `playgrounds/vite-react/src/graphql/callback-builder-features.ts`

### Code Examples

**Union field selection (tagged template):**
```typescript
const searchQuery = gql.default(({ query }) =>
  query("Search")`($term: String!) {
    search(term: $term) {
      __typename
      ... on User {
        id
        name
        email
      }
      ... on Post {
        id
        title
        content
      }
    }
  }`(),
);
```

**Union with fragment spread (callback builder):**
```typescript
const userFields = gql.default(({ fragment }) =>
  fragment("UserFields", "User")`{ id name email }`(),
);
const postFields = gql.default(({ fragment }) =>
  fragment("PostFields", "Post")`{ id title content }`(),
);

const searchQuery = gql.default(({ query, $var }) =>
  query.operation({
    name: "Search",
    variables: { ...$var("term").String("!") },
    fields: ({ f, $ }) => ({
      ...f.search({ term: $.term })(({ f }) => ({
        __typename: f.__typename,
        ...userFields.spread(),
        ...postFields.spread(),
      })),
    }),
  }),
);
```

### Common Patterns

✅ **Always include __typename:**
```typescript
const q = gql.default(({ query }) =>
  query("Search")`($term: String!) {
    search(term: $term) {
      __typename
      ... on TypeA { id fieldA }
      ... on TypeB { id fieldB }
    }
  }`(),
);
```

✅ **Exhaustive member handling:**
```typescript
const q = gql.default(({ query }) =>
  query("SearchAll")`($term: String!) {
    search(term: $term) {
      __typename
      ... on User { id name }
      ... on Organization { id name }
      ... on Bot { id label }
    }
  }`(),
);
```

### Related Topics

- **fragment** — Using fragments with union members
- **metadata** — Type-specific metadata callbacks

---

## Topic 5: directive

### Concept

GraphQL directives modify field behavior (@include, @skip) or provide metadata for tools. soda-gql supports standard and custom directives.

### Directive Types

1. **Standard directives:** `@include(if: Boolean)`, `@skip(if: Boolean)`
2. **Custom directives:** Defined in schema, used for metadata or tooling

### Directive Syntax

**Tagged template with static values:**
```typescript
gql.default(({ fragment }) =>
  fragment("UserFields", "User")`{
    id
    name
    email @include(if: true)
  }`(),
);
```

**Tagged template with variables:**
```typescript
gql.default(({ fragment }) =>
  fragment("ConditionalUser", "User")`($includeEmail: Boolean!) {
    id
    name
    email @include(if: $includeEmail)
  }`(),
);
```

### Documentation References

- **Directive verification:** `playgrounds/vite-react/src/graphql/directive-verification.ts`

### Code Examples

**@include directive:**
```typescript
const conditionalFields = gql.default(({ fragment }) =>
  fragment("ConditionalUser", "User")`($showEmail: Boolean!) {
    id
    name
    email @include(if: $showEmail)
  }`(),
);
```

**@skip directive:**
```typescript
const fields = gql.default(({ fragment }) =>
  fragment("SkipEmail", "User")`($hideEmail: Boolean!) {
    id
    name
    email @skip(if: $hideEmail)
  }`(),
);
```

**Custom directive:**
```typescript
// Assuming schema has: directive @sensitive on FIELD_DEFINITION
const userFields = gql.default(({ fragment }) =>
  fragment("SensitiveUser", "User")`{
    id
    name
    socialSecurityNumber @sensitive
  }`(),
);
```

### Common Patterns

✅ **Conditional field inclusion (callback builder with fragment spread):**
```typescript
const detailsFragment = gql.default(({ fragment }) =>
  fragment("UserDetails", "User")`($includeDetails: Boolean!) {
    bio @include(if: $includeDetails)
    website @include(if: $includeDetails)
  }`(),
);

const getUserQuery = gql.default(({ query, $var }) =>
  query.operation({
    name: "GetUser",
    variables: {
      ...$var("id").ID("!"),
      ...$var("includeDetails").Boolean("!"),
    },
    fields: ({ f, $ }) => ({
      ...f.user({ id: $.id })(({ f }) => ({
        id: f.id,
        name: f.name,
        ...detailsFragment.spread({ includeDetails: $.includeDetails }),
      })),
    }),
  }),
);
```

### Related Topics

- **fragment** — Directives in fragment definitions
- **metadata** — Directive metadata for tooling

---

## Topic 6: metadata

### Concept

soda-gql allows attaching metadata to fragments and operations for build-time processing (e.g., component mapping, documentation generation). Metadata is passed as an argument to the template call.

### Metadata APIs

**Static metadata** — passed as argument to the template call:

```typescript
const frag = gql.default(({ fragment }) =>
  fragment("UserFields", "User")`{
    id
    name
  }`({
    metadata: { component: "UserCard" },
  }),
);
```

**Callback metadata** — receives variables for dynamic values:

```typescript
const frag = gql.default(({ fragment }) =>
  fragment("UserFields", "User")`($userId: ID!) {
    id
    name
  }`({
    metadata: ({ $ }: { $: { userId: string } }) => ({
      cacheKey: `user:${$.userId}`,
    }),
  }),
);
```

### Documentation References

- **Callback-only features:** `playgrounds/vite-react/src/graphql/callback-builder-features.ts`
- **Metadata verification:** `playgrounds/vite-react/src/graphql/metadata-verification.ts`

### Code Examples

**Static fragment metadata:**
```typescript
const userFragment = gql.default(({ fragment }) =>
  fragment("UserCard", "User")`{
    id
    name
    email
  }`({
    metadata: { component: "UserCard", description: "User profile data" },
  }),
);
```

**Callback metadata with variables:**
```typescript
const userFragment = gql.default(({ fragment }) =>
  fragment("CachedUser", "User")`($userId: ID!) {
    id
    name
    email
  }`({
    metadata: ({ $ }: { $: { userId: string } }) => ({
      cacheKey: `user:${$.userId}`,
    }),
  }),
);
```

**Callback builder operation with metadata:**
```typescript
const q = gql.default(({ query, $var }) =>
  query.operation({
    name: "GetUser",
    variables: { ...$var("id").ID("!") },
    metadata: ({ $, fragmentMetadata }) => ({
      entityId: $.id,
      fragmentCount: fragmentMetadata?.length ?? 0,
    }),
    fields: ({ f, $ }) => ({
      ...f.user({ id: $.id })(({ f }) => ({
        ...userFragment.spread(),
      })),
    }),
  }),
);
```

### Common Patterns

✅ **Component mapping:**
```typescript
gql.default(({ fragment }) =>
  fragment("UserProfile", "User")`{ id name }`({
    metadata: { component: "UserProfile" },
  }),
);
```

✅ **Dynamic cache key:**
```typescript
gql.default(({ fragment }) =>
  fragment("CachedUser", "User")`($id: ID!) { id name }`({
    metadata: ({ $ }: { $: { id: string } }) => ({
      cacheKey: `user:${$.id}`,
    }),
  }),
);
```

### Related Topics

- **fragment** — Metadata on fragments
- **colocation** — $colocate with metadata

---

## Topic 7: setup

### Concept

Setting up a new soda-gql project involves config file creation, schema setup, and initial codegen.

### Setup Steps

1. **Install dependencies:**
   ```bash
   bun add @soda-gql/core @soda-gql/builder
   bun add -d @soda-gql/cli
   ```

2. **Add framework plugin (optional but recommended):**
   ```bash
   # For Vite
   bun add -d @soda-gql/vite-plugin
   # For Next.js
   bun add -d @soda-gql/next-plugin
   ```

3. **Create config file (`soda-gql.config.ts`):**
   ```typescript
   import { defineConfig } from '@soda-gql/config';

   export default defineConfig({
     outdir: './src/graphql/generated',
     schemas: {
       default: {
         schemaFiles: ['./schema.graphql'],
       },
     },
   });
   ```

4. **Run initial codegen:**
   ```bash
   bun run soda-gql codegen schema
   ```

5. **Configure build plugin (Vite example):**
   ```typescript
   // vite.config.ts
   import { sodaGql } from '@soda-gql/vite-plugin';

   export default {
     plugins: [sodaGql()],
   };
   ```

### Documentation References

- **Main README:** `README.md` — Installation and quick start
- **Playground examples:** `playgrounds/vite-react/src/graphql/` — Working examples of all features

### Common Issues

**Issue: codegen fails with "config not found"**
- Check config file is named correctly: `soda-gql.config.{ts,js,mjs}`
- Check config exports with `export default`

**Issue: "Cannot find module '@soda-gql/core'"**
- Run `bun install` to install dependencies
- Check package.json includes @soda-gql packages

**Issue: Types not updating in editor**
- Restart TypeScript server
- Check outdir matches tsconfig include paths

### Related Topics

- **codegen** — Running codegen and typegen
- **lsp** — Editor integration

---

## Topic 8: lsp

### Concept

soda-gql provides LSP (Language Server Protocol) integration for real-time diagnostics, autocomplete, and hover information in editors.

### LSP Features

1. **Real-time diagnostics:** Type errors, invalid fields, syntax errors
2. **Autocomplete:** Field names, types, directives
3. **Hover information:** Type info, documentation
4. **Go to definition:** Jump to schema type definitions

### Editor Setup

**VS Code:**
1. Install LSP extension (if available)
2. Configure workspace settings:
   ```json
   {
     "soda-gql.configPath": "./soda-gql.config.ts"
   }
   ```

**Other editors:**
- Use language server client compatible with your editor
- Point to soda-gql LSP server

### LSP Diagnostics

The LSP validates:
- Field existence on types
- Argument types and required args
- Fragment type compatibility
- Variable types
- Directive usage

### Common Issues

**Issue: LSP not providing diagnostics**
- Check config file is found (`found: true` in project detection)
- Restart editor/LSP server
- Check LSP logs for errors

**Issue: False positive errors**
- Run `bun run soda-gql codegen schema` to sync generated types
- Check schema files are up to date

### Related Topics

- **setup** — Initial LSP configuration
- **codegen** — Keeping LSP in sync with schema

---

## Topic 9: codegen

### Concept

soda-gql codegen generates TypeScript types from GraphQL schemas (schema codegen) and validates/generates types from tagged templates (typegen).

### Codegen Commands

1. **Schema codegen:**
   ```bash
   bun run soda-gql codegen schema
   ```
   - Reads schema files from config
   - Generates runtime type system in outdir
   - Creates fragment and query builder types

2. **Type generation (typegen):**
   ```bash
   bun run soda-gql typegen
   ```
   - Scans codebase for tagged templates
   - Validates field selections against schema
   - Generates TypeScript types for fragments/operations

### Build Integration

**Development workflow:**
1. Edit schema files
2. Run `bun run soda-gql codegen schema`
3. Edit fragments/operations
4. Build plugin auto-runs typegen during build

**Watch mode (if supported):**
```bash
bun run soda-gql codegen schema --watch
```

### Documentation References

- **Monorepo infrastructure:** `docs/guides/monorepo-infrastructure.md` — Build system

### Common Issues

**Issue: "Schema file not found"**
- Check schemaFiles paths in config are relative to config file
- Use Read tool to verify file exists at expected path

**Issue: Typegen shows "unknown field"**
- Run schema codegen first: `bun run soda-gql codegen schema`
- Check field name spelling matches schema

**Issue: Generated types not updating**
- Delete generated directory and re-run codegen
- Check outdir in config matches import paths

### Related Topics

- **setup** — Initial codegen configuration
- **lsp** — Real-time validation vs build-time codegen

---

## Topic 10: colocation

### Concept

Fragment colocation places fragment definitions near the components that use them, improving code organization and enabling build-time optimizations.

### Colocation Patterns

**$colocate in callback builder spread:**
```typescript
const userQuery = gql.default(({ query, $var }) =>
  query.operation({
    name: "GetUser",
    variables: { ...$var("id").ID("!") },
    fields: ({ f, $ }) => ({
      ...f.user({ id: $.id })(({ f }) => ({
        id: f.id,
        ...userFragment.spread({ $colocate: true }),
      })),
    }),
  }),
);
```

**Component colocation:**
```typescript
// UserCard.tsx
export const userCardFragment = gql.default(({ fragment }) =>
  fragment("UserCardFields", "User")`{
    id
    name
    email
    avatarUrl
  }`(),
);

export function UserCard({ user }) {
  // Component uses fragment data
}
```

### Build-Time Processing

Colocation enables:
- Dead code elimination (unused fragments removed)
- Component-fragment association tracking
- Automatic fragment composition

### Documentation References

- **Callback-only features:** `playgrounds/vite-react/src/graphql/callback-builder-features.ts` — $colocate examples

### Common Patterns

✅ **Component-fragment pair:**
```typescript
// UserProfile.tsx
const userProfileFragment = gql.default(({ fragment }) =>
  fragment("UserProfile", "User")`{
    id
    name
    email
    bio
  }`(),
);

function UserProfile({ data }) {
  // Use fragment data
}
```

✅ **Fragment composition with colocation:**
```typescript
const parentQuery = gql.default(({ query, $var }) =>
  query.operation({
    name: "GetUserPage",
    variables: { ...$var("id").ID("!") },
    fields: ({ f, $ }) => ({
      ...f.user({ id: $.id })(({ f }) => ({
        ...profileFragment.spread({ $colocate: true }),
        ...settingsFragment.spread({ $colocate: true }),
      })),
    }),
  }),
);
```

### Related Topics

- **fragment** — Fragment spreading and composition
- **metadata** — Component metadata with colocation

---

## Interactive Mode

If the user's question doesn't match a specific topic, use Grep to search documentation:

1. Extract keywords from user's question
2. Use Grep to search docs/guides/ and playgrounds/
3. Read matching files and synthesize answer
4. Reference specific documentation sections

### Example: User asks "How do I add pagination?"

1. Grep for "pagination" in docs and playground
2. If not found, explain general approach:
   - Add limit/offset arguments to field
   - Use variables in operation
   - Show example with args
3. Suggest related topics: operation, fragment

## Validation Checklist

Before completing this skill, ensure:
- ✅ Topic was identified from $ARGUMENTS or user selection
- ✅ Relevant documentation was referenced
- ✅ Code examples were provided from playground or synthesized
- ✅ Common patterns and anti-patterns were shown
- ✅ Related topics were suggested for further exploration
