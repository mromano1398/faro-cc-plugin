# Template Rules — 11 file

## Nota sul tono (wizard vs adopt)

I template seguenti sono scritti in tono PRESCRITTIVO (usa X, vieta Y) adatto a wizard/new.
Quando mode=adopt, il generator DEVE trasformare il tono in DESCRITTIVO:
- "Usa colori X" → "Il progetto usa colori X (estratti dal codice)"
- "Vietato Y" → "Il codice attuale non usa Y"
- "Font heading: Clash Display" → "Font heading in uso: [quello estratto]"

In adopt, le rules DESCRIVONO ciò che c'è. Possono SEGNALARE lacune (es: "manca rate limiting su login")
ma non prescrivono decisioni architetturali già prese.

## Nota sul filtro per tipo progetto

Alcune rules contengono regole specifiche per web (es: "usa next/image", "kebab-case per file .tsx")
che NON si applicano a tutti i tipi di progetto. Il generator DEVE filtrare per tipo:

| Rule | Gestionale | SaaS | E-com | Landing | Showcase | Mobile | CLI/bot | Python |
|------|------------|------|-------|---------|----------|--------|---------|--------|
| design-system.md | sì | sì | sì | sì | sì | adatta | no | adatta |
| next/image, app router | sì | sì | sì | sì | sì | no | no | no |
| kebab-case .tsx | sì | sì | sì | sì | sì | sì | no | no |
| snake_case .py | no | no | no | no | no | no | no | sì |
| React hooks rules | sì | sì | sì | sì | sì | sì | no | no |
| Expo Router conventions | no | no | no | no | no | sì | no | no |

Il generator: se una rule contiene dettagli specifici per un tipo non attivo, li SOSTITUISCE con
equivalenti per il tipo del progetto, oppure li RIMUOVE se non applicabili.

## 1. design-system.md

