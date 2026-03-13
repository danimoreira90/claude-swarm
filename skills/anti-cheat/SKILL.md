# Anti-Cheat Skill
**Version:** 1.0 | **Source:** antigravity-awesome-skills/anti-rationalization + superpowers/testing-anti-patterns

> You are an agent that ships real results. Not simulated results. Not approximated results. Real ones.
> **Violating the letter of these rules is violating the spirit of these rules.**

---

## When to Load This Skill

Load automatically when:
- Writing or modifying tests
- Running tests and reporting results
- Reporting coverage, metrics, or CI status
- Adding mocks, skips, or exception handlers
- Tempted to summarize output instead of showing it

---

## The 3 Test Integrity Iron Laws

```
1. NEVER test mock behavior — test what the real code does
2. NEVER add test-only methods to production classes
3. NEVER mock without first understanding the dependency's side effects
```

**Violating any iron law means**: Stop. Delete the offending code. Do it correctly.

---

## The 8-Technique Anti-Rationalization Framework

### Technique 1: Every Loophole Closed Explicitly

Don't just state the rule — forbid specific workarounds.

**No exceptions for:**
- "This function is too simple to need a real test" → It isn't. Write the test.
- "I'll fix the test output display later" → No. Show real output now.
- "The test is passing, I just can't show the output right now" → Show it or it didn't happen.
- "I'll summarize the results to save space" → Paste the actual output.
- "The test technically passes if I just..." → If you're rewriting logic to pass, you're cheating.

### Technique 2: Spirit vs Letter

**The letter IS the spirit.** These rules exist because the specific behaviors they forbid destroy trust in outputs. An agent that fakes a green CI is worse than useless — it actively deceives. Every workaround that "achieves the same goal" does not achieve the same goal.

### Technique 3: Rationalization Table

| Excuse | Reality |
|--------|---------|
| "Tests pass, trust me" | Show the output. Trust is earned with evidence. |
| "I'll add real tests later" | Tests after = "what does this do?" Tests before = "what should this do?" |
| "Skipping this test because it's flaky" | Fix the flakiness. Don't skip. |
| "The mock covers the same behavior" | Mocks lie. Run against real behavior. |
| "Coverage is 80%+ so we're good" | Show the actual `coverage report` output. |
| "The CI is green" | Paste the CI run URL or output. Don't assert it. |
| "This environment doesn't support real tests" | Solve the environment problem. Don't fake the result. |
| "I already checked manually" | Manual checks don't appear in history. Run the command. |

### Technique 4: Red Flags — STOP and Fix Before Proceeding

If you catch yourself doing ANY of the following, **STOP immediately**:

- `pytest.skip`, `pytest.mark.xfail`, `unittest.skip` anywhere in the diff
- `return True` / `return []` / `return {}` hardcoded in a function under test
- `except Exception: pass` or `except Exception: return None` swallowing errors silently
- `if os.environ.get('CI'):` or `if TEST_MODE:` gates that change behavior in test environments
- Assertions commented out: `# assert result == expected`
- `--omit` flags covering substantial logic, or `# pragma: no cover` on non-trivial code
- Reporting "X tests passed" without showing the terminal output
- Reporting "coverage: 85%" without showing `coverage report`

**All of these mean**: Do not proceed. Report the pattern. Fix it. Re-run.

### Technique 5: Pre-Violation Symptoms

You are about to cheat when you think:
- "The user just wants to see it working..."
- "I'll clean up the tests in the next pass..."
- "This skip is temporary..."
- "Showing the full output would be too long..."
- "The important thing is the logic, not the exact test output..."

These are rationalization signals. Stop. Run the real thing. Show the real output.

### Technique 6: Strong Language

- ALWAYS run the actual test command and paste its stdout
- NEVER summarize test results without the raw output attached
- ALWAYS show the coverage report, not just the percentage
- NEVER skip a failing test instead of fixing the underlying code
- ALWAYS show the failure message when tests fail — never hide it

### Technique 7: Commitment & Consistency

You are an agent that claims to do TDD. You claim to verify before shipping. You claim tests are first-class.

**Either:**
- Run real tests and show real output, or
- Admit you are not doing what you claim

Do not redefine "tests passing" to mean "I believe the tests would pass."

### Technique 8: Legitimate Escape Hatches

There are exactly two valid reasons not to run real tests right now:
1. The test infrastructure is genuinely broken and you are actively fixing it (state this explicitly)
2. The user has explicitly asked for a prototype/spike that will be replaced (state this explicitly and mark all code as `# SPIKE - not for production`)

**Everything else**: Run real tests. Show real output. No exceptions.

---

## CHEAT PATTERN GREP LIST

Run these before reporting any test result as passing. If any match, **SUBAGENT-STOP**.

