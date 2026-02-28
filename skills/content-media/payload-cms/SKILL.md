---
name: payload-cms
description: Build content management systems with Payload CMS including collections, globals, fields, hooks, access control, and admin customization. Use when implementing headless CMS functionality, content APIs, or admin dashboards backed by Payload.
version: 1.0.0
---

# Payload CMS

Build headless content management systems with Payload CMS. Payload is a TypeScript-first, code-configured CMS that gives you a full admin panel, REST and GraphQL APIs, authentication, access control, and file uploads out of the box.

## When to Use This Skill

- Building content-managed websites or applications
- Creating admin panels for structured data management
- Implementing complex content models with relationships, blocks, and arrays
- Managing file uploads and media libraries
- Generating type-safe APIs from content schemas
- Adding versioning and draft/publish workflows to content

## Setup

```bash
# Create new Payload project
pnpm create payload-app@latest

# Or add to existing project
pnpm add payload @payloadcms/next @payloadcms/db-mongodb @payloadcms/richtext-lexical
```

### Configuration

```typescript
// payload.config.ts
import { buildConfig } from 'payload';
import { mongooseAdapter } from '@payloadcms/db-mongodb';
import { lexicalEditor } from '@payloadcms/richtext-lexical';
import { uploadthingStorage } from '@payloadcms/storage-uploadthing';
import { Pages } from './collections/Pages';
import { Posts } from './collections/Posts';
import { Media } from './collections/Media';
import { Users } from './collections/Users';
import { SiteSettings } from './globals/SiteSettings';

export default buildConfig({
  collections: [Users, Posts, Pages, Media],
  globals: [SiteSettings],
  editor: lexicalEditor(),
  db: mongooseAdapter({
    url: process.env.DATABASE_URI!,
  }),
  secret: process.env.PAYLOAD_SECRET!,
  typescript: {
    outputFile: 'src/payload-types.ts', // Auto-generated types
  },
  plugins: [
    uploadthingStorage({
      collections: { media: true },
    }),
  ],
});
```

## Collections

Collections are the primary data model. Each collection gets a database table, REST/GraphQL endpoints, and an admin UI.

### Basic Collection

```typescript
// collections/Posts.ts
import type { CollectionConfig } from 'payload';

export const Posts: CollectionConfig = {
  slug: 'posts',
  labels: { singular: 'Post', plural: 'Posts' },
  admin: {
    useAsTitle: 'title',
    defaultColumns: ['title', 'status', 'author', 'updatedAt'],
    listSearchableFields: ['title', 'slug'],
    group: 'Content',
  },
  versions: {
    drafts: {
      autosave: { interval: 1500 },
    },
    maxPerDoc: 25,
  },
  fields: [
    {
      name: 'title',
      type: 'text',
      required: true,
      minLength: 3,
      maxLength: 200,
    },
    {
      name: 'slug',
      type: 'text',
      required: true,
      unique: true,
      admin: { position: 'sidebar' },
    },
    {
      name: 'status',
      type: 'select',
      required: true,
      defaultValue: 'draft',
      options: [
        { label: 'Draft', value: 'draft' },
        { label: 'Published', value: 'published' },
        { label: 'Archived', value: 'archived' },
      ],
      admin: { position: 'sidebar' },
    },
    {
      name: 'author',
      type: 'relationship',
      relationTo: 'users',
      required: true,
      admin: { position: 'sidebar' },
    },
    {
      name: 'publishedAt',
      type: 'date',
      admin: {
        date: { pickerAppearance: 'dayAndTime' },
        position: 'sidebar',
      },
    },
    {
      name: 'featuredImage',
      type: 'upload',
      relationTo: 'media',
    },
    {
      name: 'excerpt',
      type: 'textarea',
      maxLength: 300,
    },
    {
      name: 'content',
      type: 'richText', // Uses configured editor (Lexical)
    },
    {
      name: 'tags',
      type: 'relationship',
      relationTo: 'tags',
      hasMany: true,
    },
    {
      name: 'seo',
      type: 'group',
      fields: [
        { name: 'metaTitle', type: 'text', maxLength: 60 },
        { name: 'metaDescription', type: 'textarea', maxLength: 160 },
        { name: 'ogImage', type: 'upload', relationTo: 'media' },
      ],
    },
  ],
};
```

### Upload Collection (Media)

```typescript
// collections/Media.ts
import type { CollectionConfig } from 'payload';

export const Media: CollectionConfig = {
  slug: 'media',
  upload: {
    mimeTypes: ['image/png', 'image/jpeg', 'image/webp', 'image/svg+xml', 'application/pdf'],
    imageSizes: [
      { name: 'thumbnail', width: 300, height: 300, position: 'centre' },
      { name: 'card', width: 768, height: 432, position: 'centre' },
      { name: 'hero', width: 1920, height: undefined }, // Maintain aspect ratio
    ],
    adminThumbnail: 'thumbnail',
    focalPoint: true,
  },
  fields: [
    { name: 'alt', type: 'text', required: true },
    { name: 'caption', type: 'text' },
  ],
  access: {
    read: () => true, // Public access to media
  },
};
```

