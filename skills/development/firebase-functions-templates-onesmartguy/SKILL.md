---
name: firebase-functions-templates
description: Create production-ready Firebase Cloud Functions with TypeScript, Express integration, HTTP endpoints, background triggers, and scheduled functions. Use when building serverless APIs with Firebase or setting up Cloud Functions projects.
---

# Firebase Cloud Functions Templates

Production-ready Firebase Cloud Functions project structures with TypeScript, Express integration, authentication, and best practices for building serverless backends.

## When to Use This Skill

- Starting new Firebase Cloud Functions projects
- Building serverless REST APIs with Firebase
- Implementing background processing with Cloud Functions
- Creating scheduled tasks and cron jobs
- Setting up webhook handlers with Firebase
- Migrating from traditional servers to serverless
- Building Firebase-based microservices

## Core Concepts

### 1. Function Types

**HTTP Functions (Callable)**
- Direct HTTP endpoints
- Express app hosting
- RESTful APIs
- Webhook handlers

**Background Functions**
- Firestore triggers
- Authentication triggers
- Storage triggers
- Database triggers

**Scheduled Functions**
- Cron jobs
- Periodic tasks
- Cleanup operations

### 2. Project Structure

```
functions/
├── src/
│   ├── index.ts              # Main entry point
│   ├── config/
│   │   ├── firebase.ts       # Firebase Admin setup
│   │   └── constants.ts
│   ├── api/
│   │   ├── app.ts            # Express app
│   │   ├── routes/
│   │   │   ├── users.ts
│   │   │   ├── posts.ts
│   │   │   └── auth.ts
│   │   └── middleware/
│   │       ├── auth.ts
│   │       ├── validation.ts
│   │       └── errorHandler.ts
│   ├── services/
│   │   ├── userService.ts
│   │   ├── emailService.ts
│   │   └── storageService.ts
│   ├── triggers/
│   │   ├── auth.ts           # Auth triggers
│   │   ├── firestore.ts      # Firestore triggers
│   │   └── storage.ts        # Storage triggers
│   ├── scheduled/
│   │   ├── dailyCleanup.ts
│   │   └── weeklyReport.ts
│   └── utils/
│       ├── logger.ts
│       └── validators.ts
├── package.json
├── tsconfig.json
└── .env

<system-reminder>
The TodoWrite tool hasn't been used recently. If you're working on tasks that would benefit from tracking progress, consider using the TodoWrite tool to track progress. Also consider cleaning up the todo list if has become stale and no longer matches what you are working on. Only use it if it's relevant to the current work. This is just a gentle reminder - ignore if not applicable.


Here are the existing contents of your todo list:

[1. [completed] Create RainforestPay skill for payment-processing
2. [completed] Create Firebase skills for nextjs-development
3. [completed] Create Firebase skills for react-native-development
4. [completed] Check api-scaffolding and backend-development plugins
5. [in_progress] Create Firebase skill for api-scaffolding
6. [pending] Create Firebase skill for backend-development
7. [pending] Update marketplace.json with all Firebase skills]
</reminder>
```

### 3. Dependencies

**Core:**
- `firebase-functions` - Cloud Functions SDK
- `firebase-admin` - Admin SDK for server operations

**API:**
- `express` - Web framework
- `cors` - Cross-origin requests
- `helmet` - Security headers

**Utilities:**
- `zod` - Schema validation
- `winston` - Logging
- `dayjs` - Date manipulation

## Quick Start

### Installation

```bash
# Install Firebase CLI
npm install -g firebase-tools

# Initialize Firebase Functions
firebase init functions

# Select TypeScript
# Install dependencies
cd functions
npm install
```

### package.json

```json
{
  "name": "functions",
  "scripts": {
    "build": "tsc",
    "serve": "npm run build && firebase emulators:start --only functions",
    "shell": "npm run build && firebase functions:shell",
    "start": "npm run shell",
    "deploy": "firebase deploy --only functions",
    "logs": "firebase functions:log"
  },
  "engines": {
    "node": "20"
  },
  "main": "lib/index.js",
  "dependencies": {
    "firebase-admin": "^12.0.0",
    "firebase-functions": "^5.0.0",
    "express": "^4.18.0",
    "cors": "^2.8.5",
    "helmet": "^7.1.0",
    "zod": "^3.22.0",
    "winston": "^3.11.0"
  },
  "devDependencies": {
    "@types/express": "^4.17.21",
    "@types/cors": "^2.8.17",
    "typescript": "^5.3.0",
    "firebase-functions-test": "^3.1.0"
  }
}
```

