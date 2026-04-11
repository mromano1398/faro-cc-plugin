# Supabase Patterns — Faro

Pattern completi per Row Level Security, Auth, Storage, Realtime ed Edge Functions in progetti Faro.

## SETUP INIZIALE

```bash
npm install @supabase/supabase-js @supabase/ssr
npx supabase init
npx supabase login
npx supabase link --project-ref your-project-ref
```

`.env.local`:
```
NEXT_PUBLIC_SUPABASE_URL=https://xxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGc...
SUPABASE_SERVICE_ROLE_KEY=eyJhbGc... # server-side only!
```

## CLIENT SSR — Next.js

`lib/supabase/server.ts`:
```ts
import { createServerClient } from '@supabase/ssr'
import { cookies } from 'next/headers'

export async function createClient() {
  const cookieStore = await cookies()
  return createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll() { return cookieStore.getAll() },
        setAll(cookiesToSet) {
          try {
            cookiesToSet.forEach(({ name, value, options }) =>
              cookieStore.set(name, value, options)
            )
          } catch { /* chiamata da RSC */ }
        },
      },
    }
  )
}
```

`lib/supabase/client.ts`:
```ts
import { createBrowserClient } from '@supabase/ssr'

export function createClient() {
  return createBrowserClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
  )
}
```

`lib/supabase/admin.ts` (MAI importare lato client):
```ts
import { createClient } from '@supabase/supabase-js'

export const adminClient = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!,
  { auth: { autoRefreshToken: false, persistSession: false } }
)
```

## MIDDLEWARE — Route protection

`middleware.ts`:
```ts
import { createServerClient } from '@supabase/ssr'
import { NextResponse, type NextRequest } from 'next/server'

export async function middleware(request: NextRequest) {
  let response = NextResponse.next({ request })

  const supabase = createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll() { return request.cookies.getAll() },
        setAll(cookiesToSet) {
          cookiesToSet.forEach(({ name, value }) => request.cookies.set(name, value))
          response = NextResponse.next({ request })
          cookiesToSet.forEach(({ name, value, options }) =>
            response.cookies.set(name, value, options)
          )
        },
      },
    }
  )

  const { data: { user } } = await supabase.auth.getUser()

  const isProtected = request.nextUrl.pathname.startsWith('/dashboard')
  if (isProtected && !user) {
    return NextResponse.redirect(new URL('/login', request.url))
  }

  return response
}

export const config = {
  matcher: ['/((?!_next/static|_next/image|favicon.ico).*)'],
}
```

## RLS — Pattern owner

```sql
-- Tabella con owner
CREATE TABLE progetti (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  utente_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  nome TEXT NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Attiva RLS (obbligatorio!)
ALTER TABLE progetti ENABLE ROW LEVEL SECURITY;

-- SELECT: solo owner
CREATE POLICY "utenti_vedono_propri_progetti"
ON progetti FOR SELECT
USING (auth.uid() = utente_id);

-- INSERT: solo sui propri record
CREATE POLICY "utenti_creano_propri_progetti"
ON progetti FOR INSERT
WITH CHECK (auth.uid() = utente_id);

-- UPDATE: solo propri record
CREATE POLICY "utenti_modificano_propri_progetti"
ON progetti FOR UPDATE
USING (auth.uid() = utente_id)
WITH CHECK (auth.uid() = utente_id);

-- DELETE: solo propri record
CREATE POLICY "utenti_cancellano_propri_progetti"
ON progetti FOR DELETE
USING (auth.uid() = utente_id);
```

## RLS — Pattern RBAC (ruoli)

