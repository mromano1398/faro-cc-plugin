---
name: faro-team
description: |
  Workflow multi-developer: branch strategy, PR, code review, onboarding.
  Trigger: "team", "PR workflow", "onboarding developer", "branch strategy"
---

# faro-team — Coordinamento Team · PR · Sprint · Multi-piano

**Lingua:** Sempre italiano.
**Contesto:** Questa skill estende Faro per team da 2 a 8 developer. Non sostituisce il workflow Git — lo orchestra e lo documenta.

## QUANDO USARE

- Team di 2+ developer su stesso progetto
- Coordinamento branch paralleli con dipendenze
- Gestione PR review e approvazioni
- Sprint planning e assegnazione task
- Standup e stato giornaliero
- Release con tag + changelog

## FLUSSO DECISIONALE

```
Nuova feature → crea piano in faro/plans/team/[developer]_[branch].md
                → aggiorna team-state.md

PR aperta → assegna reviewer dalla matrice
          → aggiorna stato in team-state.md
          → blocco se dipendenze non risolte

Sprint → leggi PRD → stima capacità (N dev × giorni × 0.7)
       → prioritizza → assegna → aggiorna team-state.md

Release → conventional commits → changelog → tag semver → deploy
```

## REGOLE CHIAVE

1. **Un piano per developer per branch** — in `faro/plans/team/`
2. **Non auto-approvarti** — chi scrive il codice non fa review da solo
3. **Branch vita max 3 giorni** — branch più lunghi = conflitti certi
4. **Rebase frequente** — almeno ogni giorno lavorativo
5. **schema.prisma: MAI merge automatico** — confronta manualmente
6. **Reviewer senior su codice critico** — DB, auth, pagamenti
7. **CI verde prima del merge** — build + test obbligatori
8. **Aggiorna team-state.md ad ogni cambio** — fonte di verità del team
9. **Blocchi comunicati subito** — non aspettare lo standup
10. **Tag semver per ogni release** — triggera deploy automatico

## FLUSSO OPERATIVO

1. **Setup team**: crea `faro/team-state.md`, matrix reviewer, branch protection
2. **Sprint planning**: leggi PRD, stima capacità, assegna task
3. **Daily**: ogni dev aggiorna il proprio piano + team-state
4. **PR**: apri con template, reviewer dalla matrix, CI verde
5. **Merge**: rebase su main, squash commit con conventional message
6. **Release**: tag semver, changelog, deploy

## CHECKLIST

- [ ] `faro/team-state.md` creato e aggiornato
- [ ] Branch protection configurata su main
- [ ] Reviewer matrix definita
- [ ] PR template in `.github/pull_request_template.md`
- [ ] Sprint backlog assegnato con stime
- [ ] Nessun branch più vecchio di 3 giorni senza merge/rebase
- [ ] Tutti i PR con almeno 1 review approvata prima del merge
- [ ] CHANGELOG aggiornato ad ogni release

## MIGRATION PARALLELE — Regola d'oro

Il problema più comune in team con Prisma: due developer creano migration su branch diversi
che poi collidono su `prisma/migrations/` al merge.

```
feature/add-payment-fields     ← solo codice applicativo
migration/add-payment-fields   ← solo schema.prisma + migration
```

La migration viene mergiata PRIMA del branch feature. Il feature branch poi fa rebase.

Naming convention: `YYYYMMDD_HHMM_nome_feature_INITIALS`

## REFERENCES

Per dettagli tecnici completi, leggi:
- [references/team-workflow.md] — PR workflow, branch protection, reviewer matrix, sprint planning, team-state.md format, migration parallele, release automation
