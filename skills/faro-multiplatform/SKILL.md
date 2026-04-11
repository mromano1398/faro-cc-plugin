---
name: faro-multiplatform
description: |
  Porta un progetto su una nuova piattaforma: webapp → mobile, webapp → desktop,
  mobile → webapp, cross-platform.
  Trigger: "porta su mobile", "versione desktop", "cross-platform", "porting"
---

# Faro Multiplatform — Porting

## Percorsi supportati

| Da | A | Strategia |
|----|---|-----------|
| Next.js webapp | Expo mobile | Condividi logica/tipi, nuova UI nativa |
| Next.js webapp | Electron desktop | Wrapper Electron o Tauri |
| Expo mobile | Next.js webapp | Condividi logica, nuova UI web |
| Qualsiasi | PWA | Aggiungi manifest + service worker |

## Flusso

1. Analizza il progetto sorgente (codice, struttura, dipendenze)
2. Identifica cosa è PORTABILE (logica, tipi, validazione, API) vs cosa va RISCRITTO (UI)
3. Proponi strategia con effort stimato
4. Se approvato: genera la struttura del progetto target
5. Configura monorepo se necessario (turborepo o nx)
6. Genera rules per la nuova piattaforma
   - Delega a `faro-blueprints/references/mobile.md` (o desktop.md / web.md a seconda del target)
   - Invoca `faro-generator` con i parametri:
     - `nome_progetto`: "<nome>-mobile" (o desktop/web)
     - `tipo_progetto`: "mobile" (o "desktop")
     - `tier`: normalmente 2 (mobile standard) o lo stesso del sorgente
     - `blueprint`: "mobile" / "desktop"
     - `mode`: "new"
     - `target_directory`: `apps/mobile/` (invece di root)
   - Il generator produce `.claude/rules/` dentro `apps/mobile/.claude/rules/` (non root)
   - Le rules generate per mobile NON devono contenere regole web-specific (next/image, kebab-case .tsx non applicabili in Expo senza adattamento — filtro per tipo del B1-010)
7. Aggiorna il monorepo root CLAUDE.md con una sezione "Progetti":
   - `apps/web/` (Next.js) — vedi `apps/web/CLAUDE.md`
   - `apps/mobile/` (Expo) — vedi `apps/mobile/CLAUDE.md`
8. Linka le skill di dominio pertinenti nel README del target: faro-mobile per mobile, faro-payments con nota IAP per mobile con pagamenti

## Principio
La LOGICA è condivisa. La UI è riscritta per la piattaforma.
I tipi TypeScript e gli schema Zod sono il ponte tra le piattaforme.

## Riferimenti
Per strategie di porting dettagliate per ogni percorso, leggi `references/porting-strategies.md`.