```markdown
# Design System — [NOME_PROGETTO]

## Tier: [N] — [NOME_TIER]
Questo progetto usa il Tier [N]. Ogni componente e animazione deve rispettare le regole del tier.
Tier scelti:
- **1 — FUNZIONALE** — zero animazioni, shadcn base, desktop-first
- **2 — PROFESSIONALE** — Framer Motion, skeleton, empty state
- **3 — CINEMATIC** — GSAP, Lenis, dark theme, font display
- **4 — IMMERSIVO** — Three.js, 3D, .glb models

## Palette
- Primary: [COLORE_PRIMARY] — usato per CTA, link, accent
- Primary hover: [COLORE_PRIMARY_HOVER]
- Background: [COLORE_BG]
- Surface: [COLORE_SURFACE] — card, dialog
- Border: [COLORE_BORDER]
- Text: [COLORE_TEXT]
- Text muted: [COLORE_TEXT_MUTED]
- Success: [COLORE_SUCCESS]
- Warning: [COLORE_WARNING]
- Error: [COLORE_ERROR]

## Tipografia
- Font heading: [FONT_HEADING] / peso [PESO_HEADING]
- Font body: [FONT_BODY] / peso [PESO_BODY]
- Font mono: [FONT_MONO] (per codice, numeri, input)
- Scale: text-xs (12px), text-sm (14px), text-base (16px), text-lg (18px), text-xl (20px), text-2xl (24px), text-3xl (30px), text-4xl (36px)

## Spacing
- Padding card: p-4 (mobile) p-6 (desktop)
- Gap griglia: gap-4 (mobile) gap-6 (desktop)
- Sezione: py-16 (mobile) py-24 (desktop)

## Bordi
- Raggio default: rounded-[RAGGIO] (rounded-md per Tier 1-2, rounded-lg per Tier 3-4)
- Bordo default: border border-[COLORE_BORDER]
- Focus ring: ring-2 ring-[PRIMARY] ring-offset-2

## Componenti standard (Tier [N])
[BLOCCO_COMPONENTI_TIER]
Il generator inlinea qui il blocco esatto del tier scelto (vedi sotto per i 4 blocchi).

<!-- Blocchi disponibili, il generator copia solo quello del tier attivo -->

### Tier 1 (FUNZIONALE)
- Button, Input, Select, Textarea, Card, Badge, Table, Dialog, Toast, Form, Tabs
- Breadcrumb, Sidebar, Command (Cmd+K), Calendar, DatePicker
- Zero animazioni decorative. Max: fade toast/dialog.

### Tier 2 (PROFESSIONALE)
- Include tutto Tier 1 +
- Skeleton, EmptyState, Avatar, AvatarGroup, HoverCard, Progress, Accordion, Sheet
- Animazioni: fade in 200-300ms, layout animation Framer Motion, shimmer skeleton, toast slide-in
- Vietato: parallax, scroll-trigger, text reveal, animazioni >300ms

### Tier 3 (CINEMATIC)
- Include tutto Tier 1-2 +
- ScrollReveal, TextReveal, ParallaxImage, PageTransition, Navbar trasparente→solida, HeroSection 100vh, SmoothScroll
- Stack: GSAP + ScrollTrigger + Lenis + Framer Motion
- Font: MAI Inter/Roboto come heading. Usare Clash Display, Satoshi, Space Grotesk, ecc.
- Dark theme default, accent vibrante

### Tier 4 (IMMERSIVO)
- Include tutto Tier 1-2-3 +
- Scene3D (R3F Canvas), Model3D (useGLTF), LoadingScreen (useProgress), ProductConfigurator, ShowroomVirtual
- Fallback 2D OBBLIGATORIO su ogni componente 3D
- Stack: three + @react-three/fiber + @react-three/drei
- Performance: max 5MB .glb, dpr={[1,2]}, lazy load Canvas

## Animazioni (Tier [N])
[BLOCCO_ANIMAZIONI_TIER]
Il generator inlinea qui le regole di animazione del tier scelto:

- **Tier 1**: zero animazioni decorative. Max fade su toast/dialog.
- **Tier 2**: fade in 200-300ms, layout animation Framer Motion, shimmer skeleton, toast slide-in. Vietato >300ms, parallax, scroll-trigger.
- **Tier 3**: GSAP + ScrollTrigger + Lenis smooth scroll, text reveal, parallax image, page transition, navbar trasparente→solida.
- **Tier 4**: tutto Tier 3 + animazioni 3D (R3F), camera movement, model interaction. Rispetto `prefers-reduced-motion` obbligatorio.

## Dark mode
[SE ATTIVO: colori dark mode]

## Responsive breakpoint
- sm: 640px, md: 768px, lg: 1024px, xl: 1280px, 2xl: 1536px
- Approccio: [mobile-first | desktop-first] in base al tipo

## Regole assolute
- USA SOLO i colori definiti sopra — mai colori hardcoded fuori palette
- USA SOLO i font definiti — mai font non dichiarati
- Ogni componente DEVE usare le variabili di design — non valori arbitrari
- [REGOLE SPECIFICHE PER TIER]
```

## 2. security.md

```markdown
# Sicurezza — [NOME_PROGETTO]

## Livello 1 — Setup (SEMPRE)
- Security headers nel middleware:
  X-Content-Type-Options: nosniff
  X-Frame-Options: DENY
  X-XSS-Protection: 1; mode=block
  Referrer-Policy: strict-origin-when-cross-origin
  [CSP se Tier 3-4 con 3D]
- HTTPS obbligatorio in produzione
- `.env` in `.gitignore` — MAI committare segreti
- `.env.example` con chiavi vuote

## Livello 2 — Auth (se il progetto ha login)
- Rate limiting login: max 5 tentativi in 15 minuti
- Password: bcrypt con salt rounds >= 10
- Session: httpOnly, secure, sameSite: 'lax'
- CSRF: token su form mutation o header custom
- Validazione Zod su OGNI input (form, query params, body API)
- MAI fidarsi di dati dal client senza validazione server

## Livello 3 — Dati (se il progetto ha dati utente)
- WHERE user_id su OGNI query che tocca dati utente (anti-IDOR)
- File upload: validazione MIME type + dimensione max
- Audit log su operazioni critiche (delete, update ruolo, export)
- Transaction su operazioni multi-step

## Sicurezza per tipo
[SEZIONE SPECIFICA PER TIPO — es: RBAC per gestionale, tenant isolation per SaaS]

## Regole assolute
- MAI SQL raw con interpolazione — usa query parametriche
- MAI esporre stack trace in produzione
- MAI log di password, token, dati sensibili
- Validazione Zod su OGNI endpoint, OGNI form — NESSUNA eccezione
```

