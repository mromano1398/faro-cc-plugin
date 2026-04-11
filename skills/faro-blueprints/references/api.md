# Blueprint — API (backend only)

Progetto API puro senza frontend. Per casi: microservizio, backend di app mobile/desktop, webhook
processor, integrazione tra sistemi, API pubblica per sviluppatori terzi.

## Stack consigliati

### Node.js/TypeScript
- **Hono** (leggero, edge-first) — raccomandato per API piccole-medie
- **Express** — legacy, ok per progetti grandi con ecosistema ricco
- **NestJS** — enterprise, con opinioni forti (DI, decoratori)

### Python
- **FastAPI** (moderno, async, auto OpenAPI) — raccomandato
- **Django REST Framework** — se serve admin + ORM pesante

### Go
- **Gin** o **Echo** — performance-critical

### Rust
- **Axum** — correttezza + performance

Default Faro: **Hono + Node.js** per API TypeScript moderne, **FastAPI** per Python.

## Struttura cartelle (Hono + TypeScript)

```
src/
├── index.ts                        # Entry point: new Hono()
├── routes/
│   ├── auth.ts                     # /auth/*
│   ├── users.ts                    # /users/*
│   └── [resource].ts               # /[resource]/*
├── middleware/
│   ├── auth.ts                     # Bearer token validation
│   ├── rate-limit.ts               # Upstash rate limit
│   ├── cors.ts
│   └── logging.ts
├── lib/
│   ├── db.ts                       # Prisma/Drizzle client
│   ├── auth.ts                     # JWT verify, session
│   └── errors.ts                   # HTTPException helpers
├── schemas/                        # Zod schemas per request/response
│   ├── user.ts
│   └── [resource].ts
├── services/                       # Business logic
│   └── [resource].ts
└── types/
```

## Struttura cartelle (FastAPI + Python)

```
src/
├── main.py                         # FastAPI app, middleware, routers include
├── api/
│   ├── v1/
│   │   ├── auth.py
│   │   ├── users.py
│   │   └── [resource].py
│   └── deps.py                     # Dependencies (auth, db session)
├── core/
│   ├── config.py                   # Settings (pydantic BaseSettings)
│   ├── security.py                 # JWT, password hashing
│   └── errors.py
├── db/
│   ├── base.py                     # SQLAlchemy Base
│   ├── session.py
│   └── models/
├── schemas/                        # Pydantic schemas
│   └── [resource].py
├── services/
│   └── [resource].py
└── alembic/                        # Migrations
```

## Security checklist (sempre)

- [ ] Autenticazione: API key o JWT Bearer token (OBBLIGATORIA salvo endpoint pubblici espliciti)
- [ ] HTTPS forzato in produzione
- [ ] Rate limiting per-key/per-IP (Upstash Redis o memoria + LRU)
- [ ] CORS configurato esplicitamente (non `*` in prod)
- [ ] Validazione input con Zod (TS) o Pydantic (Py) su OGNI endpoint
- [ ] Output: no dati sensibili (password hash, token interni)
- [ ] Error handling: no stack trace in risposta produzione
- [ ] Logging strutturato (JSON) con request ID + user ID
- [ ] API versioning (/v1/, /v2/) per backward compatibility
- [ ] OpenAPI spec generata automaticamente (FastAPI nativo, Hono con @hono/zod-openapi)

## Convenzioni API REST

- Risorse plurali: `/users`, `/posts`
- CRUD: GET /users, GET /users/:id, POST /users, PATCH /users/:id, DELETE /users/:id
- Nested: /users/:id/posts
- Status code standard: 200 OK, 201 Created, 400 Bad Request, 401 Unauthorized, 403 Forbidden,
  404 Not Found, 409 Conflict, 422 Unprocessable Entity, 429 Rate Limit, 500 Server Error
- Paginazione: `?page=1&limit=20` oppure cursor-based `?cursor=xxx&limit=20`
- Filtri: `?status=active&created_after=2024-01-01`
- Formato risposta:
  ```json
  { "data": {...}, "meta": { "page": 1, "total": 100 } }
  { "error": { "code": "VALIDATION_ERROR", "message": "...", "details": [] } }
  ```

## Deploy

- **Vercel Functions** (Hono, Express) — scaling automatico, edge ready
- **Railway** — Docker-ready, backend classico con DB
- **Fly.io** — multi-region, Docker
- **Docker + VPS** — controllo completo, DIY

## Testing

- Unit: singoli service/handler con mock DB
- Integration: test che chiamano l'API contro un DB di test (Docker compose)
- Contract: validare risposte contro OpenAPI spec
- Load test: k6 o Artillery per verificare rate limit e capacity

## Anti-pattern

- API senza autenticazione "perché è interna" → prima o poi diventa pubblica
- Response body con campi diversi a seconda dei casi senza wrapper standard
- Nessun versioning → break change = disastro per i consumer
- Status code inconsistenti (es: 200 con `{ error: ... }` invece di 400)
- Rate limiting "ci pensiamo dopo" → prima incidente e poi urgenza
