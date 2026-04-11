---
name: faro-tier-system
description: |
  Sistema a 4 tier di design per qualsiasi tipo di progetto. Ogni tier include tutto quello
  del precedente. La palette NON è mai fissa — viene personalizzata per ogni progetto.
  Trigger: "tier", "design", "livello design", "quale stile", invocato dal wizard
---

# Faro Tier System

## I 4 Tier

| Tier | Nome | Target | Stack aggiuntivo | Stile riferimento |
|------|------|--------|------------------|--------------------|
| 1 | FUNZIONALE | Gestionale, ERP, tool | shadcn/ui, zero animazioni, desktop-first | Notion, Linear |
| 2 | PROFESSIONALE | SaaS, e-commerce | + Framer Motion, skeleton, empty state | Vercel, Cal.com, Stripe |
| 3 | CINEMATIC | Portfolio, brand, landing | + GSAP, Lenis, dark theme, font display | Siti Awwwards |
| 4 | IMMERSIVO | Luxury, architettura | + Three.js/R3F, scene 3D, .glb | petralithe.com, oryzo.ai |

## Logica di scelta

- Ogni tier INCLUDE tutto quello del precedente
- La palette NON è mai fissa — sempre personalizzata
- Font: Tier 1-2 possono usare Inter/system-ui. Tier 3-4 MAI Inter/Roboto — usare font bold/display
- Animazioni: Tier 1 zero. Tier 2 sottili ≤300ms. Tier 3 cinematiche. Tier 4 + 3D

## Dettagli per tier

Per i dettagli completi di ogni tier (componenti, animazioni, responsive, font, pattern), leggi:
- Tier 1: `references/tier-1-funzionale.md`
- Tier 2: `references/tier-2-professionale.md`
- Tier 3: `references/tier-3-cinematic.md`
- Tier 4: `references/tier-4-immersivo.md`