### Users Collection with Auth

```typescript
// collections/Users.ts
import type { CollectionConfig } from 'payload';

export const Users: CollectionConfig = {
  slug: 'users',
  auth: {
    tokenExpiration: 7200, // 2 hours
    maxLoginAttempts: 5,
    lockTime: 600000, // 10 minutes
    useAPIKey: true, // Enable API key auth
  },
  admin: {
    useAsTitle: 'email',
    group: 'Admin',
  },
  fields: [
    { name: 'name', type: 'text', required: true },
    {
      name: 'role',
      type: 'select',
      required: true,
      defaultValue: 'editor',
      options: [
        { label: 'Admin', value: 'admin' },
        { label: 'Editor', value: 'editor' },
        { label: 'Viewer', value: 'viewer' },
      ],
    },
    { name: 'avatar', type: 'upload', relationTo: 'media' },
  ],
};
```

## Field Types Reference

| Type | Usage |
|------|-------|
| `text` | Short strings (titles, names) |
| `textarea` | Multi-line text |
| `number` | Numeric values |
| `email` | Email addresses (validated) |
| `select` | Dropdown / enum |
| `radio` | Radio button selection |
| `checkbox` | Boolean toggle |
| `date` | Date/datetime picker |
| `richText` | Rich text editor (Lexical) |
| `upload` | File upload reference |
| `relationship` | Reference to another collection |
| `array` | Repeatable groups of fields |
| `blocks` | Flexible content with named block types |
| `group` | Nested field group (no repetition) |
| `tabs` | Organize fields into tabs |
| `row` | Horizontal field layout |
| `collapsible` | Collapsible field group |
| `json` | Raw JSON data |
| `point` | GeoJSON point coordinates |

### Blocks Field (Flexible Content)

```typescript
{
  name: 'layout',
  type: 'blocks',
  blocks: [
    {
      slug: 'hero',
      fields: [
        { name: 'heading', type: 'text', required: true },
        { name: 'subheading', type: 'text' },
        { name: 'backgroundImage', type: 'upload', relationTo: 'media' },
        { name: 'cta', type: 'group', fields: [
          { name: 'label', type: 'text' },
          { name: 'url', type: 'text' },
        ]},
      ],
    },
    {
      slug: 'contentSection',
      fields: [
        { name: 'content', type: 'richText' },
        { name: 'alignment', type: 'select', options: ['left', 'center', 'right'] },
      ],
    },
    {
      slug: 'gallery',
      fields: [
        { name: 'images', type: 'array', fields: [
          { name: 'image', type: 'upload', relationTo: 'media', required: true },
          { name: 'caption', type: 'text' },
        ]},
        { name: 'columns', type: 'number', min: 1, max: 4, defaultValue: 3 },
      ],
    },
  ],
}
```

## Globals

Globals are single-instance configurations (site settings, navigation, etc.).

```typescript
// globals/SiteSettings.ts
import type { GlobalConfig } from 'payload';

export const SiteSettings: GlobalConfig = {
  slug: 'site-settings',
  access: {
    read: () => true,
    update: ({ req: { user } }) => user?.role === 'admin',
  },
  fields: [
    { name: 'siteName', type: 'text', required: true },
    { name: 'siteDescription', type: 'textarea' },
    { name: 'logo', type: 'upload', relationTo: 'media' },
    {
      name: 'navigation',
      type: 'array',
      fields: [
        { name: 'label', type: 'text', required: true },
        { name: 'url', type: 'text', required: true },
        { name: 'newTab', type: 'checkbox', defaultValue: false },
      ],
    },
    {
      name: 'socialLinks',
      type: 'group',
      fields: [
        { name: 'twitter', type: 'text' },
        { name: 'github', type: 'text' },
        { name: 'linkedin', type: 'text' },
      ],
    },
  ],
};
```

## Access Control

Every collection and field can have fine-grained access control.

```typescript
// collections/Posts.ts (access property)
access: {
  // Who can read posts
  read: ({ req: { user } }) => {
    if (!user) {
      // Unauthenticated users can only see published posts
      return { status: { equals: 'published' } };
    }
    if (user.role === 'admin') return true;
    // Editors can see their own posts regardless of status
    return {
      or: [
        { status: { equals: 'published' } },
        { author: { equals: user.id } },
      ],
    };
  },

  // Who can create posts
  create: ({ req: { user } }) => {
    return user?.role === 'admin' || user?.role === 'editor';
  },

  // Who can update posts
  update: ({ req: { user } }) => {
    if (user?.role === 'admin') return true;
    // Editors can only update their own posts
    return { author: { equals: user?.id } };
  },

  // Who can delete posts
  delete: ({ req: { user } }) => user?.role === 'admin',
},

// Field-level access
{
  name: 'internalNotes',
  type: 'textarea',
  access: {
    read: ({ req: { user } }) => user?.role === 'admin',
    update: ({ req: { user } }) => user?.role === 'admin',
  },
}
```

