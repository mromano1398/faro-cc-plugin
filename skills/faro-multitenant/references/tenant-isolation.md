# Tenant Isolation — Faro Multitenant

Pattern completi per multi-tenancy con isolamento dati sicuro.

## SCHEMA DB — Pattern row-level

```sql
-- Organizzazioni (tenant)
CREATE TABLE organizzazioni (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  nome TEXT NOT NULL,
  slug TEXT UNIQUE NOT NULL,
  piano TEXT CHECK (piano IN ('FREE', 'PRO', 'ENTERPRISE')) DEFAULT 'FREE',
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX ON organizzazioni (slug);

-- Membri (relazione N:N utenti ↔ org con ruolo)
CREATE TABLE membri_organizzazione (
  organizzazione_id UUID REFERENCES organizzazioni(id) ON DELETE CASCADE,
  utente_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
  ruolo TEXT CHECK (ruolo IN ('OWNER', 'ADMIN', 'MEMBER')) DEFAULT 'MEMBER',
  joined_at TIMESTAMPTZ DEFAULT NOW(),
  PRIMARY KEY (organizzazione_id, utente_id)
);

CREATE INDEX ON membri_organizzazione (utente_id);

-- Tabella business: organizzazione_id è OBBLIGATORIO
CREATE TABLE progetti (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  organizzazione_id UUID NOT NULL REFERENCES organizzazioni(id) ON DELETE CASCADE,
  nome TEXT NOT NULL,
  creato_da UUID REFERENCES auth.users(id),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX ON progetti (organizzazione_id);
```

## FUNZIONE HELPER

```sql
-- Restituisce le organizzazioni di cui l'utente fa parte
CREATE OR REPLACE FUNCTION mie_organizzazioni()
RETURNS SETOF UUID
LANGUAGE SQL
SECURITY DEFINER
STABLE
AS $$
  SELECT organizzazione_id
  FROM membri_organizzazione
  WHERE utente_id = auth.uid();
$$;

-- Variante: verifica ruolo minimo
CREATE OR REPLACE FUNCTION ho_ruolo_in(org_id UUID, ruolo_richiesto TEXT)
RETURNS BOOLEAN
LANGUAGE SQL
SECURITY DEFINER
STABLE
AS $$
  SELECT EXISTS (
    SELECT 1 FROM membri_organizzazione
    WHERE organizzazione_id = org_id
      AND utente_id = auth.uid()
      AND CASE ruolo_richiesto
            WHEN 'OWNER'  THEN ruolo = 'OWNER'
            WHEN 'ADMIN'  THEN ruolo IN ('OWNER', 'ADMIN')
            WHEN 'MEMBER' THEN ruolo IN ('OWNER', 'ADMIN', 'MEMBER')
          END
  );
$$;
```

## RLS POLICY — Multi-tenant

```sql
ALTER TABLE progetti ENABLE ROW LEVEL SECURITY;

-- SELECT: tutti i membri vedono
CREATE POLICY "membri_vedono_progetti"
ON progetti FOR SELECT
USING (organizzazione_id IN (SELECT mie_organizzazioni()));

-- INSERT: membri+ creano
CREATE POLICY "membri_creano_progetti"
ON progetti FOR INSERT
WITH CHECK (organizzazione_id IN (SELECT mie_organizzazioni()));

-- UPDATE: admin+ modificano
CREATE POLICY "admin_modificano_progetti"
ON progetti FOR UPDATE
USING (ho_ruolo_in(organizzazione_id, 'ADMIN'));

-- DELETE: solo owner
CREATE POLICY "owner_cancellano_progetti"
ON progetti FOR DELETE
USING (ho_ruolo_in(organizzazione_id, 'OWNER'));
```

## TENANT DETECTION — Subdomain

```ts
// middleware.ts
import { NextResponse, type NextRequest } from 'next/server'

const ROOT_DOMAIN = 'faro.example.com'
const RESERVED = ['www', 'api', 'admin', 'app']

export function middleware(request: NextRequest) {
  const host = request.headers.get('host') ?? ''
  const subdomain = host.replace(`.${ROOT_DOMAIN}`, '').split(':')[0]

  if (RESERVED.includes(subdomain) || host === ROOT_DOMAIN) {
    return NextResponse.next()
  }

  // Rewrite a path /t/[slug]/...
  const url = request.nextUrl.clone()
  url.pathname = `/t/${subdomain}${url.pathname}`
  return NextResponse.rewrite(url)
}
```

