---
name: tdd-workflow
description: >
  Test-driven development workflow. Red-green-refactor cycle. Write failing tests first,
  then minimum code to pass, then refactor. Used by qa-champion and builder agents.
---

# TDD Workflow Skill

> Write the test first. Make it pass. Clean it up.

## The Cycle

```
RED   → Write a failing test that describes desired behavior
GREEN → Write minimum code to make the test pass
REFACTOR → Clean up code while keeping tests green
REPEAT
```

## TDD Workflow Steps

### 1. RED — Write a Failing Test

Before writing any implementation code:

```typescript
// Start with the most important behavior
describe('PasswordHasher', () => {
  it('produces a different string than the input', async () => {
    const hasher = new PasswordHasher();
    const hash = await hasher.hash('mypassword');
    expect(hash).not.toBe('mypassword');
  });

  it('produces different hashes for the same input (salted)', async () => {
    const hasher = new PasswordHasher();
    const hash1 = await hasher.hash('mypassword');
    const hash2 = await hasher.hash('mypassword');
    expect(hash1).not.toBe(hash2);
  });

  it('verifies correct password against hash', async () => {
    const hasher = new PasswordHasher();
    const hash = await hasher.hash('mypassword');
    expect(await hasher.verify('mypassword', hash)).toBe(true);
  });

  it('rejects wrong password', async () => {
    const hasher = new PasswordHasher();
    const hash = await hasher.hash('mypassword');
    expect(await hasher.verify('wrongpassword', hash)).toBe(false);
  });
});
```

Run the tests — they should ALL fail (RED):
```bash
npm test -- PasswordHasher  # Expected: all fail with "Cannot find module"
```

### 2. GREEN — Minimum Code to Pass

Write the simplest code that makes the tests pass:

```typescript
// src/auth/password-hasher.ts
import bcrypt from 'bcrypt';

export class PasswordHasher {
  async hash(password: string): Promise<string> {
    return bcrypt.hash(password, 12);
  }

  async verify(password: string, hash: string): Promise<boolean> {
    return bcrypt.compare(password, hash);
  }
}
```

Run tests again — they should ALL pass (GREEN):
```bash
npm test -- PasswordHasher  # Expected: all pass
```

### 3. REFACTOR — Clean It Up

With green tests, clean up the code:
- Extract constants (cost factor)
- Add error handling
- Improve naming
- Add types

```typescript
// src/auth/password-hasher.ts
import bcrypt from 'bcrypt';

const BCRYPT_COST_FACTOR = 12;  // ~300ms — slow enough for brute-force protection

export class PasswordHasher {
  async hash(plaintext: string): Promise<string> {
    return bcrypt.hash(plaintext, BCRYPT_COST_FACTOR);
  }

  async verify(plaintext: string, hash: string): Promise<boolean> {
    return bcrypt.compare(plaintext, hash);
  }
}
```

Run tests again — still GREEN:
```bash
npm test -- PasswordHasher  # Expected: still all pass
```

## Test-First for APIs

For API endpoints, write the integration test first:

```typescript
// Step 1: Write the test
describe('POST /auth/login', () => {
  it('returns access token on valid credentials', async () => {
    const user = await createTestUser({ password: 'correctpassword' });

    const response = await request(app)
      .post('/auth/login')
      .send({ email: user.email, password: 'correctpassword' })
      .expect(200);

    expect(response.body.accessToken).toBeDefined();
    expect(response.body.expiresIn).toBe(900);  // 15 minutes
  });

  it('returns 401 on wrong password', async () => {
    const user = await createTestUser({ password: 'correctpassword' });
    await request(app)
      .post('/auth/login')
      .send({ email: user.email, password: 'wrongpassword' })
      .expect(401);
  });
});

// Step 2: Run — fails (RED)
// Step 3: Implement the route
// Step 4: Run — passes (GREEN)
// Step 5: Refactor
```

## TDD Rules

1. **Never write implementation before a failing test**
2. **Write ONE failing test at a time** — don't write all tests upfront
3. **Minimum code to pass** — don't gold-plate in the GREEN phase
4. **Refactor in small steps** — run tests after every refactor
5. **Keep tests fast** — unit tests < 100ms each
6. **Name tests as specifications** — `it('returns 401 when token expired')`

## When to Skip TDD

TDD slows down initial exploration but pays off for:
- Business logic
- Security code
- Algorithms
- Data transformations

Skip for:
- Glue code (just wiring things together)
- Configuration files
- Simple wrappers around external libraries

For these, write tests after but before the next feature.
