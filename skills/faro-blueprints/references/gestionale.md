# Blueprint — Gestionale / ERP

## Struttura cartelle (Next.js App Router)
```
src/
├── app/
│   ├── (auth)/
│   │   ├── login/page.tsx
│   │   └── layout.tsx
│   ├── (dashboard)/
│   │   ├── layout.tsx              # Sidebar + header
│   │   ├── page.tsx                # Dashboard home
│   │   ├── [modulo]/               # Pattern dinamico per moduli
│   │   │   ├── page.tsx            # Lista
│   │   │   ├── nuovo/page.tsx      # Creazione
│   │   │   └── [id]/
│   │   │       ├── page.tsx        # Dettaglio
│   │   │       └── modifica/page.tsx
│   │   └── impostazioni/page.tsx
│   ├── api/
│   │   └── [modulo]/route.ts       # API per modulo
│   ├── layout.tsx
│   └── globals.css
├── components/
│   ├── ui/                         # shadcn components
│   ├── layout/
│   │   ├── sidebar.tsx
│   │   ├── header.tsx
│   │   └── breadcrumb.tsx
│   ├── forms/                      # Form riutilizzabili
│   └── data/                       # DataTable, filtri
├── lib/
│   ├── db.ts                       # Prisma client
│   ├── auth.ts                     # Auth config
│   ├── validations/                # Zod schemas per modulo
│   └── utils.ts
├── types/
└── prisma/
    └── schema.prisma
```

## Navigazione
- Sidebar fissa con icone + testo (collapsible)
- Breadcrumb: Home > Modulo > Azione
- Cmd+K per navigazione rapida
- Tab interni per sotto-sezioni del dettaglio

## Sicurezza specifica
- RBAC (admin, manager, operatore, visualizzatore)
- WHERE user_id / WHERE tenant_id su OGNI query
- Audit log su operazioni CRUD critiche
- Session timeout configurabile

## UX specifica
- Dashboard con KPI card + tabella attività recenti
- Liste con DataTable + filtri + ricerca + export CSV
- Form con validazione Zod in tempo reale
- Azioni batch (seleziona multipli → azione)
- Scorciatoie tastiera per azioni frequenti
- Conferma su azioni distruttive (elimina, annulla)
