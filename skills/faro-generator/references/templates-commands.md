# Template Commands — 11 file

## 1. reskin.md

```markdown
---
description: Cambia il design del progetto senza rompere la logica. Sceglie nuovo tier/palette e propaga.
---

# /project:reskin

## Flusso

1. Chiedi: "Cosa vuoi cambiare? Colori, font, tier, tutto?"
2. Se cambio tier -> invoca faro-tier-system
3. Se solo palette/font -> chiedi i nuovi valori
4. Opzionale: "Hai un riferimento? URL, screenshot, Stitch?"

5. Classifica OGNI file del progetto:
   - DESIGN: file che contengono solo styling (classi Tailwind, CSS, componenti UI puri)
   - LOGICA: file con business logic, API, validazione, DB
   - MISTO: file con entrambi (la maggior parte)

6. Aggiorna .claude/rules/design-system.md con i nuovi valori

## Update dipendenze (OBBLIGATORIO per cambio tier)

Se il reskin è un UPGRADE di tier (es: Tier 1->2, 2->3, 3->4):
1. Installa le nuove dipendenze:
   - Tier 1->2: `framer-motion`
   - Tier 2->3: `gsap @studio-freight/lenis` (+ eventuali font display)
   - Tier 3->4: `three @react-three/fiber @react-three/drei`
2. Aggiorna `.claude/rules/architecture.md` sezione "Dipendenze esterne" aggiungendo le nuove
3. Aggiorna `.claude/rules/design-system.md` sezione "Tier" col nuovo numero
4. Aggiorna CLAUDE.md overview se serve

Se il reskin è SOLO cambio palette/font (stesso tier): niente nuove dipendenze, solo design-system.md.

## Downgrade di tier
Se il reskin è un DOWNGRADE (es: Tier 3->2), proponi all'utente di:
- Rimuovere le dipendenze non più necessarie (`gsap`, `@studio-freight/lenis`)
- Rimuovere i componenti Tier 3 (ScrollReveal, TextReveal, ParallaxImage)
- Cercare e rimuovere usi di `ScrollTrigger`, `useGSAP`, `Lenis`
⚠️ Downgrade è destructive — conferma esplicita obbligatoria.

7. Piano chirurgico:
   Per ogni file MISTO:
   a. Modifica SOLO le classi Tailwind/componenti UI
   b. MAI toccare: lib/, middleware, actions/, queries/, auth, validazione
   c. build-checker dopo ogni pagina modificata

8. /project:audit per verifica anti-slop
9. context-updater per aggiornare CLAUDE.md

## Regola: "Solo qui o ovunque?"
Quando cambi un componente, chiedi: "Applico questo cambio solo qui o in tutto il progetto?"
Se ovunque -> lancia design-propagator agent.
```

## 2. audit.md

```markdown
---
description: Audit completo del progetto — design anti-slop + sicurezza + performance + accessibilità + CLAUDE.md quality
---

# /project:audit

## 1. Design Anti-AI-Slop
Scansiona il progetto e rileva:
- Inter usato come heading font in Tier 3-4 -> SLOP
- Purple gradient (#7c3aed -> #a855f7) -> SLOP
- Card grid tutte identiche (stesso layout ripetuto >3 volte) -> GENERICO
- "Welcome to Our Platform" o testi placeholder -> SLOP
- rounded-lg su TUTTO senza variazione -> GENERICO
- Metriche a "0" o "1,234" (dati placeholder) -> SLOP
- Nessuna animazione in Tier 2+ -> GENERICO
- Stessi colori di default shadcn senza personalizzazione -> GENERICO

Report: UNICO / PARZIALMENTE GENERICO / AI SLOP
Per ogni issue: file, riga, problema, suggerimento fix.

### Check specifico per tier >= 2

**Come leggere il tier dichiarato:** grep `^## Tier: [0-9]` in `.claude/rules/design-system.md`
(la sezione Tier introdotta dalla v1.1). Il numero estratto è il tier dichiarato. Se grep fallisce
o restituisce multipli match, FLAG "TIER_UNDECLARED: il progetto non dichiara un tier —
esegui /project:reskin o aggiorna manualmente design-system.md".

