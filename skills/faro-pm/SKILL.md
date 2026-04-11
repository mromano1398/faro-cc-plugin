---
name: faro-pm
description: |
  Vista business e project management: roadmap, prioritizzazione, stakeholder, metriche.
  Trigger: "roadmap", "priorità", "sprint planning", "stakeholder update"
---

# faro-pm — Vista Business · Status · Rischi · Lancio

**Lingua:** Sempre italiano.
**Principio:** Nessun termine tecnico senza spiegazione. Il PM non sa cosa sia un advisory lock o una migration — sa se il progetto è in ritardo, quali rischi ci sono, e quando si lancia.

## QUANDO USARE

- PM o stakeholder chiede stato del progetto
- Richiesta report per superiori o cliente
- Valutazione "siamo pronti al lancio?"
- Analisi rischi in linguaggio business
- Sprint report settimanale non tecnico
- Prioritizzazione roadmap

## FLUSSO — Come generare la vista PM

```
1. Leggi: faro/state.md + faro/log.md (ultime 30 righe) + faro/PRD.md + faro/roadmap.md
2. Genera Dashboard PM con: stato generale verde/giallo/rosso, avanzamento %, rischi attivi
3. Traduci TUTTI i termini tecnici in linguaggio business
4. Valuta launch readiness: quanti punti della checklist sono verdi?
5. Se richiesto: report per stakeholder (1 pagina, no tecnico)
```

## REGOLE CHIAVE

1. **Zero gergo tecnico** senza spiegazione immediata
2. **Ogni problema tecnico = rischio business** — traduci sempre
3. **Semaforo sempre presente** — VERDE / GIALLO / ROSSO
4. **% completamento sempre visibile** — il PM non legge log tecnici
5. **Data lancio o "non definita"** — mai rispondere senza una stima
6. **Rischi ordinati per impatto** — critici prima, medi dopo
7. **"Cosa ci serve" esplicito** — decidi se serve una decisione del cliente/PM
8. **Roadmap per trimestre** — max 3 trimestri nel futuro
9. **Metriche business visibili** — MAU, conversion, churn, NPS se applicabili
10. **Mai promettere date che non puoi mantenere**

## DASHBOARD PM — Template output

```markdown
# Dashboard Progetto — [NOME]
**Aggiornato:** [DATA]
**Stato generale:** VERDE / GIALLO / ROSSO

## Avanzamento
- MVP completato: 72%
- Data lancio stimata: 2026-06-15
- Sprint corrente: 14 di 20

## Rischi attivi
1. ROSSO — Integrazione pagamenti in ritardo (3 gg)
2. GIALLO — Design non ancora approvato
3. VERDE — Test suite coverage 85%

## Cosa ci serve
- Decisione: palette finale (deadline venerdì)
- Feedback: demo prototipo con 3 utenti

## Prossima milestone
"Beta interna" — 2026-05-01
```

## TABELLA TRADUZIONE TECNICO → BUSINESS

| Termine tecnico | Traduzione PM |
|-----------------|---------------|
| Migration DB rotta | "Database non aggiornato, feature X ferma" |
| CI red | "Il codice non compila, blocco rilasci" |
| Advisory lock | "Protezione contro ordini duplicati" |
| Race condition | "Due utenti possono creare lo stesso ordine" |
| Technical debt | "Lavoro di manutenzione posticipato" |
| Coverage < 80% | "Mancano test su aree critiche" |
| Feature flag | "Interruttore per attivare la feature solo ad alcuni" |
| Rollback | "Torno alla versione precedente" |
| Deploy bloccato | "Non possiamo pubblicare la nuova versione" |

## FLUSSO OPERATIVO

1. **Leggi state**: `faro/state.md`, `log.md`, `PRD.md`, `roadmap.md`
2. **Calcola avanzamento**: task completati / task totali sprint
3. **Identifica rischi**: problemi aperti + blocchi + ritardi
4. **Traduci**: ogni tecnicismo in business
5. **Genera dashboard**: usa template sopra
6. **Valuta launch readiness**: checklist lancio

## CHECKLIST LAUNCH READINESS

- [ ] MVP feature complete (100% must-have)
- [ ] Test E2E flusso critico passa
- [ ] Security audit verde (no critici)
- [ ] Privacy policy + cookie banner pubblicati
- [ ] Monitoring attivo (uptime + errori)
- [ ] Backup DB automatico
- [ ] Piano rollback documentato
- [ ] Team on-call definito
- [ ] Comunicazione lancio pronta (email, social, changelog)
- [ ] Assistenza utenti: canale definito (email, chat, ticket)

## QUANDO PASSARE A ALTRE SKILL

`faro-pm` è una vista — non esegue azioni tecniche. Per:
- Pianificare lavoro tecnico → skill Faro standard
- Gestire PR e team developer → `faro-team`
- Deploy e infrastruttura → `faro-devops`
- Sicurezza e GDPR → `faro-security`

## REFERENCES

Per dettagli tecnici completi, leggi:
- [references/pm-framework.md] — risk matrix, launch readiness dashboard, stakeholder template, roadmap format, metriche business
