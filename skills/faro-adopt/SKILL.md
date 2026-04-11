---
name: faro-adopt
description: |
  Adotta un progetto esistente generando configurazione Faro dalle convenzioni già presenti nel codice.
  Analizza package.json, struttura, componenti, CSS, schema DB per dedurre le regole.
  Trigger: invocato da faro quando rileva codice esistente senza .claude/rules/
---

# Faro Adopt — Progetto esistente

## Step 1: Analisi profonda (silenzioso)

Leggi e analizza:
1. `package.json` / `requirements.txt` / `go.mod` → stack tecnologico
2. Struttura cartelle (`ls -R` primo livello) → architettura
3. File CSS/Tailwind config → palette, font, spacing, breakpoint
4. Componenti UI (glob `**/components/**`) → pattern UI
5. Schema DB (se esiste: prisma/schema.prisma, supabase/migrations, schema.sql) → modello dati
6. `.env.example` → servizi esterni
7. `tsconfig.json` / eslintrc → code style
8. Route/pagine → navigazione

## Step 2: Presenta l'analisi

Mostra all'utente:
```
📊 ANALISI PROGETTO

Stack:         [framework, linguaggio, DB, deploy]
Tipo dedotto:  [gestionale | SaaS | e-commerce | ...]
Tier dedotto:  [1-4] in base alle dipendenze e stili attuali
Moduli:        [N moduli/route trovati]
Design:
  Colori:      [colori estratti dal CSS/Tailwind]
  Font:        [font usati]
  Componenti:  [pattern UI trovati]
Sicurezza:     [livello attuale: basic/intermedio/avanzato]
Test:          [framework test | nessuno]
```

## Step 3: Chiedi lacune (max 3 domande)

- "È corretto ciò che ho dedotto? Cosa manca o è sbagliato?"
- "Priorità: nuove feature / fix bug / uniformare design / lancio?"
- "Vuoi formalizzare il design com'è, o vuoi cambiarlo?"

## Step 4: Proponi (come wizard Step 4)

Stesso formato della proposta, ma basato sull'analisi reale.

## Step 5: Genera file nativi DAL CODICE REALE

Invoca `faro-generator` con parametro `mode: adopt`.
Le rules generate riflettono ciò che C'È nel codice, non template generici:
- `design-system.md` usa i colori/font REALI trovati
- `architecture.md` riflette la struttura REALE
- `security.md` documenta ciò che c'è e ciò che manca

Passa al generator un oggetto tipato seguendo lo "Schema input mode=adopt" definito in
`skills/faro-generator/SKILL.md`. In particolare:
- `palette_estratta` — colori dal tailwind.config.js / CSS variables
- `font_estratti` — font da next/font, link rel, o CSS
- `stack` — framework + db + orm + auth + deploy + css + ui_library (dedotto da package.json)
- `moduli_trovati` — lista moduli con nome, path, route, tabelle DB
- `route_trovate` — lista route da src/app/ o equivalente
- `sicurezza_attuale` — stato reale (headers, validazione Zod, rate limiting, RBAC)
- `test_framework` — vitest / jest / playwright / null
- `gap_rilevati` — lista di feature mancanti che valgono la pena di segnalare

NON usare il campo `extra` come catch-all per questi dati — usa lo schema tipato.

## Riferimenti
Per pattern di deduzione per ogni framework e come estrarre informazioni, leggi `references/adopt-patterns.md`.
