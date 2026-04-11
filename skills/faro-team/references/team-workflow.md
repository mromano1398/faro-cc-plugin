# Team Workflow — Faro

Pattern completi per coordinare team 2-8 developer su progetti Faro.

## TEAM-STATE.MD — Fonte di verità

`faro/team-state.md`:

```markdown
# Team State — Aggiornato: 2026-04-11 10:30

## Sprint corrente
**Sprint 14** — 2026-04-08 → 2026-04-19
**Obiettivo:** Completare modulo pagamenti

## Developer attivi
| Dev | Branch | Task | Stato | Bloccato da |
|-----|--------|------|-------|-------------|
| Anna | feature/stripe-webhooks | Webhook handler | in review | PR #123 |
| Marco | feature/customer-portal | Portal Stripe | in dev | - |
| Luca | migration/payment-schema | Schema pagamenti | merged | - |

## PR aperte
| # | Titolo | Autore | Reviewer | Stato |
|---|--------|--------|----------|-------|
| 123 | Stripe webhook handler | Anna | Marco | changes requested |
| 124 | Customer portal link | Marco | Anna | awaiting review |

## Blocchi attivi
- Nessuno

## Note
- Release v2.3.0 prevista venerdì 2026-04-19
```

## REVIEWER MATRIX

| Area codice | Primary reviewer | Backup |
|-------------|------------------|--------|
| `prisma/schema.prisma` | Luca (DB lead) | Anna |
| `src/lib/auth/` | Anna (security lead) | Luca |
| `src/lib/stripe/` | Marco | Anna |
| `.github/workflows/` | DevOps lead | tutti |
| Business logic | Team lead | peer |

## BRANCH PROTECTION — main

```yaml
# .github/settings.yml (usa probot/settings)
branches:
  - name: main
    protection:
      required_pull_request_reviews:
        required_approving_review_count: 1
        dismiss_stale_reviews: true
        require_code_owner_reviews: true
      required_status_checks:
        strict: true
        contexts:
          - lint
          - typecheck
          - test
          - build
      enforce_admins: false
      required_linear_history: true
      allow_force_pushes: false
      allow_deletions: false
```

## CODEOWNERS

`.github/CODEOWNERS`:
```
# Default
* @faro-team/devs

# Critical areas
/prisma/ @faro-team/db-lead
/src/lib/auth/ @faro-team/security
/src/lib/stripe/ @faro-team/payments
/.github/ @faro-team/devops
/faro/state.md @faro-team/leads
```

## PR TEMPLATE

`.github/pull_request_template.md`:

```markdown
## Cosa fa questa PR
<!-- 1-2 frasi -->

## Tipo
- [ ] feat
- [ ] fix
- [ ] refactor
- [ ] docs
- [ ] test
- [ ] chore

## Checklist
- [ ] Test aggiunti/aggiornati
- [ ] CI verde
- [ ] Changelog aggiornato (se user-facing)
- [ ] Nessuna migration in questa PR (o vedi note)
- [ ] team-state.md aggiornato
- [ ] Ownership check su ogni nuova query

## Piano di rollback
<!-- Come torno indietro se va male in prod? -->

## Screenshot (se UI)
<!-- -->

## Issue correlate
Closes #
```

## CONVENTIONAL COMMITS

```
feat: aggiunge webhook Stripe per subscription.updated
fix: corregge race condition su numerazione ordini
refactor: sposta calcoloTotale in service layer
docs: aggiorna README con nuovo flusso onboarding
test: aggiunge test integration per webhook
chore: bump dependencies

feat!: rimuove endpoint legacy /api/v1/orders (breaking)
```

Enforcement con commitlint:
```json
// commitlint.config.json
{
  "extends": ["@commitlint/config-conventional"]
}
```

## SPRINT PLANNING

**Formula capacità:**
```
Capacità sprint = N developer × giorni lavorativi × 0.7
```