## 3. code-style.md

~~~markdown
# Code Style — [NOME_PROGETTO]

## TypeScript
- Strict mode: true (in tsconfig.json)
- No `any` — usa tipi precisi o `unknown` + type guard
- Interface per oggetti, Type per union/intersection
- Enum: preferisci `as const` objects

## Naming
- File: kebab-case (user-profile.tsx)
- Componenti: PascalCase (UserProfile)
- Funzioni/variabili: camelCase (getUserProfile)
- Costanti: UPPER_SNAKE_CASE (MAX_RETRY_COUNT)
- Hook: useNome (useUserProfile)
- Validazione: nomeSchema (userSchema)
- Tipo/Interface: PascalCase (UserProfile, CreateUserInput)

## Import
Ordine: 1) React/framework, 2) librerie esterne, 3) @/ alias interni, 4) relativi
Usa `@/` alias per import da src/

## Pattern
- Server Components default (Next.js) — "use client" solo dove serve
- Colocation: componente + tipo + validazione nella stessa cartella
- Early return per ridurre nesting
- Custom hook per logica riutilizzabile
- Zod schema come single source of truth per tipi e validazione

## Git
- Conventional commits: feat:, fix:, chore:, refactor:, docs:, test:
- Branch: feature/nome, fix/nome, refactor/nome
- Commit atomici — un commit = un cambiamento logico
- MAI committare .env, node_modules, build output

## Lint
- ESLint con config Next.js (o equivalente)
- Prettier per formatting
- husky + lint-staged per pre-commit check

## Dove vanno le cose

Convenzioni di directory per il progetto (adatta se il blueprint è diverso):

| Tipo file | Directory | Esempio |
|-----------|-----------|---------|
| Pagine | `src/app/<route>/page.tsx` | `src/app/prodotti/page.tsx` |
| Layout | `src/app/<segment>/layout.tsx` | `src/app/(dashboard)/layout.tsx` |
| API route | `src/app/api/<resource>/route.ts` | `src/app/api/prodotti/route.ts` |
| Server actions | `src/lib/<modulo>/actions.ts` | `src/lib/ddt/actions.ts` |
| Query DB | `src/lib/<modulo>/queries.ts` | `src/lib/ddt/queries.ts` |
| Validazioni Zod | `src/lib/validations/<modulo>.ts` | `src/lib/validations/ddt.ts` |
| Tipi condivisi | `src/types/<dominio>.ts` | `src/types/ddt.ts` |
| Componenti UI riusabili | `src/components/ui/` | `src/components/ui/button.tsx` |
| Componenti condivisi app | `src/components/shared/` | `src/components/shared/navbar.tsx` |
| Componenti feature | `src/components/<modulo>/` | `src/components/ddt/ddt-row.tsx` |
| Filtri riusabili | `src/components/filters/` | `src/components/filters/date-range.tsx` |
| Hook custom | `src/hooks/` | `src/hooks/use-debounce.ts` |
| Utility puro | `src/lib/utils.ts` | — |
| Costanti | `src/lib/constants.ts` | — |

Regola: se un nuovo file non trova posto naturale nella tabella → chiedi all'utente prima di creare
una nuova directory.

## Pattern form standard

Per form in App Router usa **una** di queste 2 strategie, NON mescolarle:

### Strategia A: Server Actions + useActionState (preferita)
```tsx
// src/lib/ddt/actions.ts
"use server";
import { z } from "zod";
import { ddtSchema } from "@/lib/validations/ddt";

export async function createDdt(prevState: any, formData: FormData) {
  const parsed = ddtSchema.safeParse(Object.fromEntries(formData));
  if (!parsed.success) return { error: parsed.error.flatten() };
  // ... create
  return { success: true };
}

// src/app/(dashboard)/ddt/nuovo/page.tsx
"use client";
import { useActionState } from "react";
import { createDdt } from "@/lib/ddt/actions";

export default function NewDdtPage() {
  const [state, formAction] = useActionState(createDdt, null);
  return <form action={formAction}>...</form>;
}
```

### Strategia B: react-hook-form + Zod resolver
```tsx
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { ddtSchema } from "@/lib/validations/ddt";

const form = useForm({ resolver: zodResolver(ddtSchema) });
```
Usa questa strategia se hai form molto dinamici (field array, validazione live complessa).