## TENANT DETECTION — Path based

```ts
// app/(tenant)/[slug]/layout.tsx
import { notFound } from 'next/navigation'
import { createClient } from '@/lib/supabase/server'

export default async function TenantLayout({
  children,
  params,
}: {
  children: React.ReactNode
  params: Promise<{ slug: string }>
}) {
  const { slug } = await params
  const supabase = await createClient()
  const { data: { user } } = await supabase.auth.getUser()
  if (!user) return notFound()

  const { data: org } = await supabase
    .from('organizzazioni')
    .select('*, membri_organizzazione!inner(ruolo)')
    .eq('slug', slug)
    .eq('membri_organizzazione.utente_id', user.id)
    .maybeSingle()

  if (!org) return notFound() // isolamento: non riveli esistenza

  return (
    <TenantProvider organizzazione={org}>
      {children}
    </TenantProvider>
  )
}
```

## PROVISIONING TRANSAZIONALE

```ts
// app/api/organizzazioni/route.ts
import { adminClient } from '@/lib/supabase/admin'
import { z } from 'zod'

const schema = z.object({
  nome: z.string().min(2).max(60),
  slug: z.string().regex(/^[a-z0-9-]{3,30}$/),
})

export async function POST(request: Request) {
  const session = await requireAuth()
  const body = await request.json()
  const { nome, slug } = schema.parse(body)

  // Verifica slug disponibile
  const { data: existing } = await adminClient
    .from('organizzazioni')
    .select('id')
    .eq('slug', slug)
    .maybeSingle()
  if (existing) {
    return Response.json({ error: 'Slug già in uso' }, { status: 409 })
  }

  // Transazione: crea org + membro owner + dati default
  const { data: org, error } = await adminClient.rpc('crea_organizzazione', {
    p_nome: nome,
    p_slug: slug,
    p_owner_id: session.user.id,
  })

  if (error) {
    return Response.json({ error: error.message }, { status: 500 })
  }

  return Response.json({ org }, { status: 201 })
}
```

Funzione SQL transazionale:
```sql
CREATE OR REPLACE FUNCTION crea_organizzazione(
  p_nome TEXT,
  p_slug TEXT,
  p_owner_id UUID
)
RETURNS UUID
LANGUAGE plpgsql
SECURITY DEFINER
AS $$
DECLARE
  v_org_id UUID;
BEGIN
  INSERT INTO organizzazioni (nome, slug)
  VALUES (p_nome, p_slug)
  RETURNING id INTO v_org_id;

  INSERT INTO membri_organizzazione (organizzazione_id, utente_id, ruolo)
  VALUES (v_org_id, p_owner_id, 'OWNER');

  -- Dati default: progetto di benvenuto
  INSERT INTO progetti (organizzazione_id, nome, creato_da)
  VALUES (v_org_id, 'Primo progetto', p_owner_id);

  RETURN v_org_id;
END;
$$;
```

## SISTEMA INVITI

```sql
CREATE TABLE inviti (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  organizzazione_id UUID NOT NULL REFERENCES organizzazioni(id) ON DELETE CASCADE,
  email TEXT NOT NULL,
  ruolo TEXT CHECK (ruolo IN ('ADMIN', 'MEMBER')) DEFAULT 'MEMBER',
  token TEXT UNIQUE NOT NULL DEFAULT encode(gen_random_bytes(32), 'hex'),
  scade_il TIMESTAMPTZ NOT NULL DEFAULT NOW() + INTERVAL '7 days',
  invitato_da UUID REFERENCES auth.users(id),
  accettato_il TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

```ts
// Invia invito
export async function inviaInvito(orgId: string, email: string, ruolo: 'ADMIN' | 'MEMBER') {
  const session = await requireAuth()
  if (!await haRuoloIn(orgId, 'ADMIN')) throw new Error('Solo admin possono invitare')

  const { data: invito } = await adminClient
    .from('inviti')
    .insert({ organizzazione_id: orgId, email, ruolo, invitato_da: session.user.id })
    .select()
    .single()

  await inviaEmail({
    to: email,
    subject: 'Sei stato invitato',
    html: `<a href="${process.env.APP_URL}/invito/${invito.token}">Accetta</a>`,
  })
}

