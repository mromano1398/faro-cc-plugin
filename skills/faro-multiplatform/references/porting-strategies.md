# Strategie di porting multipiattaforma

Guida operativa per portare un progetto da una piattaforma all'altra senza
buttare via il lavoro fatto e senza creare un Frankenstein ingestibile.

---

## 1. Filosofia del porting

### Portare vs riscrivere da zero

Il porting ha senso quando:
- Il progetto sorgente è **maturo** e ha logica di business non banale.
- Gli schema dei dati, la validazione e le regole di dominio sono stabili.
- Il team conosce il dominio e vuole risparmiare tempo sulla parte business.
- La piattaforma target ha un pubblico **aggiuntivo**, non sostitutivo.

La riscrittura da zero ha senso quando:
- Il progetto sorgente è un prototipo con logica accoppiata alla UI.
- Le scelte tecniche sono datate (es. classi React + Redux legacy).
- La UX della piattaforma target è radicalmente diversa (mobile-first vs
  desktop-first con tastiera e shortcut).
- Il costo di estrarre la logica è maggiore di quello di riscriverla.

### Cosa è davvero condivisibile

| Condivisibile | Non condivisibile |
|---------------|-------------------|
| Tipi TypeScript | Componenti UI |
| Schema Zod / Valibot | Navigation (router) |
| API client (fetch wrapper) | Gesture / animazioni |
| Business logic pura (funzioni) | Storage locale (localStorage vs SecureStore vs fs) |
| Query / mutation keys (TanStack) | Layout responsive |
| Schema DB (Prisma / Drizzle) | Auth provider (web cookies vs mobile keychain) |
| Costanti e configurazione | Notifiche e permessi |
| Traduzioni (i18n) | File system e path |

Regola d'oro: se una funzione tocca `window`, `document`, `fs`, o un'API
nativa, **non è condivisibile**. Se prende input e restituisce output senza
side effect, lo è.

---

## 2. Percorso 1 — Next.js webapp → Expo mobile

Effort stimato: **M-L** (2-6 settimane a seconda della complessità).

### Setup monorepo con Turborepo

```
my-project/
├── apps/
│   ├── web/                  # Next.js (progetto originale spostato qui)
│   └── mobile/               # Expo (nuovo)
├── packages/
│   ├── shared/               # Tipi, Zod schema, utils puri
│   ├── api-client/           # Client HTTP condiviso
│   └── config/               # ESLint, TS config, tailwind preset
├── package.json              # workspaces
├── turbo.json
└── pnpm-workspace.yaml
```

Passi operativi:
1. `pnpm dlx create-turbo@latest` o converti a mano.
2. Sposta il progetto Next.js attuale in `apps/web` senza modifiche.
3. Crea `packages/shared` con `package.json` che esporta tipi e funzioni pure.
4. Crea `apps/mobile` con `pnpm create expo-app`.
5. Installa le dipendenze shared nei due app workspace.

### Metro config per workspace pnpm (OBBLIGATORIO)

Se il monorepo usa pnpm (e non yarn), il Metro bundler di Expo NON risolve i package
workspace per default. Senza questa config, `import { schema } from '@project/shared'`
fallisce con "Unable to resolve module".

File: `apps/mobile/metro.config.js`

```js
const { getDefaultConfig } = require("expo/metro-config");
const path = require("path");

const projectRoot = __dirname;
const workspaceRoot = path.resolve(projectRoot, "../..");

const config = getDefaultConfig(projectRoot);

// 1. Watch tutti i file del monorepo (non solo apps/mobile)
config.watchFolders = [workspaceRoot];

// 2. Risolvi node_modules da mobile E dalla root del workspace
config.resolver.nodeModulesPaths = [
  path.resolve(projectRoot, "node_modules"),
  path.resolve(workspaceRoot, "node_modules"),
];

// 3. Disabilita hierarchical lookup (pnpm non usa hoisting)
config.resolver.disableHierarchicalLookup = true;

// 4. Risolvi i simlink pnpm
config.resolver.unstable_enableSymlinks = true;
config.resolver.unstable_enablePackageExports = true;

module.exports = config;
```

Aggiungi anche:
- `apps/mobile/package.json` deve avere i package shared come dipendenze:
  `"@project/shared": "workspace:*"`
- Esegui `pnpm install` dalla root del workspace
- Verifica: `pnpm --filter mobile start` → l'app parte senza errori di risoluzione