Se il progetto dichiara Tier 2+ in .claude/rules/design-system.md, verifica:
- [ ] `framer-motion` è in package.json (Tier 2+)
- [ ] Almeno un componente usa `<motion.*>` o `useAnimation` (Tier 2+)
- [ ] Esiste un componente Skeleton o skeleton.tsx (Tier 2+)
- [ ] Esiste un componente EmptyState (Tier 2+)
- [ ] Se Tier 3+: `gsap` e `@studio-freight/lenis` in package.json
- [ ] Se Tier 3+: almeno un componente ScrollReveal o uso di ScrollTrigger
- [ ] Se Tier 4+: `three` + `@react-three/fiber` + `@react-three/drei` in package.json
- [ ] Se Tier 4+: almeno un `<Canvas>` o `useGLTF` nel codice

Se un check fallisce -> FLAG "TIER_MISMATCH: il progetto dichiara Tier N ma manca [feature]".
Suggerimento: "esegui /project:reskin per completare l'upgrade al Tier N".

## 2. Sicurezza
Verifica:
- [ ] Security headers presenti
- [ ] .env in .gitignore
- [ ] Validazione Zod su tutti gli endpoint
- [ ] WHERE user_id su query dati utente
- [ ] Rate limiting su login/API sensibili
- [ ] Password hashate con bcrypt
- [ ] Session httpOnly + secure + sameSite
- [ ] No SQL injection (query parametriche)
- [ ] File upload: MIME + size check

## 3. Performance
- next/image su tutte le immagini (se Next.js)
- Dynamic import per componenti pesanti
- No import non usati
- Bundle size ragionevole
- Lighthouse target per tier: Tier 1-2 >=90, Tier 3 >=80, Tier 4 >=60

## 4. Accessibilità
- Contrasto >= 4.5:1
- Focus visibile
- Alt text
- Heading gerarchici
- Keyboard navigation
- Target: Lighthouse a11y >= 90

## 5. CLAUDE.md Quality
- Comandi documentati e funzionanti?
- Architettura riflette struttura ATTUALE?
- Rules hanno contenuto concreto?
- Stack aggiornato?
- Score: [X]/100 — se < 70 -> proponi /project:update-context
```

## 3. advisor.md

```markdown
---
description: Consulente prodotto con memoria. Suggerisce feature per crescita, engagement, conversione.
---

# /project:advisor $ARGUMENTS

Lancia l'agente faro-advisor con la modalità richiesta.

## Modalità classiche (senza ruolo)

- **crescita**: suggerimenti per acquisizione utenti
- **engagement**: suggerimenti per retention e utilizzo
- **conversione**: suggerimenti per monetizzazione
- **social**: suggerimenti per viralità e condivisione
- **check**: verifica coerenza progetto con rules

## Sub-modalità role-based

Ogni sub-modalità attiva un priming specifico del ruolo. L'advisor risponde
mettendosi nei panni di quel ruolo. Utili per decisioni multi-prospettiva.

### /project:advisor ceo
Prospettiva business / prodotto / mercato. Domande di priming:
1. Questa feature sposta una metrica di business chiave? Quale?
2. Qual è il time-to-value per l'utente pagante?
3. C'è un competitor che lo fa meglio? Come ci differenziamo?
4. Quale segmento di utenti beneficia di più?
5. Se dovessi tagliare il 50% dello scope, cosa resta?

### /project:advisor eng
Prospettiva tecnica / architetturale / scalabilità. Domande di priming:
1. Quali vincoli di performance/scalabilità impone questa scelta?
2. Che debito tecnico introduciamo? Come lo ripagheremo?
3. Il pattern è coerente con `.claude/rules/architecture.md`?
4. Ci sono test rilevanti? Cosa copre, cosa no?
5. Come si comporta con 10x dati/utenti rispetto a oggi?

### /project:advisor design
Prospettiva UX / accessibilità / coerenza visiva. Domande di priming:
1. Il flusso utente è chiaro senza istruzioni?
2. L'interazione rispetta `design-system.md` e `accessibility.md`?
3. Ci sono stati errori/empty state/loading gestiti?
4. Mobile e desktop hanno parità funzionale?
5. Un utente screen-reader può completare il flusso?

### /project:advisor pm
Prospettiva priorità / roadmap / risorse. Domande di priming:
1. Dove si colloca nella roadmap attuale? (`docs/roadmap.md`)
2. Quali feature vengono rimandate per farle spazio?
3. Effort stimato vs valore atteso (alto/medio/basso)
4. Quali rischi di scope creep vedi?
5. Cosa blocca l'inizio: dipendenze, decisioni, input mancanti?

## Comportamento di default

