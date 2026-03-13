# Security Rules

> These rules are non-negotiable. Security failures are the most expensive bugs.

## Absolute Rules (Zero Tolerance)

1. **No secrets in code** — no passwords, API keys, tokens, certificates, or connection strings in any committed file
2. **No SQL string concatenation** — parameterized queries always
3. **No authentication bypass** — never disable auth checks, even in tests
4. **No trust of user input** — validate and sanitize all external data
5. **No security review skip** — security-guardian must review all auth/data/API code

## Secrets Management

```bash
# Store in: .env (gitignored), environment variables, Vault, AWS SSM, K8s secrets
# NEVER in: code files, config files, logs, URLs, git history

# .env.example (committed — shows what's needed, not the values)
DATABASE_URL=postgresql://user:password@host:5432/db
JWT_SECRET_KEY=your-256-bit-secret
STRIPE_SECRET_KEY=sk_test_...

# .env (never committed — add to .gitignore)
DATABASE_URL=postgresql://realuser:realpassword@localhost:5432/myapp
```

## Authentication

```typescript
// JWT: short-lived access tokens + longer refresh tokens
// Access token: 15 minutes
// Refresh token: 7-30 days (rotated on use)

// Algorithm: RS256 or ES256 for distributed systems (asymmetric)
// HS256 only for single-service (symmetric — requires shared secret)

// Always validate: exp, iss, aud, algorithm (prevent 'alg:none' attack)
const payload = jwt.verify(token, publicKey, {
  algorithms: ['RS256'],  // explicit — reject other algorithms
  issuer: 'https://auth.example.com',
  audience: 'https://api.example.com',
});
```

## Input Validation

```typescript
// Validate at ALL system boundaries: API routes, webhooks, file uploads, CLI args

// Use Zod (TypeScript) or Pydantic (Python) for structured validation
const CreateUserSchema = z.object({
  email: z.string().email().max(255).toLowerCase(),
  password: z.string().min(8).max(128),
  name: z.string().min(1).max(100).regex(/^[\w\s'-]+$/),
  // Never accept HTML unless you sanitize it (DOMPurify)
});

// Reject unexpected fields (strict mode)
const data = CreateUserSchema.strict().parse(req.body);
```

## SQL / NoSQL Injection Prevention

```typescript
// ✓ Good: parameterized query
const user = await db.query(
  'SELECT * FROM users WHERE email = $1',
  [email]  // parameterized — never interpolated
);

// ✗ Bad: string interpolation = SQL injection vulnerability
const user = await db.query(
  `SELECT * FROM users WHERE email = '${email}'`  // NEVER DO THIS
);

// ✓ Good: ORM (Prisma, TypeORM) — safe by default
const user = await prisma.user.findUnique({ where: { email } });
```

## CORS Configuration

```typescript
// Never use wildcard (*) on auth/sensitive endpoints
app.use(cors({
  origin: process.env.ALLOWED_ORIGINS?.split(',') ?? ['https://app.example.com'],
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
  credentials: true,
  maxAge: 86400,  // 24 hours
}));
```

## Rate Limiting

```typescript
// Required on ALL public endpoints
// Auth endpoints: aggressive limiting (5-10 per 15 minutes)
// API endpoints: 100-1000 per minute depending on resource cost
// File uploads: 10-50 per hour

import rateLimit from 'express-rate-limit';

const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 5,                      // max 5 attempts
  standardHeaders: true,
  message: { error: 'Too many attempts. Try again in 15 minutes.' },
  skipSuccessfulRequests: true,  // only count failures
});
```

## Security Headers

```typescript
import helmet from 'helmet';

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"],  // tighten if possible
      imgSrc: ["'self'", "data:", "https:"],
    },
  },
  hsts: { maxAge: 31536000, includeSubDomains: true, preload: true },
}));
```

## Dependency Security

```bash
# Run before every release
npm audit --audit-level=high
pip-audit

# Block on high/critical CVEs in CI
npm audit --audit-level=high --omit=dev
```

## Audit Logging

```typescript
// Log all security-relevant events
const SECURITY_EVENTS = [
  'login.success', 'login.failure', 'logout',
  'password.changed', 'password.reset',
  'token.refresh', 'token.revoked',
  'permission.denied', 'resource.accessed',
  'admin.action',
];

// Never log: passwords, tokens, full credit card numbers, SSN
// Do log: user_id, ip_address, user_agent, timestamp, action, resource_id
```

## File Upload Security

```typescript
// Validate: mime type (not just extension), file size, content
// Store outside web root
// Generate random filename (never use user-supplied filename)
// Virus scan for user content

const upload = multer({
  limits: { fileSize: 10 * 1024 * 1024 },  // 10MB max
  fileFilter: (req, file, cb) => {
    const allowedTypes = ['image/jpeg', 'image/png', 'image/webp'];
    cb(null, allowedTypes.includes(file.mimetype));
  },
  storage: multer.diskStorage({
    filename: (req, file, cb) => cb(null, `${crypto.randomUUID()}${path.extname(file.originalname)}`),
  }),
});
```
