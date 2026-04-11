---
name: faro-stitch
description: |
  Integrazione opzionale con Google Stitch. Importa DESIGN.md → palette/font/spacing.
  MAI obbligatorio — Faro genera design completi da solo.
  Trigger: "stitch", "importa design", "design da stitch"
---

# Faro Stitch — Integrazione opzionale

**Lingua:** Sempre italiano.
**Principio:** MAI obbligatorio. Usalo solo quando l'utente ha un design esterno da importare.
L'output è sempre `faro/design-system.md` aggiornato con i token del design importato.

## QUANDO USARLO

Chiama faro-stitch quando:
- L'utente dice "ho un design su Figma/Stitch", "ho uno screenshot del sito", "ho un file di design"
- faro-wizard chiede "hai già un design?" e l'utente risponde sì
- L'utente vuole un reskin basato su fonte esterna

Non chiamarlo se:
- L'utente vuole scegliere tier e palette da zero (usa faro-tier-system direttamente)
- Non c'è nessun design esterno di riferimento
- L'utente è soddisfatto del design attuale

## FLUSSO

1. L'utente fornisce un DESIGN.md generato da Stitch (o Figma token JSON, CSS variables, URL/screenshot)
2. Estrai: palette, font, spacing, componenti
3. Mappa ai token semantici Faro (primary, secondary, surface, border, ecc.)
4. Identifica il tier più vicino al design descritto
5. Inserisci i token nel `faro/design-system.md`
6. Se l'utente vuole usare Stitch MCP → configura il server MCP

## OPZIONI DI IMPORT

### Opzione 1 — DESIGN.md (preferita)
L'utente ha preparato un file `DESIGN.md` con la descrizione del design.

**Processo:**
1. Leggi `DESIGN.md` per intero
2. Estrai: colori (hex), font, spacing, stile generale
3. Mappa ai token semantici Faro
4. Identifica il tier più vicino
5. Genera `faro/design-system.md`

### Opzione 2 — URL o Screenshot
L'utente fornisce URL o screenshot di un sito di riferimento.

**Processo:**
1. Analizza visivamente l'URL/screenshot
2. Estrai palette dominante (primary, background, accent, testo)
3. Identifica font (Google Fonts simili se non leggibili)
4. Valuta il tier in base alla complessità visiva
5. Documenta le estrazioni in `faro/design-system.md`
6. Nota: l'estrazione è approssimativa — conferma sempre hex con l'utente

### Opzione 3 — Descrizione testuale
L'utente descrive il design a parole.

**Processo:**
1. Raccogli token con domande mirate: "colore bottone principale?", "sfondo chiaro/scuro?", "font preferito?", "stile (minimal/corporate/bold/playful)?"
2. Proponi palette concreta
3. Chiedi conferma prima di generare design system

### Opzione 4 — MCP Stitch (Google Stitch)
Se il server MCP di Google Stitch è disponibile.

**Processo:**
1. Usa il tool MCP per accedere al progetto Stitch
2. Estrai i token di design dal file Stitch
3. Converti i token in formato semantico Faro
4. Genera `faro/design-system.md`

### Opzione 5 — Figma Token JSON (W3C DTCG)
Figma esporta design token via plugin (Tokens Studio) in JSON standard.

**Formato atteso:**
```json
{
  "color": {
    "primary": { "$value": "#3B82F6", "$type": "color" },
    "surface": { "$value": "#FFFFFF", "$type": "color" }
  },
  "typography": {
    "heading": { "$value": { "fontFamily": "Inter", "fontSize": "2rem" }, "$type": "typography" }
  }
}
```

**Processo:** mappa `color.*` → palette Faro, `typography.*` → font, `spacing.*` → scala.

### Opzione 6 — CSS Custom Properties
L'utente incolla un blocco CSS.

**Esempio input:**
```css
:root {
  --color-brand: #E63946;
  --color-bg: #F1FAEE;
  --color-text: #1D3557;
  --font-main: 'Playfair Display', serif;
}
```

Salva anche in `src/styles/tokens.css` senza modificarle.

## OUTPUT: design-system.md

```markdown
# Design System — [NOME PROGETTO]
**Fonte:** [DESIGN.md / URL / Screenshot / Stitch / Descrizione utente]
**Tier:** [1-4]
**Aggiornato:** [DATA]

## PALETTE
| Token | Valore | Fonte |
|-------|--------|-------|
| primary | #[hex] | [origine] |
| surface | #[hex] | ... |

## TIPOGRAFIA
- Font: [nome] — [come importarlo]
- Heading: [size/weight]
- Body: [size/weight]

## NOTE ADATTAMENTO
[Elenco adattamenti rispetto al design originale]
```

## PRINCIPIO "MAI OBBLIGATORIO"

faro-stitch non è mai imposto all'utente. Se non c'è un design esterno, si usa `faro-tier-system` per scegliere tier e palette da zero.

L'import è un acceleratore, non un requisito. Il workflow Faro funziona perfettamente senza faro-stitch.

## NOTA

Stitch è opzionale. Faro genera design personalizzati da solo con il tier system.
Non dipendere MAI da Stitch per funzionare.
