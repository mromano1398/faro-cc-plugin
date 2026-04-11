# Blueprint — E-commerce

## Struttura
```
src/
├── app/
│   ├── (shop)/
│   │   ├── page.tsx                # Home shop
│   │   ├── prodotti/
│   │   │   ├── page.tsx            # Catalogo
│   │   │   └── [slug]/page.tsx     # Dettaglio prodotto
│   │   ├── categorie/[slug]/page.tsx
│   │   ├── carrello/page.tsx
│   │   ├── checkout/page.tsx
│   │   └── ordini/page.tsx         # I miei ordini (auth)
│   ├── (admin)/
│   │   ├── prodotti/page.tsx
│   │   ├── ordini/page.tsx
│   │   └── analytics/page.tsx
│   ├── api/
│   │   ├── webhooks/stripe/route.ts
│   │   ├── prodotti/route.ts
│   │   └── ordini/route.ts
│   └── layout.tsx
├── components/
│   ├── shop/                       # ProductCard, CartDrawer, CheckoutForm
│   ├── admin/                      # OrderTable, ProductForm
│   └── ui/
├── lib/
│   ├── cart/                       # Cart state (zustand)
│   ├── stripe/
│   └── search/                     # Meilisearch client
```

## Sicurezza specifica
- Inventory lock su checkout (evita overselling)
- Stripe Checkout (MAI gestire carte direttamente)
- Validazione quantità/prezzo server-side
- Rate limiting su checkout

## UX specifica
- Ricerca prodotti con autocomplete
- Filtri: prezzo, categoria, disponibilità
- Cart drawer (non pagina separata)
- Urgency: "Solo 3 rimasti", "Spedizione gratuita sopra X€"
- Cross-sell: "Chi ha comprato questo ha anche..."
- Wishlist