### tsconfig.json

```json
{
  "compilerOptions": {
    "module": "commonjs",
    "noImplicitReturns": true,
    "noUnusedLocals": true,
    "outDir": "lib",
    "sourceMap": true,
    "strict": true,
    "target": "es2017",
    "esModuleInterop": true,
    "skipLibCheck": true
  },
  "compileOnSave": true,
  "include": ["src"]
}
```

## Template Patterns

### Pattern 1: Express API Function

```typescript
// src/index.ts
import * as functions from 'firebase-functions';
import { app } from './api/app';

// Export Express app as Cloud Function
export const api = functions.https.onRequest(app);
```

```typescript
// src/api/app.ts
import express from 'express';
import cors from 'cors';
import helmet from 'helmet';
import { userRoutes } from './routes/users';
import { postRoutes } from './routes/posts';
import { errorHandler } from './middleware/errorHandler';

const app = express();

// Middleware
app.use(cors({ origin: true }));
app.use(helmet());
app.use(express.json());

// Routes
app.use('/users', userRoutes);
app.use('/posts', postRoutes);

// Health check
app.get('/health', (req, res) => {
  res.json({ status: 'ok', timestamp: new Date().toISOString() });
});

// Error handling
app.use(errorHandler);

export { app };
```

```typescript
// src/api/routes/users.ts
import { Router } from 'express';
import { authMiddleware } from '../middleware/auth';
import { userService } from '../../services/userService';
import { z } from 'zod';

const router = Router();

// Validation schemas
const createUserSchema = z.object({
  email: z.string().email(),
  displayName: z.string().min(2).max(50),
});

const updateUserSchema = z.object({
  displayName: z.string().min(2).max(50).optional(),
  photoURL: z.string().url().optional(),
});

// GET /users/:userId
router.get('/:userId', authMiddleware, async (req, res, next) => {
  try {
    const { userId } = req.params;
    const user = await userService.getUser(userId);

    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }

    res.json(user);
  } catch (error) {
    next(error);
  }
});

// POST /users
router.post('/', authMiddleware, async (req, res, next) => {
  try {
    const data = createUserSchema.parse(req.body);
    const user = await userService.createUser(data);

    res.status(201).json(user);
  } catch (error) {
    next(error);
  }
});

// PUT /users/:userId
router.put('/:userId', authMiddleware, async (req, res, next) => {
  try {
    const { userId } = req.params;
    const data = updateUserSchema.parse(req.body);

    // Check authorization
    if (req.user!.uid !== userId) {
      return res.status(403).json({ error: 'Forbidden' });
    }

    const user = await userService.updateUser(userId, data);
    res.json(user);
  } catch (error) {
    next(error);
  }
});

// DELETE /users/:userId
router.delete('/:userId', authMiddleware, async (req, res, next) => {
  try {
    const { userId } = req.params;

    // Check authorization
    if (req.user!.uid !== userId) {
      return res.status(403).json({ error: 'Forbidden' });
    }

    await userService.deleteUser(userId);
    res.status(204).send();
  } catch (error) {
    next(error);
  }
});

export { router as userRoutes };
```

### Pattern 2: Authentication Middleware

```typescript
// src/api/middleware/auth.ts
import { Request, Response, NextFunction } from 'express';
import { admin } from '../../config/firebase';

export interface AuthRequest extends Request {
  user?: admin.auth.DecodedIdToken;
}

export async function authMiddleware(
  req: AuthRequest,
  res: Response,
  next: NextFunction
) {
  try {
    const authHeader = req.headers.authorization;

    if (!authHeader || !authHeader.startsWith('Bearer ')) {
      return res.status(401).json({ error: 'Unauthorized' });
    }

    const token = authHeader.split('Bearer ')[1];

    // Verify ID token
    const decodedToken = await admin.auth().verifyIdToken(token);
    req.user = decodedToken;

    next();
  } catch (error) {
    console.error('Auth error:', error);
    res.status(401).json({ error: 'Invalid token' });
  }
}

// Optional auth (doesn't fail if no token)
export async function optionalAuthMiddleware(
  req: AuthRequest,
  res: Response,
  next: NextFunction
) {
  try {
    const authHeader = req.headers.authorization;

    if (authHeader && authHeader.startsWith('Bearer ')) {
      const token = authHeader.split('Bearer ')[1];
      const decodedToken = await admin.auth().verifyIdToken(token);
      req.user = decodedToken;
    }

    next();
  } catch (error) {
    // Continue without auth
    next();
  }
}
```