### Regole
- Lo schema Zod è la SINGLE SOURCE OF TRUTH dei tipi e della validazione
- Il tipo `z.infer<typeof schema>` è il tipo del form data
- La validazione gira sempre server-side (anche se il client valida)
- Errori: mostra sotto il campo con aria-describedby
~~~

## 4. api-conventions.md

~~~markdown
# API Conventions — [NOME_PROGETTO]

## Formato risposta standard
```json
// Successo
{ "data": { ... }, "meta": { "page": 1, "total": 100 } }

// Errore
{ "error": { "code": "VALIDATION_ERROR", "message": "...", "details": [...] } }
```

## Status code
- 200: OK
- 201: Created
- 400: Bad Request (validazione fallita)
- 401: Unauthorized (non autenticato)
- 403: Forbidden (non autorizzato)
- 404: Not Found
- 409: Conflict (duplicato)
- 422: Unprocessable Entity
- 429: Too Many Requests (rate limit)
- 500: Internal Server Error

## Paginazione
Query params: ?page=1&limit=20&sort=created_at&order=desc
Risposta meta: { page, limit, total, totalPages }
Limit max: 100

## Rate limiting
- API pubbliche: 60 req/min
- API autenticate: 120 req/min
- Login: 5 req/15min per IP
- Header: X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset

## Validazione
Zod su OGNI endpoint. Schema definiti in lib/validations/[risorsa].ts
~~~

## 5. architecture.md

```markdown
# Architettura — [NOME_PROGETTO]

## Stack
[LISTA STACK COMPLETA]

## Struttura cartelle
[DA BLUEPRINT — struttura reale del progetto]

## Moduli
Per ogni modulo elenca in questo formato (aggiornato da context-updater):

### [nome-modulo]
- **Scopo:** [1 riga — cosa fa il modulo]
- **File principali:** [src/app/.../page.tsx, src/lib/.../actions.ts, src/components/.../]
- **Tabelle DB:** [tab1, tab2, ...]
- **Relazioni:** [dipende da: modulo-X; usato da: modulo-Y]
- **Stato:** [lista stati possibili con transizioni, se applicabile]

Esempio per un gestionale:

### ddt
- **Scopo:** Documenti di trasporto (creazione, stampa, annullamento)
- **File principali:** src/app/(dashboard)/ddt/, src/lib/ddt/actions.ts, src/lib/validations/ddt.ts
- **Tabelle DB:** ddt, ddt_righe, ddt_annullamenti
- **Relazioni:** usa bobine; usato da trasferimenti
- **Stato:** bozza -> confermato -> annullato (annullamento ripristina stato_precedente)

### bobine
- **Scopo:** Gestione bobine cavo (disponibilità, assegnazione)
- **File principali:** src/app/(dashboard)/bobine/, src/lib/bobine/actions.ts
- **Tabelle DB:** bobine, bobine_movimenti
- **Relazioni:** assegnate a ddt e trasferimenti
- **Stato:** disponibile -> assegnata -> esaurita

Aggiornato da context-updater dopo ogni piano completato.

## Route / Pagine
[LISTA ROUTE]
Aggiornato automaticamente.

## Schema database
[TABELLE PRINCIPALI con campi chiave]
Aggiornato automaticamente.

## Componenti condivisi
[LISTA COMPONENTI in components/ui/ e components/shared/]

## Dipendenze esterne
[SERVIZI TERZI: DB, auth, email, storage, deploy]
```

## 6. accessibility.md

```markdown
# Accessibilità — [NOME_PROGETTO]

## Standard: WCAG 2.1 AA

## Regole
- Contrasto testo: ratio >= 4.5:1 (normal), >= 3:1 (large)
- Focus visibile su TUTTI gli elementi interattivi (ring-2)
- Navigazione completa da tastiera (Tab, Enter, Escape, Arrow)
- ARIA label su elementi senza testo visibile (icone, immagini decorative)
- Alt text su TUTTE le immagini informative (alt="" per decorative)
- Form: label associato a ogni input, errori annunciati con aria-live
- Heading gerarchici: h1 -> h2 -> h3 (mai saltare livelli)
- prefers-reduced-motion: disabilita animazioni non essenziali
- prefers-color-scheme: supporta dark mode se presente
- Screen reader: testare con VoiceOver (Mac) o NVDA (Windows)

## Target
Lighthouse Accessibility: >= 90
```

