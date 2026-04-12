---
name: faro-wizard
description: |
  Wizard per inizializzare un progetto nuovo con Faro. Guida l'utente dalla descrizione
  alla generazione completa di tutti i file nativi Claude Code.
  Trigger: invocato da faro quando rileva progetto nuovo
---

# Faro Wizard — Progetto nuovo

## Step 1: Raccogli la visione

Chiedi: "Descrivimi cosa vuoi costruire in 2-3 frasi."

Dalla risposta, deduci:
- **TIPO**: gestionale | SaaS | e-commerce | landing | showcase/portfolio | mobile | API | CLI/bot
- **LIVELLO UTENTE**: BEGINNER (linguaggio non tecnico) o DEV (usa termini tecnici)

Se BEGINNER → invoca `faro-beginner` e torna qui con le risposte.

Per la classificazione DEV vs BEGINNER, applica il **Classificatore v2** descritto in
`references/wizard-flow.md` sezione "Classificatore DEV / BEGINNER — v2": considera DEV anche
gli utenti che usano solo lessico di dominio professionale (DDT, bobine, DPI, prescrizione,
anamnesi, fattura elettronica, SAL, distinta base, ecc.) senza nominare framework.

## Step 2: Scegli il Tier di design

Presenta i 4 tier (leggi `faro-tier-system` per i dettagli):

| Tier | Nome | Per chi | Esempio |
|------|------|---------|---------|
| 1 | FUNZIONALE | Gestionale, ERP, tool interni | shadcn base, zero animazioni |
| 2 | PROFESSIONALE | SaaS, e-commerce, aziendale | + Framer Motion, skeleton, stile Vercel |
| 3 | CINEMATIC | Portfolio, brand, landing premium | + GSAP, Lenis, dark theme, stile Awwwards |
| 4 | IMMERSIVO | Luxury, architettura, configuratore | + Three.js, scene 3D, modelli .glb |

Chiedi: "Quale tier vuoi? (1-4)"

Opzionale: "Hai un design da Stitch, un URL di riferimento, o uno screenshot da cui partire?"
→ Se sì, invoca `faro-stitch` o analizza il riferimento per estrarre palette/font.

## Step 3: Domande specifiche (SOLO ciò che non è deducibile)

BEGINNER: max 2 domande. DEV: max 4.

**Prima domanda obbligatoria (sempre):** "Come si chiama il progetto?"
→ Se l'utente non risponde, proponi un nome dedotto dall'input iniziale (es: da "voglio un
  gestionale per i cavi ottici" → suggerisci "CaviOtticiGest" o "FibraManager").
→ Il nome finisce in `nome_progetto` nel payload al generator.
→ Usalo per tutti i placeholder `[NOME_PROGETTO]` nei file generati.

Domande possibili (scegli solo quelle necessarie):
- Chi usa l'app? (B2B, B2C, interno, pubblico)
- Modello di business? (SaaS subscription, one-time, freemium, gratuito)
- Vincoli/preferenze tecniche? (hosting, DB, auth provider)
- Budget per servizi? (gratis | <50€/mese | <200€/mese | illimitato)

## Step 4: Proposta unica

Consulta `faro-stack-advisor` per lo stack e `faro-blueprints` per l'architettura.
Presenta UN blocco unico:

```
📋 PROPOSTA FARO

Tipo:          [tipo progetto]
Tier:          [N] — [nome tier]
Stack:         [lista tecnologie principali]
Database:      [DB scelto]
Auth:          [soluzione auth]
Deploy:        [piattaforma]
Architettura:  [blueprint scelto]
Sicurezza:     Livello [1-3] integrato
Design:
  Palette:     [colori principali — MAI default]
  Font:        [font scelti — MAI Inter per Tier 3-4]
  Stile:       [riferimento visivo]

Costo stimato: [range/mese per servizi terzi]

[OK] Approva e genera  |  [M] Modifica qualcosa  |  [AP FULL] Autopilot totale
```

## Step 5: Genera tutti i file

Invoca `faro-generator` con tutti i parametri raccolti.

Se l'utente è stato classificato BEGINNER (dal Classificatore v2), passa esplicitamente
`modalita: "beginner"` al generator. Il generator applica il filtro di "traduzione" ai file
documentali (CLAUDE.md, docs/MVP.md, docs/PRD.md, docs/roadmap.md, .faro/state.md) usando il
glossario definito nella skill `faro-beginner` (file `beginner-profiles.md`). Le rules in .claude/rules/
restano tecniche.

## Step 6: Avvia costruzione

Presenta le 5 fasi di costruzione:
1. FONDAMENTA — scaffolding + sicurezza + auth + componenti base + layout
2. COSTRUZIONE — feature da PRD + audit anti-slop a fine fase
3. PRE-LANCIO — staging, Lighthouse, OWASP, SEO, a11y
4. LANCIO — deploy + smoke test (gate: conferma esplicita)
5. OPERAZIONI — monitoring, feature discovery, iterazione

Chiedi: "Da quale fase partiamo? Di solito si parte dalla 1."

## Riferimenti
Per template di domande per tipo progetto e diagramma di flusso dettagliato, leggi `references/wizard-flow.md`.