## Hooks

Hooks execute custom logic at various points in the document lifecycle.

```typescript
// collections/Posts.ts (hooks property)
hooks: {
  beforeChange: [
    // Auto-generate slug from title
    ({ data, operation }) => {
      if (operation === 'create' || data?.title) {
        data.slug = data.title
          .toLowerCase()
          .replace(/[^a-z0-9]+/g, '-')
          .replace(/(^-|-$)/g, '');
      }
      return data;
    },
  ],

  beforeValidate: [
    // Set publishedAt when status changes to published
    ({ data, originalDoc }) => {
      if (data?.status === 'published' && originalDoc?.status !== 'published') {
        data.publishedAt = new Date().toISOString();
      }
      return data;
    },
  ],

  afterChange: [
    // Revalidate cache when post is updated
    async ({ doc, operation }) => {
      if (operation === 'update' && doc.status === 'published') {
        await fetch(`${process.env.APP_URL}/api/revalidate?path=/blog/${doc.slug}`);
      }
    },
    // Send notification on new post
    async ({ doc, operation, req }) => {
      if (operation === 'create' && doc.status === 'published') {
        await notifySubscribers(doc, req);
      }
    },
  ],

  afterDelete: [
    // Clean up related resources
    async ({ doc, req }) => {
      await cleanupOrphanedMedia(doc, req);
    },
  ],
},
```

## Custom Endpoints

```typescript
// collections/Posts.ts (endpoints property)
endpoints: [
  {
    path: '/popular',
    method: 'get',
    handler: async (req) => {
      const posts = await req.payload.find({
        collection: 'posts',
        where: { status: { equals: 'published' } },
        sort: '-viewCount',
        limit: 10,
      });

      return Response.json(posts);
    },
  },
],
```

## Querying Data

### Local API (Server-Side)

```typescript
import { getPayload } from 'payload';
import config from '@payload-config';

const payload = await getPayload({ config });

// Find documents
const posts = await payload.find({
  collection: 'posts',
  where: {
    status: { equals: 'published' },
    tags: { contains: tagId },
  },
  sort: '-publishedAt',
  limit: 10,
  page: 1,
  depth: 2, // Resolve relationships 2 levels deep
});

// Find by ID
const post = await payload.findByID({
  collection: 'posts',
  id: postId,
  depth: 1,
});

// Create
const newPost = await payload.create({
  collection: 'posts',
  data: { title: 'New Post', status: 'draft', author: userId },
});

// Update
const updated = await payload.update({
  collection: 'posts',
  id: postId,
  data: { status: 'published' },
});

// Delete
await payload.delete({ collection: 'posts', id: postId });

// Get global
const settings = await payload.findGlobal({ slug: 'site-settings' });
```

### REST API (Auto-Generated)

```
GET    /api/posts           # List posts
GET    /api/posts/:id       # Get single post
POST   /api/posts           # Create post
PATCH  /api/posts/:id       # Update post
DELETE /api/posts/:id       # Delete post
GET    /api/globals/site-settings  # Get global
```

## TypeScript Type Generation

```bash
# Generate types from your config
pnpm payload generate:types
```

```typescript
// Auto-generated: src/payload-types.ts
import type { Post, Media, User } from '@/payload-types';

// Use in your application code
function renderPost(post: Post) {
  // Full type safety on all fields
  console.log(post.title, post.author, post.tags);
}
```

## Anti-Patterns

| Anti-Pattern | Correct Approach |
|-------------|-----------------|
| Defining collections in a single file | One file per collection in a `collections/` directory |
| Skipping access control | Define access for every collection, even if it returns `true` |
| Using `depth: 0` globally to avoid N+1 | Set `depth` per query based on what you need |
| Putting business logic in afterChange hooks with no error handling | Wrap hook logic in try/catch; a failed hook can block the entire operation |
| Using raw database queries instead of Local API | The Local API respects hooks, access control, and validation |
| Hardcoding admin credentials | Use environment variables and the built-in auth system |
| Not generating TypeScript types after schema changes | Run `pnpm payload generate:types` after every collection/field change |

## Plugins

```typescript
// payload.config.ts
import { seoPlugin } from '@payloadcms/plugin-seo';
import { formBuilderPlugin } from '@payloadcms/plugin-form-builder';
import { searchPlugin } from '@payloadcms/plugin-search';

export default buildConfig({
  plugins: [
    seoPlugin({
      collections: ['posts', 'pages'],
      uploadsCollection: 'media',
      generateTitle: ({ doc }) => `${doc.title} | My Site`,
      generateDescription: ({ doc }) => doc.excerpt ?? '',
    }),
    formBuilderPlugin({
      fields: { text: true, email: true, textarea: true, select: true },
    }),
    searchPlugin({
      collections: ['posts', 'pages'],
      searchOverrides: { fields: [{ name: 'slug', type: 'text' }] },
    }),
  ],
});
```
