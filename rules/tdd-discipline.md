# IRON LAW: TDD Discipline

> **NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST.**

This rule is absolute. Red → Green → Refactor. Always. The test comes first. The test defines the contract. The implementation proves the contract holds.

---

## The Law

**Before writing any production code:**

1. Write a failing test that captures the desired behavior
2. Confirm it fails (RED — the test must actually fail, not error)
3. Write the minimum code to make it pass
4. Confirm it passes (GREEN)
5. Refactor with confidence (tests protect you)
6. Commit: `test: add [behavior] test` → `feat: implement [behavior]`

---

## The RED → GREEN → REFACTOR Cycle

```
RED: Write the test
  → Test must FAIL
  → If it passes immediately, you wrote the wrong test
  → If it errors (not fails), fix the error first

GREEN: Write minimum implementation
  → Make the test pass
  → Minimum means: don't write code not exercised by a test
  → No "while we're here" additions

REFACTOR: Clean up with safety net
  → All tests still pass? Refactor is safe.
  → Extract, rename, simplify — tests protect you
  → Do NOT change behavior during refactor

COMMIT: Two-commit pattern
  → Commit 1: test(scope): add [behavior] test
  → Commit 2: feat(scope): implement [behavior]
```

---

## TDD by Context

### TypeScript/NestJS

```typescript
// RED — Write this FIRST. Run it. Watch it fail.
describe('UserService', () => {
  it('should hash password on create', async () => {
    const user = await userService.create({
      email: 'test@example.com',
      password: 'plaintext'
    });
    expect(user.passwordHash).not.toBe('plaintext');
    expect(user.passwordHash).toMatch(/^\$2[aby]\$/);  // bcrypt pattern
  });
});

// Run: npx vitest run src/users/user.service.spec.ts
// Expected: FAIL — UserService.create is not implemented
// ✅ RED confirmed

// GREEN — Now write the minimum implementation
async create(dto: CreateUserDto): Promise<User> {
  const passwordHash = await bcrypt.hash(dto.password, 12);
  return this.userRepo.create({ ...dto, passwordHash });
}

// Run: npx vitest run src/users/user.service.spec.ts
// Expected: PASS
// ✅ GREEN confirmed
```

### Python/FastAPI

```python
# RED — Write this FIRST. Run it. Watch it fail.
def test_create_user_hashes_password():
    response = client.post("/users", json={
        "email": "test@example.com",
        "password": "plaintext"
    })
    user = response.json()
    assert response.status_code == 201
    assert user["password_hash"] != "plaintext"
    assert user["password_hash"].startswith("$2b$")  # bcrypt

# Run: pytest tests/test_users.py::test_create_user_hashes_password -v
# Expected: FAILED — endpoint not implemented
# ✅ RED confirmed

# GREEN — Now implement
@router.post("/users", status_code=201)
async def create_user(dto: CreateUserRequest, db: Session = Depends(get_db)):
    hashed = bcrypt.hash(dto.password)
    user = User(email=dto.email, password_hash=hashed)
    db.add(user)
    db.commit()
    return user

# Run: pytest tests/test_users.py::test_create_user_hashes_password -v
# Expected: PASSED
# ✅ GREEN confirmed
```

---

## TDD Verification Commands

After every RED step:
```bash
# The test MUST output FAILED or similar — not PASSED, not ERROR
npx vitest run [test-file] --reporter=verbose
# or
pytest [test-file] -v
```

After every GREEN step:
```bash
# The specific test MUST output PASSED
# All other tests MUST still pass (no regressions)
npx vitest run --reporter=verbose
# or
pytest -v
```

---

## What Counts as a Test

**These count:**
- Unit tests with assertions on behavior
- Integration tests with database/service calls
- API tests with HTTP assertions
- Contract tests with schema validation

**These do NOT count:**
- `console.log` and visual inspection
- "I ran it manually and it worked"
- Type annotations only
- Comments describing expected behavior

---

## No-Exceptions List

TDD is required for ALL of these. No exceptions:

```
✅ New functions and methods
✅ New API endpoints (REST, GraphQL, gRPC)
✅ New data models and repositories
✅ Business logic (calculations, transformations, validations)
✅ Auth and authorization logic
✅ Error handling paths
✅ Edge cases discovered during implementation
✅ Bug fixes (write a test that reproduces the bug first)
```

## Legitimate Skip Cases

TDD may be skipped ONLY for:
```
✅ SKIP: Pure configuration files (tsconfig, docker-compose)
✅ SKIP: Pure documentation (README, ADRs)
✅ SKIP: Database migrations (use integration test instead)
✅ SKIP: Boilerplate scaffolding with no logic
```

If unsure: write the test. The overhead is seconds. The benefit is permanent.

---

## The TDD Iron Laws

```
❌ NEVER write production code before a failing test exists
❌ NEVER say "I'll add tests later" — later is never
❌ NEVER delete tests to make CI pass
❌ NEVER skip tests for "simple" functions — simple functions have bugs too
❌ NEVER write tests that always pass (they prove nothing)
❌ NEVER commit without running the full test suite
✅ ALWAYS confirm RED before writing GREEN
✅ ALWAYS run tests after refactor to confirm no regressions
✅ ALWAYS use the two-commit pattern (test commit, then implementation commit)
```
