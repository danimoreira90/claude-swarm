---
name: build
description: Execute PLAN.md or build the described feature using orchestrator + builder.
allowed-tools: ["Read", "Write", "Bash", "Glob", "Grep", "Task"]
argument-hint: "[feature description or 'from PLAN.md']"
---

<objective>
Build: $ARGUMENTS

If a PLAN.md exists, execute it. Otherwise, create a plan first then execute.
Output must be production-ready: tested, typed, linted, documented.
</objective>

<execution_context>
@agents/tier1/orchestrator.md
@agents/tier2/builder.md
@agents/tier2/planner.md
@skills/research-first.md
</execution_context>

<context>
**Build Target**: $ARGUMENTS

Check for PLAN.md:
```bash
cat PLAN.md 2>/dev/null || echo "No PLAN.md — will create one first"
```
</context>

<process>
1. Research-first: map codebase
2. If no PLAN.md: activate planner to create it
3. Activate builder to execute PLAN.md phase by phase
4. Run verification after each phase
5. Report completion with file list
</process>
