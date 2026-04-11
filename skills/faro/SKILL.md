---
name: faro
description: |
  Orchestratore principale del plugin Faro. Si attiva quando l'utente vuole inizializzare un progetto,
  riprendere il lavoro, cambiare design, valutare lo stack, o usare l'autopilot.
  Trigger: "faro", "inizializza", "nuovo progetto", "riprendi", "reskin", "cambia design",
  "autopilot", "da dove parto", "stato progetto", "adotta", "configura progetto"
---

# Faro — Orchestratore

## Step 0: Orientamento silenzioso

PRIMA di rispondere, leggi silenziosamente:
1. `CLAUDE.md` (se esiste)
2. `.claude/rules/` (se esistono)
3. `.faro/state.md` (se esiste)
4. `package.json` (se esiste)
5. Struttura cartelle (ls -la, ls src/ o app/)

## Step 1: Rileva percorso

In base a ciò che trovi:

### NUOVO (cartella vuota o quasi, nessun .claude/rules/)
→ "Ciao! Vedo che è un progetto nuovo. Descrivimi cosa vuoi costruire in 2-3 frasi."
→ Invoca skill `faro-wizard`

### ESISTENTE (codice presente, nessun .claude/rules/)
→ "Vedo codice esistente ma nessuna configurazione Faro. Analizzo il progetto per generare le regole."
→ Invoca skill `faro-adopt`

### RIPRENDI (.faro/state.md esiste)
→ Leggi stato, mostra: "Progetto [nome], Fase [N], Score [N]/100. Piano attivo: [nome] — step [N/M]."
→ Se in `.faro/state.md` trovi una sezione `## Recovery point` valorizzata, chiedi:
   "Vedo che la sessione precedente si è interrotta a [task N — titolo]
   (ultimo build check: [pass/fail]). Vuoi riprendere da lì, o fare altro?"
→ Altrimenti: "Cosa vuoi fare? Continuare il piano, o qualcos'altro?"

### RESKIN (utente dice "cambia design", "reskin", "nuovo stile")
→ Invoca skill `faro-tier-system` per scelta nuovo tier
→ Poi `/project:reskin` per propagare

### AUTOPILOT (utente dice "autopilot", "vai", "fai tutto")
→ Invoca skill `faro-autopilot`

## Step 2: Menu (se richiesto o se nessun percorso è chiaro)

```
[INIT]    Inizializza nuovo progetto     → faro-wizard
[ADOPT]   Adotta progetto esistente      → faro-adopt
[STATUS]  Stato e score                  → /project:status
[TIER]    Cambia tier design             → faro-tier-system
[STACK]   Valuta/cambia stack            → faro-stack-advisor
[RESKIN]  Cambia design senza rompere    → /project:reskin
[AUDIT]   Audit completo                 → /project:audit
[ADVISOR] Consulente prodotto            → /project:advisor
[AP]      Autopilot prossimo task        → faro-autopilot
[AP FULL] Autopilot completo (con gate)  → faro-autopilot --full
[PORT]    Porta su altra piattaforma     → faro-multiplatform
```

## Regole assolute

1. MAI modificare/creare database senza conferma esplicita dell'utente
2. MAI fare git push senza richiesta esplicita
3. Lancia build-checker dopo ogni 3 file modificati
4. Lancia context-updater dopo ogni piano COMPLETE
5. Prima di modificare file, LISTA i file che vuoi toccare e chiedi conferma
6. MAI modificare file non esplicitamente richiesti (anti-scope-creep)
7. Dopo 3 fallimenti sullo stesso errore → STOP, mostra cosa hai provato, chiedi input

## Sistema piani

I piani vivono in `.faro/plans/`. Formato completo in
`.claude/rules/workflow.md` sezione "Piano prima di implementare".

Header minimo:
```
# Piano: [nome]
Data: [YYYY-MM-DD]
Tipo: [feature|fix|refactor|reskin|chore]
Riferimento: docs/PRD.md -> [RF-001, RF-002]
Stato: [bozza|approvato|in esecuzione|completato]

## Task
- [ ] Task 1: [titolo]
  File: path/al/file.ts
  Cosa fare: [descrizione precisa]
  Verifica: [comando/condizione osservabile]
  Dipendenze: [task precedenti o "nessuna"]
```

## Sistema log

Log in `.faro/log.md`. Formato append-only:
```
## [YYYY-MM-DD HH:mm]
**Azione:** [cosa è stato fatto]
**File:** [file toccati]
**Risultato:** [esito]
```

## Score progetto

Calcola score /100:
- Fondamenta (20): scaffolding, lint, TS strict, git
- Sicurezza (20): headers, auth, validazione, IDOR
- Design (20): design-system rule, coerenza, anti-slop
- Funzionalità (20): feature completate vs PRD
- Qualità (20): test, build clean, no TODO/FIXME, a11y

## Menu dettagliato
Per il dettaglio completo di ogni voce del menu, quando serve, leggi `references/menu-completo.md`.
