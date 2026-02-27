---
name: hive-endpoint
description: How to create API endpoints in Hive framework
globs:
  - src/resources/**/endpoints/*.js
alwaysApply: false
---

# Creating Endpoints

Location: `/src/resources/{name}/endpoints/{action}.js`

## Template (4 Required Exports)

```javascript
import { z } from 'zod';
import db from 'db';

const myService = db.services.myResource;

export const handler = async (ctx) => {
  const { param } = ctx.validatedData;
  // Logic here
  return { result: 'data' };
};

export const middlewares = [];

export const endpoint = {
  url: '/',
  method: 'get',
};

export const requestSchema = z.object({
  param: z.coerce.string().nullable().optional(),
});
```

## Handler Context (ctx)

```javascript
ctx.validatedData       // Validated input (body + query + params)
ctx.state.user          // Authenticated user
ctx.params              // URL params
ctx.throw(404, 'msg')   // Throw error
ctx.assert(cond, 400)   // Assert or throw
```

## Common Patterns

**List:**
```javascript
export const handler = async (ctx) => {
  const { page, perPage } = ctx.validatedData;
  const { results, count } = await service.find(
    { 'user._id': ctx.state.user._id },
    { page, perPage, sort: '-createdOn' }
  );
  return { results, count };
};
export const endpoint = { url: '/', method: 'get' };
```

**Create:**
```javascript
export const handler = async (ctx) => {
  const doc = await service.create({
    ...ctx.validatedData,
    user: ctx.state.user,
  });
  return doc;
};
export const endpoint = { url: '/', method: 'post' };
```

**Update:**
```javascript
export const handler = async (ctx) => {
  const { id, title } = ctx.validatedData;
  return await service.updateOne({ _id: id }, (doc) => ({ ...doc, title }));
};
export const endpoint = { url: '/:id', method: 'put' };
```

## Middlewares

```javascript
// By name
export const middlewares = ['allowNoAuth'];

// With args
export const middlewares = [{ name: 'shouldExist', args: ['projects'] }];

// Direct function
import canEditProject from 'middlewares/canEditProject';
export const middlewares = [canEditProject((ctx) => ctx.params.projectId)];
```

## Public Endpoint (No Auth)

```javascript
export const middlewares = ['allowNoAuth'];
```