```sql
-- Tabella ruoli
CREATE TABLE profili (
  id UUID PRIMARY KEY REFERENCES auth.users(id),
  ruolo TEXT CHECK (ruolo IN ('admin', 'editor', 'viewer')) DEFAULT 'viewer'
);

-- Helper function (security definer per bypassare RLS internamente)
CREATE OR REPLACE FUNCTION get_user_role()
RETURNS TEXT
LANGUAGE SQL SECURITY DEFINER STABLE
AS $$
  SELECT ruolo FROM profili WHERE id = auth.uid();
$$;

-- Policy: solo admin cancella
CREATE POLICY "admin_cancellano"
ON articoli FOR DELETE
USING (get_user_role() = 'admin');

-- Policy: editor e admin modificano
CREATE POLICY "editor_admin_modificano"
ON articoli FOR UPDATE
USING (get_user_role() IN ('admin', 'editor'));
```

## RLS — Pattern multi-tenant

```sql
CREATE TABLE organizzazioni (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  nome TEXT NOT NULL,
  slug TEXT UNIQUE NOT NULL
);

CREATE TABLE membri_organizzazione (
  organizzazione_id UUID REFERENCES organizzazioni(id) ON DELETE CASCADE,
  utente_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
  ruolo TEXT CHECK (ruolo IN ('owner', 'admin', 'member')),
  PRIMARY KEY (organizzazione_id, utente_id)
);

-- Helper function
CREATE OR REPLACE FUNCTION mie_organizzazioni()
RETURNS SETOF UUID
LANGUAGE SQL SECURITY DEFINER STABLE
AS $$
  SELECT organizzazione_id
  FROM membri_organizzazione
  WHERE utente_id = auth.uid();
$$;

-- Ogni tabella business ha organizzazione_id
CREATE TABLE progetti_org (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  organizzazione_id UUID NOT NULL REFERENCES organizzazioni(id),
  nome TEXT NOT NULL
);

CREATE INDEX ON progetti_org (organizzazione_id);
ALTER TABLE progetti_org ENABLE ROW LEVEL SECURITY;

CREATE POLICY "membri_vedono_progetti_org"
ON progetti_org FOR SELECT
USING (organizzazione_id IN (SELECT mie_organizzazioni()));

CREATE POLICY "membri_creano_progetti_org"
ON progetti_org FOR INSERT
WITH CHECK (organizzazione_id IN (SELECT mie_organizzazioni()));
```

## STORAGE — Bucket privati

```sql
-- Crea bucket privato
INSERT INTO storage.buckets (id, name, public)
VALUES ('documenti-utenti', 'documenti-utenti', false);

-- Policy SELECT: solo file nel proprio path
CREATE POLICY "utenti_leggono_propri_file"
ON storage.objects FOR SELECT
USING (
  bucket_id = 'documenti-utenti'
  AND (storage.foldername(name))[1] = auth.uid()::text
);

-- Policy INSERT: upload solo nel proprio path
CREATE POLICY "utenti_caricano_propri_file"
ON storage.objects FOR INSERT
WITH CHECK (
  bucket_id = 'documenti-utenti'
  AND (storage.foldername(name))[1] = auth.uid()::text
);

-- Policy DELETE: solo propri file
CREATE POLICY "utenti_cancellano_propri_file"
ON storage.objects FOR DELETE
USING (
  bucket_id = 'documenti-utenti'
  AND (storage.foldername(name))[1] = auth.uid()::text
);
```

### Upload client-side

```tsx
'use client'
import { createClient } from '@/lib/supabase/client'

export function UploadForm() {
  async function upload(file: File) {
    const supabase = createClient()
    const { data: { user } } = await supabase.auth.getUser()
    if (!user) return

    const path = `${user.id}/${Date.now()}_${file.name}`
    const { data, error } = await supabase.storage
      .from('documenti-utenti')
      .upload(path, file)

    if (error) console.error(error)
    return data
  }
  // ...
}
```

### Signed URL per download temporaneo

```ts
const { data } = await supabase.storage
  .from('documenti-utenti')
  .createSignedUrl(path, 60) // 60 secondi

return data?.signedUrl
```

## REALTIME — Subscription

