---
name: eval-driven-development
description: >
  Eval-Driven Development (EDD) — TDD's counterpart for AI/ML work. Define evals BEFORE
  writing any AI code. Evals are first-class artifacts versioned with the codebase.
  Ship criteria: capability pass@3 >= 0.90, regression pass^3 = 1.00.
  Use for any feature involving LLMs, agents, RAG pipelines, or AI behavior.
---

# Eval-Driven Development (EDD) Skill

> Define what "done" looks like before writing a single line of AI code. Then prove it.

## Core Principle

EDD is to AI/ML what TDD is to general software:

| TDD | EDD |
|-----|-----|
| Write failing test first | Define failing evals first |
| Red → Green → Refactor | Baseline → Implement → Gate |
| Tests are code artifacts | Evals are code artifacts |
| Coverage gates | pass@3 / pass^3 gates |
| CI blocks on test failure | CI blocks on eval gate failure |

**Iron Law:** No AI feature ships without passing evals. Evals defined AFTER implementation defeat the purpose.

---

## The EDD Lifecycle (5 Phases)

### Phase 1: DEFINE (Before Any Code)

Write `.claude/evals/<feature-name>.md` BEFORE implementing anything.

Define:
1. **Capability evals** — what should this AI feature DO?
2. **Regression evals** — what must NOT break?
3. **Grader type** for each eval (see Grader Decision Tree below)
4. **Ship criteria** — what pass@k threshold is required?

```markdown
## EVAL: <feature-name>
**Date:** [date]
**Feature:** [what we're building]
**Ship criteria:** capability pass@3 >= 0.90, regression pass^3 = 1.00

### Capability Evals
| ID | Description | Grader | Threshold |
|----|-------------|--------|-----------|
| C1 | [what it should do] | [code/rule/model/human] | [pass criterion] |
| C2 | [what it should do] | [code/rule/model/human] | [pass criterion] |

### Regression Evals
| ID | Description | Grader | Threshold |
|----|-------------|--------|-----------|
| R1 | [what must not break] | [code/rule/model] | pass^3 = 1.00 |
```

### Phase 2: BASELINE

Run evals against the current state — capture failure signatures.

```bash
# Run eval suite
python -m pytest .claude/evals/ -v --json-report --json-report-file=.claude/evals/baseline.json

# Or with DeepEval
deepeval test run test_evals.py --json-output .claude/evals/baseline.json
```

Document:
- `pass@1`: ran once, result
- `pass@3`: ran 3 times, % passed
- `pass^3`: ran 3 times, ALL passed?
- Failure signatures: what goes wrong, how often

### Phase 3: IMPLEMENT

Build the feature/agent/prompt. Re-run evals on each significant change.

```bash
# After each significant change, re-run and compare to baseline
python -m pytest .claude/evals/ -v

# Track improvement
# Expected: capability evals go from 0% → towards 90%
# Expected: regression evals stay at pass^3 = 1.00
```

### Phase 4: GATE

Before shipping, all gates must pass:

```
GATE 1 — Capability: pass@3 >= 0.90
  → Run each capability eval 3 times
  → At least 90% must pass across all runs
  → If fail: BLOCK. Do not ship.

GATE 2 — Regression: pass^3 = 1.00
  → Run each regression eval 3 times
  → ALL must pass all 3 runs (no flakiness)
  → If fail: BLOCK. Do not ship.

GATE 3 — Non-functional
  → Latency p99 within SLA
  → Cost per query within budget
  → If fail: BLOCK (unless explicitly waived with documented trade-off)
```

```python
# Automated gate runner
def run_edd_gate(eval_suite: str) -> bool:
    capability_results = run_evals(eval_suite, category="capability", runs=3)
    regression_results = run_evals(eval_suite, category="regression", runs=3)

    cap_pass_rate = sum(r.passed for r in capability_results) / len(capability_results)
    reg_all_pass = all(r.passed_all_runs for r in regression_results)

    if cap_pass_rate < 0.90:
        print(f"GATE BLOCKED: Capability pass rate {cap_pass_rate:.0%} < 90%")
        return False

    if not reg_all_pass:
        print("GATE BLOCKED: Regression evals not passing on all 3 runs")
        return False

    print(f"GATE PASSED: capability {cap_pass_rate:.0%}, regression 100%")
    return True
```

### Phase 5: RELEASE SNAPSHOT

Freeze the passing eval baseline for regression tracking.

```
docs/releases/<version>/eval-summary.md
.claude/evals/baseline.json  ← updated with passing state
```

