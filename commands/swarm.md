---
name: swarm
description: >
  Meta-command: activates the full agent swarm. Orchestrator takes the user's intent,
  decomposes it, and coordinates all needed specialists in parallel to deliver
  complete, production-ready output. Use for: full apps, complex features, multi-domain tasks.
allowed-tools: ["Read", "Write", "Bash", "Glob", "Grep", "Task", "WebFetch"]
---

<objective>
Activate the full Claude Code agent swarm for the task described in $ARGUMENTS.

The orchestrator takes over: it researches, plans, delegates to specialists, synthesizes results,
and delivers production-ready output. No half-measures. Complete or escalate.
</objective>

<execution_context>
@agents/tier1/orchestrator.md
@skills/research-first.md
@skills/autonomous-loop.md
@CLAUDE.md
</execution_context>

<context>
**User Request**: $ARGUMENTS

**Available Swarm Agents**:
- Tier 1: orchestrator (you — the brain)
- Tier 2: planner, architect, builder, git-master, cloud-engineer, security-guardian,
          qa-champion, devops-engineer, ai-ml-engineer, db-architect,
          frontend-expert, backend-expert, doc-engineer

**Skills Available**:
- research-first, spec-driven-dev, tdd-workflow, autonomous-loop
- api-design, system-design, code-review, security-review
- rag-pipeline, agent-builder, mcp-builder, db-migrations
- cloud-deploy, docker-compose, ci-cd-pipeline, refactor-clean, memory-context

**MCP Servers**: github, memory, sequential-thinking, context7, filesystem, exa-web-search
</context>

<process>

<step name="activate_orchestrator" priority="first">
You ARE the orchestrator. Read agents/tier1/orchestrator.md for your full behavior spec.

Your first action: run the research-first skill on the target codebase.
Then decompose the user's request and build your execution plan.
</step>

<step name="decompose">
Break the request into domains:
1. What areas does this touch? (frontend/backend/DB/infra/security/AI)
2. What are the dependencies? (what must happen before what?)
3. What can be parallelized?
4. What are the security, data, or breaking-change risks?
</step>

<step name="spawn_planner">
Spawn the planner agent with:
- Full codebase context from research step
- User intent restated precisely
- Constraints and success criteria

Wait for PLAN.md. Validate it meets the quality bar before proceeding.
</step>

<step name="parallel_execution">
Spawn specialists in waves per the dependency graph.

Wave 1 (independent work — run in parallel):
- architect → system design, data models, API contracts (if needed)
- db-architect → schema, migrations (if DB changes needed)
- security-guardian → threat model, auth design (if security-sensitive)

Wave 2 (after Wave 1 outputs):
- backend-expert → API implementation
- frontend-expert → UI implementation
- builder → file creation per PLAN.md

Wave 3 (after core implementation):
- qa-champion → test suite
- doc-engineer → documentation

Wave 4 (before delivery):
- security-guardian → final review
- git-master → commit + PR description
</step>

<step name="quality_gate">
Before delivery, verify:
- [ ] All PLAN.md success criteria met
- [ ] Test suite passes (run it)
- [ ] No TypeScript errors (npm run typecheck)
- [ ] No lint errors
- [ ] Security guardian review clean
- [ ] No secrets in code
- [ ] Documentation updated
</step>

<step name="deliver">
Output structured delivery summary:

## Swarm Delivery Complete

### What Was Built
[Concise description of what was created/modified]

### Files
[List every created or modified file with what it does]

### How to Use / Run
[Exact commands to run what was built]

### Tests
[How to run tests, current coverage]

### Next Steps
[Optional: suggested follow-up tasks]
</step>

</process>

<success_criteria>
- [ ] User's request is fully satisfied
- [ ] All code is production-ready (not prototype)
- [ ] Tests exist and pass
- [ ] Security reviewed
- [ ] Documentation updated
- [ ] No hardcoded secrets
- [ ] Conventional commits applied
</success_criteria>
