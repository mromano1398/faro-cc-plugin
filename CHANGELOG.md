# Changelog

Tutte le modifiche notevoli di questo plugin saranno documentate in questo file.

Il formato segue [Keep a Changelog](https://keepachangelog.com/it/1.1.0/)
e il versionamento segue [Semantic Versioning](https://semver.org/lang/it/).

## [Unreleased]

Nessuna modifica al momento.

## [1.2.0] - 2026-04-12

### Aggiunto
- Integrazione pattern da everything-claude-code (ECC): 11 pattern selezionati da 47 agent e 181 skill
- **build-checker potenziato**: tabella 8 errori comuni TS/Next.js con fix mirati, comandi diagnostici, recovery cache (da ECC build-error-resolver)
- **workflow.md context awareness espanso**: stima token budget, tabella quando fare /compact, riduzione proattiva contesto (da ECC harness-optimizer)
- **faro-autopilot stuck detection avanzata**: protocollo escalation 4 livelli (retry→decompose→skip→stop), timeout per tipo task, De-Sloppify pattern (da ECC loop-operator)
- **Nuovo comando `/project:cleanup`**: dead code scan, unused imports/exports/deps, categorizzazione rischio, ordine rimozione sicuro (da ECC refactor-cleaner)
- **code-reviewer Stadio 3 — review schema DB**: anti-pattern (N+1, no indici FK, raw SQL), RLS Supabase, migration safety expand-contract (da ECC database-reviewer)
- Confidence filtering nel code-reviewer (evita segnalare issue incerte)
- AI-generated code addendum nel code-reviewer (rileva codice probabilmente generato)
- TypeScript async patterns in code-style.md (no floating promises, error boundaries)
- Security scan rapido nel build-checker (.env esposti, eval(), innerHTML)
- Quick-scan pattern di rischio in security.md (path traversal, prototype pollution)
- CHANGELOG.md dedicato in formato keep-a-changelog
- CONTRIBUTING.md con filosofia, tipi di contributo, convenzioni commit, policy semver
- Issue templates GitHub (bug report, feature request, config)
- CI GitHub Actions con 2 job (validate-plugin, markdown-lint)
- Sezione FAQ nel README con 12 domande

### Modificato
- Generator SKILL.md: dichiara 11 commands e 34 file totali
- README.md: puntatore a CHANGELOG.md (changelog non più inline), conteggi aggiornati

### Corretto
- 3 broken cross-skill references (path `references/xxx.md` cross-skill interpretati come interni dal CI)

## [1.1.0] - 2026-04-11

### Aggiunto
- Nuovo blueprint API (backend-only): `faro-blueprints/references/api.md` con stack Hono/FastAPI/NestJS, security checklist, REST conventions
- Combo stack "SaaS con AI" (Next.js + Supabase pgvector + Vercel AI SDK + Upstash rate-limit) in `faro-stack-advisor/references/recommendations.md`
- Combo stack "API (backend only)" in `faro-stack-advisor/references/recommendations.md`
- Classificatore DEV/BEGINNER v2 con lessico di dominio (logistica, sanità, finanza, legale, HR, edilizia, retail, produzione) in `faro-wizard/references/wizard-flow.md`
- Sezione "Tier" nominata nel template `design-system.md`
- Componenti Tier 1-4 inline in `design-system.md` (elimina dipendenza runtime da `faro-tier-system`)
- Pattern form standard (Strategia A: Server Actions + `useActionState`, Strategia B: react-hook-form + Zod) nel template `code-style.md`
- Convenzioni directory "Dove vanno le cose" (lib/, components/ui, components/filters, ecc.) nel template `code-style.md`
- Nota tono descrittivo per `mode=adopt` in `templates-rules.md` (generator produce rules descrittive quando adotta progetti esistenti)
- Nota filtro rules per tipo progetto in `templates-rules.md` (quali regole si applicano a web / mobile / CLI / Python)
- Tabella mapping placeholder → input nel generator `SKILL.md` (22 placeholder mappati ai campi del wizard)
- Schema input `mode=adopt` tipato (JSON schema) nel generator `SKILL.md` per l'handoff adopt → generator
- Modalità BEGINNER propagata nei file generati (CLAUDE.md, docs/MVP.md) con glossario e 5 step procedurali
- Prima domanda obbligatoria nel wizard Step 3: "Come si chiama il progetto?" per `nome_progetto`
- Metro config per workspace pnpm in `porting-strategies.md` (obbligatoria per Next.js → Expo monorepo)
- Handoff esplicito multipiattaforma (step 6-8) in `faro-multiplatform/SKILL.md` per generare rules `apps/mobile/`
- Sezione "Web vs Mobile: Stripe, IAP e policy store" in `faro-payments` con anti-pattern
- Check `TIER_MISMATCH` nel command `audit.md` (verifica che Tier ≥2 abbia Framer Motion, skeleton, GSAP, Three.js effettivamente installati)
- Update dipendenze nel command `reskin.md` (aggiorna `architecture.md` con nuove dipendenze tier, gestisce downgrade con conferma)
- Sezione FAQ nel `README.md` con 12 domande
- Regola "componente condiviso" in `workflow.md` template (disclosure proattivo impatto)
- Struttura tabellare moduli in `architecture.md` template
- Pattern pagine standard per tipo in `product-guidance.md` template
- Parametro `target_directory` in `faro-generator` (per generare rules in subdirectory)
- Tipi progetto `api` e `python` aggiunti al generator
- Istruzione filtro rules per tipo nel generator
- Schema handoff tipizzato in `faro-adopt/SKILL.md` (`palette_estratta`, `font_estratti`, `moduli_trovati`, `sicurezza_attuale`, `gap_rilevati`)

### Modificato
- Generator `SKILL.md`: dichiara 10 commands (invece di 9) e 33 file totali (invece di 31)
- `README.md`: sincronizzati conteggi "33 file" / "10 commands" / "33 file nativi"
- Orchestratore `faro`: percorso RIPRENDI riconosce "Recovery point" in `.faro/state.md`

### Corretto
- Drift cross-reference tra generator `SKILL.md` e wizard/adopt post fix paralleli v1.1
- Audit check tier: specificato formato grep `^## Tier: [0-9]` per leggere il tier dichiarato
- Wizard propaga esplicitamente `modalita: "beginner"` al generator quando applicabile

Totale: 23 fix (3 alti v1.0 + 20 item backlog v1.1), ~750 righe aggiunte su 15 file, 1 nuovo blueprint (`api.md`), 0 skill modificate nel `plugin.json`.

## [1.0.0] - 2026-04-11

### Aggiunto
- Release iniziale
- 24 skill (11 orchestrazione + 13 dominio)
- 31 file generati nel progetto utente (CLAUDE.md + 11 rules + 9 commands + 6 agents + 3 docs + 2 tracking)
- 4 tier di design (funzionale, professionale, cinematic, immersivo)
- 80+ tecnologie nel catalogo in 16 categorie
- 7 blueprint architetturali
- Advisor prodotto con memoria project-scoped
- Autopilot con 4 gate di sicurezza (DB, Deploy, Distruttivo, Costo)
- Audit anti-AI-slop integrato
- Debugging sistematico con tecnica 5 perché
- Wizard per progetti nuovi
- Adopt per progetti esistenti
- Beginner mode per utenti non-tecnici
- Multipiattaforma / porting (web → mobile → desktop)
- Reskin chirurgico con design-propagator
- Hook SessionStart per caricamento stato automatico
- Supporto italiano completo

[Unreleased]: https://github.com/mromano1398/faro-cc-plugin/compare/v1.2.0...HEAD
[1.2.0]: https://github.com/mromano1398/faro-cc-plugin/compare/v1.1.0...v1.2.0
[1.1.0]: https://github.com/mromano1398/faro-cc-plugin/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/mromano1398/faro-cc-plugin/releases/tag/v1.0.0
