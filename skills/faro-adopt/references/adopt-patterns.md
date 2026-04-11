# Adopt Patterns — Pattern di deduzione per progetti esistenti

Riferimento operativo per la skill `faro-adopt`. Contiene regole pratiche per estrarre informazioni dal codice esistente senza modificare nulla e generare rules Faro aderenti alla realtà del progetto.

---

## 1. Come dedurre il TIPO di progetto

L'obiettivo è classificare il progetto in uno dei seguenti archetipi: **gestionale**, **SaaS**, **e-commerce**, **landing**, **showcase**, **mobile**, **CLI/bot**, **API/backend**, **ibrido**.

Usa un sistema a punteggio: ogni segnale trovato aggiunge punti a uno o più archetipi. L'archetipo con il punteggio più alto vince. In caso di parità o scarto < 3 punti, chiedi conferma all'utente.

### 1.1 Segnali per GESTIONALE (ERP/CRM/back-office)

| Segnale | Dove cercare | Punti |
|---|---|---|
| Cartelle `admin/`, `dashboard/`, `back-office/` | struttura | +3 |
| Componenti `Sidebar`, `DataTable`, `DataGrid`, `CrudTable` | `components/` | +3 |
| Libreria tabelle: `@tanstack/react-table`, `ag-grid`, `material-table` | `package.json` | +4 |
| Ruoli/permessi: file `roles.ts`, `permissions.ts`, enum `UserRole` | codice | +3 |
| Pattern CRUD ripetuto su N entità (`clienti`, `fornitori`, `commesse`, `ordini`) | routes | +3 |
| Libreria PDF/export: `jspdf`, `exceljs`, `xlsx`, `react-pdf` | deps | +2 |
| Filtri, ricerca globale, paginazione server-side | codice | +2 |
| Nessuna landing pubblica (`/` fa redirect a login) | routes | +2 |

### 1.2 Segnali per SaaS

| Segnale | Dove cercare | Punti |
|---|---|---|
| Stripe: `stripe`, `@stripe/stripe-js`, webhook in `api/webhooks/stripe` | deps + routes | +5 |
| Pagine `pricing`, `billing`, `subscription` | routes | +4 |
| Multi-tenant: `tenant_id`, `organization_id` nello schema DB | schema | +4 |
| Onboarding flow: `onboarding/`, step wizard | routes | +3 |
| Auth library completa: Clerk, Auth.js, Supabase Auth, WorkOS | deps | +3 |
| Invite/team: `invitations`, `members`, `teams` | schema/routes | +3 |
| Usage tracking, plan limits, feature flags | codice | +2 |
| Landing pubblica + dashboard protetta | struttura | +2 |

### 1.3 Segnali per E-COMMERCE

| Segnale | Dove cercare | Punti |
|---|---|---|
| Entità `products`, `orders`, `cart`, `checkout` | schema + routes | +5 |
| Context/store `CartContext`, `useCart` | codice | +4 |
| Gateway pagamento: Stripe Checkout, PayPal, Klarna | deps | +3 |
| Platform headless: Medusa, Saleor, Shopify Storefront, Commerce.js | deps | +5 |
| Catalogo con varianti, inventario, SKU | schema | +3 |
| Pagine `product/[slug]`, `collections/`, `shop/` | routes | +3 |
| Librerie immagini zoom/gallery | deps | +1 |

### 1.4 Segnali per LANDING page

| Segnale | Dove cercare | Punti |
|---|---|---|
| Single page o poche route (`/`, `/privacy`, `/terms`) | routes | +3 |
| Sezioni `Hero`, `Features`, `Testimonials`, `CTA`, `Footer` | components | +4 |
| Nessun DB o solo form contatto | deps | +2 |
| Libreria animazioni pesante: Framer Motion, GSAP, Lenis | deps | +2 |
| Nessun auth, nessun dashboard | struttura | +3 |
| Form newsletter: Mailchimp, Resend, ConvertKit | deps | +2 |
| CMS headless: Contentful, Sanity, Payload per contenuti marketing | deps | +1 |

