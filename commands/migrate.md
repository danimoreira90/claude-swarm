---
name: migrate
description: DB migration via db-architect. Safe, backward-compatible, zero-downtime.
allowed-tools: ["Bash", "Read", "Write", "Grep", "Glob"]
argument-hint: "<describe the schema change needed>"
---

<objective>
Create and apply database migration for: $ARGUMENTS

Migration must be: backward-compatible, reversible, non-locking in production.
Always involve db-architect for schema review before applying.
</objective>

<execution_context>
@agents/tier2/db-architect.md
@rules/coding-standards.md
</execution_context>

<context>
**Migration**: $ARGUMENTS

```bash
# Check current schema state
ls -la migrations/ 2>/dev/null || ls -la prisma/migrations/ 2>/dev/null
```
</context>

<process>
1. Analyze what schema change is needed
2. Design the migration using safe patterns (backward-compatible columns first)
3. Create migration file
4. Test migration on dev database
5. Document rollback procedure
6. Apply to target environment
</process>