Senza questi 4 punti, il porting NON funziona. È il gap più bloccante documentato nella Fase 7
di test.

### Cosa estrarre in `packages/shared`

- Tutti i tipi TypeScript che rappresentano entità di dominio
  (`User`, `Order`, `Project`, ecc.).
- Tutti gli schema Zod usati per validazione input.
- Costanti di business (ruoli, stati, enum).
- Funzioni pure: calcoli, formatter, parser, date helper.
- Configurazione API (URL base, endpoint, retry logic).
- Client API tipato (preferibilmente con `ofetch` o `ky`, NON `axios`
  perché pesa e non aggiunge valore su RN).

### Cosa riscrivere in `apps/mobile`

- Tutte le pagine Next.js → schermate Expo Router (file-based routing
  in `app/`, praticamente identico come filosofia ma file diversi).
- Tutti i componenti UI. `shadcn/ui` non funziona su React Native.
  Usa invece: `nativewind` (Tailwind su RN), `react-native-reusables`
  (cloni shadcn per RN), oppure `tamagui` se vuoi cross-platform.
- Form: `react-hook-form` funziona su entrambi, ottimo riuso.
- Navigation: Expo Router copia molto bene la mental model di Next App
  Router. File `_layout.tsx` = `layout.tsx`, `index.tsx` = `page.tsx`.

### Problemi tipici

- **Auth**: NextAuth non gira su Expo. Opzioni: Clerk Expo SDK,
  Supabase Auth, Better Auth, o API token + SecureStore.
- **Cookies**: sul web usi cookie HttpOnly. Su mobile usa
  `expo-secure-store` e manda il token in header `Authorization`.
- **Fetch**: RN ha `fetch` globale ma NON ha streaming nativo completo.
  Attenzione a Server-Sent Events e response.body.
- **Immagini**: `next/image` → `expo-image` (ottima libreria, non
  usare `Image` di react-native se puoi evitarlo).
- **Env**: `process.env.NEXT_PUBLIC_*` → `process.env.EXPO_PUBLIC_*`.
  Crea un file `env.ts` in `packages/shared` che astrae i due.
- **Deep link**: mobile ha scheme custom + universal link. Setup
  separato in `app.json` di Expo.

### Esempio di struttura condivisa

```ts
// packages/shared/src/schemas/user.ts
import { z } from "zod";

export const UserSchema = z.object({
  id: z.string().uuid(),
  email: z.string().email(),
  role: z.enum(["admin", "user", "guest"]),
});

export type User = z.infer<typeof UserSchema>;
```

```ts
// packages/api-client/src/users.ts
import { UserSchema, type User } from "@my-project/shared";

export async function getUser(id: string): Promise<User> {
  const res = await fetch(`${API_URL}/users/${id}`);
  const json = await res.json();
  return UserSchema.parse(json);
}
```

Questo codice gira identico su web e mobile.

---

## 3. Percorso 2 — Next.js webapp → Electron/Tauri desktop

Effort stimato: **S-M** (1-3 settimane).

### Electron — wrapper minimo

L'idea è che la tua webapp gira in una finestra Chromium. Passi:
1. `pnpm add -D electron electron-builder`.
2. Crea `electron/main.ts` che crea una `BrowserWindow` e la punta a
   `http://localhost:3000` in dev o a `out/index.html` in prod.
3. In prod: `next build && next export` (o `output: "export"` nel
   `next.config.js`) → Electron serve i file statici.
4. Se ti servono API route: considera `next start` embedded, oppure
   sposta le API fuori (backend separato).

Pro: setup rapido, riuso 100% della UI web.
Contro: bundle pesante (~150MB), consumo RAM alto, auto-update manuale.

### Tauri — wrapper leggero

Tauri usa il webview del sistema operativo invece di bundlare Chromium.
Richiede Rust installato ma il bundle finale è ~5MB.

1. `pnpm create tauri-app` nella root del monorepo o nuovo progetto.
2. Configura `tauri.conf.json` con il path della tua build web.
3. Per API desktop (file system, menu nativi, notifiche) usa i
   comandi Tauri scritti in Rust, esposti via `invoke()`.

Pro: bundle piccolo, performance migliori, più sicuro (capability-based).
Contro: serve Rust, ecosistema più piccolo, webview diverso per OS
(rischio di bug cross-OS).

### Quando scegliere cosa