### 1.5 Segnali per SHOWCASE / portfolio

| Segnale | Dove cercare | Punti |
|---|---|---|
| GSAP, Three.js, Lenis, Spline, R3F | deps | +5 |
| Cartella `projects/`, `works/`, `case-studies/` | struttura | +4 |
| Gallery, grid masonry, immagini pesate | components | +3 |
| MDX per contenuti case study | deps | +2 |
| Font display custom (non Inter/Roboto) | font config | +2 |
| Cursor custom, page transitions | codice | +3 |
| Nessun CRUD, nessun auth | struttura | +2 |

### 1.6 Segnali per MOBILE

| Segnale | Dove cercare | Punti |
|---|---|---|
| `expo`, `react-native`, `@react-native/*` | deps | +10 |
| `app.json`, `eas.json`, `metro.config.js` | root | +5 |
| Cartelle `ios/`, `android/` | root | +5 |
| `expo-router`, `@react-navigation/*` | deps | +3 |
| Librerie native: `expo-camera`, `expo-notifications` | deps | +2 |

### 1.7 Segnali per CLI / BOT

| Segnale | Dove cercare | Punti |
|---|---|---|
| Campo `bin` in `package.json` | package.json | +5 |
| `commander`, `yargs`, `clipanion`, `oclif` | deps | +5 |
| `telegraf`, `node-telegram-bot-api` (Telegram) | deps | +5 |
| `discord.js`, `@discordjs/*` (Discord) | deps | +5 |
| `@slack/bolt` (Slack) | deps | +5 |
| `inquirer`, `prompts`, `enquirer` | deps | +2 |
| Shebang `#!/usr/bin/env node` in entrypoint | src | +3 |
| Nessuna UI web, nessun framework frontend | deps | +3 |

### 1.8 Segnali per API / BACKEND puro

| Segnale | Dove cercare | Punti |
|---|---|---|
| `fastify`, `express`, `hono`, `koa`, `nestjs` senza frontend | deps | +5 |
| Cartella `routes/`, `controllers/`, `handlers/` | struttura | +3 |
| OpenAPI/Swagger: `@fastify/swagger`, `swagger-ui` | deps | +3 |
| Nessuna cartella `pages/` o `app/` o `src/app/` frontend | struttura | +3 |
| Dockerfile che espone solo porta API | root | +2 |

---

## 2. Come estrarre il DESIGN SYSTEM esistente

### 2.1 Da `tailwind.config.js` / `tailwind.config.ts`

Leggi il file e cerca:
- `theme.extend.colors` → palette custom (primary, secondary, accent, surface, ...)
- `theme.extend.fontFamily` → font famiglie (sans, serif, display, mono)
- `theme.extend.spacing` → spacing custom
- `theme.extend.borderRadius` → radius design token
- `theme.extend.screens` → breakpoint custom
- `content` → quali file sono scansionati (utile per capire dove stanno i componenti)
- `darkMode` → `class` / `media` / `selector`
- `plugins` → `@tailwindcss/typography`, `@tailwindcss/forms`, daisyui, ...

Se esiste solo la config di default (nessun `extend`), marca il design come **"tailwind default — da formalizzare"**.

### 2.2 Da `globals.css` / `app.css` / `styles/`

Cerca:
- CSS variables nella sezione `:root { ... }` → token di colore base (`--background`, `--foreground`, `--primary`, ...)
- Variante `.dark { ... }` o `[data-theme="dark"]` → conferma dark mode
- `@import "tailwindcss"` (v4) o `@tailwind base/components/utilities` (v3) → versione Tailwind
- Font `@font-face` custom → font locali
- Classi utility custom → pattern design ricorrenti

Pattern shadcn/ui: presenza di `--background`, `--foreground`, `--card`, `--popover`, `--primary`, `--muted`, `--accent`, `--destructive`, `--border`, `--input`, `--ring` è una firma quasi certa di shadcn.

### 2.3 Da `components/ui/`

