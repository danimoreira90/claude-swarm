---
name: document
description: Generate/update documentation via doc-engineer — README, API docs, ADRs, runbooks.
allowed-tools: ["Read", "Write", "Glob", "Grep"]
argument-hint: "[type: readme | api | adr | runbook | changelog | all]"
---

<objective>
Create or update documentation: $ARGUMENTS

Keep docs in sync with code. Accurate, concise, useful.
</objective>

<execution_context>
@agents/tier2/doc-engineer.md
</execution_context>

<context>
**Documentation Target**: $ARGUMENTS

```bash
ls docs/ README.md CHANGELOG.md 2>/dev/null
cat package.json | jq '{name, description, scripts}' 2>/dev/null
```
</context>

<process>
1. Read current code to understand what needs documenting
2. Read existing docs to understand current state
3. Identify gaps between code and docs
4. Create/update documentation
5. Verify docs match current implementation exactly
</process>
