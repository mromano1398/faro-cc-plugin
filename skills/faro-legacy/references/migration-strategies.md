# Migration Strategies — Faro Legacy

Strategie dettagliate per migrare applicazioni legacy (PHP/MySQL/jQuery) a stack moderno.

## FASE 0 — INVENTARIO

Prima di qualsiasi riga di codice, compila `faro/legacy-inventory.md`:

```markdown
# Legacy Inventory

## Pagine pubbliche
| URL | Scopo | Traffico/mese | Priorità |
|-----|-------|---------------|----------|
| /home | Landing | 100k | 1 |
| /catalogo | Lista prodotti | 80k | 1 |
| /prodotto/{id} | Dettaglio | 50k | 1 |

## Tabelle database
| Tabella | Record | Ha trigger? | Ha FK? | Note |
|---------|--------|-------------|--------|------|
| users | 50k | no | sì | password MD5 (!) |
| orders | 200k | sì (log) | sì | |

## Stored procedures
| Nome | Chiamata da | Complessità | Piano migrazione |
|------|-------------|-------------|------------------|
| sp_calcola_totale | PHP checkout | media | riscrivi in service layer |

## Cron job
| Script | Frequenza | Scopo | Sostituto moderno |
|--------|-----------|-------|-------------------|
| cleanup_sessions.php | ogni 15m | pulisci sessioni scadute | Next.js cron route |

## File upload
- `/var/www/uploads/` → ~20GB, 120k file
- Path salvato in DB come relativo (`uploads/2020/01/foto.jpg`)

## Integrazioni esterne
| Servizio | API | Auth | Uso |
|----------|-----|------|-----|
| Stripe | v2020 | API key | Pagamenti |
| Mailjet | REST | Basic | Newsletter |

## Auth legacy
- Cookie sessione PHP (`PHPSESSID`)
- Sessione salvata in `/tmp/sess_*`
- No MFA
```

## FASE 1 — NGINX ROUTING IBRIDO

```nginx
upstream legacy_app {
  server 127.0.0.1:8080; # vecchio PHP
}

upstream faro_new {
  server 127.0.0.1:3000; # nuovo Next.js
}

server {
  listen 443 ssl http2;
  server_name app.example.com;

  # Feature già migrate → nuovo
  location /catalogo {
    proxy_pass http://faro_new;
  }
  location /prodotti {
    proxy_pass http://faro_new;
  }
  location /api {
    proxy_pass http://faro_new;
  }

  # Redirect vecchi URL a nuovi
  location = /old-catalog.php {
    return 301 /catalogo;
  }

  # Tutto il resto → legacy
  location / {
    proxy_pass http://legacy_app;
  }

  # Bridge cookie sessione
  location /auth/bridge {
    proxy_pass http://faro_new/api/auth/bridge-legacy;
    proxy_set_header X-Legacy-Session $cookie_PHPSESSID;
  }
}
```

## FASE 2 — SESSION BRIDGING

Utenti già loggati nel legacy non devono essere disconnessi.

```ts
// app/api/auth/bridge-legacy/route.ts
import { NextResponse } from 'next/server'
import { createHash } from 'crypto'

export async function GET(request: Request) {
  const legacySession = request.headers.get('x-legacy-session')
  if (!legacySession) return NextResponse.json({ ok: false }, { status: 401 })

  // Chiama il legacy per validare la sessione
  const res = await fetch('http://legacy-internal/api/session-check', {
    headers: { Cookie: `PHPSESSID=${legacySession}` },
  })
  if (!res.ok) return NextResponse.json({ ok: false }, { status: 401 })

  const { user_id, email } = await res.json()

  // Trova o crea utente nel nuovo DB
  let user = await db.user.findUnique({ where: { legacyId: user_id } })
  if (!user) {
    user = await db.user.create({
      data: {
        legacyId: user_id,
        email,
        migrated: true,
      },
    })
  }

  // Crea sessione NextAuth e imposta cookie
  const token = await createJwtSession(user)
  const response = NextResponse.json({ ok: true, user })
  response.cookies.set('faro-session', token, { httpOnly: true, secure: true })
  return response
}
```

## FASE 3 — DUAL WRITE (opzionale, complesso)

Scrivi su entrambi i DB durante il periodo di transizione:

```ts
// services/ordini.ts
export async function creaOrdine(data) {
  // Scrivi nel nuovo
  const ordineNuovo = await db.ordine.create({ data })

  // Scrivi nel legacy (per coerenza)
  try {
    await legacyDb.query(
      'INSERT INTO ordini (user_id, totale, stato) VALUES (?, ?, ?)',
      [data.userId, data.totale, 'pending']
    )
  } catch (error) {
    // Legacy fallito — compensa rimuovendo dal nuovo
    await db.ordine.delete({ where: { id: ordineNuovo.id } })
    throw new Error('Dual-write fallito: ' + error)
  }

  return ordineNuovo
}
```

**Warning:** Dual-write è fragile. Considera cutover rapido come alternativa.

## FASE 4 — CONVERSIONE MYSQL → POSTGRESQL

### 4.1 pgloader (raccomandato)