Se `$ARGUMENTS` è vuoto -> fai un check generale e mostra le 4 prospettive
(ceo/eng/design/pm) in sintesi, poi suggerisci la sub-modalità più rilevante
da approfondire.
```

## 4. update-context.md

```markdown
---
description: Aggiorna rules e CLAUDE.md con lo stato attuale del progetto dopo modifiche significative.
---

# /project:update-context

Lancia l'agente context-updater che:
1. Scansiona tutte le route/pagine -> aggiorna architecture.md
2. Scansiona moduli/componenti -> aggiorna architecture.md
3. Legge schema DB (prisma/migrations) -> aggiorna architecture.md
4. Aggiorna CLAUDE.md con stato attuale
5. Verifica che tutti i file referenziati nelle rules esistano ancora
```

## 5. status.md

~~~markdown
---
description: Mostra stato del progetto, score, piano attivo, log recenti.
---

# /project:status

1. Leggi .faro/state.md
2. Calcola score /100 (fondamenta + sicurezza + design + funzionalità + qualità)
3. Mostra piano attivo (se esiste) con checkbox
4. Mostra ultimi 5 log da .faro/log.md

Formato output:
```
📊 STATO — [nome progetto]
━━━━━━━━━━━━━━━━━━━━━
Fase:     [N] — [nome fase]
Score:    [N]/100
Stack:    [stack principale]
Tier:     [N] — [nome tier]

📋 Piano attivo: [nome]
  [x] Task 1 completato
  [x] Task 2 completato
  [ ] Task 3 — in corso
  [ ] Task 4

📝 Ultimi log:
  [data] [azione]
  [data] [azione]
```
~~~

## 6. review.md

```markdown
---
description: Review di coerenza — verifica che il codice rispetti tutte le rules del progetto.
---

# /project:review

## Flusso a 2 stadi

### Stadio 1 — Conformità al piano
Se c'è un piano attivo in .faro/plans/:
- Ogni task è implementato come descritto?
- File toccati corrispondono a quelli previsti?
- Scope: sono stati toccati file NON nel piano?

### Stadio 2 — Conformità alle rules
Per ogni file modificato di recente (git diff):
- design-system.md: colori, font, componenti sono quelli definiti?
- security.md: validazione, auth, IDOR rispettati?
- code-style.md: naming, import, TypeScript strict?
- api-conventions.md: formato risposta, error codes?
- accessibility.md: contrasto, focus, aria?

## Output
Per ogni issue: severità (CRITICO | IMPORTANTE | MINORE), file, descrizione, fix suggerito.
Issues CRITICHE bloccano il progresso.
```

## 7. timeline.md

```markdown
---
description: Genera un report cronologico di tutto ciò che è stato fatto nel progetto.
---

# /project:timeline

Leggi .faro/log.md e genera un report leggibile:

## Timeline — [nome progetto]

### [Data]
- [ora] [azione] — [dettagli]

Raggruppa per data. Evidenzia: milestone raggiunte, problemi risolti, decisioni prese.
```

## 8. explore.md

```markdown
---
description: Esplorazione intelligente del codebase — dependency graph, dead code, coupling nascosto, churn/hotspot, ownership, tech debt, decision freshness.
---

# /project:explore $ARGUMENTS

Analisi completa del codebase con 8 dimensioni. Se $ARGUMENTS specifica una dimensione
("explore dead-code", "explore churn"), esegui solo quella. Senza argomenti, esegui tutte.

## 1. Dependency graph
Mappa chi importa chi:
```bash
grep -rn "^import\|^from.*import\|require(" src/ --include="*.ts" --include="*.tsx" --include="*.py" --include="*.js" | sort
```
Output: grafo testuale dei moduli principali con frecce (A -> B = A importa B).
Segnala: dipendenze circolari (A->B->C->A).

## 2. Dead code detection
Trova codice potenzialmente morto:
```bash
# Export non usati: cerca ogni "export" e verifica se qualcuno lo importa
for f in $(find src/ -name "*.ts" -o -name "*.tsx"); do
  exports=$(grep -oP 'export (function|const|class|type|interface) \K\w+' "$f")
  for exp in $exports; do
    count=$(grep -r "$exp" src/ --include="*.ts" --include="*.tsx" -l | grep -v "$f" | wc -l)
    [ "$count" -eq 0 ] && echo "CANDIDATE: $exp in $f (0 importers)"
  done
done
```
Ignora file che non hanno semantica di import/export: config, migration, seed, schema,
page.tsx, layout.tsx, route.ts, middleware.ts, *.d.ts, *.config.*, __init__.py, conftest.py.
Ignora pattern dinamici: *Plugin, *Handler, *Middleware, *Adapter, register_*, on_*.
Output: lista di candidati con confidence (alta/media/bassa). MAI dire "e' morto" — dire
"Possibile dead export" o "Possibile file orfano" (solo candidati).