| Scenario | Scelta |
|----------|--------|
| MVP veloce, team solo JS | Electron |
| Bundle piccolo, distribuzione pubblica | Tauri |
| Serve tanta integrazione OS nativa | Tauri (comandi Rust) |
| Supporto enterprise consolidato | Electron |
| Riuso esatto del rendering del browser | Electron |

### Cosa cambia rispetto al web

- **Routing**: niente URL pubblici, usa hash router o memory router.
- **Storage**: `localStorage` funziona, ma per dati strutturati usa
  SQLite locale (`better-sqlite3` in Electron, `tauri-plugin-sql` in Tauri).
- **Menu**: menu nativo va configurato (Electron: `Menu.setApplicationMenu`,
  Tauri: in `tauri.conf.json`).
- **Auto-update**: Electron ha `electron-updater`, Tauri ha
  `tauri-plugin-updater`. Entrambi richiedono un endpoint di release.
- **Firma codice**: richiesta per distribuzione (Apple Developer Account,
  certificato Windows). Non scordarla, costa e richiede tempo.

---

## 4. Percorso 3 — Expo mobile → Next.js webapp

Effort stimato: **M-L** (2-5 settimane).

È l'inverso del Percorso 1, ma con una differenza chiave: dipende
molto da quanto è ben strutturato il progetto mobile sorgente.

### Caso A — Mobile già ben strutturato

Se il progetto Expo ha già la logica in un package separato (o in
`src/lib` senza accoppiamento alla UI), il porting è "solo":
1. Crea `apps/web` Next.js nel monorepo.
2. Importa gli stessi package condivisi.
3. Scrivi la UI web (shadcn/ui, tailwind).
4. Gestisci le differenze di auth e storage.

### Caso B — Mobile monolitico

Se il progetto mobile mescola logica e UI (caso comune in app nate
velocemente), serve uno step preliminare:
1. **Estrazione**: crea `packages/shared` e sposta i a mano tipi, Zod
   schema, funzioni pure, API client.
2. Refactor del mobile per usare il package (no breaking change visibili).
3. **Solo dopo**, crea `apps/web` che importa gli stessi package.

Non saltare lo step di estrazione: portare a scatola chiusa il codice
copiando e incollando crea due progetti che divergono dopo 3 mesi.

### Differenze da gestire

- **Navigation**: Expo Router ha `Stack`, `Tabs`, modali. Next.js usa
  layout nested e parallel routes. Non è traduzione 1:1, serve ripensare.
- **Forms**: `react-hook-form` riusabile. Validazione Zod riusabile.
  Solo i componenti input cambiano.
- **Auth**: se usi Clerk, Supabase o Better Auth hai SDK per entrambi.
  Se usi auth custom mobile, rivedi il flow per il web (cookie + CSRF).
- **Push notification**: il web ha Web Push, il mobile ha APNs/FCM.
  Sono due sistemi diversi, non pensare di unificarli.

---

## 5. Percorso 4 — Qualsiasi → PWA

Effort stimato: **S** (3-5 giorni).

La PWA non è una piattaforma nuova, è la webapp che si comporta
meglio offline e può essere installata. Spesso è il primo step prima
di considerare un'app nativa.

### Step concreti

1. **Manifest**: crea `public/manifest.json` con nome, icone (192x192,
   512x512, maskable), `display: "standalone"`, `theme_color`,
   `background_color`, `start_url`.
2. **Service worker**: su Next.js usa `@ducanh2912/next-pwa` (mantenuto,
   `next-pwa` originale è abbandonato) oppure Workbox direttamente.
3. **Offline**: decidi la strategia per ogni risorsa:
   - HTML: network-first con fallback cache.
   - Asset statici: cache-first.
   - API: network-only o stale-while-revalidate a seconda del caso.
4. **Add-to-home-screen**: l'utente lo fa manualmente. Puoi mostrare
   un prompt personalizzato usando `beforeinstallprompt` (solo Chrome).
5. **Icone iOS**: iOS ignora il manifest per molte cose, devi mettere
   `<link rel="apple-touch-icon">` e meta tag separati.

### Quando la PWA basta

- L'app è principalmente online.
- Non ti serve accesso profondo a API native (camera avanzata,
  bluetooth, healthkit, background task).
- Il pubblico usa principalmente Android (iOS ha supporto PWA limitato).
- Budget basso o MVP.

### Quando serve davvero nativo

