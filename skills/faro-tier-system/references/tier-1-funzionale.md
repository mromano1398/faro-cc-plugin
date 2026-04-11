# Tier 1 — FUNZIONALE

## Quando usare
Gestionale, ERP, tool interni, CRM, dashboard operative.

## Stack design
- shadcn/ui componenti base
- Tailwind CSS
- Lucide icons
- ZERO animazioni (tranne toast/dialog fade)

## Layout
- Desktop-first
- Sidebar fissa a sinistra (collassabile)
- Header con breadcrumb + ricerca Cmd+K
- Tab interni per sotto-sezioni
- DataTable per liste con sort/filter/pagination

## Componenti richiesti
Button, Input, Select, Textarea, Card, Badge, Table, Dialog, Toast, Form, Tabs,
Breadcrumb, Sidebar, Command (Cmd+K), Calendar, DatePicker

## Palette
Professionale e neutrale. Sfondo bianco (#FFFFFF) o grigio chiaro (#FAFAFA).
Accent: un colore aziendale/brand. Testo: #0A0A0A.
MAI usare colori default/generici — sempre personalizzare per il brand.

## Tipografia
Font: Inter, system-ui, o un font aziendale.
Pesi: 400 (body), 500 (label), 600 (heading), 700 (titolo pagina).
Dimensioni: text-sm per tabelle, text-base per body, text-lg per heading.

## Responsive
Desktop-first. Su mobile: sidebar diventa menu hamburger.
Tabelle su mobile: card view o scroll orizzontale.

## Navigazione
Sidebar con icone + testo → su collapse solo icone.
Breadcrumb sempre visibile.
Cmd+K per navigazione rapida.
Tab interni per sotto-sezioni (es: Dettaglio → Info | Documenti | Storico).

## Dark mode
Opzionale. Se presente: sfondo #09090b, card #1a1a1a, bordi #27272a.

## Regole assolute
- ZERO animazioni decorative
- NO gradient
- NO ombre eccessive (max shadow-sm)
- NO bordi arrotondati eccessivi (max rounded-md)
- Focus su LEGGIBILITÀ e VELOCITÀ
