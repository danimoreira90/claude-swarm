---
name: code-review
description: >
  Multi-pass code review. Covers logic correctness, security, performance, style, tests.
  Produces structured findings with severity levels and actionable fixes.
  Used by review command and as pre-merge gate.
---

# Code Review Skill

> Code that isn't reviewed ships bugs to production. Review everything.

## Review Protocol — 5 Passes

Run each pass separately to maintain focus:

### Pass 1: Logic Correctness

Questions to answer:
- Does this code do what it claims to do?
- Are there off-by-one errors?
- Is error handling comprehensive?
- Are concurrent access patterns safe?
- Are null/undefined cases handled?
- Are all code paths reachable and correct?

```
Checklist:
[ ] Happy path logic is correct
[ ] Error cases return appropriate responses
[ ] No uncaught exceptions can propagate to the user
[ ] Concurrency: no race conditions on shared state
[ ] Null safety: all nullable values are checked
[ ] Type assertions are safe (no unsafe casts)
```

### Pass 2: Security

Run full security-guardian check (see security-review skill):
- OWASP Top 10
- Secrets scanning
- Auth enforcement
- Input validation
- Rate limiting

### Pass 3: Performance

```
[ ] No N+1 queries (SELECT in a loop)
[ ] Database queries use appropriate indexes
[ ] No unbounded queries (always use LIMIT or pagination)
[ ] No blocking I/O in hot paths
[ ] No memory leaks (event listeners, closures, unclosed connections)
[ ] No unnecessary re-renders (React: check dependency arrays)
[ ] Bundle size impact (new dependencies?)
[ ] Caching used where appropriate
```

N+1 detection:
```typescript
// BAD — N+1: 1 query + N queries
const orders = await orderService.findAll();
for (const order of orders) {
  order.user = await userService.findById(order.userId);  // N queries!
}

// GOOD — 1 query with eager load
const orders = await orderService.findAll({
  include: { user: true },  // single JOIN query
});
```

### Pass 4: Tests

```
[ ] Tests exist for this code
[ ] Happy path is tested
[ ] Error cases are tested
[ ] Edge cases (null, empty, max values) are tested
[ ] Tests are readable — names describe behavior
[ ] Mocks are not over-mocking (testing too much implementation detail)
[ ] Coverage meets the project standard
```

### Pass 5: Code Style & Maintainability

```
[ ] Naming is clear and consistent with codebase conventions
[ ] Functions are ≤50 lines
[ ] Nesting depth ≤4 levels
[ ] No duplicated code (DRY)
[ ] No magic numbers (use named constants)
[ ] No commented-out code
[ ] No TODO comments without a ticket reference
[ ] SOLID principles followed (especially Single Responsibility)
```

## Review Output Format

```markdown
# Code Review: [PR/File Name]
**Date**: [date]
**Reviewer**: security-guardian + code-reviewer

---

## CRITICAL (Must fix before merge)
- [ ] **[File:Line]** [Issue] — [Why it matters] — **Fix**: [Specific fix]

## HIGH (Should fix before release)
- [ ] **[File:Line]** [Issue] — **Fix**: [Specific fix]

## MEDIUM (Fix in next sprint)
- [ ] **[File:Line]** [Issue]

## LOW (Nice to fix)
- [ ] **[File:Line]** [Suggestion]

## INFO (No action required — just notes)
- [Observation about the code]

---

## Passed Checks
- [x] Authentication enforced on all routes
- [x] Parameterized queries throughout
- [x] Input validation on all endpoints
- [x] Tests exist with 85% coverage
- [x] TypeScript strict mode compliant

---

## Verdict
**APPROVED** ✅
OR
**APPROVED WITH CONDITIONS** ⚠️ (merge after fixing MEDIUM issues)
OR
**CHANGES REQUESTED** ❌ (merge blocked until CRITICAL/HIGH are fixed)

[If blocked: "To unblock: [specific changes needed]"]
```

## Quick Review (Small PRs)

For small changes (< 50 lines):
1. Logic check: does it do what the PR title says?
2. Security: any auth/data/secrets concerns?
3. Tests: are they there?
4. Verdict: approve or request changes

Full 5-pass review for:
- Auth/security code
- Payment/financial code
- Database migrations
- Public API changes
- > 100 lines of new code
