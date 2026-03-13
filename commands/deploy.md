---
name: deploy
description: Activate cloud-engineer + devops-engineer for deployment to any platform.
allowed-tools: ["Bash", "Read", "Write", "Glob"]
argument-hint: "<platform: vercel | railway | k8s | docker | aws | gcp>"
---

<objective>
Deploy the application to: $ARGUMENTS

Zero-downtime deployment. Run DB migrations if needed. Verify health checks pass.
</objective>

<execution_context>
@agents/tier2/cloud-engineer.md
@agents/tier2/devops-engineer.md
</execution_context>

<context>
**Deploy Target**: $ARGUMENTS

```bash
git log --oneline -3
cat .env.example | head -20
ls Dockerfile docker-compose.yml k8s/ 2>/dev/null
```
</context>

<process>
1. Check deployment config exists for target platform
2. Run pre-deploy checklist: tests pass, no secrets in code, env vars documented
3. Build Docker image (tag with git SHA)
4. Run DB migrations (backward-compatible only)
5. Deploy with zero-downtime strategy
6. Wait for health checks
7. Monitor error rate for 5 minutes
8. Tag release in git
</process>
