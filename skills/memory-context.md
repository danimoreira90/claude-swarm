---
name: memory-context
description: >
  Session memory management. Capture, compress, and inject context across sessions.
  Integrates with MCP memory server for persistent knowledge graph.
  Run /checkpoint to save current session, loads automatically on session start.
---

# Memory & Context Skill

> What you know at the end of a session should be available at the start of the next.

## Session Memory Architecture

```
Session End:
  → capture key decisions, file paths, patterns discovered
  → compress into structured markdown
  → write to ~/.claude/memory/
  → update MCP memory server knowledge graph

Session Start:
  → read ~/.claude/memory/last-session.md
  → inject context into first response
  → resume work with full context
```

## Checkpoint Command

Run `/checkpoint` at logical stopping points:

```markdown
## Checkpoint: [date] — [project] — [what was accomplished]

### What Was Built
[Files created/modified with what they do]

### Key Decisions Made
[Architectural, security, or design decisions and WHY]

### Patterns Discovered
[Existing patterns found in the codebase to match]

### Current State
[What's done, what's in progress, what's next]

### Open Items
[Things to pick up next session]

### File Map
[Key files and their responsibilities]
```

## MCP Memory Server Integration

The `memory` MCP server maintains a persistent knowledge graph:

```
# Store a key fact
mcp_memory_create_entities:
  - type: "project_pattern"
    name: "auth-guard-pattern"
    observations:
      - "Uses @UseGuards(JwtAuthGuard) decorator on controllers"
      - "Custom decorator @CurrentUser() extracts user from JWT payload"
      - "Located in src/common/guards/ and src/common/decorators/"

# Store an architectural decision
mcp_memory_create_relations:
  - from: "auth-service"
    to: "jwt-strategy"
    relationType: "uses"

# Query knowledge graph at session start
mcp_memory_search_nodes: "auth"
```

## Context Injection Protocol

At the start of each session:
1. Read `~/.claude/memory/last-session.md`
2. Query MCP memory server for relevant project entities
3. Read project CLAUDE.md
4. Read `.claude/loop-state.md` if loop was active

Format context brief:
```markdown
## Session Context

**Project**: [name]
**Last Session**: [date]
**Resumed From**: [what was last accomplished]

**Active Work**:
- [In-progress items]

**Key Patterns** (from memory):
- [Pattern 1]
- [Pattern 2]

**Open Items**:
- [What needs to happen next]
```

## Memory File Structure

```
~/.claude/memory/
├── last-session.md        ← Most recent session summary
├── session-costs.jsonl    ← Usage tracking (appended per session)
├── loop-state-backup.md   ← Loop state backup before compaction
└── projects/
    ├── my-app.md          ← Project-specific memory
    └── another-project.md
```

## Anti-Patterns

- Don't store implementation details — store decisions and patterns
- Don't store things that are in the code — reference the file path instead
- Don't duplicate what's in CLAUDE.md — link to it
- Don't store sensitive data (passwords, tokens) in memory files
