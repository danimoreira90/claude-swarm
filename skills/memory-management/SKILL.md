# Memory Management Skill
**System:** claude-mem v9.1.1 | **Storage:** `~/.claude-mem/claude-mem.db` (SQLite) | **Worker:** `localhost:37777`

> claude-mem runs automatically. This skill teaches you how to read what it injects, how to classify work so observations are useful, and when to protect sensitive content.

---

## How Memory Works (Read This First)

claude-mem hooks into every Claude Code session:

```
SessionStart   → injects your recent activity as additionalContext (read it now)
UserPromptSubmit → opens a session record, starts background SDK agent
PostToolUse    → background agent synthesizes each tool call into a typed observation
Stop           → generates a structured session summary, persists everything to SQLite
```

**You do not write to memory manually.** The SDK agent observes your tool usage and writes observations. Your job is to:
1. Read the injected context at session start
2. Classify your work correctly so the SDK agent picks the right observation type
3. Tag private content so it never reaches the database
4. Interpret folder-level CLAUDE.md context blocks when you read source files

---

## Reading the Injected Context (SessionStart)

At every session start, claude-mem injects a `# Recent Activity` block into your context. It looks like this:

```markdown
# Recent Activity

### Jan 10, 2026

| ID    | Time     | T  | Title                              | Read  |
|-------|----------|----|------------------------------------|-------|
| #39050 | 3:44 PM | ✅ | Fixed JWT token expiry bug         | ~255  |
| #38802 | 5:11 PM | 🔵 | Claude-Mem Hook Configuration      | ~450  |

### Jan 9, 2026

| ID    | Time     | T  | Title                              | Read  |
|-------|----------|----|------------------------------------|-------|
| #38175 | 7:26 PM | 🔵 | Hook Output Architecture Documented | ~530 |
```

**Column meanings:**
| Column | Meaning |
|--------|---------|
| ID | Observation ID (`#NNNN`) or session ID (`#SNNNN`) |
| Time | When the observation was recorded |
| T | Type emoji (see observation types below) |
| Title | One-line summary of what was discovered/done |
| Read | Approximate token cost of the full observation (`~NNN`) |

**What to do with it:**
- Scan the titles to orient yourself before starting new work
- If the user references something from a previous session, use the observation ID to search: `mem search #39050`
- `next_steps` from the most recent summary tells you what was left unfinished
- If you see a `⚖️` (decision), that decision is still in effect unless explicitly reversed

---

## Observation Types

The SDK agent classifies every observation into one of six types. Understanding these helps you frame your work clearly so the agent classifies correctly.

| Type | Emoji | When it applies | Example |
|------|-------|----------------|---------|
| `decision` | ⚖️ | An architectural or design choice was made | "Chose SQLite over PostgreSQL for simplicity" |
| `bugfix` | 🔴 | A defect was identified and fixed | "Fixed race condition in session cleanup" |
| `feature` | 🟣 | New functionality was added | "Implemented JWT refresh token rotation" |
| `refactor` | 🔄 | Code restructured without behavior change | "Extracted session logic into SessionManager" |
| `discovery` | 🔵 | Something was learned about the codebase | "Found that pool_size is invalid for SQLite" |
| `change` | 🟡 | Configuration, dependency, or infra change | "Added aiosqlite to production dependencies" |

**Tip:** Frame summaries using these categories. Instead of "worked on auth", say "fixed the JWT expiry calculation" (bugfix) or "decided to use bcrypt instead of argon2" (decision).

---

## Session Summary Structure

At `Stop`, the SDK agent generates a structured summary stored as:

```
request:      What the user originally asked for
investigated: What was explored/read to understand the problem
learned:      Key discoveries made during the session
completed:    What was actually shipped/committed
next_steps:   What remains to be done (carries forward to next session)
notes:        Anything else worth remembering
```

**At session start, the most recent summary's `next_steps` is your backlog.** If the user doesn't give you a new task, check `next_steps` first.

---

## Folder-Level CLAUDE.md Context Blocks

After every tool call that touches a file, claude-mem writes a `CLAUDE.md` in that file's directory with a `<claude-mem-context>` block. When you read any source file, Claude Code automatically loads the CLAUDE.md in that directory.

**What you'll see when reading files in a tracked project:**

```markdown
<claude-mem-context>
# Recent Activity

### Jan 10, 2026

| ID | Time | T | Title | Read |
|----|------|---|-------|------|
| #39050 | 3:44 PM | ✅ | Fixed JWT token expiry | ~255 |
| #38802 | " | 🔵 | Reviewed auth flow | ~450 |
</claude-mem-context>
```

**How to interpret it:**
- This block shows what was done **in this specific directory** recently
- `"` in Time column = ditto mark (same time as row above)
- The block is automatically maintained — never edit it manually
- If you write to a file in this directory, the block will update after your tool call

**What you must NOT do:**
- Do not copy, move, or manually edit content inside `<claude-mem-context>...</claude-mem-context>`
- Do not write to CLAUDE.md files in source directories manually — claude-mem owns those tags
- User content outside the tags is preserved; content inside the tags is overwritten each update

---

## Tagging Private Content

Wrap any content you want excluded from memory with `<private>` tags:

```
<private>
API key: sk-proj-abc123...
Customer PII: John Doe, john@example.com
</private>
```

**Tag stripping happens at the hook layer** — private content never reaches the worker or database. Use it for:
- API keys, tokens, passwords
- Customer PII or sensitive business data
- Internal URLs or credentials
- Content the user explicitly says should not be stored

```
<private>This whole thought should not be stored</private>
```

You can also tag at the message level — if an entire response contains sensitive data, wrap the sensitive parts.

---

## Searching Past Memory

The mem-search skill (auto-loaded when you ask about history) queries the worker API:

```
GET http://localhost:37777/api/search?q=jwt+expiry&project=fastapi-crud-service
GET http://localhost:37777/api/search/by-file?filePath=/path/to/file&isFolder=true
GET http://localhost:37777/api/context/inject?projects=my-project
```

**Natural language queries that trigger memory search:**
- "What did we do with auth last time?"
- "Have I fixed this before?"
- "What was the decision on the database schema?"
- "Show me recent activity in app/services/"

---

## What NOT to Do

| Don't | Do instead |
|-------|-----------|
| Manually write observation JSON to any file | Let the SDK agent synthesize from tool calls |
| Edit `<claude-mem-context>` blocks in CLAUDE.md | Leave them — they auto-update |
| Summarize "tests passed" without showing output | Show real output (anti-cheat rule) |
| Store secrets in tool call inputs | Wrap with `<private>` before the tool call |
| Ignore the injected context at session start | Read `next_steps` from the last summary |
| Assume a decision is still valid without checking | Check the `⚖️ decision` observations for reversals |

---

## Quick Reference

```
Observation written?  → Automatically, after every PostToolUse
Where stored?         → ~/.claude-mem/claude-mem.db
Worker running?       → curl http://localhost:37777/health
View memory UI?       → http://localhost:37777
Search past work?     → /mem-search <query>   OR  ask naturally about history
Protect content?      → Wrap with <private>...</private>
Session summary?      → Generated at Stop, visible next SessionStart
```
