---
name: faro-generator
description: |
  Genera TUTTI i file nativi Claude Code nel progetto dell'utente: CLAUDE.md, 11 rules,
  11 commands, 6 agents, 3 docs, 2 tracking files (totale 34 file). Invocato dal wizard
  e dall'adopt. Trigger: invocato da faro-wizard o faro-adopt dopo l'approvazione della proposta
---

# Faro Generator

## Input richiesti

Ricevi dal wizard/adopt:
- nome_progetto
- tipo_progetto (gestionale | SaaS | ecommerce | landing | showcase | mobile | cli | api | python)
- tier (1-4)
- stack (lista tecnologie)
- palette (colori scelti)
- font (font scelti)
- blueprint (struttura da faro-blueprints)
- mode (new | adopt)
- target_directory (opzionale, default: "./") — directory di destinazione dove generare i file.
  Usato dal percorso multipiattaforma per generare rules dentro `apps/mobile/`, `apps/desktop/`, ecc.
  Se omesso, i file vengono generati nella root del progetto.
- extra (opzionale) — vincoli, preferenze utente
- Se mode=adopt: riceve un oggetto tipato descritto nella sezione "Schema input mode=adopt"
  (vedi sotto). NON usare più `extra` come catch-all per i dati estratti dal codice —
  usa i campi tipati dello schema.

## Ordine di generazione

1. `CLAUDE.md` — overview progetto
2. `.claude/rules/` — 11 rule files
3. `.claude/commands/` — 11 command files
4. `.claude/agents/` — 6 agent files
5. `docs/` — MVP.md, PRD.md, roadmap.md
6. `.faro/` — state.md, log.md, plans/

## Template

Per i template di ogni file, leggi:
- Rules: `references/templates-rules.md`
- Commands: `references/templates-commands.md`
- Agents: `references/templates-agents.md`
- Docs + tracking: `references/templates-docs.md`

## Regole di generazione

1. Se mode=adopt: le rules riflettono il codice REALE, non template generici
2. La palette è SEMPRE personalizzata — MAI usare colori default
3. Font Tier 3-4: MAI Inter/Roboto come heading
4. Ogni file ha contenuto CONCRETO, non placeholder
5. I flussi nei commands sono COMPLETI
6. Gli agents hanno prompt DETTAGLIATI
7. **Filtro regole per tipo:** consulta la nota "Nota sul filtro per tipo progetto" in
   `references/templates-rules.md` (in cima al file). Se una rule contiene regole specifiche
   per web (es: `next/image`, `kebab-case .tsx`) NON applicabili al tipo del progetto, il
   generator le SOSTITUISCE con equivalenti o le RIMUOVE. Vedi tabella filter-by-type nel file.

## Mapping placeholder → input

I template in `references/templates-*.md` contengono placeholder in formato `[NOME_CAMPO]`.
Il generator li sostituisce con i valori ricevuti dal wizard/adopt secondo questa mappa:

| Placeholder | Campo input | Esempio |
|-------------|-------------|---------|
| `[NOME_PROGETTO]` | `nome_progetto` | "NutriCoach" |
| `[TIPO_PROGETTO]` | `tipo_progetto` | "SaaS" |
| `[TIER]` | `tier` (numero) | 2 |
| `[NOME_TIER]` | `tier` → nome | "PROFESSIONALE" |
| `[COLORE_PRIMARY]` | `palette.primary` | "#0ea5e9" |
| `[COLORE_PRIMARY_HOVER]` | `palette.primary_hover` | "#0284c7" |
| `[COLORE_BG]` | `palette.background` | "#ffffff" |
| `[COLORE_SURFACE]` | `palette.surface` | "#f8fafc" |
| `[COLORE_BORDER]` | `palette.border` | "#e2e8f0" |
| `[COLORE_TEXT]` | `palette.text` | "#0f172a" |
| `[COLORE_TEXT_MUTED]` | `palette.text_muted` | "#64748b" |
| `[COLORE_SUCCESS]` | `palette.success` | "#10b981" |
| `[COLORE_WARNING]` | `palette.warning` | "#f59e0b" |
| `[COLORE_ERROR]` | `palette.error` | "#ef4444" |
| `[FONT_HEADING]` | `font.heading` | "Space Grotesk" |
| `[PESO_HEADING]` | `font.heading_weight` | 700 |
| `[FONT_BODY]` | `font.body` | "Inter" |
| `[PESO_BODY]` | `font.body_weight` | 400 |
| `[FONT_MONO]` | `font.mono` | "JetBrains Mono" |
| `[RAGGIO]` | `radius` | "md" |
| `[LISTA STACK COMPLETA]` | `stack` (array) | "Next.js 14, PostgreSQL, Prisma, ..." |
| `[BLUEPRINT]` | `blueprint` | "saas" |
| `[LISTA MODULI]` | dal codice/wizard | inizialmente vuota, popolata da context-updater |

