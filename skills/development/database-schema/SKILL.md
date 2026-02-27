---
name: database-schema
description: Design a database schema
---

# Database Schema Design

Help me design a database schema:

## Requirements

1. **What are we storing?** Describe the data.
2. **Relationships**: How do entities relate?
3. **Access patterns**: How will data be queried?
4. **Scale**: Expected data volume?
5. **Database**: PostgreSQL, MySQL, MongoDB, etc.?

## Entity Design

For each entity, define:

### Table Structure

- Table name (plural, snake_case)
- Columns with types
- Primary key
- Timestamps (created_at, updated_at)
- Soft delete (deleted_at) if needed

### Constraints

- NOT NULL where required
- UNIQUE constraints
- CHECK constraints for validation
- DEFAULT values

### Relationships

- Foreign keys
- ON DELETE behavior
- Junction tables for many-to-many

### Indexes

- Primary key index
- Foreign key indexes
- Query-specific indexes
- Composite indexes where needed

## Schema Generation

Generate:

1. CREATE TABLE statements
2. Index creation
3. RLS policies (if using Supabase)
4. Sample seed data

## Optimization Review

Check for:

- Normalization level appropriate
- Index strategy sound
- Query patterns supported
- Future scaling considerations

Generate the complete schema with comments explaining decisions.
