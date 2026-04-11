---
name: faro-python
description: |
  Pattern Python: FastAPI, SQLAlchemy, Alembic, Pydantic, virtual env, packaging.
  Trigger: "fastapi", "python", "sqlalchemy", "django", "flask"
---

# faro-python — FastAPI · SQLAlchemy · Alembic · pytest · Docker

**Lingua:** Sempre italiano. Riferimenti ufficiali:
- FastAPI: https://fastapi.tiangolo.com
- SQLAlchemy: https://docs.sqlalchemy.org
- Alembic: https://alembic.sqlalchemy.org
- Pydantic: https://docs.pydantic.dev
- pytest: https://docs.pytest.org
- Ruff: https://docs.astral.sh/ruff

## QUANDO USARE

- Progetto Python: FastAPI, Django, Flask, script, pipeline dati
- Setup nuovo progetto con virtual environment e pyproject.toml
- API REST con autenticazione JWT
- Migration DB con Alembic
- Test con pytest + httpx (transazione rollback)
- Build Docker per deployment

## FLUSSO DECISIONALE

```
Nuovo progetto:
→ FastAPI (API REST async) | Django (app web + admin) | Flask (prototipo) | Script puro

Setup base FastAPI:
1. Virtual environment + pyproject.toml
2. Struttura src/{api,core,db,models,schemas,services}
3. SQLAlchemy 2.0 async + asyncpg
4. Alembic per migrations
5. Auth JWT con bcrypt
6. pytest con transazione rollback
7. Dockerfile + docker-compose
```

## REGOLE CHIAVE

1. **`pydantic-settings` per config** — mai hardcoded in codice
2. **SQLAlchemy 2.0 async** con `asyncpg` — non la versione sync
3. **Alembic `--autogenerate`** dopo ogni modifica ai models
4. **`expire_on_commit=False`** in `async_sessionmaker` — obbligatorio per async
5. **Test con transazione rollback** — DB sempre pulito dopo ogni test
6. **CORS con origini specifiche** — mai `*` in produzione
7. **`/health` endpoint** sempre implementato (per Docker + monitoring)
8. **User non-root nel Dockerfile** — sicurezza container
9. **Ruff + mypy nel CI** — lint e type check obbligatori
10. **4 workers uvicorn** in produzione (`--workers 4`)

## FLUSSO OPERATIVO

1. **Init**: `python -m venv .venv && source .venv/bin/activate`
2. **pyproject.toml** con dipendenze pinned
3. **Struttura**: `src/{api,core,db,models,schemas,services}`
4. **Config**: `pydantic-settings` con `.env`
5. **DB**: SQLAlchemy 2.0 async + Alembic
6. **Auth**: JWT + bcrypt + dependency `get_current_user`
7. **Test**: pytest + httpx + fixture con rollback
8. **Docker**: multi-stage, user non-root, `/health`

## CHECKLIST

- [ ] Virtual environment attivo e `pyproject.toml` con dipendenze pinned
- [ ] `pydantic-settings` per config da env (no hardcoded)
- [ ] SQLAlchemy 2.0 async con `asyncpg`
- [ ] Alembic configurato, migration autogenerate funzionante
- [ ] Auth JWT con hash bcrypt per password
- [ ] CORS configurato con origini specifiche (non `*`)
- [ ] `/health` endpoint (per Docker + monitoring)
- [ ] Test con transazione rollback (DB sempre pulito)
- [ ] Dockerfile con user non-root
- [ ] Ruff + mypy nel CI

## TABELLA — Quando scegliere cosa

| Framework | Quando |
|-----------|--------|
| FastAPI | API REST moderna, async, type-safe |
| Django | App web completa, admin incluso, ORM sync |
| Flask | Prototipo rapido, microservizio semplice |
| Litestar | Alternative a FastAPI, MSA |
| Typer | CLI tool |
| Streamlit | Dashboard data, ML demo |

## REFERENCES

Per dettagli tecnici completi, leggi:
- [references/fastapi-patterns.md] — pyproject.toml, struttura progetto, config, auth JWT, endpoint pattern, SQLAlchemy async, Alembic, pytest, Dockerfile
