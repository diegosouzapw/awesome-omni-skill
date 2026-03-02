---
name: architecting-database-schema
description: Defines schema, attributes, indexes, and enums for Tourly collections. Use when setting up the database in Appwrite.
---

# Database Schema Architecture

## When to use this skill
- When creating new collections in the Appwrite Console.
- When defining TypeScript interfaces for database documents.

## Schema Definition
### Users (Private Data)
- `userId` (string, required)
- `name` (string)
- `email` (string, required)
- `role` (enum: user, admin)

### Tours
- `title` (string, required)
- `description` (markdown/text)
- `location` (string, required)
- `price` (float, required)
- `images` (string array, storage IDs)
- `rating` (float)
- `availableDates` (datetime array)

### Bookings
- `userId` (string, required)
- `tourId` (string, required)
- `status` (enum: pending, confirmed, cancelled)
- `totalPrice` (float)

## Instructions
- **Indexes**: Add indexes for searchable fields like `location` and `price`.
- **Permissions**: Ensure "Users" can only read their own bookings.
