---
name: Global Error Handling
description: Your approach to handling global error handling. Use this skill when working on files where global error handling comes into play.
globs:
  - backend/src/routes/**/*.ts
  - backend/src/middleware/**/*.ts
  - backend/src/services/**/*.ts
  - backend/src/controllers/**/*.ts
alwaysApply: true
---

# Global Error Handling

This Skill provides Claude Code with specific guidance on how to handle errors consistently across the MorrisB application.

## CRITICAL: Always Follow This Pattern

When writing or modifying any backend route, service, or middleware, you MUST use the custom error handling system.

## Error Classes Available

Import errors from `../errors` (adjust path as needed):

```typescript
import {
    // Error classes
    AppError,
    ValidationError,
    UnauthorizedError,
    ForbiddenError,
    NotFoundError,
    ConflictError,
    RateLimitError,
    InternalError,
    ExternalServiceError,

    // Async handler wrapper
    asyncHandler,
    asyncAuthHandler,
} from "../errors";
```

## Route Handler Pattern (REQUIRED)

### Option 1: asyncHandler Wrapper (Preferred)

Use `asyncHandler` or `asyncAuthHandler` to automatically catch errors:

```typescript
import { asyncAuthHandler, NotFoundError, ForbiddenError } from "../errors";

router.get("/:workspaceId/contacts/:id", authenticate, asyncAuthHandler(async (req, res) => {
    const { workspaceId, id } = req.params;
    const userId = req.user?._id?.toString();

    // Validate workspace access - throws ForbiddenError
    const workspace = await Project.findById(workspaceId);
    if (!workspace) {
        throw NotFoundError.workspace(workspaceId);
    }
    if (workspace.userId.toString() !== userId) {
        throw ForbiddenError.workspaceAccess();
    }

    // Find contact - throws NotFoundError
    const contact = await Contact.findOne({ _id: id, workspaceId });
    if (!contact) {
        throw NotFoundError.contact(id);
    }

    res.json({ success: true, data: { contact } });
}));
```

### Option 2: Traditional try/catch (Use sparingly)

Only use try/catch when you need specific error handling:

```typescript
router.post("/register", async (req, res, next) => {
    try {
        const validatedData = registerSchema.parse(req.body);
        // ... logic
    } catch (error: any) {
        if (error.name === "ZodError") {
            return next(ValidationError.fromZod(error));
        }
        next(error); // Pass to global error handler
    }
});
```

## Error Factory Methods

Use static factory methods for common error scenarios:

### ValidationError
```typescript
throw ValidationError.missingField("email");
throw ValidationError.invalidFormat("phone", "+1-XXX-XXX-XXXX");
throw ValidationError.invalidId("contact");
throw ValidationError.fromZod(zodError);
```

### UnauthorizedError
```typescript
throw UnauthorizedError.noToken();
throw UnauthorizedError.invalidToken();
throw UnauthorizedError.tokenExpired();
throw UnauthorizedError.invalidCredentials();
```

### ForbiddenError
```typescript
throw ForbiddenError.emailNotVerified();
throw ForbiddenError.workspaceAccess();
throw ForbiddenError.insufficientPermissions("delete contacts");
```

### NotFoundError
```typescript
throw NotFoundError.workspace(workspaceId);
throw NotFoundError.contact(contactId);
throw NotFoundError.user(userId);
throw NotFoundError.campaign(campaignId);
throw NotFoundError.sequence(sequenceId);
throw new NotFoundError("Custom Resource", identifier);
```

### ConflictError
```typescript
throw ConflictError.emailExists();
throw ConflictError.duplicate("Contact", "email", "user@example.com");
```

### ExternalServiceError
```typescript
throw ExternalServiceError.email("Failed to send verification email");
throw ExternalServiceError.ai("OpenAI rate limit exceeded");
```

## DO NOT DO THIS

❌ **Never manually send error responses in routes:**
```typescript
// BAD - Don't do this
res.status(404).json({
    success: false,
    error: "Contact not found",
});
```

❌ **Never catch and swallow errors silently:**
```typescript
// BAD - Don't do this
try {
    await someOperation();
} catch (error) {
    console.error(error); // Lost error!
}
```

❌ **Never use generic error messages:**
```typescript
// BAD - Not helpful
throw new Error("Something went wrong");
```

## DO THIS INSTEAD

✅ **Throw specific error classes:**
```typescript
// GOOD
throw NotFoundError.contact(contactId);
```

✅ **Use asyncHandler to auto-catch:**
```typescript
// GOOD - Errors automatically passed to global handler
router.get("/", asyncHandler(async (req, res) => {
    // If this throws, global handler catches it
    const data = await getData();
    res.json({ success: true, data });
}));
```

✅ **Pass errors to next() when in try/catch:**
```typescript
// GOOD - Passes to global error handler
} catch (error) {
    next(error);
}
```

## Server Setup Required

The global error handler must be registered in `server.ts`:

```typescript
import { globalErrorHandler, notFoundHandler } from "./middleware/errorHandler";

// After all routes
app.use(notFoundHandler);
app.use(globalErrorHandler);
```

## Error Response Format

All errors are returned in this consistent format:

```json
{
    "success": false,
    "error": {
        "message": "Contact not found",
        "code": "RES_001",
        "details": {
            "resource": "Contact",
            "identifier": "123abc"
        }
    }
}
```

## Best Practices Summary

1. **Always use asyncHandler/asyncAuthHandler** for async route handlers
2. **Always throw specific error classes** instead of generic Error
3. **Always use factory methods** when available (e.g., `NotFoundError.contact()`)
4. **Never manually send error responses** - let the global handler do it
5. **Pass errors to next()** if you catch them locally
6. **Use logger for debugging**, not console.log
7. **Include helpful details** in error constructors for debugging

## File References

- Error classes: `backend/src/errors/AppError.ts`
- Error handler middleware: `backend/src/middleware/errorHandler.ts`
- Index exports: `backend/src/errors/index.ts`