```markdown
## Eval Summary — v1.4.0

**Feature:** RAG pipeline with multi-hop retrieval
**Date:** 2026-03-12

### Capability Results
| Eval | pass@1 | pass@3 | Threshold | Status |
|------|--------|--------|-----------|--------|
| Retrieval precision | 0.88 | 0.92 | >= 0.90 | ✅ |
| Answer faithfulness | 0.91 | 0.94 | >= 0.85 | ✅ |
| Answer relevancy | 0.85 | 0.87 | >= 0.80 | ✅ |

### Regression Results
| Eval | pass^3 | Status |
|------|--------|--------|
| Existing retrieval | 3/3 | ✅ |
| Latency p99 < 3s | 3/3 | ✅ |

### Cost
- Average cost per query: $0.0042
- Budget: $0.01 per query ✅
```

---

## Grader Decision Tree

```
Is the output deterministic?
(file exists, test passes, value equals X, regex matches, exit code 0)
  YES → Code Grader (grep, bash assertions, pytest, exit codes)
   NO → Is it structurally constrained?
        (valid JSON schema, correct format, required fields present)
          YES → Rule Grader (schema validation, regex, structural checks)
           NO → Is it a quality / reasoning judgment?
                (good explanation, faithful to context, relevant answer)
                  YES → Model Grader (Claude-as-judge with rubric)
                   NO → Human Grader (security, legal, highly subjective)
```

### Code Grader Examples

```python
# File exists
def test_migration_created(feature: str):
    assert Path(f"migrations/{feature}.sql").exists(), "Migration file not created"

# Command succeeds
def test_build_passes():
    result = subprocess.run(["npm", "run", "build"], capture_output=True)
    assert result.returncode == 0, f"Build failed:\n{result.stderr.decode()}"

# Output matches
def test_structured_output(response: dict):
    schema = {"type": "object", "required": ["answer", "sources"]}
    jsonschema.validate(response, schema)

# Latency within SLA
def test_latency_sla(query_func, query: str):
    start = time.time()
    query_func(query)
    latency_ms = (time.time() - start) * 1000
    assert latency_ms < 3000, f"p99 latency {latency_ms:.0f}ms exceeds 3000ms SLA"
```

### Model Grader — Claude-as-Judge

```python
FAITHFULNESS_RUBRIC = """
You are evaluating whether an AI response is faithful to the provided context.

Context:
{context}

Question: {question}
Response: {response}

Score the response on faithfulness (0-10):
- 10: Every claim is directly supported by the context
- 7-9: Most claims supported; minor extrapolation
- 4-6: Some claims not in context
- 1-3: Multiple unsupported claims
- 0: Hallucination — major claims contradict or aren't in context

Respond with JSON: {{"score": N, "reason": "one sentence explanation"}}
"""

def model_grader_faithfulness(context: str, question: str, response: str) -> dict:
    result = anthropic.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=200,
        messages=[{"role": "user", "content": FAITHFULNESS_RUBRIC.format(
            context=context, question=question, response=response
        )}],
    )
    return json.loads(result.content[0].text)
```

### Rule Grader Examples

```python
def rule_grader_format(response: str) -> bool:
    """Check structural constraints."""
    try:
        data = json.loads(response)
        required_fields = {"answer", "sources", "confidence"}
        return required_fields.issubset(data.keys())
    except json.JSONDecodeError:
        return False

def rule_grader_no_hallucination_keywords(response: str) -> bool:
    """Reject responses with hedging that indicates uncertainty about facts."""
    hallucination_indicators = [
        "I believe", "I think", "probably", "I'm not sure but",
        "you might want to check"
    ]
    return not any(phrase.lower() in response.lower() for phrase in hallucination_indicators)
```

---

## pass@k Quick Reference

| Metric | Definition | Use For |
|--------|-----------|---------|
| `pass@1` | Ran once, passed | Baseline measurement only — not sufficient |
| `pass@3` | Ran 3 times, ≥90% passed | Capability evals — practical reliability |
| `pass^3` | Ran 3 times, ALL 3 passed | Regression evals — zero-tolerance stability |
| `pass@5` | Ran 5 times, ≥80% passed | High-stakes capability evals |

```python
def pass_at_k(eval_fn, k: int, threshold: float = 0.9) -> bool:
    """Run eval k times, return True if threshold% pass."""
    results = [eval_fn() for _ in range(k)]
    return sum(results) / k >= threshold

def pass_caret_k(eval_fn, k: int) -> bool:
    """Run eval k times, return True only if ALL pass (stability)."""
    return all(eval_fn() for _ in range(k))
```

