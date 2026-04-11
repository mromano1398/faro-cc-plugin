# Menu Faro — Riferimento Completo

Questo documento descrive in dettaglio ogni voce del menu dell'orchestratore Faro.
Viene letto dall'orchestratore principale solo quando serve approfondire una voce specifica,
così da mantenere `SKILL.md` compatto.

Ogni sezione segue la stessa struttura:

- **Comando/skill invocato** — cosa viene eseguito concretamente
- **Quando usarlo** — situazioni tipiche
- **Prerequisiti** — cosa deve esistere nel progetto
- **Step-by-step** — cosa accade quando la voce viene attivata
- **Output atteso** — cosa trova l'utente al termine
- **Trigger frasali** — esempi di frasi che devono attivare questa voce

---

## [INIT] — Inizializza nuovo progetto

**Comando/skill invocato:** `faro-wizard`

**Quando usarlo:**
Quando l'utente sta partendo da zero e non esiste ancora alcun codice significativo
nel repository, oppure quando la cartella è vuota o contiene solo file di scaffolding
generati automaticamente (README vuoto, .gitignore di default, licenza).

**Prerequisiti:**
- Nessun `.claude/rules/` esistente
- Nessun `.faro/state.md`
- Struttura del progetto vuota o minimale
- L'utente deve avere almeno un'idea di cosa vuole costruire (anche vaga)

**Step-by-step:**
1. L'orchestratore invoca la skill `faro-wizard`
2. Il wizard chiede all'utente una descrizione sintetica del progetto (2-3 frasi)
3. Il wizard determina il tipo di progetto (web app, API, mobile, CLI, desktop, ecc.)
4. Il wizard propone uno stack di partenza e chiama `faro-stack-advisor` per confermarlo
5. Il wizard invoca `faro-tier-system` per la scelta del tier di design (1-4)
6. Il wizard genera i file di configurazione: `CLAUDE.md`, `.claude/rules/`, `.faro/state.md`
7. Il wizard crea un primo piano iniziale in `.faro/plans/00-bootstrap.md`
8. L'orchestratore mostra il menu e suggerisce di passare ad `AP` per avviare l'autopilot

**Output atteso:**
- Cartelle `.claude/rules/` e `.faro/` popolate
- `CLAUDE.md` con regole di progetto, design system e stack
- Primo piano in `.faro/plans/`
- Score iniziale calcolato
- Utente orientato sul prossimo passo

**Trigger frasali:**
- "Voglio iniziare un nuovo progetto"
- "Faro, inizializza"
- "Parto da zero, aiutami"
- "Non ho ancora niente, voglio creare un'app"

---

## [ADOPT] — Adotta progetto esistente

**Comando/skill invocato:** `faro-adopt`

**Quando usarlo:**
Quando il repository contiene già del codice (anche parziale o legacy) ma non c'è
alcuna configurazione Faro precedente. È il percorso tipico quando Faro viene installato
su un progetto in corso o su un legacy da svecchiare.

**Prerequisiti:**
- Codice esistente nel repository (src/, app/, lib/, ecc.)
- Nessun `.claude/rules/` già configurato da Faro
- Idealmente un `package.json`, `pyproject.toml` o file equivalente di manifest

**Step-by-step:**
1. L'orchestratore invoca `faro-adopt`
2. La skill scansiona il progetto: struttura cartelle, dipendenze, linguaggi usati
3. Identifica framework e versione (es. Next.js 15, Django 5, Expo SDK 52)
4. Deduce il tier di design attualmente in uso osservando `globals.css`, `tailwind.config`, ecc.
5. Rileva eventuali problemi critici (TS non strict, mancanza di lint, auth fatta a mano)
6. Genera automaticamente `CLAUDE.md` e `.claude/rules/` coerenti con quello che trova
7. Crea `.faro/state.md` con score iniziale e lista di debito tecnico rilevato
8. Propone all'utente un primo piano di "stabilizzazione" in `.faro/plans/`

**Output atteso:**
- Configurazione Faro allineata al codice esistente (non al contrario)
- Lista di osservazioni sul debito tecnico
- Primo piano di stabilizzazione opzionale
- Score realistico del progetto così com'è

**Trigger frasali:**
- "Ho già un progetto, adottalo"
- "Faro, configurati su questo codice"
- "Aggiungi Faro a un progetto esistente"
- "Analizza questo repo e genera le regole"