```tsx
'use client'
import { useEffect, useState } from 'react'
import { createClient } from '@/lib/supabase/client'

export function MessaggiList({ chatId }: { chatId: string }) {
  const [messaggi, setMessaggi] = useState([])

  useEffect(() => {
    const supabase = createClient()

    // Carica iniziali
    supabase
      .from('messaggi')
      .select('*')
      .eq('chat_id', chatId)
      .order('created_at')
      .then(({ data }) => setMessaggi(data ?? []))

    // Subscribe
    const channel = supabase
      .channel(`chat:${chatId}`)
      .on('postgres_changes', {
        event: '*',
        schema: 'public',
        table: 'messaggi',
        filter: `chat_id=eq.${chatId}`,
      }, (payload) => {
        if (payload.eventType === 'INSERT') {
          setMessaggi(prev => [...prev, payload.new])
        } else if (payload.eventType === 'UPDATE') {
          setMessaggi(prev => prev.map(m => m.id === payload.new.id ? payload.new : m))
        } else if (payload.eventType === 'DELETE') {
          setMessaggi(prev => prev.filter(m => m.id !== payload.old.id))
        }
      })
      .subscribe()

    return () => { supabase.removeChannel(channel) }
  }, [chatId])

  return <ul>{messaggi.map(m => <li key={m.id}>{m.testo}</li>)}</ul>
}
```

## EDGE FUNCTIONS

```bash
npx supabase functions new invia-email
```

`supabase/functions/invia-email/index.ts`:
```ts
import { serve } from 'https://deno.land/std@0.208.0/http/server.ts'

serve(async (req) => {
  const authHeader = req.headers.get('Authorization')
  if (!authHeader) return new Response('Unauthorized', { status: 401 })

  const { to, subject, body } = await req.json()

  const response = await fetch('https://api.resend.com/emails', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${Deno.env.get('RESEND_API_KEY')}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({ from: 'noreply@faro.example', to, subject, html: body }),
  })

  return new Response(JSON.stringify({ ok: response.ok }), {
    headers: { 'Content-Type': 'application/json' },
  })
})
```

Deploy: `npx supabase functions deploy invia-email`

## MIGRATIONS — Workflow

```bash
# Crea migration
npx supabase migration new add_progetti_table

# Edita il file in supabase/migrations/
# Applica in locale
npx supabase db reset

# Applica in remoto
npx supabase db push

# Genera tipi TypeScript
npx supabase gen types typescript --local > types/supabase.ts
```

## TYPED QUERIES

```ts
import type { Database } from '@/types/supabase'
import { createClient } from '@/lib/supabase/server'

type Progetto = Database['public']['Tables']['progetti']['Row']

export async function getProgetti(): Promise<Progetto[]> {
  const supabase = await createClient()
  const { data, error } = await supabase
    .from('progetti')
    .select('*')
    .order('created_at', { ascending: false })

  if (error) throw error
  return data ?? []
}
```

## TEST CROSS-USER — RLS verification

```ts
// tests/rls.test.ts
import { describe, it, expect } from 'vitest'
import { createClient } from '@supabase/supabase-js'

describe('RLS enforcement', () => {
  it('utente A non può leggere progetti di utente B', async () => {
    const clientA = createClient(URL, ANON_KEY)
    await clientA.auth.signInWithPassword({ email: 'a@test', password: 'x' })

    const clientB = createClient(URL, ANON_KEY)
    await clientB.auth.signInWithPassword({ email: 'b@test', password: 'x' })

    // B crea un progetto
    const { data: progetto } = await clientB
      .from('progetti')
      .insert({ nome: 'Segreto B' })
      .select()
      .single()

    // A cerca di leggerlo
    const { data, error } = await clientA
      .from('progetti')
      .select('*')
      .eq('id', progetto!.id)
      .maybeSingle()

    expect(data).toBeNull() // RLS nasconde
  })
})
```
