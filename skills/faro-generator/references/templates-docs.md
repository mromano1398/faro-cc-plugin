# Template Docs + Tracking — 6 file

## 1. CLAUDE.md (root del progetto)

~~~markdown
# [NOME_PROGETTO]

## Overview
[Descrizione breve del progetto — 2-3 frasi]

## Stack
[Lista stack tecnologico]

## Comandi utili
```bash
npm run dev          # Avvia in sviluppo
npm run build        # Build produzione
npm run test         # Esegui test
npm run lint         # Lint
npx tsc --noEmit     # Type check
```

## Architettura
Vedi `.claude/rules/architecture.md` per la struttura completa.

## Design
Tier [N] — [nome]. Vedi `.claude/rules/design-system.md`.

## Comandi progetto
- `/project:status` — stato e score
- `/project:audit` — audit completo
- `/project:reskin` — cambia design
- `/project:advisor [ceo|eng|design|pm|crescita|...]` — consulente prodotto
- `/project:review` — review coerenza
- `/project:update-context` — aggiorna rules
- `/project:timeline` — report cronologico
- `/project:explore` — mappa codebase
- `/project:debug [bug]` — debugging guidato
- `/project:commit` — commit atomici conventional (no push)
- `/project:cleanup` — rimuovi dead code e dipendenze inutilizzate

## Regole importanti
- Leggi SEMPRE `.claude/rules/` prima di modificare
- Chiedi conferma prima di toccare file non richiesti
- Build check dopo ogni 3 file
- Validazione Zod su OGNI input
~~~

## 2. docs/MVP.md

```markdown
# MVP — [NOME_PROGETTO]

## Obiettivo
[Cosa deve fare il prodotto minimo]

## Utente target
[Chi lo usa]

## Feature MVP
1. [Feature 1] — [descrizione]
2. [Feature 2] — [descrizione]
3. [Feature 3] — [descrizione]

## Fuori scope (v1)
- [Feature rimandata]
- [Feature rimandata]

## Criteri di successo
- [Metrica 1]
- [Metrica 2]
```

## 3. docs/PRD.md

```markdown
# PRD — [NOME_PROGETTO]

## Problema
[Quale problema risolve]

## Soluzione
[Come lo risolve]

## Requisiti funzionali

Ogni requisito ha un ID stabile `RF-NNN` (tre cifre, mai riutilizzato).
Gli ID sono la chiave di tracciabilità: piani, commit, log e test possono
citarli per legare implementazione a requisito.

Regole ID:
- Formato: `RF-001`, `RF-002`, ..., `RF-NNN`
- Mai riutilizzare un ID rimosso (marca "ritirato" nel changelog invece)
- Mai rinumerare — gli ID sono append-only
- Per requisiti non funzionali usa `RNF-NNN`

Tabella:

| ID | Titolo | Descrizione | Priorità | Stato |
|----|--------|-------------|----------|-------|
| RF-001 | [Titolo] | [Descrizione] | Alta | [proposto\|approvato\|implementato\|ritirato] |
| RF-002 | ... | ... | Media | proposto |

## Requisiti non funzionali
- Performance: [target]
- Sicurezza: [standard]
- Accessibilità: WCAG 2.1 AA
- Compatibilità: [browser/device]

## Flussi utente
[Descrizione flussi principali]
```

## 4. docs/roadmap.md

```markdown
# Roadmap — [NOME_PROGETTO]

## Fase 1 — Fondamenta
- [ ] Setup scaffolding + lint + TypeScript strict
- [ ] Database + schema + migration
- [ ] Auth + sicurezza base
- [ ] Layout + navigazione
- [ ] Componenti UI base per il tier

## Fase 2 — Costruzione
- [ ] Feature 1: [nome]
- [ ] Feature 2: [nome]
- [ ] Feature 3: [nome]
- [ ] Audit anti-slop

## Fase 3 — Pre-lancio
- [ ] Performance (Lighthouse)
- [ ] Sicurezza (OWASP check)
- [ ] SEO (se web-facing)
- [ ] Accessibilità (>=90)

## Fase 4 — Lancio
- [ ] Deploy produzione
- [ ] Smoke test
- [ ] DNS + SSL

## Fase 5 — Operazioni
- [ ] Monitoring (Sentry)
- [ ] Feature discovery
- [ ] Iterazione
```

## 5. .faro/state.md

```markdown
# Stato Faro — [NOME_PROGETTO]

## Info
- Tipo: [tipo_progetto]
- Tier: [N] — [nome]
- Stack: [lista]
- Creato: [data]
- Ultimo aggiornamento: [data]

## Fase attuale: [N] — [nome fase]

## Score: [N]/100
- Fondamenta: [N]/20
- Sicurezza: [N]/20
- Design: [N]/20
- Funzionalità: [N]/20
- Qualità: [N]/20

## Piano attivo
[Nome piano o "nessuno"]

## Recovery point
Ultimo task completato: [task N — titolo]
Ultimo build check: [pass | fail — data YYYY-MM-DD HH:mm]
Context utilization stimato: [low | medium | high]
Prossimo task previsto: [task N+1 — titolo]
Note: [eventuali blocchi, decisioni in sospeso, ecc]
```

## 6. .faro/log.md

```markdown
# Log Faro — [NOME_PROGETTO]

Log append-only di tutte le azioni significative del progetto.

## [YYYY-MM-DD HH:mm]
**Azione:** Progetto inizializzato da Faro
**Piano:** [nome piano o "init"] — Ref: [RF-001, RF-002] oppure "n/a"
**File:** CLAUDE.md, .claude/rules/, .claude/commands/, .claude/agents/, docs/, .faro/
**Risultato:** OK
```

## 7. .faro/decisions/ (ADR — Architecture Decision Records)

Cartella opzionale per documentare decisioni architetturali importanti.
Creata su richiesta dall'utente, dal wizard/adopt quando rilevano decisioni, o da `/project:explore`.

### Template decisione

```markdown
# ADR-NNN: [Titolo decisione]

## Data
[YYYY-MM-DD]

## Stato
[proposta | accettata | deprecata | sostituita da ADR-NNN]

## Contesto
[Perche' questa decisione e' stata necessaria? Quale problema risolveva?]

## Decisione
[Cosa e' stato deciso. Conciso ma completo.]

## Conseguenze
- **Pro:** [vantaggi]
- **Contro:** [svantaggi accettati]
- **Rischi:** [cosa potrebbe andare storto]

## File coinvolti
[Lista dei file principali toccati da questa decisione — usata per staleness tracking]
- `src/lib/auth.ts`
- `src/middleware.ts`
- `prisma/schema.prisma`

## Alternativa scartata
[Cosa e' stato valutato ma NON scelto, e perche']
```

### Staleness tracking
Il context-updater e `/project:explore` verificano le decisioni:
- Se i file coinvolti sono stati modificati >3 volte dopo la data della decisione -> la decisione e' potenzialmente stale
- Se lo stato e' "accettata" ma il codice e' stato riscritto -> suggerisci revisione
- Le decisioni con stato deprecata/sostituita non generano alert

### Quando creare una decisione
- Scelta del framework (Next.js vs Astro)
- Scelta auth (JWT vs session, Supabase vs Clerk)
- Scelta database (Postgres vs Mongo, Prisma vs Drizzle)
- Scelta architetturale (monorepo vs multi-repo, server actions vs API routes)
- Scelta di NON fare qualcosa (es: "non usiamo Redis per ora perche'...")
