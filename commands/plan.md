---
name: plan
description: Activate planner agent to create PLAN.md for the requested feature or task.
allowed-tools: ["Read", "Grep", "Glob", "Write"]
argument-hint: "<feature or task description>"
---

<objective>
Create a detailed, executable PLAN.md for: $ARGUMENTS

Use the planner agent. The plan must be specific enough that a builder can implement it
without asking questions. Include file paths, success criteria, and verification steps.
</objective>

<execution_context>
@agents/tier2/planner.md
@skills/research-first.md
</execution_context>

<context>
**Task**: $ARGUMENTS

**Current directory**: Run `ls` and `cat CLAUDE.md 2>/dev/null || true` to understand the project.
</context>

<process>
1. Run research-first skill on current directory
2. Analyze the task requirements
3. Create PLAN.md with phased implementation
4. Output plan summary to user
</process>

<success_criteria>
- [ ] PLAN.md created with specific file paths
- [ ] All phases have verification criteria
- [ ] Dependencies between phases are explicit
- [ ] No ambiguous steps
</success_criteria>
