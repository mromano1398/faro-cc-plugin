# Wizard Flow — Riferimento operativo

Questo documento estende `SKILL.md` con template di domande, decision tree, mappature
tipo→tier, proposte precompilate e gestione dei casi limite. È pensato come riferimento
da consultare durante l'esecuzione del wizard, non come script lineare.

---

## 1. Decision tree iniziale — dedurre TIPO e LIVELLO UTENTE

Quando l'utente risponde alla prima domanda ("Descrivimi cosa vuoi costruire"),
analizza il testo cercando pattern linguistici specifici prima di porre altre domande.

### 1.1 Pattern → TIPO progetto

**Gestionale / ERP / tool interno**
Cerca verbi/sostantivi: "gestire", "gestionale", "amministrare", "interno", "dipendenti",
"magazzino", "commesse", "fatture", "anagrafica", "CRUD", "tabella", "filiale",
"ordini interni", "ticket", "helpdesk", "registro".
Segnale forte: l'utente parla di entità da gestire più che di persone da convincere.

**SaaS (Software as a Service)**
Cerca: "abbonamento", "piano", "tier", "mensile", "utenti finali", "dashboard", "multi-utente",
"workspace", "team", "multi-tenant", "subscription", "paghi al mese", "pricing".
Segnale forte: menzione di pricing ricorrente o "utenti che pagano".

**E-commerce**
Cerca: "vendere", "vendita", "prodotti", "catalogo", "carrello", "checkout", "pagamento",
"spedizione", "Stripe", "negozio online", "shop", "marketplace", "cliente compra".
Segnale forte: "vendere X a Y" con X fisico/digitale.

**Landing page / sito marketing**
Cerca: "landing", "presentare", "lanciare un prodotto", "raccogliere email", "lead",
"waitlist", "coming soon", "sito one-page", "pre-lancio", "hero", "call to action".
Segnale forte: obiettivo è conversione, non funzionalità.

**Showcase / portfolio / brand**
Cerca: "portfolio", "mostrare lavori", "studio", "agenzia", "brand", "identità",
"wow", "impressionare", "cliente premium", "cinematic", "awwwards", "estetica",
"arte", "fotografia", "architetto", "designer".
Segnale forte: parla di estetica e impatto visivo più che di funzioni.

**Mobile app**
Cerca: "app", "iOS", "Android", "mobile", "smartphone", "push notification", "offline",
"React Native", "Expo", "App Store", "Play Store", "nativa".
Segnale forte: menzione esplicita di store o piattaforma mobile.

**API / backend**
Cerca: "API", "endpoint", "REST", "GraphQL", "backend", "microservizio", "integrazione",
"webhook", "servizio", "SDK".
Segnale forte: nessuna UI menzionata, solo contratto dati.

**CLI / bot / automazione**
Cerca: "CLI", "comando", "terminale", "bot", "Discord", "Telegram", "Slack",
"automazione", "script", "scheduler", "cron", "GitHub Action".
Segnale forte: niente utenti web, gira in background o da terminale.

### 1.2 Pattern → LIVELLO UTENTE

**BEGINNER (linguaggio non tecnico)**
Segnali:
- Usa frasi come "voglio fare un sito", "non so da dove partire", "ho un'idea"
- Non menziona nessuna tecnologia specifica
- Usa metafore generiche ("come Amazon", "tipo Instagram")
- Chiede spiegazioni di termini base ("cosa è un database?")
- Non parla mai di deploy, hosting, framework

→ Invoca `faro-beginner` prima di procedere.

**DEV (utente tecnico)**
Segnali:
- Menziona framework (Next.js, React, Vue, Django, FastAPI, Rails)
- Cita provider (Supabase, Vercel, AWS, PlanetScale)
- Parla di architettura (monorepo, SSR, RSC, edge, serverless)
- Usa acronimi tecnici senza spiegarli (ORM, JWT, RLS, CI/CD)
- Ha opinioni su scelte tecniche ("voglio usare Drizzle invece di Prisma")

→ Procedi col flusso standard del wizard.

**AMBIGUO**
Se non riesci a classificare con sicurezza, tratta come BEGINNER by default
(meglio sovra-spiegare che sovrastimare la competenza).

> **Nota di migrazione:** la classificazione sopra (1.2 v1) resta valida come base,
> ma è estesa dal **Classificatore v2** nella sezione 1.3 — che aggiunge il criterio
> del lessico di dominio professionale per evitare falsi BEGINNER su utenti esperti
> di settori industriali (logistica, sanità, produzione, ecc.). **In caso di conflitto,
> la v2 prevale.**

