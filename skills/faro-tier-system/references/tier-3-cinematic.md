# Tier 3 — CINEMATIC

Include tutto il Tier 1-2 più:

## Quando usare
Portfolio, siti brand, landing page premium, agenzie creative, prodotti lifestyle.

## Stack aggiuntivo
- GSAP + ScrollTrigger (animazioni scroll)
- Lenis (smooth scroll)
- Font display bold/custom (MAI Inter, MAI Roboto)
- Framer Motion per micro-interazioni

## Dipendenze npm
```json
{
  "gsap": "^3.12",
  "@studio-freight/lenis": "^1.0",
  "framer-motion": "^11"
}
```

## Theme
- Dark theme DEFAULT (#09090b sfondo, #FAFAFA testo)
- Accent vibrante personalizzato (MAI purple gradient default)
- Contrasto alto per impatto visivo

## Font
MAI Inter, Roboto, Poppins, Montserrat.
USARE font bold/display:
- Heading: Clash Display, Satoshi, Space Grotesk, Syne, Cabinet Grotesk, General Sans
- Body: Inter è ok SOLO per body se heading è display font
- Peso heading: 700-900. Body: 400-500.

## Componenti esclusivi Tier 3

### ScrollReveal
```tsx
// Componente che anima l'ingresso di elementi allo scroll
// Usa GSAP ScrollTrigger
// Varianti: fade-up, fade-left, fade-right, scale-in, blur-in
// Stagger per liste di elementi
```

### TextReveal
```tsx
// Testo che si rivela parola per parola o riga per riga allo scroll
// Usa GSAP SplitText o split manuale
// Per heading hero e quote
```

### ParallaxImage
```tsx
// Immagine con effetto parallasse allo scroll
// Usa GSAP ScrollTrigger con yPercent
// Clip-path per bordi creativi
```

### PageTransition
```tsx
// Transizione tra pagine con overlay animato
// Colore overlay = accent del progetto
// Durata: 600-800ms
```

### Navbar
```tsx
// Trasparente su hero → solida dopo scroll
// Usa useScroll di Framer Motion o IntersectionObserver
// Hamburger animato su mobile
```

### HeroSection
```tsx
// 100vh, centrato, heading grande, sotto-titolo, CTA
// Background: gradient, video, immagine con overlay
// Animazione entrata: stagger heading + subtitle + CTA
```

### SmoothScroll setup
```tsx
// Inizializza Lenis in layout.tsx
// Sync con GSAP ScrollTrigger
// Disable su prefers-reduced-motion
```

## Animazioni consentite
- Scroll-triggered reveal (fade, slide, scale, blur)
- Text split animation (per heading)
- Parallax su immagini
- Cursor custom (opzionale)
- Page transition
- Magnetic buttons (opzionale)
- Marquee/ticker (opzionale)

## Responsive Tier 3
- Mobile: disabilita parallax, riduci animazioni
- `prefers-reduced-motion`: disabilita TUTTE le animazioni scroll, mantieni solo fade

## Performance
- Lazy load immagini sotto il fold
- Dynamic import per GSAP (non caricarlo se non serve)
- Font display: swap
- Lighthouse target: ≥80 performance, ≥90 a11y

## Regole assolute
- MAI Inter/Roboto come heading font
- MAI purple gradient (AI slop)
- MAI "Welcome to" come heading
- MAI card grid identiche senza variazione
- Ogni sezione deve avere un RITMO diverso (non tutte uguali)
