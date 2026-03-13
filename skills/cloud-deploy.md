---
name: cloud-deploy
description: >
  Deploy to Vercel/Railway/K8s with env management and rollback capability.
  Zero-downtime deployments with health check verification.
  Use via /deploy command.
---

# Cloud Deploy Skill

> Deploy confidently. Verify before and after. Always have a rollback plan.

## Pre-Deploy Checklist

```
[ ] Tests pass (npm test / pytest)
[ ] No TypeScript errors
[ ] No hardcoded secrets (git grep for keys)
[ ] Environment variables documented in .env.example
[ ] Docker image builds cleanly
[ ] DB migrations are backward-compatible
[ ] Health check endpoint exists: GET /health → { "status": "ok" }
[ ] Rollback plan documented
```

## Platform Quick Reference

### Vercel (Next.js)
```bash
# Deploy (auto on push to main if connected)
vercel --prod

# Preview deploy
vercel

# Environment variables
vercel env add NEXT_PUBLIC_API_URL production
vercel env ls

# Rollback
vercel rollback  # to previous deployment
```

### Railway
```bash
# Install CLI
npm install -g @railway/cli

# Deploy
railway up

# Set env vars
railway variables set DATABASE_URL=postgresql://...

# View logs
railway logs

# Rollback: redeploy previous deployment from dashboard
```

### Kubernetes
```bash
# Build + push image
docker build -t ghcr.io/org/api:${GIT_SHA} .
docker push ghcr.io/org/api:${GIT_SHA}

# Apply DB migrations first (backward-compatible)
kubectl exec -n production deploy/api -- npx prisma migrate deploy

# Rolling deploy
kubectl set image deployment/api api=ghcr.io/org/api:${GIT_SHA} -n production
kubectl rollout status deployment/api -n production --timeout=5m

# Verify health
kubectl get pods -n production -l app=api
curl https://api.example.com/health

# Rollback (if something goes wrong)
kubectl rollout undo deployment/api -n production
```

## Post-Deploy Verification

```bash
# 1. Health check
curl -f https://your-app.com/health || echo "HEALTH CHECK FAILED"

# 2. Smoke test key endpoints
curl -f https://your-app.com/api/v1/status

# 3. Monitor error rate (5 minutes)
# In Grafana/Datadog: error rate should be < 0.1%

# 4. Monitor latency
# p99 should not degrade from baseline

# 5. Tag the release
git tag -a v$(cat package.json | jq -r .version) -m "Release $(date)"
git push origin --tags
```

## Environment Variable Management

```
Development:  .env (local, gitignored)
Staging:      Platform env vars (Railway/Vercel dashboard) or K8s secrets
Production:   AWS SSM / GCP Secret Manager / K8s External Secrets
```

**Never** deploy with production secrets in the image or repository.