---

## 1.3 Classificatore DEV / BEGINNER — v2 con lessico di dominio

Un utente è DEV se soddisfa ALMENO UNO di questi criteri:

### Criterio A — Lessico tecnico (framework / ORM)
L'input contiene almeno una parola chiave tecnica:
- Framework: Next.js, React, Vue, Svelte, Nuxt, Astro, Expo, FastAPI, Django, Flask, Rails
- ORM / DB: Prisma, Drizzle, SQLAlchemy, Postgres, MongoDB, Redis
- Auth: NextAuth, Supabase, Clerk, Auth0, Firebase Auth
- Deploy: Vercel, Docker, Railway, Fly, Cloudflare
- CSS/UI: Tailwind, shadcn, Framer Motion, GSAP, Three.js
- Concetti: API, REST, GraphQL, webhook, middleware, server action, RSC, hydration, CSR, SSR
- Tooling: TypeScript, ESLint, Prettier, Vitest, Playwright, Husky

### Criterio B — Lessico di dominio professionale
L'input contiene terminologia specifica di un settore che indica esperienza professionale,
indipendentemente dal fatto che l'utente nomini o meno lo stack tecnico. Esempi:

| Settore | Parole indicatrici |
|---------|---------------------|
| Logistica / magazzino | DDT, bolle, picking, packing, bobine, pallet, SKU, lotti, scadenze, UDC, ASN |
| Produzione / MES | BOM, distinta base, ciclo di lavoro, commessa, commessa chiusa, scarti, OEE |
| Sanità | prescrizione, triage, anamnesi, cartella clinica, referto, NTS, SDO, ICD-10 |
| Finanza / amministrazione | fattura elettronica, SDI, partita IVA, reverse charge, ritenuta, F24 |
| Legale | sentenza, atto, udienza, ricorso, pratica, fascicolo, precedente |
| HR | assunzione, contratto, busta paga, ferie, ROL, TFR, assumibili |
| Edilizia / cantiere | SAL, computo metrico, POS, DURC, subappalto, cantiere aperto |
| Retail / ecommerce | SKU, EAN, GTIN, marketplace, marketplace fee, reso, aging stock |

Se l'utente usa 2+ parole di un settore → è un **professionista del dominio** → flusso **DEV**
(ma nel flusso DEV offrigli di partire da un tipo progetto "gestionale/ERP" se rilevante).

### Criterio C — Autoclassificazione esplicita
Se l'utente dice "non sono un developer", "non so programmare", "non capisco di tecnologia"
→ flusso **BEGINNER**.

