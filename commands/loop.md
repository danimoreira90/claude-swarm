---
name: loop
description: >
  Start autonomous self-correcting build loop. Plan → execute → verify → fix → repeat.
  Use for complex features that need unattended execution with quality gates.
allowed-tools: ["Read", "Write", "Bash", "Glob", "Grep", "Task"]
argument-hint: "<feature description or 'from PLAN.md'>"
---

<objective>
Start autonomous build loop for: $ARGUMENTS

Execute the autonomous-loop skill. Self-correct on failures up to 3 times per phase.
Escalate to user if: security issue, DB schema change needed, 3 consecutive failures,
or ambiguous requirement.
</objective>

<execution_context>
@skills/autonomous-loop.md
@agents/tier1/orchestrator.md
@skills/research-first.md
</execution_context>

<context>
**Loop Task**: $ARGUMENTS

Initialize loop state:
```bash
mkdir -p .claude
cat > .claude/loop-state.md << 'EOF'
# Loop State
**Task**: $ARGUMENTS
**Status**: starting
**Iteration**: 0
EOF
```
</context>

<process>
1. Research-first: map codebase
2. Create or load PLAN.md
3. Start loop: execute phase → verify → fix if needed (max 3x) → next phase
4. Maintain .claude/loop-state.md throughout
5. Escalate immediately on: security issues, DB schema changes, 3 failures
6. Deliver completion summary on success
</process>

<success_criteria>
- [ ] All PLAN.md phases completed
- [ ] All verifications pass
- [ ] Tests pass
- [ ] No escalation conditions triggered
</success_criteria>
