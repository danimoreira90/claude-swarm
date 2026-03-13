---
name: twelve-factor
description: >
  12-Factor App compliance skill. Audits any service against all 12 factors.
  Each factor: what it means, violations (with examples), fix pattern, verification command.
  Run before every new service design and deployment. Non-negotiable for production.
---

# 12-Factor App Compliance Skill

> Build software-as-a-service that is portable, reliable, and scalable. No exceptions.

The [12-Factor App methodology](https://12factor.net) applies to any service the swarm builds. Non-compliance is a production risk, not a style preference.

---

## Factor I — Codebase

**One codebase tracked in revision control, many deploys.**

**Means:** Single git repo. The same code runs in dev, staging, and production with different configs.

**Violations:**
```
❌ Separate repos per environment (repo-dev, repo-prod)
❌ Config files committed per environment (config.prod.ts, config.dev.ts)
❌ Hotfixes applied directly to production repo, not merged back
```

**Fix:**
```
✅ Single repo
✅ deploy = git tag or SHA → env-specific config comes from environment
✅ Hotfix branch → merge to main → deploy the same artifact
```

**Verification:**
```bash
# Check for per-env config files
find . -name "*.prod.*" -o -name "*.staging.*" -o -name "config.production.*" \
  | grep -v node_modules | grep -v dist
# Should return nothing
```

---

## Factor II — Dependencies

**Explicitly declare and isolate all dependencies.**

**Means:** All dependencies listed in manifest. No implicit reliance on system packages.

**Violations:**
```
❌ pip install something without adding to pyproject.toml
❌ No lockfile (package-lock.json, uv.lock, Pipfile.lock)
❌ require('some-module') not in package.json
❌ Assumes curl, ffmpeg, or imagemagick is installed on the host
```

**Fix:**
```
✅ package.json + package-lock.json (npm ci in Dockerfile)
✅ pyproject.toml + uv.lock (uv pip install --frozen)
✅ System-level deps: declare in Dockerfile, not assumed on host
✅ Dockerfile FROM pins the base image version
```

**Verification:**
```bash
# Check for undeclared Node deps
node -e "require('some-package')" 2>&1 | grep "Cannot find module"
# Or just: npm ci (fails on undeclared)

# Check lockfile is committed
git ls-files package-lock.json uv.lock | grep -c .
# Should return > 0
```

---

## Factor III — Config ⚠️ MOST COMMONLY VIOLATED

**Store config in the environment.**

**Means:** Everything that varies between deploys (dev/staging/prod) lives in environment variables. NEVER in code or committed config files.

**What counts as config:** Database URLs, API keys, service endpoints, feature flags, ports.

**Violations:**
```
❌ Hardcoded database URLs
❌ if (process.env.NODE_ENV === 'production') { use different URL }
❌ config.dev.json, config.prod.json in the repo
❌ API keys in code files
```

**Bad examples:**
```typescript
// ❌ NEVER
const db = new Client({ host: 'localhost', port: 5432 });
const apiKey = 'sk-abc123...';
```

```python
# ❌ NEVER
DATABASE_URL = "postgresql://user:pass@localhost:5432/myapp"
```

**Fix:**
```typescript
// ✅ Correct
const db = new Client({ connectionString: process.env.DATABASE_URL });
const apiKey = process.env.OPENAI_API_KEY;
if (!apiKey) throw new Error('OPENAI_API_KEY env var is required');
```

```python
# ✅ Correct — using pydantic-settings
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    DATABASE_URL: str
    OPENAI_API_KEY: str
    REDIS_URL: str = "redis://localhost:6379"

settings = Settings()  # raises if required vars missing
```

**Local development:** `.env` file (gitignored) + `.env.example` (committed template)

**Production:** Platform env vars (Railway, Vercel), K8s Secrets, AWS SSM/Secrets Manager

**Verification:**
```bash
# Check for hardcoded localhost config
grep -r "localhost:5432\|localhost:6379\|localhost:9092" src/ \
  --include="*.ts" --include="*.py" --include="*.js"
# Should return nothing

# Check for hardcoded passwords/keys
grep -rE "password\s*=\s*['\"][^'\"]+['\"]" src/ \
  --include="*.ts" --include="*.py"
# Should return nothing

# Verify .env is gitignored
git check-ignore .env && echo "gitignored ✅" || echo "NOT gitignored ❌"
```

---

## Factor IV — Backing Services

**Treat backing services as attached resources.**

**Means:** Database, Redis, S3, email service, queue — all accessed via URL/credentials from config. Swap postgres on localhost for RDS by changing DATABASE_URL. No code changes.

**Violations:**
```
❌ Different code paths for "local" vs "remote" database
❌ if (env === 'prod') { connect to RDS } else { connect to local postgres }
❌ S3 in prod, local filesystem in dev (different behavior, not just different config)
```

**Fix:**
```
✅ DATABASE_URL env var — same code regardless of postgres location
✅ REDIS_URL env var — same code for local Redis or ElastiCache
✅ S3 in dev too (or MinIO as local S3 — same SDK, different endpoint)
```

**docker-compose for Factor IV compliance:**
```yaml
services:
  api:
    environment:
      DATABASE_URL: postgresql://app:app@postgres:5432/app
      REDIS_URL: redis://redis:6379
      # S3 endpoint for local dev:
      S3_ENDPOINT: http://minio:9000
  postgres:
    image: postgres:16-alpine
  redis:
    image: redis:7-alpine
  minio:
    image: minio/minio  # local S3 — same SDK as production AWS S3
```

---

## Factor V — Build, Release, Run

**Strictly separate build and run stages.**

**Means:** Build: compile code → artifact. Release: artifact + config = deployable. Run: execute release in environment. Never rebuild in production.

**Violations:**
```
❌ npm install at runtime (mixing build and run)
❌ git pull in production to "deploy" (no versioned artifacts)
❌ Building Docker image on the production server
❌ Compiling TypeScript at container startup
```

**Fix (Docker multi-stage):**
```dockerfile
# Stage 1: Build — compile, install all deps
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Release — production deps only + built artifact
FROM node:20-alpine AS release
WORKDIR /app
COPY --from=build /app/dist ./dist
COPY --from=build /app/package*.json ./
RUN npm ci --only=production

# Stage 3: Run — immutable artifact, config injected at runtime
FROM node:20-alpine AS production
WORKDIR /app
COPY --from=release /app .
ENV NODE_ENV=production
CMD ["node", "dist/main.js"]
# Config (DATABASE_URL, etc.) injected via docker run -e or K8s secrets
```

---

## Factor VI — Processes

**Execute the app as stateless, share-nothing processes.**

**Means:** Process state is ephemeral. Any state that must survive a process restart must be in a backing service (Redis, PostgreSQL, S3).

**Violations:**
```
❌ In-memory session store (sessions lost on restart)
❌ File uploads to local disk (lost when container restarts)
❌ In-process job queue (jobs lost on crash)
❌ Sticky sessions (users must hit same server)
```

**Fix:**
```
✅ Sessions → Redis (express-session with Redis store / FastAPI sessions in Redis)
✅ Uploads → S3 / GCS / R2 (presigned URL pattern)
✅ Jobs → BullMQ (Redis-backed) or Kafka
✅ No sticky sessions needed — any instance can serve any user
```

**Verification:**
```bash
# Check for in-memory session stores
grep -r "MemoryStore\|new session.MemoryStore\|sessions = {}" src/ --include="*.ts"
# Should return nothing

# Check for local file writes (that aren't temp files)
grep -r "fs\.writeFile\|fs\.writeFileSync\|open(.*'w')" src/ --include="*.ts" --include="*.py"
# Acceptable: /tmp/, ephemeral temp files
# Not acceptable: ./uploads/, ./data/, any persistent directory
```

---

## Factor VII — Port Binding

**Export services via port binding.**

**Means:** The app embeds its own HTTP server. No reliance on an external runtime (Tomcat, Apache, IIS). App starts, binds to $PORT, becomes HTTP server.

**Violations:**
```
❌ Deploying as WAR to Tomcat
❌ Apache/nginx serves the app directly (not as reverse proxy)
❌ App hardcodes port 3000 instead of reading from $PORT
```

**Fix:**
```typescript
// ✅ NestJS — reads port from env
const port = process.env.PORT ?? 3000;
await app.listen(port, '0.0.0.0');
```

```python
# ✅ FastAPI / uvicorn
import uvicorn
if __name__ == "__main__":
    uvicorn.run("main:app", host="0.0.0.0", port=int(os.environ.get("PORT", 8000)))
```

---

## Factor VIII — Concurrency

**Scale out via the process model.**

**Means:** Scale horizontally by running more processes. Don't try to solve scaling with threading within a single large process.

**Fix:**
```yaml
# K8s HPA — scale horizontally based on CPU/memory
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
spec:
  minReplicas: 2
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

---

## Factor IX — Disposability

**Maximize robustness with fast startup and graceful shutdown.**

**Means:** Processes start fast, shut down gracefully on SIGTERM. Crash-only design — if something goes wrong, crash and let the orchestrator restart. Don't try to recover in-process.

**Violations:**
```
❌ Startup takes > 30 seconds (waiting for data migration, heavy initialization)
❌ Ignoring SIGTERM — killed with SIGKILL, leaving in-flight requests incomplete
❌ In-flight HTTP requests aborted on shutdown
```

**Fix — Node.js SIGTERM handler:**
```typescript
// Graceful shutdown
const server = app.getHttpServer();

process.on('SIGTERM', async () => {
  console.log('SIGTERM received. Shutting down gracefully...');

  // Stop accepting new connections
  server.close(async () => {
    // Drain: wait for in-flight requests to complete
    await app.close();  // NestJS lifecycle hooks run

    // Close DB connections
    await prisma.$disconnect();

    console.log('Graceful shutdown complete.');
    process.exit(0);
  });

  // Force exit if graceful shutdown takes too long
  setTimeout(() => process.exit(1), 30000);
});
```

**Fix — Python SIGTERM handler:**
```python
import signal
import asyncio

async def shutdown(loop, signal=None):
    """Graceful shutdown coroutine."""
    if signal:
        print(f"Received {signal.name}, shutting down...")
    tasks = [t for t in asyncio.all_tasks() if t is not asyncio.current_task()]
    [task.cancel() for task in tasks]
    await asyncio.gather(*tasks, return_exceptions=True)
    loop.stop()

loop = asyncio.get_event_loop()
for sig in (signal.SIGTERM, signal.SIGINT):
    loop.add_signal_handler(sig, lambda: asyncio.create_task(shutdown(loop, sig)))
```

---

## Factor X — Dev/Prod Parity

**Keep development, staging, and production as similar as possible.**

**Violations:**
```
❌ SQLite in dev, PostgreSQL in production
❌ Node 18 in dev, Node 20 in production
❌ No Redis in development (cache not tested)
❌ Different OS: macOS dev, Linux prod (line endings, path separators)
```

**Fix — docker-compose for prod parity:**
```yaml
# Same services as production, same versions
services:
  postgres:
    image: postgres:16-alpine  # same version as prod
  redis:
    image: redis:7-alpine       # same version as prod
  kafka:
    image: confluentinc/cp-kafka:7.6.0  # same as prod
```

---

## Factor XI — Logs

**Treat logs as event streams.**

**Means:** App writes to stdout. Infrastructure captures and routes logs. App never opens log files, never rotates logs, never manages log aggregation.

**Violations:**
```
❌ logging.FileHandler in Python
❌ new transports.File() in Winston
❌ RotatingFileHandler, TimedRotatingFileHandler
❌ Structured logging written to files app manages
```

**Fix — Structured JSON to stdout:**
```typescript
// NestJS — pino (structured JSON to stdout)
import pino from 'pino';
const logger = pino({ level: 'info' });

// Output: {"level":30,"time":1234567890,"msg":"User created","userId":"abc"}
logger.info({ userId: user.id }, 'User created');
```

```python
# Python — structlog (structured JSON to stdout)
import structlog
logger = structlog.get_logger()
logger.info("user.created", user_id=str(user.id), email=user.email)
# Output: {"event": "user.created", "user_id": "...", "timestamp": "..."}
```

**Verification:**
```bash
grep -r "FileHandler\|RotatingFile\|createWriteStream.*\.log\|appendFile.*\.log" \
  src/ --include="*.ts" --include="*.py"
# Should return nothing
```

---

## Factor XII — Admin Processes

**Run admin/management tasks as one-off processes in the same codebase.**

**Violations:**
```
❌ Separate admin codebase for migrations
❌ SSH into production server to run scripts
❌ Admin scripts that import from a different version of the code
```

**Fix:**
```bash
# Kubernetes one-off job
kubectl exec -n production deploy/api -- npx prisma migrate deploy

# Docker one-off
docker run --env-file .env.prod myapp:v1.4.0 node scripts/migrate.js

# Railway
railway run npx prisma migrate deploy
```

---

## 12-Factor Audit Checklist

**Run this before every deployment:**

```
[ ] I.   Single codebase? (not per-env forks)
[ ] II.  All dependencies declared and locked? (lockfile committed)
[ ] III. Zero hardcoded config? (all in env vars, .env gitignored)
[ ] IV.  All backing services as attached resources? (URL from env)
[ ] V.   Build/release/run stages strictly separated? (Docker multi-stage)
[ ] VI.  No local state? (stateless processes, Redis/S3 for persistence)
[ ] VII. Port binding self-contained? (reads $PORT, embeds HTTP server)
[ ] VIII.Scales horizontally? (no in-process state preventing replication)
[ ] IX.  Fast startup (<30s) and graceful SIGTERM shutdown?
[ ] X.   Dev/prod parity? (same DB type + version, same services)
[ ] XI.  Logs to stdout only? (no log files, JSON structured)
[ ] XII. Admin tasks as one-off processes? (same codebase, not SSH)
```

---

## Worked Audit: NestJS + PostgreSQL + Redis

**Service under audit:** User management API (NestJS + Prisma + Redis sessions)

**Findings:**

| Factor | Status | Issue | Fix |
|--------|--------|-------|-----|
| I | ✅ | Single repo | — |
| II | ✅ | package-lock.json committed | — |
| III | ❌ | `host: 'localhost'` hardcoded in database.config.ts | Move to DATABASE_URL env var |
| IV | ✅ | Redis URL from env | — |
| V | ✅ | Docker multi-stage build | — |
| VI | ❌ | Winston FileTransport writing to ./logs/ | Switch to stdout transport |
| VII | ✅ | Reads PORT from env | — |
| VIII | ✅ | Stateless, K8s HPA configured | — |
| IX | ✅ | SIGTERM handler implemented | — |
| X | ⚠️ | Redis missing from docker-compose (dev) | Add Redis service |
| XI | ❌ | Same as VI — log files | Switch to pino/stdout |
| XII | ✅ | Migrations via `kubectl exec` | — |

**Blockers (must fix before production):** Factors III, VI, XI
**Warning (fix in next sprint):** Factor X