## 7. testing.md

~~~markdown
# Testing — [NOME_PROGETTO]

## Framework
[Vitest | Jest] per unit test
[Playwright | Cypress] per E2E (se necessario)

## Cosa testare
- Utility functions: SEMPRE
- Validazione Zod schema: SEMPRE
- API route handler: logica business
- Componenti critici: form, checkout, auth flow
- Hook custom con logica complessa

## Cosa NON testare
- Componenti UI puri senza logica
- Wrapper di librerie esterne
- Styling/layout

## Pattern
```ts
describe('nomeModulo', () => {
  it('dovrebbe [comportamento atteso]', () => {
    // Arrange
    // Act
    // Assert
  });
});
```

## Modalità TDD (default: consigliato)

### TDD consigliato (default)
Scrivi test dopo il codice. Ogni feature critica deve avere test.

### TDD forzato (attivabile)
Per attivare: aggiungi `TDD: forzato` in CLAUDE.md.
Quando attivo:
1. PRIMA scrivi il test -> eseguilo -> DEVE fallire (RED)
2. Scrivi il codice MINIMO per far passare il test (GREEN)
3. Refactora mantenendo il test verde (REFACTOR)
4. Se scrivi codice senza test fallente -> STOP, scrivi prima il test

## Build check
Dopo ogni 3 file: `npm run build && npx tsc --noEmit`
~~~

## 8. product-guidance.md

```markdown
# Product Guidance — [NOME_PROGETTO]

## Tipo: [TIPO_PROGETTO]

## Pattern pagine standard per tipo

### SaaS
- **Landing (/ o /home):** Hero + Features + Social proof + Pricing + FAQ + CTA
- **Pricing (/pricing):** 2-3 piani con feature comparate + FAQ + testimonial
- **Login / Register (/login, /register):** form minimal + OAuth + link cross
- **Dashboard (/dashboard):** KPI card + widget attività + quick actions
- **Settings (/settings):** sezioni Profilo / Account / Notifiche / Billing / Team / Security / Danger Zone
- **Billing (/settings/billing):** piano corrente + storico fatture + metodi pagamento + cancel
- **Team (/settings/team):** lista membri + inviti + ruoli
- **Onboarding (/onboarding):** wizard 3-5 step al primo accesso

### E-commerce
- **Home (/):** hero + categorie + prodotti in evidenza + newsletter
- **Catalogo (/prodotti):** filtri + grid + sort + paginazione
- **Dettaglio prodotto (/prodotti/[slug]):** galleria + info + varianti + CTA + correlati
- **Carrello:** drawer o pagina con edit quantità + costi + checkout CTA
- **Checkout:** step shipping -> payment -> review
- **Account / ordini:** lista ordini + dettaglio + tracking

### Gestionale / ERP
- **Dashboard:** KPI card + tabella attività recenti + widget moduli
- **Lista modulo (/[modulo]):** DataTable + filtri + ricerca + bulk actions + export
- **Creazione (/[modulo]/nuovo):** form con validazione live + sezioni collassabili
- **Dettaglio (/[modulo]/[id]):** tab Info / Documenti / Storico / Azioni
- **Modifica (/[modulo]/[id]/modifica):** stesso form della creazione pre-popolato
- **Impostazioni:** utenti / ruoli / preferenze / integrazioni / audit log

### Landing page
Pagina unica con sezioni: Hero, Social proof, Features, How it works, Testimonials, Pricing, FAQ, CTA finale, Footer.

### Portfolio / Showcase
- **Home:** hero + progetti in evidenza
- **Lavori (/lavori):** griglia progetti
- **Dettaglio progetto (/lavori/[slug]):** galleria + case study + stack + link live
- **Chi sono (/chi-sono):** bio + skill + timeline
- **Contatti:** form + info

### Mobile (Expo)
- **Tabs principali:** Home / [Feature] / Profile (3-5 tab max)
- **Stack drill-down:** lista -> dettaglio -> azioni
- **Modal:** per creazione e azioni contestuali
- **Settings:** come SaaS ma con schermate nested

### CLI / Bot
- **CLI:** main command + subcommands (init/build/deploy) + help + version
- **Bot:** /start /help /[comandi domain] + conversation state

## Regola d'uso
Quando Claude riceve un prompt breve tipo "fai la pagina X" o "aggiungi la sezione Y":
1. Legge questa sezione per il tipo di progetto
2. Usa il pattern come SCHELETRO di partenza
3. Adatta contenuto specifico al progetto
4. Le sezioni non vanno tutte implementate se non hanno senso — solo quelle pertinenti

## Guida proattiva

Alla fine di ogni feature implementata, SE RILEVANTE (non interrompere il flusso),
suggerisci UNA miglioria tra:

### Per SaaS
- Card condivisibili (stile Strava/Spotify Wrapped) per viralità
- Streak / gamification per retention
- Onboarding wizard per ridurre churn
- Upgrade prompt contestuali (non invasivi)
- Referral program con incentivo
- Email di re-engagement per utenti inattivi

### Per E-commerce
- "Solo N rimasti" per urgency
- Social proof: "X persone stanno guardando"
- Cross-sell: "Chi ha comprato questo..."
- Recupero carrello abbandonato (email)
- Wishlist con notifica prezzo
- Review/rating prodotti

### Per Gestionale
- Scorciatoie tastiera per azioni frequenti
- Azioni batch (seleziona + azione)
- Notifiche soglie (magazzino sotto X, scadenze)
- Workflow approvazione multi-step
- Export CSV/PDF dei report
- Dashboard personalizzabile

### Per Landing
- A/B test headline (se possibile)
- Exit intent popup (con moderazione)
- Lead magnet / contenuto gated
- Waitlist con referral (invita e sali in classifica)
- Social proof: contatore iscritti

### Per Mobile
- Share sheet con immagine generata
- Widget home screen
- Deep link per condivisione
- Push notification contestuali (non spam)
- Review prompt dopo N utilizzi

## Formato suggerimento
"💡 Suggerimento prodotto: [suggerimento]. Perché: [motivazione con esempio reale].
Vuoi che lo implementi? [Sì/No/Dopo]"

## Regola
Massimo 1 suggerimento per sessione. MAI interrompere durante l'implementazione.
Solo alla fine di un task completato.
```