// Accetta invito
export async function accettaInvito(token: string) {
  const session = await requireAuth()
  const { data: invito } = await adminClient
    .from('inviti')
    .select('*')
    .eq('token', token)
    .is('accettato_il', null)
    .gt('scade_il', new Date().toISOString())
    .maybeSingle()

  if (!invito) throw new Error('Invito non valido o scaduto')

  await adminClient.from('membri_organizzazione').insert({
    organizzazione_id: invito.organizzazione_id,
    utente_id: session.user.id,
    ruolo: invito.ruolo,
  })

  await adminClient
    .from('inviti')
    .update({ accettato_il: new Date().toISOString() })
    .eq('id', invito.id)
}
```

## FEATURE GATES — Piano limits

```ts
// lib/limits.ts
export const PIANI = {
  FREE: { maxProgetti: 3, maxMembri: 2, maxStorageMB: 100 },
  PRO: { maxProgetti: 50, maxMembri: 10, maxStorageMB: 5000 },
  ENTERPRISE: { maxProgetti: Infinity, maxMembri: Infinity, maxStorageMB: Infinity },
}

export async function checkLimiteProgetti(orgId: string) {
  const { data: org } = await adminClient
    .from('organizzazioni')
    .select('piano')
    .eq('id', orgId)
    .single()

  const { count } = await adminClient
    .from('progetti')
    .select('*', { count: 'exact', head: true })
    .eq('organizzazione_id', orgId)

  const limite = PIANI[org!.piano].maxProgetti
  if ((count ?? 0) >= limite) {
    throw new Error(`Hai raggiunto il limite del piano ${org!.piano} (${limite} progetti)`)
  }
}

// Usa in ogni create
export async function creaProgetto(orgId: string, nome: string) {
  await checkLimiteProgetti(orgId) // gate
  return adminClient.from('progetti').insert({ organizzazione_id: orgId, nome })
}
```

## MIGRATION ZERO-DOWNTIME (Expand/Contract)

```sql
-- FASE 1 — EXPAND (aggiungi nullable)
ALTER TABLE utenti ADD COLUMN organizzazione_id UUID REFERENCES organizzazioni(id);

-- FASE 2 — MIGRATE (batch)
UPDATE utenti SET organizzazione_id = 'xxx-default'
WHERE organizzazione_id IS NULL AND id IN (
  SELECT id FROM utenti WHERE organizzazione_id IS NULL LIMIT 1000
);
-- Ripeti in loop finché COUNT = 0

-- FASE 3 — CONTRACT (applica constraint)
ALTER TABLE utenti ALTER COLUMN organizzazione_id SET NOT NULL;
CREATE INDEX CONCURRENTLY ON utenti (organizzazione_id);
```

## RATE LIMITING PER TENANT

```ts
import { Ratelimit } from '@upstash/ratelimit'
import { Redis } from '@upstash/redis'

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(100, '1 m'),
  prefix: 'rl:tenant',
})

export async function withTenantRateLimit(tenantId: string) {
  const { success, remaining } = await ratelimit.limit(tenantId)
  if (!success) throw new Error(`Rate limit superato per il tenant ${tenantId}`)
  return remaining
}
```

## ERROR CORRELATION — Sentry con tenant_id

```ts
import * as Sentry from '@sentry/nextjs'

export function setTenantContext(tenantId: string, userId: string) {
  Sentry.setTag('tenant_id', tenantId)
  Sentry.setUser({ id: userId })
}

// In layout tenant:
const session = await getServerSession()
setTenantContext(session.organizzazioneId, session.user.id)
```

## TEST CROSS-TENANT

```ts
describe('multi-tenant isolation', () => {
  it('utente di org A non vede progetti di org B', async () => {
    const { user: a } = await createTestUser('a@test')
    const { user: b } = await createTestUser('b@test')
    const orgA = await createOrg('A', a.id)
    const orgB = await createOrg('B', b.id)

    await adminClient.from('progetti').insert({
      organizzazione_id: orgB,
      nome: 'Segreto B',
    })

    const clientA = await authenticatedClient(a)
    const { data } = await clientA.from('progetti').select('*').eq('organizzazione_id', orgB)
    expect(data).toEqual([]) // RLS nasconde
  })
})
```

## SSO/SAML — Nota

Per SSO enterprise completo considera **Clerk**, **WorkOS** o **Auth.js + samlify**.
Non reinventare SAML: è complesso e security-critical.
