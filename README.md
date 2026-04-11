# 🔦 Faro — Illumina il tuo progetto dall'idea alla produzione

Faro è un plugin universale per Claude Code che inizializza, guida e accompagna qualsiasi progetto software. Dal gestionale all'esperienza 3D immersiva.

## Filosofia

Faro NON è un runtime da invocare ogni volta. È un **inizializzatore + advisor**:

1. **Configura** il progetto generando file nativi Claude Code (`.claude/rules/`, `commands/`, `agents/`)
2. Una volta configurato, **Claude Code segue le regole da solo** — senza bisogno di invocare Faro
3. L'**Advisor** con memoria persistente ti accompagna nel tempo

## Installazione

### Sessione singola
```bash
/plugin install faro@[marketplace]
```

### Permanente
```bash
claude plugin add faro
```

### Directory plugins
Clona in `~/.claude/plugins/faro-claude-plugin/`

## Come si usa

### 4 percorsi + porting

| Percorso | Quando | Comando |
|----------|--------|---------|
| Nuovo | Cartella vuota | `faro` → wizard automatico |
| Esistente | Codice senza Faro | `faro` → adopt automatico |
| Riprendi | .faro/ esiste | `faro` → riprende |
| Reskin | Cambiare design | `/project:reskin` |
| Port | Nuova piattaforma | Invoca `faro-multiplatform` |

### 4 tier di design

| Tier | Nome | Per chi | Animazioni |
|------|------|---------|------------|
| 1 | Funzionale | Gestionale, ERP | Zero |
| 2 | Professionale | SaaS, e-commerce | Framer Motion sottili |
| 3 | Cinematic | Portfolio, brand | GSAP + Lenis |
| 4 | Immersivo | Luxury, architettura | Three.js + 3D |

### Comandi progetto (generati nel progetto)

| Comando | Cosa fa |
|---------|---------|
| `/project:status` | Stato, score, piano |
| `/project:audit` | Audit design + sicurezza + performance + a11y |
| `/project:reskin` | Cambia design senza rompere logica |
| `/project:advisor` | Consulente prodotto con memoria |
| `/project:review` | Review coerenza con rules |
| `/project:update-context` | Aggiorna rules dopo modifiche |
| `/project:timeline` | Report cronologico |
| `/project:explore` | Mappa codebase |
| `/project:debug` | Debugging guidato |

## Skill del plugin

### Orchestrazione (11)
faro, faro-wizard, faro-adopt, faro-beginner, faro-tier-system, faro-stack-advisor, faro-blueprints, faro-generator, faro-autopilot, faro-multiplatform, faro-stitch

### Dominio (13)
faro-devops, faro-security, faro-testing, faro-legacy, faro-supabase, faro-multitenant, faro-mobile, faro-ai, faro-payments, faro-python, faro-cli, faro-team, faro-pm

## File generati nel progetto (33 file)

11 rules + 10 commands + 6 agents + 3 docs + 2 tracking + CLAUDE.md = 33 file nativi Claude Code

## Faro sostituisce

Faro è progettato per essere l'UNICO plugin di cui hai bisogno:

| Plugin | Sostituito? | Come |
|--------|------------|------|
| superpowers | ✅ Sì | Brainstorming (workflow.md), TDD (testing.md), piani (.faro/plans/), review (code-reviewer), worktree (workflow.md), debugging (debugging.md) |
| frontend-design | ✅ Sì | 4 tier design + audit anti-AI-slop |
| claude-mem | ✅ Sì | Advisor con memoria + /project:timeline + /project:explore |
| claude-code-setup | ✅ Sì | faro-wizard |
| claude-hud | ✅ Sì | /project:status |
| claude-md-management | ✅ Sì | CLAUDE.md generato e aggiornato da context-updater |
| commit-commands | ✅ Sì | Git workflow in code-style.md + /project:commit |
| omega | ✅ Sì | Faro è il successore |
| vercel-plugin | ❌ Tieni | Plugin vendor-specifico per Vercel — Faro non sostituisce tool vendor-specifici |

### Per disinstallare i plugin sostituiti

```bash
/plugin remove superpowers
/plugin remove frontend-design
/plugin remove claude-mem
/plugin remove claude-code-setup
/plugin remove claude-hud
/plugin remove claude-md-management
/plugin remove commit-commands
/plugin remove omega
```

### Plugin complementari consigliati (opzionali)

| Plugin | Perché |
|--------|--------|
| vercel-plugin | Se usi Vercel per il deploy |
| Token Optimizer | Se vuoi controllo avanzato sul context window |
| gstack | Se vuoi browser QA headless automatizzato (feature fuori scope Faro) |