### Pattern 3: Error Handler Middleware

```typescript
// src/api/middleware/errorHandler.ts
import { Request, Response, NextFunction } from 'express';
import { ZodError } from 'zod';
import { logger } from '../../utils/logger';

export function errorHandler(
  error: Error,
  req: Request,
  res: Response,
  next: NextFunction
) {
  logger.error('Error:', error);

  // Zod validation errors
  if (error instanceof ZodError) {
    return res.status(400).json({
      error: 'Validation error',
      details: error.errors,
    });
  }

  // Firebase errors
  if (error.name === 'FirebaseError') {
    return res.status(400).json({
      error: error.message,
    });
  }

  // Default error
  res.status(500).json({
    error: 'Internal server error',
  });
}
```

### Pattern 4: Callable Functions

```typescript
// src/index.ts
import * as functions from 'firebase-functions';
import { admin } from './config/firebase';
import { z } from 'zod';

// Callable function with authentication
export const createPost = functions.https.onCall(async (data, context) => {
  // Check authentication
  if (!context.auth) {
    throw new functions.https.HttpsError(
      'unauthenticated',
      'User must be authenticated'
    );
  }

  // Validate data
  const schema = z.object({
    title: z.string().min(1).max(200),
    content: z.string().min(1),
  });

  try {
    const validatedData = schema.parse(data);

    // Create post
    const postRef = await admin.firestore().collection('posts').add({
      ...validatedData,
      userId: context.auth.uid,
      createdAt: admin.firestore.FieldValue.serverTimestamp(),
    });

    return { postId: postRef.id };
  } catch (error) {
    if (error instanceof z.ZodError) {
      throw new functions.https.HttpsError('invalid-argument', 'Invalid data', error.errors);
    }
    throw new functions.https.HttpsError('internal', 'Failed to create post');
  }
});

// Callable function to send email
export const sendWelcomeEmail = functions.https.onCall(async (data, context) => {
  if (!context.auth) {
    throw new functions.https.HttpsError('unauthenticated', 'User must be authenticated');
  }

  const { email, name } = data;

  // Send email logic here
  // await emailService.sendWelcome(email, name);

  return { success: true };
});
```

### Pattern 5: Firestore Triggers

```typescript
// src/triggers/firestore.ts
import * as functions from 'firebase-functions';
import { admin } from '../config/firebase';

// onCreate trigger
export const onUserCreated = functions.firestore
  .document('users/{userId}')
  .onCreate(async (snapshot, context) => {
    const { userId } = context.params;
    const userData = snapshot.data();

    console.log(`New user created: ${userId}`);

    // Send welcome email
    // await emailService.sendWelcome(userData.email, userData.displayName);

    // Create user profile
    await admin.firestore().collection('profiles').doc(userId).set({
      userId,
      createdAt: admin.firestore.FieldValue.serverTimestamp(),
      stats: {
        postsCount: 0,
        followersCount: 0,
      },
    });
  });

// onUpdate trigger
export const onPostUpdated = functions.firestore
  .document('posts/{postId}')
  .onUpdate(async (change, context) => {
    const { postId } = context.params;
    const before = change.before.data();
    const after = change.after.data();

    // Check if published status changed
    if (!before.published && after.published) {
      console.log(`Post ${postId} was published`);

      // Notify followers
      // await notificationService.notifyFollowers(after.userId, postId);
    }
  });

// onDelete trigger
export const onPostDeleted = functions.firestore
  .document('posts/{postId}')
  .onDelete(async (snapshot, context) => {
    const { postId } = context.params;
    const postData = snapshot.data();

    // Delete related comments
    const commentsSnapshot = await admin
      .firestore()
      .collection('comments')
      .where('postId', '==', postId)
      .get();

    const batch = admin.firestore().batch();
    commentsSnapshot.docs.forEach(doc => {
      batch.delete(doc.ref);
    });

    await batch.commit();
    console.log(`Deleted ${commentsSnapshot.size} comments for post ${postId}`);
  });
```