| Indizio | Conclusione |
|---|---|
| Esistono `button.tsx`, `input.tsx`, `dialog.tsx`, `card.tsx` con `cva`, `cn`, `class-variance-authority` | shadcn/ui |
| `@radix-ui/*` nelle deps | Radix Primitives (base shadcn o custom) |
| `@headlessui/react` | Headless UI (ecosistema Tailwind UI) |
| `@mui/material`, `@mui/joy` | Material UI |
| `@chakra-ui/react` | Chakra UI |
| `@mantine/core` | Mantine |
| `antd` | Ant Design |
| `daisyui` tra i plugin Tailwind | DaisyUI |
| `@nextui-org/*` | NextUI/HeroUI |

Se nessuna di queste ma esistono componenti custom, marca come **"design system custom interno"** ed estrai i pattern dai file stessi.

### 2.4 Font in uso

- `next/font/google` con `Inter({ ... })` → Google Font via Next
- `next/font/local` → font locale (estrai path del file)
- `<link rel="stylesheet" href="https://fonts.googleapis.com/...">` in `_document.tsx` o `layout.tsx` → Google Font via CDN
- `@font-face` in CSS → font locale vecchia maniera

Se ci sono due font heading diversi in due file, marca come **incoerenza** e chiedi all'utente quale formalizzare.

### 2.5 Deduzione TIER dal codice

| Dipendenze presenti | Tier dedotto |
|---|---|
| Solo Tailwind default, nessuna lib animazione | Tier 1 (minimal) |
| Tailwind custom + shadcn/ui + Lucide icons | Tier 2 (standard) |
| + Framer Motion, CSS complesse, micro-interazioni | Tier 3 (refined) |
| + GSAP, Three.js, Lenis, Spline, R3F, ScrollTrigger | Tier 4 (experiential) |

Avvisa l'utente se il tier dedotto è più alto del necessario per il tipo di progetto (es. gestionale con Three.js = probabile overkill).

---

## 3. Come estrarre l'ARCHITETTURA

### 3.1 Next.js

| Indizio | Conclusione |
|---|---|
| `src/app/` o `app/` con `page.tsx`, `layout.tsx` | App Router |
| `src/pages/` o `pages/` con `index.tsx`, `_app.tsx` | Pages Router |
| Entrambe presenti | Hybrid (frequente in migrazioni) |
| `middleware.ts` in root o `src/` | Middleware attivo |
| `next.config.js` con `output: 'standalone'` | Self-hosted Docker |
| `next.config.js` con `output: 'export'` | Static export |
| Route group `(dashboard)`, `(auth)`, `(marketing)` | Segmentazione logica |
| `@slot` con `@modal/` | Parallel routes / modali |

Versione Next dal campo `next` in `package.json`. Se >= 15, ricorda che `params` e `searchParams` sono async. Se >= 16, `middleware.ts` è rinominato `proxy.ts`.

### 3.2 Altri framework frontend

| File/cartella | Framework |
|---|---|
| `svelte.config.js`, `src/routes/+page.svelte` | SvelteKit |
| `astro.config.mjs`, `src/pages/*.astro` | Astro |
| `nuxt.config.ts`, `pages/*.vue` | Nuxt |
| `vite.config.ts` + `src/App.vue` | Vue + Vite |
| `vite.config.ts` + `src/App.tsx` (senza Next) | React + Vite |
| `remix.config.js` o `app/routes/*.tsx` | Remix |
| `qwik.config.ts` | Qwik |
| `solid-js` in deps | SolidJS |

### 3.3 Monorepo

| File | Tool |
|---|---|
| `turbo.json` | Turborepo |
| `nx.json` | Nx |
| `pnpm-workspace.yaml` | pnpm workspaces |
| `lerna.json` | Lerna (legacy) |
| `rush.json` | Rush |
| `package.json` con `workspaces` array | npm/yarn workspaces |

Se monorepo, elenca gli apps (`apps/*`) e i packages (`packages/*`) trovati e chiedi all'utente su quale app/package applicare Faro (o se su tutti).

### 3.4 Backend Python

