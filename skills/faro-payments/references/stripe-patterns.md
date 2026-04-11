# Stripe Patterns — Faro Payments

Pattern completi per integrazione Stripe in progetti Faro.

## SETUP

```bash
npm install stripe @stripe/stripe-js
```

`.env.local`:
```
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_...
STRIPE_PRICE_PRO=price_...
STRIPE_PRICE_ENTERPRISE=price_...
```

`lib/stripe.ts`:
```ts
import Stripe from 'stripe'

export const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: '2024-06-20',
  typescript: true,
})
```

## SCHEMA DB

```sql
CREATE TABLE abbonamenti (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  utente_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  stripe_customer_id TEXT NOT NULL,
  stripe_subscription_id TEXT UNIQUE,
  stripe_price_id TEXT,
  stato TEXT CHECK (stato IN ('TRIALING', 'ACTIVE', 'PAST_DUE', 'CANCELLED', 'INCOMPLETE')),
  piano TEXT CHECK (piano IN ('FREE', 'PRO', 'ENTERPRISE')),
  periodo_fine TIMESTAMPTZ,
  cancel_at_period_end BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE UNIQUE INDEX ON abbonamenti (utente_id);

-- Idempotenza webhook
CREATE TABLE webhook_events (
  id TEXT PRIMARY KEY, -- evt_xxx da Stripe
  tipo TEXT NOT NULL,
  processato_il TIMESTAMPTZ DEFAULT NOW()
);
```

## CHECKOUT — Una tantum

```ts
// app/api/checkout/route.ts
import { stripe } from '@/lib/stripe'
import { requireAuth } from '@/lib/auth'

export async function POST(req: Request) {
  const session = await requireAuth()
  const { priceId } = await req.json()

  const checkoutSession = await stripe.checkout.sessions.create({
    mode: 'payment',
    payment_method_types: ['card'],
    line_items: [{ price: priceId, quantity: 1 }],
    success_url: `${process.env.APP_URL}/success?session_id={CHECKOUT_SESSION_ID}`,
    cancel_url: `${process.env.APP_URL}/cancelled`,
    customer_email: session.user.email,
    metadata: { utente_id: session.user.id }, // CRITICO
    automatic_tax: { enabled: true },
  })

  return Response.json({ url: checkoutSession.url })
}
```

## CHECKOUT — Abbonamento con trial

```ts
export async function POST(req: Request) {
  const session = await requireAuth()
  const { priceId } = await req.json()

  // Recupera o crea customer Stripe
  let customerId: string
  const esistente = await db.abbonamento.findUnique({
    where: { utenteId: session.user.id },
  })
  if (esistente) {
    customerId = esistente.stripeCustomerId
  } else {
    const customer = await stripe.customers.create({
      email: session.user.email,
      metadata: { utente_id: session.user.id },
    })
    customerId = customer.id
  }

  const checkoutSession = await stripe.checkout.sessions.create({
    mode: 'subscription',
    customer: customerId,
    line_items: [{ price: priceId, quantity: 1 }],
    success_url: `${process.env.APP_URL}/success`,
    cancel_url: `${process.env.APP_URL}/pricing`,
    subscription_data: {
      trial_period_days: 14,
      metadata: { utente_id: session.user.id },
    },
    metadata: { utente_id: session.user.id },
    automatic_tax: { enabled: true },
    billing_address_collection: 'required',
    tax_id_collection: { enabled: true }, // P.IVA per B2B
  })

  return Response.json({ url: checkoutSession.url })
}
```

## WEBHOOK HANDLER — Idempotente

