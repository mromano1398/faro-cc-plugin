# Beginner Profiles — Guida dettagliata

## Introduzione

Questo documento è il **manuale operativo** della skill `faro-beginner`. Lo usi quando davanti hai una persona che **non programma**, non sa cosa sia un database e ha solo un'idea in testa del tipo "vorrei un sito per la mia attività".

### Filosofia del flusso beginner
1. **Zero jargon**: mai parole come "framework", "stack", "deploy", "API", "CI/CD". Se devi usarle, le traduci subito.
2. **Frasi corte**: massimo 2 righe per concetto. Niente paragrafi densi.
3. **Max 3 domande prima della proposta**: più di 3 domande e l'utente si spaventa. Se servono più info, le chiedi dopo la proposta.
4. **Esempi concreti sempre**: non dire "animazioni fluide", dì "come quando scrolli il sito di Apple".
5. **Valori di default intelligenti**: non chiedere cose che puoi decidere tu. Se sceglie "sito vetrina + semplice", tu sai già lo stack, non glielo chiedere.
6. **Mostra il risultato, non il processo**: l'utente beginner non vuole sapere che userai Next.js 16 con Tailwind. Vuole sapere "il tuo sito sarà veloce, bello sul telefono, e potrai aggiornarlo da solo".
7. **Rassicura sui costi**: è la prima ansia. Parla subito di prezzi indicativi.

### Come usare questo file
- Quando l'utente sceglie un profilo, rileggi la sezione corrispondente per avere la mappatura tecnica corretta.
- Le "feature tipiche" sono il punto di partenza: le proponi all'utente come **pacchetto preconfezionato**, non come lista di opzioni.
- Le "domande extra opzionali" sono riserve: le usi solo se dopo le 2-3 domande base manca ancora un dato critico.
- Il tono è sempre **caldo, umano, da amico che ne capisce**. Non da consulente IT.

---

## I 6 profili in dettaglio

### 1. 🏪 Sito vetrina

**Cos'è in parole semplici**
Un sito che racconta chi sei e cosa offri. Serve per essere trovato su Google, per dare fiducia ai nuovi clienti, per avere un posto dove mandarli quando chiedono "hai un sito?". Non vende online, non gestisce dati complessi: è una **vetrina**, come quella di un negozio in strada.

**A chi serve (esempi concreti)**
- Il barbiere di quartiere che vuole mostrare orari, prezzi e dove si trova
- Lo studio di avvocati con 3 soci che vuole apparire professionale
- Il ristorante che vuole mettere online il menu e permettere di prenotare
- L'idraulico freelance che vuole essere chiamato da chi cerca "idraulico a Torino"
- La palestra che vuole mostrare corsi, orari e far iscrivere online