### Regola di precedenza
1. Se C è presente → BEGINNER (l'utente si autoclassifica, rispetta)
2. Altrimenti se A o B → DEV
3. Altrimenti (input vago come "voglio un sito per la mia attività") → chiedi 1 domanda di
   disambiguazione: "Ti va se ti faccio alcune domande tecniche veloci, o preferisci che te le
   semplifichi?" → risposta guida il flusso

### Esempi di classificazione
| Input | Classificazione | Motivo |
|-------|----------------|--------|
| "Voglio gestire i DDT e le bobine del magazzino cavi" | DEV | 2 parole dominio logistica |
| "SaaS Next.js con Supabase e Stripe" | DEV | criterio A |
| "Vorrei un sito per la mia pizzeria" | — | vago → chiedi disambiguazione |
| "Fai tu, non capisco di queste cose" | BEGINNER | criterio C |
| "Devo tracciare le cartelle cliniche dei miei pazienti, sono un medico" | DEV | dominio sanità + "miei pazienti" = professionista |

---

## 2. Template domande per tipo

Per ciascun tipo, usa **solo le domande necessarie** che non sei già riuscito a dedurre.
BEGINNER: max 2. DEV: max 4.

### 2.1 Gestionale / ERP

1. Chi userà l'applicazione? (dipendenti interni, filiali esterne, clienti con ruoli diversi)
2. Quali sono le entità principali da gestire? (es: clienti, commesse, fatture, magazzino)
3. Serve multi-ruolo con permessi diversi? (admin, operatore, visualizzatore)
4. Ci sono integrazioni obbligatorie? (SAP, software esistente, Excel, gestionale legacy)

### 2.2 SaaS

1. Il modello è B2B, B2C, o ibrido?
2. È multi-tenant? (ogni cliente vede solo i propri dati)
3. Pricing: freemium, trial + paid, solo paid? Ci sono piani differenziati?
4. Integrazioni essenziali da day-1? (Stripe, Slack, Google, OAuth providers)

### 2.3 E-commerce

1. Quanti prodotti e che tipo? (pochi artigianali, catalogo ampio, digitali, servizi)
2. Solo B2C o anche B2B con listini differenziati?
3. Pagamenti: solo carta, anche PayPal/Apple Pay, fatturazione elettronica?
4. Serve gestione magazzino e spedizioni integrata o solo vetrina con checkout?

### 2.4 Landing page / sito marketing

1. Obiettivo principale: raccogliere email, vendere un prodotto, promuovere un evento?
2. Serve un CMS per aggiornare contenuti senza dev, o è statica?
3. Multilingua? Quante lingue?
4. Analytics e A/B testing richiesti dal day-1?

### 2.5 Showcase / portfolio / brand

1. Chi è il pubblico? (clienti premium, recruiter, pubblico generico, press)
2. Quanti "progetti" o case study vanno mostrati e quanto sono ricchi di media?
3. Estetica di riferimento? (hai URL, mood, tier visivo preferito)
4. Serve area contatti/preventivo o solo vetrina?

### 2.6 Mobile app

1. iOS, Android, o entrambi?
2. Serve offline-first o always-online?
3. Funzionalità native richieste? (camera, GPS, push, biometria, NFC)
4. C'è già un backend o va costruito da zero?

### 2.7 CLI / bot / automazione

1. Dove gira? (terminale locale, server, GitHub Action, bot su piattaforma X)
2. Input da cosa? (argomenti CLI, messaggi chat, webhook, scheduler)
3. Output dove? (stdout, file, API, notifica)
4. Deve essere installabile da altri o è solo per te?

---

## 3. Mappatura tipo → Tier consigliato

| Tipo progetto          | Tier default | Tier alternativo | Note                                         |
|------------------------|--------------|------------------|----------------------------------------------|
| Gestionale / ERP       | 1            | 2                | Tier 2 solo se cliente esigente              |
| SaaS                   | 2            | 1 o 3            | Tier 3 se brand/premium, Tier 1 se interno   |
| E-commerce             | 2            | 3                | Tier 3 per brand luxury o fashion            |
| Landing marketing      | 2            | 3                | Tier 3 se prelancio hype/premium             |
| Showcase / portfolio   | 3            | 4                | Tier 4 per architetti, luxury, car configurator |
| Mobile app             | 2            | —                | Tier 3/4 non applicabile a mobile nativo     |
| API / backend          | —            | —                | Nessun tier (no UI)                          |
| CLI / bot              | —            | —                | Nessun tier (no UI)                          |

**Regole inderogabili del tier:**
- Tier 3 e 4 NON possono usare Inter come font primario.
- Tier 1 e 2 possono usare Inter ma non come unico font del brand.
- Nessun tier può usare i colori default di Tailwind (slate/gray/blue shadcn base) come palette brand.

---

## 4. Template di proposta precompilata (fallback)

Usa queste proposte quando `faro-stack-advisor` non è disponibile o per pre-popolare
rapidamente il blocco proposta dello Step 4. Adatta in base a vincoli espressi.

### 4.1 Gestionale

```
Stack:         Next.js 15 (App Router) + TypeScript + Tailwind + shadcn/ui
Database:      PostgreSQL (Neon o Supabase)
ORM:           Drizzle
Auth:          Auth.js (credentials + email) o Clerk se budget permette
Deploy:        Vercel
Architettura:  blueprint "dashboard-crud" (sidebar + data tables + form modali)
Sicurezza:     Livello 2 (RBAC, audit log, rate limit)
Tier:          1 — FUNZIONALE
Costo:         0-20€/mese
```

### 4.2 SaaS

```
Stack:         Next.js 15 + TypeScript + Tailwind + shadcn/ui + Framer Motion
Database:      PostgreSQL (Supabase o Neon) con RLS
ORM:           Drizzle o Prisma
Auth:          Clerk (più rapido) o Auth.js
Pagamenti:     Stripe (subscription + customer portal)
Deploy:        Vercel
Architettura:  blueprint "saas-multi-tenant" (workspace, membri, billing, dashboard)
Sicurezza:     Livello 2-3 (RLS, rate limit, webhook verification)
Tier:          2 — PROFESSIONALE
Costo:         25-80€/mese (Stripe + Clerk + hosting)
```

### 4.3 E-commerce

```
Stack:         Next.js 15 + TypeScript + Tailwind + shadcn/ui + Framer Motion
Database:      PostgreSQL (Supabase)
CMS/Catalog:   Sanity o Payload (se servono molti prodotti editoriali) oppure DB diretto
Pagamenti:     Stripe Checkout + webhook ordini
Auth:          Clerk o Auth.js
Deploy:        Vercel
Architettura:  blueprint "ecommerce-storefront" (catalogo, PDP, carrello, checkout, area account)
Sicurezza:     Livello 2 (validazione checkout, webhook firmati, protezione bot)
Tier:          2 (default) o 3 (luxury/fashion)
Costo:         30-100€/mese
```

### 4.4 Landing page

```
Stack:         Next.js 15 + TypeScript + Tailwind + shadcn/ui + Framer Motion
CMS:           Opzionale — Sanity o MDX locale
Form:          Resend/Postmark per lead, o Formspree
Analytics:     Vercel Analytics + Plausible
Deploy:        Vercel
Architettura:  blueprint "landing" (hero, features, pricing, FAQ, footer, lead capture)
Sicurezza:     Livello 1 (captcha sui form, anti-spam)
Tier:          2 (default) o 3 (prelancio premium)
Costo:         0-15€/mese
```

### 4.5 Showcase / portfolio

```
Stack:         Next.js 15 + TypeScript + Tailwind + GSAP + Lenis (+ Three.js per Tier 4)
CMS:           Sanity (se l'utente deve aggiornare) o MDX
Deploy:        Vercel
Architettura:  blueprint "showcase" (hero cinematic, grid progetti, case study dettaglio, contatti)
Sicurezza:     Livello 1 (form contatti protetto)
Tier:          3 — CINEMATIC (default) o 4 — IMMERSIVO (luxury)
Font:          Font display custom (es: Migra, PP Neue Montreal, Fraunces) — MAI Inter
Palette:       Dark theme dominante, accento brand distintivo
Costo:         0-20€/mese
```

### 4.6 Mobile app

```
Stack:         Expo (React Native) + TypeScript + NativeWind + Zustand
Backend:       Supabase (auth + DB + storage + realtime) oppure API custom
Auth:          Supabase Auth o Clerk Expo
Push:          Expo Notifications
Deploy:        EAS Build + EAS Update + TestFlight + Play Console
Architettura:  blueprint "mobile-app" (tab bar, stack auth, deep link, offline cache)
Sicurezza:     Livello 2 (secure store per token, cert pinning se serve)
Tier:          2 — PROFESSIONALE
Costo:         0-30€/mese (EAS free tier + Supabase free tier)
```

### 4.7 CLI / bot / automazione

```
Stack:         TypeScript + Node/Bun, oppure Python (FastAPI se serve webhook)
CLI parser:    commander / clipanion / typer (Python)
Config:        file .json o .toml, env vars
Distribuzione: npm package / pip package / binario (pkg, bun build --compile)
Deploy (bot):  Railway, Fly.io, Vercel Functions, o GitHub Actions
Architettura:  blueprint "cli-tool" o "bot-service"
Sicurezza:     Livello 1-2 (secrets in env, validazione input)
Tier:          — (nessun UI visuale)
Costo:         0-10€/mese (o gratis)
```

---

## 5. Gestione vincoli tecnici imposti

Quando l'utente impone vincoli durante lo Step 3, NON discutere: accetta e adatta.
Segnala solo i trade-off che l'utente probabilmente non vede.

### 5.1 Vincoli comuni e risposte consigliate

**"Devo usare MySQL"**
Accetta. Cambia ORM se serve (Drizzle supporta MySQL, Prisma anche).
Avvisa: "MySQL va bene, perdiamo però alcune feature PostgreSQL come RLS nativa
e tipi JSON più ricchi. Per auth multi-tenant dovremo gestire permessi a livello
applicativo invece che DB. Confermi?"

**"Deploy su AWS obbligatorio"**
Accetta. Cambia piattaforma di deploy nella proposta.
Avvisa: "Su AWS perdiamo l'automatismo di Vercel (preview deploy, edge network).
Consiglio: Next.js standalone su ECS/Fargate, oppure sst.dev, oppure AWS Amplify
se vuoi restare declarativo. Quale preferisci?"

**"Deve essere self-hosted / on-premise"**
Accetta. Cambia stack verso qualcosa di self-hostable end-to-end.
Proponi: Docker Compose con Next.js + PostgreSQL + Redis; auth con Auth.js
(Clerk non è self-hostable). Avvisa: "Perdiamo i managed service, dovremo
gestire backup, SSL e monitoring."

**"Budget zero, tutto gratis"**
Accetta. Sposta tutto sui free tier: Vercel Hobby, Supabase free, Auth.js
invece di Clerk, Resend free tier, niente Sentry paid.
Avvisa i limiti dei free tier (es: Supabase pausa dopo 1 settimana inattività).

**"Voglio usare [framework X obsoleto o non mainstream]"**
Accetta se supportato da Faro. Se non supportato, proponi l'alternativa
più vicina e spiega perché: "Meteor non è più consigliato nel 2026, l'alternativa
moderna per realtime full-stack è Next.js + Supabase realtime oppure Convex.
Vuoi proseguire con Meteor comunque o provi l'alternativa?"

**"Preferisco Python invece di Node"**
Accetta. Passa a blueprint FastAPI + frontend Next.js separato, oppure
Django full-stack se preferisce monolite. Segnala: "Con stack Python perdiamo
type safety end-to-end, compensiamo con Pydantic + OpenAPI generator per il client."

### 5.2 Regola d'oro sui vincoli

Se il vincolo rende impossibile il tier scelto (es: Tier 4 su deploy WordPress),
fermati e spiegalo con chiarezza: "Con questo vincolo il Tier 4 non è
raggiungibile. Puoi: a) cambiare il vincolo, b) scendere a Tier 2, c) accettare
Tier 4 parziale senza 3D. Quale preferisci?"