```ts
// app/api/stripe/webhook/route.ts
import { NextResponse } from 'next/server'
import { headers } from 'next/headers'
import { stripe } from '@/lib/stripe'
import { db } from '@/lib/db'
import type Stripe from 'stripe'

export async function POST(req: Request) {
  const body = await req.text()
  const sig = (await headers()).get('stripe-signature')!

  let event: Stripe.Event
  try {
    event = stripe.webhooks.constructEvent(
      body,
      sig,
      process.env.STRIPE_WEBHOOK_SECRET!
    )
  } catch (err) {
    return new Response(`Webhook signature invalid: ${err}`, { status: 400 })
  }

  // IDEMPOTENZA
  const already = await db.webhookEvent.findUnique({ where: { id: event.id } })
  if (already) return NextResponse.json({ received: true, duplicate: true })

  try {
    switch (event.type) {
      case 'checkout.session.completed':
        await handleCheckoutCompleted(event.data.object as Stripe.Checkout.Session)
        break
      case 'customer.subscription.updated':
        await handleSubscriptionUpdated(event.data.object as Stripe.Subscription)
        break
      case 'customer.subscription.deleted':
        await handleSubscriptionDeleted(event.data.object as Stripe.Subscription)
        break
      case 'invoice.payment_failed':
        await handlePaymentFailed(event.data.object as Stripe.Invoice)
        break
      case 'invoice.paid':
        await handleInvoicePaid(event.data.object as Stripe.Invoice)
        break
      default:
        console.log(`Webhook ignorato: ${event.type}`)
    }

    // Salva evento processato
    await db.webhookEvent.create({
      data: { id: event.id, tipo: event.type },
    })

    return NextResponse.json({ received: true })
  } catch (error) {
    console.error(`Errore processing webhook ${event.type}:`, error)
    // NON salvare come processato → Stripe riprova
    return new Response(`Internal error: ${error}`, { status: 500 })
  }
}

async function handleCheckoutCompleted(session: Stripe.Checkout.Session) {
  const utenteId = session.metadata?.utente_id
  if (!utenteId) throw new Error('utente_id mancante nei metadata')

  if (session.mode === 'subscription') {
    const sub = await stripe.subscriptions.retrieve(session.subscription as string)
    await upsertAbbonamento(utenteId, sub)
  }
}

async function handleSubscriptionUpdated(sub: Stripe.Subscription) {
  const utenteId = sub.metadata?.utente_id
  if (!utenteId) return

  await upsertAbbonamento(utenteId, sub)
}

async function handleSubscriptionDeleted(sub: Stripe.Subscription) {
  await db.abbonamento.update({
    where: { stripeSubscriptionId: sub.id },
    data: { stato: 'CANCELLED', piano: 'FREE' },
  })
}

async function handlePaymentFailed(invoice: Stripe.Invoice) {
  if (!invoice.subscription) return

  await db.abbonamento.update({
    where: { stripeSubscriptionId: invoice.subscription as string },
    data: { stato: 'PAST_DUE' },
  })

  // Email utente (via Resend / SendGrid / Postmark)
  const abb = await db.abbonamento.findUnique({
    where: { stripeSubscriptionId: invoice.subscription as string },
    include: { utente: true },
  })
  if (abb) {
    await sendEmail({
      to: abb.utente.email,
      subject: 'Pagamento fallito — aggiorna il metodo di pagamento',
      html: `Il tuo ultimo pagamento è fallito. <a href="${process.env.APP_URL}/billing">Aggiorna</a>`,
    })
  }
}

async function handleInvoicePaid(invoice: Stripe.Invoice) {
  if (!invoice.subscription) return
  const sub = await stripe.subscriptions.retrieve(invoice.subscription as string)
  const utenteId = sub.metadata?.utente_id
  if (!utenteId) return
  await upsertAbbonamento(utenteId, sub)
}

async function upsertAbbonamento(utenteId: string, sub: Stripe.Subscription) {
  const priceId = sub.items.data[0].price.id
  const piano = mapPriceToPiano(priceId)

  await db.abbonamento.upsert({
    where: { utenteId },
    create: {
      utenteId,
      stripeCustomerId: sub.customer as string,
      stripeSubscriptionId: sub.id,
      stripePriceId: priceId,
      stato: sub.status.toUpperCase().replace('TRIALING', 'TRIALING') as any,
      piano,
      periodoFine: new Date(sub.current_period_end * 1000),
      cancelAtPeriodEnd: sub.cancel_at_period_end,
    },
    update: {
      stripeSubscriptionId: sub.id,
      stripePriceId: priceId,
      stato: sub.status.toUpperCase() as any,
      piano,
      periodoFine: new Date(sub.current_period_end * 1000),
      cancelAtPeriodEnd: sub.cancel_at_period_end,
    },
  })
}

function mapPriceToPiano(priceId: string): 'FREE' | 'PRO' | 'ENTERPRISE' {
  if (priceId === process.env.STRIPE_PRICE_PRO) return 'PRO'
  if (priceId === process.env.STRIPE_PRICE_ENTERPRISE) return 'ENTERPRISE'
  return 'FREE'
}
```

## CUSTOMER PORTAL

```ts
// app/api/stripe/portal/route.ts
import { stripe } from '@/lib/stripe'
import { requireAuth } from '@/lib/auth'
import { db } from '@/lib/db'

export async function POST(req: Request) {
  const session = await requireAuth()

  const abb = await db.abbonamento.findUnique({
    where: { utenteId: session.user.id },
  })
  if (!abb) return Response.json({ error: 'Nessun abbonamento' }, { status: 404 })

  const portalSession = await stripe.billingPortal.sessions.create({
    customer: abb.stripeCustomerId,
    return_url: `${process.env.APP_URL}/settings/billing`,
  })

  return Response.json({ url: portalSession.url })
}
```

## FEATURE GATE