## 9. workflow.md

```markdown
# Workflow — [NOME_PROGETTO]

## Prima di scrivere codice: BRAINSTORMING

Quando l'utente chiede una feature complessa (non un fix banale):
1. Ripeti ciò che hai capito in 2-3 frasi
2. Chiedi: "Ho capito bene? Qualcosa da aggiungere?"
3. Se ci sono decisioni architetturali -> presenta 2-3 opzioni con pro/contro
4. Quando l'utente approva -> scrivi il piano

### Checklist post-brainstorming (prima di passare al piano)
Il brainstorming non è concluso finché tutti questi punti hanno una risposta
esplicita. Se un punto manca, torna a chiedere all'utente.

- [ ] **Scope definito**: cosa è IN (elenca), cosa è OUT (elenca)
- [ ] **Dipendenze identificate**: moduli, API, librerie, dati necessari
- [ ] **Edge case considerati** (almeno 3): errori rete, input vuoti, utente non autenticato, dati grandi, concorrenza, ecc.
- [ ] **Alternative valutate** (almeno 2 approcci) — solo su decisioni architetturali
- [ ] **Metriche di successo**: come sapremo che la feature funziona? (numero, comportamento, test)
- [ ] **Rischi principali**: cosa può andare storto e qual è il piano B
- [ ] **Output**: design doc sintetico salvato in `.faro/plans/<nome>-design.md` prima del piano operativo

## Piano prima di implementare

Per feature che toccano più di 3 file, scrivi un piano in `.faro/plans/<nome>.md`.
Il piano deve essere eseguibile da un subagent SENZA contesto aggiuntivo.

### Header obbligatorio
```
# Piano: [nome breve]
Data: [YYYY-MM-DD]
Tipo: [feature | fix | refactor | reskin | chore]
Riferimento: docs/PRD.md -> [RF-001, RF-002, ...]  (o "nessuno" se fuori PRD)
Stato: [bozza | approvato | in esecuzione | completato]
```

### Formato task granulare
Ogni task deve essere 2-5 minuti di lavoro, autocontenuto, con verifica esplicita.
Un task più lungo va spezzato.

```
- [ ] Task N: [titolo breve, imperativo]
  File: path/al/file.ts
  Cosa fare: [descrizione precisa — non generica. Cita funzioni, tipi, righe se serve]
  Verifica: [comando concreto o condizione osservabile — es: "npx tsc --noEmit passa",
             "test X/Y/file.test.ts verde", "pagina /foo renderizza senza errori console"]
  Dipendenze: [Task precedenti richiesti, oppure "nessuna"]
