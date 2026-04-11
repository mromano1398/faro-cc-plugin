# OWASP Checklist — Faro Security

Checklist dettagliata OWASP Top 10:2025 + pattern auth + GDPR per progetti Faro.

## A01 — Broken Access Control

- [ ] Ogni Server Action usa `withAuth()` wrapper
- [ ] Ogni query filtra per `userId` o `organizationId` del session
- [ ] Ownership check esplicito: `WHERE id = $1 AND user_id = $2`
- [ ] Nessun parametro `userId` accettato dal client
- [ ] Admin routes protette con middleware role check

**Pattern `withAuth`:**
```ts
export function withAuth<T>(
  handler: (session: Session, ...args: any[]) => Promise<T>
) {
  return async (...args: any[]) => {
    const session = await getServerSession()
    if (!session?.user) throw new UnauthorizedError()
    return handler(session, ...args)
  }
}
```

## A02 — Cryptographic Failures

- [ ] TLS ovunque (`?sslmode=require` per DB)
- [ ] Password hash con `bcrypt` (cost >= 12) o `argon2id`
- [ ] Token JWT firmati con chiave >= 256 bit
- [ ] Secret in env vars, mai in repo
- [ ] Backup cifrati at-rest (GPG o S3 SSE-KMS)

## A03 — Injection

- [ ] Query parametriche sempre (Prisma/Drizzle/SQLAlchemy)
- [ ] Validazione input con Zod/Pydantic prima della DB
- [ ] Escape HTML in output (React lo fa per default — verifica `dangerouslySetInnerHTML`)
- [ ] Command injection: mai `exec(userInput)`
- [ ] LDAP/XML/NoSQL injection verificati se applicabili

## A04 — Insecure Design

- [ ] Threat modeling fatto prima dello sviluppo
- [ ] Rate limiting su tutti i flussi sensibili
- [ ] Business logic validata server-side (mai fidarsi del client)
- [ ] Workflow multi-step con state server-side

## A05 — Security Misconfiguration

- [ ] Security headers configurati:
  ```js
  // next.config.js
  const securityHeaders = [
    { key: 'Strict-Transport-Security', value: 'max-age=63072000; includeSubDomains; preload' },
    { key: 'X-Frame-Options', value: 'DENY' },
    { key: 'X-Content-Type-Options', value: 'nosniff' },
    { key: 'Referrer-Policy', value: 'strict-origin-when-cross-origin' },
    { key: 'Permissions-Policy', value: 'camera=(), microphone=(), geolocation=()' },
    { key: 'Content-Security-Policy', value: "default-src 'self'; ..." },
  ]
  ```
- [ ] Error messages non rivelano stack in produzione
- [ ] Default credentials rimosse
- [ ] Admin UI non esposta pubblicamente

## A06 — Vulnerable Components

- [ ] `npm audit` zero critical/high
- [ ] Dependabot attivo o `npm outdated` weekly
- [ ] Lockfile committato (`package-lock.json`)
- [ ] Unused dependencies rimosse

## A07 — Identification and Authentication Failures

- [ ] Password min 12 char + complexity
- [ ] MFA opzionale per utenti, obbligatorio per admin
- [ ] Rate limiting su login (5 tentativi / 15 min per IP)
- [ ] Session timeout ragionevole (8h work, 30min sensitive)
- [ ] Logout invalida token server-side
- [ ] Reset password con token monouso + scadenza (15 min)

## A08 — Software and Data Integrity Failures

- [ ] CI pipeline verifica checksum dipendenze
- [ ] Webhook firma verificata (HMAC)
- [ ] Serializzazione sicura (no `pickle` Python su input untrusted)

## A09 — Security Logging and Monitoring Failures

- [ ] Audit log per azioni sensibili (login, admin, payment, export)
- [ ] PII mascherata: `user_id` sì, `email` no in chiaro
- [ ] Log centralizzati (CloudWatch / Loki / Datadog)
- [ ] Alert su: login anomali, spike 5xx, failed payments
- [ ] Retention log: 90 giorni minimo (1 anno per compliance)

**Pattern audit log:**
```ts
await db.auditLog.create({
  data: {
    userId: session.user.id,
    action: 'USER_DELETE',
    resourceId: targetId,
    ipAddress: maskIp(request.headers.get('x-forwarded-for')),
    userAgent: request.headers.get('user-agent'),
    timestamp: new Date(),
  },
})
```