---

## [STATUS] — Stato e score

**Comando/skill invocato:** `/project:status`

**Quando usarlo:**
Quando l'utente vuole una fotografia istantanea dello stato del progetto: a che punto
è, quale piano è attivo, qual è lo score corrente. È la voce più rapida e non modifica
nulla nel progetto.

**Prerequisiti:**
- `.faro/state.md` esistente (quindi INIT o ADOPT già eseguiti)

**Step-by-step:**
1. Legge `.faro/state.md`
2. Legge eventuali piani attivi in `.faro/plans/`
3. Ricalcola lo score su fondamenta / sicurezza / design / funzionalità / qualità
4. Mostra una tabella riassuntiva con i 5 assi dello score
5. Mostra il piano attivo con i task completati e quelli rimanenti
6. Suggerisce la prossima azione naturale (di solito `AP` o continuare il piano)

**Output atteso:**
Un report compatto in chat con:
- Nome progetto, fase corrente, score /100
- Dettaglio score per asse
- Stato piano attivo (x/y task)
- Prossima azione suggerita

**Trigger frasali:**
- "A che punto siamo?"
- "Stato progetto"
- "Status"
- "Qual è lo score?"
- "Da dove riprendo?"

---

## [TIER] — Cambia tier design

**Comando/skill invocato:** `faro-tier-system`

**Quando usarlo:**
Quando l'utente vuole cambiare il livello di ambizione del design system, senza
necessariamente fare un reskin completo. Utile all'inizio del progetto o quando si
decide di alzare l'asticella (es. da MVP a prodotto lanciabile).

**Prerequisiti:**
- Progetto inizializzato (INIT o ADOPT già eseguiti)
- `CLAUDE.md` presente per aggiornare il riferimento al tier

**Step-by-step:**
1. La skill mostra i 4 tier disponibili con descrizione sintetica:
   - Tier 1: funzionale/wireframe
   - Tier 2: pulito/professionale
   - Tier 3: distintivo/brandizzato
   - Tier 4: editoriale/awwwards-grade
2. Chiede all'utente quale tier desidera
3. Aggiorna `CLAUDE.md` e `.claude/rules/design-system.md` con nuovi token
4. Non tocca il codice esistente: serve `RESKIN` per propagare
5. Aggiorna `.faro/state.md` registrando il cambio tier

**Output atteso:**
- Tier aggiornato nelle regole Faro
- Utente informato che serve `RESKIN` per applicare il nuovo tier al codice

**Trigger frasali:**
- "Cambia tier"
- "Che livello di design voglio?"
- "Alza il design a tier 3"
- "Voglio qualcosa di più distintivo"

---

## [STACK] — Valuta/cambia stack

**Comando/skill invocato:** `faro-stack-advisor`

**Quando usarlo:**
Quando l'utente vuole una raccomandazione ragionata sullo stack tecnico, sia in fase
iniziale sia per valutare una migrazione (es. da Pages Router a App Router, da Express
a Hono, da REST a tRPC).

**Prerequisiti:**
- Nessuno. Funziona anche su progetti vuoti (consiglio iniziale) o già avviati (consiglio di migrazione).

**Step-by-step:**
1. La skill chiede tipo di progetto, requisiti principali, vincoli noti
2. Propone uno stack consigliato con motivazione per ogni scelta
3. Propone 1-2 alternative con pro/contro
4. Se il progetto esiste già, confronta lo stack attuale con quello consigliato
5. Stima il costo di migrazione (basso/medio/alto) per ciascuna sostituzione
6. Scrive il risultato in `.faro/stack-decision.md`

**Output atteso:**
- Raccomandazione chiara su framework, lingua, ORM, auth, hosting
- Eventuale piano di migrazione ad alto livello

**Trigger frasali:**
- "Che stack uso?"
- "Consigliami le tecnologie"
- "Vale la pena migrare?"
- "Stack advisor"

---

## [RESKIN] — Cambia design senza rompere

**Comando/skill invocato:** `/project:reskin`

**Quando usarlo:**
Quando l'utente vuole cambiare l'aspetto visivo del progetto (palette, tipografia,
spaziature, componenti) senza toccare la logica di business. È l'operazione più rischiosa
insieme all'autopilot e richiede attenzione.

**Prerequisiti:**
- Progetto inizializzato con design system definito
- Build pulita prima di iniziare (build-checker deve passare)
- Idealmente un tier già scelto via `TIER`