- Push notification su iOS (le Web Push iOS funzionano solo da
  home-screen install, con restrizioni).
- Accesso a healthkit, contatti, calendario, bluetooth LE.
- Performance grafiche (giochi, AR, video editing).
- Presenza negli app store è un requisito business.
- Deep link con scheme custom.

---

## 6. Struttura monorepo consigliata (Turborepo)

Per progetti con più di un target, la struttura consigliata è:

```
my-project/
├── apps/
│   ├── web/             # Next.js
│   ├── mobile/          # Expo
│   └── desktop/         # Electron o Tauri
├── packages/
│   ├── shared/          # Tipi, Zod schema, utils puri, costanti
│   ├── api-client/      # Client API condiviso (fetch tipato)
│   ├── ui-web/          # Componenti solo web (shadcn + tailwind)
│   ├── ui-mobile/       # Componenti solo mobile (nativewind)
│   ├── db/              # Schema Prisma/Drizzle + migration
│   └── config/          # ESLint, TS, Tailwind preset condivisi
├── turbo.json
├── pnpm-workspace.yaml
└── package.json
```

Note operative:
- Usa **pnpm** come package manager (non npm/yarn): gestisce workspaces
  molto meglio e non rompe con Expo.
- In `turbo.json` configura pipeline `build`, `dev`, `lint`, `test`
  con le dipendenze corrette tra package.
- Evita di mettere `ui` condivisa tra web e mobile nello stesso package:
  meglio due package separati con naming chiaro (`ui-web`, `ui-mobile`).
- Il package `db` ha senso solo se tutti gli app parlano allo stesso DB;
  altrimenti tienilo nel backend.

---

## 7. Anti-pattern del porting

Evita questi errori classici:

- **Condividere componenti UI cross-platform**. React Native Web esiste
  ma nella pratica produce UI che non sembra né web né mobile. Peggio:
  paghi un costo di dipendenze e bundle su entrambi.
- **Forzare un design system identico** su web e mobile. Le piattaforme
  hanno convenzioni diverse (bottom tab su mobile, top nav su desktop).
  Il brand si preserva nei colori e nella tipografia, non nei layout.
- **Skippare il wizard/adopt sul nuovo target**. Ogni piattaforma ha le
  sue regole: rigenera le rules Cursor/Claude, aggiorna il CLAUDE.md del
  nuovo app, non pensare che "tanto è lo stesso progetto".
- **Deploy accoppiato**. Se il web va giù per un deploy sbagliato, il
  mobile non deve essere coinvolto. Release indipendenti, CI separata
  per app, versioning indipendente.
- **Copia-incolla del codice** senza passare dal package shared.
  Dopo 3 mesi i due progetti divergono e il bug fix va fatto due volte.
- **Pensare che il porting è "solo UI"**. Auth, storage, notifiche,
  analytics, error reporting, feature flags: tutti vanno rivisti.
- **Ignorare le convenzioni store**. App Store e Play Store hanno
  regole sulla privacy, permission, pagamenti (IAP vs Stripe) che
  non esistono sul web.

---

## 8. Checklist pre-porting

Prima di partire con un porting, rispondi onestamente a queste domande:

- [ ] Il progetto sorgente ha logica già separata dalla UI, o devo
      rifattorizzare prima?
- [ ] Gli schema Zod esistono e sono effettivamente condivisibili, o
      sono mescolati con logica di presentazione?
- [ ] Il sistema di auth attuale funziona sulla piattaforma target,
      o serve un provider nuovo?
- [ ] C'è budget per mantenere 2 (o 3) target invece di 1? Ogni target
      raddoppia il lavoro di bug fix, release, monitoring.
- [ ] Il team ha competenze sulla piattaforma target? (React Native
      non è React, Electron non è Next.js, serve learning curve)
- [ ] La UX della piattaforma target è stata pensata, o sto solo
      "mettendo lo stesso prodotto su un altro schermo"?
- [ ] Gli analytics e l'error tracking hanno SDK per il target?
- [ ] I pagamenti funzionano sul target? (Mobile = IAP obbligatori per
      contenuto digitale, commissioni 15-30%)
- [ ] Il backend è già pronto per servire più client, o assume cookie
      + CSRF del web?
- [ ] Ho pensato alla strategia di versioning indipendente tra app?

Se rispondi "no" a più di 3 domande, il porting è prematuro:
prima rimetti in ordine il sorgente, poi affronta il nuovo target.