## FAQ

### Devo invocare Faro ogni volta che lavoro al progetto?
No. Una volta configurato, Claude Code legge automaticamente `.claude/rules/` ad ogni messaggio.
Faro serve per inizializzare, adottare, cambiare design o riprendere — non per tutte le interazioni.

### Posso disinstallare Faro dopo l'inizializzazione?
Sì. Le regole in `.claude/rules/`, i comandi in `.claude/commands/`, gli agent in `.claude/agents/`
e i documenti in `docs/` e `.faro/` sono file nativi Claude Code. Continuano a funzionare senza
il plugin Faro installato. Reinstallerai Faro solo se vorrai un nuovo reskin, adopt o porting.

### Funziona con progetti esistenti?
Sì — usa il percorso `adopt`. Faro analizza il codice esistente (package.json, struttura cartelle,
Tailwind config, schema DB) e genera rules che RIFLETTONO lo stato attuale, non template generici.

### Cosa succede se il mio progetto usa uno stack non supportato?
Faro supporta 80+ tecnologie in 3 tier di supporto (pattern completi, pattern base, guida generica).
Se la tua tech è in Tier 3, Faro ti darà consigli generali e rimanderà alla documentazione ufficiale
via `/faro:fetch-docs`. Puoi sempre adattare i template a mano.

### Posso cambiare design (reskin) senza rompere la logica?
Sì — usa `/project:reskin`. Faro classifica ogni file del progetto in DESIGN / LOGICA / MISTO
e tocca solo le classi Tailwind e i componenti UI. I file `lib/`, `actions/`, `queries/`, `auth/`,
`validations/`, `prisma/`, `api/` non vengono mai toccati durante un reskin.

### Come funziona l'autopilot?
`faro-autopilot` esegue un piano di lavoro autonomamente, fermandosi a 4 gate obbligatori:
- **Gate DB**: prima di creare/modificare tabelle database
- **Gate Deploy**: prima di deployare in produzione
- **Gate Distruttivo**: prima di eliminare file/dati/funzionalità
- **Gate Costo**: prima di aggiungere servizi a pagamento

In più, anti-loop automatico: se lo stesso errore compare 3 volte, l'autopilot si ferma e chiede input.

### Posso usare Faro con utenti non-tecnici?
Sì — il wizard rileva il livello dell'utente dalla prima risposta. Se sei un utente non-tecnico,
invoca il flusso **BEGINNER** che usa linguaggio semplice, max 3 domande, 6 profili preconfezionati
e 4 livelli visivi (Semplice / Bello / Wow / 3D invece di Tier 1-4).

### Faro funziona in inglese?
Il plugin è pensato in italiano (trigger, documentazione, prompts). Puoi usarlo con Claude Code in
qualsiasi lingua, ma le rules generate saranno in italiano per default. Per un progetto in inglese,
chiedi esplicitamente "rules in english" al wizard.

### Come si integra con il plugin vercel-plugin?
Sono complementari. Faro gestisce inizializzazione, design system, sicurezza, review e workflow
generali. vercel-plugin gestisce deploy, functions, env variables, rollback, domini e altre feature
vendor-specifiche di Vercel. Mantieni entrambi installati se usi Vercel.

### Dove va la memoria dell'advisor?
`faro-advisor` usa `memory: project` (feature nativa Claude Code) per ricordare decisioni, pattern
e pitfall tra le sessioni. Le learning strutturate vanno in `.faro/learnings/` (append-only, filesystem,
nessun MCP server richiesto).

### Posso contribuire al plugin?
Sì. Il plugin è open source. Proponi pull request su pattern di dominio mancanti (es: nuove combo
stack, nuovi blueprint settoriali), ma evita di aggiungere feature fuori scope (browser QA,
token compression binaria, parallelismo multi-sprint massivo) — quelle sono scelte di posizionamento
deliberate.

### Il plugin è davvero necessario se Claude Code ha già tutti gli strumenti?
Faro non aggiunge strumenti a Claude Code. Li ORCHESTRA. La differenza: senza Faro, ogni progetto
parte da zero — Claude deve chiedere ad ogni sessione quali sono i colori, la struttura, la sicurezza,
il workflow. Con Faro, queste informazioni vivono in `.claude/rules/` e Claude le legge in automatico.
Meno prompt ingegneria, più lavoro reale.

## Changelog

Vedi [CHANGELOG.md](./CHANGELOG.md) per la storia completa delle release.

Ultima release: **v1.1.0** (2026-04-11) — 23 fix post-release iniziale, nessun breaking change.

## Licenza

MIT
