---
name: security
description: Full security audit via security-guardian. OWASP Top 10, secrets, auth, deps.
allowed-tools: ["Read", "Grep", "Glob", "Bash"]
argument-hint: "[scope: full | auth | api | deps | secrets]"
---

<objective>
Run comprehensive security audit: $ARGUMENTS

Cover all OWASP Top 10 vectors. Produce actionable findings with severity and fixes.
</objective>

<execution_context>
@agents/tier2/security-guardian.md
@rules/security.md
</execution_context>

<context>
**Audit Scope**: $ARGUMENTS

```bash
# Dependency audit
npm audit 2>/dev/null || pip-audit 2>/dev/null | head -30

# Quick secrets scan
git log --oneline -5
```
</context>

<process>
1. Secrets scan (hardcoded credentials, API keys)
2. OWASP Top 10 review on target code
3. Dependency CVE scan
4. Auth flow review (if auth code in scope)
5. Input validation check
6. Produce findings: CRITICAL → HIGH → MEDIUM → LOW → INFO
7. Verdict: APPROVED | BLOCKED (with specific fixes required)
</process>