**Mappatura tecnica interna (non mostrare all'utente)**
- tipo_progetto: `sito-vetrina` / `marketing-site`
- tier consigliato di default: **Tier 1 (Semplice)** — upgradabile a Tier 2 se chiede "bello"
- stack consigliato: Next.js + Tailwind + shadcn/ui, deploy su Vercel, CMS opzionale (Sanity o Notion)
- costo stimato mensile: **0–15€** (hosting gratis su Vercel, dominio ~12€/anno)
- complessità: **bassa** (1-3 giorni di lavoro per la versione base)

**Feature tipiche incluse**
- Una pagina principale con presentazione e call-to-action ("Chiamaci", "Prenota")
- Pagina "Chi siamo" con foto e storia
- Pagina "Servizi" o "Menu" con dettagli
- Pagina "Contatti" con form, indirizzo, mappa, telefono
- Versione che funziona perfettamente su telefono
- Ottimizzato per essere trovato su Google
- Possibilità di aggiornare testi e foto da solo (se sceglie il CMS)

**Feature NON incluse (cose che l'utente spesso chiede ma non servono qui)**
- Login utenti (non serve, è un sito pubblico)
- Carrello e pagamenti (quello è un negozio online, profilo diverso)
- Gestione prenotazioni complesse (basta un form semplice, per prenotazioni vere serve un gestionale)
- Chat in tempo reale (sovraccarico per una vetrina)

**Domande extra opzionali**
- "Hai già un logo e foto della tua attività, o li creiamo?"
- "Vuoi poter aggiornare i testi da solo senza chiamare nessuno?"
- "Hai già un dominio (tipo `www.ilbarbieredipaolo.it`)?"

**Esempio di proposta finale in linguaggio non-tecnico**
> Perfetto! Ti creo un **sito vetrina semplice e pulito** per il tuo barbiere.
>
> Conterrà:
> - Una pagina di benvenuto con le tue foto e un grosso pulsante "Prenota ora"
> - Gli orari, l'indirizzo con la mappa, il numero di telefono
> - I servizi che offri con i prezzi
> - Un form per farti contattare
>
> Sarà **veloce**, si vedrà bene **sul telefono**, e sarà **facile da trovare su Google** quando qualcuno cerca "barbiere [la tua città]".
>
> Costo: hosting **gratis** il primo anno, poi circa **12€/anno** solo per il dominio.
> Lo metto online in **2-3 giorni**. Ti va?

---

### 2. 📊 Gestionale

**Cos'è in parole semplici**
Un'applicazione privata dove tu (o i tuoi dipendenti) entrate con un login per gestire i dati della vostra attività: clienti, fornitori, magazzino, fatture, appuntamenti, pratiche. Non è per il pubblico: è uno **strumento di lavoro interno**.

**A chi serve (esempi concreti)**
- La piccola azienda con 5 dipendenti che oggi usa Excel e vuole smettere
- Lo studio commercialista che deve tracciare pratiche dei clienti
- L'officina che vuole gestire ordini di lavoro e storico riparazioni
- L'agenzia immobiliare che gestisce immobili, visite e contratti
- Il laboratorio artigianale che vuole tenere traccia di materiali e commesse

**Mappatura tecnica interna**
- tipo_progetto: `gestionale` / `admin-dashboard` / `internal-tool`
- tier consigliato di default: **Tier 2 (Bello)** — la produttività guadagna tanto dal design
- stack consigliato: Next.js + Tailwind + shadcn/ui + Supabase/Neon (DB + auth), Vercel
- costo stimato mensile: **20–80€** (database gestito, hosting, email transazionali)
- complessità: **media-alta** (dipende da quante entità e regole di business)

**Feature tipiche incluse**
- Sistema di login per te e i tuoi dipendenti con ruoli diversi (chi può fare cosa)
- Tabelle con ricerca, filtri, ordinamento per ogni tipo di dato
- Moduli per creare, modificare, eliminare record
- Esportazione dati in Excel o PDF
- Dashboard iniziale con numeri chiave (quanti clienti, quanti ordini aperti)
- Storico delle modifiche (chi ha fatto cosa e quando)
- Backup automatico dei dati

**Feature NON incluse**
- Sito pubblico con SEO (è un'app interna, non serve essere su Google)
- Design cinematografico con animazioni 3D (qui conta la **velocità d'uso**, non l'effetto wow)
- Carrello e checkout pubblico (se serve vendere, è profilo "negozio online")
- App mobile nativa (di solito basta il browser, anche su telefono)

**Domande extra opzionali**
- "Quante persone lo useranno? Solo tu o più colleghi?"
- "Devono avere permessi diversi? (es: il magazziniere vede solo il magazzino)"
- "I dati li avete già da qualche parte? (Excel, vecchio software, carta)"
- "Ci sono fatture o documenti PDF da generare?"

**Esempio di proposta finale**
> Ti creo un **gestionale interno** per la tua officina, accessibile solo da te e i tuoi meccanici con un login.
>
> Potrai:
> - Registrare ogni cliente con tutti i suoi veicoli
> - Aprire "ordini di lavoro" per ogni riparazione
> - Vedere lo **storico completo** di ogni veicolo (cosa abbiamo fatto e quando)
> - Tracciare pezzi di ricambio e quanto costano
> - Vedere quanto hai fatturato questo mese con un grafico
> - Esportare tutto in Excel se ti serve
>
> Funziona dal computer dell'officina, dal telefono, ovunque ci sia internet. I dati sono **al sicuro** e c'è un **backup automatico ogni giorno**.
>
> Costo: circa **30€/mese** per il database e l'hosting. Lo costruiamo in **1-2 settimane**.

---

### 3. 🛒 Negozio online

**Cos'è in parole semplici**
Un sito dove il pubblico può **vedere i tuoi prodotti, metterli nel carrello e pagarli con carta**. Include la gestione degli ordini e delle spedizioni, e una zona riservata a te per aggiungere prodotti e vedere le vendite.

**A chi serve (esempi concreti)**
- L'artigiano che fa borse in pelle e vuole venderle in tutta Italia
- Il produttore di vino locale che vuole spedire bottiglie ai clienti
- La boutique di moda che vuole affiancare l'online al negozio fisico
- Il corso online di yoga che vende videocorsi digitali
- Lo chef che vende kit per ricette gourmet a casa

**Mappatura tecnica interna**
- tipo_progetto: `ecommerce`
- tier consigliato di default: **Tier 2 (Bello)** — un e-commerce brutto non vende
- stack consigliato: Next.js + Tailwind + Shopify Storefront API (o Medusa self-hosted per progetti su misura), Stripe per pagamenti, Vercel
- costo stimato mensile: **40–150€** (Shopify Lite o hosting + DB + Stripe commissioni)
- complessità: **media-alta**

**Feature tipiche incluse**
- Catalogo con foto, descrizioni, prezzi, varianti (taglia, colore)
- Carrello e pagamento con carta (o anche bonifico, Klarna, Apple Pay)
- Account utente per chi compra (storico ordini, indirizzi)
- Pannello per te per aggiungere prodotti, vedere ordini, stampare etichette
- Email automatiche al cliente: conferma ordine, spedito, consegnato
- Calcolo spese di spedizione (per zona, peso, corriere)
- Codici sconto e promozioni
- Statistiche di vendita

**Feature NON incluse**
- Gestione interna complessa di produzione/magazzino fisico (quello è un gestionale)
- Social network interno o commenti ai prodotti tipo forum
- App mobile nativa (il sito è già responsive e funziona benissimo)
- Sistema di dropshipping con 50 fornitori (non è un marketplace)

**Domande extra opzionali**
- "Quanti prodotti avrai più o meno? (10, 100, 1000?)"
- "Vendi solo in Italia o anche all'estero?"
- "Accetti solo carta o anche bonifico/contrassegno?"
- "Hai già un account Stripe o PayPal, o li creiamo?"

**Esempio di proposta finale**
> Ti creo un **negozio online** per le tue borse in pelle.
>
> I clienti potranno:
> - Vedere le foto dei prodotti con zoom
> - Scegliere colore e misura
> - Pagare con **carta** o **PayPal** in sicurezza
> - Ricevere **email automatiche** per conferma e spedizione
>
> Tu avrai un pannello dove:
> - Aggiungi nuove borse con poche righe
> - Vedi tutti gli ordini del giorno
> - Stampi le **etichette di spedizione**
> - Controlli quanto hai venduto nel mese
>
> Spese di spedizione calcolate in automatico. Codici sconto quando vuoi fare promozioni.
>
> Costo: circa **50€/mese** + **1,4% + 25 centesimi** per ogni pagamento con carta (è Stripe, la stessa cosa che usano i grandi siti). In **2-3 settimane** è online.

---

### 4. 📱 App mobile

**Cos'è in parole semplici**
Un'applicazione vera che si scarica dall'**App Store** (iPhone) e **Google Play** (Android). Si installa sul telefono, ha la sua icona, può mandare notifiche, funziona anche offline. Non è un sito aperto col browser: è un'**app nativa**.

**A chi serve (esempi concreti)**
- La palestra che vuole dare un'app ai soci per prenotare lezioni dal telefono
- Lo startupper con un'idea tipo "come Tinder ma per cani" (davvero, capita)
- Il personal trainer che vuole un'app con i workout da assegnare ai clienti
- L'azienda che vuole un'app interna per i tecnici sul campo
- Chi ha un'idea di social con notifiche, chat, geolocalizzazione

**Mappatura tecnica interna**
- tipo_progetto: `mobile-app`
- tier consigliato di default: **Tier 2 (Bello)** o Tier 3 per app consumer
- stack consigliato: Expo + React Native, Supabase o Firebase come backend, EAS per build
- costo stimato mensile: **30–100€** + **99$/anno** Apple Developer + **25$ una tantum** Google Play
- complessità: **alta** (serve anche pubblicazione sugli store, tempi di review Apple)

**Feature tipiche incluse**
- Login con email, Google, Apple, telefono
- Notifiche push (promemoria, avvisi)
- Uso di fotocamera, GPS, contatti (con permessi)
- Funzionamento parziale offline
- Aggiornamenti via store
- Pubblicazione su App Store e Google Play (ci pensiamo noi alla review Apple, che è rognosa)

**Feature NON incluse**
- Sito web pubblico (l'app è per chi la scarica; un sito è un altro progetto)
- Pannello admin complesso (per quello a volte si aggiunge un piccolo gestionale web)
- Videogiochi 3D (quello è un mondo a parte, servono Unity/Unreal)

**Domande extra opzionali**
- "Deve essere su iPhone, Android, o entrambi?"
- "Serve che funzioni anche senza internet?"
- "Ha bisogno di mandare notifiche?"
- "Ha un backend (dati condivisi tra utenti) o funziona da sola?"

**Esempio di proposta finale**
> Ti creo un'**app mobile** per la tua palestra, disponibile sia su **iPhone** che **Android**.
>
> I tuoi iscritti potranno:
> - Fare login con email o Google
> - **Prenotare lezioni** scorrendo il calendario
> - Ricevere **notifiche** ("la tua lezione inizia tra un'ora")
> - Vedere i loro abbonamenti attivi
> - Disdire se non possono venire
>
> Tu avrai un piccolo pannello web dove vedi chi ha prenotato cosa.
>
> L'app sarà pubblicata sugli store (Apple costa **99$/anno**, Google **25$ una volta**). Ci pensiamo noi alla review.
>
> Costo di gestione: circa **40€/mese**. Tempo di sviluppo: **3-4 settimane**.

---

### 5. 🔧 Tool personale

**Cos'è in parole semplici**
Uno strumento **su misura** per un problema specifico che hai tu (o il tuo team). Non è un prodotto da vendere, non è per il pubblico: è una **macchina personalizzata** che ti fa risparmiare tempo ogni giorno. Può essere un calcolatore, un convertitore, un'automazione, una dashboard.

**A chi serve (esempi concreti)**
- Il freelance che vuole un timer + fatturatore per tracciare le ore lavorate
- Il fotografo che vuole un tool per organizzare e taggare migliaia di foto
- La mamma di 3 figli che vuole una dashboard familiare con turni e promemoria
- L'appassionato di trading che vuole grafici custom sui suoi dati
- Chi vuole automatizzare un lavoro ripetitivo (leggere email, riorganizzarle)

**Mappatura tecnica interna**
- tipo_progetto: `personal-tool` / `internal-tool`
- tier consigliato di default: **Tier 1 (Semplice)** — conta la funzione, non l'estetica
- stack consigliato: Next.js + Tailwind + SQLite/Supabase (minimo), Vercel
- costo stimato mensile: **0–20€**
- complessità: **bassa-media**

**Feature tipiche incluse**
- Interfaccia minimale che fa una cosa e la fa bene
- Eventuale login se usato da più persone
- Salvataggio dati su database (o in locale nel browser)
- Pochi pulsanti, pochi schermi
- Veloce e diretto

**Feature NON incluse**
- Landing page di marketing (non devi vendere niente)
- Sistemi di pagamento
- SEO (non ti serve essere trovato)
- Design molto elaborato (sarebbe pure controproducente)

**Domande extra opzionali**
- "Lo usi solo tu o anche altre persone?"
- "I dati devono essere condivisi o privati?"
- "Funziona dal computer, telefono, o entrambi?"

**Esempio di proposta finale**
> Ti creo un **tool personale** per tracciare le ore di lavoro.
>
> Quando inizi, premi "Start", scegli il cliente, e il timer parte. A fine giornata vedi **quante ore hai fatto per chi**, e con un click esporti una **fattura PDF**.
>
> Minimalista, veloce, niente fronzoli. Funziona anche dal telefono.
>
> Costo: **gratis** (hosting gratuito Vercel, database piccolo). Pronto in **3-4 giorni**.

---

### 6. 🎨 Portfolio

**Cos'è in parole semplici**
Un sito per **mostrare il tuo lavoro** in modo bello ed emozionante. Serve a chi crea cose: fotografi, designer, illustratori, architetti, musicisti, sviluppatori, videomaker. Non vende, non gestisce ordini: **impressiona**. L'obiettivo è farti chiamare o assumere.

**A chi serve (esempi concreti)**
- Il fotografo di matrimoni che vuole mostrare i suoi scatti migliori
- L'illustratrice che vuole attirare clienti editoriali
- L'architetto freelance che vuole presentare i progetti realizzati
- Lo sviluppatore che cerca lavoro e vuole far vedere i suoi progetti
- Il videomaker che vuole mostrare showreel e progetti cinematografici

**Mappatura tecnica interna**
- tipo_progetto: `portfolio`
- tier consigliato di default: **Tier 3 (Wow)** — per un portfolio il design è tutto
- stack consigliato: Next.js + Framer Motion + Tailwind, opzionale R3F per Tier 4
- costo stimato mensile: **0–15€**
- complessità: **media** (bassa a livello funzionale, alta a livello design)

**Feature tipiche incluse**
- Home d'impatto con un'animazione forte o un'immagine/video a tutto schermo
- Galleria progetti con filtri (per tipo, anno, cliente)
- Pagina singola per ogni progetto con tante foto e racconto
- Sezione "Chi sono" emozionale
- Contatti con form e social
- Transizioni fluide tra le pagine
- Lightbox per ingrandire le immagini
- Suono opzionale (per videomaker)

**Feature NON incluse**
- Carrello / vendita prodotti (è un portfolio, non un e-commerce)
- Pannello gestionale complesso (aggiornamenti via CMS semplice)
- Login utenti (nessun login serve)

**Domande extra opzionali**
- "Hai già le foto/video dei tuoi lavori pronti?"
- "Vuoi poter aggiungere nuovi progetti da solo?"
- "Hai un dominio o uno nel nome d'arte preferito?"
- "Vuoi effetti tipo parallax, o qualcosa di più sobrio?"

**Esempio di proposta finale**
> Ti creo un **portfolio con effetto wow** per i tuoi progetti di architettura.
>
> Appena si apre, i visitatori vedono un **video panoramico** di uno dei tuoi edifici con il tuo nome. Scrollando scoprono i progetti uno dopo l'altro con **animazioni fluide**.
>
> Cliccando su un progetto si apre una pagina dedicata con tutte le foto, i render, la storia del progetto.
>
> C'è una sezione "Chi sono" con la tua foto e il tuo manifesto, e un form contatti diretto. Tutto funziona anche sul telefono.
>
> Costo: **gratis** (hosting Vercel, dominio escluso). Pronto in **1-2 settimane**.

---

## I 4 livelli in dettaglio

### ⚡ Semplice (Tier 1)

**Cosa significa concretamente**
Un'interfaccia pulita, leggibile, moderna ma senza effetti speciali. Tipografia chiara, spaziature generose, colori solidi, pochi click per fare ogni cosa. Veloce come una scheggia. Zero animazioni superflue.

**Esempi visuali capibili da chiunque**
- Il sito di **Notion** (chiaro, ordinato, pratico)
- Il pannello di **Gmail** (sobrio, funziona e basta)
- Il sito di **Stripe** (pulito, professionale, diretto)

**Mappatura interna**: Tier 1

**Quando è sbagliato sceglierlo**
Se è un portfolio artistico o un progetto che deve impressionare visivamente (brand di moda, creativo, luxury). Lì "semplice" equivale a "anonimo".

---

### ✨ Bello (Tier 2)

**Cosa significa concretamente**
Tutto il Tier 1 + micro-animazioni fluide al passaggio del mouse, transizioni morbide tra pagine, curve eleganti, scroll che si muove con ritmo, piccoli dettagli che fanno pensare "questo è fatto bene". Aspetto professionale da agenzia.

**Esempi visuali**
- Il sito di **Linear** (app di project management — elegantissima)
- Il sito di **Vercel** (transizioni fluide, tipografia curata)
- L'interfaccia di **Arc Browser**

**Mappatura interna**: Tier 2

**Quando è sbagliato sceglierlo**
Se è un puro tool interno dove i dipendenti vogliono solo fare il loro lavoro velocemente. Le animazioni rallentano la produttività.

---

### 🌟 Wow (Tier 3)

**Cosa significa concretamente**
Effetti cinematografici: parallax, transizioni elaborate, elementi che si rivelano mentre scrolli, video a tutto schermo, tipografia gigante animata. È il tipo di sito che ti fa fermare e dire "wow, come l'hanno fatto?". Spesso candidato a premi di design (Awwwards).

**Esempi visuali**
- Il sito di **Apple** (scroll, video, animazioni a ogni sezione)
- Il sito di **Stripe Sessions** (micro-interazioni ovunque)
- I siti che vedi premiati su **Awwwards** o **Godly**

**Mappatura interna**: Tier 3

**Quando è sbagliato sceglierlo**
Per un gestionale interno. Per un tool semplice che deve essere veloce. Per qualsiasi cosa dove l'utente vuole **finire presto** piuttosto che ammirare. E se il cliente ha poco budget: un Tier 3 richiede più tempo di sviluppo.

---

### 🚀 3D (Tier 4)

**Cosa significa concretamente**
Scene tridimensionali interattive: modelli 3D che ruotano, ambienti esplorabili, personaggi animati, effetti glass/liquid con vera fisica, shader custom. È il livello massimo: esperienza da videogioco nel browser.

**Esempi visuali**
- Il sito di **Bruno Simon** (l'auto 3D in mezzo a una città giocabile)
- I lanci di prodotti Apple con scene interattive tipo Vision Pro
- Esperienze immersive di brand come **Lusion**, **Active Theory**

**Mappatura interna**: Tier 4

**Quando è sbagliato sceglierlo**
Quasi sempre, a meno che non sia un lancio di prodotto premium, un sito artistico, un portfolio di motion designer, o un'esperienza promozionale. Per qualsiasi uso quotidiano (gestionali, e-commerce, siti vetrina) è **sovradimensionato**: pesante da caricare, difficile da mantenere, costoso.

---

## Mappatura profilo → livello consigliato

| Profilo | Livello di default | Upgrade sensato | Livello sconsigliato |
|---------|--------------------|-----------------|----------------------|
| 🏪 Sito vetrina | ⚡ Semplice | ✨ Bello (per brand curati) | 🚀 3D |
| 📊 Gestionale | ✨ Bello | ⚡ Semplice (se serve solo velocità) | 🌟 Wow, 🚀 3D |
| 🛒 Negozio online | ✨ Bello | 🌟 Wow (per brand moda/luxury) | 🚀 3D (rallenta conversioni) |
| 📱 App mobile | ✨ Bello | 🌟 Wow (per app consumer ambiziose) | 🚀 3D |
| 🔧 Tool personale | ⚡ Semplice | ✨ Bello (se ci tieni all'aspetto) | 🌟 Wow, 🚀 3D |
| 🎨 Portfolio | 🌟 Wow | 🚀 3D (per creativi audaci) | ⚡ Semplice (troppo anonimo) |

---

## Linguaggio da usare (glossario esteso)

Questo è il **dizionario di traduzione** dal tecnichese all'italiano quotidiano. Usa sempre la colonna di destra quando parli con l'utente beginner.

| Termine tecnico | Come lo dici all'utente |
|-----------------|--------------------------|
| Deploy | Mettere online / pubblicare |
| Database | Dove i dati vengono salvati |
| Frontend | La parte che le persone vedono |
| Backend | La parte dietro le quinte |
| API | Il modo in cui il sito parla con altri servizi |
| Framework | Gli "attrezzi" con cui lo costruisco |
| Stack | Gli strumenti che userò |
| Hosting | Il "computer" dove vive il sito |
| Dominio | L'indirizzo del sito (tipo `ilmiosito.it`) |
| SSL / HTTPS | Il lucchetto verde di sicurezza |
| Auth / Authentication | Il sistema di login |
| Authorization | Chi può fare cosa (permessi) |
| Responsive | Si adatta a telefono, tablet, computer |
| SEO | Farsi trovare su Google |
| CMS | Pannello dove aggiorni testi e foto |
| CDN | Rete che rende il sito veloce ovunque nel mondo |
| Cache | Memoria veloce che evita di ricaricare sempre tutto |
| Repository / Repo | Dove è conservato il codice |
| Git | Il sistema che tiene lo storico delle modifiche |
| Commit | Salvare una modifica nello storico |
| Branch | Una "copia di lavoro" del progetto |
| Pull request | Proposta di cambiamento da approvare |
| Build | Preparare il sito per metterlo online |
| Bug | Errore |
| Debug | Trovare e sistemare errori |
| Endpoint | Un "pulsante" del server |
| Query | Una domanda fatta al database |
| Schema | La "forma" dei dati |
| Migration | Aggiornamento della struttura dei dati |
| Webhook | Una notifica automatica tra servizi |
| Token | Una specie di password temporanea |
| Environment variable | Un'impostazione segreta |
| Cron / scheduled job | Compiti che partono da soli a orari prestabiliti |
| Middleware | Un "controllore" che vede tutto quello che passa |
| Component | Un pezzo riutilizzabile dell'interfaccia |
| Props / State | I dati che un pezzo del sito gestisce |
| Hook | Un "aggancio" per aggiungere comportamenti |
| Serverless | Niente server da gestire, paghi solo quando serve |
| Container / Docker | Una "scatola" che contiene l'applicazione |
| CI/CD | Tutto si aggiorna in automatico quando modifichi il codice |

---

## Frasi pronte

Queste sono risposte **preconfezionate** per le situazioni più comuni. Non copiarle parola per parola, ma usale come stampo per mantenere tono e sostanza giusti.

### Quando chiede il prezzo
> Dipende da cosa costruiamo, ma per darti un'idea:
> - Un sito vetrina semplice: **gratis di hosting**, solo il dominio (~12€/anno)
> - Un gestionale interno: circa **20-50€/mese** per il database
> - Un negozio online: circa **40-100€/mese** + commissioni sui pagamenti
>
> Il grosso del lavoro è nella **creazione**, non nella gestione mensile. Appena ho più chiara la tua idea ti do un numero preciso.

### Quando non sa cosa scegliere
> Tranquillo, è normale! Descrivimi con parole tue **cosa dovrebbe fare la cosa che hai in mente**, o **chi dovrebbe usarla**. Da quello capisco subito quale è il profilo giusto per te. Non serve essere tecnici.

### Quando vuole "tutto"
> Capisco, in testa sembra bello avere tutto. Ma se mettiamo tutto insieme il primo giorno, rischiamo di non finire mai e far salire i costi.
>
> Ti propongo così: partiamo con **l'essenziale che risolve il tuo problema principale**, lo mettiamo online in poche settimane, lo usi davvero. Poi, se serve, aggiungiamo pezzi a mano a mano.
>
> Qual è **la cosa più importante** che deve fare al primo giorno?

### Quando si spaventa per la complessità
> Fermo un attimo. **Non devi capire niente di tutto questo**, è compito mio. Tu mi dici cosa ti serve a parole tue, io ti faccio vedere il risultato.
>
> Pensalo come un idraulico: non devi sapere come funzionano i tubi, devi solo sapere dove vuoi il lavandino. Il resto lo faccio io.

### Quando vuole cambiare idea a metà
> Ottima notizia: si può! Aspetta che ti spieghi cosa significa praticamente.
>
> - Se è una **piccola modifica** (cambiare testi, aggiungere una pagina), si fa in giornata.
> - Se è un **cambio di direzione grosso** (da sito vetrina a negozio online, per esempio), ricominciamo da capo con un profilo diverso — meglio fermarsi ora che dopo.
>
> Dimmi la modifica che hai in testa e ti dico subito in che categoria cade.

### Quando dice "voglio uguale a [sito famoso]"
> Bel riferimento! Guardiamo insieme cosa ti piace di quel sito:
> - È l'**aspetto generale** (colori, tipografia, sensazione)?
> - Sono le **animazioni** (come si muovono le cose)?
> - È la **struttura** (come sono disposte le pagine)?
>
> Ti faccio qualcosa **ispirato** ma **originale tuo**, non una copia. I siti famosi costano centinaia di migliaia di euro, ma possiamo catturarne lo spirito con un budget normale.

### Quando chiede "quanto ci mettiamo"
> Indicativamente:
> - **Sito vetrina semplice**: 2-5 giorni
> - **Portfolio curato**: 1-2 settimane
> - **Negozio online**: 2-4 settimane
> - **Gestionale**: 1-4 settimane (dipende da quante funzioni)
> - **App mobile**: 3-6 settimane (Apple impiega ~1 settimana a revisionarla, non dipende da noi)
>
> Prima del lavoro ti do una stima precisa basata sulle tue esigenze.

---

## Segnali di "upgrade a flusso DEV"

A volte parti con qualcuno che sembra beginner, ma dopo due battute inizia a dirti cose tecniche. In quel caso **passa al flusso completo** di `faro-wizard` — la persona sa di cosa parla e si annoierà con le metafore.

### Segnali forti (passa subito al wizard completo)
- Usa nomi di tecnologie: "voglio che sia in Next.js / React / Vue / Python / Supabase / Postgres / Firebase"
- Parla di architettura: "monorepo", "microservizi", "serverless", "edge runtime"
- Chiede dettagli di hosting/deploy: "dove si deploya?", "CI/CD", "Docker", "Kubernetes"
- Menziona design system tecnico: "shadcn", "Tailwind config", "radix", "headless UI"
- Parla di database in modo tecnico: "PostgreSQL vs MySQL", "ORM", "migrations", "RLS"
- Parla di testing: "unit test", "E2E", "Playwright", "Vitest"

### Segnali deboli (chiedi conferma ma probabilmente sì)
- Dice "lo faccio io ma mi serve una guida" → forse è tecnico junior, chiedi il livello
- Chiede "quali alternative ci sono allo stack?" → chiaramente vuole scegliere lui
- Menziona GitHub, VS Code, terminale → usa strumenti da dev

### Come passare al wizard
> Ah perfetto, vedo che ci capisci! Allora possiamo saltare la versione semplificata e andare diretto al flusso completo, dove puoi scegliere tu **stack, tier tecnico preciso e feature di dettaglio**. Ti va?

Se conferma, richiami `faro-wizard` passando il contesto già raccolto (idea di alto livello e profilo di riferimento).

### Come NON sovra-interpretare
Non passare al wizard completo se dice solo:
- "Sito" → è una parola italiana, non un termine tecnico
- "App" → tutti la dicono
- "Database" → l'avrà sentita al telegiornale
- "Cloud" → idem

Serve più di un segnale chiaro prima di switchare. Nel dubbio, **resta in beginner** e chiedi se gli va di vedere opzioni più tecniche.
