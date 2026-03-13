---
name: review
description: Multi-pass code review — logic, security, performance, style, tests.
allowed-tools: ["Read", "Grep", "Glob", "Bash"]
argument-hint: "[file, directory, or PR number]"
---

<objective>
Perform a thorough multi-pass code review of: $ARGUMENTS

Cover: logic correctness, security (OWASP Top 10), performance, code style,
test coverage, documentation. Produce an actionable review with severity levels.
</objective>

<execution_context>
@agents/tier2/security-guardian.md
@agents/tier2/qa-champion.md
@skills/research-first.md
</execution_context>

<context>
**Review Target**: $ARGUMENTS

```bash
# If a PR number:
gh pr diff $ARGUMENTS 2>/dev/null || git diff HEAD~1 -- $ARGUMENTS
```
</context>

<process>
1. Identify all changed/target files
2. Pass 1: Logic review (correctness, edge cases, error handling)
3. Pass 2: Security review (OWASP Top 10, secrets, auth, injection)
4. Pass 3: Performance review (N+1, unbounded queries, memory leaks)
5. Pass 4: Tests review (coverage, quality of assertions)
6. Pass 5: Style review (conventions, naming, complexity)
7. Produce structured review with CRITICAL/HIGH/MEDIUM/LOW/INFO

Final verdict: APPROVED | APPROVED_WITH_CONDITIONS | CHANGES_REQUESTED
</process>
