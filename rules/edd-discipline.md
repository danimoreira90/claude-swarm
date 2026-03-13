# IRON LAW: EDD Discipline

> **NO AI FEATURE SHIPS WITHOUT PASSING EVALS.**

This rule is absolute. Eval-Driven Development (EDD) is to AI/ML what TDD is to general software. Evals are defined before implementation. Evals are first-class code artifacts. The ship gate requires passing evals.

---

## The Law

**Before writing any AI/ML code:**

1. Define capability evals — what must the AI DO?
2. Define regression evals — what must NOT break?
3. Capture baseline — confirm evals fail before implementation (like RED in TDD)
4. Implement — re-run evals on every significant change
5. Gate — capability pass@3 >= 0.90, regression pass^3 = 1.00
6. Release snapshot — freeze passing baseline

**Evals defined AFTER implementation defeat the purpose of EDD. This is prohibited.**

---

## The EDD Lifecycle

```
DEFINE evals (before any code)
  ↓
BASELINE (confirm current state fails — like RED)
  ↓
IMPLEMENT (re-run evals on each change)
  ↓
GATE (pass@3 >= 0.90 and pass^3 = 1.00)
  ↓
RELEASE SNAPSHOT (freeze baseline)
```

---

## Ship Criteria (Non-Negotiable)

```
GATE 1 — Capability: pass@3 >= 0.90
  → Run each capability eval 3 times
  → At least 90% pass across all runs
  → FAIL = BLOCK. Do not ship.

GATE 2 — Regression: pass^3 = 1.00
  → Run each regression eval 3 times
  → ALL must pass ALL 3 runs (no flakiness)
  → FAIL = BLOCK. Do not ship.

GATE 3 — Non-functional
  → Latency p99 within SLA
  → Cost per query within budget
  → FAIL = BLOCK (unless waived with documented trade-off)
```

---

## Eval Definition (Before Any Code)

Create `.claude/evals/<feature-name>.md` BEFORE implementation:

```markdown
## EVAL: <feature-name>
**Date:** [date]
**Feature:** [what we're building]
**Ship criteria:** capability pass@3 >= 0.90, regression pass^3 = 1.00

### Capability Evals
| ID | Description | Grader | Threshold |
|----|-------------|--------|-----------|
| C1 | [what it should do] | code/rule/model | pass@3 >= 0.90 |
| C2 | [what it should do] | code/rule/model | pass@3 >= 0.90 |

### Regression Evals
| ID | Description | Grader | Threshold |
|----|-------------|--------|-----------|
| R1 | [what must not break] | code | pass^3 = 1.00 |
| R2 | Latency p99 < [SLA] | code | pass^3 = 1.00 |
| R3 | Cost per query < $[budget] | code | pass^3 = 1.00 |
```

---

## Grader Selection

```
Is the output deterministic?
(file exists, test passes, value equals X, exit code 0)
  YES → Code Grader

Is it structurally constrained?
(valid JSON schema, required fields, correct format)
  YES → Rule Grader

Is it a quality / reasoning judgment?
(good explanation, faithful to context, relevant answer)
  YES → Model Grader (Claude-as-judge with rubric)

Is it highly subjective / legal / security?
  YES → Human Grader
```

---

## pass@k Reference

| Metric | Definition | Required For |
|--------|-----------|--------------|
| `pass@1` | Ran once, passed | Baseline measurement only |
| `pass@3` | Ran 3 times, ≥90% passed | All capability evals |
| `pass^3` | Ran 3 times, ALL passed | All regression evals |
| `pass@5` | Ran 5 times, ≥80% passed | High-stakes capability evals |

```python
def pass_at_k(eval_fn, k: int, threshold: float = 0.9) -> bool:
    results = [eval_fn() for _ in range(k)]
    return sum(results) / k >= threshold

def pass_caret_k(eval_fn, k: int) -> bool:
    return all(eval_fn() for _ in range(k))
```

---

## Gate Runner

```python
def run_edd_gate(eval_suite: str) -> bool:
    capability_results = run_evals(eval_suite, category="capability", runs=3)
    regression_results = run_evals(eval_suite, category="regression", runs=3)

    cap_pass_rate = sum(r.passed for r in capability_results) / len(capability_results)
    reg_all_pass = all(r.passed_all_runs for r in regression_results)

    if cap_pass_rate < 0.90:
        print(f"GATE BLOCKED: Capability {cap_pass_rate:.0%} < 90%")
        return False

    if not reg_all_pass:
        print("GATE BLOCKED: Regression evals not passing on all 3 runs")
        return False

    print(f"GATE PASSED: capability {cap_pass_rate:.0%}, regression 100%")
    return True
```

---

## Eval Storage Convention

```
.claude/
  evals/
    <feature-name>.md       ← eval definition (written FIRST, committed)
    <feature-name>.log      ← run history (auto-generated)
    baseline.json           ← regression baselines (frozen at release)
docs/
  releases/
    <version>/
      eval-summary.md       ← release snapshot
```

---

## EDD Iron Laws

```
❌ NEVER define evals after implementation (defeats the purpose)
❌ NEVER overfit prompts to known eval examples (eval set leakage)
❌ NEVER measure only happy-path outputs (evil inputs exist in production)
❌ NEVER ignore cost and latency drift while chasing pass rates
❌ NEVER allow flaky graders in release gates
❌ NEVER trust "it worked once" — pass@1 is not sufficient for release
❌ NEVER write evals that always pass (make sure baseline shows failures)
❌ NEVER ship AI features without running the gate
✅ ALWAYS define evals before the first line of AI code
✅ ALWAYS commit eval files alongside the implementation
✅ ALWAYS run the gate before shipping
✅ ALWAYS save a release snapshot after gate passes
```

---

## What Counts as an "AI Feature"

EDD is required for ALL of these:

```
✅ Any LLM call (Claude, GPT, Gemini, local model)
✅ RAG pipelines (retrieval + generation)
✅ Multi-agent workflows (orchestrators, subagents)
✅ Embedding generation and similarity search
✅ Prompt templates with dynamic inputs
✅ AI-powered classifiers, extractors, summarizers
✅ Structured output generation (tool use, JSON mode)
✅ Streaming AI responses
```

EDD may be skipped ONLY for:
```
✅ SKIP: Non-AI utility functions called by AI features
✅ SKIP: Configuration for AI infrastructure (env vars, URLs)
✅ SKIP: Pure data preprocessing with no AI component
```