```ts
// lib/subscription.ts
export async function getSubscriptionPlan(utenteId: string) {
  const abb = await db.abbonamento.findUnique({
    where: { utenteId },
  })

  if (!abb || abb.stato === 'CANCELLED' || abb.stato === 'PAST_DUE') {
    return { piano: 'FREE' as const, attivo: false }
  }

  const attivo =
    (abb.stato === 'ACTIVE' || abb.stato === 'TRIALING') &&
    abb.periodoFine > new Date()

  return { piano: abb.piano, attivo }
}

export async function requirePro(utenteId: string) {
  const { piano, attivo } = await getSubscriptionPlan(utenteId)
  if (!attivo || (piano !== 'PRO' && piano !== 'ENTERPRISE')) {
    throw new Error('Questa funzionalità richiede il piano PRO')
  }
}
```

Uso:
```ts
// Server Action protetta
export async function esportaDatiAdvanced() {
  const session = await requireAuth()
  await requirePro(session.user.id) // gate
  // ...
}
```

## TEST CON STRIPE CLI

```bash
# Installa
brew install stripe/stripe-cli/stripe   # macOS
# o https://stripe.com/docs/stripe-cli

stripe login

# Forward webhook al locale
stripe listen --forward-to localhost:3000/api/stripe/webhook
# Copia il whsec_... nel .env.local

# Triggera eventi di test
stripe trigger checkout.session.completed
stripe trigger customer.subscription.updated
stripe trigger invoice.payment_failed
```

**Carte di test:**
- Successo: `4242 4242 4242 4242`
- Declina: `4000 0000 0000 0002`
- Richiede 3DS: `4000 0025 0000 3155`
- Declina insufficiente: `4000 0000 0000 9995`

## FATTURAZIONE ITALIANA — IVA

```ts
// Quando crei prodotti + prezzi
const product = await stripe.products.create({
  name: 'Faro PRO',
  tax_code: 'txcd_10000000', // SaaS tax code
})

const price = await stripe.prices.create({
  product: product.id,
  unit_amount: 2900, // 29.00 €
  currency: 'eur',
  recurring: { interval: 'month' },
  tax_behavior: 'exclusive', // prezzi al netto IVA
})
```

Abilita **Stripe Tax** nel Dashboard → Tax → Settings.
Con `automatic_tax: { enabled: true }` Stripe calcola IVA per paese.

## DEPLOY — Test → Produzione

- [ ] Crea endpoint webhook produzione su Stripe Dashboard
- [ ] Copia nuovo `STRIPE_WEBHOOK_SECRET` in env produzione
- [ ] Switcha a chiavi `sk_live_` e `pk_live_`
- [ ] Crea prodotti/prezzi live corrispondenti ai test
- [ ] Aggiorna env con nuovi `STRIPE_PRICE_*`
- [ ] Test finale con carta reale (rimborsabile)
- [ ] Monitoring attivo su webhook fail rate (<1%)
- [ ] Alert su `invoice.payment_failed` spike

## Web vs Mobile: Stripe, IAP, e le policy degli store

Stripe funziona su web senza restrizioni. Su mobile (iOS/Android) le policy degli app store
impongono vincoli diversi in base al tipo di bene venduto:

### Digital goods consumati nell'app (sub SaaS, crediti, feature premium)
- **iOS**: OBBLIGATORIO usare In-App Purchases (Apple 30% o 15% small business). Stripe VIETATO.
- **Android**: OBBLIGATORIO Google Play Billing. Stripe VIETATO (salvo in Europa dopo DMA 2024).
- **Workaround** (legittimi):
  - "Reader app" exception (iOS): contenuto acquistato fuori dall'app, l'app è solo lettore
  - Link "out-of-app" a sito web (iOS dopo ruling Epic vs Apple 2024, vedi [docs Apple])
  - In UE: DMA permette alternative ma servono setup specifici

### Physical goods o servizi fuori-app (ristorante, prenotazioni, shop fisico)
- **iOS e Android**: Stripe OK, nessuna commissione extra agli store

### Strategia consigliata per un SaaS web + mobile
1. **Web**: Stripe subscriptions e one-time
2. **Mobile (iOS)**: Apple IAP con identificatore subscription matching Stripe
3. **Mobile (Android)**: Google Play Billing
4. **Sincronizzazione account**: l'utente che paga su web accede su mobile senza ri-pagare
   (lato backend, entitlements centralizzati tramite RevenueCat o custom)
5. **Piattaforma di riferimento: RevenueCat** (faro-payments Tier 2 support) gestisce sia
   Stripe che IAP con un'unica API

### Implementazione consigliata
- Web + mobile insieme → valuta **RevenueCat** (dashboard unica, webhook centralizzato)
- Solo web → Stripe diretto
- Solo mobile → IAP nativo (expo-in-app-purchases o react-native-iap)

### Anti-pattern
- Provare a usare Stripe webview su iOS → REJECTED in review
- "Web link esterno per pagare" senza reader exception → REJECTED
- Ignorare la differenza → l'app viene rimossa dallo store al primo report