| File | Framework |
|---|---|
| `main.py` con `from fastapi import FastAPI` | FastAPI |
| `manage.py`, `settings.py`, `wsgi.py` | Django |
| `app.py` con `from flask import Flask` | Flask |
| `pyproject.toml` con `poetry` | Poetry |
| `requirements.txt` + `venv/` | pip tradizionale |
| `uv.lock` | uv |

### 3.5 Deduzione dei moduli dal pattern di cartelle

Se `app/(dashboard)/` contiene `clienti/`, `fornitori/`, `commesse/`, `ordini/`, ognuna con `page.tsx` + subcartelle → sono N moduli CRUD. Elencali esplicitamente nell'analisi.

Se `app/api/` contiene cartelle omonime alle entità DB → conferma del dominio.

---

## 4. Come estrarre il MODELLO DATI

### 4.1 Prisma

File: `prisma/schema.prisma`. Estrai:
- Ogni `model X { ... }` → entità
- Campi principali, tipi, relazioni (`@relation`)
- Enum
- Provider (`postgresql`, `mysql`, `sqlite`, `mongodb`)

### 4.2 Drizzle

File tipici: `src/db/schema.ts`, `drizzle/schema.ts`. Cerca:
- `export const users = pgTable('users', { ... })`
- `relations(users, ({ many, one }) => ({ ... }))`
- Dialetto da `drizzle.config.ts` (`dialect: 'postgresql' | 'mysql' | 'sqlite'`)

### 4.3 Supabase

- `supabase/migrations/*.sql` → leggi l'ultimo file `*_baseline.sql` o concatena le migrazioni
- `supabase/config.toml` → configurazione progetto
- Row Level Security: cerca `create policy` → livello di sicurezza DB

### 4.4 SQL grezzo

- `schema.sql`, `migrations/*.sql`, `db/init.sql`
- Estrai ogni `CREATE TABLE` e le colonne principali

### 4.5 SQLAlchemy + Alembic (Python)

- `models.py`, `models/*.py` con `class X(Base):`
- `alembic/versions/*.py` per lo storico delle migrazioni

### 4.6 MongoDB / Mongoose

- `models/*.ts` con `new Schema({ ... })`
- Nessuno schema rigido: documenta solo le collection usate nel codice

### 4.7 Come estrarre le entità principali

Per ogni entità trovata, cattura:
- Nome (es. `User`, `Order`, `Commessa`)
- Relazioni principali (es. `Order → User`, `Commessa → Cantiere`)
- Campi chiave (id, foreign key, status, createdAt)

Scrivi in `architecture.md` generato un diagramma testuale delle entità trovate, senza aggiungere campi inventati.

---

## 5. Come rilevare la SICUREZZA attuale

### 5.1 Security headers

Cerca in `middleware.ts` / `proxy.ts` / `next.config.js`:
- `Content-Security-Policy`
- `X-Frame-Options`
- `Strict-Transport-Security`
- `X-Content-Type-Options`
- `Referrer-Policy`
- `Permissions-Policy`

Se mancano, segnala come **gap** nel documento `security.md` generato.

### 5.2 Rate limiting

