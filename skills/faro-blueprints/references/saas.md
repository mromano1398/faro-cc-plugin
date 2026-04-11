# Blueprint — SaaS B2C / B2B

## Struttura cartelle (Next.js App Router)
```
src/
├── app/
│   ├── (marketing)/                # Pagine pubbliche
│   │   ├── page.tsx                # Landing
│   │   ├── pricing/page.tsx
│   │   ├── features/page.tsx
│   │   └── layout.tsx
│   ├── (auth)/
│   │   ├── login/page.tsx
│   │   ├── register/page.tsx
│   │   ├── forgot-password/page.tsx
│   │   └── layout.tsx
│   ├── (app)/                      # App autenticata
│   │   ├── layout.tsx
│   │   ├── dashboard/page.tsx
│   │   ├── [feature]/page.tsx
│   │   ├── settings/
│   │   │   ├── page.tsx
│   │   │   ├── billing/page.tsx
│   │   │   └── team/page.tsx
│   │   └── onboarding/page.tsx
│   ├── api/
│   │   ├── webhooks/stripe/route.ts
│   │   └── [feature]/route.ts
│   ├── layout.tsx
│   └── globals.css
├── components/
│   ├── ui/
│   ├── marketing/                  # Hero, Pricing, Features
│   ├── app/                        # Dashboard, Settings
│   └── shared/                     # Navbar, Footer
├── lib/
│   ├── supabase/                   # Client + server
│   ├── stripe/                     # Config + helpers
│   ├── email/                      # Resend templates
│   └── validations/
└── types/
```

## Navigazione
Marketing: Navbar top con logo + link + CTA login/signup
App: Sidebar (desktop) o bottom tab (mobile) + header con avatar/notifiche
Onboarding: wizard 3-5 step al primo accesso

## Sicurezza specifica
- Rate limiting su login (5 tentativi / 15 min)
- Email verification obbligatoria
- Password: bcrypt, min 8 char
- CSRF protection
- Webhook Stripe: verifica signature

## UX specifica
- Onboarding wizard al primo accesso
- Empty state su ogni sezione (guida l'utente)
- Upgrade prompt contestuali (non invasivi)
- Skeleton loading su tutto
- Toast per feedback azioni