---

## EDD Templates

### RAG Pipeline Evals

```markdown
## EVAL: rag-<feature-name>

### Capability Evals
- [ ] C1: Retrieval precision — top-5 results contain relevant doc (precision@5)
      Grader: RAGAS context_precision, threshold >= 0.75
- [ ] C2: Answer faithfulness — response grounded in context
      Grader: RAGAS faithfulness, threshold >= 0.85
- [ ] C3: Answer relevancy — response addresses the question
      Grader: RAGAS answer_relevancy, threshold >= 0.80
- [ ] C4: Context recall — relevant docs actually retrieved
      Grader: RAGAS context_recall, threshold >= 0.75
- [ ] C5: Zero hallucination — no claims outside context
      Grader: Model grader (Claude-as-judge), threshold = 0%

### Regression Evals
- [ ] R1: Existing benchmark test suite passes
      Grader: Code grader (pytest), pass^3 = 1.00
- [ ] R2: Latency p99 < 3s
      Grader: Code grader (timing assertion), pass^3 = 1.00
- [ ] R3: Cost per query < $0.01
      Grader: Code grader (token count), pass^3 = 1.00
```

### Agent Behavior Evals

```markdown
## EVAL: agent-<feature-name>

### Capability Evals
- [ ] C1: Task completion — agent completes the assigned task
      Grader: Code grader (check output artifacts exist), threshold pass@3 >= 0.90
- [ ] C2: Tool use accuracy — correct tools called in right order
      Grader: Rule grader (check tool call sequence), threshold pass@3 >= 0.85
- [ ] C3: Error handling — agent recovers from tool failures
      Grader: Code grader (inject failures, verify recovery), threshold pass@3 >= 0.80
- [ ] C4: Output quality — final output meets spec
      Grader: Model grader with rubric, threshold pass@3 >= 0.85

### Regression Evals
- [ ] R1: Existing agent test cases pass
      Grader: Code grader, pass^3 = 1.00
- [ ] R2: No infinite loops (max_turns respected)
      Grader: Code grader, pass^3 = 1.00
```

---

## Storage Convention

```
.claude/
  evals/
    <feature-name>.md       ← eval definition + criteria (written FIRST)
    <feature-name>.log      ← run history (auto-generated)
    baseline.json           ← regression baselines (frozen at release)
docs/
  releases/
    <version>/
      eval-summary.md       ← release snapshot
```

---

## EDD Iron Law Anti-Patterns

These are never acceptable — no exceptions:

```
❌ Defining evals AFTER implementation (defeats the purpose of EDD)
❌ Overfitting prompts to known eval examples (eval set leakage)
❌ Measuring only happy-path outputs (evil inputs exist in production)
❌ Ignoring cost and latency drift while chasing pass rates
❌ Allowing flaky graders in release gates (non-deterministic graders fail randomly)
❌ Trusting "it worked once" (pass@1 is not sufficient for release)
❌ Writing evals that always pass (make sure baseline shows failures before implementing)
❌ Shipping AI features without running the gate
```

---

## Worked Example: Add Streaming to FastAPI Endpoint

### Phase 1: DEFINE (Before Any Code)

```markdown
## EVAL: fastapi-streaming-endpoint

### Capability Evals
- C1: Stream starts within 500ms of request
      Grader: Code grader (time-to-first-byte assertion)
- C2: All chunks received correctly (no truncation)
      Grader: Code grader (concatenate chunks == full response)
- C3: Stream handles client disconnection gracefully
      Grader: Code grader (no 500 error after disconnect)
- C4: Response content is valid and coherent
      Grader: Model grader (coherence rubric), threshold >= 0.85

### Regression Evals
- R1: Non-streaming endpoint still works
      Grader: Code grader (existing test suite), pass^3 = 1.00
- R2: Error responses still JSON (not SSE)
      Grader: Rule grader (Content-Type check on errors), pass^3 = 1.00
```

### Phase 2: BASELINE

Run current tests → all streaming tests fail (RED — expected).

### Phase 3: IMPLEMENT

Add `StreamingResponse` to FastAPI endpoint. Re-run evals after each change.

### Phase 4: GATE

```bash
# Run capability evals 3 times
pytest test_streaming.py -k "capability" --count=3
# Expected: >= 90% pass rate

# Run regression evals 3 times
pytest test_streaming.py -k "regression" --count=3
# Expected: 100% (all 3 runs, all pass)
```

### Phase 5: RELEASE SNAPSHOT

Save to `docs/releases/1.5.0/eval-summary.md`. Update `baseline.json`.