**Step-by-step:**
1. Il comando verifica che la build sia pulita
2. Fa il backup dei file di design (tailwind.config, globals.css, tokens)
3. Applica i nuovi token coerenti con il tier scelto
4. Aggiorna i componenti UI base (Button, Card, Input) rispettando le API esistenti
5. Propaga il nuovo look a pagine e layout principali
6. Lancia build-checker dopo ogni pagina
7. Se qualcosa si rompe, fa rollback automatico di quella pagina e chiede conferma
8. Aggiorna `.faro/log.md` con tutti i file toccati

**Output atteso:**
- Progetto con nuovo aspetto, stessa logica
- Build verde
- Log dettagliato delle modifiche

**Trigger frasali:**
- "Cambia design"
- "Reskin"
- "Rifai l'aspetto"
- "Nuovo stile"
- "Aggiorna il look"

---

## [AUDIT] — Audit completo

**Comando/skill invocato:** `/project:audit`

**Quando usarlo:**
Quando l'utente vuole una revisione approfondita del progetto: sicurezza, qualità del
codice, coerenza del design, accessibilità, performance. È più pesante di `STATUS` ma
non modifica codice.

**Prerequisiti:**
- Progetto inizializzato
- Build funzionante (altrimenti l'audit si ferma sui primi errori)

**Step-by-step:**
1. Esegue build + typecheck + lint
2. Scansiona per pattern vulnerabili (SQL string concat, eval, IDOR, CORS aperto)
3. Controlla headers di sicurezza, auth, sessioni
4. Verifica a11y di base (alt, label, contrasto token)
5. Cerca TODO/FIXME/XXX lasciati nel codice
6. Controlla coerenza del design system rispetto al tier dichiarato
7. Stila un report con priorità ALTA/MEDIA/BASSA
8. Salva il report in `.faro/audits/YYYY-MM-DD.md`

**Output atteso:**
- Report dettagliato con lista di problemi priorizzati
- Suggerimenti di fix per ciascuno
- Score aggiornato in `.faro/state.md`

**Trigger frasali:**
- "Audit"
- "Fai una revisione completa"
- "Controlla tutto"
- "Security audit"

---

## [ADVISOR] — Consulente prodotto

**Comando/skill invocato:** `/project:advisor`

**Quando usarlo:**
Quando l'utente ha dubbi strategici sul prodotto: quali feature costruire, quale MVP
tagliare, come prioritizzare, se lanciare adesso o dopo. È un aiuto di product thinking,
non di codice.

**Prerequisiti:**
- `CLAUDE.md` e idealmente `.faro/PRD.md` o `MVP.md` presenti
- Una minima idea del target utente

**Step-by-step:**
1. Legge PRD / MVP / state.md
2. Chiede all'utente quale decisione sta valutando
3. Propone 2-3 direzioni possibili con pro/contro
4. Applica euristiche (Pareto sulle feature, cost of delay, RICE)
5. Raccomanda una direzione motivata
6. Se l'utente concorda, aggiorna il PRD e crea un nuovo piano in `.faro/plans/`

**Output atteso:**
- Decisione documentata
- PRD eventualmente aggiornato
- Nuovo piano se l'utente vuole agire

**Trigger frasali:**
- "Cosa dovrei fare dopo?"
- "Aiutami a decidere"
- "Consulente prodotto"
- "Product advisor"
- "Vale la pena questa feature?"

---

## [AP] — Autopilot prossimo task

**Comando/skill invocato:** `faro-autopilot`

**Quando usarlo:**
Quando l'utente vuole che Faro esegua autonomamente il prossimo task del piano attivo
senza intervento, ma un solo task per volta. Modalità conservativa.

**Prerequisiti:**
- Piano attivo in `.faro/plans/`
- Build pulita (raccomandato)
- Nessuna modifica non committata conflittuale

**Step-by-step:**
1. Legge il piano attivo
2. Trova il primo task non completato
3. Mostra all'utente quali file intende toccare
4. Chiede conferma implicita (solo se il task è rischioso, altrimenti procede)
5. Esegue il task
6. Lancia build-checker
7. Aggiorna il piano marcando il task come completato
8. Aggiorna `.faro/log.md`
9. Si ferma, restituisce il controllo all'utente

**Output atteso:**
- Un task completato
- Build verde
- Log aggiornato
- Utente pronto a decidere il prossimo passo

**Trigger frasali:**
- "Autopilot"
- "Vai avanti"
- "Prossimo task"
- "AP"
- "Fai il prossimo"

---

## [AP FULL] — Autopilot completo (con gate)

**Comando/skill invocato:** `faro-autopilot --full`

**Quando usarlo:**
Quando l'utente vuole che Faro completi tutti i task rimanenti del piano attivo in sequenza,
fermandosi solo su gate di sicurezza (build rossa, modifica al DB, task ambiguo, 3 errori
consecutivi).

**Prerequisiti:**
- Piano attivo con più task
- Build pulita all'inizio
- Esplicita accettazione del rischio da parte dell'utente

**Step-by-step:**
1. Legge il piano attivo
2. Entra in loop: per ogni task rimanente
3. Esegue il task
4. Lancia build-checker ogni 3 file
5. Se build rossa, tenta fix automatico (max 1 tentativo), altrimenti STOP e gate
6. Se il task richiede una modifica DB, STOP e chiede conferma
7. Se 3 errori consecutivi sullo stesso punto, STOP e riassume i tentativi
8. Alla fine del piano, lancia `context-updater` per aggiornare `CLAUDE.md` e `state.md`

**Output atteso:**
- Piano completato o fermo su gate chiaro
- `CLAUDE.md` e state.md aggiornati
- Log completo di tutte le azioni

**Trigger frasali:**
- "Autopilot full"
- "AP full"
- "Fai tutto"
- "Completa il piano"
- "Vai fino in fondo"

---

## [PORT] — Porta su altra piattaforma

**Comando/skill invocato:** `faro-multiplatform`

**Quando usarlo:**
Quando l'utente vuole portare il progetto su una piattaforma diversa: da web a mobile,
da Next.js a Remix, da Vercel a self-hosted, da React Native a Flutter, ecc. È un'operazione
rara e molto invasiva.

**Prerequisiti:**
- Progetto stabile con build verde
- Nessun piano attivo (o completato)
- Backup committato su git

**Step-by-step:**
1. Chiede piattaforma di destinazione e vincoli
2. Analizza la struttura attuale e cosa è portabile
3. Identifica componenti specifici della piattaforma sorgente (da riscrivere)
4. Propone una strategia: rewrite completo, adapter layer, monorepo
5. Crea un piano di porting in `.faro/plans/`
6. Richiede conferma esplicita prima di iniziare qualsiasi scrittura
7. Esegue solo su comando dell'utente, non in automatico

**Output atteso:**
- Piano di porting dettagliato
- Analisi di fattibilità
- Stima costo e rischio

**Trigger frasali:**
- "Porta su mobile"
- "Migra a Remix"
- "Vogliamo anche la versione iOS"
- "Port su altra piattaforma"

---

## Quando NON serve Faro

L'orchestratore Faro è progettato per operazioni di alto livello sul ciclo di vita del
progetto. NON deve essere invocato in queste situazioni:

**Task banali / one-liner**
- Fix di un typo, rinomina di una variabile, correzione di un import
- Aggiunta di un console.log, di un commento, di una prop statica
- Modifiche puramente cosmetiche a un singolo file

**Domande informative pure**
- "Cosa fa questa funzione?"
- "Spiegami questo file"
- "Come funziona questo pattern?"
- "Documenta questa classe"

**Debug puntuale**
- "Perché questo test fallisce?"
- "Correggi questo errore"
- Eccezione: se l'errore è sistemico (più file) e ci sono 3+ tentativi falliti,
  allora `FARO` può essere invocato per passare all'autopilot con gate.

**Ricerca di codice**
- "Dove è definita questa funzione?"
- "Trovami tutti i file che usano X"

**Modifiche esplicitamente puntuali richieste dall'utente**
- "Modifica SOLO questo file e basta"
- "Non fare altro, solo questa riga"

**Regola pratica:**
Se un task richiede meno di 3 file toccati, meno di 5 minuti di lavoro, e non cambia
l'architettura o il design system, l'orchestratore Faro NON va invocato. Rispondere
direttamente con le skill standard di Claude Code.

Se invece il task tocca architettura, design system, più di 3 file, richiede un piano,
o tocca il ciclo di vita del progetto (init, adopt, reskin, audit, autopilot, port),
allora Faro è il punto di ingresso corretto.