Indicatori:
- `@upstash/ratelimit`, `express-rate-limit`, `hono/rate-limit`
- `slowapi` (Python)
- Cloudflare / Vercel edge rules (fuori dal codice — chiedi all'utente)

### 5.3 Validazione input

- `zod`, `valibot`, `yup`, `joi`, `ajv` nelle deps → validazione presente
- `pydantic` (Python)
- Se assente: gap critico

### 5.4 Password hashing

- `bcrypt`, `bcryptjs`, `argon2`, `@node-rs/argon2` → hashing presente
- `passlib` (Python)
- Se usano auth library gestita (Clerk, Auth.js, Supabase Auth) → hashing gestito dal provider, non è un gap

### 5.5 Auth library

| Dep | Provider |
|---|---|
| `@clerk/nextjs` | Clerk |
| `next-auth`, `@auth/*` | Auth.js (NextAuth v5) |
| `@supabase/supabase-js` + `@supabase/ssr` | Supabase Auth |
| `@workos-inc/node` | WorkOS |
| `lucia`, `better-auth` | Lucia / BetterAuth |
| `firebase-admin` + `firebase/auth` | Firebase Auth |
| Nessuna delle sopra + login custom | Auth custom (alto rischio) |

### 5.6 Segreti

- `.env` committato? (controlla `.gitignore`)
- `.env.example` presente? → buona pratica
- Hard-coded keys nel codice? (grep veloce di pattern tipo `sk_live_`, `AKIA`)

---

## 6. Come rilevare TEST e CODE STYLE

### 6.1 Framework test

| Dep | Framework |
|---|---|
| `vitest` | Vitest (consigliato per Vite/Next) |
| `jest`, `ts-jest` | Jest |
| `@playwright/test` | Playwright (E2E) |
| `cypress` | Cypress (E2E) |
| `mocha`, `chai` | Mocha (legacy) |
| `pytest`, `pytest-*` | Pytest (Python) |
| `@testing-library/react` | React Testing Library (componenti) |
| `msw` | Mock Service Worker |

Cerca script in `package.json`: `"test"`, `"test:e2e"`, `"test:unit"`. Se assenti e framework presente → test non integrati in CI.

### 6.2 TypeScript strict

In `tsconfig.json` cerca:
- `"strict": true` → strict mode attivo
- `"noUncheckedIndexedAccess": true` → livello alto
- `"exactOptionalPropertyTypes": true` → livello massimo
- Se `strict: false` o assente → code style lasco

### 6.3 ESLint / Prettier

- `.eslintrc.*` o `eslint.config.*` → estrai `extends` per capire i preset (`next/core-web-vitals`, `airbnb`, `standard`)
- `.prettierrc*` → stile formattazione
- `biome.json` → Biome (alternativa moderna)

### 6.4 Git hooks

- `.husky/` → Husky presente
- `lint-staged` in `package.json` → lint-staged
- `simple-git-hooks` → alternativa leggera
- `commitlint.config.js` → convenzione commit

Se presenti → il team ha disciplina di commit. Faro userà le stesse convenzioni.

---

## 7. Mappatura TECH TROVATA → LIVELLO SUPPORTO FARO

Faro distingue tre livelli di supporto per le tecnologie:

- **Tier 1 (first-class)**: rules complete, esempi pronti, pattern testati
- **Tier 2 (supportato)**: rules generiche, pattern derivati da Tier 1
- **Tier 3 (best-effort)**: documentate, ma le rules sono fornite come "principi" non come codice specifico

### 7.1 Tabella Tier

| Tecnologia | Tier |
|---|---|
| Next.js (App Router, 14+) | 1 |
| React + Vite | 1 |
| TypeScript | 1 |
| Tailwind CSS | 1 |
| shadcn/ui + Radix | 1 |
| Prisma | 1 |
| Supabase | 1 |
| Postgres | 1 |
| Auth.js / Clerk | 1 |
| Stripe | 1 |
| Vercel (deploy) | 1 |
| Next.js Pages Router | 2 |
| SvelteKit | 2 |
| Astro | 2 |
| Nuxt 3 | 2 |
| Drizzle ORM | 2 |
| FastAPI | 2 |
| Django | 2 |
| Expo / React Native | 2 |
| MongoDB / Mongoose | 2 |
| Railway / Render / Fly.io | 2 |
| Remix | 3 |
| Qwik / SolidJS | 3 |
| Flask | 3 |
| Rails | 3 |
| Laravel | 3 |
| MySQL (vs Postgres) | 3 |
| Firebase (completo) | 3 |
| Heroku | 3 |
| Self-hosted VPS custom | 3 |

### 7.2 Quando avvisare l'utente

- Se la stack primaria è **Tier 3**, avvisa esplicitamente: "Faro supporta questo stack in modalità best-effort. Le rules generate saranno principi generici, non pattern specifici del framework."
- Se c'è un mix Tier 1 + Tier 3 (es. Next.js + un CMS Tier 3), genera rules complete per la parte Tier 1 e principi per la parte Tier 3.

---

## 8. PRECAUZIONI — adopt non deve rompere nulla

Queste sono regole invalicabili della skill `faro-adopt`.

### 8.1 Zero modifiche al codice durante l'analisi

- **MAI** editare un file del progetto durante lo Step 1 o Step 2
- **MAI** rinominare file, spostare cartelle, riformattare codice
- Usa esclusivamente tool di lettura (`Read`, `Glob`, `Grep`) e `ls`
- Gli unici file creati sono in `.claude/rules/` e sono generati dallo Step 5

### 8.2 Zero forzature dal wizard

- Il wizard (`faro-init`) propone una configurazione "pulita". `faro-adopt` **non** deve imporre quella configurazione al progetto esistente.
- Se il progetto esistente usa MUI e il wizard consiglia shadcn, `faro-adopt` **rispetta** MUI e genera rules MUI-compatibili.
- Solo se l'utente chiede esplicitamente "voglio uniformare/migrare", allora proponi il cambio.

### 8.3 Rules descrittive, non prescrittive

- In modalità adopt, `design-system.md` inizia con: "Il progetto attualmente usa..."
- `architecture.md` inizia con: "La struttura corrente è organizzata come..."
- `security.md` elenca cosa c'è e cosa manca, senza riscrivere il middleware
- Solo quando l'utente chiede "uniforma / normalizza / consolida", le rules diventano prescrittive

### 8.4 Gestione dei conflitti e delle incoerenze

Il codice esistente può contenere pattern contraddittori. Gestione:

| Conflitto | Azione |
|---|---|
| Due font heading diversi (`Inter` in un file, `Poppins` in un altro) | Elenca entrambi nell'analisi, chiedi quale formalizzare |
| Tailwind v3 + Tailwind v4 syntax mix | Segnala migrazione in corso, documenta stato |
| Componenti shadcn + componenti MUI nello stesso progetto | Segnala ibrido, chiedi all'utente la direzione |
| File con `"use client"` async (bug RSC) | Segnala come **gap**, non correggere |
| Schema Prisma con modelli non usati | Elenca tutti, chiedi se deprecare |
| Routes con `params` sincrono su Next 15+ | Segnala come **gap tecnico**, non correggere |
| Mix tabs/spaces | Documenta lo stile dominante, non toccare |
| Env vars nel codice invece che in `.env` | Segnala come **rischio sicurezza** |

**Regola d'oro**: in caso di dubbio, l'analisi elenca, l'utente decide. Faro non deve mai sostituirsi al team nel prendere decisioni su codice pre-esistente.

### 8.5 Output conservativo

Le rules generate in modalità `adopt` devono:
- Avere un header che dichiara "Generato da faro-adopt il [data] — basato sul codice esistente"
- Avere una sezione finale `## Note di adopt` con i gap e le incoerenze rilevate
- Non contenere esempi di codice che contraddicono lo stile del progetto (se il progetto usa `function` components, non mettere esempi con `const X = () =>`)
- Essere rigenerabili: l'utente può rilanciare `faro-adopt` in futuro per aggiornare le rules senza perdere modifiche manuali (segnare i blocchi auto-generati con marker `<!-- faro:auto-start -->` / `<!-- faro:auto-end -->`)

---

## 9. Riepilogo del flusso adopt

1. **Silenzioso** → leggi tutto, non dire nulla finché non hai un quadro
2. **Presenta** → mostra l'analisi in forma tabellare, senza opinioni
3. **Chiedi** → max 3 domande, solo su lacune reali
4. **Proponi** → proposta basata su dati reali, non su template
5. **Genera** → invoca `faro-generator` con `mode: adopt`, output descrittivo
6. **Chiudi** → messaggio finale con elenco dei file creati e dei gap segnalati

Il successo di `faro-adopt` si misura così: l'utente apre le rules generate e dice **"sì, questo è davvero il mio progetto"**, non "questo è un template che qualcuno ha inventato".
