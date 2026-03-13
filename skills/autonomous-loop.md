---
name: autonomous-loop
description: >
  Self-correcting autonomous build loop. Plan → execute → verify → fix → repeat.
  Uses a structured loop with exit conditions to build complex features without
  human intervention. Activate via /loop command. Requires PLAN.md or clear spec.
---

# Autonomous Loop Skill

> Build it. Check it. Fix it. Repeat until done or escalate.

## When to Use

- Multi-step implementation tasks where you want unattended execution
- Features where you can write the verification criteria upfront
- Bug fixing where the fix can be tested automatically
- Refactoring with a clear "before/after" test suite

## Loop Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    AUTONOMOUS LOOP                       │
│                                                          │
│  [PLAN] ──→ [EXECUTE] ──→ [VERIFY] ──→ [DONE? ✓]       │
│                  ↑            │                          │
│                  └── [FIX] ←──┘ (if failed, max 3x)     │
│                       │                                  │
│               [ESCALATE? ✓] if 3 fails                  │
└─────────────────────────────────────────────────────────┘
```

## Implementation

### Loop State

Maintain loop state in `.claude/loop-state.md`:

```markdown
# Loop State

**Task**: [description]
**Started**: [timestamp]
**Iteration**: N of [max]
**Status**: running | paused | completed | escalated

## Phases
- [x] Phase 1: Setup — COMPLETED
- [ ] Phase 2: Core logic — IN PROGRESS (attempt 2/3)
- [ ] Phase 3: Tests — PENDING
- [ ] Phase 4: Security — PENDING

## Current Iteration Log
**Attempt**: 2
**Action**: Implementing JWT refresh rotation
**Failure**: `TypeError: Cannot read property 'userId' of undefined at line 42`
**Fix attempted**: Added null check on token payload

## Verification Results
| Check | Result | Detail |
|-------|--------|--------|
| typecheck | ❌ FAIL | 3 errors in auth.service.ts |
| unit tests | ❌ FAIL | 2 tests fail |
| lint | ✅ PASS | |

## Escalation Triggers
- 3 consecutive failures on same step
- Security vulnerability found
- Database schema change needed (not in plan)
- Ambiguous requirement
```

### Loop Execution Protocol

```python
# Pseudocode for the loop behavior

MAX_RETRIES = 3
phases = load_plan("PLAN.md")

for phase in phases:
    retries = 0
    while retries < MAX_RETRIES:
        # Execute phase
        execute_phase(phase)

        # Verify phase
        results = verify_phase(phase)

        if results.all_pass:
            mark_phase_complete(phase)
            break

        # Failed — analyze and fix
        failure_analysis = analyze_failures(results)

        if failure_analysis.is_escalation_trigger:
            escalate_to_user(failure_analysis)
            return

        # Apply fix and retry
        apply_fix(failure_analysis)
        retries += 1
        update_loop_state(phase, attempt=retries, failure=failure_analysis)

    if retries == MAX_RETRIES:
        escalate_to_user(f"Phase {phase.name} failed after {MAX_RETRIES} attempts")
        return

# All phases complete
deliver_results()
```

### Verification Commands

Define per-phase verification in PLAN.md:

```markdown
### Phase 1 Verification
\`\`\`bash
# These commands must ALL exit 0 to pass
npm run typecheck
npm run test:unit -- --testPathPattern=auth
npm run lint
\`\`\`

### Phase 2 Verification
\`\`\`bash
npm run test:integration -- --testPathPattern=users
curl -s http://localhost:3000/health | jq '.status == "ok"'
\`\`\`
```

### Fix Strategy per Failure Type

| Failure | Fix Strategy |
|---------|-------------|
| TypeScript error | Read error, fix type annotation, check propagation |
| Test failure | Read test + implementation, find mismatch, fix implementation |
| Lint error | Apply specific lint fix, don't auto-disable rules |
| Import error | Check file paths, verify exports, check index files |
| Runtime error | Add error handling, check null cases, log more context |
| DB error | Check migration status, verify connection, check schema |
| Build error | Check dependencies, clear cache, check config |

### Fix Protocol

For each failure:
1. **Read the full error** — don't skim, include stack trace
2. **Locate the source** — find the exact file and line
3. **Understand the cause** — why is this happening?
4. **Apply minimal fix** — change only what fixes the specific error
5. **Re-run verification** — don't assume the fix worked

```bash
# Always re-run full verification after any fix
npm run typecheck && npm run test:unit && npm run lint
```

## Escalation Triggers (Stop Immediately)

Escalate to user if:
1. Same step fails 3 consecutive times
2. A security vulnerability is found
3. A DB schema change is needed that wasn't in the plan
4. A requirement is ambiguous (can't determine correct behavior)
5. A breaking change to a public API is needed
6. Secrets or credentials are required

Escalation format:
```markdown
## Loop Escalation Required

**Status**: BLOCKED after N attempts
**Phase**: [phase name]
**Issue**: [clear description of the problem]

**What I tried**:
1. [attempt 1 and why it failed]
2. [attempt 2 and why it failed]

**What I need from you**:
- [Specific question or decision needed]
OR
- [Specific value/credential needed]
OR
- [Approval to change the plan]

**Context**:
[Relevant code snippet or error message]
```

## Loop Exit Conditions

**Success**: All phases complete, all verifications pass
**Escalation**: Escalation trigger hit
**Time limit**: Max iterations reached (default: 10)
**Manual stop**: User types "stop" or sends interrupt

## Example Usage

```
User: "Build the user authentication system per SPEC.md. Use /loop."

Loop behavior:
1. Read SPEC.md and generate PLAN.md (4 phases)
2. Phase 1: DB migration
   - Create migration file ✓
   - Apply migration ✓
   - Verify: schema matches spec ✓
3. Phase 2: Service layer
   - Write AuthService ✓
   - Run unit tests → 2 failures
   - Fix: token validation edge case ✓
   - Re-run → all pass ✓
4. Phase 3: API routes
   - Write controllers ✓
   - Run integration tests → 1 failure (missing middleware)
   - Fix: add rate limiting middleware ✓
   - Re-run → all pass ✓
5. Phase 4: Security review
   - Run security checks → 1 issue (missing CSRF protection)
   - Add CSRF middleware ✓
   - Security review passes ✓
6. All phases complete → deliver summary
```

## Integration with Sequential Pipeline

For long tasks, use `claude -p` sequential pipeline pattern:

```bash
#!/bin/bash
set -e

# Step 1: Research + Plan
claude -p "Read SPEC.md. Run research-first skill. Create PLAN.md with 4 phases."

# Step 2-N: Execute each phase with verification
claude -p "Read PLAN.md. Execute Phase 1. Verify per Phase 1 checklist. Update loop state."
claude -p "Read PLAN.md and loop-state.md. Execute Phase 2. Verify. Update loop state."

# Final: Deliver
claude -p "Read PLAN.md and loop-state.md. All phases complete? Generate delivery summary."
```