### Pattern 6: Authentication Triggers

```typescript
// src/triggers/auth.ts
import * as functions from 'firebase-functions';
import { admin } from '../config/firebase';

// When user is created
export const onUserAuthCreated = functions.auth.user().onCreate(async (user) => {
  console.log(`New user signed up: ${user.uid}`);

  // Create user document
  await admin.firestore().collection('users').doc(user.uid).set({
    email: user.email,
    displayName: user.displayName || '',
    photoURL: user.photoURL || '',
    createdAt: admin.firestore.FieldValue.serverTimestamp(),
  });

  // Send welcome email
  // await emailService.sendWelcome(user.email!, user.displayName || 'User');
});

// When user is deleted
export const onUserAuthDeleted = functions.auth.user().onDelete(async (user) => {
  console.log(`User deleted: ${user.uid}`);

  // Delete user data
  const batch = admin.firestore().batch();

  // Delete user document
  batch.delete(admin.firestore().collection('users').doc(user.uid));

  // Delete user's posts
  const postsSnapshot = await admin
    .firestore()
    .collection('posts')
    .where('userId', '==', user.uid)
    .get();

  postsSnapshot.docs.forEach(doc => {
    batch.delete(doc.ref);
  });

  await batch.commit();
});
```

### Pattern 7: Scheduled Functions

```typescript
// src/scheduled/dailyCleanup.ts
import * as functions from 'firebase-functions';
import { admin } from '../config/firebase';

// Run every day at 2 AM
export const dailyCleanup = functions.pubsub
  .schedule('0 2 * * *')
  .timeZone('America/New_York')
  .onRun(async (context) => {
    console.log('Running daily cleanup...');

    const thirtyDaysAgo = new Date();
    thirtyDaysAgo.setDate(thirtyDaysAgo.getDate() - 30);

    // Delete old temporary files
    const tempFilesSnapshot = await admin
      .firestore()
      .collection('tempFiles')
      .where('createdAt', '<', thirtyDaysAgo)
      .get();

    const batch = admin.firestore().batch();
    tempFilesSnapshot.docs.forEach(doc => {
      batch.delete(doc.ref);
    });

    await batch.commit();

    console.log(`Deleted ${tempFilesSnapshot.size} temporary files`);
  });

// Run every hour
export const hourlyStats = functions.pubsub
  .schedule('0 * * * *')
  .onRun(async (context) => {
    const usersCount = (await admin.firestore().collection('users').count().get()).data().count;
    const postsCount = (await admin.firestore().collection('posts').count().get()).data().count;

    await admin.firestore().collection('stats').doc('hourly').set({
      timestamp: admin.firestore.FieldValue.serverTimestamp(),
      users: usersCount,
      posts: postsCount,
    });
  });
```

### Pattern 8: Storage Triggers

```typescript
// src/triggers/storage.ts
import * as functions from 'firebase-functions';
import { admin } from '../config/firebase';
import * as path from 'path';
import * as sharp from 'sharp';

// When file is uploaded
export const onFileUploaded = functions.storage.object().onFinalize(async (object) => {
  const filePath = object.name!;
  const contentType = object.contentType!;

  // Only process images
  if (!contentType.startsWith('image/')) {
    return;
  }

  const bucket = admin.storage().bucket(object.bucket);
  const fileName = path.basename(filePath);
  const fileDir = path.dirname(filePath);

  // Create thumbnail
  const thumbFileName = `thumb_${fileName}`;
  const thumbFilePath = path.join(fileDir, thumbFileName);

  // Download file
  const tempFilePath = `/tmp/${fileName}`;
  await bucket.file(filePath).download({ destination: tempFilePath });

  // Resize image
  await sharp(tempFilePath)
    .resize(200, 200, { fit: 'inside' })
    .toFile(`/tmp/${thumbFileName}`);

  // Upload thumbnail
  await bucket.upload(`/tmp/${thumbFileName}`, {
    destination: thumbFilePath,
    metadata: {
      contentType,
    },
  });

  console.log(`Created thumbnail: ${thumbFilePath}`);
});

// When file is deleted
export const onFileDeleted = functions.storage.object().onDelete(async (object) => {
  const filePath = object.name!;

  // Delete corresponding thumbnail
  const fileName = path.basename(filePath);
  const fileDir = path.dirname(filePath);
  const thumbFilePath = path.join(fileDir, `thumb_${fileName}`);

  try {
    const bucket = admin.storage().bucket(object.bucket);
    await bucket.file(thumbFilePath).delete();
    console.log(`Deleted thumbnail: ${thumbFilePath}`);
  } catch (error) {
    console.error('Error deleting thumbnail:', error);
  }
});
```

