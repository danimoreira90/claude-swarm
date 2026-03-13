---
name: db-migrations
description: >
  Safe database migration patterns. Backward-compatible, zero-downtime, reversible.
  Always involve db-architect before applying. Use /migrate command.
---

# Database Migrations Skill

> Schema changes are the most dangerous operations. Get them wrong and you cause downtime.

## Core Principles

1. **Backward-compatible** — old app can run against new schema
2. **Zero-downtime** — no locking operations that block reads/writes
3. **Reversible** — every migration has a rollback
4. **Tested** — run against a staging copy of production first
5. **Small** — one logical change per migration

## The Expand-Contract Pattern

For any breaking change, use 3 phases across 3 deployments:

```
Phase 1: EXPAND — add new column/table (old code ignores it)
Phase 2: MIGRATE — deploy new code that uses new column, backfill data
Phase 3: CONTRACT — remove old column (after confirming all code uses new)
```

## Common Safe Patterns

### Add Column
```sql
-- Phase 1: Add nullable (backward-compatible — old code ignores it)
ALTER TABLE users ADD COLUMN phone VARCHAR(20);

-- Phase 2: App deployed that reads/writes phone
-- Backfill if needed:
UPDATE users SET phone = lookup_phone(id) WHERE phone IS NULL;

-- Phase 3: Make NOT NULL (after all rows have data)
ALTER TABLE users ALTER COLUMN phone SET NOT NULL;
```

### Rename Column
```sql
-- Step 1: Add new column
ALTER TABLE users ADD COLUMN full_name VARCHAR(255);

-- Step 2: Sync both (app writes to both during transition)
-- Backfill:
UPDATE users SET full_name = name WHERE full_name IS NULL;

-- Step 3: App deployed reading only new column
-- Step 4: Drop old column
ALTER TABLE users DROP COLUMN name;
```

### Add Index (Non-Blocking)
```sql
-- Always use CONCURRENTLY to avoid table lock
CREATE INDEX CONCURRENTLY idx_users_email ON users(email);

-- Drop old index
DROP INDEX CONCURRENTLY idx_old_users_email;
```

### Rename Table
```sql
-- Step 1: Create new table with correct name
CREATE TABLE memberships AS SELECT * FROM users;

-- Step 2: Create view with old name for backward compat
CREATE VIEW users AS SELECT * FROM memberships;

-- Step 3: Migrate app to use new name
-- Step 4: Drop view + old table
```

## Prisma Migrations

```bash
# Create a new migration
npx prisma migrate dev --name add_phone_to_users
# This: creates migration file, applies to dev DB, regenerates client

# Preview migration SQL without applying
npx prisma migrate dev --create-only --name add_phone_to_users

# Apply to production (never run dev against production)
npx prisma migrate deploy

# Check migration status
npx prisma migrate status

# Rollback (undo last migration)
# Note: Prisma doesn't have built-in rollback
# Strategy: create a new migration that reverses the change
```

## Raw SQL Migration Template

```sql
-- Migration: 003_add_phone_to_users.sql
-- Description: Add phone number column to users table
-- Rollback: migrations/rollback/003_rollback.sql

-- UP
ALTER TABLE users ADD COLUMN phone_number VARCHAR(20);
COMMENT ON COLUMN users.phone_number IS 'E.164 format: +12125551234';

-- Rollback (in separate file or section)
-- ALTER TABLE users DROP COLUMN phone_number;
```

## Migration CI Checklist

```yaml
# In CI pipeline, run:
- name: Test migrations
  run: |
    # Apply all migrations to test database
    npx prisma migrate deploy
    # Verify schema matches Prisma schema
    npx prisma db pull --print | diff - prisma/schema.prisma || exit 1
```

## Pre-Production Migration Checklist

```
[ ] Migration tested on staging with production-size data
[ ] Estimated lock time: < 100ms (or use CONCURRENT/batching)
[ ] Rollback script prepared and tested
[ ] Fresh backup taken (< 1 hour before)
[ ] Runbook prepared: what to watch, when to rollback
[ ] DB team/DBA notified
[ ] Deploy during low-traffic window (if risky)
[ ] Monitoring dashboard open during migration
```

## Anti-Patterns (Never Do)

```sql
-- NEVER: Lock entire table for long operation
ALTER TABLE large_table ADD COLUMN status VARCHAR NOT NULL DEFAULT 'active';
-- (Adding NOT NULL with DEFAULT rewrites every row on old PostgreSQL)

-- INSTEAD: In phases
ALTER TABLE large_table ADD COLUMN status VARCHAR;  -- Phase 1
UPDATE large_table SET status = 'active' WHERE status IS NULL;  -- Phase 2 (batched)
ALTER TABLE large_table ALTER COLUMN status SET NOT NULL;  -- Phase 3

-- NEVER: Drop column without deprecation period
ALTER TABLE users DROP COLUMN legacy_field;  -- crashes old app instances still running

-- INSTEAD: Expand-contract pattern (deprecate first, then remove)
```
