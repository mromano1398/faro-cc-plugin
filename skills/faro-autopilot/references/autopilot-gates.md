# Autopilot Gates — Riferimento dettagliato

Questo documento descrive i 4 gate obbligatori dell'autopilot Faro, con esempi
concreti di cosa li triggera, come vengono presentati all'utente e come NON
devono essere gestiti.

---

## 1. Introduzione — Filosofia dei gate

L'autopilot Faro è progettato per eseguire piani interi senza intervento umano.
Questo è potente, ma pericoloso: un agente autonomo può rapidamente causare
danni irreversibili se non viene fermato nei momenti critici.

### Reversibile vs Irreversibile

- **Reversibile**: creare un file, modificare codice in un branch feature,
  installare una dipendenza, aggiungere un componente UI. Si può annullare con
  `git checkout`, `npm uninstall`, o semplicemente rimuovendo il file.
- **Irreversibile**: eliminare righe da una tabella prod, pushare in produzione,
  droppare colonne, cancellare file senza backup, pubblicare un pacchetto npm,
  aggiungere un abbonamento mensile. Nessun comando inverso riporta allo stato
  precedente senza perdita di dati, tempo, denaro o reputazione.

### Perché l'utente DEVE confermare

L'autopilot non ha contesto di business: non sa quanto vale un record, quanti
utenti sta servendo, qual è il budget del cliente. Il gate trasferisce la
decisione finale a chi ha questo contesto. È una protezione esplicita: l'utente
può sempre dire "sì", ma la conferma genera un checkpoint mentale.

**Regola aurea**: se un'operazione è irreversibile o costosa, DEVE esserci un
gate. Nessuna eccezione, nessun raggruppamento, nessun "l'utente ha già
autorizzato tutto".

---

## 2. Gate DB

### Trigger

- Istruzioni SQL: `CREATE TABLE`, `ALTER TABLE`, `DROP TABLE`, `DROP COLUMN`,
  `RENAME COLUMN`, `TRUNCATE`.
- Modifiche a `prisma/schema.prisma` che cambiano model, field, relation o
  enum.
- Creazione di nuovi file in `supabase/migrations/*.sql`.
- Modifiche a `schema.sql` o equivalenti.
- Esecuzione di `prisma migrate deploy`, `prisma db push`, `supabase db push`.

### Dev DB locale vs Prod DB

Il gate DB si applica SEMPRE, ma l'output differenzia il livello di rischio:

- **Dev DB locale** (sqlite, docker postgres locale, supabase start): gate con
  avviso leggero, reversibile tramite reset.
- **Prod DB** (Supabase prod, Neon prod, RDS): gate pieno, irreversibile,
  richiede backup verificato prima di procedere.

### Esempio 1 — CREATE TABLE in dev

```
GATE DB — Richiede conferma

Operazione: CREATE TABLE "Invoice" (id, customer_id, amount, created_at)
Target: Dev DB locale (postgresql://localhost:5432/faro_dev)
Impatto: Aggiunge una nuova tabella vuota. Nessun dato esistente a rischio.
Irreversibile: No (reversibile con DROP TABLE o db reset)

Confermi?
```

### Esempio 2 — DROP COLUMN in prod

```
GATE DB — Richiede conferma (PROD)

Operazione: ALTER TABLE "User" DROP COLUMN "phone"
Target: Prod DB (Supabase — progetto faro-prod)
Impatto: ELIMINA DEFINITIVAMENTE la colonna "phone" e tutti i suoi dati
         (stima: ~12.400 righe con valore non-null).
Irreversibile: SI. Serve backup pre-operazione.

Hai un backup recente? Confermi?
```

### Esempio 3 — Migrazione Prisma con rename

```
GATE DB — Richiede conferma

Operazione: prisma migrate deploy
Cambiamenti rilevati in schema.prisma:
  - Model "Order": field "status" rinominato in "orderStatus"
  - Model "Product": nuovo field "sku" (required, no default)
Target: Prod DB
Impatto: Rename è sicuro se gestito come rename in migration SQL.
         Il nuovo field required SENZA default fallirà su righe esistenti.
Irreversibile: Parzialmente (dipende dalla migration generata).

Vuoi che riveda la migration prima di eseguirla?
```

---

## 3. Gate Deploy

### Trigger

- `vercel deploy --prod`, `vercel --prod`.
- `git push origin main` SE il branch `main` è collegato ad auto-deploy prod.
- `docker push <registry>/<image>:<tag>` su registry di produzione.
- `npm publish` (qualsiasi pacchetto pubblico).
- Comandi di deploy custom rilevati dal `package.json` (script con nomi come
  `deploy`, `release`, `publish`, `ship`).
- `gh release create`, `gh workflow run` di workflow di deploy prod.

### Esempio 1 — Deploy Vercel prod

