---
name: MongoDB with Mongoose
description: Database patterns for user data and credentials storage
---

# MongoDB with Mongoose Skill

This skill covers MongoDB database operations for the autoclaim bot.

## Connection Setup

```typescript
// src/database.ts
import mongoose from "mongoose";

export async function connectDatabase() {
    const uri = process.env.MONGODB_URI;
    if (!uri) throw new Error("MONGODB_URI not set");

    await mongoose.connect(uri);
    console.log("Connected to MongoDB");
}
```

## User Schema Pattern

```typescript
// src/models/user.ts
import { Schema, model, Document } from "mongoose";

interface IHoyolabCredentials {
    ltoken: string;
    ltuid: string;
    cookieToken?: string;
    games: string[];
}

interface IEndfieldCredentials {
    skOauthCredKey: string;
    gameId: string;
    server: string;
}

interface IUser extends Document {
    discordId: string;
    hoyolab?: IHoyolabCredentials;
    endfield?: IEndfieldCredentials;
    settings: {
        notifications: boolean;
        autoClaimEnabled: boolean;
    };
    lastClaim?: Date;
    createdAt: Date;
    updatedAt: Date;
}

const userSchema = new Schema<IUser>(
    {
        discordId: { type: String, required: true, unique: true, index: true },
        hoyolab: {
            ltoken: { type: String },
            ltuid: { type: String },
            cookieToken: { type: String },
            games: [{ type: String }]
        },
        endfield: {
            skOauthCredKey: { type: String },
            gameId: { type: String },
            server: { type: String }
        },
        settings: {
            notifications: { type: Boolean, default: true },
            autoClaimEnabled: { type: Boolean, default: true }
        },
        lastClaim: { type: Date }
    },
    { timestamps: true }
);

export const User = model<IUser>("User", userSchema);
```

## Common Operations

### Find or Create User

```typescript
async function findOrCreateUser(discordId: string) {
    let user = await User.findOne({ discordId });
    if (!user) {
        user = await User.create({ discordId });
    }
    return user;
}
```

### Update Credentials

```typescript
async function updateHoyolabCredentials(discordId: string, credentials: IHoyolabCredentials) {
    return User.findOneAndUpdate({ discordId }, { $set: { hoyolab: credentials } }, { new: true, upsert: true });
}
```

### Get All Active Users

```typescript
async function getActiveUsers() {
    return User.find({
        "settings.autoClaimEnabled": true,
        $or: [{ "hoyolab.ltoken": { $exists: true } }, { "endfield.skOauthCredKey": { $exists: true } }]
    });
}
```

### Delete User Data

```typescript
async function deleteUserData(discordId: string) {
    return User.deleteOne({ discordId });
}
```

### Update Last Claim Time

```typescript
async function updateLastClaim(discordId: string) {
    return User.findOneAndUpdate({ discordId }, { $set: { lastClaim: new Date() } }, { new: true });
}
```

## Claim History Schema

```typescript
const claimHistorySchema = new Schema({
    discordId: { type: String, required: true, index: true },
    game: { type: String, required: true },
    success: { type: Boolean, required: true },
    message: { type: String },
    rewards: [
        {
            name: String,
            count: Number,
            icon: String
        }
    ],
    claimedAt: { type: Date, default: Date.now }
});

// Index for efficient queries
claimHistorySchema.index({ discordId: 1, claimedAt: -1 });

export const ClaimHistory = model("ClaimHistory", claimHistorySchema);
```

## Best Practices

- Always use indexes for frequently queried fields
- Use `lean()` for read-only queries
- Handle connection errors gracefully
- Use transactions for multi-document updates
- Never log or expose raw credentials

## Security Considerations

```typescript
// Encrypt sensitive data before storing
import crypto from "crypto";

const ENCRYPTION_KEY = process.env.ENCRYPTION_KEY!;

function encrypt(text: string): string {
    const iv = crypto.randomBytes(16);
    const cipher = crypto.createCipheriv("aes-256-cbc", Buffer.from(ENCRYPTION_KEY, "hex"), iv);
    let encrypted = cipher.update(text);
    encrypted = Buffer.concat([encrypted, cipher.final()]);
    return iv.toString("hex") + ":" + encrypted.toString("hex");
}

function decrypt(text: string): string {
    const parts = text.split(":");
    const iv = Buffer.from(parts[0], "hex");
    const encrypted = Buffer.from(parts[1], "hex");
    const decipher = crypto.createDecipheriv("aes-256-cbc", Buffer.from(ENCRYPTION_KEY, "hex"), iv);
    let decrypted = decipher.update(encrypted);
    decrypted = Buffer.concat([decrypted, decipher.final()]);
    return decrypted.toString();
}
```