## 3. Coupling nascosto
File che cambiano insieme (git log) ma non si importano:
```bash
# Trova coppie di file che compaiono spesso nello stesso commit
git log --pretty=format:"%H" --since="3 months ago" | while read hash; do
  git show --stat --name-only "$hash" | grep "^src/" | sort
done
# Analizza: se A e B compaiono insieme in >3 commit ma grep non trova import tra loro
# -> accoppiamento nascosto (forse condividono un concetto non esplicito)
```
Output: coppie sospette con numero di co-change e motivo possibile.

## 4. Churn / hotspot
File piu' modificati = probabili hotspot:
```bash
git log --pretty=format: --name-only --since="3 months ago" | sort | uniq -c | sort -rn | head -20
```
Regola: file con >10% dei commit totali = hotspot. Questi file meritano piu' test e
piu' attenzione nelle review.
Output: top 20 file + percentuale commit + suggerimento (piu' test? refactor? split?).

## 5. Ownership mapping
Chi ha scritto cosa:
```bash
for dir in src/*/; do
  echo "=== $dir ==="
  git log --pretty=format:"%an" --since="6 months ago" -- "$dir" | sort | uniq -c | sort -rn | head -3
done
```
Output: per ogni modulo, primary owner (chi ha fatto piu' commit).
Uso: quando tocchi un modulo, il primary owner e' la persona giusta da consultare.

## 6. Tech debt scan
```bash
grep -rn "TODO\|FIXME\|HACK\|@todo\|XXX\|WORKAROUND" src/ --include="*.ts" --include="*.tsx" --include="*.py"
```
Output: lista raggruppata per file con conteggio. File con >5 TODO = tech debt hotspot.

## 7. Route e link integrity
```bash
# Route definite (Next.js App Router)
find src/app -name "page.tsx" | sed 's|src/app||;s|/page.tsx||'
# Link usati
grep -rn 'href="/\|<Link.*to="/\|router.push("/\|navigate("/' src/ --include="*.tsx"
```
Segnala: route senza link (pagine raggiungibili solo via URL diretto).

## 8. Decision freshness
Se esiste `.faro/decisions/`:
- Lista le decisioni architetturali documentate
- Per ogni decisione: estrai i "File coinvolti" e la data
- Conta quante volte ciascun file e' stato modificato dopo la data della decisione:
  `git log --oneline --since="[data decisione]" -- [file]`
- Se un file coinvolto e' stato modificato >3 volte dopo la decisione -> FLAG
  "Decisione potenzialmente stale — i file coinvolti sono cambiati significativamente"
- Decisioni con stato "deprecata" o "sostituita" non generano alert

Se `.faro/decisions/` non esiste: suggerisci di crearlo con
"Vuoi documentare le decisioni architetturali? Posso creare .faro/decisions/ con un template."

## Output finale
Report strutturato con le 8 sezioni. Per ogni sezione:
- Risultati concreti (numeri reali, non "analisi in corso")
- Suggerimenti actionable (cosa fare con queste informazioni)
- Priorita' (CRITICO / IMPORTANTE / INFORMATIVO)
```

## 9. debug.md

```markdown
---
description: Debugging guidato con tecnica dei 5 perché. Segue il flusso di debugging.md rule.
---

# /project:debug $ARGUMENTS

Applica il flusso di debugging definito in .claude/rules/debugging.md:

1. RIPRODUCI il bug descritto in $ARGUMENTS
2. ISOLA dove si manifesta
3. ROOT CAUSE con tecnica 5 perché
4. FIX della root cause
5. VERIFICA con build check
6. PREVIENI con test

Se dopo 3 tentativi non risolto -> STOP e presenta report all'utente.
```

## 10. commit.md

```markdown
---
description: Crea commit atomici conventional dai cambiamenti attuali. Nessun push automatico.
---

# /project:commit

Flusso per commit ergonomico e sicuro. NON esegue mai `git push`.

1. `git status` — vedi cosa è cambiato (staged + unstaged + untracked)
2. `git diff` e `git diff --staged` — capisci il contenuto dei cambi
3. Raggruppa i cambi in commit logici atomici:
   - Un commit = un cambiamento logico (feat, fix, refactor, docs, test, chore)
   - Evita commit "misti" che toccano aree scollegate
4. Per ogni gruppo proponi:
   - Tipo conventional: `feat | fix | refactor | docs | test | chore | style | perf`
   - Scope opzionale: `feat(auth): ...`
   - Messaggio breve all'imperativo (max 72 char nel subject)
   - Body opzionale con contesto se la modifica non è banale
5. MAI committare automaticamente file sospetti:
   - `.env`, `.env.local`, credenziali, chiavi API -> segnala e fermati
   - File binari grandi, build output, `node_modules` -> segnala e fermati
6. Mostra all'utente il piano di commit (lista gruppi + messaggi) e chiedi conferma
7. Esegui `git add` mirato (mai `git add .`) e `git commit` per ogni gruppo
8. NON eseguire `git push`. Se l'utente vuole push, lo chiederà esplicitamente.
```

## 11. cleanup.md

```markdown
---
description: Scansiona e rimuove dead code, import/export/dipendenze inutilizzate con workflow sicuro per step.
---

# /project:cleanup

## Flusso

1. **Scan** — Esegui strumenti di detection:
   ```bash
   npx knip                                    # File, export, dipendenze inutilizzate
   npx depcheck                                # Dipendenze npm inutilizzate
   npx ts-prune                                # Export TypeScript inutilizzati
   npx eslint . --report-unused-disable-directives  # Direttive eslint inutili
   ```
   Se un tool non è installato, skippa con nota.

2. **Categorizza** per rischio:
   | Categoria | Rischio | Esempi |
   |-----------|---------|--------|
   | SAFE | Basso | Dipendenze npm inutilizzate, export non referenziati |
   | CAREFUL | Medio | File apparentemente inutilizzati (verifica dynamic import) |
   | RISKY | Alto | API pubbliche, componenti con import dinamico |

3. **Mostra report** all'utente con lista categorizzata. Chiedi conferma prima di procedere.

4. **Rimuovi in ordine sicuro** (sempre in questo ordine):
   a. **Dipendenze** npm inutilizzate → test → commit
   b. **Export** inutilizzati → test → commit
   c. **File** inutilizzati (solo SAFE) → test → commit
   d. **Duplicati** (scegli implementazione migliore) → test → commit

5. **Safety checklist** per ogni elemento rimosso:
   - [ ] Tool di detection conferma inutilizzato
   - [ ] Grep conferma zero referenze (inclusi dynamic import)
   - [ ] Non fa parte di API pubblica
   - [ ] Test passano dopo rimozione

6. **Build check finale**: `npm run build && npx tsc --noEmit`

## Dead code analysis (pre-cleanup)
Prima di rimuovere qualsiasi cosa, analizza con metodi indipendenti dai tool npm:

### Export non usati
```bash
for f in $(find src/ -name "*.ts" -o -name "*.tsx"); do
  for exp in $(grep -oP 'export (function|const|class|type|interface) \K\w+' "$f"); do
    importers=$(grep -rl "$exp" src/ --include="*.ts" --include="*.tsx" | grep -v "$f" | wc -l)
    [ "$importers" -eq 0 ] && echo "CANDIDATE: $exp in $f (0 importers)"
  done
done
```

### File senza importers
```bash
for f in $(find src/ -name "*.ts" -o -name "*.tsx" | grep -v "page.tsx\|layout.tsx\|route.ts\|middleware.ts"); do
  basename=$(basename "$f" .tsx | sed 's/.ts$//')
  importers=$(grep -rl "$basename" src/ --include="*.ts" --include="*.tsx" | grep -v "$f" | wc -l)
  [ "$importers" -eq 0 ] && echo "CANDIDATE: $f (0 importers, non e' entry point)"
done
```

### Regola di sicurezza
- MAI rimuovere automaticamente — mostra la lista e chiedi conferma
- Classifica: SAFE (nessun dubbio), PROBABILE (quasi certamente morto), INCERTO (potrebbe essere usato dinamicamente)
- Rimuovi solo SAFE e PROBABILE-con-conferma
- Dopo ogni rimozione: `npx tsc --noEmit` per verificare

## Quando NON fare cleanup
- Durante sviluppo attivo di una feature
- Prima di un deploy in produzione
- Senza copertura test adeguata
- Su codice che non capisci
```
