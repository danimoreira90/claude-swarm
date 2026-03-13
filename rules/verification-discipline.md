# IRON LAW: Verification Discipline

> **NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE.**

This rule is absolute. There are no exceptions. There are no "I'm confident it works" excuses. Confidence is not evidence. Evidence is evidence.

---

## The Law

**Before you say "done", "complete", "fixed", "implemented", or "working":**

1. **Identify** — What specific command proves this works?
2. **Run** — Execute it. Right now. Not the same session from earlier.
3. **Read** — Read the actual output. Every line.
4. **Verify** — Does the output match what was required?
5. **Claim** — Only now may you say it works.

---

## The 5-Step Protocol (Mandatory)

```
STEP 1 — IDENTIFY: What is the verification command?
  "This task is complete when: [specific command] returns [exact expected output]"

STEP 2 — RUN: Execute the command. Now.
  $ [verification command]

STEP 3 — READ: Read every line of the output.
  [Don't scan — read. Errors hide in scroll-off.]

STEP 4 — VERIFY: Does it match the expected output?
  Expected: Tests: 12 passed, 0 failed
  Actual:   Tests: 12 passed, 0 failed
  ✅ Match → proceed to claim
  ❌ Mismatch → fix, then restart from STEP 2

STEP 5 — CLAIM: State the evidence.
  "Done. Verified: `npm test` → 12 passed, 0 failed. Output: [paste]"
```

---

## Red Flag Words

If you catch yourself using any of these **without fresh verification evidence**, STOP:

```
❌ "should work"          ← run it and check
❌ "I believe it works"   ← belief is not evidence
❌ "it looks correct"     ← looking is not running
❌ "that should fix it"   ← run the test to confirm
❌ "I'm confident"        ← confidence is not evidence
❌ "it's working"         ← which output proves this?
❌ "done"                 ← done according to what evidence?
❌ "complete"             ← complete verified by what command?
```

Replace every red flag word with actual command output.

---

## Verification Commands by Context

| Context | Verification Command | Expected Evidence |
|---------|---------------------|------------------|
| TypeScript/Node | `npm test` or `npx vitest run` | `Tests: N passed, 0 failed` |
| Python | `pytest -v` | `N passed in Xs` |
| TypeScript types | `npx tsc --noEmit` | No output (silence = success) |
| Python types | `mypy src/` | `Success: no issues found in N source files` |
| Lint | `npx biome check src/` or `ruff check src/` | `No errors.` |
| Build | `npm run build` | `Successfully compiled` or exit code 0 |
| Docker | `docker compose up -d && docker compose ps` | All services `Up (healthy)` |
| API endpoint | `curl -s http://localhost:3000/health \| jq .status` | `"ok"` |
| Migration | `npx prisma migrate status` | All migrations applied |

---

## Verification Evidence Format

When claiming completion, always include:

```
✅ VERIFIED

Command: [exact command run]
Output:
  [paste actual output here]

Matches expected: Yes
```

If you cannot paste actual output, you have not verified.

---

## Why This Law Exists

The most common failure mode in AI-assisted development:

1. Agent generates code
2. Agent reads the code and thinks "this looks correct"
3. Agent says "done"
4. Developer runs it → fails

Looking at code is not verification. Running code is verification. Every single time.

This law exists because **"it looks correct" has been wrong thousands of times**. Fresh execution eliminates assumption. There is no substitute.