---

## 6. Flusso BEGINNER

Quando rilevi un utente beginner al Step 1, cambia completamente registro.

### 6.1 Regole di comunicazione

- **Zero jargon non spiegato**: se usi un termine tecnico, affiancalo a una metafora.
  Esempio: "un database (cioè l'archivio dove salviamo i dati, come un grosso Excel)".
- **Max 2 domande totali nello Step 3**, non 4.
- **Nessuna scelta tecnica nella proposta**: niente "PostgreSQL vs MySQL",
  niente "Drizzle vs Prisma". Mostra solo risultati ("useremo un database moderno e affidabile").
- **Prezzi in euro/mese**, mai "free tier" o "pay-as-you-go".
- **Tier in parole semplici**:
  - Tier 1 → "semplice e funzionale, come un gestionale classico"
  - Tier 2 → "moderno e curato, come i siti professionali che vedi online"
  - Tier 3 → "cinematografico, quando il design deve far dire wow"
  - Tier 4 → "ultra-premium con effetti 3D, come un sito di auto di lusso"

### 6.2 Flusso modificato

1. **Step 1** — Descrizione visione (come sempre).
2. **Invoca `faro-beginner`** che:
   - Riformula la visione per confermare comprensione.
   - Se serve, fa domande alternative in linguaggio naturale.
   - Ritorna al wizard con: tipo dedotto, pubblico, obiettivo business, 1-2 vincoli.