```

### Divieto placeholder
Se un task contiene `[TODO]`, `[???]`, `[da decidere]`, `[ecc]` o descrizioni
vaghe tipo "sistemare" / "migliorare" / "vedi sopra", il piano NON è pronto.
Torna al brainstorming e risolvi l'ambiguità prima di scrivere il task.

### Self-review checklist a fine piano
Prima di chiedere conferma all'utente, rileggi il piano e rispondi a queste
domande. Se anche una sola è "no", il piano va riscritto.

- [ ] Ogni task è eseguibile in 2-5 minuti?
- [ ] Ogni task è autocontenuto (un subagent senza contesto può eseguirlo)?
- [ ] Ogni task ha un comando/condizione di verifica concreta?
- [ ] Le dipendenze tra task sono esplicite?
- [ ] Nessun task contiene placeholder o linguaggio vago?
- [ ] Header compilato con data, tipo, riferimento PRD, stato?
- [ ] Il piano cita solo file che esistono (o che verranno creati in task precedenti)?

### Esecuzione
1. Chiedi conferma del piano all'utente
2. Esegui task per task, segna checkbox appena la verifica passa
3. build-checker dopo ogni 3 file
4. Se un task fallisce -> NON passare al successivo, segnala blocco

## Scope control (anti-scope-creep)

PRIMA di modificare file:
1. LISTA i file che vuoi toccare
2. Chiedi conferma: "Modifico questi file: [lista]. Ok?"
3. Se l'utente chiede X, modifica SOLO ciò che serve per X
4. MAI modificare file non richiesti "perché tanto ci siamo"
5. Se noti qualcosa da fixare in un altro file -> segnala, non fixare
6. Se un file richiesto è un componente condiviso (path: `components/ui/`, `components/shared/`, o chiaramente importato da >=3 altri file), SEGNALA l'impatto PRIMA di modificare:
   "Questo componente è usato in [N] file. Modifiche si propagano a tutti. Vuoi:
    (a) modificare globalmente (tutti i consumer),
    (b) creare una variante dedicata (es: <Component>DDT, <Component>Settings),
    (c) vedere la lista dei consumer prima di decidere?"

## Task paralleli

Quando un piano ha task INDIPENDENTI:
- Identifica quali task non dipendono l'uno dall'altro
- Lancia subagent paralleli per quelli indipendenti
- Aspetta completamento prima di procedere ai task dipendenti

## Git workflow

- Feature branch per feature grandi: `git checkout -b feature/nome`
- Commit atomici con conventional commits
- Prima di merge: build-checker + code-reviewer

### Git worktree — isolamento fisico

Un worktree è una copia del repository su un'altra cartella, agganciata
a un branch diverso. Serve per isolare il lavoro senza rompere il workspace
principale.

**Quando usarlo:**
- Feature grandi che richiedono molti file modificati
- Task eseguiti da subagent in parallelo (ogni subagent nel suo worktree)
- Esecuzione di un piano con task destructive (rename, refactor, migrazioni)
- Prove esplorative che non devono inquinare il branch corrente

**Directory selection**
- Usa `../<repo>-<feature>/` accanto al repo principale
- Mai dentro il repo stesso (confonde git e tooling)

**Safety verification prima di creare il worktree**
- `git status` deve essere clean (no modifiche unstaged che rischi di perdere)
- `git check-ignore <percorso>` per non finire in zone ignorate
- Baseline: il branch di partenza deve buildare (`npm run build` o equivalente)

**Comandi base**
```bash
# Creazione
git worktree add ../<repo>-<feature> -b feature/<nome>

# Lista
git worktree list

