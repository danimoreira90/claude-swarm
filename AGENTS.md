# Agent Registry — Claude Swarm

> Auto-discovery index. All agents in this swarm. Updated manually when agents are added/modified.

## Tier 1 — Orchestrator (The Brain)

| Agent | Model | File | Activate When |
|-------|-------|------|---------------|
| `orchestrator` | opus | `agents/tier1/orchestrator.md` | Complex/multi-step tasks, full app builds, any `/swarm` command |

## Tier 2 — Domain Specialists (The Experts)

| Agent | Model | File | Activate When |
|-------|-------|------|---------------|
| `planner` | opus | `agents/tier2/planner.md` | Any feature needing a PLAN.md, `/plan` command |
| `architect` | opus | `agents/tier2/architect.md` | System design, ADRs, tech selection, `/architect` command |
| `builder` | sonnet | `agents/tier2/builder.md` | Raw implementation, executing PLAN.md, `/build` command |
| `git-master` | sonnet | `agents/tier2/git-master.md` | Any git operation, `/git` command |
| `cloud-engineer` | sonnet | `agents/tier2/cloud-engineer.md` | AWS/GCP/K8s/Terraform, `/deploy` command |
| `security-guardian` | opus | `agents/tier2/security-guardian.md` | Auth/security/data code (mandatory), `/security` command |
| `qa-champion` | sonnet | `agents/tier2/qa-champion.md` | Testing, coverage, CI eval, `/test` command |
| `devops-engineer` | sonnet | `agents/tier2/devops-engineer.md` | CI/CD, Docker, monitoring, GitHub Actions |
| `ai-ml-engineer` | opus | `agents/tier2/ai-ml-engineer.md` | LLM, RAG, vector DBs, evals, `/rag` command |
| `db-architect` | opus | `agents/tier2/db-architect.md` | Schema design, migrations (mandatory), `/migrate` command |
| `frontend-expert` | sonnet | `agents/tier2/frontend-expert.md` | Next.js, React, React Native, Tailwind |
| `backend-expert` | sonnet | `agents/tier2/backend-expert.md` | NestJS, FastAPI, APIs, Kafka, gRPC |
| `doc-engineer` | sonnet | `agents/tier2/doc-engineer.md` | README, API docs, ADRs, runbooks, `/document` command |

## Tier 3 — Micro-Executors (The Hands)

> Tier 3 agents are language/tool-specific executors spawned by tier-2 agents.
> Add them as needed for your specific projects.

| Agent | Purpose |
|-------|---------|
| `lang-python` | Python-specific patterns, async, type system |
| `lang-typescript` | TypeScript advanced patterns, generics, decorators |
| `lang-go` | Go conventions, goroutines, interfaces |
| `tool-docker` | Dockerfile optimization, multi-stage builds |
| `tool-k8s` | Kubernetes manifests, Helm, operators |
| `tool-terraform` | IaC patterns, state management, modules |

---

## Skills Registry

| Skill | File | Use When |
|-------|------|---------|
| `research-first` | `skills/research-first.md` | **Always** — before any implementation |
| `spec-driven-dev` | `skills/spec-driven-dev.md` | New features, unclear requirements |
| `autonomous-loop` | `skills/autonomous-loop.md` | Long tasks, `/loop` command |

---

## Commands Registry

| Command | File | Action |
|---------|------|--------|
| `/swarm` | `commands/swarm.md` | Full swarm activation |
| `/plan` | `commands/plan.md` | Create PLAN.md |
| `/build` | `commands/build.md` | Execute implementation |
| `/review` | `commands/review.md` | Code review |
| `/test` | `commands/test.md` | Write & run tests |
| `/git` | `commands/git.md` | Git workflow |
| `/security` | `commands/security.md` | Security audit |
| `/deploy` | `commands/deploy.md` | Cloud deployment |
| `/migrate` | `commands/migrate.md` | DB migrations |
| `/document` | `commands/document.md` | Documentation |

---

## Routing Quick Reference

```
Need to build something complex?     → /swarm <description>
Need a plan?                         → /plan <description>
Have a PLAN.md ready?                → /build
Need code reviewed?                  → /review
Need tests?                          → /test <file or module>
Need to commit/PR?                   → /git commit|pr|branch
Security concern?                    → /security
Deploying?                           → /deploy <platform>
Schema change?                       → /migrate <description>
Docs out of date?                    → /document
```

---

*Swarm v1.0 | 2026-03-11 | Built by Daniel Moreira*