### Regole di sostituzione

1. **Mai lasciare placeholder letterali** in un file generato. Se un campo è mancante, usa un fallback
   documentato (es: testo vuoto, TODO con nota chiara) E segnalalo nel log di generazione.
2. **Colori devono essere hex validi** (#rrggbb o #rgb). Se il wizard fornisce un nome ("blu"), il
   generator fa mapping a hex prima di inserire.
3. **Font devono essere font reali** caricati via Google Fonts o next/font. Il generator aggiunge
   l'import/link corrispondente in CLAUDE.md se serve.
4. **Tier determina anche placeholder condizionali**: i blocchi `[BLOCCO_COMPONENTI_TIER]` in
   design-system.md vengono sostituiti solo col blocco del tier attivo (vedi templates-rules.md
   nota sul filtro).
5. **Mode=adopt**: i placeholder vengono popolati dai valori ESTRATTI dal codice, non da quelli
   scelti dall'utente. Per esempio `[COLORE_PRIMARY]` diventa il colore trovato in tailwind.config.js.

## Schema input mode=adopt

Quando mode=adopt, il generator riceve questo oggetto tipato da faro-adopt (al posto del payload
wizard standard):

```json
{
  "mode": "adopt",
  "nome_progetto": "string",
  "tipo_progetto": "gestionale | SaaS | ecommerce | landing | showcase | mobile | cli | api",
  "tier_dedotto": 1,
  "stack": {
    "framework": "next | react | astro | vue | sveltekit | expo | fastapi | ...",
    "language": "typescript | javascript | python",
    "database": "postgres | mysql | sqlite | mongodb | null",
    "orm": "prisma | drizzle | sqlalchemy | null",
    "auth": "nextauth | supabase-auth | clerk | custom | null",
    "deploy": "vercel | docker | railway | ...",
    "css": "tailwind | css-modules | styled-components",
    "ui_library": "shadcn | radix | chakra | custom"
  },
  "palette_estratta": {
    "primary": "#hex",
    "background": "#hex",
    "text": "#hex",
    "border": "#hex",
    "accent": "#hex"
  },
  "font_estratti": {
    "heading": "string",
    "body": "string",
    "mono": "string | null"
  },
  "moduli_trovati": [
    { "nome": "string", "path": "string", "route": "string", "db_tables": ["..."] }
  ],
  "route_trovate": ["/path", "..."],
  "sicurezza_attuale": {
    "headers": true,
    "validazione_zod": false,
    "rate_limiting": false,
    "rbac": false
  },
  "test_framework": "vitest | jest | playwright | null",
  "gap_rilevati": ["string", "..."]
}
```

Il generator in mode=adopt:
- Usa i campi `*_estratti` e `*_trovati` per popolare i template invece dei valori del wizard.
- Applica il tono DESCRITTIVO (vedi nota in `references/templates-rules.md`): le rules descrivono
  il codice esistente, non prescrivono uno standard ideale.
- Per i `gap_rilevati`, aggiunge una sezione `## Gap rilevati dall'adopt` in `architecture.md`
  che li elenca con priorità.
- Per la `sicurezza_attuale`, popola `security.md` con lo stato REALE (cosa c'è) + suggerimenti
  espliciti su cosa manca rispetto ai baseline del tier dedotto.

## Modalità BEGINNER

Se il wizard passa `modalita: "beginner"` al generator, i file DOCUMENTALI generati
(`CLAUDE.md`, `docs/MVP.md`, `docs/PRD.md`, `docs/roadmap.md`, `.faro/state.md`) DEVONO usare
linguaggio non tecnico:

- "Database" → "l'archivio dove vengono salvati i dati"
- "Deploy" → "mettere online il sito"
- "API" → "collegamento con altri servizi"
- "Auth" → "sistema di accesso con password"
- "Responsive" → "si adatta a computer e telefono"
- "Tier 2" → "con animazioni fluide" (non "Tier 2 Professionale")

Le RULES (`.claude/rules/*.md`) restano tecniche — sono per Claude, non per l'utente.

Il generator:
1. Rileva `modalita: "beginner"` dall'input.
2. Applica un filtro di "traduzione" ai soli file in `docs/` e `.faro/state.md`, e al `CLAUDE.md`.
3. Usa il glossario definito nella skill `faro-beginner` (file `beginner-profiles.md`) per le sostituzioni.
4. NON modifica `.claude/rules/` — restano tecniche.
5. In cima a `CLAUDE.md` e `docs/MVP.md` in modalità beginner, aggiunge una nota:
   "Questo file è scritto in linguaggio semplice per [nome]. La documentazione tecnica completa
   è in `.claude/rules/` (destinata a Claude Code)."
