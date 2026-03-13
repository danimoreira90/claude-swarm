---
name: architect
description: System design session. ADRs, component diagrams, API contracts, tech decisions.
allowed-tools: ["Read", "Grep", "Glob", "Write", "WebFetch"]
argument-hint: "<describe the system or decision to design>"
---

<objective>
Run an architecture design session for: $ARGUMENTS

Produce: architecture diagram, ADR, data model, API contract, implementation notes.
</objective>

<execution_context>
@agents/tier2/architect.md
@skills/research-first.md
</execution_context>

<context>
**Design Task**: $ARGUMENTS
</context>

<process>
1. Research existing architecture (read CLAUDE.md, existing ADRs, system files)
2. Analyze requirements and constraints
3. Design the system/component
4. Write ADR with trade-offs
5. Produce implementation notes for builder agents
</process>
