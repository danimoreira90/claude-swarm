---
name: git
description: Activate git-master for any git workflow — commit, branch, PR, tag, changelog.
allowed-tools: ["Bash", "Read", "Write"]
argument-hint: "<git action: commit | pr | branch | tag | changelog | cleanup>"
---

<objective>
Execute git workflow: $ARGUMENTS

Follow conventional commits, clean branching strategy, and quality PR descriptions.
</objective>

<execution_context>
@agents/tier2/git-master.md
@rules/git-conventions.md
</execution_context>

<context>
**Git Action**: $ARGUMENTS

```bash
git status
git log --oneline -10
git diff --stat HEAD
```
</context>

<process>
Execute the requested git action per git-master agent spec.
Always verify: no secrets in staged files, conventional commit format, tests pass.
</process>