```bash
# 1. Hardcoded returns that fake success
grep -rn "return True$\|return \[\]$\|return {}\|return None  # TODO\|return 0  # stub" app/ src/

# 2. Swallowed exceptions (silent failure hiding)
grep -rn "except.*:\s*pass$\|except.*:\s*return None$\|except Exception as.*:\s*\.\.\." app/ src/

# 3. pytest skip/xfail (test avoidance)
grep -rn "pytest\.skip\|pytest\.mark\.xfail\|unittest\.skip\|@skip\|skipTest" tests/

# 4. Environment-gated fake behavior
grep -rn "if.*TEST_MODE\|if.*CI.*:\|os\.environ\.get.*TEST\|if.*FAKE\|if.*MOCK_.*=.*true" app/ src/

# 5. Commented-out assertions
grep -rn "# assert\|# expect\|# self\.assert\|#assert" tests/

# 6. Coverage gaming (excluding real code)
grep -rn "pragma: no cover" app/ src/
grep -rn "\-\-omit\|omit\s*=" .coverage* setup.cfg pyproject.toml tox.ini

# 7. Fake metrics / hardcoded output values in test fixtures
grep -rn "accuracy.*=.*0\.\(9[5-9]\|[1-9][0-9]\)\|score.*=.*1\.0\|loss.*=.*0\.0" tests/ fixtures/

# 8. Return early without work (stub bodies)
grep -rn "def .*:$" app/ src/ | xargs -I{} grep -A2 "{}" | grep -E "^\s+\.\.\.$|^\s+pass$"
```

If any pattern fires:

```
⛔ SUBAGENT-STOP: Cheat pattern detected.

Pattern: [which grep matched]
File:    [file:line]
Code:    [the offending line]

This is a [skip / swallowed exception / hardcoded return / ...].
Proceeding is not permitted.

Action: [explain what the real fix is]
Re-running after fix.
```

---

## Honest Reporting Mandate

### NEVER do this:

```
✅ All 38 tests passed with 92% coverage.
```

### ALWAYS do this:

```
$ pytest -v --tb=short
========================= test session starts ==========================
platform linux -- Python 3.11.9
collected 38 items

tests/test_auth.py::TestRegister::test_register_success PASSED    [  2%]
...
tests/test_users.py::TestDeleteMe::test_delete_me_success PASSED  [100%]

========================= 38 passed in 9.71s ===========================

$ coverage report --fail-under=80
Name                          Stmts   Miss  Cover
-------------------------------------------------
app/config.py                    18      0   100%
app/services/auth.py             52      4    92%
app/services/user.py             34      2    94%
...
TOTAL                           312     25    92%
```

**Rule**: If the output is too long, paste the summary section. Do not paraphrase it. Do not invent it.

### Confidence levels (from get-shit-done honest reporting mandate):

- `"I couldn't run this"` is valid — state it explicitly with reason
- `"This is LOW confidence"` is valid — flag it before the user relies on it
- `"The test run was incomplete because..."` is valid — explain exactly what happened

Pretending certainty you don't have is a cheat.

---

## SUBAGENT-STOP Protocol

When a cheat pattern is detected at any point during execution:

1. **STOP** — do not proceed to the next step
2. **REPORT** — paste the exact grep match or failure output
3. **FIX** — implement the real solution (not a workaround)
4. **RE-RUN** — execute the actual command again
5. **SHOW** — paste the new output proving the fix worked

There is no step 6. You do not continue until steps 1–5 are complete.

---

## Quick Self-Check Before Reporting "Done"

```
[ ] Did I run the actual test command (not assume it passes)?
[ ] Did I paste the actual terminal output (not paraphrase it)?
[ ] Did I run the grep cheat-pattern list?
[ ] Are there zero pytest.skip / xfail in the diff?
[ ] Are there zero swallowed exceptions in new code?
[ ] Is coverage reported from actual `coverage report` output?
[ ] Are all assertions active (none commented out)?
[ ] Is production code free of test-only methods?
[ ] Did I mock only after understanding the dependency's side effects?
[ ] Are mocks complete (mirror full real API structure)?
```

If any box is unchecked: do not report done.

---

## Anti-Pattern Quick Reference

| Pattern | Iron Law | Fix |
|---------|----------|-----|
| Assert on mock element (`data-testid="*-mock"`) | Iron Law 1 | Test real behavior or unmock |
| `destroy()` method only called in `afterEach` | Iron Law 2 | Move to `test-utils/`, not production class |
| Mock high-level method that test depends on | Iron Law 3 | Mock at lower level, preserve needed side effects |
| Partial mock missing downstream fields | Iron Law 3 | Mirror complete real API structure |
| `pytest.skip` on failing test | Technique 1 | Fix the code the test is failing on |
| `except Exception: pass` | Technique 1 | Handle the exception or let it propagate |
| `return True` stub | Technique 1 | Implement the real logic |
| Summary without output | Mandate | Paste the actual stdout |
