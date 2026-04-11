---
name: faro-testing
description: |
  Pattern di test avanzati: unit, integration, E2E, mocking, fixture.
  Trigger: "scrivi test", "testing strategy", "coverage", "test E2E"
---

# faro-testing — Unit · Integration · E2E · Load

**Lingua:** Sempre italiano. Riferimenti:
- Vitest: https://vitest.dev/guide
- Playwright: https://playwright.dev/docs/intro
- k6: https://k6.io/docs

## QUANDO USARE

- Setup iniziale test suite in un nuovo progetto
- Scrittura test per una nuova feature
- Verifica coverage prima del deploy
- Load test per validare performance sotto concorrenza

## FLUSSO — Scegli il tipo di test

```
Logica pura (calcoli, validators, formatters) → Unit test (Vitest)
Server actions + DB reale → Integration test (Vitest + transazione rollback)
Flusso utente completo → E2E (Playwright)
Concorrenza + performance → Load test (k6)
API con contratto OpenAPI → Contract test (supertest + spectral)
```

## FLUSSO OPERATIVO

1. **Inventario** — quali moduli business logic hanno coverage < 80%?
2. **Unit test** helpers, validators, formatters, calcoli puri
3. **Integration test** Server Actions + DB reale con rollback
4. **E2E** flusso critico (login → operazione principale → logout)
5. **Load test** dove c'è concorrenza (advisory lock, numerazione, stock)
6. **CI** tutti i livelli girano su ogni PR

## REGOLE CHIAVE

1. **Mai mockare il DB negli integration test** — usa DB di test reale con transazioni rollback
2. **Ogni integration test parte pulito** — ROLLBACK garantisce isolamento
3. **Load test obbligatorio su advisory lock** — verifica zero duplicati sotto concorrenza
4. **Coverage ≥ 80% su business logic** — non su componenti UI
5. **DB di test separato da produzione** — mai `.env` di prod nei test
6. **E2E sui flussi critici** — almeno login → operazione principale → verifica
7. **CI deve girare tutti i test** — unit + integration + E2E + load
8. **Fixture deterministiche** — usa seed fissi per date e random
9. **Test indipendenti** — ordine non deve influenzare il risultato
10. **`data-testid` stabili** su elementi critici per E2E

## CHECKLIST

- [ ] Unit test: helpers, calcoli, validators
- [ ] Integration test: server actions con DB reale (transazione rollback)
- [ ] Integration test: flussi critici (evasione, giacenza, numerazione)
- [ ] E2E: flusso principale dell'applicazione
- [ ] Load test: advisory lock → zero duplicati sotto concorrenza
- [ ] Load test: endpoint critici ≥ target performance
- [ ] Coverage ≥ 80% su moduli business logic
- [ ] Tutti i test passano in CI con DB di test isolato
- [ ] Nessun test che usa `.env` di produzione
- [ ] [Solo API] Spec OpenAPI generata e validata con spectral
- [ ] [Solo API] Contract test per ogni endpoint pubblico
- [ ] [Solo API] Test 401/403 per ogni route protetta

## TABELLA DECISIONE

| Cosa testare | Tool | Quando |
|---|---|---|
| Logica pura | Vitest | Sempre |
| Server Action + DB | Vitest + pg | Sempre |
| UI interattiva | Playwright | Flussi critici |
| Concorrenza | k6 | Advisory lock, stock |
| API contract | supertest + spectral | API pubbliche |
| Visual regression | Playwright screenshots | UI stabili |

## REFERENCES

Per dettagli tecnici completi, leggi:
- [references/testing-patterns.md] — setup Vitest, integration con rollback, Playwright E2E, k6 load test, fixture, CI config
