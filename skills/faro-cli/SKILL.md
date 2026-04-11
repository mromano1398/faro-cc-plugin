---
name: faro-cli
description: |
  Pattern CLI e bot: parsing argomenti, output colorato, interattività, bot Telegram/Discord.
  Trigger: "cli", "command line", "bot telegram", "bot discord", "script"
---

# faro-cli — CLI · Bot · Automazioni · Script · Background Services

**Lingua:** Sempre italiano. Riferimenti ufficiali:
- Commander.js: https://github.com/tj/commander.js
- Typer (Python): https://typer.tiangolo.com
- Discord.js: https://discord.js.org/docs
- Telegraf (Telegram): https://telegraf.js.org
- GitHub Actions: https://docs.github.com/en/actions

## QUANDO USARE

- CLI distribuita via npm o binario
- Script di automazione (Node.js o Python)
- Bot Discord o Telegram
- GitHub Action personalizzata
- Cron job schedulato

## FLUSSO DECISIONALE

```
CLI npm → Node.js + Commander.js + TypeScript
CLI binario → Go (unico binario cross-platform)
Script automazione → Python + Typer (se team Python) o Node.js
Bot Discord → Node.js + Discord.js v14
Bot Telegram → Node.js + Telegraf v4
GitHub Action → Node.js action + workflow YAML
Cron job → GitHub Actions schedule o /api/cron + cron-job.org
```

## REGOLE CHIAVE

1. **`#!/usr/bin/env node`** in cima al file entry point — obbligatorio
2. **`bin` field in `package.json`** con path corretto al file compilato
3. **`--help` funzionante** per ogni comando e sottocomando
4. **Exit code 1 sugli errori** — tool CI-friendly, mai swallowcare eccezioni
5. **Nessun secret hardcodato** — usa variabili d'ambiente
6. **Token bot in variabile d'ambiente** — mai nel codice o in git
7. **Graceful shutdown** per bot (SIGINT/SIGTERM) — `bot.stop()`
8. **`npm publish --dry-run`** prima di pubblicare su npm
9. **GitHub Secrets** per variabili sensitive nei workflow, mai in YAML
10. **`process.exit(1)`** nello script cron se errore — GitHub Action fallisce e notifica

## FLUSSO OPERATIVO

1. **Scegli runtime**: Node.js (general), Python (data/scripting team), Go (binario)
2. **Parser argomenti**: Commander.js / Typer / cobra
3. **Output colorato**: chalk / rich / lipgloss
4. **Interattività** (se serve): inquirer / questionary
5. **Spinner/progress**: ora / rich.progress
6. **Config persistente**: conf (Node) / pydantic-settings (Python)
7. **Test** per ogni comando principale
8. **Distribuzione**: npm / PyPI / GitHub releases binari

## CHECKLIST

- [ ] `#!/usr/bin/env node` in cima al file entry point
- [ ] `bin` field in `package.json` con path corretto
- [ ] `--help` funzionante per ogni comando e sottocomando
- [ ] Gestione errori con exit code 1 (tool CI-friendly)
- [ ] Nessun secret hardcodato (usa variabili d'ambiente)
- [ ] Test per ogni comando principale
- [ ] [Bot] Token in variabile d'ambiente, mai nel codice
- [ ] [Bot] Gestione graceful shutdown (SIGINT/SIGTERM)
- [ ] [CLI npm] `npm publish --dry-run` prima di pubblicare
- [ ] [GitHub Action] Segreti in GitHub Secrets, non in workflow YAML

## TABELLA — Scelta tecnologica

| Caso | Tecnologia | Note |
|------|------------|------|
| CLI dev tool | Node.js + Commander | Distribuzione npm |
| CLI end user | Go | Binario cross-platform |
| Automation script | Python + Typer | Se team Python |
| Bot Discord | Discord.js v14 | Slash commands nativi |
| Bot Telegram | Telegraf v4 | Session built-in |
| GitHub Action | Node.js | TypeScript + @actions/core |
| Cron task | GitHub Schedule | Gratis, semplice |

## REFERENCES

Per dettagli tecnici completi, leggi:
- [references/cli-patterns.md] — CLI Node+Commander, Python Typer, bot Discord/Telegram, GitHub Action, pubblicazione npm, cron
