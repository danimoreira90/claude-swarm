---
name: ci-cd-pipeline
description: >
  GitHub Actions CI/CD pipeline design. Lint → test → build → deploy.
  Covers Node.js, Python, Docker, K8s, Vercel, Railway deployments.
  Use when setting up or improving CI/CD for a project.
---

# CI/CD Pipeline Skill

> Automate the boring stuff. Ship with confidence.

## Pipeline Stages

```
PR Created:
  lint → typecheck → unit tests → integration tests → security scan

PR Merged to main:
  ↑ (above) + build → push image → deploy staging → smoke tests

Release Tag:
  ↑ (above) + deploy production → health check → tag release
```

## Core GitHub Actions Templates

See `agents/tier2/devops-engineer.md` for full CI pipeline YAML.

## Pipeline Best Practices

1. **Fail fast**: Run fast checks (lint, typecheck) before slow ones (integration tests)
2. **Cache aggressively**: Dependencies, Docker layers, build artifacts
3. **Parallelize**: Run independent jobs simultaneously
4. **Secrets in GitHub**: Never in workflow files
5. **Pin action versions**: `actions/checkout@v4` not `@latest`
6. **Timeout every job**: Prevent runaway builds

## Security in CI

```yaml
# Never hardcode secrets
- run: echo "SECRET=${{ secrets.MY_SECRET }}"  # ✓ From GitHub secrets
- run: echo "SECRET=hardcoded-value"            # ✗ Never

# Minimal permissions
permissions:
  contents: read
  packages: write  # only if pushing to GHCR

# Verify dependency integrity
- uses: actions/setup-node@v4
  with:
    cache: 'npm'
- run: npm ci  # uses lockfile exactly
```

## Deployment Strategies

| Strategy | Downtime | Risk | Use When |
|----------|----------|------|----------|
| Rolling update | None | Low | Default for K8s |
| Blue-green | None | Low | Easy rollback needed |
| Canary | None | Very Low | High-risk changes |
| Recreate | Yes | Medium | Dev/staging only |
