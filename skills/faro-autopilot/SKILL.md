---
name: faro-autopilot
description: |
  Esecuzione autonoma dei task con gate obbligatori. Può eseguire un piano intero
  senza intervento umano, fermandosi solo ai gate di sicurezza.
  Trigger: "autopilot", "vai", "fai tutto", "esegui il piano", invocato dal menu faro
---

# Faro Autopilot

## Modalità

### AP (singolo task)
Esegui il prossimo task del piano attivo.

### AP FULL (piano intero)
Esegui TUTTI i task del piano attivo, fermandoti solo ai gate.

## 4 Gate obbligatori (STOP e chiedi conferma)

1. **Gate DB** — Prima di creare/modificare tabelle database
2. **Gate Deploy** — Prima di deploy in produzione
3. **Gate Distruttivo** — Prima di eliminare file/dati/funzionalità
4. **Gate Costo** — Prima di aggiungere servizi a pagamento

## Flusso AP FULL

```
Per ogni task nel piano:
  1. Lancia task-executor con il task
  2. Dopo completamento: build-checker
  3. Se gate → STOP, mostra contesto, chiedi conferma
  4. Ogni 3 task: build-checker completo
  5. Ogni 5 task: code-reviewer
  6. Fine piano: code-reviewer completo + context-updater
```

## Auto-audit a fine fase

Quando AP FULL completa l'ultimo piano di una fase della roadmap
(in particolare Fase 2 — Costruzione feature), NON segnalare la fase come
completata senza audit.

Sequenza obbligatoria:
1. Lancia `/project:audit` automaticamente
2. Leggi il report:
   - Se **audit design = AI SLOP** -> crea automaticamente un piano di fix in
     `.faro/plans/fix-design-slop.md`, mostralo all'utente, chiedi conferma
     ed eseguilo prima di passare alla fase successiva
   - Se **audit sicurezza contiene issue CRITICHE** -> crea piano di fix
     obbligatorio in `.faro/plans/fix-security-critical.md`, richiedi conferma,
     eseguilo
   - Se **audit PASS** -> aggiorna `.faro/state.md` (fase completata,
     recovery point) e procedi
3. Non passare alla fase successiva senza audit PASS o piani di fix eseguiti
   e verificati con build-checker verde

## Anti-loop
- Se lo stesso errore appare 3 volte → STOP
- Se un task impiega più di 15 minuti → avvisa l'utente
- Se il context window è al 70%+ → proponi /compact

## Log
Ogni azione viene loggata in .faro/log.md con timestamp e risultato.

## Riferimenti
Per esempi dettagliati di ogni gate, leggi `references/autopilot-gates.md`.