```
GATE DEPLOY — Richiede conferma

Operazione: vercel deploy --prod
Target: faro-app.vercel.app (production)
Ultimo deploy prod: 2 giorni fa (v1.4.2)
Modifiche da deployare: 8 commit, 23 file modificati
Test locali: passati (build + typecheck + test)
Impatto: Sostituisce immediatamente la versione live.
Irreversibile: No (rollback disponibile tramite vercel rollback).

Confermi il deploy?
```

### Esempio 2 — git push su main con auto-deploy

```
GATE DEPLOY — Richiede conferma

Operazione: git push origin main
Target: main → auto-deploy su faro-app.vercel.app (prod)
Commit da pushare: 3
  - a1b2c3d "feat: nuova tabella clienti"
  - d4e5f6g "fix: validazione IVA"
  - g7h8i9j "refactor: API handler"
Impatto: Trigger automatico di deploy prod via Vercel Git Integration.
Irreversibile: No (rollback disponibile).

Confermi?
```

### Esempio 3 — npm publish

```
GATE DEPLOY — Richiede conferma

Operazione: npm publish
Pacchetto: @faro/sdk@2.0.0
Registry: https://registry.npmjs.org
Impatto: Pubblicazione pubblica. La versione 2.0.0 sarà immediatamente
         installabile da chiunque. La versione è un MAJOR (breaking changes).
Irreversibile: SI. `npm unpublish` è consentito solo entro 72h e sconsigliato.

Hai aggiornato il CHANGELOG? Confermi?
```

---

## 4. Gate Distruttivo

### Trigger

- `rm -rf`, `rmdir /s`, `Remove-Item -Recurse -Force`.
- Eliminazione massiva di file (>10 file in una sola operazione).
- Eliminazione di un intero modulo/feature dal progetto (es: `rm -rf
  src/features/billing`).
- Downgrade di dipendenze major (es: Next.js 15 → 14).
- `git reset --hard`, `git push --force`, `git push --force-with-lease` su
  branch condivisi.
- `DELETE FROM <table>` senza clausola `WHERE`, o con `WHERE` che matcha
  tutto.
- `TRUNCATE`, `DROP DATABASE`.

### Esempio 1 — rm -rf di una feature

```
GATE DISTRUTTIVO — Richiede conferma

Operazione: rm -rf src/features/legacy-billing/
Impatto: Elimina definitivamente 47 file (~3.200 righe di codice).
         Include componenti UI, API handlers, test, seed data.
Irreversibile: Parzialmente (recuperabile da git se committato).
Ultimo commit che toccava questi file: 4 giorni fa.

Ci sono riferimenti a legacy-billing in altre parti del progetto?
Confermi l'eliminazione?
```

### Esempio 2 — git push --force su main

```
GATE DISTRUTTIVO — Richiede conferma (ALTO RISCHIO)

Operazione: git push --force origin main
Impatto: SOVRASCRIVE la storia del branch main remoto.
         Commit che verrebbero persi: 2
           - x1y2z3 "hotfix: bug pagamento" (autore: Marco, 3h fa)
           - z4w5v6 "docs: update readme" (autore: Lucia, 1h fa)
Irreversibile: SI per gli altri collaboratori.

ATTENZIONE: ci sono commit di altri autori che andranno persi.
Sei sicuro di voler forzare il push?
```

### Esempio 3 — DELETE senza WHERE

```
GATE DISTRUTTIVO — Richiede conferma

Operazione: DELETE FROM "SessionLog"
Target: Prod DB
Impatto: Elimina TUTTE le righe della tabella SessionLog
         (stima: ~184.000 righe).
Irreversibile: SI senza backup.

Volevi forse aggiungere una clausola WHERE (es: created_at < NOW() - 30 days)?
Confermi l'eliminazione totale?
```

---

## 5. Gate Costo

### Trigger

- Aggiunta di servizi a pagamento non free-tier (es: Algolia Standard, Sentry
  Team, Resend Pro).
- Upgrade piani (Supabase Free → Pro, Vercel Hobby → Pro, Neon Free → Scale).
- Acquisto di dominio, certificati, add-on.
- Superamento del budget dichiarato dall'utente nel wizard iniziale.
- Abilitazione di servizi con costo variabile non capped (es: OpenAI API
  senza spending limit).

### Esempio 1 — Upgrade Supabase Pro

```
GATE COSTO — Richiede conferma

Operazione: Upgrade progetto Supabase da Free a Pro
Motivazione: superato limite 500 MB database (attuale: 487 MB, crescita ~20 MB/settimana)
Costo stimato: $25/mese + $0.125/GB oltre 8 GB
Budget dichiarato: $50/mese totale infra
Budget già utilizzato: $20 (Vercel Pro)
Reversibile: Sì (downgrade possibile ma con perdita feature Pro)

Confermi l'upgrade?
```

### Esempio 2 — Aggiunta servizio a pagamento