3. **Step 2** — Tier proposto direttamente (non chiesto): "Per quello che vuoi fare
   consiglio il Tier X — [spiegazione semplice]. Va bene o preferisci più semplice / più wow?"
4. **Step 3** — Solo 1-2 domande essenziali (pubblico + budget).
5. **Step 4** — Proposta in linguaggio semplice, senza nomi di tecnologie.
   Esempio: "Useremo gli strumenti più moderni disponibili, hosting gratuito per iniziare,
   sistema di login sicuro, database affidabile. Costo: gratis per i primi mesi,
   poi circa 20€/mese quando avrai traffico."
6. **Step 5 e 6** — Identici, ma sempre con spiegazione in linguaggio semplice
   ("ora preparo tutti i file che servono al progetto, ci metto circa un minuto").

### 6.3 Cosa NON fare mai con un beginner

- Non chiedere "Auth.js o Clerk?"
- Non chiedere "multi-tenant sì/no?"
- Non elencare 10 tecnologie nella proposta
- Non usare la parola "stack", "deploy", "scaffolding"
- Non proporre scelte che richiedono competenze tecniche per essere valutate

---

## 7. Uscite di emergenza

### 7.1 L'utente non risponde chiaramente alla prima domanda

Esempio: "Boh, voglio qualcosa di bello e utile."