# Rimozione dopo merge/cleanup
git worktree remove ../<repo>-<feature>
git branch -d feature/<nome>   # solo se già merged
```

**Regola**: il worktree è REQUIRED prima di subagent-driven execution di task
destructive o di un piano che tocca più di 10 file. Per task piccoli o fix
isolati il worktree è opzionale.

## Context awareness — gestione finestra di contesto

Il contesto è una risorsa finita. Monitorarlo evita allucinazioni e
dimenticanze a metà piano.

**Segnali di contesto pieno (70%+)**
- Le risposte diventano meno dettagliate o saltano dettagli chiesti prima
- Dimentichi decisioni prese nei primi messaggi della sessione
- Ripeti letture di file già letti in questa sessione
- Perdi traccia dei task completati vs pendenti

**Prima di proporre /compact**
1. Aggiorna `.faro/state.md` con: fase attuale, piano attivo, ultimo task completato, recovery point
2. Appendi log in `.faro/log.md` con stato corrente
3. Proponi all'utente: "Il contesto è pieno (~N%). Propongo /compact per continuare senza perdere dettagli. Continuare?"

**Dopo /compact**
1. Rileggi `.faro/state.md` per ricostruire dove eri
2. Rileggi le rules rilevanti per il task corrente (non tutte)
3. Riprendi dal recovery point, non dal task 1

## Context update

Dopo ogni piano COMPLETATO:
- Lancia context-updater per aggiornare architecture.md e CLAUDE.md

## Fine fase — auto-audit

Alla fine di una fase della roadmap (in particolare Fase 2 — Costruzione feature),
NON passare alla fase successiva senza audit.

**Passi obbligatori**
1. Lancia `/project:audit` automaticamente
2. Leggi il report. Se audit design = AI SLOP -> crea automaticamente un piano
   di fix in `.faro/plans/fix-design-slop.md`, chiedi conferma, eseguilo prima
   di proseguire
3. Se audit sicurezza contiene issue CRITICHE -> crea piano di fix obbligatorio
   in `.faro/plans/fix-security-critical.md`, eseguilo prima di proseguire
4. Solo con audit PASS (o con i piani di fix eseguiti e verificati) puoi
   segnare la fase come completata e passare alla successiva
```

## 10. debugging.md

```markdown
# Debugging — [NOME_PROGETTO]

## Quando trovi un bug, NON fixarlo immediatamente. Segui questi step:

1. **RIPRODUCI** — Conferma il bug. Se non riesci a riprodurlo, non puoi fixarlo.
2. **ISOLA** — Dove si manifesta? Frontend? Backend? DB? Network?
3. **ROOT CAUSE** — Perché succede? Non fermarti al sintomo.
   Chiediti 5 volte "perché?" (tecnica dei 5 perché):
   - Il form non salva -> perché? L'API ritorna 500
   - L'API ritorna 500 -> perché? La query fallisce
   - La query fallisce -> perché? Il campo è NULL
   - Il campo è NULL -> perché? La validazione non lo controlla
   - La validazione non lo controlla -> perché? Lo schema Zod non ha quel campo
   -> ROOT CAUSE: schema Zod incompleto
4. **FIX** — Correggi la root cause, non il sintomo
5. **VERIFICA** — Il bug è risolto? Non hai rotto altro? Build check.
6. **PREVIENI** — Aggiungi test per questo caso. Aggiorna regole se necessario.

## Dopo 3 tentativi falliti sullo stesso bug

STOP. Mostra all'utente:
- Cosa hai provato (3 approcci)
- Cosa hai scoperto
- Possibili cause rimaste
- Chiedi input prima di procedere

## Log del debugging
Appendi in .faro/log.md ogni bug significativo con root cause trovata.
```

## 11. prd-alignment.md

```markdown
# PRD Alignment — [NOME_PROGETTO]

Ogni feature implementata deve allinearsi al PRD in `docs/PRD.md`.

## Regole

1. Prima di iniziare una feature, verifica che sia coperta dal PRD (o da un'estensione approvata).
2. Se una richiesta utente esce dallo scope del PRD, segnala l'allineamento mancante PRIMA di implementare.
3. Quando lo scope cambia ufficialmente, aggiorna `docs/PRD.md` con la nuova decisione e la data.
4. Non implementare feature "silenziose" che non sono tracciate nel PRD o nella roadmap.
5. Se il PRD contraddice una rule tecnica, segnala il conflitto e chiedi conferma.
```
