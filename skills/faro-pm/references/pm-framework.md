# PM Framework — Faro

Framework completo per gestione project management e comunicazione stakeholder in Faro.

## RISK MATRIX

Classifica ogni rischio per **impatto × probabilità**:

| Impatto \ Probabilità | Bassa (10%) | Media (50%) | Alta (90%) |
|-----------------------|-------------|-------------|------------|
| **Alto** (blocca lancio) | GIALLO | ROSSO | ROSSO |
| **Medio** (ritarda 1 sprint) | VERDE | GIALLO | ROSSO |
| **Basso** (workaround OK) | VERDE | VERDE | GIALLO |

### Template rischio

```markdown
## Rischio: Integrazione pagamenti

**Severity:** ROSSO
**Impatto:** Alto — blocca monetizzazione
**Probabilità:** Alta (90%)
**Mitigazione:**
1. Parallel track: dev Marco su Stripe
2. Fallback: solo pagamento a ricezione per MVP
**Owner:** Marco
**Deadline:** 2026-05-01
**Aggiornato:** 2026-04-11
```

## LAUNCH READINESS DASHBOARD

```markdown
# Launch Readiness — [PROGETTO]
**Data lancio target:** 2026-06-15
**Giorni rimanenti:** 65

## Score complessivo: 72% pronto

## Per area

### Prodotto — 85% VERDE
- [x] Feature MVP complete
- [x] Design system applicato
- [x] Testi italiano + inglese
- [ ] Onboarding tour (in dev)

### Tecnico — 65% GIALLO
- [x] Backend stabile
- [x] DB schema finalizzato
- [ ] Load test (manca)
- [ ] Performance budget rispettato (6s LCP, target 3s)

### Sicurezza — 90% VERDE
- [x] OWASP Top 10 verificato
- [x] Security headers
- [x] Rate limiting
- [ ] Penetration test esterno (scheduled)

### Compliance — 70% GIALLO
- [x] Privacy policy
- [x] Cookie banner
- [ ] DPA con Supabase firmato
- [ ] Registro trattamenti completato

### Operations — 50% ROSSO
- [x] Monitoring base
- [ ] Runbook incidenti
- [ ] On-call rotation
- [ ] SLA definiti

### Go-to-market — 60% GIALLO
- [x] Landing page
- [ ] Email sequence onboarding
- [ ] Post social
- [ ] Comunicato stampa
```

## ROADMAP — Template quarter

```markdown
# Roadmap 2026

## Q2 2026 (Apr-Giu) — Lancio MVP
**Obiettivo:** Primi 100 utenti paganti

- Lancio pubblico 15 giugno
- Onboarding semplificato
- Pagamenti Stripe
- Dashboard utente base

**Metriche target:**
- 100 utenti paganti
- Conversion free → paid: 5%
- Churn mensile: <5%

## Q3 2026 (Lug-Set) — Consolidamento
**Obiettivo:** Retention + word of mouth

- Dashboard analytics avanzata
- Integrazione Zapier
- Mobile app iOS
- NPS survey automatica

**Metriche target:**
- 500 utenti paganti
- NPS >40
- Feature adoption: 70% utenti usano feature X

## Q4 2026 (Ott-Dic) — Scale
**Obiettivo:** Fondamenta enterprise

- SSO/SAML
- Audit log esportabile
- API pubblica v1
- Piano Enterprise

**Metriche target:**
- 1500 utenti paganti
- 3 contratti enterprise firmati
- ARR 150k€
```

## STAKEHOLDER UPDATE — Template 1 pagina

```markdown
# Update Settimanale — Settimana 15 (2026-04-11)

## Situazione in 3 parole: IN CARREGGIATA

## Avanzamento
MVP **72% completato** — in linea con piano.
Lancio confermato **15 giugno 2026**.

## Cosa abbiamo fatto questa settimana
- Integrato Stripe (pagamenti test funzionanti)
- Completato design system per dashboard
- Risolto bug critico su login mobile

## Cosa faremo la prossima settimana
- Test su flusso pagamenti completo
- Onboarding nuovo utente (tour)
- Prima demo interna martedì

## Rischi
- ROSSO: Integrazione pagamenti leggermente in ritardo (stima 3 giorni)
- GIALLO: Ancora in attesa testi definitivi dal copywriter

## Cosa ci serve da te
- Conferma palette colori finale entro venerdì
- Testi pagina "Come funziona" (stima 500 parole)

## Metriche (se già in beta)
- Utenti attivi: 45
- Sessioni settimana: 230
- Conversion signup: 12%
```

