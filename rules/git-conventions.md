# Git Conventions

> These conventions keep the git history readable, the CI fast, and releases automatic.

## Commit Format

**[Conventional Commits v1.0](https://www.conventionalcommits.org/)**

```
<type>(<scope>): <subject>

[optional body — explain WHY, not what]

[optional footer: BREAKING CHANGE, Closes #N, Refs #N]
```

### Types

| Type | When | Version Bump |
|------|------|-------------|
| `feat` | New feature | MINOR |
| `fix` | Bug fix | PATCH |
| `perf` | Performance improvement | PATCH |
| `refactor` | Code restructure, no behavior change | none |
| `test` | Tests only | none |
| `docs` | Documentation only | none |
| `ci` | CI/CD pipeline changes | none |
| `chore` | Maintenance, dependency updates | none |
| `build` | Build system changes | none |
| `style` | Formatting only | none |

### Rules
- Subject: imperative mood (`add`, not `added`)
- Subject: ≤72 characters
- Subject: lowercase (no capital start unless proper noun)
- Subject: no period at the end
- Breaking change: append `!` after type (`feat!:`) or add `BREAKING CHANGE:` footer
- Reference issues: `Closes #123` (auto-closes), `Fixes #456`, `Refs #789`

### Examples
```
feat(auth): add JWT refresh token rotation
fix(users): prevent duplicate email registration on case mismatch
perf(db): add composite index on (user_id, created_at) — 40% query improvement
refactor(payments): extract Stripe client to dedicated service class
test(auth): add edge cases for expired token and missing claims
docs(api): update OpenAPI spec for v2 subscription endpoints
ci: add PostgreSQL 16 service to integration test workflow
chore(deps): update all dependencies to latest minor versions
feat!: rename /api/v1/users to /api/v2/members

BREAKING CHANGE: The /api/v1/users endpoint is removed.
Migrate to /api/v2/members. See migration guide in docs/migrations/v2.md.
```

## Branching

### Feature Branches
```bash
git checkout -b feature/add-stripe-webhooks
git checkout -b feature/JIRA-456-user-onboarding
```

### Fix Branches
```bash
git checkout -b fix/null-pointer-user-service
git checkout -b hotfix/security-jwt-expiry   # for urgent production fixes
```

### Branch Rules
- Branch from: `develop` (or `main` for trunk-based)
- Merge to: `develop` via PR (never direct push to main/develop)
- Delete after merge
- Keep short-lived (< 1 week ideal)
- Name: lowercase, hyphens, no spaces

## Pull Requests

### PR Title: same format as commit messages
```
feat(auth): add JWT refresh token rotation
fix: resolve race condition in payment processing
```

### PR Description Template
```markdown
## Summary
- [What changed — specific]
- [Why — business or technical reason]

## Changes
| File | What Changed |
|------|-------------|
| `src/auth/jwt.service.ts` | Added refresh token rotation |
| `tests/auth/jwt.service.spec.ts` | Tests for rotation edge cases |

## Testing
- [ ] Unit tests pass (`npm test`)
- [ ] Integration tests pass
- [ ] Manual testing: [steps]

## Checklist
- [ ] Conventional commit format
- [ ] Tests added/updated
- [ ] No TypeScript errors
- [ ] No new lint errors
- [ ] No secrets committed
- [ ] Documentation updated (if needed)
- [ ] Breaking changes documented (if any)
```

## Merge Strategy

- **Feature PRs → develop**: Squash merge (clean history)
- **Hotfix PRs → main**: Merge commit (preserve context)
- **develop → main** (release): Merge commit with release tag

## Tags and Releases

```bash
# Release tag format: v{MAJOR}.{MINOR}.{PATCH}
git tag -a v1.4.0 -m "Release v1.4.0: Add Stripe subscriptions"
git push origin v1.4.0

# Pre-release
git tag -a v2.0.0-beta.1 -m "Beta release for v2 API"
```

## .gitignore Requirements

Always gitignore:
```
.env
.env.local
.env.*.local
node_modules/
dist/
build/
.next/
__pycache__/
*.pyc
.pytest_cache/
.mypy_cache/
*.log
.DS_Store
coverage/
.nyc_output/
```
