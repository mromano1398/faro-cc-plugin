---
name: faro-beginner
description: |
  Flusso semplificato per utenti non tecnici. Max 3 domande, zero jargon,
  tier semplificati (Semplice/Bello/Wow/3D), 6 profili preconfezionati.
  Trigger: invocato da faro-wizard quando rileva linguaggio non tecnico
---

# Faro Beginner — Per chi non programma

## Flusso semplificato

### Domanda 1: "Cosa vuoi costruire?"

Presenta 6 profili:
1. 🏪 **Sito vetrina** — Presentare la tua attività, servizi, contatti
2. 📊 **Gestionale** — Gestire dati, magazzino, clienti, fatture
3. 🛒 **Negozio online** — Vendere prodotti con pagamenti
4. 📱 **App mobile** — App per telefono (iOS/Android)
5. 🔧 **Tool personale** — Uno strumento per te o il tuo team
6. 🎨 **Portfolio** — Mostrare i tuoi lavori con stile

Oppure: "Descrivimelo a parole tue e scelgo io il profilo giusto."

### Domanda 2: "Come vuoi che sia?"

| Livello | Nome | Cosa significa |
|---------|------|----------------|
| ⚡ | Semplice | Funziona bene, aspetto pulito, niente fronzoli |
| ✨ | Bello | Animazioni fluide, aspetto professionale |
| 🌟 | Wow | Effetti cinematografici, design da premio |
| 🚀 | 3D | Scene tridimensionali interattive |

### Domanda 3 (se serve): "Chi lo userà?"
Solo se non è chiaro dal profilo: clienti, dipendenti, pubblico generico.

### Output
Restituisci al wizard:
- tipo_progetto: [mappato dal profilo]
- tier: [mappato dal livello]
- target_utente: [chi lo usa]
- modalità: beginner (per formattazione non tecnica di tutti gli output successivi)

## Glossario (usato in tutto il flusso beginner)

| Termine tecnico | Spiegazione |
|----------------|-------------|
| Deploy | Mettere il sito online |
| Database | Dove i dati vengono salvati |
| Auth | Il sistema di login |
| API | Come il sito parla con altri servizi |
| Responsive | Si adatta a telefono e computer |
| Hosting | Il "computer" dove vive il sito |

## Riferimenti
Per dettagli di ogni profilo e mappatura a tipo/tier/stack, leggi `references/beginner-profiles.md`.
