# Blueprint — Portfolio / Showcase

## Struttura
```
src/
├── app/
│   ├── page.tsx                    # Home con hero + progetti in evidenza
│   ├── lavori/
│   │   ├── page.tsx                # Griglia progetti
│   │   └── [slug]/page.tsx         # Dettaglio progetto
│   ├── chi-sono/page.tsx           # (o about)
│   ├── contatti/page.tsx
│   └── layout.tsx
├── components/
│   ├── layout/
│   │   ├── navbar.tsx              # Trasparente → solida
│   │   ├── footer.tsx
│   │   └── page-transition.tsx
│   ├── home/
│   │   ├── hero.tsx
│   │   └── featured-projects.tsx
│   ├── projects/
│   │   ├── project-card.tsx
│   │   └── project-gallery.tsx
│   ├── animations/
│   │   ├── scroll-reveal.tsx
│   │   ├── text-reveal.tsx
│   │   ├── parallax-image.tsx
│   │   └── smooth-scroll.tsx
│   └── ui/
├── content/
│   └── projects/                   # MDX o JSON per i progetti
```

## Navigazione
Navbar minima: Logo + Lavori + Chi sono + Contatti
Mobile: menu full-screen con animazione
No sidebar, no footer pesante

## UX specifica
- Ogni progetto: galleria immagini, descrizione, tecnologie, link live
- Transizioni tra pagine
- Cursor custom (opzionale)
- Smooth scroll
- Hover effect sulle card progetto
