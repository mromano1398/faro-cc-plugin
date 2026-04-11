# Raccomandazioni Stack per tipo progetto

## Gestionale / ERP
Stack consigliato: Next.js + PostgreSQL + Prisma + NextAuth + shadcn/ui
Deploy: Docker + VPS (per privacy dati) oppure Vercel (se dati non sensibili)
Auth: NextAuth con credenziali + RBAC custom
Costo: ~0-20€/mese (VPS) o ~0€ (Vercel free)
Alternativa Python: FastAPI + SQLAlchemy + PostgreSQL

## SaaS Bootstrap
Stack: Next.js + Supabase + Stripe + Resend + shadcn/ui + Framer Motion
Deploy: Vercel
Auth: Supabase Auth
Costo: ~0-50€/mese (Supabase free + Vercel free + Stripe pay-per-use)
Nota: Supabase offre auth + DB + storage + realtime in un unico servizio

## SaaS Enterprise
Stack: Next.js + PostgreSQL + Prisma + Clerk + Stripe + SendGrid + Redis + Sentry
Deploy: Vercel + Railway (DB)
Costo: ~100-300€/mese
Nota: Clerk per auth enterprise con SSO, Redis per cache e rate limiting

## SaaS con AI (LLM integrati)
Stack: Next.js + Supabase + Stripe + Vercel AI SDK + Anthropic/OpenAI API + Resend + Upstash Redis
Deploy: Vercel (fluid compute per streaming)
Auth: Supabase Auth o Clerk
Costo: ~30-150€/mese (Supabase free/pro + Vercel pro + Anthropic/OpenAI pay-per-use + Upstash)
Note:
- Usa Vercel AI SDK per streaming UI e tool calling multi-provider (Anthropic, OpenAI, Google)
- Upstash Redis per rate limiting per-utente sulle API LLM (costo controllato)
- Supabase pgvector per embeddings e RAG (vector search built-in, no vector DB extra)
- Edge Functions di Supabase per logica AI che serve bassa latenza
- Stripe per billing consumption-based (token usati) con Stripe Meter
- Resend per email transazionali (conferme, password reset, notifiche AI)
- Anthropic API: default per qualità (Claude Opus/Sonnet)
- OpenAI API: fallback o per feature specifiche (embedding text-embedding-3, DALL-E)

Skill Faro attivabili: faro-ai (pattern LLM + streaming + RAG), faro-supabase (RLS + pgvector),
faro-payments (Stripe metering), faro-multitenant (se B2B con org isolation).

Pattern architetturali tipici:
- `app/api/chat/route.ts` con streamText() o generateText() del Vercel AI SDK
- `lib/ai/prompts.ts` per prompt template versionati
- `lib/ai/tools.ts` per tool calling (function calling)
- `lib/rag/retrieve.ts` per pgvector similarity search
- Rate limiting middleware (Upstash @upstash/ratelimit) su tutte le API /api/chat /api/completion

Gap da monitorare:
- Costi LLM possono esplodere → metti sempre rate limit per-user e quote
- Prompt injection: validazione input utente prima di inoltrare al modello
- Privacy: se contenuti utenti finiscono nei prompt, valuta zero-data-retention con AI Gateway

## API (backend only)
Stack consigliato: Hono + TypeScript + PostgreSQL + Prisma + Zod + Upstash Redis
Deploy: Vercel Functions o Railway
Auth: JWT Bearer token custom o Supabase Auth (se serve session)
Costo: ~0-30€/mese
Alternativa Python: FastAPI + SQLAlchemy + PostgreSQL + Pydantic
Alternativa enterprise: NestJS + PostgreSQL + Prisma + Passport + Redis
Nota: API-only progetti. Vedi faro-blueprints/references/api.md per struttura cartelle,
security checklist, REST conventions e deploy options.
Skill Faro attivabili: faro-security (OWASP), faro-testing (integration test), faro-devops (Docker).

## E-commerce
Stack: Next.js + PostgreSQL + Prisma + Stripe + Resend + Meilisearch
Deploy: Vercel
Costo: ~30-100€/mese
Nota: Meilisearch per ricerca prodotti veloce, Stripe per checkout

## Landing Page
Stack: Astro (se statico) o Next.js (se con form/waitlist) + Tailwind
Deploy: Vercel o Cloudflare Pages
Costo: ~0€
Nota: Astro per performance massima se non serve interattività

## Portfolio / Showcase
Stack: Next.js + Tailwind + GSAP + Lenis (Tier 3) o + Three.js (Tier 4)
Deploy: Vercel
Costo: ~0€
Nota: Font custom obbligatorio, dark theme consigliato

## Mobile
Stack: Expo + React Native + Supabase + Stripe
Deploy: EAS Build + App Store / Google Play
Costo: ~0-30€/mese (Supabase + EAS)
Nota: Expo per sviluppo rapido cross-platform

## CLI / Bot
Stack: Node.js/TypeScript + Commander/Yargs (CLI) o Node-Telegram-Bot-API/Discord.js (bot)
Deploy: npm publish (CLI) o Docker + VPS (bot)
Costo: ~5-10€/mese (VPS per bot)
