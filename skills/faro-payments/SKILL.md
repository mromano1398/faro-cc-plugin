---
name: faro-payments
description: |
  Pattern pagamenti: Stripe setup, webhook, subscription, checkout, invoicing.
  Trigger: "stripe", "pagamenti", "subscription", "checkout", "webhook stripe"
---

# faro-payments — Stripe · Abbonamenti · Webhook · Portal · Fatture

**Lingua:** Sempre italiano. Riferimenti ufficiali:
- Stripe Docs: https://stripe.com/docs
- Stripe API Reference: https://stripe.com/docs/api
- Stripe Next.js: https://stripe.com/docs/payments/quickstart

## QUANDO USARE

- Integrazione pagamenti Stripe (una tantum o abbonamenti)
- Setup webhook handler per ciclo vita abbonamento
- Customer portal self-service
- Protezione route per piano (FREE/PRO/ENTERPRISE)
- Fatturazione italiana con IVA automatica

## FLUSSO DECISIONALE

```
Pagamento una tantum → Checkout Session mode='payment'
Abbonamento         → Checkout Session mode='subscription' + trial_period_days
Upgrade/downgrade   → Customer Portal (Stripe gestisce tutto)
Verifica piano      → getSubscriptionPlan(userId) in ogni Server Action protetta

Webhook obbligatori da implementare:
checkout.session.completed → attiva abbonamento
customer.subscription.updated → aggiorna piano
customer.subscription.deleted → cancella abbonamento
invoice.payment_failed → stato PAST_DUE + email utente
invoice.paid → aggiorna periodo_fine
```

## REGOLE CHIAVE

1. **Chiavi test vs produzione separate** in env — mai chiavi prod in sviluppo
2. **Webhook endpoint configurato** su Stripe Dashboard prima del deploy
3. **`STRIPE_WEBHOOK_SECRET`** obbligatorio per validare firma webhook
4. **Idempotenza webhook** — tabella `webhook_events` per evitare doppio processing
5. **`utente_id` nei metadata** del checkout — necessario per associare il pagamento
6. **Implementa tutti e 5 gli eventi webhook** — non solo `checkout.session.completed`
7. **Customer Portal** per gestione self-service — non costruirlo da zero
8. **`automatic_tax: true`** per IVA automatica per paese utente
9. **Test con Stripe CLI** prima del deploy in produzione
10. **`stato: PAST_DUE`** su `invoice.payment_failed` + email immediata all'utente

## FLUSSO OPERATIVO

1. **Stripe setup**: crea prodotti + prezzi nel Dashboard
2. **Schema DB**: tabella `abbonamenti` + `webhook_events` per idempotenza
3. **Checkout Session**: endpoint `/api/checkout` con `userId` nei metadata
4. **Webhook handler**: `/api/stripe/webhook` con signature verify + idempotency
5. **Customer Portal**: link in settings account
6. **Feature gate**: `getSubscriptionPlan()` prima di operazioni PRO
7. **Test**: Stripe CLI `stripe listen` + card test `4242 4242 4242 4242`
8. **Deploy**: cambia chiavi + endpoint webhook produzione

## CHECKLIST

- [ ] Chiavi test vs produzione separate in env
- [ ] Webhook endpoint configurato su Stripe Dashboard
- [ ] `STRIPE_WEBHOOK_SECRET` aggiunto all'env
- [ ] Idempotenza webhook (tabella `webhook_events`)
- [ ] Gestione `invoice.payment_failed` con email all'utente
- [ ] Customer Portal configurato
- [ ] Schema DB abbonamenti con `stato` e `periodo_fine`
- [ ] Protezione route per piano (FREE non vede funzionalità PRO)
- [ ] Test con Stripe CLI prima del deploy
- [ ] Transizione da test a produzione: aggiorna chiavi env in produzione
- [ ] Automatic tax abilitato (IVA automatica per paese utente)

## TABELLA — Eventi webhook critici

| Evento Stripe | Azione nella tua app |
|---------------|----------------------|
| `checkout.session.completed` | Crea/attiva abbonamento, stato=ACTIVE |
| `customer.subscription.updated` | Aggiorna piano, periodo_fine |
| `customer.subscription.deleted` | Stato=CANCELLED, disattiva feature PRO |
| `invoice.payment_failed` | Stato=PAST_DUE, email utente |
| `invoice.paid` | Estendi periodo_fine, stato=ACTIVE |

## Web vs Mobile: Stripe, IAP, e policy degli store

Stripe funziona su web senza restrizioni. Su mobile (iOS/Android) digital goods
richiedono **Apple IAP** o **Google Play Billing** (Stripe VIETATO); physical goods
e servizi fuori-app possono usare Stripe. Per un SaaS web + mobile valuta
**RevenueCat** (unifica Stripe e IAP con un'unica API).

Dettagli, workaround legittimi (reader app, DMA UE, ruling Epic 2024),
strategia per SaaS multi-target e anti-pattern in
`references/stripe-patterns.md` → sezione "Web vs Mobile".

## REFERENCES

Per dettagli tecnici completi, leggi:
- [references/stripe-patterns.md] — checkout session, subscription lifecycle, webhook handler idempotente, customer portal, fatturazione IVA, test con Stripe CLI, Web vs Mobile (IAP)
