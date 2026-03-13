---
name: test
description: Activate QA champion to write and run tests for specified code.
allowed-tools: ["Read", "Write", "Bash", "Glob", "Grep"]
argument-hint: "[file, module, or 'all']"
---

<objective>
Write and run comprehensive tests for: $ARGUMENTS

Cover: unit tests, integration tests where applicable. Target 80%+ coverage.
Follow TDD principles. Tests should be readable documentation.
</objective>

<execution_context>
@agents/tier2/qa-champion.md
@rules/testing-requirements.md
@skills/research-first.md
</execution_context>

<context>
**Test Target**: $ARGUMENTS

```bash
# Discover existing tests
find . -name "*.spec.ts" -o -name "*.test.ts" -o -name "test_*.py" | head -10
# Check current coverage
npm run test:coverage 2>/dev/null || pytest --cov 2>/dev/null | tail -20
```
</context>

<process>
1. Read target files to understand what to test
2. Read existing test files to match conventions
3. Write tests: happy path → validation → edge cases → errors
4. Run tests, check coverage
5. Fix any issues
6. Report: test count, coverage %, any gaps
</process>