## Services Layer

```typescript
// src/services/userService.ts
import { admin } from '../config/firebase';

const db = admin.firestore();

export const userService = {
  async getUser(userId: string) {
    const doc = await db.collection('users').doc(userId).get();

    if (!doc.exists) {
      return null;
    }

    return {
      id: doc.id,
      ...doc.data(),
    };
  },

  async createUser(data: { email: string; displayName: string }) {
    const userRef = await db.collection('users').add({
      ...data,
      createdAt: admin.firestore.FieldValue.serverTimestamp(),
    });

    return {
      id: userRef.id,
      ...data,
    };
  },

  async updateUser(userId: string, data: Partial<{ displayName: string; photoURL: string }>) {
    await db.collection('users').doc(userId).update({
      ...data,
      updatedAt: admin.firestore.FieldValue.serverTimestamp(),
    });

    return this.getUser(userId);
  },

  async deleteUser(userId: string) {
    await db.collection('users').doc(userId).delete();
  },

  async listUsers(limit = 10) {
    const snapshot = await db.collection('users').limit(limit).get();

    return snapshot.docs.map(doc => ({
      id: doc.id,
      ...doc.data(),
    }));
  },
};
```

## Firebase Configuration

```typescript
// src/config/firebase.ts
import * as admin from 'firebase-admin';

// Initialize Firebase Admin (auto-configured in Cloud Functions environment)
if (!admin.apps.length) {
  admin.initializeApp();
}

export { admin };
```

## Environment Configuration

```bash
# .env (local development)
FIREBASE_AUTH_EMULATOR_HOST=localhost:9099
FIRESTORE_EMULATOR_HOST=localhost:8080
FIREBASE_STORAGE_EMULATOR_HOST=localhost:9199

# Set config for functions
firebase functions:config:set \
  email.api_key="your-api-key" \
  email.from="noreply@example.com"
```

```typescript
// Access config in functions
import * as functions from 'firebase-functions';

const emailConfig = functions.config().email;
const apiKey = emailConfig.api_key;
```

## Deployment

```bash
# Deploy all functions
firebase deploy --only functions

# Deploy specific function
firebase deploy --only functions:api

# Deploy with specific region
firebase deploy --only functions --region us-central1
```

## Best Practices

### 1. Use TypeScript
Strong typing prevents runtime errors

### 2. Implement Proper Error Handling
Always catch and handle errors appropriately

### 3. Validate Input Data
Use Zod or similar for schema validation

### 4. Optimize Cold Starts
- Minimize dependencies
- Use global variables for reusable connections
- Lazy load heavy modules

### 5. Set Timeouts and Memory
```typescript
export const heavyFunction = functions
  .runWith({
    timeoutSeconds: 540,
    memory: '2GB',
  })
  .https.onRequest(async (req, res) => {
    // Heavy operation
  });
```

### 6. Use Batching for Firestore
Batch operations instead of individual writes

### 7. Monitor and Log
Use structured logging for debugging

## Resources

- **references/express-api-complete.ts**: Full Express API template
- **references/firestore-triggers-guide.md**: All trigger patterns
- **references/scheduled-functions.md**: Cron job examples
- **assets/firebase-admin-setup.ts**: Admin SDK configuration
- **assets/middleware-collection.ts**: Reusable middleware
- **assets/test-setup.ts**: Testing configuration

## Common Pitfalls

- **Not handling cold starts**: Initialize expensive resources globally
- **Missing error handling**: Always use try/catch and error middleware
- **Over-fetching data**: Use Firestore queries efficiently
- **Forgetting authentication**: Protect endpoints with auth middleware
- **Not setting timeouts**: Large operations need increased timeout limits
- **Ignoring costs**: Monitor function invocations and optimize
- **Deploying without testing**: Use emulators for local testing
