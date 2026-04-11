# Contribuire a Faro

Grazie per voler contribuire! Faro è un plugin universale vendor-neutral per Claude Code, scritto in italiano.

## Filosofia del plugin

Prima di aprire una PR, assicurati che il contributo sia allineato con la filosofia di Faro:

1. **Init + advisor + ritirati** — Faro configura il progetto e poi lascia Claude Code lavorare da solo leggendo `.claude/rules/`
2. **Vendor-neutral** — Faro NON integra feature vendor-specifiche (niente tight coupling a Vercel, AWS, specifici marketplace)
3. **Italiano** — trigger, documentazione e prompts sono in italiano
4. **File nativi Claude Code** — Faro genera file `.claude/rules/`, `.claude/commands/`, `.claude/agents/` che funzionano senza dipendere dal plugin installato
5. **Fuori scope deliberato** — browser QA, token compression binaria, parallelismo massivo e mockup AI sono decisioni esplicite (vedi FARO-VS-IL-RESTO.md)

## Tipi di contributo benvenuti

- ✅ Nuove combo stack per tipi di progetto esistenti
- ✅ Nuovi blueprint settoriali (es: edilizia, farmacia, legal-tech)
- ✅ Pattern per framework/librerie non ancora supportati (Tier 2 o 3)
- ✅ Fix di drift cross-reference tra file
- ✅ Traduzioni migliori del glossario beginner
- ✅ Aggiunte al classificatore DEV/BEGINNER (nuovi settori verticali)
- ✅ Miglioramenti ai template dei file generati
- ✅ Bug fix e correzione refusi

## Tipi di contributo NON benvenuti

- ❌ Feature che richiedono runtime esterni (Bun, Deno, Playwright, browser headless)
- ❌ Integrazioni vendor-specifiche in skill core (vanno in plugin separati)
- ❌ Traduzioni complete in altre lingue (Faro è italiano per design — aprire issue per discuterne prima)
- ❌ Rimozione di skill esistenti senza deprecation path
- ❌ Breaking change in `templates-*.md` senza bump major (cambierebbe i file generati per progetti esistenti)

## Come strutturare una PR

### 1. Issue prima
Apri un'issue descrittiva PRIMA di lavorare su PR grandi (>100 righe o cambio struttura). Per fix piccoli puoi aprire PR diretta.

### 2. Branch
```bash
git checkout -b feature/nome-breve    # feature nuova
git checkout -b fix/nome-breve        # bug fix
git checkout -b docs/nome-breve       # solo documentazione
```

### 3. Coerenza
- Non rompere le 24 skill esistenti
- Non introdurre `omega` residui (Faro è il successore di omega)
- Non aggiungere emoji nei template generati (solo quelli esplicitamente richiesti)
- Italiano coerente in tutti i file del plugin
- Frontmatter YAML valido su ogni SKILL.md modificato

### 4. Verifica locale
Prima di pushare:

```bash
# Verifica JSON validi
node -e "JSON.parse(require('fs').readFileSync('.claude-plugin/plugin.json','utf8'))"
node -e "JSON.parse(require('fs').readFileSync('hooks/hooks.json','utf8'))"

# Verifica che ogni skill in plugin.json esista come directory
# Verifica frontmatter --- su ogni SKILL.md
# Verifica nessun residuo "omega" nelle skill
grep -ri "omega" skills/ || echo "OK: zero omega"
```

Il CI GitHub Actions farà queste verifiche automaticamente.

### 5. Commit
Usa conventional commits:

```
feat: aggiunge combo stack SaaS + AI in recommendations.md
fix: drift cross-reference tra generator e adopt
docs: espande FAQ con sezione disinstallazione
refactor: semplifica template design-system Tier 2
```

### 6. PR
- Titolo: conventional (come sopra)
- Descrizione: spiega COSA e PERCHÉ, non solo COSA
- Screenshots / before-after se tocchi UI dei template
- Checklist: ho letto CONTRIBUTING.md, non ho introdotto gap cross-reference, ho verificato localmente

## Struttura del plugin

```
faro-claude-plugin/
├── .claude-plugin/
│   └── plugin.json           # Lista 24 skill
├── hooks/
│   └── hooks.json            # SessionStart hook
├── commands/
│   └── fetch-docs.md         # Command globale /faro:fetch-docs
├── skills/                   # 24 skill
│   ├── faro/                 # Orchestratore
│   ├── faro-wizard/          # Wizard progetto nuovo
│   ├── faro-adopt/           # Adopt progetto esistente
│   ├── faro-beginner/        # Flusso non-tecnico
│   ├── faro-tier-system/     # 4 tier design
│   ├── faro-stack-advisor/   # 80+ tecnologie
│   ├── faro-blueprints/      # 8 blueprint architetturali
│   ├── faro-generator/       # Generatore file nativi (CUORE)
│   ├── faro-autopilot/       # Esecuzione autonoma con gate
│   ├── faro-multiplatform/   # Porting cross-platform
│   ├── faro-stitch/          # Integrazione Stitch (opzionale)
│   ├── faro-devops/          # Pattern Docker + CI/CD
│   ├── faro-security/        # OWASP audit
│   ├── faro-testing/         # Pattern di test
│   ├── faro-legacy/          # Migrazione legacy
│   ├── faro-supabase/        # Pattern Supabase
│   ├── faro-multitenant/     # Tenant isolation
│   ├── faro-mobile/          # Expo/React Native
│   ├── faro-ai/              # LLM + streaming + RAG
│   ├── faro-payments/        # Stripe + web vs IAP
│   ├── faro-python/          # FastAPI + SQLAlchemy
│   ├── faro-cli/             # CLI + bot
│   ├── faro-team/            # Multi-dev workflow
│   └── faro-pm/              # Roadmap + prioritizzazione
├── README.md
├── CHANGELOG.md
├── CONTRIBUTING.md
├── LICENSE
└── .gitignore
```

## Versionamento

Faro segue [Semantic Versioning](https://semver.org/lang/it/):

- **MAJOR** (2.0.0) — breaking change nei file `.claude/rules/` generati che richiederebbero aggiornamento dei progetti esistenti (es: rinomina di una rule, rimozione di un comando)
- **MINOR** (1.x.0) — aggiunta di skill, template, pattern senza rompere i progetti esistenti. Additivo.
- **PATCH** (1.0.x) — bug fix, correzione refusi, miglioramenti interni che non cambiano il comportamento dei file generati

Le release sono annotate con tag `vX.Y.Z` e documentate in `CHANGELOG.md`.

## Codice di condotta

Sii rispettoso, costruttivo e specifico nei feedback. Le PR vengono valutate sul merito tecnico. Le review non sono personali.

## Domande?

Apri un'issue con label `question`. Risposte tipiche entro qualche giorno.

## Licenza

Contribuendo accetti che il tuo contributo venga rilasciato sotto la [MIT License](./LICENSE) del progetto.
