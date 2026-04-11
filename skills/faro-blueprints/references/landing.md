# Blueprint — Landing Page

## Struttura
```
src/
├── app/
│   ├── page.tsx                    # Pagina unica con sezioni
│   ├── layout.tsx
│   └── globals.css
├── components/
│   ├── sections/
│   │   ├── hero.tsx
│   │   ├── features.tsx
│   │   ├── social-proof.tsx
│   │   ├── pricing.tsx
│   │   ├── faq.tsx
│   │   ├── cta.tsx
│   │   └── footer.tsx
│   └── ui/
```

Alternativa con Astro:
```
src/
├── pages/
│   └── index.astro
├── components/
│   ├── Hero.astro
│   ├── Features.astro
│   └── ...
├── layouts/
│   └── Layout.astro
```

## Sezioni tipiche (in ordine)
1. Hero: headline + sub + CTA + visual
2. Social proof: loghi clienti o "Usato da X persone"
3. Features: 3-6 feature con icona + titolo + descrizione
4. How it works: 3 step con numeri
5. Testimonials: quote con foto + nome + ruolo
6. Pricing: 2-3 piani
7. FAQ: accordion
8. CTA finale: ripeti la CTA principale

## SEO specifico
- Meta title/description ottimizzati
- Structured data: Organization, Product, FAQ
- Open Graph + Twitter Card
- Sitemap.xml
- robots.txt
