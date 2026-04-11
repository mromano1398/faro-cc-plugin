---
name: faro-legacy
description: |
  Strategia di migrazione da codebase legacy. Analisi, piano incrementale, strangler fig.
  Trigger: "migrazione", "legacy", "refactor completo", "modernizzare"
---

# faro-legacy — Migrazione Legacy · Strangler Fig · MySQL→PostgreSQL

**Lingua:** Sempre italiano.
**Principio:** Una migrazione legacy reale richiede mesi, non settimane. Non fare big bang — usa il pattern Strangler Fig per migrare feature per feature con zero downtime.

## QUANDO USARE

- Migrazione da PHP/jQuery/MySQL/old Node.js a stack moderno
- Pianificazione Strangler Fig (migrazione graduale)
- Conversione MySQL → PostgreSQL
- Bridging sessioni legacy → NextAuth
- Coesistenza old system + new system durante transizione

## FLUSSO DECISIONALE

```
1. PRIMA di tutto → Inventario legacy in faro/legacy-inventory.md
   (pagine, tabelle, SP, trigger, cron, file upload, URL, auth, integrazioni)

2. Scegli strategia:
   Pochi mesi + team piccolo → Strangler Fig graduale
   Deadline stretta + sistema semplice → Cutover rapido con logout forzato

3. Configura Nginx routing ibrido (vecchio + nuovo in parallelo)

4. Migra feature in ordine: lettura → creazione → modifica → cancellazione

5. Cutover finale → legacy read-only 30 giorni → poi spento
```

## REGOLE CHIAVE

1. **Inventario PRIMA del codice** — non scrivere una riga senza conoscere tutto il sistema
2. **Strangler Fig, non big bang** — migra una feature alla volta
3. **Backup DB prima di ogni fase** — pg_dump obbligatorio
4. **Verifica count record** — originale = nuovo per ogni tabella
5. **Rollback plan documentato per ogni fase** — chi decide, procedura, max 15 minuti
6. **Session bridging testato** — utenti non devono essere disconnessi durante migrazione
7. **Redirect 301 per tutti i vecchi URL** — SEO + UX
8. **Stored procedures → application layer** — non convertire 1:1 in PostgreSQL
9. **Dual-write è complesso** — considera cutover veloce come alternativa
10. **Legacy read-only 30 giorni** dopo il cutover prima di spegnerlo

## CHECKLIST SINTETICA

- [ ] Inventario completo: pagine, tabelle, SP, trigger, cron, file upload, URL
- [ ] Strangler Fig: proxy configurato, routing ibrido testato
- [ ] MySQL→PG: conversione tipi verificata, no dati corrotti silenziosamente
- [ ] Stored procedures: tutte mappate → application layer
- [ ] Cron job: tutti migrati con equivalente moderno
- [ ] File upload: migrati in storage nuovo, path aggiornati in DB
- [ ] Session bridging: testato che utenti loggati non vengono disconnessi
- [ ] URL redirect: 301 per tutti i vecchi URL, testati con curl
- [ ] Rollback plan documentato per ogni fase
- [ ] Verifica count record: originale = nuovo per ogni tabella
- [ ] Smoke test su 10 operazioni critiche post-migrazione

## PATOLOGIE COMUNI MYSQL→POSTGRESQL

1. **Charset latin1 → UTF8** — verifica con `information_schema.TABLES`
2. **Date `0000-00-00`** — converti in NULL prima del dump
3. **TINYINT(1) → BOOLEAN** — cast esplicito durante migrazione
4. **AUTO_INCREMENT → IDENTITY** — resetta sequences con `setval()` al MAX
5. **ENUM MySQL → CHECK constraint** o tipo custom PostgreSQL
6. **Collation case-insensitive** — usa `citext` o `LOWER()` in query
7. **`LIMIT x,y` → `LIMIT y OFFSET x`**
8. **`IFNULL` → `COALESCE`, `IF()` → `CASE WHEN`**

## REFERENCES

Per dettagli tecnici completi, leggi:
- [references/migration-strategies.md] — inventario, Strangler Fig, dual-write, session bridging, MySQL→PG patterns, rollback
