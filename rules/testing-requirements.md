# Testing Requirements

> Tests are not optional. Untested code is broken code waiting to be discovered.

## Coverage Minimums

| Layer | Minimum | Target |
|-------|---------|--------|
| Overall | 80% | 85% |
| Business logic (services) | 90% | 95% |
| Auth / Security code | 95% | 100% |
| Payment / Financial code | 95% | 100% |
| Utility functions | 85% | 95% |
| API routes | 80% | 90% |

Coverage is a floor, not a goal. High coverage with weak assertions is useless. Test behavior, not implementation.

## What Must Be Tested

### Always Test
1. Happy path (expected inputs → expected output)
2. Validation errors (invalid inputs → correct error responses)
3. Authorization (wrong user/role → 403/401)
4. Not-found cases (missing resource → 404)
5. Duplicate/conflict cases (creating duplicate → 409)
6. Business rule violations (e.g., insufficient balance → 422)

### Test These Edge Cases
- Empty arrays/strings/null/undefined inputs
- Maximum length inputs (boundary values)
- Concurrent operations (race conditions)
- Database errors (connection failure, constraint violation)
- External service failures (timeouts, 5xx responses)

### For Auth Code: Also Test
- Expired tokens
- Malformed tokens
- Wrong algorithm (alg:none attack attempt)
- Missing required claims (exp, iss, aud)
- Revoked tokens
- Insufficient permissions for the action

## TDD Workflow

```
1. Write the test (it fails — RED)
2. Write minimum code to pass (GREEN)
3. Refactor while keeping tests green (REFACTOR)
4. Repeat
```

For features: write tests from the API spec before implementation.
For bugs: write a failing test that reproduces the bug, then fix it.

## Test Structure

```typescript
// File: users.service.spec.ts
describe('UsersService', () => {
  describe('createUser', () => {
    it('creates user with hashed password on valid input', async () => { ... });
    it('throws ConflictException when email exists', async () => { ... });
    it('throws ValidationError when email is invalid', async () => { ... });
  });

  describe('findOne', () => {
    it('returns user DTO when found and authorized', async () => { ... });
    it('throws NotFoundException when user does not exist', async () => { ... });
    it('throws ForbiddenException when user not authorized', async () => { ... });
  });
});
```

## Test Isolation

- **No shared state between tests** — each test starts from clean state
- **No real external calls** — mock all: databases, APIs, queues, file system
- **No time dependencies** — mock `Date.now()`, `new Date()`
- **No random without seeds** — deterministic tests only
- **No network calls** — use interceptors or mock clients

```typescript
// Good: clean mock per test
beforeEach(() => {
  jest.clearAllMocks();
  mockRepo = {
    findById: jest.fn(),
    create: jest.fn(),
    update: jest.fn(),
  };
});
```

## Integration Tests

Integration tests use real databases (via TestContainers or Docker):

```typescript
// Setup real PostgreSQL for integration tests
beforeAll(async () => {
  container = await new PostgreSqlContainer('postgres:16-alpine').start();
  app = await createTestApp(container.getConnectionUri());
});

afterAll(async () => {
  await app.close();
  await container.stop();
});

beforeEach(async () => {
  await truncateAllTables();  // clean state per test
});
```

## E2E Tests (Playwright)

```typescript
test('user can register, login, and access profile', async ({ page }) => {
  await page.goto('/register');
  await page.fill('[name=email]', 'test@example.com');
  await page.fill('[name=password]', 'SecurePass123!');
  await page.click('[type=submit]');

  await expect(page).toHaveURL('/dashboard');
  await page.goto('/profile');
  await expect(page.locator('h1')).toContainText('test@example.com');
});
```

## CI Requirements

Tests must run in CI on every PR:

```yaml
- run: npm run test:unit -- --coverage --ci
- run: npm run test:integration
- run: npm run test:e2e  # on staging deploy preview
```

**CI fails if**:
- Any test fails
- Coverage drops below threshold
- TypeScript errors exist
- Lint errors exist

## Testing Libraries

| Stack | Unit | Integration | E2E |
|-------|------|-------------|-----|
| NestJS | Jest + @nestjs/testing | Jest + Supertest + TestContainers | Playwright |
| FastAPI | pytest + pytest-asyncio | httpx + TestContainers | Playwright |
| Next.js | Vitest + @testing-library/react | — | Playwright |
| React Native | Jest + @testing-library/react-native | — | Detox |
| AI/LLM | pytest + DeepEval | RAGAS | — |
