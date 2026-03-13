---
name: agent-builder
description: >
  Build new Claude Code agents from scratch with proper YAML frontmatter, behavior spec,
  and tool assignments. Use when extending the swarm with new specialist agents.
---

# Agent Builder Skill

> Growing the swarm: how to create a new high-quality agent.

## Agent File Format

```markdown
---
name: agent-name                    ← kebab-case, unique in swarm
description: >                      ← CRITICAL: this is what the orchestrator reads to route
  One-line summary. Then specify EXACTLY when to activate this agent.
  Include: domain expertise, what triggers it, slash command if any.
tools: ["Read", "Write", "Bash", "Glob", "Grep"]  ← only what's needed
model: sonnet                       ← sonnet for execution, opus for deep reasoning
---

[Full agent behavior specification in markdown]
```

## Description Field — The Most Important Field

The `description` is what the orchestrator reads to decide whether to spawn this agent.
It must answer:
1. What domain does this agent own?
2. When should it be activated?
3. What does it NOT handle (scope limits)?

```markdown
# Good description:
description: >
  Database specialist. PostgreSQL schema design, migrations, query optimization.
  NON-NEGOTIABLE: activate before any schema change or migration.
  Does NOT handle: application-layer DB access patterns (that's backend-expert).

# Bad description:
description: Database agent that helps with databases.
```

## Tool Assignment

Only assign tools the agent genuinely needs:

| Tool | When to Include |
|------|----------------|
| `Read` | Agent reads existing files for context |
| `Write` | Agent creates or modifies files |
| `Bash` | Agent runs commands (tests, builds, git ops) |
| `Glob` | Agent discovers file patterns |
| `Grep` | Agent searches for patterns in code |
| `Task` | Agent spawns sub-agents |
| `WebFetch` | Agent needs live documentation or web research |

Research-only agents: `["Read", "Grep", "Glob"]`
Implementation agents: `["Read", "Write", "Bash", "Glob", "Grep"]`
Orchestrator-tier: `["Read", "Write", "Bash", "Glob", "Grep", "Task", "WebFetch"]`

## Agent Behavior Structure

Every agent file should have:

```markdown
## Your Role
[What you are, what you own, what you don't own]

## [Domain Knowledge / Patterns Reference]
[The domain expertise — code examples, patterns, checklists]

## Workflow
[Step-by-step how you execute your tasks]

## Output Format
[What you produce — file formats, report structures, etc.]

## Anti-Patterns / Rules
[What you never do]
```

## New Agent Checklist

Before writing your agent:
- [ ] Define the domain clearly (what does it own exclusively?)
- [ ] Define activation criteria (when is it called vs other agents?)
- [ ] Define output (what does it produce?)
- [ ] Choose model (opus for reasoning, sonnet for execution)
- [ ] Assign minimal tools
- [ ] Write example invocations

After writing:
- [ ] Add to `AGENTS.md` registry
- [ ] Add command in `commands/` if user-invocable
- [ ] Test: give it a realistic task, does it work well?
- [ ] Add to orchestrator routing table if needed

## Example: Building a New Agent

```bash
# User asks: "I need an agent that handles Stripe integration"

# 1. Identify the domain
# Domain: Stripe API — webhooks, subscriptions, payment intents, products

# 2. Write agents/tier2/stripe-expert.md
cat > agents/tier2/stripe-expert.md << 'EOF'
---
name: stripe-expert
description: >
  Stripe integration specialist. Payment intents, subscriptions, webhooks, products/prices.
  Activate for any Stripe API integration, billing flows, or subscription management.
  Works alongside security-guardian (payment code is always security-reviewed).
tools: ["Read", "Write", "Bash", "Glob", "Grep", "WebFetch"]
model: sonnet
---
...
EOF

# 3. Add to AGENTS.md
# 4. Add /stripe command if needed
# 5. Add routing rule to orchestrator
```
