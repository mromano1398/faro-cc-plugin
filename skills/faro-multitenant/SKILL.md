---
name: faro-multitenant
description: |
  Pattern multi-tenant: tenant isolation, schema per tenant, shared schema con tenant_id.
  Trigger: "multi-tenant", "tenant", "isolamento dati tra clienti"
---

# faro-multitenant — Isolamento Dati · Tenant Detection · Provisioning

**Lingua:** Sempre italiano.
**Principio:** La scelta tra schema-per-tenant e row-level isolation determina tutto il design. Prendila prima di scrivere una riga.

## QUANDO USARE

- SaaS con multiple organizzazioni/workspace che condividono la stessa app
- Dati di un tenant non devono mai essere visibili a un altro tenant
- Onboarding self-service di nuovi tenant
- Sistema di inviti per aggiungere membri a un'organizzazione
- Feature gates per piano (FREE/PRO/ENTERPRISE)

## FLUSSO DECISIONALE

```
Quanti tenant prevedi?
< 100 + dati sensibili → Schema-per-tenant (isolamento completo)
100-10.000            → Row-level con RLS (consigliato SaaS B2B)
> 10.000              → Database separato (solo se hai team infra dedicato)

Setup base:
1. Crea tabelle organizzazioni + membri con organizzazione_id su ogni tabella business
2. Attiva RLS su tutte le tabelle business
3. Crea funzione helper mie_organizzazioni()
4. Scegli tenant detection: subdomain vs path-based
5. Implementa provisioning transazionale (org + owner + dati default)
6. Aggiungi feature gates per piano
```

## REGOLE CHIAVE

1. **`organizzazione_id` su TUTTE le tabelle business** — senza eccezioni
2. **RLS policy attive e testate su ogni tabella** — non solo abilitare RLS
3. **Test cross-tenant obbligatorio** — utente A non può mai vedere dati utente B
4. **Provisioning in transazione** — org + owner + dati default in un'unica TX
5. **Inviti con token sicuro e scadenza** — mai inviare link senza scadenza
6. **Feature gates prima di creare risorse** — verifica limite piano prima di ogni INSERT
7. **`adminClient` mai esposto al client** — bypassa RLS
8. **Slug univoco per ogni tenant** — gestire collisioni nel provisioning
9. **Indici su `organizzazione_id`** obbligatori per performance
10. **Ruoli: OWNER, ADMIN, MEMBER** — solo OWNER/ADMIN possono cancellare

## FLUSSO OPERATIVO

1. **Scegli strategia isolamento** (vedi flusso decisionale)
2. **Crea schema**: organizzazioni + membri + tabelle business con `organizzazione_id`
3. **Attiva RLS** e scrivi policy basate su `mie_organizzazioni()`
4. **Tenant detection**: subdomain middleware o path-based
5. **Provisioning**: endpoint transazionale (org + owner + default)
6. **Inviti**: token con scadenza, flusso accept
7. **Feature gates**: check piano prima di operazioni costose
8. **Test cross-tenant**: verifica isolamento in integration test

## CHECKLIST

- [ ] Strategia scelta (row-level / schema-per-tenant / database separato)
- [ ] `organizzazione_id` su tutte le tabelle business
- [ ] RLS policy attive e testate su ogni tabella
- [ ] Funzione helper `mie_organizzazioni()` per RLS
- [ ] Middleware tenant detection (subdomain o path)
- [ ] Provisioning transazionale (org + owner + dati default)
- [ ] Sistema inviti con token sicuro e scadenza
- [ ] Feature gates per piano (FREE/PRO/ENTERPRISE)
- [ ] Test cross-tenant: utente A non può mai vedere dati utente B
- [ ] `adminClient` (service role) mai esposto al client

## TABELLA — Strategie a confronto

| Aspetto | Row-level + RLS | Schema per tenant | Database separato |
|---------|-----------------|-------------------|-------------------|
| Setup | Semplice | Medio | Complesso |
| Isolamento | Logico | Fisico (schema) | Totale |
| Costo infra | Basso | Medio | Alto |
| Migration | 1 per tutti | 1 ripetuta N | N separate |
| Backup per tenant | Difficile | Facile | Facile |
| Scala max | ~10k tenant | ~500 tenant | Illimitata |
| Consigliato per | SaaS B2B mid | Healthcare/compliance | Enterprise |

## REFERENCES

Per dettagli tecnici completi, leggi:
- [references/tenant-isolation.md] — schemi DB, RLS policy multi-tenant, subdomain detection, provisioning transazionale, sistema inviti, feature gates, migration zero-downtime
