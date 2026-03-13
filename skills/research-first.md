---
name: research-first
description: >
  Always-run research protocol. Maps the codebase before any implementation.
  Reads CLAUDE.md, discovers stack, greps existing patterns, identifies conventions.
  Run this before EVERY implementation task — never guess at existing patterns.
---

# Research First Skill

> **Non-negotiable**: Run this before any implementation task. No exceptions.

## When to Use

- Before implementing any feature
- Before modifying any existing code
- Before writing any tests
- Before making architectural decisions
- When joining a new codebase

## Research Protocol

### Step 1: Master Context

```bash
# Read the master context file
cat CLAUDE.md 2>/dev/null || cat .claude/CLAUDE.md 2>/dev/null || echo "No CLAUDE.md found"
cat README.md | head -100
```

**Capture**:
- Project name and purpose
- Tech stack
- Key architectural decisions
- Existing patterns and conventions
- Test setup
- Deployment targets

### Step 2: Stack Discovery

```bash
# Detect package manager and runtime
ls package.json pyproject.toml go.mod Cargo.toml pom.xml build.gradle 2>/dev/null

# Node.js stack
cat package.json | jq '{dependencies, devDependencies}'

# Python stack
cat pyproject.toml | grep -A 30 '\[project\]'
cat requirements.txt 2>/dev/null | head -30

# Check TypeScript config
cat tsconfig.json 2>/dev/null | head -30
```

### Step 3: Structure Map

```bash
# High-level structure
ls -la src/ 2>/dev/null || ls -la app/ 2>/dev/null

# Key directories
find . -type d -name "controllers" -o -name "services" -o -name "models" \
  -o -name "routes" -o -name "handlers" -o -name "api" 2>/dev/null | grep -v node_modules | head -20
```

### Step 4: Pattern Discovery (Most Important)

Find the existing patterns so you can match them:

```bash
# 1. Authentication pattern — how is auth done?
grep -r "JwtAuthGuard\|UseGuards\|Depends(get_current_user)" src/ --include="*.ts" --include="*.py" -l
grep -r "auth\|jwt\|bearer" src/ --include="*.ts" --include="*.py" -l | head -5

# 2. Error handling pattern
grep -r "throw new\|raise\|HttpException\|HTTPException" src/ --include="*.ts" --include="*.py" -l | head -3
# Read one example:
# grep -r "throw new\|raise" src/ -A 2 --include="*.ts" | head -20

# 3. DTO / Schema validation pattern
grep -r "class.*Dto\|class.*Schema\|BaseModel\|z\.object" src/ --include="*.ts" --include="*.py" -l | head -3

# 4. Test pattern — how are tests structured?
find . -name "*.spec.ts" -o -name "*.test.ts" -o -name "test_*.py" | head -5
# Read one example test file

# 5. Database access pattern
grep -r "repository\|prisma\|typeorm\|sqlalchemy\|drizzle" src/ --include="*.ts" --include="*.py" -l | head -3

# 6. Logging pattern
grep -r "logger\|Logger\|this\.logger\|logging\." src/ --include="*.ts" --include="*.py" | head -5
```

### Step 5: Conventions Check

```bash
# Naming conventions
# Look at existing files in the target directory
ls src/users/ 2>/dev/null  # or equivalent module

# Import conventions
head -20 src/app.module.ts 2>/dev/null
head -20 src/main.py 2>/dev/null

# Code style
cat .eslintrc.js 2>/dev/null || cat eslint.config.js 2>/dev/null | head -30
cat .prettierrc 2>/dev/null
cat pyproject.toml | grep -A 10 '\[tool.ruff\]' 2>/dev/null
```

### Step 6: Test Infrastructure

```bash
# What test runner? What setup?
grep -r "jest\|vitest\|pytest" package.json pyproject.toml 2>/dev/null
cat jest.config.ts 2>/dev/null | head -30
cat vitest.config.ts 2>/dev/null | head -30

# Existing test utilities / factories
ls src/**/__tests__/ tests/ spec/ 2>/dev/null | head -10
grep -r "createUser\|userFactory\|buildUser" src/ --include="*.ts" --include="*.py" -l | head -3
```

## Research Output Format

After completing the research, produce a context brief:

```markdown
## Codebase Context

**Project**: [name] — [purpose]
**Stack**: [runtime] + [framework] + [database] + [key libs]
**Test setup**: [runner] + [coverage tool]

**Patterns found**:
- Auth: [how auth works, e.g., "NestJS JwtAuthGuard + @CurrentUser() decorator"]
- Errors: [e.g., "throw new HttpException or specific typed exceptions"]
- DTOs: [e.g., "class-validator decorators + class-transformer"]
- DB access: [e.g., "Repository pattern via Prisma service injection"]
- Logging: [e.g., "NestJS Logger service via constructor injection"]

**Conventions**:
- File naming: [e.g., "kebab-case.service.ts"]
- Import style: [e.g., "barrel files via index.ts"]
- Test location: [e.g., "co-located .spec.ts files"]

**Key files to read before implementing**:
- [path] — [why relevant]
- [path] — [why relevant]

**Gotchas / Known issues**:
- [anything unusual about this codebase]
```

## Anti-Patterns (Don't Do These)

- Don't start coding before completing this research
- Don't guess at naming conventions — look them up
- Don't create new patterns when existing ones work
- Don't use different error handling than what's already in the codebase
- Don't create a new auth mechanism when one exists

## Fast Path (≥3rd feature in same codebase)

After you know the codebase well:
1. Check CLAUDE.md for updates
2. Run just the pattern-discovery step for the specific domain
3. Read the 1-2 most relevant existing files
4. Proceed

Full research is for first contact and unfamiliar areas.