Il `0.7` tiene conto di:
- Meeting (15%)
- Code review (10%)
- Imprevisti (5%)

**Esempio:** 4 dev × 10 gg × 0.7 = 28 giorni-uomo effettivi.

**Priorità con MoSCoW:**
- **Must**: blocca il lancio, priorità 1
- **Should**: importante ma non bloccante
- **Could**: nice-to-have
- **Won't**: fuori sprint

## DAILY STANDUP (async)

Ogni mattina ogni dev scrive in `team-state.md`:

```markdown
## Daily 2026-04-11

### Anna
- Ieri: completato webhook handler test
- Oggi: portal integration
- Blocchi: nessuno

### Marco
- Ieri: debug race condition
- Oggi: PR review #123 + customer portal
- Blocchi: serve test subscription da Anna
```

## MIGRATION PARALLELE — Pattern completo

### Regola: branch dedicato per migration

```bash
# Developer A crea migration
git checkout -b migration/add-payment-fields
# modifica schema.prisma
npx prisma migrate dev --name "$(date +%Y%m%d_%H%M)_add_payment_fields_AM"
git add prisma/
git commit -m "feat(db): aggiunge campi pagamento"
git push
# Apri PR → merge FAST su main

# Developer A poi rebase il suo feature branch
git checkout feature/payment-api
git rebase main
```

### Shadow database per CI

```env
# .env.test
DATABASE_URL="postgresql://user:pass@localhost:5432/faro_test"
SHADOW_DATABASE_URL="postgresql://user:pass@localhost:5432/faro_shadow"
```

```json
{
  "scripts": {
    "prisma:migrate:ci": "prisma migrate deploy",
    "prisma:migrate:dev": "prisma migrate dev"
  }
}
```

### Checklist review migration PR

- [ ] Migration è reversibile (o ha piano di rollback documentato)?
- [ ] Aggiunge colonne come nullable prima di NOT NULL?
- [ ] Non droppa colonne con dati senza piano di migrazione dati?
- [ ] Non tocca tabelle con lock durante ore di punta?
- [ ] `prisma generate` è stato rieseguito dopo la migration?
- [ ] Il team è stato avvisato: "migration in arrivo, fare pull prima del prossimo sviluppo"

## RELEASE — Semver + changelog

### Bump version

```bash
npm version patch  # 2.3.0 → 2.3.1 (fix)
npm version minor  # 2.3.0 → 2.4.0 (feat)
npm version major  # 2.3.0 → 3.0.0 (breaking)
```

### Changelog automatico

Con `standard-version`:
```json
{
  "scripts": {
    "release": "standard-version && git push --follow-tags"
  }
}
```

### Release GitHub Action

```yaml
name: Release
on:
  push:
    tags: ['v*']
jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci
      - run: npm run build
      - uses: softprops/action-gh-release@v2
        with:
          generate_release_notes: true
      - name: Deploy
        run: npm run deploy:production
```

## ONBOARDING NUOVO DEVELOPER

Checklist primo giorno:

- [ ] Repo clonato, `.env.local` configurato
- [ ] `npm install && npm run dev` funziona
- [ ] DB locale seed OK
- [ ] `npm test` passa
- [ ] Accesso GitHub al repo
- [ ] Aggiunto al team CODEOWNERS
- [ ] Invitato su Slack/Discord team
- [ ] Letto: README, CLAUDE.md, faro/state.md
- [ ] Prima PR: piccola (typo, test aggiunto, doc) per familiarizzare col flusso

## MERGE CONFLICT PROTOCOL

1. Chi apre la PR risolve il conflitto (non il reviewer)
2. Su `schema.prisma`: ferma tutti, coordinare con DB lead
3. Su `package.json` lockfile: rebuild da zero (`rm -rf node_modules package-lock.json && npm install`)
4. Se dubbio sulla risoluzione: call di 5 minuti, non guessare