## METRICHE BUSINESS — Framework

### Per tipo progetto

| Tipo progetto | Metriche chiave |
|---------------|-----------------|
| SaaS B2C | MAU, DAU, conversion free→paid, churn, LTV, CAC |
| SaaS B2B | MRR, ARR, logo retention, NPS, expansion revenue |
| Marketplace | GMV, take rate, supply/demand ratio, repeat rate |
| Content | MAU, session time, bounce rate, ad CPM |
| Mobile app | Install, D1/D7/D30 retention, crash rate, store rating |
| Tool interno | Adozione team, task completati, errori operativi |

### Nord star metric

**Scegli UNA metrica che riflette valore consegnato:**
- Notion: "Pagine create per user"
- Slack: "Messaggi inviati"
- Airbnb: "Notti prenotate"
- Faro template: "Operazioni business completate per user per settimana"

## PRIORITIZZAZIONE — ICE score

```
ICE = (Impact × Confidence × Ease) / 3
```

- **Impact** (1-10): quanto sposta la metrica chiave?
- **Confidence** (1-10): quanto sei sicuro che funzioni?
- **Ease** (1-10): quanto è facile implementarlo? (10 = facile)

**Esempio:**
| Feature | Impact | Confidence | Ease | ICE |
|---------|--------|------------|------|-----|
| Onboarding tour | 8 | 9 | 7 | 8.0 |
| Dark mode | 4 | 10 | 9 | 7.7 |
| AI assistant | 9 | 4 | 3 | 5.3 |

Priorità: tour > dark mode > AI.

## SPRINT REPORT — Template

```markdown
# Sprint 14 Report — 2026-04-08 → 2026-04-19

## Obiettivo dichiarato
Completare modulo pagamenti end-to-end

## Risultato: PARZIALMENTE RAGGIUNTO (80%)

### Completato
- Stripe SDK integrato
- Webhook handler
- Customer portal funzionante
- Test pagamenti passano

### Non completato
- Fatturazione IVA automatica (spostato a sprint 15)

### Velocity
- Planned: 28 story points
- Done: 24 story points
- Velocity trend: stabile (ultime 3 sprint: 22, 24, 24)

### Rischi per sprint 15
- Sprint pagamenti aperto → non iniziare "altre feature" prima di chiudere
- Design tour onboarding ancora in review

### Retro highlights
**Cosa è andato bene:**
- Pair programming su webhook (Anna+Marco)
- Daily async ha ridotto call inutili

**Cosa migliorare:**
- Stima feature pagamenti troppo ottimistica
- Design review troppo lente (5 gg vs 2 target)
```

## COMUNICAZIONE LANCIO — Checklist

- [ ] Email annuncio a waiting list (template + data)
- [ ] Post Twitter/LinkedIn (1 per piattaforma)
- [ ] Product Hunt launch (prep 2 settimane prima)
- [ ] Changelog pubblico
- [ ] Video demo 90 secondi
- [ ] Screenshot marketing (3-5)
- [ ] Landing page ottimizzata (SEO + analytics)
- [ ] Supporto: email/chat attivo giorno lancio
- [ ] Monitoring attivo con alert su incidenti
- [ ] Team on-call disponibile prima ora

## KPI REVIEW — Cadenza

| Cadenza | KPI | Chi |
|---------|-----|-----|
| Daily | Errori, uptime | On-call |
| Weekly | MAU, signup, feature usage | PM + team |
| Monthly | MRR, churn, NPS | Leadership |
| Quarterly | ARR, CAC, LTV, roadmap review | Board |