Strategia:
1. Offri 3 esempi concreti: "Ok, mi aiuti a capire? Vuoi qualcosa tipo:
   (a) un sito per far vedere i tuoi lavori a clienti,
   (b) un'app dove le persone si registrano e usano un servizio a pagamento,
   (c) un gestionale per organizzare il lavoro interno?"
2. Tratta come BEGINNER automaticamente.
3. Se ancora non è chiaro dopo 2 tentativi, chiedi: "Qual è il risultato finale
   che ti aspetti? Chi dovrebbe usarlo e per fare cosa?"

### 7.2 L'utente cambia idea a metà wizard

Esempio: l'utente era su SaaS, a Step 3 dice "ah no, in realtà è solo una landing".

Strategia:
1. Conferma il cambio esplicitamente: "Ok chiariamo: ora vuoi una landing per
   raccogliere email, non più una SaaS con login. Giusto?"
2. Se sì, **ricomincia dallo Step 2** con il nuovo tipo, ma riusa le info
   già raccolte (pubblico, budget) se ancora valide.
3. Non far mai finta che il cambio non sia avvenuto. Non procedere
   con la proposta vecchia "per non rifare tutto".

### 7.3 L'utente chiede di tornare indietro

Se l'utente dice "aspetta, torniamo al tier" o "posso cambiare lo stack?":
1. Sempre sì, mai resistere.
2. Chiedi cosa vuole cambiare con precisione.
3. Modifica solo quel campo, mantieni il resto della proposta.
4. Ripresenta il blocco proposta aggiornato e chiedi conferma.

### 7.4 L'utente è indeciso tra 2 tier o 2 stack

Strategia:
1. Non decidere per lui senza chiederlo.
2. Presenta le differenze concrete: "Tier 2 = più veloce da fare, più economico,
   meno wow. Tier 3 = 2-3x tempo in più, 2x costo, ma risultato cinematografico."
3. Chiedi: "Su quale lato vuoi ottimizzare: velocità/costo o impatto visivo?"
4. Procedi con la scelta.

### 7.5 L'utente dice "fai tu, decidi al posto mio"

Strategia:
1. Ok, attiva modalità **Autopilot proposta**: scegli i default consigliati per
   il tipo rilevato, senza chiedere altro.
2. Mostra comunque il blocco proposta completo.
3. Chiedi SOLO: "Va bene così? [OK] / [M] modifica qualcosa / [AP FULL] autopilot totale"
4. Non saltare la conferma finale, anche in modalità autopilot proposta.

### 7.6 L'utente ha fretta estrema

Esempio: "devo fare questa cosa stasera, non ho tempo, vai subito".

Strategia:
1. Attiva il percorso più corto: Step 1 → Step 4 direttamente.
2. Usa proposta precompilata dal template sezione 4.
3. Conferma in una riga: "Ok, procedo con proposta standard per [tipo].
   Tier default, stack default, sicurezza base. OK?"
4. Se OK, salta anche la scelta fase in Step 6 e parti direttamente da Fase 1.

### 7.7 L'utente contesta il tier suggerito

Esempio: "Vuoi il Tier 1 su un gestionale ma io voglio Tier 3."

Strategia:
1. Sempre ok, l'utente decide.
2. Segnala il trade-off una volta sola: "Ok, Tier 3 su un gestionale è raro
   ma fattibile: avremo un gestionale cinematico. Costo: più tempo sulle animazioni.
   Confermi?"
3. Se conferma, procedi senza ulteriori obiezioni.

---

## 8. Checklist finale prima di passare a faro-generator

Prima di invocare `faro-generator` allo Step 5, verifica di avere TUTTI questi campi:

- [ ] Tipo progetto
- [ ] Livello utente (BEGINNER o DEV)
- [ ] Tier di design (1-4, o N/A per API/CLI)
- [ ] Pubblico target
- [ ] Modello di business (se applicabile)
- [ ] Stack completo (framework, DB, auth, deploy)
- [ ] Blueprint architetturale
- [ ] Livello sicurezza (1-3)
- [ ] Palette colori (mai default)
- [ ] Font scelti (mai Inter su Tier 3-4)
- [ ] Stile visivo di riferimento
- [ ] Budget mensile stimato
- [ ] Conferma esplicita dall'utente ([OK] o [AP FULL])

Se manca anche uno solo di questi, NON invocare il generator. Torna indietro
e raccogli il campo mancante. Eccezione: API/CLI non hanno tier, palette, font, stile.
