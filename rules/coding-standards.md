# Coding Standards

> These rules are always active. Not suggestions — requirements.

## Universal Rules

1. **Type safety everywhere**
   - TypeScript: `"strict": true` in tsconfig.json. No `any` without explicit `// eslint-disable-next-line @typescript-eslint/no-explicit-any` + justification comment
   - Python: Full type hints on all functions. Mypy strict compliant.

2. **No hardcoded values**
   - No magic numbers — use named constants
   - No hardcoded URLs, IDs, or paths — use config/env vars
   - No inline secrets of any kind (see security.md)

3. **Error handling at boundaries**
   - Validate all external inputs (user input, API responses, file contents)
   - Use typed errors, not generic `Error` or `Exception`
   - Log errors with context (operation, user_id, request_id)
   - Never swallow errors silently

4. **Naming conventions**
   - Functions: `camelCase` (JS/TS), `snake_case` (Python) — verbs: `createUser`, `findById`
   - Classes: `PascalCase`
   - Constants: `SCREAMING_SNAKE_CASE`
   - Files: `kebab-case.service.ts`, `kebab_case_module.py`
   - Booleans: prefix with `is`, `has`, `can`, `should`: `isActive`, `hasPermission`

5. **Function size**
   - Max 50 lines per function. Extract helpers if longer.
   - Max 4 levels of nesting. Extract functions or early-return to flatten.
   - Single responsibility: one function does one thing.

## TypeScript

```typescript
// ✓ Good: typed, explicit, readable
async function createUser(dto: CreateUserDto): Promise<UserResponseDto> {
  const validated = CreateUserSchema.parse(dto);  // Zod validation
  const hashed = await bcrypt.hash(validated.password, 12);
  const user = await db.users.create({ data: { ...validated, password: hashed } });
  return UserResponseDto.fromEntity(user);
}

// ✗ Bad: untyped, implicit, dangerous
async function createUser(dto: any) {
  return db.users.create({ data: dto });  // no validation, no hashing
}
```

### Imports
- Use absolute imports (configured in tsconfig paths)
- Barrel files (`index.ts`) for public module APIs
- No circular imports — use dependency injection

### Async/Await
- Always `await` Promises — no floating promises
- Use `Promise.all()` for independent async operations (not sequential awaits)
- Handle rejection: `try/catch` or `.catch()` on every async call

## Python

```python
# ✓ Good: typed, validated, readable
async def create_user(dto: CreateUserDTO) -> UserResponse:
    validated = CreateUserDTO.model_validate(dto)
    hashed = get_password_hash(validated.password)
    user = await user_repository.create(
        CreateUserEntity(email=validated.email, password_hash=hashed)
    )
    return UserResponse.model_validate(user)

# ✗ Bad: untyped, unvalidated
async def create_user(dto):
    return await user_repository.create(dto)
```

### Style
- Follow [PEP 8](https://peps.python.org/pep-0008/)
- Max line length: 88 (Black/Ruff default)
- f-strings for string formatting (not `.format()` or `%`)
- Pathlib for file paths (not `os.path`)
- Dataclasses or Pydantic for data structures (not raw dicts)

## Code Comments

Comment **why**, not **what**. The code shows what; the comment explains why.

```typescript
// ✓ Good: explains non-obvious decision
// Use bcrypt cost factor 12 for ~300ms hash time — slow enough to prevent brute force,
// fast enough for login UX
const hashed = await bcrypt.hash(password, 12);

// ✗ Bad: restates the code
// Hash the password with bcrypt using cost factor 12
const hashed = await bcrypt.hash(password, 12);
```

Add comments for:
- Non-obvious algorithms
- Business logic rationale ("per the requirement in SEC-234")
- Security decisions ("timing-safe comparison to prevent oracle attacks")
- Temporary workarounds ("TODO: remove after vendor fixes their API — JIRA-456")

## Dependencies

- Prefer well-maintained libraries with >1M weekly downloads (unless specialized)
- Check for CVEs before adding a new dependency: `npm audit` / `pip-audit`
- Don't add a library for a task that's 5 lines of code
- Pin exact versions in production (`^` is OK for development tools)
