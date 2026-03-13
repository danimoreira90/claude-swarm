# Anti-Cheat Discipline — Iron Law

**Status:** ACTIVE — Non-negotiable. No exceptions.

---

## The Iron Law

> **NEVER fake a passing state.**
> **Run real tests. Show real output.**
> **If tests fail, fix the code — not the test.**

This is not a preference. This is the line between working software and theater.

---

## What "Fake a Passing State" Means

You are faking a passing state when you:

- Report "tests pass" without running them
- Paste invented or cached output instead of fresh command output
- Skip failing tests with `pytest.skip` / `@skip` / `xfail` instead of fixing the code
- Swallow exceptions with `except: pass` so failures disappear silently
- Hardcode return values in functions under test (`return True`, `return []`)
- Gate real behavior behind `if TEST_MODE:` or `if CI:` so tests never hit production paths
- Comment out assertions that would fail (`# assert result == expected`)
- Add `# pragma: no cover` to logic that should be tested
- Report a coverage percentage without showing `coverage report` output
- Summarize CI results without a run URL or pasted output

**Every one of these is a lie about the state of the software.**

---

## The Mandatory Protocol

### When tests pass:

```
REQUIRED: Paste the actual pytest/jest/go test output.
REQUIRED: Paste the actual coverage report output.
NOT SUFFICIENT: "All tests passed."
NOT SUFFICIENT: "Coverage is 87%."
```

### When tests fail:

```
REQUIRED: Paste the full failure output including traceback.
REQUIRED: Fix the production code that caused the failure.
FORBIDDEN: Modify the test to accept wrong behavior.
FORBIDDEN: Skip the test.
FORBIDDEN: Add exception handling to hide the failure.
```

### When you find a cheat pattern:

```
REQUIRED: STOP immediately.
REQUIRED: Report the exact file and line.
REQUIRED: Fix it.
REQUIRED: Re-run and show the new output.
FORBIDDEN: Proceed past a detected cheat pattern.
```

---

## Rationalization Table

Every agent will eventually produce one of these arguments. All of them are wrong.

| Argument | Why It's Wrong |
|----------|----------------|
| "The test is too brittle, skip is cleaner" | Fix the brittleness. Skips hide real problems. |
| "Showing full output is too long" | Paste the summary block. Do not invent it. |
| "The code is obviously correct" | Tests prove it. Opinions don't. |
| "I tested it manually" | Manual results don't appear in CI. Run the command. |
| "This is just a spike" | Say that explicitly and mark every file `# SPIKE`. |
| "The exception is harmless" | Handle it explicitly. Silent swallowing is a lie. |
| "I'll fix the test coverage later" | Fix it now. "Later" shipped as never 100% of the time. |
| "The mock is equivalent to real behavior" | Verify that claim by running against real behavior. |

---

## Red Flags — Immediate Stop Required

Any of the following in a diff or command output means **STOP before proceeding**:

```
pytest.skip(
pytest.mark.xfail
@unittest.skip
except Exception: pass
except Exception: return None
return True  # (in a stub or untested function)
if os.environ.get("TEST_MODE")
if os.environ.get("CI")
# assert
# pragma: no cover   ← on non-trivial logic
--omit              ← covering real application logic
```

---

## Scope

This rule applies to:
- All test output reported in any session
- All CI status reported or committed
- All coverage numbers stated in any context
- All "done" claims on tasks that include code changes
- All mock usage in tests
- All exception handling in new code

There is no context where faking a passing state is acceptable.

---

## Relationship to Other Iron Laws

This rule enforces the output integrity of:
- **TDD** (`rules/tdd-discipline.md`) — TDD only works if the RED step is real
- **EDD** (`rules/edd-discipline.md`) — Evals only gate if results are real
- **Verification** (`rules/verification-discipline.md`) — Verification requires real output

A fake green test breaks all three simultaneously.

---

## Skill Reference

Full grep patterns, SUBAGENT-STOP protocol, and the 8-technique anti-rationalization framework:
→ `skills/anti-cheat/SKILL.md`
