# Tier 2 — PROFESSIONALE

Include tutto il Tier 1 più:

## Quando usare
SaaS B2C/B2B, e-commerce, siti aziendali, dashboard pubbliche.

## Stack aggiuntivo
- Framer Motion (animazioni layout e di entrata)
- Skeleton loading (react-loading-skeleton o custom)
- Empty state illustrati

## Animazioni consentite
- Fade in all'ingresso di pagina/sezione (opacity 0→1, 200-300ms)
- Slide in per sidebar/panel (translateX, 200ms)
- Layout animation per liste che cambiano (Framer Motion layoutId)
- Card hover: leggero scale(1.02) o shadow-md, 150ms
- Skeleton shimmer durante caricamento
- Toast slide-in

## Animazioni VIETATE
- Parallax
- Scroll-triggered animations
- Text reveal lettera per lettera
- Transizioni di pagina full-screen
- Qualsiasi animazione >300ms

## Componenti aggiuntivi
Skeleton, EmptyState (icona + testo + CTA), Avatar, AvatarGroup,
HoverCard, Progress, Accordion, Sheet (slide panel), Command palette avanzata

## Layout aggiuntivo
- Mobile-first per SaaS
- Bottom tab bar su mobile
- Card grid per dashboard (1-2-3-4 colonne responsive)
- Pricing table

## Empty state pattern
Ogni lista/tabella vuota deve avere:
- Illustrazione/icona coerente col design
- Testo che spiega perché è vuoto
- CTA per aggiungere il primo elemento

## Skeleton pattern
Ogni contenuto con caricamento asincrono deve mostrare skeleton:
- Skeleton ha la stessa forma del contenuto finale
- Animazione shimmer (linear-gradient animate)
- Durata: visibile per almeno 200ms (evita flash)

## Stile riferimento
Vercel: minimal bianco/nero con accent blue
Cal.com: bianco con accent verde, bordi sottili
Stripe: gradient sottili, tipografia curata