```lisp
;; migrate.load
LOAD DATABASE
  FROM mysql://user:pass@legacy-db/old_app
  INTO postgresql://user:pass@new-db/faro

WITH
  include drop,
  create tables,
  create indexes,
  reset sequences,
  foreign keys,
  downcase identifiers

CAST
  type datetime to timestamptz drop default drop not null using zero-dates-to-null,
  type date drop not null drop default using zero-dates-to-null,
  type tinyint when (= precision 1) to boolean drop typemod keep default,
  type int with extra auto_increment to integer drop typemod keep default

BEFORE LOAD DO
  $$ CREATE SCHEMA IF NOT EXISTS faro; $$

SET
  search_path TO faro;
```

Esegui: `pgloader migrate.load`

### 4.2 Verifica post-migrazione

```sql
-- Count per tabella
SELECT 'users' AS t, COUNT(*) FROM users
UNION ALL SELECT 'orders', COUNT(*) FROM orders
UNION ALL SELECT 'products', COUNT(*) FROM products;

-- Confronta con MySQL originale

-- Verifica sequences
SELECT setval(pg_get_serial_sequence('users', 'id'), (SELECT MAX(id) FROM users));
SELECT setval(pg_get_serial_sequence('orders', 'id'), (SELECT MAX(id) FROM orders));

-- Verifica integrità FK
SELECT conrelid::regclass, conname, pg_catalog.pg_get_constraintdef(oid)
FROM pg_constraint WHERE contype = 'f';
```

### 4.3 Fix patologie comuni

```sql
-- TINYINT(1) non convertiti → converti manualmente
ALTER TABLE users ALTER COLUMN active TYPE BOOLEAN USING active::int::boolean;

-- ENUM MySQL → tipo custom PostgreSQL
CREATE TYPE user_role AS ENUM ('admin', 'user', 'guest');
ALTER TABLE users ALTER COLUMN role TYPE user_role USING role::user_role;

-- citext per email case-insensitive
CREATE EXTENSION IF NOT EXISTS citext;
ALTER TABLE users ALTER COLUMN email TYPE citext;
```

## FASE 5 — FILE UPLOAD MIGRATION

```bash
#!/bin/bash
# Migra file da filesystem legacy a S3/R2
set -euo pipefail

SOURCE=/var/www/legacy/uploads
BUCKET=s3://faro-uploads

rsync -av --progress $SOURCE/ /tmp/uploads-migration/
aws s3 sync /tmp/uploads-migration/ $BUCKET --storage-class STANDARD

# Aggiorna path in DB
psql $DATABASE_URL <<EOF
UPDATE uploads
SET path = REPLACE(path, '/var/www/legacy/uploads/', 'https://cdn.faro.example.com/'),
    migrated_at = NOW()
WHERE migrated_at IS NULL;
EOF
```

## FASE 6 — STORED PROCEDURES → APPLICATION LAYER

**NO conversione 1:1** in PL/pgSQL. Riscrivi come service.

```ts
// Era: sp_calcola_totale(order_id)
// Ora: services/ordini/calcolaTotale.ts
export async function calcolaTotale(orderId: string) {
  const righe = await db.ordineRiga.findMany({
    where: { orderId },
    include: { prodotto: true },
  })

  let subtotale = 0
  for (const riga of righe) {
    subtotale += riga.quantita * riga.prezzoUnitario
  }

  const sconto = await calcolaSconto(subtotale)
  const iva = (subtotale - sconto) * 0.22
  return { subtotale, sconto, iva, totale: subtotale - sconto + iva }
}
```

## FASE 7 — CUTOVER E ROLLBACK

### Cutover plan

```markdown
# Cutover 2026-05-15

## T-7 giorni
- [ ] Backup full legacy DB
- [ ] Freeze feature legacy
- [ ] Comunicazione utenti (banner)

## T-1 giorno
- [ ] Backup incrementale
- [ ] Test restore
- [ ] Team on-call confermato

## Giorno X (notturno, 02:00-06:00)
- [ ] 02:00 — Banner "manutenzione"
- [ ] 02:15 — Stop scritture legacy
- [ ] 02:30 — Dump finale + import delta
- [ ] 03:30 — Switch Nginx routing su faro_new
- [ ] 04:00 — Smoke test 10 flussi critici
- [ ] 05:00 — Rimuovi banner
- [ ] 06:00 — Monitor attivo per 4h

## Rollback
- Criterio: error rate > 5% o flusso critico down
- Procedura: `nginx -s reload` con config vecchia (ready in /etc/nginx/sites-available/legacy.conf)
- Decision maker: tech lead (numero: ...)
- Tempo max decisione: 15 minuti
```

### Legacy read-only 30 giorni

```nginx
# Dopo cutover: mantieni legacy in read-only
server {
  listen 8080;
  server_name legacy.internal;

  location / {
    if ($request_method !~ ^(GET|HEAD)$) {
      return 503 "Legacy in sola lettura. Vai su https://app.example.com";
    }
    proxy_pass http://legacy_backend;
  }
}
```

Dopo 30 giorni senza incidenti → spegni definitivamente legacy.