```
GATE COSTO — Richiede conferma

Operazione: Integrazione Algolia Search (piano Build)
Motivazione: il task richiede ricerca full-text performante
Costo stimato: $0 fino a 10k record e 10k ricerche/mese,
               poi $0.50 per 1k ricerche aggiuntive
Budget dichiarato: $50/mese totale infra
Reversibile: Sì (rimozione integrazione, ma codice da riscrivere)

Alternative free valutate: Postgres full-text search, Meilisearch self-hosted.
Preferisci procedere con Algolia o valutare le alternative?
```

### Esempio 3 — API con costo variabile

```
GATE COSTO — Richiede conferma

Operazione: Abilitazione OpenAI API (modello gpt-4o)
Costo stimato: $2.50/1M token input, $10/1M token output
Uso previsto: ~50k token/giorno (stima dal task corrente)
Costo mensile previsto: ~$15-20
Budget dichiarato: $50/mese
Reversibile: Sì (disabilitazione API key)

Vuoi impostare uno spending limit su OpenAI dashboard prima di procedere?
```

---

## 6. Formato standard di conferma gate

Tutti i gate devono usare questo template:

```
⚠️ GATE [TIPO] — Richiede conferma

Operazione: [dettaglio preciso del comando/azione]
Impatto: [dati/utenti/costi coinvolti, con numeri quando possibile]
Reversibile: [sì/no/parziale + come]
Costo stimato: [solo per gate costo]

Vuoi procedere? [sì/no/modifica]
```

Regole di formato:
- `TIPO` è uno di: `DB`, `DEPLOY`, `DISTRUTTIVO`, `COSTO`.
- `Operazione` deve contenere il comando letterale che verrebbe eseguito.
- `Impatto` deve essere quantitativo (numero righe, numero file, stima utenti).
- `Reversibile` non può essere omesso.
- La domanda finale accetta 3 risposte: `sì`, `no`, `modifica` (per chiedere di
  cambiare i parametri dell'operazione).

---

## 7. Anti-pattern dei gate

L'autopilot NON deve comportarsi così:

### Non raggruppare più gate in un'unica conferma

SBAGLIATO:
```
Sto per: creare tabella Invoice, deployare in prod, eliminare vecchia tabella.
Confermi tutto?
```

CORRETTO: tre gate separati, in sequenza, uno alla volta.

### Non saltare un gate "perché l'utente ha già confermato prima"

Ogni gate è atomico. Se l'utente ha confermato un deploy 10 minuti fa, il
prossimo deploy richiede una nuova conferma. Nessuna "sessione di fiducia".

### Non proseguire dopo "sì" se qualcosa è cambiato dall'ultimo checkpoint

Se tra la presentazione del gate e l'esecuzione:
- sono passati più di 5 minuti
- sono cambiati file nel working directory
- è stato eseguito un altro comando non previsto

l'autopilot DEVE ri-presentare il gate.

### Non interpretare risposte ambigue come "sì"

- "ok" → chiedi conferma esplicita (sì/no)
- "va bene" → chiedi conferma esplicita
- "procedi" → ACCETTATO
- "sì" → ACCETTATO
- "forse" → rifiuta, chiedi chiarimento

### Non nascondere dettagli per "non spaventare"

L'impatto va sempre mostrato nella sua interezza, anche se sembra allarmante.
Se un `DELETE` elimina 184k righe, va scritto "184k righe", non "alcuni record".

---

## 8. Comandi utente per gestire l'autopilot

Durante l'esecuzione autopilot, l'utente può digitare questi comandi in
qualsiasi momento:

| Comando | Effetto |
|---|---|
| `stop` | Ferma immediatamente l'autopilot. Nessuna pulizia, nessun commit. Lo stato corrente resta così com'è. |
| `pausa` | Mette in pausa l'autopilot, salva lo stato del piano in `.faro/autopilot-state.json`. Si può riprendere con `riprendi`. |
| `riprendi` | Riprende dal punto in cui era stato messo in pausa, legge lo stato salvato. |
| `skip task N` | Salta il task numero N del piano attivo, marca come "skipped" nel log. |
| `solo questo gate` | Approva UN gate specifico ma ferma l'autopilot subito dopo, così l'utente riprende il controllo al prossimo gate. |
| `status` | Mostra stato corrente: task in corso, task completati, prossimo gate atteso. |
| `log` | Mostra le ultime 20 righe di `.faro/log.md`. |

### Note operative

- I comandi sono case-insensitive.
- `stop` è sempre disponibile e ha priorità su tutto.
- `pausa` non interrompe un task a metà: aspetta il completamento del task
  corrente prima di fermarsi.
- `skip task N` funziona solo per task non ancora iniziati.
- Dopo `solo questo gate`, l'autopilot torna in modalità AP (singolo task),
  l'utente deve rilanciare `AP FULL` per riattivare l'esecuzione continua.