## A10 — Server-Side Request Forgery (SSRF)

- [ ] URL user-supplied: whitelist di domini consentiti
- [ ] Blocca IP privati (10.x, 172.16.x, 192.168.x, 169.254.x, localhost)
- [ ] Timeout aggressivi su fetch (5s max)
- [ ] Redirect disabilitati o limitati (max 3 hop)

## FILE UPLOAD — Pattern sicuro

```ts
// app/api/upload/route.ts
import { writeFile } from 'fs/promises'
import { fileTypeFromBuffer } from 'file-type'

const ALLOWED_TYPES = ['image/jpeg', 'image/png', 'application/pdf']
const MAX_SIZE = 10 * 1024 * 1024 // 10MB

export async function POST(request: Request) {
  const session = await requireAuth()
  const formData = await request.formData()
  const file = formData.get('file') as File

  if (!file) return new Response('No file', { status: 400 })
  if (file.size > MAX_SIZE) return new Response('Too large', { status: 413 })

  const buffer = Buffer.from(await file.arrayBuffer())

  // Magic bytes check — non fidarsi del Content-Type
  const detectedType = await fileTypeFromBuffer(buffer)
  if (!detectedType || !ALLOWED_TYPES.includes(detectedType.mime)) {
    return new Response('Invalid type', { status: 415 })
  }

  // Salva FUORI da public/
  const filename = `${crypto.randomUUID()}.${detectedType.ext}`
  const path = `/app/private/uploads/${session.user.id}/${filename}`
  await writeFile(path, buffer)

  // DB reference con ownership
  await db.upload.create({
    data: { userId: session.user.id, path, mimeType: detectedType.mime },
  })

  return Response.json({ ok: true, id: filename })
}
```

**Download con auth:**
```ts
// app/api/uploads/[id]/route.ts
export async function GET(req: Request, { params }) {
  const session = await requireAuth()
  const upload = await db.upload.findFirst({
    where: { id: params.id, userId: session.user.id }, // ownership
  })
  if (!upload) return new Response('Not found', { status: 404 })
  const file = await readFile(upload.path)
  return new Response(file, {
    headers: { 'Content-Type': upload.mimeType },
  })
}
```

## GDPR — Checklist essenziale

- [ ] **Art. 5** — minimizzazione dati: raccogli solo il necessario
- [ ] **Art. 13** — privacy policy chiara, linkata da ogni form
- [ ] **Art. 15** — export dati utente (JSON downloadable)
- [ ] **Art. 17** — cancellazione account (soft delete + purge dopo N giorni)
- [ ] **Art. 25** — privacy by design (default restrittivi)
- [ ] **Art. 30** — registro trattamenti (per DPO)
- [ ] **Art. 32** — misure tecniche: encryption, backup, access control
- [ ] **Art. 33** — data breach notification (72h)
- [ ] Cookie banner con consenso granulare (no, no tracking senza OK)
- [ ] DPA firmati con tutti i sub-processor (Vercel, Stripe, Supabase, Sentry)

## RATE LIMITING — Pattern con Upstash

```ts
import { Ratelimit } from '@upstash/ratelimit'
import { Redis } from '@upstash/redis'

const authLimiter = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(5, '15 m'),
  prefix: 'ratelimit:auth',
})

export async function checkAuthRateLimit(ip: string) {
  const { success, remaining, reset } = await authLimiter.limit(ip)
  if (!success) {
    throw new Error(`Too many attempts. Retry after ${new Date(reset).toISOString()}`)
  }
  return remaining
}
```

## TOOL AUTOMATICI — Pre-commit

```yaml
# .github/workflows/security.yml
name: Security
on: [push, pull_request]
jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with: { fetch-depth: 0 }
      - name: Gitleaks
        uses: gitleaks/gitleaks-action@v2
      - name: Semgrep
        uses: returntocorp/semgrep-action@v1
        with: { config: 'p/owasp-top-ten' }
      - name: npm audit
        run: npm audit --audit-level=high
```

## SECURITY.TXT

`public/.well-known/security.txt`:
```
Contact: mailto:security@faro.example.com
Expires: 2027-01-01T00:00:00.000Z
Preferred-Languages: it, en
Canonical: https://faro.example.com/.well-known/security.txt
```
