---
name: spec-driven-dev
description: >
  Spec-driven development workflow. SPEC.md → PLAN.md → implementation → verification loop.
  Ensures features are fully specified before a single line is written.
  Use for any non-trivial feature or when requirements are unclear.
---

# Spec-Driven Development Skill

> Write the spec first. Build to the spec. Verify against the spec.

## Philosophy

Code without a spec is a guess. Specs force you to think through the full problem before writing a single line. They prevent half-baked implementations, missing edge cases, and "we need to refactor this" moments three weeks later.

A good spec answers:
- What does this do? (behavior)
- What does this NOT do? (scope limits)
- What are the success criteria? (measurable)
- What are the edge cases? (completeness)
- What are the security considerations? (safety)

## The Loop

```
SPEC.md → review → PLAN.md → implement → verify → iterate
    ↑                                          |
    └──────────── refine if needed ────────────┘
```

## Step 1: Write SPEC.md

Create `SPEC.md` (or `docs/specs/feature-name.md` for organized projects):

```markdown
# Spec: [Feature Name]

**Status**: Draft | Under Review | Approved | Implemented
**Date**: [date]
**Author**: [who]

## Problem Statement
[What problem does this feature solve? Why does it need to exist?]

## User Stories

### Primary
> As a [user type], I want to [do something] so that [I get value].

**Acceptance Criteria**:
- [ ] [Specific, testable criterion]
- [ ] [Specific, testable criterion]

### Secondary (if applicable)
> As a [user type], I want to [do something else].

**Acceptance Criteria**:
- [ ] [Specific, testable criterion]

## Scope

### In Scope
- [What this feature WILL do]
- [What this feature WILL do]

### Out of Scope
- [What this feature will NOT do (equally important)]
- [Save this for v2: ...]

## Functional Requirements

| ID | Requirement | Priority |
|----|-------------|----------|
| F1 | [Specific requirement] | Must Have |
| F2 | [Specific requirement] | Must Have |
| F3 | [Specific requirement] | Should Have |
| F4 | [Specific requirement] | Nice to Have |

## Non-Functional Requirements

| Requirement | Target |
|-------------|--------|
| Latency (p99) | < 200ms |
| Throughput | 1000 req/s |
| Availability | 99.9% |
| Data retention | 90 days |

## API Contract (if applicable)

### Request
\`\`\`
POST /api/v1/[resource]
Authorization: Bearer {token}

{
  "field1": "string",
  "field2": 123
}
\`\`\`

### Response
\`\`\`
HTTP 201 Created

{
  "id": "uuid",
  "field1": "string",
  "createdAt": "ISO8601"
}
\`\`\`

### Error Cases
| HTTP Status | When | Response Body |
|------------|------|---------------|
| 400 | Invalid input | `{ "error": "validation_error", "details": [...] }` |
| 401 | Missing/invalid token | `{ "error": "unauthorized" }` |
| 409 | Duplicate resource | `{ "error": "already_exists" }` |

## Data Model

\`\`\`
Table: [table_name]
- id: UUID (PK)
- [field]: [type] [constraints]
- created_at: TIMESTAMP
- updated_at: TIMESTAMP

Relationships:
- [table] 1:N [other_table] via [foreign_key]
\`\`\`

## Security Considerations
- [ ] Authentication required? Which roles?
- [ ] Data validation: what inputs need sanitizing?
- [ ] Rate limiting: what limits are appropriate?
- [ ] Sensitive data: any PII or secrets in this feature?
- [ ] Authorization: can users access other users' data?

## Edge Cases

| Case | Expected Behavior |
|------|-------------------|
| Empty input | Return 400 with clear error |
| Concurrent requests | Idempotent or conflict detection |
| DB unavailable | Graceful degradation, clear error |
| [other case] | [expected behavior] |

## Testing Strategy
- **Unit tests**: [What to unit test]
- **Integration tests**: [What flows to test]
- **E2E tests**: [User journeys]
- **Security tests**: [Auth checks, injection attempts]

## Open Questions
- [ ] [Question that needs answering before implementation]
- [ ] [Dependency on another team/feature]

## Changelog
- [date]: Initial draft
```

## Step 2: Spec Review

Before creating PLAN.md, verify the spec answers:

```
✓ All acceptance criteria are testable (not vague)
✓ Edge cases are covered
✓ Security considerations addressed
✓ Data model is complete
✓ API contract is fully specified (request + response + errors)
✓ Out-of-scope is explicitly listed
✓ No open questions remain blocking implementation
```

If any are missing → update the spec first.

## Step 3: PLAN.md (from Spec)

Once spec is approved, create PLAN.md derived from it:

```markdown
# Plan: [Feature Name]
**Spec**: SPEC.md
**Status**: Ready to implement

## Implementation Phases

### Phase 1: Data Layer
Implements: F1, F2
- Task 1.1: Create migration [file path]
- Task 1.2: Create model/entity [file path]
- Task 1.3: Create repository [file path]
Verification: Migration runs clean, model validates correctly

### Phase 2: Service Layer
Implements: F1-F4
- Task 2.1: Create service [file path]
- Task 2.2: Unit tests for service [file path]
Verification: Unit tests pass, 80%+ coverage

### Phase 3: API Layer
Implements: API Contract from spec
- Task 3.1: Create controller/router [file path]
- Task 3.2: Create DTOs [file path]
- Task 3.3: Integration tests [file path]
Verification: All API test cases from spec pass

### Phase 4: Security
Implements: Security considerations from spec
- Task 4.1: Auth guard applied
- Task 4.2: Rate limiting configured
- Task 4.3: Input validation on all endpoints
Verification: Security guardian review approved

## Success Criteria
[Copy acceptance criteria from SPEC.md as checkboxes]
```

## Step 4: Verify Against Spec

After implementation, audit against the spec:

```bash
# Check each acceptance criterion
# For each F1-FN: does the implementation match?
# Run the test cases from the spec
# Verify edge cases are handled
# Confirm security considerations are implemented
```

Update SPEC.md status to `Implemented` with the PR number.

## When to Skip

Spec-driven dev adds overhead. Skip it for:
- Bug fixes (use the bug report as your "spec")
- Refactoring (behavior doesn't change)
- Documentation updates
- Configuration changes

Use it for:
- Any new user-facing feature
- New API endpoints
- New data models
- Significant algorithm changes
- Security-sensitive features
